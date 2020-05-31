---
layout: post
author: Tony
title : Copying from terminal vim
date  : 2012-02-22
published: false
---

So, I have almost completely made the switch from MacVim to terminal vim. It hasn’t been easy giving up a lot of the niceties that come for free with MacVim. But, there was one thing in particular that really bothered me – copying code from vim and pasting it somewhere else.

Well, never fear. Vim’s “*” register maps to pbcopy (at least on OSX)

So, next time you want to show that kickass piece of code to your friends, just do the following after making a selection:

```
“*y
```

You now have that awesome bit of code in your clipboard! Show it off with pride!
