---
layout: post
author: Tony Schneider
title : Rails Engine Reflections
date  : 2020-05-21
tags  : software
published: false
---

This is 100% spot on with my experience using engines.

I’ve used engines a fair bit. Here are some of my observations…

As you point out, the underlying tech of a gem (in this case, plus a railtie) seems more suited for library dependencies as opposed to a way of specifying broad application level dependencies.

I can see the initial appeal with engines. They allow you to see dependencies in your code (via gems) without having to write code in a reflective way (e.g using some notion of dependency injection). An additional benefit to this, particularly in large apps, is to leverage this dependency graph to only run a subset of your tests.

I’ve seen two main kinds of engines in the wild. I’ll call them “application level” and “library level”.

The canonical example of a library level engine is devise. You can mount it in any rails app (or engine for that matter). You could even mount it in many different engines.

Worth pointing out that it’s possible to have a library level engine that doesn’t have rails models, views or controllers. In this case, it probably doesn’t need the railtie provided by engines and could just be a gem. The reason I point this out is that once you go down the path of engines, it’s possible folks (understandably) forget or don’t know there are other options!

An example of a well behaved application level engine could be cruddy admin interface. It has its own controllers, views, and maybe even models – but ultimately the dependency goes in one direction. The rails app mounts the admin interface and the admin interface has access to the contents of the rails app. I’m distinguishing it as well-behaved because ideally nothing (or in practice, effectively nothing) in the rails app proper cares about the existence of the admin engine.

An example of a potentially misbehaved application level engine might be the extraction of a subsection of your domain into an engine. For instance, most rails apps are going to have some notion of billing. The code smell I’d look for is an engine with the the absence of controllers or views – or an engine that is never mounted.

If this engine is only ever consumed by the rails app, you probably won’t feel too much pain. But with just the one consumer, what are you really getting from being in an engine? It really isn’t much more than a namespace.

IMO, where things start to go wrong is the moment an application level engine depends on another application level engine. If you’ve opted to pursue this pattern, it’s conceivable that you’ll end up with two consumers of this billing engine – probably with different requirements.

Since it’s a gem with a railtie, you’re stuck giving access to the whole engine to both consumer engines. So what do you do? Well, you probably at least consider splitting the billing engine into two engines.

I’ll concede that you can see these application dependencies as gem dependencies – but is getting that visibility also the source of the problem?

It’s hard to talk in generalities since the sheer size of the app and complexity of the domain is likely to make you feel these pains more or less.

I love the idea of formalizing the module namespace convention. It seems a number of folks have arrived at variations of the same solution. Perhaps a way to make engines better is to canonicalize the module namespace pattern so we stop using engines as a means of organizing our domain – leaving them as a means of organizing our rails application.

And if this convention is formalized, perhaps it makes it easier to build tooling to help solve the more domain specific problems like only running subsets of tests or visualizing domain dependencies.

Note: Purposefully didn’t get into when to extract something into a separate rails app. I do think it’s a relevant part of the conversation when considering how engines can be used. Personally, I would defer defer defer until it’s so blatantly obvious that it’s the right thing to do – if ever.