---
layout: post
title: Debug Solidity Contracts with WebStorm and Truffle
date: 2021-05-20 20:55 -0700
published: false
---

# How to debug Solidity contracts with WebStorm + Truffle

Today I am going to cover how to debug your Solidity contracts in WebStorm. To date, I have not found a solution for debugging the actual Solidity code, but using the JavaScript Application Binary Interface (ABI), we can test our Solidity code using JavaScript (therefore use a node debugger!) But, before I get ahead of myself, there may be some backstory required, readers well-versed in blockchain technology and Ethereum can skip the next paragraph.

Solidity is the language that the Ethereum Virtual Machine speaks. As it stands today. Thus, in order deploy a decentralized application on the Ethereum blockchain, you need to develop, test, and (potentially) debug your smart contracts before deploying to the main Ethereum network. WebStorm is my IDE of choice especially when building medium to large sized applications. It is a great choice for all size projects. The ability to build your application along side your smart contracts decreases the development-test lifecycle.

There is one tool to introduce before we go further: Truffle. Truffle is a full-featured Ethereum SDK/toolkit/CLI for developing smart contracts. That will be required before continue any further.
Other things that are assumed is that you have an up-to-date version of NodeJS installed. (I tested this with v14.17.0). Quick note, the following commands were run on macOS.
npm install -g truffle
Once that completes, go into your project with solidity contracts, or we can use Truffle to pull down an example project.
mkdir MetaCoin; cd MetaCoin; truffle unbox metacoin
This example project is a little barebones for our purposes. So let’s flesh it out a bit more by hooking into NPM with a package.json.
npm init -y
There should now be a new file at the root of the project named, package.json. Now, if you have WebStorm installed locally and hooked up to your Terminal path. Open the project folder with this command.
open -a webstorm .

WebStorm project to debug Solidity contracts
Now, open the package.json file and change the field "$.scripts.test" to truffle test. Your package.json should look like the following.
{
  "name": "metacoin",
  "version": "1.0.0",
  "description": "",
  "main": "truffle-config.js",
  "directories": {
    "test": "test"
  },
  "scripts": {
    "test": "truffle test"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
From there, WebStorm should automatically pick up the new run configuration. Click the small play icon to the left of the line of "test" and then click Debug.

Click the play icon on the scripts field in the package.json to automatically create a run configuration.

Breakpoint set within JavaScript to test Solidity contract.
Now, you should be able to open up the file ./test/metacoin.js and set breakpoints and debug to your hearts content! I know this is not a true debug of Solidity, I’ve yet to find a plugin for debugging/executing Solidity code directly from WebStorm. You’ll have to switch to the Remix IDE if you want to give that a go. — Allen
Allen Eubank is software engineer chronicling his journey through web, mobile and crypto.
