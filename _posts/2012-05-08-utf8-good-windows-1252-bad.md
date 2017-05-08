---
layout: post
author: Tony
title : UTF-8 Good, Windows-1252 Bad
date  : 2012-05-08
---

So I’m using [sidekiq](https://github.com/mperham/sidekiq), with [sidekiq-scheduler](https://github.com/moove-it/sidekiq-scheduler) at work in production to handle background tasks. Well, the other night around 10pm, just as I was settling down to watch the last episode of Mad Men, all hell broke lose.

After digging through the logs to find out what happened, all I could find was this

```
2012-05-07T01:59:10Z 24340 TID-fmbc0 ERROR: Manager#assign died
2012-05-07T01:59:10Z 24340 TID-fmbc0 ERROR: "\xC3" on US-ASCII
2012-05-07T01:59:10Z 24340 TID-fmbc0 ERROR: /home/web_apps/dream/shared/bundle/ruby/1.9.1/gems/json-1.6.6/lib/json/common.rb:148:in `encode'
```

None of the stack trace bubbled up to my app, so I knew I had no chance at rescuing from an exception. That left me with one thing to do, try and figure out what the heck caused this. Fortunately, I was able to lean on a few more seasoned colleagues of mine for advice.

After some discussion, we landed on our app being sent a windows-1252 encoded character, and trying to parse it as UTF-8.

So, to combat such things, we came up with this:

```
# This method exists because it has been shown that we cannot
# trust data coming from facebook
#
def clean_fb_str(str)
  ic = Iconv.new('UTF-8//IGNORE', 'UTF-8')
  if str == ic.iconv(str)
    # valid UTF-8 string should match its iconv version
    str
  else
    # UTF-8 conversion changed string, force windows-1252 into utf-8
    windows_ic = Iconv.new('UTF-8//IGNORE//TRANSLIT', 'WINDOWS-1252')
    windows_ic.iconv(str)
  end
end
```

Seems to do the job, as we haven’t yet had a reoccurrence of the issue.

*NOTE:* After deploying this on ruby-1.9.3, I’ve noticed we are getting deprecation warnings. I suppose we should be using `String#encode` instead.
