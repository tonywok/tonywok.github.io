---
layout: post
author: Tony Schneider
title : A Peek Behind ActiveModel::Model
date  : 2020-07-08
tags  : software
---

I find myself using `ActiveModel::Model` quite a bit.

It’s a quick and easy way to supercharge your ruby objects with all kinds of functionality to make them compatible with `ActionView` (forms) and `ActionPack` (routing).

Aside from this compatibility, it makes your ruby object _feel_ a bit more like an `ActiveRecord` object.
In fact, much of ActiveModel was extracted from ActiveRecord during the Rails 3 refactors of old.

I often reach for it when making form objects using [The Command Pattern]({% post_url 2020-06-24-the-command-pattern %}).

```ruby

class CancelSubscription
  include ActiveModel::Model

  validates :subscription, :presence => true
  validates :cancellation_date, :presence => true

  attr_accessor :subscription, :cancellation_date

  def execute
    # logic for canceling a subscription
  end
end

form = CancelSubscription.new(
  subscription: subscription,
  cancellation_date: cancellation_date
)
form.valid? # => true / false
form.execute # => subscription cancelled!
```

Let’s take a closer look at `ActiveModel::Model` to see what it’s doing for us behind the scenes.

It implements the following methods:

* `initialize`
* `persisted?`

It includes these modules:

* `ActiveModel::AttributeAssignment`
* `ActiveModel::Validations`
* `ActiveModel::Conversion`

It is extended with these modules:

* `ActiveModel::Naming`
* `ActiveModel::Translation`

Each of these modules serves a purpose, but can also be used in isolation.

### ActiveModel::AttributeAssignment

This module is included for the sole purpose of making this work:

```ruby
form = CancelSubscription.new
form.assign_attributes(subscription: subscription, cancellation_date: cancellation_date)
form.subscription # => subscription
form.cancellation_date # => cancellation_date
```

The `ActiveModel::Model#initialize` method uses this functionality to implement the `ActiveRecord`-like constructor interface:

```ruby
form = CancelSubscription.new(
  subscription: subscription,
  cancellation_date: cancellation_date
)
```

It iterates over the hash of parameters passed into the constructor, assigning values using the setters you’re expected to have defined via `attr_accessor`.

If a setter hasn’t been defined, it will raise an `UnknownAttributeError`.

### ActiveModel::Validation

Validations define a rich class-level DSL for expressing what your model considers “valid”.

As a result, instances of the model gain a handful of methods allowing you to inquire about the validity (e.g `model.valid?`).

Once `model.valid?` has been called, it populates `model.errors` with error messages indicating that your model has attribute values that the classes' validators consider _invalid_.

The `model.errors` method returns an instance of `ActiveModel::Error`, which is a hash-like object for accessing attribute errors and messages.

This is extremely useful, especially when constructing models from user-provided data or data from programmer provided configuration.

```ruby
form = CancelSubscription.new
form.valid? # => false
form.errors.full_messages # => ["subscription can't be blank", "cancellation_date can't be blank"]
```

`Validations` comes with a whole sweet of built-in validations - some of which you've probably used with `ActiveRecord`.
It also provides an easy way to define ad hoc validations via `validate` as well as a full-featured `Validator` interface for more re-usable validations.

### ActiveModel::Translation

As you may have guessed, this module is the glue between your model and the Rails i18n internationalization framework.

This module exposes a class level interface for defining translations that correspond to your attributes a la: 

```ruby
CancelSubscription.human_attribute_name(:cancellation_date) 
# => Fecha de cancelación
```

Also, it allows you to define an `i18n_scope` method to control the expected i18n key path for your model's translations.

### ActiveModel::Naming

Extending this module gives you a class method called `model_name` that returns an instance of `ActiveModel::Name`.
Also, instances of your class will delegate the `model_name` to your class.

`ActiveModel::Name` implements much of what powers Rails' "convention over configuration" by taking the name of your class and hooking it up with `ActiveSupport::Inflector`.

Understanding how this module works really pulls the curtain back from the infamous "Rails' Magic".

### ActiveModel::Conversion

Continuing the "Rails Magic" misnomer, `Conversion` is another common source of confusion.

It does the job of extracting data from your model to further inform rails conventions such as:

* Finding partial paths via `to_partial_path`
* Constructing URLs via `to_param`

Worth noting that by default `to_param` will return `nil`  unless your model is `persisted?`.
However, you can certainly implement it yourself to your liking.

## Putting It All Together

Hopefully, now you see that `ActiveModel::Model` is nothing more than a series of modules meant to make your Ruby objects work more seamlessly with the Rails framework.

Once you realize what they do, it's pretty easy to customize them or omit them entirely to suit your needs.

For example, a few things that bother me about `ActiveModel::Model`:

1. I like to utilize keword args to fail fast.
1. I dislike having to call super if I need to modify the constructor.
1. I don't like that calling code has access to setters from `attr_accessor`.

```ruby
module FormModel
  extend ActiveSupport::Concern

  include ActiveModel::Validations
  include ActiveModel::Conversion

  included do
    extend ActiveModel::Naming
    extend ActiveModel::Translation
  end

  def persisted?
    false
  end
end
```

Now I can do something like this, opting out of the `AttributeAssignment` behavior I don't care for:

```ruby
class CancelSubscription
  include FormModel

  validates :subscription, :presence => true
  validates :cancellation_date, :presence => true

  attr_reader :subscription, :cancellation_date

  def initialize(subscription:, cancellation_date: Time.zone.today)
    @subscription = subscription
    @cancellation_date = cancellation_date
  end

  def execute
    # logic for canceling a subscription
  end
end
```
