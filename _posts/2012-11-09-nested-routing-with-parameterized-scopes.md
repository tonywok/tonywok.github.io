---
layout: post
author: Tony
title : Nested Routing w/ a parameterized scope
date  : 2012-11-09
---

The Rails router is awesome. But every once in a while I come across something I want to do and it’s not immediately obvious. In this particular case I wanted to have a simple nested resource with a scope.

i.e

```
/foos/:foo_id/:scope/bars/:id
```

Sounds easy enough, right?

Well, here was my first attempt:

```
resources :foos do
  member do
    scope ":scope" do
      resources :bars
    end
  end
end
```

But, to my surprise this generates the following:

```
/foos/:id/:scope/bars/:id
```

Oops! That’s no good. In your controller, you’ll only get one of those :id parameters. In fact, I think it might be a bug. Hopefully if I have a few cycles I can try and get a fix in. At the very least a test exposing the observed behavior to see if this is intended.

Anyway, as a work around, I was able to do the following after digging through the source.

```
resources :foos do
  resources :bars, path: ":scope/children"
end
```

I wasn’t aware of the path option to resources, and I certainly didn’t know you could make the path parameterized. I hope to improve the docs on this a bit after work.

*NOTE:* In production, it’s probably a good idea to add a custom constraint to limit the allowed parameterized scopes - Especially if you’re using that parameter as… I don’t know… a scope.
