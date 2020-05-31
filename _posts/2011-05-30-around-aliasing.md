---
layout: post
author: Tony
title : Around Aliasing
date  : 2011-05-30
published: false
---

I’m making my way through Paolo Perrotta’s Metaprogramming Ruby book and I stumbled upon this nugget of Ruby meta-magic..

```ruby
class Fixnum
  alias :new_plus :+

  def +(num)
    old_plus(old_plus(1))
  end
end

1 + 1 => 3
```

At first glance this may appear to be merely a technique to show off the flexibility of Ruby, but as it turns out, it is quite useful in day-to-day programming. This technique is known as an “Around Alias” and can be seen in many popular Ruby libraries. One in particular being Rails, which uses this technique in the mind numbingly cool #alias_method_chain method. Go check out Rails source if you’d like to read up on #alias_method_chain.

Here’s an immediately useful and simple application of Around Aliasing adapted from an example given in the text:

```ruby
alias :safe_method_that_calls_api :method_that_calls_api

def method_that_calls_api
  start_time = Time.now
  result = safe_method_that_calls_api
  time_taken = Time.now - start_time

  if time_taken > 2
    puts "method_that_calls_api() took #{time_taken} seconds."
  end
  result
rescue
  puts "method_that_calls_api() failed."
  []
end
```
