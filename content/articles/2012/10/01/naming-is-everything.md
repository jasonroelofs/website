---
title: Naming is Everything
date: '2012-10-01T12:00:00-05:00'
draft: false
series: designing_a_rails_app
titleOnly: true
aliases:
  - /2012/10/01/naming-is-everything
---

I would like to start this off by recommending all developers who care about building readable, well designed code go get a copy of Uncle Bob's [Clean Code](http://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882/ref=sr_1_1?ie=UTF8&qid=1348537881&sr=8-1&keywords=clean+code). This book is a treasure trove of examples on refactoring and renaming to make cleaner code.

---

I've come to realize that there is nothing more important to crafting good, well designed code than choosing good names. Software could be exquisitely designed, with multiple levels of tests, and a beautiful web of clean, decoupled objects and services but all that work will be for naught if another developer can't read and understand the resulting code. Software development is two processes: Informing others what the computer is doing and Telling the computer what to do. If the code fails at the first then any work done towards the second is for all intents and purposes moot. Computers couldn't care less, but the people who follow up on our work definitely do.

The most unfortunate consequence of the "get it done quick" culture is we forget our code will be read and maintained by someone else at some point in the future, and we confuse our understanding of our code with code being understandable. Why? Because the first thing get-it-quick development does is skimp on naming. We end up with classes named Controller, Manager, and Factory. We end up with God objects that sit on top of massive hierarchies and bloated Utility objects because we couldn't think of another place to put said code. We write methods with ambiguous names, or worse we leave in names that no longer match the method's functionality. We write code blocks full of single-letter variable names, use names with no rhyme or reason (enterprise = spock + kirk) or even reuse the same small set of variables requiring the reader to keep register memory in his or her head.

And yes, I have done pretty much all of this, though if I've "spock + kirk"'d up any code I don't remember, and I'm terribly sorry if I have. At least that kind of code tells the reader outright that what they are reading is nonsensical. I have also let curse words into production, something I will never, ever let happen again, but I digress...

So what am I doing to stop this madness, and what can we all do to make code readable, understandable, and maintainable?

<p style="text-align: center; font-size: 30px; font-weight: bold; padding-top: 20px; padding-bottom: 20px;">Choose Good Names</p>

Of course, there's one *small* caveat...

> There are two hard things in Computer Science: cache invalidation and naming things.
>
> -- Phil Karlton

  Choosing good names is hard. *Really* hard, but anyone who said software development was easy was lying. Oh yes giving directions to a computer is rather simple, but as I said previously, that's a small part of what we do. As many prominent developers have been saying lately, Software is a Social Art. We are constantly learning from each other, sharing tips and tricks, yet we then turn around and drop turds of code on each other. Software is written once, maybe modified a few times, but read many times more, by people we do and do not know, and every step of our development should live and breath this fact.

Even the simplest methods are exponentially more understandable with good naming. Take a copy method, which is easier to read?

<pre><code data-language="ruby">
  def copy(x, y)
    x.member1 = y.member1
    x.member2 = y.member2
  end
</code></pre>

vs

<pre><code data-language="ruby">
  def copy(from, to)
    to.member1 = from.member1
    to.member2 = from.member2
  end
</code></pre>

A simple set of names makes the method immediately and completely descriptive without even looking at the implementation. It's copying data from the object at *from* to the object at *to*. With the nondescript *x* and *y* parameters we have to look at the implementation to know which one is copying to and from where. Taking the care, the few seconds or few minutes to write out the variables in full, to name your objects descriptively, to make sure the code is readable, will pay off in spades. Spending ten minutes now will save the next developer potentially hours of maintenance work.

To help with this understanding I've pulled together a number of examples from [raidit](https://github.com/jasonroelofs/raidit) on good naming, some bad naming, and some refactoring I've done to get to better naming.

#### Parameters and Variables

[Renaming params to attributes](https://github.com/jasonroelofs/raidit/commit/3a942a83aeece200c7c40661ccbc1cec55d37e64)

Parameter names don't have to be short to be bad. Sometimes the name just has too much leeway in what it could be. In this case *params* is too open of a name. I don't want to be passing any random bit if data into this method through it's parameters. What I'm passing down are the *attributes* of the model I'm building, so lets name the parameter as such.

[ChangePassword](https://github.com/jasonroelofs/raidit/blob/5cf5cd01255721954c6eb143cf739fe053a82657/app/interactors/change_password.rb)

Verbose variable names are always better than one- or two-letter names. My choice of very explicit, verbose variable names in this object leaves no room for doubt or confusion when reading this code.

#### Methods and Functions

[Refactor #run to a static method](https://github.com/jasonroelofs/raidit/commit/b4c079816225270b946eaa12e9900067f2476a64)

As I mentioned in [Part 2](/2012/06/05/rules-for-rails-app-development) of this series, as well as in my post at the [Collective Idea blog](http://collectiveidea.com/blog/archives/2012/06/28/wheres-your-business-logic/), I've been defaulting my Interactor objects to have a *#run* instance method which contains the main chunk of functionality for that action. Now I was at the time aware that *run* is a not a good name and as I have built out the functionality in raidit I've been finding better names for these methods. I've also come to understand and appreciate that [Dude, not everything is an object](http://steve-yegge.blogspot.com/2006/03/execution-in-kingdom-of-nouns.html) and have been changing some simpler Interactors to be static methods with names that are very explicit on what they do and/or return.

#### Constructors

Now this may come across as an odd section given my focus on Ruby, but a constructor is nothing more than a method that builds and returns a new object. In Ruby's case, any static method that returns an instance of the class said method resides in can be considered a constructor. For example, take the following class with a few named constructors:

<pre><code data-language="ruby">
class ListMessages

  def self.received_by(user)
    new(:received, user)
  end

  def self.sent_to(user)
    new(:sent, user)
  end

  def initialize(sent_or_received, user)
    @messages = Message.send("#{sent_or_received}_by", user)
  end

end
</code></pre>

Then compare the code to run them

<pre><code data-language="ruby">
messages = ListMessages.sent_to(user)

# vs

messages = ListMessages.new(:sent, user)
</code></pre>

The differences are subtle but noticable. The first one reads more like a sentance, a statement of intent, while the second requires you to understand that you need to pass in a symbol stating which type of Message you want for the given user. When code tells you it's intent without getting you bogged down in the implementation details, then you know you have good naming and are progressing towards better overall design.

#### Behavioral Naming and Consistency

Once you're in the habit of spending time picking good names, and figuring out how to write readable code, you'll find yourself in situations where the names you've chosen are fine names in and of themselves but there's something still not right about them. You may be running into definition and naming inconsistencies. One of the less immediately obvious aspects of well designed, well named code is that the names are consistent and follow a convention across the application. If you have Create and Add that do the same thing, you should pick one and refactor the rest. In raidit, I have the following conventions for my Interactors:

* Find[Type]    - Return a single object of the given type
* List[Types]   - Return a list of objects, though that list can be empty
* Update[Type]  - Process an existing object with new information

For app-specific functionality I work to keep the nomenclature following the application's needs and not named according to any implementation details (such as being a website). For example, I used to have *FindRaid* named *ShowRaid* ([commit](https://github.com/jasonroelofs/raidit/commit/8d3e8b4)) but [reverted](https://github.com/jasonroelofs/raidit/commit/be40902) that when I realized that the action had nothing to do with "showing"; it was pulling the object from persistence. Adding a new raid is called [ScheduleRaid](https://github.com/jasonroelofs/raidit/blob/master/app/interactors/schedule_raid.rb). Creating a new account is done in [SignUpUser](https://github.com/jasonroelofs/raidit/blob/master/app/interactors/sign_up_user.rb) and it's counterpart is [LogUserIn](https://github.com/jasonroelofs/raidit/blob/master/app/interactors/log_user_in.rb) (and you know, I should fix those to be either LogInUser or SignUserUp), in keeping with the wording used on the site itself: Sign Up, Log In, Log Out. With objects named like this, we now have an application structure that tells you what the application does straight from the file names. Check it out for yourself! [raidit/app/interactors](https://github.com/jasonroelofs/raidit/tree/master/app/interactors)

---

As I've developed raidit, almost every time I've gotten the "this isn't quite right" feel, or whenever my test setup started to get unwieldy and painful, the most common fix has been to figure out which name is wrong. Once I've found the offending name, either via changing an existing name or extracting code from one name and giving that code its own name, the rest has simply fallen into place. Good naming may be one of the hardest jobs in software but it's also one of the most important. We must work to ensure our code is readable by other developers, not just something we understand.

Bad names facilitate bad design. Good names breed good design. Spend the time to do this part right and the rest will follow.
