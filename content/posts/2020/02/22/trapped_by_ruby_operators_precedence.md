---
title: "Trapped by Ruby Operators Precedence"
date: 2020-02-22T21:50:00+08:00
tags: [ruby,troubleshooting,operators precedence]
categories: [engineering]
---

## Let the story begin

I started to work on a new production code repository, and I could tell my productivity degraded for almost 7 workdays. But this was not because of my lack of domain knowledge in the new production   code nor the new development environment and configuration. Even though I had some issues and was unfamiliar, I still deliver several items.

It was the inability to do development and run test all in my most familiar IDE. To workaround this, I had to go back and forth between my IDE and terminal to run my test command.

## Observations and the fix

I got two from the investigation. The first one obvious => the error. It was an unexpected outgoing network request sent when running test with my IDE. (We don't want test has dependencies on network.) The other was the difference commands for running tests. In terminal, I used `bundle exec rails test <test_file>` and my IDE uses `ruby -Itest <test_file>`.

So simple thing first, could I reproduce it with the same command from my IDE? And the answer was yes. Great, I got consistency in both environments which may lead to the differences in the flow of the commands. But let me go into the error one which might be a more possible human error.

Guess what. It's caused by a network library initialization wrapped by a guard condition. Like below.

```ruby
if defined? NET_TOOL && !Rails.env.test? # <= The guard does not work
  # DO the initialization
end
```

Changing it to the code below fixed the problem!

```ruby
if !Rails.env.test? && defined?(NET_TOOL)
  # DO the initialization
end
```

### Boom!! 

## What happened?

This is something about Ruby's operator precedence. You can refer the post [here](https://ruby-doc.org/core-2.7.0/doc/syntax/precedence_rdoc.html).

The original guard condition, `defined? NET_TOOL && !Rails.env.test?`.
`&&` operator has higher order than `defined?` so the expression becomes `defined?(NET_TOOL && !Rails.env.test?)`.

Further, namespace, `NET_TOOL`, exists and `!Rails.env.test?` is `false` in testing, the code will be recognized as `defined?(false)` and it will return a `"expression"` string.

In Ruby, the if-statement cares about falsy and truthy, not the boolean value, `true` and `false`.
Truthy means values that are not `nil` or `false`. Therefore, our original if-statement always checks on truthy value so it always invokes the initialization.

Pretty interesting, huh?

## Lessons to share

I belive that the entire effort would never be reduced, it's just moved to somewhere else. Like my story, Ruby allows us to ignore parenthisis but that also makes statement evaluation implicit. To make it work as what we want, we have to be aware of it.

Maybe there are some better pattern to live with no-parenthises style. If you know, please let me know. I really want to learn that. But before that, I would suggest using parenthesis to reduce the chance that the code runs differently from we think.

But don't get me wrong. I love coding in Ruby. :D

### Another mysterious thing to dive into but not this time.

Remember the two different commands? That would be another story about what `rails` does and how it makes different from `ruby`.
