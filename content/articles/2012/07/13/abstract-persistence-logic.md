---
title: Abstract Persistence Logic
date: '2012-07-13T12:00:00-05:00'
draft: false
series: designing_a_rails_app
titleOnly: true
aliases:
  - /2012/07/13/abstract_persistence_logic
---

Choosing a persistence library at the start of a project is almost always a mistake. As the beginning is the point of highest ignorance, why make such important and hard-to-change decisions before there's any definitive knowledge of the real needs of the application? The most important aspect of any given application is it's business rules and domain model, both of which are code. In the end, users don't care how data is persisted, but they do care that the app works as expected. Focus on the application and everything else will grow from there. Persistence is an implementation detail and thus should be a low priority task.

There are a number of benefits to deferring this decision. First and foremost is that this deferment will lead to the construction of a persistence API for your application. This API then makes any persistence requirement trivial to add to the project when it's needed. Secondly, because persistence is deferred, and because every application needs data, the first persistence implementation will be a simple set of in-memory collections. This makes the code, and more importantly the tests, extremely fast.

While I love ActiveRecord and wouldn't use anything else for SQL-based databases, Rails and the typical way Rails apps are built does not make this deferment easy. Rails wants us to use ActiveRecord and it wants us to not care where we let ActiveRecord code slip in. If I had to point at one specific habit of Rails developers that has caused the most maintenance pain, it's the hard coupling of the database  details to **everything** in the app, especially the views and forms (quick note: use ActiveModel, it's awesome). So, to ensure we don't end up in this morass again, we need to look elsewhere for our solution.

It's not hard to find many different patterns developed over decades for solving this problem. Most of the best patterns can be found in Martin Fowler's book [Patterns of Enterprise Application Architecture](http://martinfowler.com/books/eaa.html) (PoEAA). The two I spent the most time investigating are [Repository](http://martinfowler.com/eaaCatalog/repository.html) and [Data Mapper](http://martinfowler.com/eaaCatalog/dataMapper.html) along with a number of existing Ruby libraries implementing these patterns.

The pattern I chose needed to fit two criteria: simple and pluggable, as I needed to be able to have an in-memory implementation for development and tests right next to any true persistence implementation. I chose to go with Repository in raidit over DataMapper as DataMapper, like ActiveRecord, still adds too much knowledge of persistence to the models (e.g. User#save) and I want a clear deliniation between persistence and the application. The Repository pattern does exactly this. Because Ruby is so dynamic, implementing this pattern is very simple. I'll show two implementations here.

**Note:** I'm implementing a very basic version of Repository. Where PoEAA talks about a Criteria system (which some have interpreted as an entire library or language, like ARel or Hibernate's HQL), I want to use nothing more complicated than method calls. I don't want any code outside of a Repository implementation knowing how data is stored or queried.

The first implementation, and the implementation that raidit currently uses, consists of a single, simple object to map models to the appropriate repository implementation. As you can see here, this object is nothing more than a few helper methods on top of a Hash:

{{< highlight ruby >}}
class Repository

  ##
  # Add mapping(s) to this repository. This will add to the existing known
  # set of mappings. To start the mapping list anew, use +.reset!+ first.
  ##
  def self.configure(options = {})
    @mappings ||= {}
    @mappings.merge!(options)
  end

  ##
  # Clear out all known mappings
  ##
  def self.reset!
    @mappings = {}
  end

  ##
  # Find the defined mapping for the given Domain Model class
  ##
  def self.for(model_class_or_name)
    @mappings[model_class_or_name] || @mappings[model_class_or_name.to_s]
  end

end
{{< /highlight >}}

It's usage is simple. First define how the classes map to their persistence:

{{< highlight ruby >}}
  Repository.configure(
    "User"        => InMemory::UserRepo.new
  )
{{< /highlight >}}

then request the persistence object whenever persistence is needed:

{{< highlight ruby >}}
  user = User.new
  Repository.for(User).save user
{{< /highlight >}}

An alternate implementation that's arguably more Ruby-esque is to simply set constants to the implementations, using those constants directly as needed, like so:

{{< highlight ruby >}}
  UserRepository = InMemory::UserRepo.new

  ...

  user = User.new
  UserRepository.save user
{{< /highlight >}}

Both setups have a number of benefits outside of the decoupling of persistence. Having this intermediate layer allows having multiple different implementations of persistence, even to the point of communicating with multiple different databases at the same time if so required. This layer also ensures that the models themselves know nothing about the persistence implementation, so I can be sure that there is no leakage of responsibilities.

You can see the current implementation of raidit's various in-memory repositories in [app/repositories/in_memory.rb](https://github.com/jasonroelofs/raidit/blob/master/app/repositories/in_memory.rb). You can also follow along on a gist I'm continuously updating with full test suite timing runs here: [https://gist.github.com/2886208](https://gist.github.com/2886208). As you can see, raidit's tests are super fast, even the cucumber features!

Now, once I get to the point where I know what persistence library will work best for raidit, I'll have a fully defined API for what the application needs, letting me implement only and exactly what's needed and ensuring that the tests for these objects are segregated from the rest of the app, keeping the application tests fast forever.

Refusing to make decisions now can lead to huge benefits in how the application performs, it's architecture, and maintainability. So stop writing ActiveRecord spahetti code and take control of your application before your database takes control of you, again.
