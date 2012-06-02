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

It's well known that starting a project can often be the most difficult phase of the project. You have ideas and probably some preliminary design work, but actually codifying those ideas requires a different thought process and a little bit of motivation. Often, to alieviate the pain of starting, you get the typical setup steps taken care of, like generating a new Rails template, getting the database set up, etc, to give you a familiar framework in which you can start your new project. These are the steps I need to **not** take if I'm going to understand where my pain is and how to better design and build an application.

In order to fully seperate myself from the "Rails Way" and the typical Rails app development process, I've put in place a number of rules to guide my development and thought process. All of these rules can be summed up in one statement: The Rails Way is Wrong. Now don't get me wrong, I don't really believe that everything Rails does is bad, I'm simply giving myself an extreme stance in order to re-learn everything I've known about application development. In terms of development, I've realized that familiarity breeds laziness, and in order to prevent any old-way development from seeping into raidit, the following rules ensure that everything is in some way unfamiliar.

What I hope to learn from this process is how much of the Rails Way is truely good to use vs how much of the Rails Way is due to cargo-culting, and how I can change my development practices in the future to be better at what I do in the long run. So with that, the following rules are:

* Don't start with Rails
* Don't start with the domain model
* Don't mock anything I own
* No persistence at all, anywhere
* Interactors are the public API of the app

## Don't start with Rails

The first step towards understanding good application design and development is to stay away from frameworks entirely. It is an unfortunate truth that Rails development is a lot of cargo-culting these days, so in order for me to avoid this, I need to stay away from Rails entirely at the beginning.

Following Uncle Bob's talk, I've decided to give the Interactor / Entity design a try. The idea here is that your business logic is codified in Interactor objects. Interactors then interact (thus the name) with Entities to fulfill a single use case. With this I have a good top-down design start point, the Interactor, which will help me flush out my domain models. As I don't have Rails, I have to explicitly include whatever dependencies a given Interactor or test needs, thus letting me feel dependency pain as soon as possible, and as there is no back end, the tests are super fast.

Starting without Rails forces you to really think about your initial design and what the application itself actually needs. That said at the time of writing I have since added Rails to the application as I found myself unsure of which use case to implement next. There is something to be said about having the actual front-end to guide full top-down development. Starting without Rails isn't a hard requirement of a new application, but doing so helps force discipline in the initial design phase of the application, and that can have lasting benefits for the lifetime of the app.

## Don't start with the domain model

Tom Stuart recently presented a fantastic talk entitled ["Stop Using Development Mode"](http://www.youtube.com/watch?v=TQrEKwb5lR0) in which he outlined the typical Rails development process perfectly. In general, we have a tendency to start with the database changes, then work up to the model, continuing **up** the stack developing a new feature. It turns out this is a horrible way to build software as you end up writing code before it is needed, which often leads to unused code. When you work top-down, you are always building only and exactly what the application needs.

As stated in the previous section, my development then starts with the use cases, building an Interactor for each use case I lined out in my design notes. You can see all the interactors in raidit's [/app/interactors](https://github.com/jasonroelofs/raidit/tree/master/app/interactors). Once I added Rails I started at the very top, with the view (and now even higher, with a full acceptance test) to ensure that all the code I wrote was exactly what was needed.

Do note that my domain models are *not* ActiveRecord, and they never will be. They are plain Ruby objects that implement the non-business rules (basically code that can and should be shared across Interactors). I will add persistence to these domain models much later via another layer in the app. In this way, the tests for the domain models are super fast, and tests that actually hit a persistence layer will be in their own suite. You can find raidit's domain models in [/app/models](https://github.com/jasonroelofs/raidit/tree/master/app/models).

## Don't mock anything I own

Mocking is not only a difficult thing to do right but it is also a discussion that as yet has no right answer. Some live by mocking everything you own, some by only mocking code you don't own, some hate having mocks in tests at all, and some pick and choose according to the situation at hand. There are pros and cons for all of these. I have personally chosen to never mock anything that I (e.g. the application) own and I'll explain why.

I have experienced the pain of mocking absolutely everything, which I henseforth will call "overmocking". While I understand the basic idea this kind of mocking, where you are testing just the code and not the interactions of other objects, in my experience, the mocking ends up testing the implementation of the code and not the behavior. Changing the implementation of a method or an object then breaks a ton of tests, even though the behavior is exactly the same. Coupled with bad or non-existent design, overmocking makes the tests incredibly brittle, and even worse I ran into situations where the tests pass but the code was completely broken, leading me to longer trust the tests at all.

In my quest to figure out what actually works, I'm going to take the exact opposite stance for raidit: don't mock anything that I own. This has a number of benefits: the code itself is exercised instead of a mock, the tests will tell me if something isn't implemented, and test setup will tell me very quickly if I have too many dependencies for a given method or object. One potential issue to this stance is that it doesn't exactly adhere to the "fail quick and only in one place" idea of testing, in that if a method on a domain model breaks, the tests for the interactors that use that method will all fail. I don't forsee this being a big problem but if it doesn't become more painful than I'm expecting I shall make the appropriate changes.

When I say "mock the code I own", I need to make a distinction between the application itself and code that uses the application. When I added Rails to raidit, I initially treated the controller code as part of the application, and thus I didn't use any mocks in the tests for these controllers. This led to some test pain where the controller tests needed to know about how the domain models work, which felt to me as too much knowledge for the front end. Thus I have since decided that the Rails code is not the application, and I have begun mocking out application code usage in these tests. To ensure the glue is in place, I am now building and running cucumber acceptence tests that test the entire stack and ensure that my mocking is not hiding any problems.

## No persistence at all, anywhere

Obviously the app will eventually need persistance, but one of the quickest way to slow down your test suite is to talk to an external service, be it a database or a web service. You want tests to be fast, and honestly your app doesn't care at all how information is persisted. Databases are nothing but an implementation detail. The Java community has known this for over a decade, why have we regressed so far with Ruby and Rails?

So don't add persistence until you absolutely need it. Who knows, like Uncle Bob found out with FitNess, they ended up deferring the database right off the feature list entirely. Build exactly what you need (in raidit's case, a set of [InMemory Repositories](https://github.com/jasonroelofs/raidit/blob/master/app/repositories/in_memory.rb)) and no more. Your test suite and your code will thank you for abstracting out this logic.

## Interactors are the public API of the app

As I mentioned in [Part 1]({% post_url 2012-05-29-designing-a-rails-app-part-1 %}), one of the problems with Rails development is that Rails code is often written to talk to whatever object it happens to need at the time, whether it be a library, a model, a controller, or even a view. While there's nothing wrong with code autoloading, it can be a very handy tool, it has been heavily abused and used as an excuse to not care about code design. I've put this last rule in place to ensure I don't fall this same trap, and my unit tests will help me stay aware of dependencies, as they do not load Rails and thus have to explicitly require each file needed.

Interactors are the public API of raidit. Rails code will never directly use any object that isn't an Interactor, though it may use objects that are the results of Interactor interaction. The Rails code is not allowed to access any of the Repositories directly, nor should it do anything with the domain models.

The one caveat here is that I'll let Rails code use domain model instances iff they come from Interactors. The original Interactor / Entity design called for Request and Response structures to communicate with Interactors. As we are in Ruby, I don't feel this extra step is necessary. Interactors can and do return domain model instances (such as [FindUser](https://github.com/jasonroelofs/raidit/blob/master/app/interactors/find_user.rb)) if so required.

The basic design of all Interactors that I've currently settled on is the following:

* An interactor implements a single use case, no more no less
* Interactors have names that start with verbs
* You pass in current state information to the constructor, like current_user (see [SignUpToRaid](https://github.com/jasonroelofs/raidit/blob/master/app/interactors/sign_up_to_raid.rb))
* Can have any number of methods, and default to having #run if nothing else fits
* Instance methods should be given input data as regular params (try to stay away from hash arguments)

So far with these rules I've found Interactors to be extremely easy to read, implement and use.

## So Lets get Started!

Well technically I already did. These rules will help guide and teach me real and proper application development and design. I don't know how long this will take, but I will work hard at keeping the posts coming as I learn more.
