---
layout: post
author: Tony Schneider
title : Classic Rails Autoloading - Eigenclasses
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
Foo.new
```

If we were to reference `Foo` in `bar.rb` without the `require "foo"`, ruby would call `const_missing`.
Without any further intervention, it would raise an... uninitialized constant error a la `NameError (uninitialized constant Foo)`

Rails autoloading works by hooking into the ruby hook `const_missing` and relying on autoloading _conventions_ to figure out where to find the missing constant.

## First some Ruby

```ruby
class YourClass
  class << self
    def your_method
      "Hi there"
    end
  end
end

YourClass.your_method # => Hi there
```

The `class << self` syntax isn’t simply an alternate syntax for defining class methods.
While ultimately it results in a class method, you’re doing so by opening the class’s "eigenclass".

Sounds intimidating, but _Eigen_ just means "self" in German.
I think it's easier to reason about when referred to as a "singleton class".

In the ruby object model you have classes and instances of those classes.
A user defined class is just an instance of the class Class (whew :sweat_smile:).

If you're curious, you might wonder how ruby keeps track of class methods vs instance methods.
The answer is that each class has an anonymous singleton class.

You can actually see it by doing `YourClass.singleton_class`.

The class methods you define (regardless of how) end up as instance methods of the singleton class, as opposed to instance methods on the class you defined.

```
# should be the same list
YourClass.singleton_class.instance_methods(false)`
YourClass.singleton_methods
```

As the name implies, you cannot (thank goodness) create instances of the singleton class `YourClass.singleton_class.new #=> NOPE`.

## Back to Rails

So... back to autoloading :upside_down_face:

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

In Rails 5, here is roughly how the "classic" autoloading algorithm works: https://guides.rubyonrails.org/autoloading_and_reloading_constants_classic_mode.html#generic-procedure

So let's go to an example you might see in the wild:

```ruby
module SomeEngine
  module SomeNamespace
    class PolicyService
      class << self
        def create_policy
          RatingService.create_rate
        end
      end

      def self.create_policy2
        RatingService.create_rate
      end
    end
  end
end
```

As defined, the nesting inside `PolicyService.create_policy` is: `[#<Class:SomeEngine::SomeNamespace::PolicyService>, SomeEngine::SomeNamespace::PolicyService]`.

Same as the earlier example, the nesting for `PolicyService.create_policy2` is `[SomeEngine::SomeNamespace::PolicyService]`.
You might see where I'm headed ;)

When trying to resolve a constant, rails uses the `Module.nesting` from the scope in which the constant is missing.

If the constant is already loaded by something else, great, no issue.
However, relying on this can result in flakey tests since depending on the order of how the tests were run, it may or may not have been already loaded. Sound familiar? :smiling_imp:

In the flakey scenario, the constant has not already been loaded.
In the first method, the nesting is anonymous (the singleton class has no name).
Looking at the algorithm above, when const_missing is fired from an anonymous nesting, Rails will attempt to load the constant from the top level name space.
And you'll get a `NameError` because (let's pretend) `::RatingService` isn't a thing)

In the second example, the module nesting is `SomeEngine::SomeNamespace::RatingService`.
As a result, `PolicyService.create_policy2` works as expected, first checking for `SomeEngine::SomeNamespace::RatingService` then `SomeEngine::RatingService` and finally `::RatingService`.

Rails 6 reworks (thankfully :sweat_smile:) how autoloading is done using a library called [Zeitwerk](https://github.com/fxn/zeitwerk) and I'm really excited for it!