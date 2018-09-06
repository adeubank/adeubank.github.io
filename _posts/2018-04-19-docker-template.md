---
layout: post
title:  "Free Docker Template For Your Project - emptybox"
date:   2018-04-19 16:36:00 -0700
categories: programming
tags: docker templates
published: true
---
Wanted to come back to writing again so I though what better way to do that than to show you what I have been up to. I'm going to breakdown how I use [Docker](https://www.docker.com/). For those of you new to Docker, I would recommend starting with a separate tutorial to become more familiar with the commands as there is a a lot going on behind the scenes when running these commands. For a video tutorial, I would recommend following (Docker in Development)[https://serversforhackers.com/s/docker-in-development]

## The Dockerfile

Contains the instructions for building your docker image. If you have ever provisioned your own servers, you know more or less which commands to run and in which order to set it up for your application. The Dockerfile describes just that, the steps to build your application with `docker build` so that later on you can use `docker run` to actually run your application.

### Isn't Docker Overkill?

The trouble for me coming into Docker was that most of the commands were so verbose. It was tedious to setup any meaningful environments to run on Docker. Once I started building this template for developing locally to deploying to production, everything became less tedious to now being push-button deployments.

**Introducing, `emptybox`**. The `emptybox` repository contains all of the files to bring Docker into your project more or less easily. Provided you know the commands you need to provision your server if you need anything custom.







{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
