---
layout: post
author: Tony
title : Starting out with GNU Screen
tags: screen, omniboard, apprenticeship
date  : 2010-10-12
---

I started up a new blog a few weeks ago, and I have to admit the timing was about as bad as it could be. I think I made my first post a day after the new quarter started at OSU. So, needless to say, I’ve fallen a bit behind. However, now that I’ve had some time to settle into my schedule, I want to get back to it.

This quarter I’m taking my senior capstone, CSE 762: Web Services. It’s a project based class that aims to simulate what it’s like to work at a small consultancy for a real client. Also, it just so happened that I was trying to find a solid project to work on while I go through the EdgeCase Apprenticeship Program. After a few meetings defining the project and a string of back and forth emails to professors this summer, the project was approved. Thankfully a fellow osurb-er and classmate, Andrew, was also looking for a project. We’ve teamed up to work on “Omniboard”, a pair-programming friendly project management app. Omniboard aims to facilitate information sharing and keep track of developer trends. Omniboard is currently being developed using all the new hotness that is Ruby 1.9.2 and Rails 3.0.0.

The project is still in its infancy, but we’ve been making steady progress and I’d like to share a few things I’ve learned on the way (I’ll try to make this a habit).

Andrew and I have been doing almost 100% pair-programming on Omniboard. To accomplish this we’ve been using GNU Screen. Ever since a short lightning talk on screen at Ruby Hoedown, I’ve been wanting to give it a try. Luckily for me, Andrew is a unix wiz. We’ve been ssh-ing into a server and using screen to pair on the same terminal window. Although this does mean that I had to give up MacVim, I’ve been making due, hoping that it will make me a better vim user in the end.

Before this years Ruby Hoedown, I had never used screen. And although today my knowledge of it is still very limited, I’d like to share a few of the basic commands I have been using alot.

### How to start screen itself

```
$ screen
```

Well this one is a bit obvious, but necessary none the less. The screen command starts up the screen window manager.

### Some useful startup flags

```
$ screen -x
```

This provides a alternative way to get into screen. The ‘x’ flag will try and attach a running screen instance. For example, after ssh-ing into the server, if Andrew has a screen started, I’ll use screen -x to attach to his instance of screen, and vice versa

```
$ screen -d
```

This will detach from the current screen, ready to be resumed some time in the future.

```
$ screen -r
```

This will look to see if there are any existing screens that you can re-attach to. Funny story, during Ruby Hoedown I played with screen for about 10 minutes and a week or two later I ran screen -r and saw that I still had that process running, oops! Kill!

```
$ screen -S name
```

If you find yourself detaching and re-attaching often, it would behove you to give your screen a meaningful name rather than just some pid, so that it is easier to recognize. Starting screen with -S followed by a name will achieve this.

```
$ screen -s
```

Being a zsh user, I found it useful to start screen telling it to point to zsh, rather than it’s default, sh. Eventually, once I get the hang of things, I’ll create a .sreenrc file and add this to the config so that I don’t have to worry about doing this when I open screen.

```
$ screen -U
```

The capital U tells screen, “yo dawg, my terminal understands UTF-8 encoding.

### Basic window movement and creation commands

#### ctrl-a ctrl-c

Once you are involved in a screen session, you might want to create a new screen window, using this command will allow you to do just that.

#### ctrl-a ctrl-a

Now that you have two screen windows open, you need an easy way to flip between the two. This allows you to flip back and forth between the screen you are currently in and the next screen. (Note: if you have more than 3 screens 	open, you may want a better way to move around.)

#### ctrl-a ctrl-n/ctrl-p

Now that you have more than 2 screens open, you need a way to cycle between them. Use ctrl-a in combination with ctrl-n or ctrl-p to take you to the next or previous screen, respectively.

#### ctrl-a ctrl-w (or ctrl-a “)

So you’ve gone crazy and opened up a number of screens. Perhaps you’ve forgotten which screen you are on and what numbers correspond to screens you’d like to get to. This command gives you a convenient way to list what screen you are on (denoted by a *), screens are open, what programs are running in each screen. An arguably nicer set of options can be viewed by pressing (ctrl-a “).

#### ctrl-a 0-9

Now that you know what is running and in which screen, you can easily go to the desired screen by typing in ctrl-a followed by the screen’s corresponding number.

### More advanced window manipulation

#### ctrl-a S

So you want to get a bit fancy and you’d like to see two screen windows at the same time. ctrl-a S will split the current screen window into two regions.

#### ctrl-a TAB

Now that you have two screen regions split in the same terminal window, you need a way to flip back and forth between them. This command does just that.

#### ctrl-a X

When you’ve grown tired of a screen region, use this command to close the active screen region.

#### ctrl-a [

We quickly came across a big limitation to what we thought screen could do. thought that screen didn’t allow you to scroll up past the current viewing window. You can achieve this by entering “copy-mode”, taking advantage of the screens frame buffer achieving the desired scroll.

Also, while in copy mode, you may want to copy some text. While in copy mode, you can move around with the arrow keys selecting text using the space-bar. Upon highlighting the desired text, hitting enter will copy the text into a buffer.

#### ctrl-a ]

And you thought I wouldn’t tell you how to paste what was copied into the buffer... Well, I did. Anyway, this command will allow you to paste into any window being managed by screen.

#### ctrl-a K

This will kill a screen window, terminating screen once all screens have been killed.

As always, the man pages are good and you can always check the googles has more info. I hope to find a way to use screen more often in my everyday workflow. If you’re new to screen like I am, I hope you’ve learned a bit from my tips.

Perhaps if I get a few hours this weekend, I’ll try and publish my .screenrc file.
