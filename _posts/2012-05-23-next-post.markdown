---
layout: post
title: 
published: false
tags:
  - software
  - architecture
  - design
  - raidit
---

There's a funny ... well no it's not funny, in fact it's extremely frustrating ... problem with Rails development. Every rails application ends up as one big pile of shit given enough time and features. I know this because I've created big piles of code shit multiple times, and no matter how I start, I end up with an unmaintainble, difficult to test and maintain pile of shit. I'm starting to wonder if I'm going insane, you know, "keep doing the same thing and expecting different results", however it was only until recently that I found out *why* this keeps happening.

When I worked at [Bloomfire](http://www.bloomfire.com), I had the good fortune of getting to work with [Mike Moore](http://www.blowmage.com) who pointed me to a number of videos and articles on softare architecture and application design. Mike had been watching and trying to fight this pattern for years now, but it's only recently that more developers, like myself, are starting to realize just how big of a mess it is we keep making. The one video that really clicked for me was Uncle Bob Martin's [Architecture: The Lost Years](http://www.confreaks.com/videos/759-rubymidwest2011-keynote-architecture-the-lost-years) At Ruby Midwest 2011.

If you haven't watched this talk, and you write software of any kind, please watch it now. Furthering your career in software depends on understanding the principles Uncle Bob stresses in his keynote.

I spent a long time reading other books and articles, and over time I came to realize the fundemental issue with all Rails apps, and the core problem that's been driving Uncle Bob for these past years: Dependency Mismanagement. In a Rails app, every object *can* know about any other object it wishes to know. There's nothing per-say wrong with this, it solves a bunch of really annoying problems in more static languages, but in most cases, because the objects *can* know about and use all other objects, they **do**, and this is where you get your mess.

Why does your model know about the current user? Why does your model know about persistence? Why does your Controller know how to write SQL?! **Why do Rails apps become unmaintainble piles of shit?**

***Because you have failed to manage your dependencies.***

And what's the correlary to this fact?

***You've been hiding or ignoring your test pain!*** (you *are* doing TDD, right?)

Nothing, and I mean nothing, will tell you faster that your dependencies are getting out of control than tests if you're doing TDD. When setup methods get too complicated, when one object requires setting up 10 others, that's pain that needs to be refactored out. Why does this refactoring not take place?

In my personal experiences, I can only answer that question by saying that I've been writing software wrong. Ruby is a fantastic language, but the dynamicism allows you to bypass, and at times not even feel, when dependencies do get out of control. The Rails Way encourages code in the controllers, and code in the models, but ends there. It doesn't help with designing code, though I'm not saying it should, but the patterns in place through the years of Rails projects has taught people The Rails Way, and unfortunately what's now known as The Rails Way is frankly wrong.

So how do we fix this? I have an idea, and it's to follow the architectural pattern Uncle Bob describes in his talk. As I've been completely unsuccessful in finding someone who's done this publicly, I've taken it upon myself to give this architecture a try, record what works, what doesn't work, and maybe, just maybe, find the way out of this mire in which we are so firmly entrenched.

So with that, please follow my [raidit](http://github.com/jasonroelofs/raidit) project on Github. It's a MMO and gaming guild focused events calendar that I wrote initially some years ago, but have decided to start over from scratch to use as a playground for design and architecture ideas.

The first of which is Use Case Driven Development.
