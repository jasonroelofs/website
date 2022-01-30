---
title: "What's Next?"
date: 2022-02-01T12:00:00-05:00
draft: false
series: whats-next
---

It has been some time since I last wrote about technology and software development. I've been professionally developing software for over fifteen years, working for multiple startups, as a consultant, and currently at [Stripe](https://stripe.com). I can map over my career how my skills and methodologies of developing software has improved, but there is one question that continues to bug me: What's next?

Now, this isn't a "What's next for me?" but "What's next in computing?". Don't get me wrong, computing is an amazing field to be a part of. The capabilities we have today were unfathomable even 10 or 20 years ago, much less when our field started back in the 1950s and 60s! We have access to a huge and increasing breadth of tools to solve problems across all fields and walks of life and in the past ten years we have seen a new wave of great programming languages such as Go, Rust, Elixir, and TypeScript. These languages let us build faster, more scalable, safer software, than we've ever been able to before.

That said, I can't help but also feel that our field has stagnated. For all of the people playing around with programming languages (including [myself](https://github.com/jasonroelofs/language)) we have not seen nor found the next leap, the next major improvement that lets us think, process, and go even further with computers. When we dive into the [history of programming languages](https://en.wikipedia.org/wiki/History_of_programming_languages), we can see that every major paradigm in software, from procedural and imperative languages, through object-oriented, purely functional, and logical was all fleshed out and well understood by 1980, over 40 years ago. Every language since then has combined and/or improved upon these paradigms in different ways and I've yet to see any language today that makes me think "huh, there's something new here."

And if even the legendary Alan Kay, the inventor of Object-oriented programming and the SmallTalk language, who spent 16 years researching new paradigms through the [Viewpoints Research Insitute](http://www.vpri.org/) didn't find it (though the papers and the research they did do is amazing and worth familiarizing yourself with), you start to wonder if there is a next step at all.

Or maybe we're looking in the wrong direction. Maybe the next step isn't going to come from programming language research at all.

Delving deeper into our field's history, you'll eventually come across a little thing called the [Dynabook](https://en.wikipedia.org/wiki/Dynabook), by Alan Kay. The  Dynambook was intended to make computing so easy and intuitive to use that it would be a fundamental educational tool, making computing accessible to all. While Steve Jobs and the iPhone and iPad were heavily influenced by this idea, it's easily argued that Jobs missed the entire point of the Dynabook, as the iPad and iPhone were then designed and built for consumption instead of education.

Millions of people today carry around computational devices in their pockets that puts even the mighty Cray supercomputers of the 90s to shame, and yet the true power of these devices is locked away. We rely on others to build the apps we use, whether they be for productivity, education, or consumption (e.g. games, videos, and music). For the vast majority of people, there's no other way to use these or other devices. Even for those of us who understand software and computing, it's still significantly harder than it should be to get a computer do what you want.

That's all true except for one paradigm described way back in 1961 that exploded into popularity in 1979: the [Spreadsheet](https://en.wikipedia.org/wiki/Spreadsheet). Whether it's Numbers on a Mac, Microsoft Excel, or Google Spreadsheets, no tool has made computing more accessible than the spreadsheet. Its paradigm is intuitive: enter in data, compute on that data. It's so useful that even the largest companies in the world still rely on spreadsheets every day to keep the business running. That countless people can both claim to be incapable of understanding or using computers yet still build out extensive programs in spreadsheets shows how successful this paradigm has been: people are programming computers without even realizing it!

That isn't to say the current implementations of the spreadsheet are perfect, as nothing ever is. Spreadsheets focus so much on the data that the logical layer often becomes very difficult to keep track of, making maintenance an increasingly burdensome task. However, nothing else in 40 years has ever come close to displacing the spreadsheet as the computational tool of choice, though not for lack of trying. But I feel that so many people have missed why the spreadsheet is so popular. While plenty of "no-code" solutions exist, they still miss the fundamental difference between "software development" and the spreadsheet:

* Software focuses on logic first.
* Spreadsheets focus on data first.

And, in my opinion, the most important difference:

* Spreadsheets are live environments.

Every software system, with some notable exceptions, require the developer to start up the entire system, run some code, then destroy everything, for every single change the developer wants to test. As software has become more complex, this has naturally led to frustrating situations where it can take minutes to test each change. Sure, there are plenty of tools to help hide this fact and some people have attempted to graft "live reload" into many frameworks, but that's putting a band-aid over a fundamental problem with software development today. To quote Bret Victor in his [Learnable Programming essay](http://worrydream.com/#!/LearnableProgramming): "But there is no future in destroy-the-world programming." There really must be a better way of writing software.

In the articles to come I'll be diving deeper into many of these topics as well as describing and designing some projects I'll be starting to help provide more concrete examples of what I'm talking about.
