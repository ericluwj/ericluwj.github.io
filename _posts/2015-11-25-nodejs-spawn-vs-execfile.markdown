---
layout: post
title:  "Node.js - spawn vs. execFile"
description: "For a long time, I had this big question in my mind: Should I use spawn or execFile? I am actually referring to the 2 Node.js functions require('child_process').spawn() and require('child_process').execFile()."
date:   2015-11-25
---

<p class="intro"><span class="dropcap">F</span>or a long time, I had this big question in my mind: Should I use spawn or execFile?</p>

I am actually referring to the 2 Node.js functions `require('child_process').spawn()` and `require('child_process').execFile()`.

These 2 functions apparently run in a largely similar manner, by executing a file in a child process. So seriously, what's the big difference? Which should I be using?

The **crucial** difference is this. `execFile` runs the executable until it exits or terminates, then returns a buffer for data on stdout or stderr with a maximum size of 200Kb. `spawn` can stream stdout or stderr back to the parent process once it starts running, and there is no limit to the size of data it can return.

As such, the conclusion is, always **prefer** `spawn` over `execFile`.

Use `execFile` only if you just need a simple return status from the child process executable or if the child process executable is totally independent of the parent program.

Otherwise using `execFile`, you are going to have a hard time debugging the child process executable when you run it from your parent program.

In my case, I was at first using `execFile` to spawn PhantomJS processes from my Node.js program, because there is an example on the `phantomjs2` npm page using `execFile`. However, with more extensive use of PhantomJS, I found out that I needed feedback from the PhantomJS child processes more than just the PhantomJS exit codes as there could be different reasons of failure within the PhantomJS script, and the feedback easily burst the buffer of 200Kb.

Since then, I have switched to using `spawn` and things look great.