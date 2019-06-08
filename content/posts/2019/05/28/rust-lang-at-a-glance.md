---
title: "Rust Lang at a Glance"
date: 2019-05-28T23:47:44+08:00
tags: [Rust, Programming Language]
categories: [engineering]
---

Just a quick glance at Rust-lang, haven’t really done things with it. But I’m interested in its memory management system, so called “ownership”, and no “null” design.

[Rust](https://www.rust-lang.org/), borned in 2006, was firstly a personal project but then sponsored by Mozilla since 2009. It’s a compiled language and is known for its compilation time check.

There are two popular approaches of memory managements in modern programming languages. The first one is you, the developer, explicit allocate and deallocate them. The other is me, the runtime, do all the nasty things for you, which is known for [garbage collection (GC)](https://en.wikipedia.org/wiki/Garbage_collection_%28computer_science%29).

As I remember the old days dealing with C programs, I should always remember to free the memory allocated for the use. Not to mention that, I should always be aware of how the memory is passed around. And I met Java, which was firstly attractive to me for it’s ambitious goal, “write once, run anywhere”. That was my first time feel great about coding (and OOP was fascinating to me). Java introduced GC mechanism for the programs. I still remember some mechanisms like [Mark and Sweep](https://www.geeksforgeeks.org/mark-and-sweep-garbage-collection-algorithm/).

Great, I don’t need to worry about memory management anymore. Right? The truth is no. How come? Well, everything has its pros and, of course, its cons. The good news is you don’t need to manually do that, but as your program grows and runs, it’s pretty much easy to see the program is getting slower and slower. And then GC jumps in and says, “dude, I’m doing good for you. Stand aside and wait for me.” Great, now that my program needs to wait. Well, this is not a bad thing though, just a trade-off.

Other than the two approaches, Rust now says you don’t need to do what you do as in C/C++, nor suffer from the GC. Rust uses the concept of “ownership” of value and by leveraging the design, it can do away more than memory management. It could also checks if your code has some potential invalid data, like memory got freed or messed up, during compilation time. This is interesting, but I’m still looking into the more complex scenario than the tutorial samples.

Another interesting thing is no null value. This concepts gets more and more popular these days. For example, Swift and Kotlin both introduce optional. Ruby actually has an optional operator, ```?.```. I’m not sure if Swift and Kotlin is designed with no null. But I’m sure Ruby has ```nil``` and ```?.``` returns ```nil``` if the operand is a ```nil```.

Still looking into it and see if there’s anything to try out with it.