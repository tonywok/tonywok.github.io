---
layout: post
author: Tony Schneider
title : Consolidating Static Lists
date  : 2020-06-09
tags  : software
---

I’ve always been a list maker.
I use them for games I want to play, status updates, important tasks — you name it, I have a list for it.
In fact, this blog post is an entry in my blog post ideas list!

While I clearly maintain lots of personal lists, I generally try to avoid making too many of them while programming.

Before I make my case, I want to clarify what I mean by *Static Lists*.
I’m not talking about any sort of “List” data structure or “static” keyword.
Full disclosure, I’ve made up the term for this post since I don’t yet know of a better way to to describe it.

## A Static List

I’ll start with an generic example involving some sort of workflow system involving tasks.
You can hopefully adapt this to whatever domain you find yourself in.

Let's imagine we have some sort of task representation in our database.
Each task record has a name, a status, and a role that represents the kind of user that can perform the task.

At some point a Static List emerges.
Dreaming up an example, let's say we have a list that enumerates the various names (or kinds) of tasks that make up our workflow tasks.

```ruby
class Task < ApplicationRecord
  NAMES = [
    "enter_data",
    "submit_for_review",
    "complete_review",
    "submit"
  ]
```

It's likely you’ve seen code like this and probably written it — I know I have.
Perhaps it starts as an inclusion validation or a way to populate a select box.

## A Portable, Accessible List

Our example above suffers from a couple problems.
For one, defining the static list as a lone constant in an ActiveRecord is not the most portable.
Secondly, we don’t have a way to reference individual task names.

A minor ergonomic improvement could be something like this, which I’ve seen become more common in ruby apps over the years:

```ruby
module TaskNames
  ALL = [
    ENTER_DATA = "enter_data",
    SUBMIT_FOR_REVIEW = "submit_for_review",
    COMPLETE_REVIEW = "complete_review",
    SUBMIT = "submit"
  ]
end
```

The module makes the list portable and the clever use of constants inside the array makes it possible to reference individual task names (e.g `TaskNames::ENTER_DATA`).
Great!

If your use case never gets more complex than this, I can definitely see a case for calling it a day and happily moving on.
In this example, let's say we gain some requirements that require us to indicate certain tasks must be performed by certain roles.

You might be tempted to achieve this by continuing the pattern, making _another_ list of task names:

```ruby
module TaskNames
  ALL = [
    # same as before
  ]

  SALES_REP_TASK_NAMES = [
    TASK_NAMES::ENTER_DATA,
    TASK_NAMES::SUBMIT_FOR_REVIEW
  ]

  MANAGER_TASK_NAMES = [
    TASK_NAMES::COMPLETE_REVIEW,
    TASK_NAMES::SUBMIT
  ]
end
```

While there's nothing inherently wrong with this, you may begin to run into trouble as you remove, modify and add new kinds of tasks to the system.
In other words, it can quickly become challenging to know all of the lists that need updated -- especially when the lists aren't centrally located.
This problem worsens as tasks gain more and more meta data.

## A Portable, Accessible, Unified List

An alternate way to approach this is to unify the lists and introduce a new concept.

```ruby
module Tasks
  ALL = [
    ENTER_DATA = TaskConfig.new(kind: :enter_data, role: :sales_rep),
    SUBMIT_FOR_REVIEW = TaskConfig.new(kind: :submit_for_review, role: :sales_rep),
    COMPLETE_FOR_REVIEW = TaskConfig.new(kind: :complete_review, role: :manager),
    SUBMIT = TaskConfig.new(kind: :submit, role: :manager)
  ]
 end

  class TaskConfig
    attr_reader :kind, :role

    def initialize(kind:, role:)
      @kind = kind
      @role = role
    end
    # NOTE: omitting manager?, sales_rep? inquiry methods for brevity
  end
end
```

We now have a single list of entries that contain metadata that is able to be queried on the fly!

We can now choose to expose these variant lists as constants, methods, or even put the calling code in the driver’s seat.
The important distinction is they are all derived from the same list.

```ruby
  module Tasks
    # ...stuff from before...

    # option A: Accessible as a constant
    MANAGER_TASKS = ALL.select(&:manager?)

    # option B: An inquiry method you can pass external args to
    def self.manager_tasks(other_stuff)
      ALL.select { other_stuff }
    end
  end

  # option C: Leaving it up to the calling code somewhere else in the codebase.
  Tasks::ALL.select { |config|  whatever_you_need }
```

We now have a home to house the complexity of tasks as our system gains more requirements.
If you're tempted to make another list, it's likely you're missing a piece of metada in your task configuration object.

In addition, we can use this same pattern again if there’s interesting behavior within a task itself (e.g maybe a role is only _sometimes_ required for that task?)

We may also want to consider taking a look at the registry pattern if we find ourselves needing to reflect on the kinds of tasks we support.

### Config vs Runtime

I would argue what we’ve actually done is start down the path of separating our “config” from our “runtime”.

The `Task` itself is managing the “runtime” state of whether or not the task has been completed or the specific user that it’s assigned to.

On the other hand, our `TaskConfig`, a higher order concept, owns how a particular kind of task is "configured" — almost like a template used for creating runtime instances of Tasks.

In addition, our runtime tasks have the flexibility to either "reach back" into the task config for static properties -- or decide to override the config with its own runtime value.

This distinction of "adding to the runtime" vs "adding to config" is useful terminology when discussing the implementation of existing and future requirements.

### A Step Further: Runtime Config

In highly dynamic situations, you may have reason to create our `TaskConfig`s on the fly (e.g defining a whole new workflow from a GUI).

The approach outlined above puts us in a nice spot since we can leverage the `TaskConfig` interface.
In other words, if we’re careful, we can write our code in such a way that users of `TaskConfig` don’t know whether they’re dealing with a statically configured `TaskConfig` or a `TaskConfig` that was instantiated from data in our database :sunglasses:

So, next time you catch yourself making many static lists, take a beat to consider if it's actually one list with lots of meta data.
