---
layout: post
author: Tony
title : A Framework for Framework Thinking
date  : 2021-08-27
tags  : software
---

To help you think about building a platform...

When building a platform, we aren’t building features; we are building capabilities that enable us to build features.
The way you do this is perhaps different than how you might just build the feature.

It’s a subtle shift in thinking and requires a different level of discipline.

**First, some terminology;**

Most frameworks make use of configuration.
This configuration can take many forms, but at the end of the day, drives how a particular capability works.

Configuration alone isn’t so useful.
It needs to be interpreted by a runtime.
This interpretation is what ultimately delivers the capability perceived by the end customer.

The ability to reason about these two layers is crucial to remaining flexible and avoiding common pitfalls.

**Then, some rules;**

The first rule: Unless it's explicitly what you're setting out to do, probably don’t create a crappy general purpose programming language.
If you leave this document with just that, I declare this a success.

If you feel like you’re starting to do this, stop, you are likely missing an abstraction.

We all know and use abstractions all the time.
For example, when was the last time you thought about checking the value of a transistor in your code?

The value of our platform is the capabilities it exposes.
These capabilities are exposed through the use of abstractions that we create and provide to customers.
When building a platform, the importance of good abstraction is amplified.

We desperately want to avoid introducing the wrong abstractions, or at least minimize the damage done by doing so.
Because of this, the second rule of framework thinking: consider your outs.

We shouldn’t always expose an abstraction simply because we need it.
We should consider several dimensions when asked to add a capability to our platform.

**And finally, some advice;**

_Ubiquity: Are we confident it is needed, or is this a truly novel ask?_

It’s often best to avoid adding something to the platform until we’re certain it’s something that will be useful to multiple customers.
The rule of three is a good place to start, but the point is to avoid making the mistake of exposing everything all the time at all costs.

By getting this wrong, we end up introducing a wider surface area.
A wider surface area increases maintenance, complexity, and most of all - the likelihood of backing ourselves into a corner; something we desperately need to avoid when building a platform.

_Urgency: Is this capability needed right now?_

When something is needed yesterday and especially when you’re under pressure to deliver, it’s not usually the best time to introduce a shiny new abstraction.

Instead it may be better to add your functionality to the runtime.
After you’ve had time to ponder the abstraction and have delivered the capability, you can expose the runtime functionality to the configuration.        

_Fit: Does it feel like you’re forcing it?_

This is incredibly important feedback.
There’s nothing worse than trying to fit a square peg into a round hole.
We need to be observant of this. If we’re in the business of providing abstractions to our customers, they will feel our missteps immediately.

The right abstraction should fit into place with ease.
It will also give you feedback in its ability to assist in the production of new abstractions that build on top of itself.
