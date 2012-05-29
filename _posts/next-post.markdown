---
layout: post
title: Designing a Rails App Part 1
published: true
tags:
  - software
  - architecture
  - design
  - raidit
---

There is a rather frustrating problem with Rails development: nigh every Rails application quickly trends towards unmaintainability. These Rails apps become monolithic, bloated, difficult-to-understand and even harder to maintain over time. This isn't an attack on Rails itself, it is just a set of libraries, but this is a pervasive problem in the Rails community today. I myself have ended up with mess of a Rails app more times than I care to admit, and most of them were ones I started! Why do I and others constantly fall into this same trap and how can we stop this downward spiral?

When I worked at [Bloomfire](http://www.bloomfire.com), I had the good fortune of getting to work with [Mike Moore](http://www.blowmage.com) who pointed me to a number of videos and articles on software architecture and application design. Mike has been watching and fighting this problem for years now, but it's only recently that more developers, like myself, are starting to understand the problem and realize just how big of a mess it is we keep making. What I also didn't know until know is that thes problems were solved 20 years ago.

The one talk that really opened my eyes was Uncle Bob Martin's [Architecture: The Lost Years](http://www.confreaks.com/videos/759-rubymidwest2011-keynote-architecture-the-lost-years) at Ruby Midwest 2011. If you haven't watched this talk, and you write software of any kind, please watch it now. Everything that follows uses and builds from what's mentioned in this talk.

Through my research and through a ton of personal introspection I've come to realize the fundemental issue with so many Rails apps: *Failing to Manage Dependencies*. We as a greater development community has lost a lot of core architecture and design lessons learned in the earlier years of software development. Just because we *can* access any model from any other place in our application, doesn't mean we *should*. In the typical Rails app though, we do. We let controllers access every model in the system. We let models talk to any other model in the system. And we write tests using tools that make it so trivially easy to not care about dependencies. Who cares of one single factory actually creates 30 different models underneath? It's Ruby, it just works! Until it doesn't, and for me the point of realizing that it no longer works is far past the point of easy refactoring. Now we're in pain-ville, again.

Are your test slow, and by slow I mean does it take longer than a few seconds to run your entire unit test suite? Can you get instant confirmation that your current test passes or fails? **Do your unit tests hit your persistence layer at all? Do the tests *require* the persistence layer?**

This is what the Agile and TDD folks have been calling Test Pain for years. Yet we keep ignoring it, treating tests as something we'll deal with later. What's the major truth of tests? **If you don't test now, you never will**. This is just as true about keeping your test suite maintainable. More often than not, if your test suite is intensely painful, you abandon the suite instead of fixing it because fixing would require redesigning the entire application... which is what we should have been doing in the first place.

How did we miss this for so long, and why do we continue to build our applications the same way? Do we even know we are making mistakes? I have to admit that before Mike, I didn't. I only knew that something wasn't quite right with whatever app I was working on, and I would do better about it next time. Yeah that has yet to happen.

So it's time to fix this, or at least give myself a chance to apply lessons learned from previous devs, to actually change how I do Rails development in general, and see if I can find a general path that lets me keep my code under control in the long term. To do this, I've taken [raidit](http://github.com/jasonroelofs/raidit), a calendar to facilitate my WoW guild's raid scheduling, and have started it over from scratch. I'll be using this project to test, prod, and experiment with different design techniques, learning how to feal test pain, and figuring out just how in the world you go about designing software.

I have no idea how long this will take, but I shall be documenting my progress as much as possible (slight apology, I've been doing this for a month now and have just got the first post made, I'll be better about this in the future).

Please feel free to comment here, comment on the raidit repo, or get a hold of me personally. This kind of learning is not something one does in isolation; I hope that my experiences here help many fellow developers learn and vastly improve their skills. We're all in this together, lets do something about it!

My next post will focus on how I got started, Use-Case driven development, and the rules I've put in place to control and guide my development and design.

