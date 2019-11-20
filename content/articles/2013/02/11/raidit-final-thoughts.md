---
title: "RaidIt: Final Thoughts"
date: '2013-02-11T12:00:00-05:00'
draft: false
series: designing_a_rails_app
titleOnly: true
aliases:
  - /2013/02/11/raidit-final-thoughts
---

Back in May of 2012 I [started an experiment](/2012/05/29/designing-a-rails-app-part-1) to help me learn how to develop and design Rails applications to be maintainable in the long term. Specifically I wanted to figure out how to stay away from Fat-Model Massively-Coupled code bases typical in Rails applications. To ensure I gave myself an environment of learning and didn't fall into old habits through this process, I gave myself a set of [rules for development](/2012/06/05/rules-for-rails-app-development). These rules were intentionally set to be extreme opposites of the typical Rails-Way development practices. The rules were as follows:

* Don't start with Rails
* Don't start with ActiveRecord
* Don't mock anything I own
* No persistence at all, anywhere
* Interactors are the public API of the app

As of today my experiment is complete and now it's time to go over each of these rules, what I learned, and what I recommend going forward.

## Start with Rails

I broke the first rule [very early on](https://github.com/jasonroelofs/raidit/commit/f7951c587fe4b85a2558a1dae45a79666062dad6) when I realized I was building functionality before I needed it. I was building interactors according to the design I had written out in my notebook but was quickly getting stuck deciding what part to implement next. Having long preferred to start as high up as possible when developing a feature and working down the stack, starting in the middle of the stack was very uncomfortable and causing me to be less, and even counter-, productive. I had initially wanted to start without Rails so I could focus on just the application, but quickly came to understand that the User-facing view is a very important part of the application, so time for Rails to come in. Once Rails was in place and I was building View-First it wasn't long before I was deleting and rewriting the code I originally wrote, as it simply didn't work with what the app actually needed. Building from the top down guarentees you are building only and exactly what's needed.

## Start with ActiveModel

You can read a more detailed account of my adventures in persistence abstraction in the [previous post](/2013/01/28/implementing-persistence) but if you're using Rails and you don't want to dive into full database setups yet, use ActiveModel. Rails and ActiveModel are best buds, letting you send your own non-ActiveRecord models around inside of Rails helpers and url methods with reckless abandon. I've never won a fight against Rails, and ActiveModel now exists to help people stay in Rails while not using all of Rails. All of raidit's domain models are [ActiveModel entities](https://github.com/jasonroelofs/raidit/blob/master/lib/entity.rb), and Rails couldn't care less that they aren't ActiveRecord. You can even use ActiveModel outside of Rails! It's a fantastic library, and if you aren't familiar with it yet, [check it out!](https://github.com/rails/rails/tree/master/activemodel)

## Mock? Don't mock? Testing is hard

Mocks and bogus objects are a great way to ensure that tests are isolating the code in question. One of the biggest pain points of most Rails applications is testing and the misuse or non-use of test isolation. In many cases, every test in the suite tests the entire stack, requiring data in the database and calling code that ends up modifying the database. This is the main cause of slow test suites, and fixing this on established applications is very difficult. Mocks can help out tremendously here but if overused they can also put the application in a worse situation: where tests pass but the application itself doesn't work.

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
  
  get :some_action
  
  assigns(:models).should == limit_mock
end
</code></pre>

There is nothing here to show intent. This only tests implementation, any single change breaks the entire test even though the behavior may not change at all (for example, swapping the order and limit calls), and the test is very hard to read. This code and test is easily fixed up while still using mocks and staying away from database objects by using an Intent-Revealing Name (nothing is more important than [good naming](/2012/10/01/naming-is-everything)). Simply refactor that scope mess into it's own method and mock that one method.

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

Now the code shows it's intent, the tests show and prove behavior, and any changes made to `active_sorted_by_name` won't cause the test to fail. Testing the behavior of this new method can then be done with actual database objects and in one place only, minimizing the speed impact on the full suite.

There is one very important caveat: this pattern can only be used safely if you also have an end-to-end acceptance suite. I believe this is so important that I'm going to say it again: **If you are going to use mocks in tests you must have a separate acceptance-level test suite!** If there are no tests to prove that objects to actually communicate as the mocks say they do, then the mocked tests, in many cases, are actually a hinderance, passing when the application itself fails.

In raidit I made sure to have a full end-to-end [Cucumber test suite](https://github.com/jasonroelofs/raidit/tree/master/features) to ensure the mocks in my [controller tests](https://github.com/jasonroelofs/raidit/tree/master/test/controllers) were correct. I did still follow my original plan of not mocking what I own; considering Rails Controllers outside of "ownership" of the application. None of the domain-level unit tests use mocking, and with in-memory objects that worked really well, though please see my [previous post](/2013/01/28/implementing-persistence) for the caveats of this approach. I'm no longer in the "mocks are bad!" crowd but I still am careful to make sure I'm using them correctly and to have another test suite to keep my mock use in check.

## Hide persistence details

Following along the previous point, and the [previous post](/2013/01/28/implementing-persistence), if any code has to do with the details of how an object is persisted, hide that code inside of a method on that object, or in objects explicitly built to handle persistence logic. Controllers should never care how persistence is implemented. When these kinds of details are kept hidden from the rest of the application, the application's domain model can grow and evolve into a readable, reusable, understandable API that reveals the intent and capabilities of the application without bogging the reader down in unimportant implementation details.

## Interactors, or Use OOP!

If I was to pick one definite win in this experiment it's how the Interactor pattern has helped me understand what Object-Oriented Programming and Single Responsibility Principle really means. There's nothing magical about Interactors &mdash; in almost every case they should be just plain Ruby objects &mdash; but nothing helps the human mind out more than giving concrete names to a concept. As Uncle Bob pointed out [in his talk](http://www.confreaks.com/videos/759-rubymidwest2011-keynote-architecture-the-lost-years), having a set of Interactor objects leads you to organize your source code in a way that makes it almost trivial to understand what an application does. Here's Raidit's [list of interactors](https://github.com/jasonroelofs/raidit/tree/master/app/interactors). Even if you didn't know what the application did, you can easily find out what operations are currently available by looking at the classes in this directory, vastly improving code readability and facilitating code reuse in ways I've honestly never experienced before.

This isn't to say **everything** should be an interactor. As stated before I took the opposite extreme position to help prove a point. There's nothing wrong with doing simple one-liners in a Rails controller, such as `#find`, vs building an entire new object just to encapsulate that one piece of logic (e.g. [FindRaid](https://github.com/jasonroelofs/raidit/blob/master/app/interactors/find_raid.rb)). As in all things, be pragmatic. If you ever get stuck wondering where functionality should go, and it doesn't fit a Controller or a Model, make a new object! That's all the Interactor pattern is: lots of small objects that each have as few responsibilities as possible.

## The Problem Isn't Rails

When you get down to it, Rails does not encourage bad code. There seems to be a case of tunnel vision, where developers get stuck in the mindset that "if Rails doesn't give it to me it doesn't exist." We need to remember that Rails isn't in control, we are. Rails is just the framework, we should always strive to keep our code well managed and organized. As Kent Beck says in his book [Smalltalk Best Practice Patterns](http://www.amazon.com/Smalltalk-Best-Practice-Patterns-Kent/dp/013476904X) 

> Good code invariably has small methods and small objects. Only by factoring the system into many small pieces of state and function can you hope to satisfy the "once and only once" rule. I get lots of resistance to this idea, especially from experienced developers, but no one thing I do to systems provides as much help as breaking it into more pieces.

So in all this, my suggestions for keeping your Rails application code base under control is as follows:

* Keep controller actions short.
* Keep models short.
* Build lots of small objects, and put them in well named locations, like `app/[object-type]`. E.g. put decorators in `app/decorators` (and update `config.autoload_paths` appropriately).
* Only put in `lib/` what can be pulled into a gem. We need to stop treating `lib/` as a catch-all dumping ground.
* If it's a domain model, put it in `app/models`. The class doesn't have to be ActiveRecord::Base to live there.
* And above all else: Listen to your code! Test pain is a good thing, it's telling you there's a problem with the design. Refactor until the code is pain free once more.

It's been fun to buck trends and intentionally go places I know I'd have troubles. I've learned a lot through this and discussions over code design and I hope I've helped out others in their quest to better software design as well.

What's next then? I plan on rewriting raidit in [Go](http://golang.org/), writing about my experiences along the way. Go is a fascinating language that is filling the gap between low-level (C / C++) and high level (Ruby / Python) languages, with full Erlang-esque concurrency support. I highly recommend at least taking a look at Go and Mozilla's similar language [Rust](http://www.rust-lang.org/). I've probably spent too much time only in Ruby and need to get branching out to new languages again.
