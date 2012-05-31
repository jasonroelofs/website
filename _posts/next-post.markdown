---
layout: post
title: Designing a Rails App Part 2
subtitle: My starting rules
published: true
tags:
  - software
  - architecture
  - design
  - raidit
---

[Designing a Rails App Part 1]({% post_url 2012-05-29-designing-a-rails-app-part-1 %})

Getting started is hard no matter what you're doing. Explicitly starting a new project in a vastly different direction than what you're used to is even harder. To give myself the best chance of not falling into the Rails hole again this time around, I've put in place a number of rules to drive my development of raidit. These rules are intentionally very strict, and I'll go through each one individually after the list. Some of these rules will come across as exceedingly strong or strange, and that's on purpose. I intent to see which of these rules last, which ones I end up breaking, and which I end up tweaking by the end of this process. My goal in all this is to cut off all of the habits I've built up over the years and to see which onces I re-develop through this strict framework, and which ones get broken entirely. Only then will I know that I've made progress.

* Don't start with Rails
* Don't start with the domain model
* Don't mock anything I own
* No persistence at all, anywhere
* Interactors are the public API of the app

Don't start with Rails
======================

What's the first thing one always does when starting a new Rails app? Why, it's "rails new [app name]"! It's so easy to just start a rails app and start coding views and models, building migrations, etc. This is the path to your typical Rails morass, and as such I need to even change how I start a project.

From Uncle Bob's talk, I'm giving the Interactor / Entity design a try, and to be sure that I didn't let Rails affect my design at all, I started the app without Rails. This then forced me to sit down and think thought through the basic needs of raidit, and to flesh out the use cases raidit will have. From there I started building a few basic interactors (objects who implement a use case, more on these later), tests, and fleshing out the domain models I would need.

The other benefit of this is that my test suite only loads up the models it needs (aka, not Rails) and is super fast.

Don't start with the domain model
=================================

Tom Stuart recently presented a fantastic talk entitled ["Stop Using Development Mode"](http://www.youtube.com/watch?v=TQrEKwb5lR0) in which he lined out the typical Rails development process better than I could. In general, we've had a tendency to start with the database changes, then work up to the model, and continue working **up** the stack developing a new feature. This is a horrible way to design software, as you are constantly writing code that isn't being used yet. It's far better to start at the top, what the user needs to do, and implement what you need **down** the stack.

My development then started with the use cases, building an Interactor for each use case I lined out in my design notes. You can see all the interactors in raidit's [/app/interactors](https://github.com/jasonroelofs/raidit/tree/master/app/interactors). I would normally start higher, but remember there is no Rails yet, so interactors are the highest point of contact for this application at this point. TDD-ing the Interactors then guided me towards what my domain models needed to support, and from that I have only the code I absolutely need.

As an aside, my domain models are *not* ActiveRecord. They are plain Ruby objects that implement the non-business rules. I will add persistence to these domain models much later. You can find raidit's domain models in [/app/models](https://github.com/jasonroelofs/raidit/tree/master/app/models).

Don't mock anything I own
=========================

Mocking is a difficult thing to do right, and above that it's a discussion that has no correct answer, but one on which many people hold very strong opinions. Some live by mocking everything you own, some by only mocking code you don't own, some hate having mocks in tests at all, and some pick and choose according to the situation at hand. There are pros and cons for all of these, though these pros and cons can be directly influenced by how well, or how badly, the code in question is designed.

I have experienced the pain of mocking everything, otherwise known as "overmocking". In this mindset, you mock every object interaction so you're unit testing just the one bit of code in one method on one object. While I understand the idea of minimizing the impact of a test, my experiences with this are such that the mocking ends up testing the implementation of the code and not the behavior. Changing the implementation then breaks a ton of tests, even though the behavior is exactly the same. Mocking has thus made the tests incredibly brittle, and even worse I ran into situations where the tests pass but the code was completely broken, and I could no longer trust the tests at all. Obviously this is a very bad situation, almost worse than having no tests at all.

In my quest to figure out what actually works, I'm going to take the exact opposite stance for raidit: don't mock anything that I own. This has a number of benefits: the code itself is exercised instead of a mock, the tests will tell me if something isn't implemented, and test setup will tell me very quickly if I have too many dependencies for a given method or object. One negative to this is for code that gets executed in many tests (like repository mapping), many tests will fail if one piece of code no longer works, breaking the "fail in one obvious position" test rule. I'm ok with breaking that rule for now.

The one caveat I'm adding to this rule is Rails controller tests. When I added Rails to raidit, I started the controller tests with the idea of no mocking. The problem I ran into was that the tests then were required to understand everything about the application, from the Interactors to the domain models and even to how model persistence is currently working. This felt dirty and was getting tedious, the Rails side of things shouldn't care, it's just the delivery mechanism for using the app. Now, with controller tests I'm mocking out the Interactor usage so that I'm only testing how Rails uses the application.

I will be adding one more level of tests to raidit: top level acceptance tests, probably with cucumber and rack::test. This suite will ensure that all use cases do properly work through the entire stack, ensuring that if controller mocks are falsly passing, the acceptance suite will still fail, letting me know something is wrong.

No persistence at all, anywhere
===============================

Obviously the app will eventually need persistance, but one of the quickest way to slow down your test suite is to talk to an external service, be it a database or a web service. You want tests to be fast, and honestly your app doesn't care at all how information is persisted. Databases are nothing but an implementation detail. The Java community has known this for over a decade, why have we regressed so far with Ruby and Rails?

So don't add persistence until you absolutely need it. Who knows, like Uncle Bob found out with FitNess, they ended up deferring the database right off the feature list entirely. Build exactly what you need (in raidit's case, a set of [InMemory Repositories](https://github.com/jasonroelofs/raidit/blob/master/app/repositories/in_memory.rb)) and no more. Your test suite and your code will thank you for abstracting out this logic.

Interactors are the public API of the app
=========================================

As I mentioned in [Part 1]({% post_url 2012-05-29-designing-a-rails-app-part-1 %}), one of the problems with Rails development is that Rails code is often written to talk to whatever object it happens to need at the time, whether it be a library, a model, a controller, or even a view. I'm not knocking Rails' autoloading capabilities, as at its core it is a very handy tool, but it's a tool that has been heavily abused. I've put this last rule in place to ensure I don't fall this same trap, and it will be one of pure discipline over a tool restriction to ensure I adhere to it at all times.

Interactors, the implementation of use cases of the app, are the public API of raidit, period. Rails code will never directly use any object that isn't an Interactor, though it may use objects that are the results of Interactor interaction. The Rails code is not allowed to access any of the Repositories directly, nor should it do anything with the domain models.

The one caveat here is that I'll let Rails code use domain model instances iff they come from Interactors. The original Interactor / Entity design called for a Request model and a Response model (plain data structures) that are used to talk with Interactors. As we are in Ruby, I don't feel this extra step is necessary. Interactors can and do return domain model instances (such as [FindUser](https://github.com/jasonroelofs/raidit/blob/master/app/interactors/find_user.rb)) if so required.

The basic design of all Interactors that I've currently settled on is the following:

* An interactor implements a single use case, no more no less
* Interactors have names that start with verbs
* You pass in current state information to the constructor, like current_user (see [SignUpToRaid](https://github.com/jasonroelofs/raidit/blob/master/app/interactors/sign_up_to_raid.rb))
* Can have any number of methods, and default to having #run if nothing else fits
* Instance methods should be given input data as regular params (try to stay away from hash arguments)

So far with these rules I've found Interactors to be extremely easy to read, implement and use.

Lets get Started!
=================

Well technically I already did, but with these rules I should have a set of strong enough walls to keep me from falling into "the Rails way" before I'm ready to admin which parts of Rails do work, and which don't.
