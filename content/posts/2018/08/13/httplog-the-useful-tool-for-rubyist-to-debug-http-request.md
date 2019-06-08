---
title: "Httplog, the Useful Tool for Rubyist to Debug Http Request"
date: 2018-08-13T15:13:58+08:00
tags: [ruby,http request,debugging]
categories: [engineering]
---

Our application introduced “Sharing to Twitter” feature and we did our own integration with Twitter API. We maintained the integration by ourselves because we did not want to introduce too many dependencies. Because of this, we knew a new tool which would be very helpful when debugging outgoing HTTP requests.

## It’s not hard to go with modern HTTP gems.

Modern HTTP gems, like faraday and typhoeus, provide developers an efficient, imaginable and readable way to organize the outgoing requests. Most of them accept an parameter to enable the verbose logging. For example,

```ruby
require 'typhoeus'
Typhoeus.get("http://google.com", verbose: true)
```

However, when we were dealing with our migration task, the issue was not about which parameter to pass for verbose logging. It was that we knew we did something wrong and we wanted to compare the low level log with another working gem to spot the mistake. But we could not control the options used by the gem so we could not get the log. Forking the code was not an option because it was too complicated then.

Fortunately, Ruby community already had the gem, [`httplog`](https://github.com/trusche/httplog) for logging the requests to the screen. So let's start to use it. Here are what we did.

```ruby
# Add it to Gemfile
gem 'httplog'
```

```sh
# Install the gem
$ bundle install
```

```ruby
# Configure HttpLog to capture the detail we want to know.

HttpLog.configure do |config|

  # Enable or disable all logging
  config.enabled = true

  # You can assign a different logger
  config.logger = Logger.new($stdout)

  # I really wouldn't change this...
  config.severity = Logger::Severity::DEBUG

  # Tweak which parts of the HTTP cycle to log...
  config.log_connect   = true
  config.log_request   = true
  config.log_headers   = true
  config.log_data      = true
  config.log_response  = false
end
```

That’s it and once we fire the request. You’ll see things like below.

```ruby
irb(main):023:0> uri = URI('http://www.google.com')
=> #<URI::HTTP http://www.google.com>
irb(main):059:0> Net::HTTP.get(uri)
D, [2018-08-13T22:33:30.030400 #19738] DEBUG -- : [httplog] Connecting: www.google.com:80
D, [2018-08-13T22:33:30.055285 #19738] DEBUG -- : [httplog] Sending: GET http://www.google.com:80/
D, [2018-08-13T22:33:30.055377 #19738] DEBUG -- : [httplog] Header: accept-encoding: gzip;q=1.0,deflate;q=0.6,identity;q=0.3
D, [2018-08-13T22:33:30.055412 #19738] DEBUG -- : [httplog] Header: accept: */*
D, [2018-08-13T22:33:30.055579 #19738] DEBUG -- : [httplog] Header: user-agent: Ruby
D, [2018-08-13T22:33:30.055605 #19738] DEBUG -- : [httplog] Header: host: www.google.com
D, [2018-08-13T22:33:30.055646 #19738] DEBUG -- : [httplog] Data:
D, [2018-08-13T22:33:30.100233 #19738] DEBUG -- : [httplog] Status: 200
D, [2018-08-13T22:33:30.100333 #19738] DEBUG -- : [httplog] Benchmark: 0.044534 seconds
D, [2018-08-13T22:33:30.100388 #19738] DEBUG -- : [httplog] Header: date: Mon, 13 Aug 2018 14:33:30 GMT
D, [2018-08-13T22:33:30.100418 #19738] DEBUG -- : [httplog] Header: expires: -1
D, [2018-08-13T22:33:30.100444 #19738] DEBUG -- : [httplog] Header: cache-control: private, max-age=0
D, [2018-08-13T22:33:30.100468 #19738] DEBUG -- : [httplog] Header: content-type: text/html; charset=ISO-8859-1
D, [2018-08-13T22:33:30.100493 #19738] DEBUG -- : [httplog] Header: p3p: CP="This is not a P3P policy! See g.co/p3phelp for more info."
D, [2018-08-13T22:33:30.100519 #19738] DEBUG -- : [httplog] Header: server: gws
D, [2018-08-13T22:33:30.100552 #19738] DEBUG -- : [httplog] Header: content-length: 4737
D, [2018-08-13T22:33:30.100581 #19738] DEBUG -- : [httplog] Header: x-xss-protection: 1; mode=block
D, [2018-08-13T22:33:30.100604 #19738] DEBUG -- : [httplog] Header: x-frame-options: SAMEORIGIN
D, [2018-08-13T22:33:30.100630 #19738] DEBUG -- : [httplog] Header: set-cookie: 1P_JAR=2018-08-13-14; expires=Wed, 12-Sep-2018 14:33:30 GMT; path=/; domain=.google.com, NID=136=NTCQz6wtmZFAJy87eM1aMSbPvW9UVRvSnQVkk5OtJyzIcAtKOmdj2ZFClqZyeURx0VbRdmSIuReGqUE8yCAuExogWjl54i5KfhX0M4LVOb6gDo6f7YJZ4iVXIo2AU9we; expires=Tue, 12-Feb-2019 14:33:30 GMT; path=/; domain=.google.com; HttpOnly
```

With [`httplog`](https://github.com/trusche/httplog) gem, we finally spot the mistake we made and resolve the issue. Case closed!

### Many thanks to the community!

### For your information
There are more configurations you can use with [`httplog`](https://github.com/trusche/httplog). Please refer to its Github repository from link below for more information.

If you’re curious about posting to twitter, here’s the gem recommended by Twitter and it’s actively maintained, [`twitter`](https://github.com/sferik/twitter) gem.

## References

* [httplog](https://github.com/trusche/httplog)
* [twitter](https://github.com/sferik/twitter)
* [typhoeus](https://github.com/typhoeus/typhoeus)
* [faraday](https://github.com/lostisland/faraday)