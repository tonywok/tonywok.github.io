---
layout: post
author: Tony
title : First Class Support
date  : 2020-07-09
tags  : software
---

You're bound to spend some amount of effort doing "support" once an application reaches a certain volume of usage.

Optimizing for this can be a difficult decision because the burden often starts as a series of small paper cuts.

Perhaps you run a script here and there.
Maybe you partially dedicate a team member on some sort of triage rotation...

However, once you reach a certain scale, you may find yourself spending a disproportionate amount of time doing repetitive tasks.

## A Framework for First Class Support

Having done my fair share of production support, I've noticed a few things that are almost always a good idea:

* Name the support action
* Define who can perform the action
* Express why the action needs to be performed
* Encode the circumstances in which the action can be done
* Encode how the action is performed
* Record who performed the action
* Organize your actions
* Track how frequently the action is performed
* Surface the performed action

### Name the Support Action

While seemingly obvious, arguably the most important step is to name the support action.
For instance, this might be something like "cancel subscription".

Having this name gives your team and organization a common way to refer to the support that needs to be done.

At worst, this will enable folks to discover the action and ask for it to be performed.
At best, they can perform it themselves.

### Define who can perform the action

Once you get a decent size of support requests, you'll probably find that some requests require more expertise than others.

Take the time to explicitly define who is authorized to act.

Pay attention to who is requesting these actions to be performed.
Perhaps they should be trained to perform it themselves.

### Express why the action needs to be performed

It's not enough to just "fix the glitch" and move on.

It's worth it to spend the time to articulate _why_.

Typically I'll include an enumerated "reason" to assist in classifying.
Also, I like to have a freeform "notes" section to capture any relevant context.

Make sure to include an "other" reason so you know when a new classification might be emerging.

### Encode the circumstances in which the action can be done

It's sometimes tempting to just jump right into writing the script that solves your problem.

Before doing so, try to establish the circumstances in which the action can be done.

For instance, if we find ourselves having to cancel subscriptions, we would want to make sure our code to perform the cancellation only runs when there's an eligible subscription to cancel!

### Perform the action

Well, duh!

Make the creation of new support actions as simple as possible!
We want a low barrier of entry to aid in democratizing support.

The goal is to make it easier to write a support action than to open up a production console.

### Record who performed the action

Make your support actions audit-able by default!

Now that your support action defines who can perform the action, record it!

### Organize your actions

As your application grows, so will the variety of things you need to support.

Use whatever features you have available to help organize support actions.

The goal is to make them easy to discover.
Maybe this means you place similar support actions in a shared namespace? Perhaps you add the ability to tag support actions with labels?

### Track how frequently the action is performed

Since we've named our support action and we record it having been run, it should be easy to track a number of interesting data points.

A few that come to mind include:

* How often is the support action being performed?
* Who is performing the support action?
* Who is requesting the support action be performed?
* Why are certain accounts requiring more support?

Put your metrics on a dashboard and review them periodically with your team/organization.

Answers to these questions can inform the efficacy of existing features and what features ought to be built!

### Surface the performed actions

Now that you're recording all of these manual support actions, put them in an activity feed!

This helps to ensure audibility and can help foster communication when rotating folks in and out of triage.

## Putting It Into Practice

How you choose to implement the above depends on your application and organization.

I'm a huge advocate of [The Command Pattern]({% post_url 2020-06-24-the-command-pattern %}) for this type of thing. 
It would probably end up looking something like this:

1. Record the manual support action record
1. Check to see if the person acting is authorized
1. Lookup the command that corresponds to the support action's name
1. Check to see if the circumstances are such that the command can be performed
1. Perform the command
1. Record the result of the command in the support action record

Once you have this in place, you can register commands to support action names, hopefully making adding new support actions as easy as filling out a simple form.

---

While all this metadata is great, and you could probably even use it to detect problems before your users notice them, make sure to listen to your metrics; you don't want your awesome support tool to cover up things that ought to be fixed outright!

Hopefully, this has you thinking about support as a first-class feature in itself as opposed to an afterthought!
