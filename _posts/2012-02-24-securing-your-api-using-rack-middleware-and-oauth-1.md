---
layout: post
author: Tony
title : Securing your API using Rack Middleware and OAuth 1.0
date  : 2012-02-24
published: false
---

One of the biggest challenges in web development is knowing when to break your monolithic application into services. If you segment too soon, you risk introducing unnecessary complexity. On the other hand, if you wait too long, your application is prone to becoming an unmaintainable ball of mud. I obviously can't give you a concrete answer to this problem. However, if you decide to break your application into services, there is one thing that remains constant; securing your services' endpoints. I've found Rack middleware to be an excellent tool for the job.

I had read quite a bit about Rack Middleware. I had seen a talk, watched a <a href="http://railscasts.com/episodes/151-rack-middleware">Railscast</a>, and often used other peoples' middlewares in my day-to-day Rails development. However, if you're anything like me, sometimes things don't fully sink in until you have an opportunity to get your hands dirty. Today we're going to do just that!

That being said, although the majority of this code was extracted from a real project, the following code and accompanying repository exists for the sole purpose of demonstrating how easy it is to get started using Rack middleware to secure your APIs endpoints.

Since I'm not a huge fan of long boring blog posts, let's make it <strike>nerdy</strike> awesome by sprinkling some Star Wars on it. Feel free to follow along with the source on <a href="https://github.com/tonywok/forcefield">Github</a>.

### Our Mission

A long time ago, in a galaxy far, far away...

After the destruction of the first Death Star, the Imperial IT department has decided to outsource some development. Apparently one of their developers forgot to properly secure their API, allowing a small group of Rebel hackers to steal the plans for the Death Star. After some "management changes", it is now up to us to prevent the Rebels from exploiting the Imperial's service oriented architecture.

But, little do the Sith know, we were using a Jedi mind trick!

To successfully pull off this ruse, we'll be ensuring that only clients who sign their requests according to OAuth 1.0 <a href="http://tools.ietf.org/html/rfc5849#section-3">RFC 5849 Section 3</a>, a protocol unknown to even the most chatty protocol droids, are allowed access. However, instead of polluting the Imperial API with logic concerning validation of requests, we'll be wrapping it all in Rack Middleware, and distributing it as a gem! 

### Rack

When talking about Rack, it's a given that someone will mention Rack's simple API. It really is a thing of beauty. There's only a couple things you need to know before you can start hacking:


1. You need to define an initialize method that accepts at least one argument, the Rack application. In other words, Rack Apps are to Rack middleware, as Jedi are to the Force.
1. Your middleware needs to respond to <code>call</code>, accepting an env (request environment). Much like a Jedi senses disturbances in the Force, a Rack middleware uses the <code>env</code> to react to, um... disturbances in the request.

Now for some setup! Woo! Excitement!

### Setup

{% highlight ruby %}
mkdir -p forcefield/lib/forcefield forcefield/spec/lib/forcefield
cd forcefield

echo "require 'forcefield/middleware" > lib/forcefield.rb

touch lib/forcefield/middleware.rb
touch lib/forcefield/request.rb
touch forcefield.gemspec

bundle init (install bundle if you haven't already)
{% endhighlight %}

Edit your Gemfile as follows:

{% highlight ruby %}
source "http://rubygems.org"
gemspec
{% endhighlight %}

Edit your gemspec to include the following dependencies:

{% highlight ruby %}
gem.add_dependency 'rack', '~> 1.4'
gem.add_dependency 'simple_oauth'
gem.add_development_dependency 'rspec', '~> 2.7'
{% endhighlight %}

"bundle" it up and we're ready to start rockin'. (Please see source for Rakefile and spec_helper boilerplate)

### Assumptions

Whoa there turbo! Before we get started, let's make a few assumptions. The Imperial's API already implemented an ImperialClient model that stores all the registered clients' consumer keys and secrets. Also, we're not checking OAuth nonces, leaving the Imperials open to <a href="http://en.wikipedia.org/wiki/Replay_attack">replay attacks</a>.

Little do they know, we've already emailed a set of Imperial credentials to the Rebels. Now we just have to make sure to help them implement the protocol by sending them helpful error messages in the response body.

### Tests!

So, let's start by failing some tests (spec/lib/forcefield/middleware_spec.rb):

{% highlight ruby %}
require 'spec_helper'

describe Forcefield::Middleware do
  let(:death_star) { lambda { |env| [200, {}, []] } }
  let(:middleware) { Forcefield::Middleware.new(death_star) }
  let(:mock_request) { Rack::MockRequest.new(middleware) }

  context "When incoming request has no Authorization header" do
    let(:resp) { mock_request.get("/") }

    it("returns a 401") { resp.status.should == 401 }

    it "notifies the client they are Unauthorized" do
      resp.body.should == "Unauthorized! You are part of the Rebel Alliance and ar Traitor!"
    end
  end
end
{% endhighlight %}

If you're unfamiliar with Rack, you may find the <code>let(:death_star)</code> line a little peculiar. Well, it's actually creating a mock Rack Application. This is because calling a Rack application results merely in an array containing a status code, HTTP response headers, and an array of strings that serve as the response body.

In combination with <code>Rack::MockRequest</code>, I've found this pattern to be incredibly useful when testing Rack Middleware.

With that said, we can now watch our tests fail, and begin to implement the Forcefield class.

### Middleware Checklist

As stated above, we need to make sure we do the following:

1. Defines <code>initialize</code>, taking a Rack application as an argument (see <code>death_star</code> in test)
1. Responds to call, passing in the request environment.

{% highlight ruby %}
# lib/forcefield/middleware.rb
module Forcefield
  class Middleare
    def initialize(app)
      @app = app
    end

    def call(env)
      if env["HTTP_AUTHORIZATION"]
        @app.call(env)
      else
        [401, {}, ["Unauthorized! You are part of the Rebel Alliance, and a Traitor!"]]
      end
    end
  end
end
{% endhighlight %}

That should be sufficient to make our tests pass, however, simply having an Authorization header isn't good enough. It has to be a valid OAuth header. When implementing a protocol, as you often are when working with Rack Middleware, it's nice to write specs against the RFC documentation. Here is an OAuth header adapted for our needs.

{% highlight ruby %}
GET /plans?type=secret&amp;tags=killswitch HTTP/1.1
  Host: api.deathstar.com
  Authorization: OAuth realm="Endor",
    oauth_consumer_key="9djdj82h48djs9d2",
    oauth_token="kkk9d7dh3k39sjv7",
    oauth_signature_method="HMAC-SHA1",
    oauth_timestamp="137131201",
    oauth_nonce="7d8f3e4a",
    oauth_signature="bYT5CMsGcbgUdFHObYMEfcx6bsw%3D"
{% endhighlight %}

Let's spec it out!

{% highlight ruby %}
# ... previous specs above ...
context "when incoming request has a Authorization header" do
  context "but is missing an OAuth Authorization scheme" do
    let(:header_with_bad_scheme) { { "HTTP_AUTHORIZATION" => "Force" } }
    let(:resp) { mock_request.get("/", header_with_bad_scheme) }

    it("returns a 401") { resp.status.should == 401 }

    it "notifies client that they sent the wrong Authorization scheme" do
      resp.body.should == "Unauthorized. Pst! You forgot to include the Auth scheme"
    end
  end

  context "but is missing an oauth_consumer_key" do
    let(:header_with_no_key) { { "HTTP_AUTHORIZATION" => "OAuth realm=\"Endor\"" } }
    let(:resp) { mock_request.get("/", header_with_no_key) }

    it("returns a 401") { resp.status.should == 401 }

    it "notifies the client that they have not included a consumer key" do
      resp.body.should == "Unauthorized. Pst! You forgot the consumer key"
    end
  end

  context "but is missing an oauth_signature" do
    let(:header_without_sig) { { "HTTP_AUTHORIZATION" => "OAuth realm=\"foo\", oauth_consumer_key=\"123\"" } }
    let(:resp) { mock_request.get("/", header_without_sig) }

    it("returns a 401") { resp.status.should == 401 }

    it "notifies the client that they have not signed the request" do
      resp.body.should == "Unauthorized. Pst! You forgot to sign the request."
    end
  end

  context "but is missing an oauth_signature_method" do
    let(:header_without_sig_method) do
      { "HTTP_AUTHORIZATION" => "OAuth realm=\"foo\", oauth_consumer_key=\"123\", oauth_signature=\"SIGNATURE\"" }
    end
    let(:resp) { mock_request.get("/", header_without_sig_method) }

    it("returns a 401") { resp.status.should == 401 }

    it "notifies the client that they haven't specified how they signed the request" do
      resp.body.should == "Unauthorized. Pst! You forgot to include the OAuth signature method."
    end
  end
end
{% endhighlight %}

We're going to need to mock out the ImperialClient model. I like to create mock classes for things like this in "spec/support".

{% highlight ruby %}
# spec/support/imperial_client.rb
class ImperialClient &lt; Struct.new(:consumer_key, :consumer_secret)
  DUMMY_KEY    = 'key'
  DUMMY_SECRET = 'shhhh'

  def self.find_by_consumer_key(key)
    if key == DUMMY_KEY
      new(key, DUMMY_SECRET)
    end
  end
end
{% endhighlight %}

### Simple OAuth - It's simple!

In order to get these tests to pass, we need to seriously parse the Authorization header. Certainly there exist some tools that can make this much easier for us. After fooling around with various OAuth libraries (looking at you oauth-ruby), I came across a pleasant abstraction - simple_oauth.

Also, this seems like an excellent place to start thinking about encapsulation and separation of concerns. We should probably have an object that handles verifying the request. Luckily, we need not look any further than Rack itself. We'll just subclass <code>Rack::Auth::AbstractRequest</code>, as it already provides a handful of useful helper methods. As I learned, be sure to give the Rack docs a once over before you end up reinventing the wheel. The Rack library provides a lot of potentially reusable, battle-tested code.

{% highlight ruby %}
# lib/forcefield/request
require 'rack/auth/abstract/request'

module Forcefield
  class Request < Rack::Auth::AbstractRequest

    # This method encapsulates the various checks we need to make against the requests
    # Authorization header before we deem it ready for verification.
    # Upon passing the checks, we yield to the block so that simple_oauth can determine
    # whether or not the request has been properly signed.
    #
    def with_valid_request
      if provided?
        if !oauth?
          [401, {}, ["Unauthorized. Pst! You forgot to include the Auth scheme"]]
        elsif params[:consumer_key].nil?
          [401, {}, ["Unauthorized. Pst! You forgot the consumer key"]]
        elsif params[:signature].nil?
          [401, {}, ["Unauthorized. Pst! You forgot to sign the request."]]
        elsif params[:signature_method].nil?
          [401, {}, ["Unauthorized. Pst! You forgot to include the OAuth signature method."]]
        else
          yield(request.env)
        end
      else
        [401, {}, ["Unauthorized. Pst! You forgot to include an Auth header."]]
      end
    end

    def verify_signature(client)
      return false unless client

      header = SimpleOAuth::Header.new(request.request_method, request.url, included_request_params, auth_header)
      header.valid?(:consumer_secret => client.consumer_secret)
    end

    def consumer_key
      params[:consumer_key]
    end

    private

    def params
      @params ||= SimpleOAuth::Header.parse(auth_header)
    end

    def oauth?
      scheme == :oauth
    end

    def auth_header
      @env[authorization_key]
    end

    # only include request params if Content-Type is set to application/x-www/form-urlencoded
    # (see http://tools.ietf.org/html/rfc5849#section-3.4.1)
    #
    def included_request_params
      request.content_type == "application/x-www-form-urlencoded" ? request.params : nil
    end
  end
end
{% endhighlight %}

### Say What?

The astute reader may have noticed that I'm calling some methods that appear to have come from nowhere. As stated above, this is because we are inheriting methods from Rack::Auth::AbstractRequest. Let's take a second to document what those methods are doing:

1. #provided? - Checks to see that the request env has included an Authorization header. Similar to what we had been doing prior to our refactor.
1. #scheme - Parses the authorization scheme from the header.
1. #params - We've actually decided to overwrite this method to better fit our needs. We'll use it to strip apart the OAuth params from the Authorization header via <code>simple_oauth</code>.

#### Refactoring our existing code

{% highlight ruby %}
require 'forcefield/request'

module Forcefield
  class Middleware

    def initialize(app)
      @app = app
    end

    def call(env)
      @request = Request.new(env)

      @request.with_valid_request do
        if client_verified?
          env["oauth_client"] = @client
          @app.call(env)
        else
          [401, {}, ["Unauthorized. You are part of the Rebel Alliance, and a Traitor!"]]
        end
      end
    end

    private

    def client_verified?
      @client = ImperialClient.find_by_consumer_key(@request.consumer_key)
      @request.verify_signature(@client)
    end

  end
end
{% endhighlight %}

In my opinion, this refactor is a much better separation of concern. Our <code>Forcefield::Middleware</code> class is now only responsible for one conditional to determine the validity of the request, delegating all the logic for handling incorrectly signed requests to our <code>@request</code> object.

{% highlight ruby %}
# ... previous test code ...
context 'client makes request with sufficient, but incorrect OAuth header' do
  let(:test_uri) { "http://api.deathstar.com" }
  let(:incorrect_secret) { "!!badsecret!!" }
  let(:bad_consumer_credentials) { { :consumer_key => ImperialClient::DUMMY_KEY, :consumer_secret => incorrect_secret } }
  let(:invalid_auth_header) { { "HTTP_AUTHORIZATION" => SimpleOAuth::Header.new(:get, test_uri, {}, bad_consumer_credentials).to_s } }
  let(:resp) { mock_request.get(test_uri, invalid_auth_header) }
  let(:client_with_good_credentials) { ImperialClient.new(ImperialClient::DUMMY_KEY, ImperialClient::DUMMY_SECRET) }

  before { ImperialClient.stub(:find_by_consumer_key).and_return(client_with_good_credentials) }

  it('returns a status of 401') { resp.status.should == 401 }

  it "notifies the client that they have failed at thwarting the Imperials" do
    resp.body.should == "Unauthorized. You are part of the Rebel Alliance and a Traitor!"
  end
end
{% endhighlight %}

We now have separated our request parsing code from our middleware's core logic. This leaves us with a lean, readable entry point for our middleware. Some additional tests, oauth specific edgecases and niceties can be found in the source on [Github](https://github.com/tonywok/forcefield)

Now that we've snuck our little gem into the Deathstar's API, the Rebels stand a chance!

Oh, and why not - May the force be with you!
