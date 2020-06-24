---
layout: post
author: Tony Schneider
title : The Command Pattern
date  : 2020-06-24
tags  : software
---

If you google the word “command”, you’ll eventually find a definition that’s something to the effect of:

> A command is a **directive** to a computer **program** to perform a specific **task**.

In other words, a command is a way to encapsulate behavior.

Ok, so now that we got all the super generic stuff out of the way, let’s try to get more specific than “program”, “directive” and “task”.

## The Program

No matter what you’re working on, you’re working in some sort of _domain_.
It could be anything; a game, a specific menu inside that game, an insurance company, or maybe you’re just implementing some forms on an admin dashboard.

After all, from 10,000 feet, no matter what software you’re building, it pretty much fits into the mold of a computer program accepting directives that in return perform tasks.

If you think it doesn’t, I challenge you to step outside the specifics of whatever you’re working on.
Forget about whatever language or framework you’re using and try to think about the problem you’re solving.

Before we start, let’s take a moment to establish our domain.
I’m going to choose the admin dashboard because it’s pretty broadly applicable and most software projects have at least some notion of an admin dashboard.
However, **at a high-level**, I think you could apply almost everything below to pretty much any domain you want.

So from here on out, our “program” is an “admin dashboard”.

In my experience an admin dashboard usually serves two main roles:

1. Exposing an insider look at data to aid in debugging and monitoring.
1. Exposing actions that only certain people can perform under certain conditions

Let’s focus on the second for now.

## The Directive

One example of a directive in our fictional admin dashboard might be the ability to cancel a user’s subscription.

Naming is certainly hard, but I think we have this one under control for now.
Let’s call our command `CancelSubscription`.

Regardless of the naming scheme you choose, try to follow these rules:

1. Name the command after the behavior it implements _(this usually involves a verb)_
1. Do this in such a way that behaviors in the same domain live near one another _(expect this to evolve)_

As domains get more mature, they often become more specialized.

Eventually, your admin dashboard might have tens of commands related to subscriptions.
If this is the case, maybe you go with something like `Subscriptions::Cancel` instead.
Renaming or reorganizing shouldn't be a herculean effort.

## The Task

Given the admin dashboard (program) and our desire to cancel subscriptions (directive), we need to define our _specific_ task.

In this case, our task is concerned with two questions:

1. Are the conditions such that I can cancel the subscription?
1. If able, how do I go about canceling the subscription?

I like that the definition uses the words _specific_ task.
In other words, if it doesn’t have to do with either of these two questions, do it somewhere else :)

If we do need a piece of data to answer either of these questions, we can pass it into our command so long as our command doesn’t know or care where it came from.
This will make your command more re-usable and easier to test.

For instance, our `CancelSubscription` command likely needs a subscription, a date the cancellation is to go into effect, the reason it’s being canceled, and maybe the administrator that is performing the cancellation.

### The Task: Am I Able?

Before we perform the task, we need to make sure we can perform the task.
This is where you implement your business rules.

For instance, a couple of usual suspects:

* Only administrators with certain permissions can cancel subscriptions
* The effective date must be between the subscription start date and the subscription end date
* A cancellation reason must be supplied and be one of several defined reasons

There are certainly other libraries out there to choose from for both Ruby and other languages.
I think a lot of this comes down to personal preference and willingness to learn new APIs.
As a heads up, Commands may go by different names such as Interactors, Mutations, Operations, and others I’m sure.

Whatever they do, they likely do something similar but vary in syntax/DSL and feature set (e.g type coercion, checking, etc).

When using Ruby I tend to gravitate towards `ActiveModel` (and friends) since it’s _good enough_, almost guaranteed to be present, and usually avoids any sort of holy war, letting us focus on stuff that matters (i.e canceling subscriptions!).

```ruby
class CancelSubscription
  include ActiveModel::Validations

  attr_reader :subscription,
              :administrator,
              :effective_date,
              :reason

  validates :subscription, presence: true
  validates :administrator, presence: true
  validates :effective_date, presence: true
  validates :reason, presence: true, inclusion: { in: Subscription::CancellationReasons::ALL }
  validate :authorized_administrator

  def initialize(subscription:, administrator:, effective_date: nil, reason: nil)
    @subscription = subscription
    @adminstrator = administrator
    @effective_date = effective_date
    @reason = reason
  end

  private

  def administrator_authorized
    unless can_cancel_subscription?(administrator, subscription)
      errors.add(:administrator, "does not have permission to cancel subscriptions")
    end
  end
end
```

Including `ActiveModel::Validations` defines a `valid?` that returns `true` or `false`.
If `valid?` returns `false`, it populates the `errors` on the `CancelSubscription` instance.

In the case of our admin dashboard, we’d probably want to use these errors to re-render an invalid form or construct a JSON payload.

#### Required

In the example above, I used keyword arguments to indicate that `subscription` and `administrator` are required.

Without these two things, we’re not even going to try to perform our task.
If this happens, something else must be wrong.

#### Optional

Similarly, I indicated that `effective_date` and `reason` are optional by having their values default to `nil`.

I have them as optional because they are likely set by the administrator’s selection in a form.
In this example, I defaulted them to `nil`, but in real life, there might be a more reasonable default.
Worth noting that as written, if the user doesn’t make a selection, the command will not execute due to our validations.

### The Task: How do I?

So, assuming we got past our validations, how does one cancel a subscription?

The good part is that it **_really doesn’t matter_** so much. :rainbow:
That’s the beauty of the command.

```ruby
def execute
  # Mark subscription as canceled as of some date
  # Maybe create a cancellation audit record documenting whodunnit/reason
  # Maybe send out cancellation email?
  # Maybe publish an event to an external system?
end
```

Sure, ideally it’s expertly modeled code that checks all the boxes that you subscribe to.
In reality, it’s probably less than ideal and that’s okay.

Because we used the command pattern, folks that want to cancel a subscription don’t have to _care_ exactly how a subscription is canceled — they just need to source the dependencies needed to perform the cancellation.

## Summary

### Fat Models or Fat Controllers?

How about neither? :sweat_smile:

For example, let’s imagine we’re exposing the `CancelSubscription` command as a form in our admin dashboard.

We might have a controller that looks something like this:

```ruby
module AdminDashboard
  module Subscriptions
    class CancellationsController < AdminDashboardController
      before_action :setup_subscription

      def new
        @command = Commands::CancelSubscription.new(
          subscription: @subscription,
          administrator: current_user
        )
      end

      def create
        @command = Commands::CancelSubscription.new(
          subscription: @subscription,
          administrator: current_user,
          **cancellation_form_params
        )

        if @command.valid?
          @command.execute
        else
          render :new
        end
      end
    end
  end
end
```

We let the controller deal with authentication, sessions, parameter parsing, and orchestrating the usage of our commands.

We let the database models handle things that have to do with persistence and data integrity.

Our command owns the business rules.
Because the command knows the calling context, we avoid the problem of bestowing behavior on all consumers of a database model.

The layer between our controllers and our database models decouples us from our database representation.
This frees us up to create representations that aren't 1-1 with database models (avoiding any nested attributes shenanigans).

We’re better positioned to handle new requirements because we can always make a new command variant or even compose commands with one another.

Also, we’re able to write high-value tests without making a single request/response (you should still write end-to-end tests, just maybe fewer than you otherwise might).

### Going a Step Further: Result Objects

Depending on the size and discipline within your codebase, you may want to limit the surface area exposed by your commands.

Rather than expecting folks to initialize the command and call execute on it, you might consider exposing a class-level method that does this for you under the covers and returns a result object.

While you can certainly do this in many ways, I usually make a simple object that exposes two methods: `success?` and `payload`.

Here’s a starting point that you can adapt to your own needs:

```ruby
# As a Caller of the command
#
result = CancelSubscription.run(
  subscription: subscription,
  administrator: administrator,
  effective_date: Time.zone.today,
  reason: "just cuz"
)
result.success? # true/false
result.payload # An interface to the external world

# In the command
#
def self.run(**kwargs)
  command = new(**kwargs)
  if command.valid?
    payload = command.execute
    Result.new(success: true, payload: payload)
  else
    Result.new(success: false, payload: command.errors)
  end
end
```

Usually, when doing this it’s because I’m exposing something that might be used by another team and I want to control their access to the internals.

This usually means taking extra care to ensure that both the arguments into the command and the result’s payload are POROs.

—-

So, in summary, just give the computer program the directives it needs to perform some tasks and you’ll be fine.
