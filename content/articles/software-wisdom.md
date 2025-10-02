---
title: "Software Wisdom"
date: 2025-10-03T12:00:00-04:00
draft: true
---

> Simple things should be simple, complex things should be possible.
>
>  - Alan Kay

They say "life comes at you fast", and as I've hit two major milestones in my life (40 years old and 20 years in a career), I've been doing a lot of thinking -- thinking about software, thinking about my career, thinking about where I've been and what I want to do next. I've learned a lot about software over these last two decades, and figured it's about time to write some of these lessons down.

I've worked, both directly and as a consultant, for companies of all sizes. Most of my career has been spent in the web, building applications with [Ruby](https://www.ruby-lang.org/) and [Ruby on Rails](https://rubyonrails.org/) as well as configuring and managing the infrastructure that runs said applications. I've worked with everything from bare metal servers to [Kubernetes](https://kubernetes.io/) clusters. I've also contributed to and run my own open source projects, though nothing I would consider large or significant. I haven't done everything you can do in software, but when you define an "expert" as "someone who has made all of the mistakes you can make in a field", I like to think I'm now an expert at software development.

Through all of this I've learned a few truths about software, and developed a number of mental tools and patterns to help me work with and around these truths. First, the truths.

## Software is Hard

Fields like Civil Engineering and Rocket Science are hard because the universe applies physical properties that cannot be ignored, and when not taken seriously, can lead to disaster. Software is hard for the opposite reason: there are *no* such rules. Every new project is a blank slate, and there are an uncountable number of ways one can implement any given project. For many, including myself, this is what draws people into software in the first place, and today all you really need is a computer and an Internet connection to get started.

What you quickly learn, though, is that this unbounded world inexplicably moves into and becomes a world full of rules and walls. It's natural to find yourself in a difficult codebase and think "I'll just rewrite this cleaner" but [that never works out](https://i.programmerhumor.io/2023/12/programmerhumor-io-programming-memes-3eff7a93a6c36ab.jpg). Instead, it's important to look at software as an organic medium. Software is always changing, either through new technology, new features, new requirements, or new patterns, and software should be built and designed to change. Working in software is both exhilarating and exhausting, and it often flip flops between the two multiple times a day. "Easy" changes end up taking weeks, "hard" changes can end up taking just a few lines, and who wrote this awful thing? Oh, I did, last month.

## Software is Complex

One pet peeve of mine in this field is the misuse of the terms ["complex"](https://www.merriam-webster.com/dictionary/complex) and ["complicated"](https://www.merriam-webster.com/dictionary/complicated). Complexity in software, where there are many parts working together to solve a problem, is normal and expected. We should be designing around the expectation of complexity, making sure a codebase remains understandable and malleable, able to adapt to changing requirements. Complicated software, where the many parts are intertwined in a way that makes them difficult to understand and work with, is always undesirable, but sometimes still unavoidable.

One very common way that "complex" becomes "complicated" is when various software components each grow to encapsulate too much logic and responsibility. What started as a well-managed set of simple components working together to solve a complex problem slowly grow into a spider's web of complex parts each trying to do too much. Such codebases are frustrating to work with as they are difficult to understand and change. Unfortunately, this is the natural progression of all codebases (one could say that "complicated" is the "entropy" of software) unless explicit care and discipline is put into staving it off. A codebase of simple components working together to solve complex problems never happens automatically. You must focus constantly and refactor mercilessly to keep it that way.

## Software is People

Many, if not most, people got into software (myself definitely included) because it promised a logic-based, clean world of computers, free from all of this messy "people" stuff like "communication" and "politics". One of the hardest lessons we as software developers have to learn (myself definitely included) is how wrong this mentality is.

Software is made by people for people. Through that, programming's most important job is *informing people what the computer is doing*. Software is generally written once and read hundreds, if not thousands of times. The vast majority of a software engineer's career is spent in an existing codebase, learning how it works and how to change it. It therefor behoves us to take extra care up front to always work on writing code that is clear, concise, and well commented, even when the other developer is just yourself in the future.

On top of this, every line of software is written to solve a problem that people have, but we people are messy, and while computers are perfectly logical, we are anything but. We make mistakes, we forget, we misremember, and most importantly we have a finite amount of mental energy each day. The vast majority of a software's users just need the software to do the job they are expecting it to do so they can spend their mental energy where they want to spend it. When software fails to work right, or gets in the way of the user, that is wasting energy, which quickly leads to frustration. We need to be sure that we are asking only exactly what we need from the user, and providing back what they expect.

With those truths defined, lets dive into some techniques I've developed to help me build better software:

## Tests are vital

I've long been a proponent of Test Driven Design (TDD) where I try to write my tests first. This has a number of benefits. First, it forces you to think about the behavior you're trying to implement: what inputs does it need, what outputs does it provide? Writing the tests first also helps narrow the range of said behavior, ensuring you write smaller individual units. Testing first also shines a spotlight on what dependencies that behavior requires. Do you need to mock out another part of your code, do you need a library, do you need to call another service? All of this impacts how easy or hard it is to test the behavior at hand.

In general, if find that the behavior in question is hard to test, that almost always means that the code needs some refactoring. This may not be easy, and one of my favorite quotes is from [Kent Beck](https://kentbeck.com/):

> For each desired change, make the change easy (warning: this may be hard), then make the easy change

I will also argue, though this can be considered semantics, that if you do not have tests, you cannot refactor. Refactoring implies that you are changing implementation without changing behavior, and if you don't have tests, you can't prove that behavior didn't change.

Of course some things are intrinsically harder to test than others. In such cases, test as much as you can, and minimize the footprint of what you can't.

## Twice isn't duplication. Thrice might be

It's a common drive to try to reduce duplication and build out abstractions to solve the problem you have, but building the wrong abstraction is an easy trap to fall into. Over the years I've come to understand that doing something twice is not duplication. Chances are the two "duplicate" code spots are going to follow one of two paths: they will immediately diverge, or you will find yourself writing the same code a third time. Once you're at three or four copies, then it's time to look and see if you can abstract out a common thread.

It's always easier to fix duplication than the wrong abstraction.

## Names Control Complexity

> There are 2 hard problems in computer science: cache invalidation, naming things, and off-by-1 errors.
>
>  - Leon Bambrick

It continues to surprise me how much of an impact choosing the right, or wrong, name has on the design and architecture of software. Almost every time I'm working on a feature or a refactoring, and something doesn't feel right, it means somewhere I have a name that needs to change. Once I find that misnamed piece of code, the rest of the work usually falls right into place. Similarly a bad name that gets left in the code for a long period of time can grow all sorts of problems. That name normally contains far more logic than it should, leading to leaky abstractions with too many other parts of the code depending on that one unit.

## Manage Your Dependencies

"Dependencies" can have multiple meanings, and all of them apply here. Whether this is how your code depends on itself, how much you depend on third-party libraries, or how much you depend on third-party services, if you want a long-term maintainable code base it is extremely important to follow two rules: minimize your dependencies, and keep them up-to-date.

In my experience, there is no problem more painful in this entire field than having to do a large upgrade of dependencies that are now years out of date. The Node ecosystem is by far the worst I've had to deal with but any application can get caught up in this. Either you spend the time every week, or every month, and do a round of upgrades, or you'll spend a lot of time later when the upgrades are now mandatory, and much, much more difficult.

I said earlier that software is organic, and like organic matter, software can rot. Minimizing your dependencies goes a long way to protecting against that rot, and the rest requires diligence to not fall behind. Ask yourself if you really need that dependency, or if there's something you can do yourself instead.

## Perfection is the enemy

The truth is sometimes you just have to do the ugly thing first. Whether this be a migration between systems or a problem that has been plaguing you for too long, eventually you just need something that works. Get it done, get tests passing, and maybe an apology in the comments to explain what happened, and make a note to return to that code later to try to clean it up (which, lets be honest, almost never happens). Make sure it's well commented for the next developer that comes along wondering what in the world happened here?

## Ignore everything I just said

Sometimes you just got to do things to break out of your comfort zone and learn something new. Build weird prototypes. Write crap code and throw it away. Try your hand at [IOCCC](https://www.ioccc.org/) or do something [totally off of the beaten path](https://en.wikipedia.org/wiki/Esoteric_programming_language) (I mean, just look at [Malbolge](https://en.wikipedia.org/wiki/Malbolge)). You can do nigh anything with software and nothing else provides such a powerful canvas for creativity. Sometimes you just have to put the professional aside and let the kid inside all of us have a little fun.

The world needs more people like [_why the lucky stiff](https://en.wikipedia.org/wiki/Why_the_lucky_stiff).
