---
title: "Specify Rails Version When Init"
date: 2021-08-30T10:02:30+08:00
tags: [ruby,ruby on rails,cli]
categories: [engineering,programming language]
---

It's common to installed multiple versions of the same `gem`. For example, `rails` with `5.2.7` and `6.1.1`, etc.
By default, it uses the latest release.

To specify the old one, just use command like below, which specifies the expected version with underscode `_`.

```shell
$ rails _5.2.6_ new project-in-rails-5.2
```

Did not go into further if the magic `_5.2.6` parsing is provided by `gem` or `rails` itself.

