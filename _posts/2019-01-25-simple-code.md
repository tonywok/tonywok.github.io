---
layout: post
author: Tony Schneider
title : Simple Code
date  : 2019-01-25
published: false
---

> I just want to be able to write simple code.

This is one of those funny phrases with highly contextual meaning. In my experience this takes one of two forms — best described, I think, by looking through the lens of the age old debate between the pragmatist and the idealist.

In order to make meaningful progress, you have to _get_ things *done*. However, in order to increase the rate at which things _get_ done, you have to make *doing* easier.

When writing software, this tends to surface as the decision between writing code for your exact use case, versus introducing some abstraction to accommodate whatever the future /might/ bring (or as already up and brought).

## What is Simple Code?
The one thing we can all agree on is we want our code to be simple. However, why is it that we can never seem to agree on what that means?

### The Pragmatic Lens

To the pragmatist, simple code is code that doesn’t try to hide away any complexity. 

In isolation, this style is said to be able to be read and written in a very straightforward, low-effort manner. Because it uses an agreed upon set of primitives, it’s often less intimidating to new team members since it requires less knowledge of existing abstraction.

The pragmatist is motivated by an uncertainty of future events. Abstraction based on conjecture will be unnecessary and/or insufficient, resulting in more complexity as the system grows.

The benefit rightfully touted by the seasoned pragmatist is the possibility of arriving at the same (or better) abstraction through repetition. In addition, the pragmatist keeps the option of withholding investment entirely, whilst still delivering initial value.

The cost identified by the idealist is that without abstraction, the system will have a lack of boundaries, resulting in having to keep large parts of the system on your mind when making changes.

### The Idealistic Lens

To the idealist, simple code is the remaining code that /must/ be written after having been distilled down by the abstraction put in place.

Ideally, the simple code becomes the /meaningful/ bits relevant to your domain. The “how”  is tucked away to a trusted layer beneath the simple code. As a result of this layering, we can now iterate on the abstraction as domain definitions change. Rather than “simply” bolting on new functionality, we’re forced to 

The idealist is motivated by optimism of future events. The abstraction we come up with will eventually yield returns far exceeding the initial investment.

The abstraction provides an interface

The cost identified by the pragmatist is the creation of the abstraction in the first place. And while it may be obvious what it claims to do, there tends to be a fear associated with not understanding how it is being achieved.

## Determine the Level of Uncertainty
(Talk about when you know enough to actually determine the abstraction)

## Know Your Audience
(You can’t be the only one that understands)

## Do Your Future Self a Favor
(Things change, you will eventually be wrong, don’t get emotionally attached)

## Clarifying our intent
Let’s stop “writing simple code”. Instead let’s be explicit in our decision to either write the code we need /right now/ versus the code we need to accommodate our /uncertainty/.

---

# Scratch

 Idealistic Pragmatism

One thing worth considering is investing in a test harness around the pragmatic solution. Aside from verifying your solution actually works, this will lessen the burden of future refactoring.

I think both parties can agree that the abstraction that reveals itself is almost certainly going to be better than something based in conjecture. 

### Pragmatic Idealism

When leaning more towards the former approach, you might . Similarly, in the former, you might

Where people fall on this spectrum depends on a number of factors:

* /Do you have the awareness that there’s even a choice?/
* /Do you have any relevant past experience?/
* /What is the urgency?/
* /What is the uncertainty?/

In the case you don’t have the awareness there is a choice, the choice becomes trivial: just do the thing in whatever way you know how. While some might describe this as ignorance, it has a major upside in having removed the burden of choice.

When approaching something similar to what you’ve experienced, regardless of how that experience went, i

So with that, let’s return to our statement and look at it through our two lenses:

> I just want to write dumb code

In the eyes of the pragmatist, this usually means a few things:

1. Anyone can write it



2. 

anyone can write it (friendly to new teammates)
it’s obvious how it works (but you have to know all of its surroundings)

I want to be able to write dumb code.

anyone can write it (expresses the “core” of 
it’s obvious what it’s doing (but might not understand how it works)
make the change easy
preparatory refactoring

What are you optimizing for?
What will you be optimizing for?
