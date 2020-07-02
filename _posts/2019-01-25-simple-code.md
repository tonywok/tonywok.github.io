---
layout: post
author: Tony Schneider
title : Simple Code
date  : 2019-01-25
tags  : software
---

> I want to write _simple_ code.

I've found this deceptively straight-forward phrase to have very contextual meaning.

To make meaningful progress, you have to _get_ things *done*.
However, to increase the rate at which things _get_ done, you have to make *doing* easier.

Perhaps it's not about writing simple code at all, but rather choosing the optimal level of abstraction.

When writing software, this tends to surface as the decision between writing code for your exact use case versus introducing some abstraction to better accommodate whatever the future _might_ bring (or has already up and brought!).

Are you after a simple solution or a simple interface?

## What is Optimal?

In short, it depends :tm:

If you write the simplest solution every time, you're likely to end up in a quagmire of sensible solutions that as a whole lack any semblance of a plan.
On the other hand, if the code works and doesn't change frequently, you can get onto more valuable tasks.

If you write a new abstraction every chance you get, you can quickly end up spending most of your time trying to force ideas together that don't belong.
The right abstraction should feel like finding a puzzle piece -- it slots in effortlessly.

Making the optimal decision is not a static choice.
You have to continuously manage it as constraints move in and out of view.

But what happens when folks disagree on what they consider optimal?
I've found there are usually two personalities in play, often operating with incomplete information.

In my opinion, the best tool we have at our disposal is empathy.
Try to remove your personal bias and look at the problem from opposing perspectives.

### The Pragmatic Lens

To the pragmatist, optimal code is code that doesn’t try to hide away any complexity.
They are after the simplest solution to the problem.

In isolation, this style is said to be able to be read and written in a very straightforward, low-effort manner.
Because it uses an agreed-upon set of primitives, it’s often less intimidating to new team members since it requires less knowledge of existing abstraction.

The pragmatist is motivated by the uncertainty of future events.
Abstraction based on conjecture will be unnecessary and/or insufficient, resulting in more complexity as the system grows.

The benefit rightfully touted by the seasoned pragmatist is the possibility of arriving at the same (or better) abstraction through repetition.
Also, the pragmatist keeps the option of withholding investment entirely, whilst still delivering initial value.

The cost identified by the idealist is that without abstraction, the system will have a lack of boundaries, resulting in having to keep large parts of the system on your mind when making changes.
Besides, if you continue to take debt, you further incentivize yourself to defer paying it off.

### The Idealistic Lens

To the idealist, optimal code is the remaining code that _must_ be written after having been distilled down by the abstraction put in place.
They are after a simple interface to make the problem melt away.

Ideally, the code you write starts to resemble relevant parts of your domain.
The “how” is tucked away to a trusted layer beneath the simple code.
As a result of this layering, we can now iterate on the abstraction as domain definitions change.

Rather than “simply” bolting on new functionality, we’re forced to confront how the new functionality fits into our existing understanding of the domain.
If done right, this can leave us with the tools to adapt as our domain is enriched.

The idealist is motivated by the optimism of future events.
The abstraction we come up with will eventually yield returns far exceeding the initial investment.

The cost identified by the pragmatist is the creation of the abstraction in the first place.
And while it may be obvious what it claims to do, there tends to be a fear associated with not understanding how it is being achieved.

## Decision Time

### Understand the Urgency

One way to increase your chances of making the wrong decision is by deciding under pressure.
If you need something yesterday, it's probably not the best time to introduce abstraction.

Add something that gets the job done to put your mind at ease so you can assess the situation without distraction.
If you can, do it with as few dependencies as possible to make removing it easier.

Make an effort to communicate this intentional decision and plan to reassess.

### Determine the Uncertainty

The cost of the wrong abstraction can be high.
The cost of no abstraction might appear small in isolation, but with enough time and functionality, it can cripple you.

Are you working on something that is core to your domain? -- Or is it more of an experiment?

Listen to the words used by experts.
If it's core to your domain, it's probably something that's discussed often.
If there's no matching concept in the system already, it's likely a good candidate.

There's a mental tax when folks have to map a concept to an unfit implementation of that concept.
If the wrong concept makes its way into the hive mind, it can fester into more unfit concepts that compound the mistake.
Try to eradicate mistakes early.

Try to identify the axioms of your domain.
If you get those right, the effort to support a new composition of those axioms is less of a lift.

If you do find yourself in a more experimental setting, take the time to define success criteria.
If experiments are common, can you optimize for them by isolating them alongside experiments with similar goals?

### Involve Other People

One sure-fire way to torpedo the success of a decision, regardless of merit, is to make it in a vacuum.

Know your audience.
You don't want to be the only one that understands how something works or why it works the way it does.

Bringing along other people is an effective way to stress tests your idea before investing in it.
Don't get emotionally attached to vaporware.

Group ownership and autonomy can be the difference between someone declaring that something is insufficient (spreading frustration) and that same person chipping in to make it incrementally better (fostering ownership).

## Clarifying Our Intent

Our goal shouldn't be to just "write simple code", but to write code at the _optimal_ level of abstraction.

We can use urgency, uncertainty, and other peoples' perspectives to help arrive at a reasonable decision.
As you learn more, the landscape changes -- expect to get it wrong and continuously re-evaluate.
