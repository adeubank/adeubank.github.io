---
layout: post
title: Ruby Theading Example with Active Record
date: 2018-09-11 09:02 -0700
categories: programming
tags: ruby thread concurrency
---
I came across a problem where a slow processing job needed to run.
The results of the job did not need to be available in _real time_ so we
could wait 5-10 minutes for the result, no problem.

The trouble began when the results had changed and the results would no longer
be valid. A new job would be enqueued to Sidekiq again to handle the re-processing.
This is where things got interesting because the original job had already begun
work on those rows, they were already locked.

{% highlight ruby %}
MyAwesomeModel.transaction do
  query = MyAwesomeModel.where(awesome_id: 123).where(day: '1999-12-31')
  # lock the rows
  query.lock(true)
  # Delete to start fresh
  MyAwesomeModel.where(awesome_id: 123).where(day: '1999-12-31').delete_all
  # Import
  MyAwesomeModel.connection.query populate_awesome_model(awesome_id: 123, day: '1999-12-31')
end
{% endhighlight %}

At first, letting MySQL handle the deadlocks seemed to work. Just handle the
exception and requeue the worker, letting Sidekiq Unique jobs filter out
any duplicate processing jobs. However, it would be much easier to handle this
in the worker and not add any extra requests to MySQL.

{% highlight ruby %}
redis = Redis.new
redis.lock("my-awesome-lock-name-#{123}-#{'1999-12-31'}") do |lock|
  # ... do the work
end
{% endhighlight %}

This approach using the redis-lock gem was great. Just handle the
`Redis::Lock::LockNotAcquired` exception and requeue.
Retrying in this fashion until the new job can start. This worked until I
realized that the locks have a timeout to ensure that any lock holder has a
finite time with the lock unless the ask for more time.

### Enter Threads

Before this, I typically avoid writing my own multi-threaded code. It's not
worth the hassle for thinking _in concurrent_ terms. My brain was just not
meant for it. For a simple task like this I am fine with it. I needed two
threads to handle this. One for keeping the lock alive, and a second for
doing the work.

After writing a class to handle creating the two threads, the API looked something like this.

{% highlight ruby %}
redis = Redis.new
redis.lock("my-awesome-lock-name-#{123}-#{'1999-12-31'}") do |lock|
  RedisLockUtils.execute_with_lock(lock: lock, logger: Rails.logger) do
    # do the work
  emd
end
{% endhighlight %}

For those with a hungrier appetite, this is the full source for the `RedisLockUtils`
class.

{% highlight ruby %}
class RedisLockUtils
  WARNING_MSG = "Execution with lock is still running!"

  # Uses a two threads, one to keep the lock alive, one to execute the passed in block
  # @param [Redis::Lock] lock
  def self.execute_with_lock(lock:, logger: Rails.logger)
    helper_progname      = "#{self}-Helper L-#{lock.key}"
    worker_progname      = "#{self}-Worker L-#{lock.key}"
    finished             = false
    execution_started_at = nil
    already_warned       = false

    helper = RedisLockThread.new do
      until finished

        unless execution_started_at
          sleep(0.1)
        end

        lock_expires_in = lock.xval.to_i - Time.now.to_i

        if lock_expires_in < 30.seconds
          logger.info(helper_progname) { "Waiting since #{execution_started_at}. Extending. lock=#{lock.key} lock_expires_in=#{lock_expires_in}" }
          lock.extend_life(1.minutes)
        end

        unless already_warned or Time.now.to_i - execution_started_at.to_i < 10.minutes
          logger.warn(helper_progname) { "#{WARNING_MSG} lock=#{lock.key}, execution_started_at=#{execution_started_at}" }
          already_warned = true
        end

        sleep(rand(1..5))
      end
    end

    worker = RedisLockThread.new do
      execution_started_at = Time.now
      logger.info(worker_progname) { "Beginning execution. Yielding to block." }

      begin
        yield
      ensure
        logger.debug(worker_progname) { "Marking as finished." }
        finished = true
      end

      logger.info(worker_progname) { "Finished execution." }
    end

    helper.join
    worker.join
    nil
  rescue => e
    logger.error(self) { "Encountered exception while executing with lock. #{e.class} #{e} lock=#{lock.key}" }
    raise e
  ensure
    helper.kill rescue nil
    worker.kill rescue nil
  end

  class RedisLockThread < Thread
    def initialize
      @abort_on_exception = true
      super
    end
  end
end
{% endhighlight %}
