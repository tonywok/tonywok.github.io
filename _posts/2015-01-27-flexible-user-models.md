---
layout: post
author: Tony Schneider
title : Flexible User Models
date  : 2015-01-27
tags  : software
---

Applications are built for users.
We give them distinguishing properties, connect them to other entities, respond to their actions, and make various decisions based on any combination of those things.

In rails, it's almost guaranteed that sitting inside of `app/models/` is a class called `User`.
I'm sure you're intimately familiar with this class.
In fact, I suspect that if you were to hack together a heat-map representing your code editor, `user.rb` would probably be molten.

## The Setup

Taking this into consideration, how would you describe the responsibility of your `User` model? I've noticed a few trends while working in rails:

1. An end-user domain entity for creating meaningful data relationships
1. Authorizing whether an end-user can perform an action within the system
1. Authenticating the end-user is who they claim to be

There are plenty of posts on the various ways to achieve #1 and #2, and solutions vary from plug-and-play gems all the way to philosophical re-imaginations of how rails could work.
Uncontroversially, I'd just like to state that they each come with their pros and cons, and it... wait for it... sorta just depends on what you're doing.

That said, I have a bit of a bone to pick with #3.
In the rails community, at least from where I'm sitting, it seems we've decided that it's such a solved problem that you can just `gem install authentication-hotness`, shove it in `user.rb` and be on your way.
For example:

```ruby
class User < ActiveRecord::Base
  devise :database_authenticatable, :validatable
end
```

The practice of doing this demonstrates just how far we've come.
We've taken a set of requirements and turned it into a deceptively uninteresting problem.
We've added a one-liner that pulls in a well-tested, battle-hardened, re-usable library that has saved clients everywhere thousands upon thousands of dollars.

At this point in development, things are usually going great.
You're pumping out features left and right.
You want posts?
You got it.
Comments?
Sure.
I'm sure if you've been developing rails apps, you've probably seen code that resembles this:

```ruby
class User < ActiveRecord::Base
  devise :database_authenticable, :validatable
  
  has_many :posts, inverse_of: :user, dependent: :destroy
  has_many :comments, inverse_of: :user, dependent: :destroy
  
  def crazy_business_logic!(foo)
    dont_do_this($1) unless foo =~ /(wat)/
  end
end
```

Overall, this is a Good Thing™, but we need to make sure we don't shoot ourselves, or future developers in the foot as our application continues to grow in complexity.

## The Problem

Undoubtedly, something comes down the pipe that tests the flexibility of your beloved `User`.
For example, let's say you are asked to implement some variation of user invitations.
Sounds easy enough, right?
I've seen this handled in a few ways.

1. Create a user with either garbage or omitted credentials
2. Create a table for invited users and hydrate them into valid users upon accepting an invitation

You can pull of #1 in a couple ways.
On one hand, you can generate a random password for them and require them to create their own during sign up.
Ew.
On the other, you can leave the password blank and complicate your validations -- likely dipping into external library code.
Using either strategy, you now have a bunch of "users" in your system that may never even become "real".

At this point alternative #2 is starting to look more appealing.
Instead, you'll separate your invitation logic into it's own model.
Take a moment to unravel your SRP flag and plant it into the ground in the most epic of fashion.

A few moments pass, and you start to wonder... what happens when your invited user has complex relationships with other domain entities within the system?
Now you have to somehow serialize that information into the invited user so that it can be rehydrated and attached to the user once she/he accepts the invitation.

I don't know about you, but I can already hear the thunderous keystrokes of future developers furiously typing `git blame`.

## The Turn

Let's take a step back.
What could we have done differently to make our lives easier?

Remember day one when we installed that amazing gem that fast-tracked us to real feature development?
I'm going to make the argument that, if you're not careful, it can silently hamstring your project.
It's not the libraries fault, it's ours.
By hastily putting devise in our user model, we've subtly coupled together our user's domain with its identity. This goes unnoticed for some time, but eventually it catches up with us -- sometimes even canceling out a good portion of the time we saved up front.

So let's take a shot at righting our wrong.
We'll setup our user like before, except now we've moved all authentication responsibility to a class called `Identity`.

```ruby
class User < ActiveRecord::Base
  has_one :identity, inverse_of: :user, dependent: :destroy
  
  # relationships and validations
end
```

Now our identity class can handle all of the functionality that was previously crammed into `User`.

```ruby
class Identity < ActiveRecord::Base
  belongs_to :user, inverse_of: :identity
  
  # BYO authentication mechanism (devise, has_secure_password & friends, etc)
end
```

With this refactored model, we can simply create a `User` with any domain relationships we see fit without having to hack around any authentication related constraints.
Once that's done, we send out an email containing a link to accept the invitation.
Upon clicking, we can find our user by token, and prompt them to create an identity to house their credentials.

The benefit of modeling your users this way doesn't stop at invitations.
You now open yourself up to a handful of other simple implementations of historically hairy features.
A few that come to mind include simple authentication setup for tests, admin user impersonation, and users with multiple identities.

Happy modeling!