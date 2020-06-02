---
layout: post
author: Tony Schneider
title : OAuth, OAuth2, and Ruby
date  : 2012-02-04
tags  : software
---

Recently, some colleagues and I have been working on a client project that involves several different internal web services. Along with the usual user authentication, we needed a way to authenticate and verify API requests across these services.

Let’s start with user authentication. Enter [OAuth2](https://tools.ietf.org/html/draft-ietf-oauth-v2-23)…

### OAuth2

If you aren’t familiar with the OAuth2 protocol, I’m sure you’ve at least used it as a client. You click the “login-using-some-3rd-party”, get redirected, enter your credentials, get redirected back. Badda boom, badda bing, done - you’re authenticated. This works great for that three-legged exchange of resource owned credentials discussed at length in section 4.1 in the RFC. There are a ton of resources for this. Especially in the ruby/rails space. Some excellent ones to check out are the following:

* devise_oauth2_providable ([github](https://github.com/socialcast/devise_oauth2_providable))
* omniauth ([github](https://github.com/omniauth/omniauth))

The devise_oauth2_providable is a rails engine that makes creating a standard OAuth2 provider insanely easy. At the time of this post, it is actively maintained. In fact, when we came across a bug during development, it wasn’t more than an hour later and someone had provided a fix, pulled it in, and cut a new version.

Omniauth is a wonderful little gem that provides an excellent abstraction for any type of client authentication. It has templates that provide a simple DSL for the most popular authentication schemes.

This workflow is excellent when you have a service that wishes to authenticate using a third party (like Facebook or Google). However, since the we owned both the OAuth2 provider and the OAuth2 client, it seems pretty silly to redirect to our Authentication server, enter credentials, and redirect back our client app. Fortunately, the OAuth2 specifies a protocol for this use case as well in [Section 4.3](https://tools.ietf.org/html/draft-ietf-oauth-v2-23#section-4.3) of the RFC.

As a fan of the omniauth abstraction, I looked into the OAuth2 abstract strategy for easily defining OAuth2 client strategies. Unfortunately it is designed for use with the Authorization code grant type. To solve this, a co-worker and I made a similar omniauth abstract strategy for providing the functionality defined in section 4.3 (should be open source shortly). Having used devise_oauth2_providable for the OAuth2 provider, it just worked.

Now, for the final piece of the puzzle, authenticating API requests. Again, OAuth2 has a flow for this use case. It’s specified in the Client Credentials grant type in [section 4.4](https://tools.ietf.org/html/draft-ietf-oauth-v2-23#section-4.4). However, we noticed a problem. The OAuth2 specification had been edited a week prior. And on top of that, section 4.4 relies only on Basic Auth. I’m sure this has its use cases, but after some research there appears to be a lot of discussion on whether or not this is a sufficient solution.

We certainly wanted something more secure than Basic Auth. Enter [OAuth 1.0](https://tools.ietf.org/html/rfc5849)…

### OAuth 1.0

OAuth 1.0 solves the same problems that OAuth 2 aims to solve, but in a different (arguably more secure) way. It also supports workflows analogous to sections 4.1, 4.3, and 4.4 of OAuth2. So, given the current state of the OAuth2 RFC, if you need the functionality described in the client credentials grant type of OAuth2, and basic auth doesn’t provide enough security, you’re probably better off using OAuth 1.0.

If you’re using Ruby, *you might be tempted to use the [oauth](https://github.com/oauth/oauth-ruby) gem. I would highly recommend not doing this given the state of the gem.* The github network graph seems to imply a pretty heavily divergent state. Having found at least one bug, and a serious lack of tests and documentation, I hope you’ll learn from my mistake and look elsewhere.

Fortunately, there are some pretty amazing options that I wish I would have found earlier. Here’s a quick recap:

* simple_oauth ([github](https://github.com/laserlemon/simple_oauth)) - It is just that. It signs and verifies OAuth requests. This gem has tests, documentation, and is well maintained.
* faraday_middleware ([github](https://github.com/pengwynn/faraday_middleware)) - This is a bit of Rack middleware that works alongside the excellent Faraday gem. It comes with a couple middlewares for tampering with outgoing requests, two of which include OAuth and OAuth2.

So, if you’re integrating the oauth or oauth2 protocols into one of your services, hopefully you’ll find this bit of first hand experience useful.
Other useful/helper links:

* http://hueniverse.com/2010/09/oauth-2-0-without-signatures-is-bad-for-the-web/
* http://tools.ietf.org/html/draft-ietf-oauth-v2-http-mac-00
