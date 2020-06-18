---
layout: post
author: Tony Schneider
title : Classic Rails Autoloading and Singleton Classes
date  : 2020-01-23
tags  : software
---

Rails constant autoloading is a really nifty feature.

It lets us reference constants without explicitly requiring them.
It is also what allows Rails to pick up changes to files without having to constantly restart your server.

It's one of those features that goes under appreciated.
If it's working you probably don't even know it's there -- until something goes wrong :laughing:

One gotcha I've run into a number of times is how it interacts with Ruby's `class << self` syntax.

## 10,000 Foot View

When you write Ruby code, you are likely defining lots of constants.
Module names are constants, class names are constants, and of course constants are constants!

When you run the code, Ruby expects you to have defined the constants you use.
Typically this is done by requiring the ruby file(s) that you plan to use.

```ruby
# in foo.rb
class Foo
end

# in bar.rb
require "foo"
class Bar
  def initialize
    @foo = Foo.new
  end
end
```

If we were to reference `Foo` in `bar.rb` without the `require "foo"`, because `const_get` fails, ruby calls `const_missing`.
Without any further intervention, it would raise an... uninitialized constant error a la `NameError (uninitialized constant Foo)`

The `const_missing` method serves as a hook for dynamically resolving the missing constant.

Classic Rails autoloading works by doing exactly this!
It implements `const_missing` and relies on file location conventions to figure out where to find the missing constant.

What happens when you break convention you ask?
Welp, let’s just say it might be a bit of a learning experience.

## First some Ruby

```ruby
class YourClass
  def self.hello
    "Why hello there"
  end

  class << self
    def hello2
      "Why hello there"
    end
  end
end

YourClass.hello # => Why hello there
YourClass.hello2 # => Why hello there
```

You may have come across this syntax in your ruby usage.

The `class << self` syntax isn’t simply an alternate syntax for defining class methods.
While ultimately it results in a class method, you’re doing so by opening the class’s "eigenclass".
Sounds intimidating at first, but _eigen_ just means "self" in German.

Remember that in the ruby object model you have classes and instances of those classes.
A user defined class is an instance of the class _Class_ (whew :sweat_smile:).

When inside an instance method, `self` is a way to refer to the instance _itself_.
This usage of `self` is different because we’re operating in the scope of the class instead of the instance.
As a result, in this example, `self` is actually `YourClass`.

Walking around saying the word “Eigenclass” sure does make you sound smart, but I find it's easier to reason about when referred to as a "singleton class".

### What's a Singleton Class?

If every class you define is an instance of `Class`, where do your class's class methods actually live?

One option would be to define them as instance methods on the class `Class`.
I think you'd be forgiven for thinking that given the "class" vs "instance of class" distinction discussed above.

While in a way poetic,  we wouldn't want Ruby to define our class methods as instance methods on the class `Class` because it would mean all instances of `Class` would have our class method.

```ruby
Class.new.hello #=> Why hello there
Class.new.hello2 #=> Why hello there
```

That would be... insanity.

To alleviate this, each ruby class has an anonymous singleton class that it uses to store class methods. 
In other words, your class methods are actually defining instance methods on this singleton class.
Similarly, when you call your class method, your class calls an instance method on the class’s singleton class.

You can actually see your class’s singleton class by doing `YourClass.singleton_class`!

```ruby
# should be the same list
YourClass.singleton_class.instance_methods(false)
YourClass.singleton_methods
```

As the name implies, you cannot (thank goodness) create instances of the singleton class:

```ruby
YourClass.singleton_class.new #=> NOPE
```

## So What’s the Difference?

When you use the `def self.` approach, ruby takes care of defining an instance method on the singleton class for you.

You’ll notice the `class << self` variant doesn’t use `def self.` at all.

Hopefully now you see why — because the `class << self` syntax opens up the singleton class allowing you to define instance methods on it directly.
As a result of doing this, your class now has access to those methods as class methods!

Probably the most common reason I see folks reaching for `class << self` is to take advantage of this instance method definition as a way to define private class methods without resorting to the admittedly awkward `private_class_method` method.

The other difference that’s seems less talked about is the impact to `Module.nesting` which is crucial to any autoloading implementation.
Because you’re defining methods in different scopes (`YourClass` vs `YourClass.singleton_class`), you’re going to get different answers when `Module.nesting` is called.

```ruby
class A
  class << self
    def foo
      Module.nesting
    end
  end

  def self.bar
    Module.nesting
  end

  def baz
    Module.nesting
  end
end

A.foo # => [#<Class:A>, A]
A.bar # => [A]
A.new.baz # => [A]
```

## Back to Rails

In Rails 5, here is roughly how the "classic" autoloading algorithm works (taken from the autoloading guides): 

```
if the class or module in which C is missing is Object
  let ns = ''
else
  let M = the class or module in which C is missing
 
  if M is anonymous
    let ns = ''
  else
    let ns = M.name
  end
end
 
loop do
  # Look for a regular file.
  for dir in autoload_paths
    if the file "#{dir}/#{ns.underscore}/c.rb" exists
      load/require "#{dir}/#{ns.underscore}/c.rb"
 
      if C is now defined
        return
      else
        raise LoadError
      end
    end
  end
 
  # Look for an automatic module.
  for dir in autoload_paths
    if the directory "#{dir}/#{ns.underscore}/c" exists
      if ns is an empty string
        let C = Module.new in Object and return
      else
        let C = Module.new in ns.constantize and return
      end
    end
  end
 
  if ns is empty
    # We reached the top-level without finding the constant.
    raise NameError
  else
    if C exists in any of the parent namespaces
      # Qualified constants heuristic.
      raise NameError
    else
      # Try again in the parent namespace.
      let ns = the parent namespace of ns and retry
    end
  end
end
```

The part we're going to focus on is the condition that says:

```
  if M is anonymous
    let ns = ''
```

From the section above we discovered the class's singleton class is an anonymous class.
Because of this, this condition is going to expect unloaded constants explicitly defined in our singleton class to be located in the top level namespace.

So let's go to an example you might see in the wild:

```ruby
  module SomeNamespace
    class PolicyService

      def self.create_policy
        RatingService.create
      end

      class << self
        def create_policy2
          RatingService.create
        end
      end
    end
  end
```

### Variant 1: `def self.` (class scope)

As defined, the nesting inside `PolicyService.create_policy` is: 

```ruby
# [
#   SomeNamespace::PolicyService
#   SomeNamespace
# ]
```

As a result, `PolicyService.create_policy` works as expected, first checking for `RatingService` in `SomeNamespace::PolicyService`, then `SomeNamespace` and finally at the top level via `::RatingService`.

### Variant 2: `class << self` (singleton class scope)

Subtly different, the nesting for `PolicyService.create_policy2` is:

```ruby
# [
#   #<Class:SomeNamespace::PolicyService>,
#   SomeNamespace::PolicyService,
#   SomeNamespace
# ]
```

If the constant is already loaded by something else, great, no autoloading required.

However, if the constant is missing, rails is going to look at the top level namespace.
If the constant isn't defined at the top level namespace, you will get a `NameError`.

Worse yet, if there is a constant defined at the top level namespace, it might not be the desired constant! :scream:

This can be a real pain in the neck to track down since you are unlikely to always load classes in the same order when running tests.
Similarly, in development, your classes will get reloaded to reflect the changes you've made, potentially causing them to be reloaded in a new order.

Sound familiar? :smiling_imp:

## Conclusion

I'd recommend **not** using `class << self` when you're working in a Rails autoloaded directory (e.g `app/**/*`).

If you're using private class methods so much that you feel the need to crack open the singleton class, perhaps there's an instance hiding in your code waiting to be discovered.

Hopefully next time you see a missing constant error, you’ll be able to track it down faster with this knowledge in your toolbox.

—--

Consider this post a farewell letter to our dear friend (and occasional mortal enemy), Classic Rails Autoloading.

Rails 6 reworks (thankfully :sweat_smile:) how autoloading is done using a library called [Zeitwerk](https://github.com/fxn/zeitwerk) and I'm really excited for it!
