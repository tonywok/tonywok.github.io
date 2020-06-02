---
layout: post
author: Tony Schneider
title : The oh-so useful script command
date  : 2012-11-23
tags  : software
---

So I wanted to do a bit of textual analysis on the results of a redis query. Naturally I thought I could pass a couple commands to redis-cli and redirect the results to a file. Well, as it turns out, redis-cli only takes one command. I needed at least two commands in order to get the relevant results. So, how can I get that file?

A coworker of mine turned me on to an often overlooked unix command typescript. Here’s how you use it:

```
$: script
... do whatever you need to ...
[ctrl-D]
```

The text results of whatever you did in between typing “script” and pressing ctl-D, should have been written to a file called “typescript” in your current directory.

Don’t you love it when there’s a command that does exactly what you want. Thanks to Mike for telling me about it!
