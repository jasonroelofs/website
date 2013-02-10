---
layout: post
title: "RaidIt: Final Thoughts"
published: true
series: Designing a Rails App
tags:
- software
- rails
- raidit
---

Back in May of 2012 I [started an experiment]({% post_url 2012-05-29-designing-a-rails-app-part-1 %}) to help me learn how to prevent Rails applications from becoming unmaintainable messes and specifically how to develop a Rails application using better OO design, separation of responsibilities, and abstraction of implementation. To ensure I gave myself an environment of learning and didn't fall into old habits through this process, I gave myself a set of [rules for development]({% post_url 2012-06-05-rules-for-rails-app-development %}). These rules were intentionally set to be the exact opposite of the typical Rails-Way development practices. The rules were as follows:

* Don't start with Rails
* Don't start with ActiveRecord
* Don't mock anything I own
* No persistence at all, anywhere
* Interactors are the public API of the app

As of today my experiment is officially over -- [raidit](http://github.com/jasonroelofs/raidit) is for all intents and purposes complete -- and now it's time to go over each of the rules I set, what I learned, and what I recommend going forward.

## Start with Rails

I broke the first rule [very early on](https://github.com/jasonroelofs/raidit/commit/f7951c587fe4b85a2558a1dae45a79666062dad6) when I realized I was building functionality before I needed it. I was building interactors according to the design I had written out in my notebook but was quickly getting stuck deciding what part to implement next. Having long preferred to start as high up as possible when developing a feature and working down the stack, not having Rails was forcing me to start in the middle of the stack. I felt this was a hinderance and not a benefit, so I installed Rails and got to work on the view layer. I then had the front-end to guide me into which feature needed to be built next.

When you get down to it, Rails does not encourage bad code. There seems to be a case of tunnel vision, where developers get stuck in the mindset that "if Rails doesn't give it to me it doesn't exist." We need to remember that Rails isn't in control, we are. Rails is just the framework, we need to always strive to keep our code managed and organized. To help keep myself out of this rut, I've been making sure to:

* Keep controller actions short.
* Keep models short.
* Don't be afraid to make new kinds of objects, and **put them in app/[object-type]**. E.g. put decorators in *app/decorators* and update *config.autoload_paths* appropriately.
* Only put in *lib/* what can be pulled into a gem. We need to stop treating *lib/* as a catch-all dumping ground.
* If it's a domain model, put it in *app/models*. The class doesn't have to be ActiveRecord::Base to live there.

Well delineated and organized source code goes a long way towards keeping code clean, readable, and understandable. Outside of this, Rails gives far too much to just ignore it, so start with a base Rails app and just remember that you aren't building a Rails app, you're using Rails to build your app.

## Start with ActiveModel

You can read a more detailed account of my adventures in persistence abstraction in the [previous post]({% post_url 2013-01-28-implementing-persistence %}) but if you're using Rails and you don't want to dive into full database setups yet, use ActiveModel. Rails and ActiveModel are best buds, letting you send your own non-ActiveRecord models around inside of Rails helpers and url methods with reckless abandon. I've never won a fight against Rails, and ActiveModel now exists to help people stay in Rails while not using all of Rails.

## Mock? Don't mock? Testing is hard

Mocks and bogus objects are a great way to ensure that tests are isolating the code in question. One of the biggest pain points of most Rails applications is testing and the use, misuse, or non-use of test isolation. In many cases, every test in the suite tests the entire stack, requiring data in the database and calling code that ends up modifying the database. This is the main cause of slow test suites, and fixing this on established applications is very difficult. Mocks can help out tremendously here but if overused they can also put the application in a worse situation: where tests pass but the application itself doesn't work.

One of the easier mistakes to make when working with mocks is mocking implementation instead of behavior. I've seen a lot of code that makes this mistake, and it is frustrating to work with. One small change to the code, without changing behavior, breaks multiple tests and updating those tests is tedious and unproductive. I'll give an example of this kind of mocking, and how to fix it. Take the following method and test:

<pre><code data-language="ruby">
def some_action
  @models = MyModel.where(:active => true).order(:name).limit(10)
end

def test_some_action
  where_mock = mock
  order_mock = mock
  limit_mock = mock
 
  where_mock.expects(:order).with(:name).returns(order_mock)
  order_mock.expects(:limit).with(10).returns(limit_mock)
  MyModel.expects(:where).with(:active => true).returns(where_mock)
  
  some_action
  
  @models.should == limit_mock
end
</code></pre>

There is nothing here to show intent, it's all implementation, and any single change breaks the entire test even though the behavior may not change at all (for example, swapping the order and limit calls). This code and test is easily fixed up while still using mocks and staying away from database objects by using an Intent-Revealing Name (nothing is more important than [good naming]({% post_url 2012-10-01-naming-is-everything %})). Simply refactor that scope mess into it's own method and mock that one method.

<pre><code data-language="ruby">
def some_action
  @models = MyModel.active_sorted_by_name(10)
end

def test_some_action
  MyModel.expects(:active_sorted_by_name).with(10).returns([])
  get :some_action
  assigns(:models).should == []
end
</code></pre>

Now the code shows it's intent, the tests show how the functionality is supposed to work, and any changes made to active_sorted_by_name won't cause the test to fail. Testing the behavior of *active_sorted_by_name* can then be done with actual database objects and in one place only, minimizing the speed impact on the full suite.

There is one very important caveat: this pattern can only be used safely if you also have an end-to-end acceptance suite. I believe this is so important that I'm going to say it again: **If you are going to use mocks in tests you must have a separate acceptance-level test suite!** If there are no tests to prove that objects to actually communicate as the mocks say they do, then the mocked tests, in many cases, are actually a hinderance, passing when the application itself fails.

## Hide persistence details

Following along the previous point, and the [previous post]({% post_url 2013-01-28-implementing-persistence %}), if any code has to do with the details of how an object is persisted, hide that code inside of a method on that object, or in objects explicitly built to handle persistence logic. Controllers should never care how persistence is implemented. When these kinds of details are kept hidden from the rest of the application, the application's domain model can grow and evolve into a readable, reusable, understandable API that reveals the intent and capabilities of the application without bogging the reader down in unimportant implementation details.

## Interactors, or Use OOP!

If I was to pick one definite win in this experiment it's how the Interactor pattern has helped me understand what Object-Oriented Programming and Single Responsibility Principle really means. There's nothing magical about Interactors, in almost every case they should be just plain Ruby objects, but nothing helps the human mind out more than giving concrete names to a concept. As Uncle Bob pointed out in his talk, having a set of Interactor objects leads you to organize your source code in a way that makes it almost trivial to understand what an application does. Here's Raidit's [list of interactors](https://github.com/jasonroelofs/raidit/tree/master/app/interactors). Even if you didn't know what the application did, you can easily find out what operations are currently available by looking at the classes in this directory, vastly improving code readability and facilitating code reuse in ways I've never experienced in Rails applications.

This isn't to say **everything** should be an interactor. As stated before I took the opposite extreme position to help prove a point. There's nothing wrong with doing simple one-liners in a Rails controller, such as *#find*, vs building an entire new object just to encapsulate that one piece of logic (e.g. [FindRaid](https://github.com/jasonroelofs/raidit/blob/master/app/interactors/find_raid.rb)). As in all things, be pragmatic. If you ever get stuck wondering where functionality should go, and it doesn't fit a Controller or a Model, make a new object! That's all the Interactor pattern is: implementing functionality in a way that keeps all of the classes and objects small, doing one thing and doing it well.

## Listen to your Tests, Feel and Fix your Pain

So in closing, use Rails, but keep every class you build small and simple. Yes you will end up with more classes and more files but that's not a bad thing! Keep your source directory structure clean, don't be afraid to add more to the usual Rails layout, and overall **if you feel pain stop and figure out why, then fix that problem. Hiding pain leads to more, worse pain later!** Seriously, nothing will point you towards better code design than listening to your tests and listening to your code. Pain in software is a Good Thing. Listen to your tests, constantly refactor, and good design will come!

It's been fun to buck trends and intentionally go places I know I'd have troubles. I've learned a lot through this and discussions over code design and I hope I've helped out others in their quest to better software design as well.

What's next then? I plan on rewriting raidit in (Go)[http://golang.org/], and you can bet I'll be writing about that experience as well. 