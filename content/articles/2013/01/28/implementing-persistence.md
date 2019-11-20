---
title: Implementing Persistence
date: '2013-01-28T12:00:00-05:00'
draft: false
series: designing_a_rails_app
titleOnly: true
aliases:
  - /2013/01/28/implementing-persistence
---

In [Part 3](/2012/07/13/abstract_persistence_logic) of this series of posts I talked about abstracting persistence layers and the various patterns used, as well as showing the pattern I built into [raidit](http://github.com/jasonroelofs/raidit). I've now built the ActiveRecord-based Repository implementation, thus finishing raidit, so now it's time to talk about how this went, what I learned, and if this pattern is a good model for future projects.

The resulting implementation can be seen in [app/repositories/active_record_repo.rb](https://github.com/jasonroelofs/raidit/blob/master/app/repositories/active_record_repo.rb).

The ActiveRecord models themselves are found in [app/repositories/active_record_repo/models](https://github.com/jasonroelofs/raidit/tree/master/app/repositories/active_record_repo/models).

Getting here unfortunately wasn't simple. This turned out to be the most complicated part of raidit, not only in building the mapping between domain models and ActiveRecord, but in dealing with bugs in the application that were hidden by the in-memory implementation. The first big issue I had to solve though was dealing with associations.

#### Associations

When working with in-memory objects, it's very easy to write code that expects objects and their associations to just exist. When I updated the app to serialize to a real database, these associations completely broke. Guilds no longer had their list of Raids, nor Users their list of Characters, for example. The last thing I wanted to do was build a half-baked version of ActiveRecord, so the quickest thing I could get working was to eagerly load every assocation on the requested object. This is fine for small sets of objects, but it can't possibly scale and is not easily maintainable.

Another solution I considered would require a redesign of most of the domain models to work more like ActiveRecord, where models would be able to query for their associations as they needed them, something like the following:

<pre><code data-language="ruby">
class Guild
  def raids
    Repository.for(Raid).find_all_by_guild(self)
  end
  
  def save
    Repository.for(Guild).save(self)
  end
end   
</code></pre>

But even with this, it's very easy to create data bugs related to memoization, or to feel you still have no real control over the queries an application is making for optimization purposes. Even so, there are further issues with this pattern.

#### The Empty State

As I worked on getting the acceptance tests running under ActiveRecord, I quickly realized that I had completely forgotten to implement raidit's Empty State. As in-memory data disappears on server reloads, I had added bootstrap data in an initializer to give me a good base state to play with. This hid the fact that the site broke completely for a new user with no characters and no guild. The simplicity of raidit meant this was easy enough to fix ([7303934a](https://github.com/jasonroelofs/raidit/commit/7303934a7945af01c99430fe0a3faeb92a15d27c)  and [188c48c5](https://github.com/jasonroelofs/raidit/commit/188c48c5d512da2bc167cc02f0a777f147eaa6c4)) but this could be substantially harder as an application gets more complex.

#### Effort vs Time Saved

It's always extra work to add an abstraction to a project. Abstractions require public APIs, converting data moving through the various layers, and implementation of the backends to handle specific details. All of these pieces require design, testing, and development. Abstractions emerge from refactoring messy or duplicate code to improve readability and maintainability, and good abstractions end up saving far more time in future development than it took to build them.

Does raidit's Repository model fit this? I'm not sure. Being able to run in-memory only means test are extremely fast ([https://gist.github.com/2886208](https://gist.github.com/2886208)). Compared to typical Rails app tests there is a lot of time saving here, but in terms of development cost, I have yet to experience any sort of time savings. While it's nice to have a clean set of methods I can call and mostly ignore persistence details, reality proves that this kind of API constantly grows. Each request requires a slightly different set of data, which requires a new API call, a new test, and updates to the underlying ActiveRecord models.

This very quickly became Not Fun and is the reason it took me so long to finish up raidit and get this post written. A good lesson I've tried to follow throughout my development career is that if some code or feature request fills me with dread, that probably means there's a fundamental issue with the design of the code or with the current development practice(s). Functionality should always be fun to add to an application. The Repository pattern, as I've implemented in raidit, removed a lot of that fun.

So would I recommend others follow this pattern? No, not as I've done it here. There may be ways, and there are probably applications that would work well with this pattern, but I would urge caution not to build in abstractions just to have them. So what about preventing another Typical Rails App? I propose the following development suggestions.

Don't leak persistence details outside of ActiveRecord (with the caveat that column names ending up in the view isn't in itself bad). For example, any code that chains scopes and/or ARel methods should refactored behind an Intention-Revealing Name <sup>[1](#footnote)</sup> (an instance method, class method, or object).

<pre><code data-language="ruby">
# Change this
current_guild.raids.order_by(:created_at).limit(10)

# To this
current_guild.recent_raids

class Guild < ActiveRecord::Base
  def recent_raids(count = 10)
    self.raids.order_by(:created_at).limit(count)
  end
end   
</code></pre>

You can still have fast tests as long as you aren't putting data in the database for every test. Have a suite of tests that do talk to the database to ensure that side of the app is working, then use bogus objects or mocks elsewhere. This pattern will only work if you also have an Acceptance test suite that tests the full stack, taking the role of "Contract Tests"<sup>[2](#footnote)</sup>. Mocked tests without tests that prove that both sides of the mock are correct will leave you with a brittle test suite that hides bugs in how your objects communicate.

In explicitly taking the other extreme from ActiveRecord coupling everywhere, I've learned a lot more about what not to do and how to find a good middle ground when dealing with serialization / persistence. Use objects and/or methods to hide implementation details (like ActiveRecord scopes and ARel). This gives you names which provide meaning and intent in your code and give you the hooks to start building fast tests and improve your overall design.

I'll have one more post in this series that takes the development rules I set for myself in [Rules for Rails App Development](/2012/06/05/rules-for-rails-app-development) and lists out which ones are good, which ones didn't work out, and overall thoughts on this experiment.

---

<a name="footnote"></a>
1: [Smalltalk Best Practice Patterns](http://www.amazon.com/Smalltalk-Best-Practice-Patterns-Kent/dp/013476904X) Get this book if you're serious about learning how to Think in Objects.

2: [Integration Tests are a Scam](http://www.infoq.com/presentations/integration-tests-scam) by [J.B. Rainsberger](http://www.jbrains.ca/)
