---
layout: post
author: Tony
title : Damnit! I forgot to change git-pair!
date  : 2012-03-12
published: false
---

I use git-pair pretty heavily. It’s really nice for assigning blame to whoever checked in crappy code :P Seriously though, it’s nice to give credit where credit is due. However, one thing I am often guilty of is forgetting to change the git-pair. Most of the time I realize what I’ve done after a few commits.

Here is an easy way to clean that up.

Warning: Only do this if your changes are still local as it IS rewriting history!

```
git rebase --interactive HEAD~3
```

### Interactive

As you probably know, rebasing interactive will allow you to mark what commits you’d like to squash/edit/etc and proceed to step you through them.

### HEAD~3

This just says how many commits I’d like to include in my interactive rebase. In my most recent case, I had three commits with the wrong git-pair.
Workflow

Once you save the git rebase file, and mark how you’d like to proceed, you will begin to step through the commits. For this case you’re going to want to mark the commits you’d like to change the author of as “e” or “edit”.

Run the following command to change the author of the commit:

```
git commit --amend --author="Your Name "
```

followed by a:

```
git rebase --continue
```

Anyway, hope that helps someone out.
