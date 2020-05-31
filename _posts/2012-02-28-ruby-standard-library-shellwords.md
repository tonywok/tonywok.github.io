---
layout: post
author: Tony
title : Ruby Standard Library - Shellwords
date  : 2012-02-28
published: false
---

It wasn’t too long ago I was working on a command line application. Luckily, ruby makes parsing the command line pretty trivial with the OptionParser. OptionParser works great if you want to specify command line arguments with flags. However, sometimes you want to accept options in a more declarative style.

The Shellwords library, also found in the standard library, may come in handy for this.

> This module manipulates strings according to the word parsing rules of the UNIX Bourne shell.

```
require "shellwords"

Shellwords.split("todo new 'Get the milk'") 
# => ["todo", "new", "Get the milk"]
```

Maybe I’m the only one, but I was not aware this existed. Sure, it may not be the most difficult thing to implement yourself. But, having not researched the word parsing rules of the UNIX Bourne shell, I’m willing to guess there are plenty of edgecases I would miss.

Yay Ruby Standard Library!
