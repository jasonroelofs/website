---
layout: post
title: "Rebooting Slartibartfast: Third Time's a Charm?"
published: true
tags:
- software
- slartibartfast
- golang
- game development
---

So game development is hard. Really hard. Some rank it up there with developing an operating system, as in many cases game engines are micro OSes. I've always wanted to build a proper game, to progress past the cloning of simpler games like Pong, Tetris, and Simon Says, but I've regularly found it  difficult to keep motivated on my "dream game" as described way back when in [Project Slartibartfast - Introduction](http://jasonroelofs.com/2011/03/09/project-slartibartfast-introduction/), and I've come to recognize two obstacles to my progress.

First, I know too much. I know how much work a game requires, I know how much raw code I'll need to write, I know how many technologies I need to learn, and I know that there's a lot I still don't know yet. Due to this I find myself frequently running into [Analysis Paralysis](http://en.wikipedia.org/wiki/Analysis_paralysis), and I often think about the people who get into game development first without any prior industry experience. On one hand these people seem to create some of the worst code I've ever seen, yet they work very hard and actually ship games. In many (most?) cases they simply don't know how big of a job they've got, so there's nothing stopping them from building and shipping. Unfortunately I don't think this is a mindset you can re-aquire once you've lost it and Analysis Paralysis is simply a mental state that needs to be conquered (best way I've found so far: keep breaking big tasks into smaller ones until you have simple, testable chunks of work).

Second is technology choices. C and C++ have long been the reigning champions of the game development world, with Java moving up the rungs slowly. 3D real-time games like the one I want to make simply require speed and while you can make a 3D engine in Ruby or Python, there's only so much that can be done before language choice starts becoming the main performance hinderance. Now I grew up on C and C++ so I'm very familiar with these languages. I've also spent time in Java but I've been spending most of my development in higher level languages like Ruby. After trying the project in C++ with the [Ogre Engine](http://ogre3d.org/) and in Java with [jMonkeyEngine](http://jmonkeyengine.org/) I've come to two realizations:

I never want to write in C++ or Java again for any serious project, and I don't want to fight existing game and rendering frameworks. I've kept the work for both of these attempts available at [https://github.com/jasonroelofs/slarti-history](https://github.com/jasonroelofs/slarti-history) for reference. It's only recently though that we are starting to get new languages that give C++ and Java a run for their money -- Google's [Go](http://golang.org/) and Mozilla's [Rust](http://www.rust-lang.org/). Both are statically typed, compiled, garbage collected, and very fast. Due to the simplicity of Go and the fact that it's more stable right now than Rust, I've decide to choose it as my next attempt at Project Slartibartfast, and so far, it has been a joy to work with.

I'm also finally learning the full ins-and-outs of OpenGL and modern graphics programming. Learning is fun, but man is there a lot of old, outdated, and misleading information out there. I'll be putting together a series of posts to help myself and others break through some of the more difficult barriers in graphics programming.

My current golang-based work is found at its usual place, [https://github.com/jasonroelofs/slartibartfast](https://github.com/jasonroelofs/slartibartfast).

Next I'll be discussing the code layout and overarching design decisions that's driving the development of the project, and maybe, just maybe, I'll end up with an actual game this time!