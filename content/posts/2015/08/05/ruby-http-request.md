---
title: "Ruby Http Request"
date: 2015-08-05T09:33:11+08:00
tags: [Ruby]
categories: [engineering]
---

During trial on sending GET request to AppFigures’ RESTful API, I started with **net/http** and add my client key to request hash.

When firing the request, I have some more request params to use. Because of misunderstanding on how the server deal with params and request header, I added those params into request header and had wrong expectation.

Code is like below.

```ruby
uri = URI(base_url)
request = Net::HTTP::Get.new(uri)
request.basic_auth('username', 'password')
request['X-Client-Key'] = 'app_key'
if args
  args.each do |k,v|
    request[k.to_s] = v.to_s if v
  end
end
```

This is totally wrong because the params is query parameters which should be put in uri query.

So then the corrected code goes to.

```ruby
uri = URI(base_url)
uri.query = URI.encode_www_form(args) if args
```
