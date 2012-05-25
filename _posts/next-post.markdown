---
layout: post
title: Designing Software
published: true
tags:
  - software
  - architecture
  - design
  - raidit
---

There's a funny ... well no it's not funny, in fact it's extremely frustrating ... problem with Rails development. Nigh every Rails application ends up as one big pile of shit given enough time and features. I know this because I've tried multiple times to not repeat the same mistake as the last Rails app ... and constantly fail. I'm starting to wonder if I'm going insane, you know, "doing the same thing and expecting different results", however it was only until recently that I found out *why* this keeps happening: I'm not actually thinking about the design of my code.

When I worked at [Bloomfire](http://www.bloomfire.com), I had the good fortune of getting to work with [Mike Moore](http://www.blowmage.com) who pointed me to a number of videos and articles on software architecture and application design. Mike has been watching and fighting this problem for years now, but it's only recently that more developers, like myself, are starting to understand the problem and realize just how big of a mess it is we keep making. What's particularly sad for me is that these problems were solved 20 years ago.

The one talk that really opened my eyes was Uncle Bob Martin's [Architecture: The Lost Years](http://www.confreaks.com/videos/759-rubymidwest2011-keynote-architecture-the-lost-years) at Ruby Midwest 2011. If you haven't watched this talk, and you write software of any kind, please watch it now. Furthering your career in software depends on understanding the principles Uncle Bob stresses here.

Through my research and through a ton of personal introspection I've come to realize the fundemental issue with all Rails apps, and the core problem that's been driving Uncle Bob for these past years: Failing to Manage Dependencies. We as a greater development community has lost a lot of lessons learned in the earlier years of software development. Just because we *can* access any model from any other place in our application, doesn't mean we *should*. In the typical Rails app though, we do. We let controllers access every model in the system. We let models talk to any other model in the system. And we write tests using tools that make it so trivially easy to not care about dependencies. Who cares of one single factory actually creates 30 different models underneath? It's Ruby, it just works! Until it doesn't, and once you realize that it's no longer working, you're already deep in shit.

Are your test slow, and by slow I mean does it take longer than a few seconds to run your entire unit test suite? Can you get instant confirmation that your current test passes or fails? **Do your unit tests hit your persistence layer at all? Do the tests *require* the persistence layer?**

This is what the Agile and TDD folks have been calling Test Pain for years. Yet we keep ignoring it, treating tests as something we'll deal with later. What's the major truth of tests? **If you don't test now, you never will**. This is just as true about keeping your test suite maintainable. More often than not, if your test suite is intensely painful, you abandon the suite instead of fixing it because fixing would require redesigning the entire application... which is what you should have been doing all along, right?

Seriously, how did we miss this for so long, and why do we continue to build our applications the same way, without learning from our mistakes? Even worse, do we ever look back and realize *what* our mistakes are? I have to admit that before Mike, I didn't even realize I was making such grave mistakes, only that something wasn't quite right, and I just told myself I'll do better next time.

Well now that I know why I've got so much code pain and it's time to figure out how to finally solve and prevent this pain from happening again. To do this, I've taken [raidit](http://github.com/jasonroelofs/raidit), a calendar to facilitate my WoW guild's raid scheduling, and have started it over from scratch. I'll be using this project to test, prod, and experiment with different design techniques, learning how to feal test pain, and figuring out just how in the world you go about designing software.

I have no idea how long this will take, but I shall be documenting my progress as much as possible (slight apology, I've been doing this for a month now and have just got the first post made, I'll be better about this in the future).

Please feel free to comment here, comment on the raidit repo, or get a hold of me personally. This kind of learning is not something one does in isolation; I hope that my experiences here help many fellow developers learn and vastly improve their skills. We're all in this together, lets do something about it!

Next: If the first thing you do is **rails new [appname]** you've already lost
