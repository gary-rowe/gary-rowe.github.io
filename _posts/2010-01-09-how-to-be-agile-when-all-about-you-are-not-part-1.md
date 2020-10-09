---
author: Gary Rowe
title: How to be agile when all about you are not - part 1
layout: post
tags:
  - Agile
  - Design
  - Principles
redirect_from: /agilestack/2010/01/09/how-to-be-agile-when-all-about-you-are-not-part-1/
---

# How to be Agile when all around you are not – part 1

Many people have heard of the Agile development methodology but how, exactly, can you introduce it into your own workplace? This series of articles aims to provide you with a simple set of instructions that you can implement in virtually any language to make your own day to day development as productive as you can. This series is aimed at developers.

I’m assuming that you’re at the position where you’ve heard of Agile development and read some hype about extreme programmers cranking out huge systems in the blink of an eye. While this is great to enthuse people about the benefits that Agile development can bring, it does little to answer the question of how. Hence this series of articles. I aim to explain how to change your current mindset to one that embraces Agile thinking and to bring that thinking into your organisation little by little through a series of positive changes that reap instant rewards. Note that Agile is not a process, it is a mindset.

One final thing, my specialist subject is Java so I will tend to use Java based examples, but the principles that I’m working to are applicable to almost any language.

Let’s get started.

First and foremost you must work to reduce complexity. Complexity takes many forms, as anyone who has worked with any large system can testify. Often it seems that in order get anything done it is necessary to edit about 10 files, each of which is scattered about over 3 projects and involves 6 people to sign off at every stage. While this appears to introduce safety, most of the time it just makes extra work for the sake of it.

Just to be clear, I agree that in safety critical environments it’s necessary to be very careful about what changes are made to an existing codebase – nobody wants buggy code in their heart monitor after all. However, for most projects a pair of developers who are well-versed with the code should be able to decide between themselves whether or not a particular change is going to cause problems. If they’re not sure then a quick phone call to the relevant authority should be able to resolve any issue.

The quickest and easiest way of reducing complexity is to apply the DRY principle, that is, [Don’t Repeat Yourself][2]. There should be a single point of reference for any piece of data within the system, and the mechanism for retrieving it should be simple, very well defined and consistent.

This doesn’t mean use the same system for everything – that way lies madness. Rather, consider what the data is about and provide a suitable mechanism from there. So, what data am I talking about here? Everything is the quick answer, but really it’s all the creative work associated with the system, not just those system configuration parameters that I bet you were thinking about. This list includes supporting documentation, source code, schema scripts, build and deployment scripts, system parameters, system data, site content (images, videos, music et cetera) and everything else that requires a human being to create.

While this may seem daunting at first, tackling each part at a time leads naturally to an extremely efficient development process that is very responsive to changing requirements and is sufficiently robust that failures are caught very early on.

As you may have guessed, the DRY principle can be thought of as the central theme that underpins all of the other Agile practices (which I’ll introduce in later articles so as not to overwhelm you at the start). Once you start looking at all the processes that support the development of your system through the DRY lens you’ll notice a lot of inefficiency which can be pared away.

[In part 2][3] I look in detail at how to apply the DRY principle to the issue of day to day coding since for most developers it is what they have the most control over.

 [1]: https://twitter.com/share
 [2]: http://en.wikipedia.org/wiki/Don%27t_repeat_yourself
 [3]: http://gary-rowe.com/agilestack/2010/01/09/how-to-be-agile-when-all-about-you-are-not-part-2/