---
layout: default
title: Pitfalls
description: Pitfalls in coding, os base tools
tags: pitfall gotcha coding os
---
## Process spawning code pitfall

The original source, [rachelbythebay](https://rachelbythebay.com/w/2014/08/19/fork/), seems to be gone.

fork() can fail: this is important

Ah, fork(). The way processes make more processes. Well, one of them, anyway. It seems I have another story to tell about it.

It can fail. Got that? Are you taking this seriously? You should. fork can fail. Just like malloc, it can fail. Neither of them fail often, but when they do, you can't just ignore it. You have to do something intelligent about it.

People seem to know that fork will return 0 if you're the child and some positive number if you're the parent -- that number is the child's pid. They sock this number away and then use it later.

Guess what happens when you don't test for failure? Yep, that's right, you probably treat "-1" (fork's error result) as a pid.

That's the beginning of the pain. The true pain comes later when it's time to send a signal. Maybe you want to shut down a child process.

Do you kill(pid, signal)? Maybe you do kill(pid, 9).

Do you know what happens when pid is -1? You really should. It's Important. Yes, with a capital I.

...

...

...

Here, I'll paste from the kill(2) man page on my Linux box.

    If pid equals -1, then sig is sent to every process for which the calling process has permission to send signals, except for process 1 (init), ...

See that? Killing "pid -1" is equivalent to massacring every other process you are permitted to signal. If you're root, that's probably everything. You live and init lives, but that's it. Everything else is gone gone gone.

Do you have code which manages processes? Have you ever found a machine totally dead except for the text console getty/login (which are respawned by init, naturally) and the process manager? Did you blame the oomkiller in the kernel?

It might not be the guilty party here. Go see if you killed -1.

Unix: just enough potholes and bear traps to keep an entire valley going.
