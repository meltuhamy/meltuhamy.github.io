---
title: TIL How a Java Debugger Works
tags:
  - debugging
  - ide
  - java
  - javascript
  - jdb
  - til
  - tools
  - websockets
id: 35
categories:
  - dev
date: 2015-02-13 10:22:26
---

I'm working with my friend on a project to implement a [web based debugger](https://github.com/jameslawson/webjdb) for Java projects. Today I learned all about the [Java Debug Interface](http://docs.oracle.com/javase/7/docs/technotes/guides/jpda/architecture.html) (JDI, which I like to pronounce as 'Jedi').

It's essentially an event-driven request/response API that allows all the features that a debugger supports - step over, step into, breakpoints, stack inspection, etc. For example, say we want to set a breakpoint on a certain line number. First, we load the target class, then create a breakpoint request, wait for the response which tells us the breakpoint event was handled successfully. Now, we can look at the stack at this point and inspect variables.

Add some websocket wizardry, and you can hook it up with a web application.

If you're interested in the details, head over to the GitHub project [https://github.com/jameslawson/webjdb](https://github.com/jameslawson/webjdb).

### Why a web based Java debugger? Aren't you reinventing the wheel?

The idea isn't to bring a fully-fledged code editor to the web. [Plenty](http://www.hongkiat.com/blog/cloud-ide-developers/) of those exist. Instead, the idea is to quickly debug an existing Java project in the browser. This simplifies the task of the web app - we don't care about writing code, we just care about a simple and purpose-built debugging experience. It's also useful for people who use Vim or Sublime instead of an IDE. Finally, at the moment this is just a proof-of-concept experiment. We'll see how it goes.