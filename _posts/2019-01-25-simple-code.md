---
layout: post
author: Tony Schneider
title : Simple Code
date  : 2019-01-25
tags  : software
---

> I want to to write _simple_ code.

I've found this deceptively straight-forward phrase to have a very contextual meaning.

In order to make meaningful progress, you have to _get_ things *done*.
However, in order to increase the rate at which things _get_ done, you have to make *doing* easier.

When writing software, this tends to surface as the decision between writing code for your exact use case versus introducing some abstraction to better accommodate whatever the future _might_ bring (or has already up and brought!).

## What is Simple Code?

The one thing we all seem agree on is we want our code to be "simple".
However, why is it that we can never seem to agree on what that means?

What happens when we remove our personal bias and instead try to look at the problem from opposing perspectives?

### The Pragmatic Lens

To the pragmatist, simple code is code that doesn’t try to hide away any complexity. 

In isolation, this style is said to be able to be read and written in a very straightforward, low-effort manner.
Because it uses an agreed upon set of primitives, it’s often less intimidating to new team members since it requires less knowledge of existing abstraction.

The pragmatist is motivated by an uncertainty of future events.
Abstraction based on conjecture will be unnecessary and/or insufficient, resulting in more complexity as the system grows.

The benefit rightfully touted by the seasoned pragmatist is the possibility of arriving at the same (or better) abstraction through repetition.
In addition, the pragmatist keeps the option of withholding investment entirely, whilst still delivering initial value.

The cost identified by the idealist is that without abstraction, the system will have a lack of boundaries, resulting in having to keep large parts of the system on your mind when making changes.

### The Idealistic Lens

To the idealist, simple code is the remaining code that _must_ be written after having been distilled down by the abstraction put in place.

Ideally, the simple code becomes the _meaningful_ bits relevant to your domain.
The “how” is tucked away to a trusted layer beneath the simple code.
As a result of this layering, we can now iterate on the abstraction as domain definitions change.

Rather than “simply” bolting on new functionality, we’re forced to confront how the new functionality fits into our existing understanding of the domain.

The idealist is motivated by optimism of future events.
The abstraction we come up with will eventually yield returns far exceeding the initial investment.

The cost identified by the pragmatist is the creation of the abstraction in the first place.
And while it may be obvious what it claims to do, there tends to be a fear associated with not understanding how it is being achieved.

## Decision Time

### Understand the Urgency

One way to increase your chances of making the wrong decision is by making the decision under pressure.
If you need something yesterday, it's probably not the best time to introduce an abstraction.

Add something that gets the job done to put your mind at ease so you can assess the situation without distraction.
If you can, do it with as few dependencies as possible to make removing it easier.

Make an effort to communicate this intentional decision and plan ahead to reassess.

### Determine the Uncertainty

The cost of the wrong abstraction can be high.
The cost of no abstraction might appear small in isolation, but with enough time and functionality, it can cripple you.

Are you working on something that is core to your domain? -- Or is it more of an experiment?

Listen to the words used by experts.
If it's core to your domain, it's probably something that's discussed often.
If there's no matching concept in the system already, it's likely a good candidate.

There's a mental tax when folks have to map a concept to an unfit implementation of that concept.
If the wrong concept makes its way into the hive mind, it can fester into more unfit concepts that compound the mistake.

Instead, try to identify the axioms of your domain.
If you get those right, the effort to support a new composition of those axioms is more likely.

If it's an experiment, take the time to define success criteria.
If experiments are common, can you optimize for them by isolating them alongside experiments with similar goals?

### Involve Other People

One sure fire way to torpedo the success of a decision, regardless of merit, is to make it in a vaccuum.

Know your audience.
You don't want to be the only one that understands how something works or why it works the way it does.

Bringing along other people is an effective way to stress tests your idea before investing in it.
Don't get emotionally attached to vaporware.

Group ownership and autonomy can be the difference between someone declaring that something is insufficient (spreading frustration) and that same person chipping in to make it incrementally better (fostering ownership).

## Clarifying Our Intent

Let’s stop “writing simple code”.

Instead let’s be explicit in our decision to either write the code we need _right now_ or the code we've consciously decided accommodates our level of uncertainty.
And when we do it, let's do it as a team.

Good luck and be ready to get it wrong sometimes :)
