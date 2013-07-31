---
layout: post
title: Application Development in Go
published: true
tags:
- software
- slartibartfast
- golang
---

Getting started with Go is delightfully easy. Not only are the [official docs](http://golang.org/doc/) detailed and comprehensive, but the Go team also put together [A Tour of Go](http://tour.golang.org/#1), a large set of tutorials runnable right in the browser. For writing quick scripts Go comes with the `go run` command, which compiles and runs a single go file as you would any language like Ruby, Python or Perl.

When it's time to build libraries and more complicated command line tools, [How To Write Go Code](http://golang.org/doc/code.html) is ready and waiting (I highly recommend spending the time to read the whole thing, it's how Go wants to work). And from there, [Effective Go](http://golang.org/doc/effective_go.html) is full of hints, tips, and suggestions on how to write good, readable Go code.

As a fair warning: Go is a strongly opinionated language, and by proxy so is the community. There is only one way to write Go code. Brackets go on the same line, `if` always needs brackets, if you're handling one return value, you must handle them all, and so on. Go code doesn't even compile if these rules are not followed. Unused package imports and unused local variables are also compiler errors. So if you're building libraries and command line tools and not using an environment as described in [How To Write Go Code](http://golang.org/doc/code.html), it's going to feel harder than it should, and questions to the mailing list will probably result in "You aren't following the Go way" with directions back to this document.

The one aspect of development that the Go docs do not cover, however, is application development: any software with multiple packages, probably a number of 3rd party libraries, and often non-source-code files and data. In Go terms, this would be any software that no-one would ever `go get` for use in their own project. I spent a lot of time reading the docs, experimenting, and discussing this problem on #golang. I have a solution now, though it's far from ideal and I'm sure others in the Go community will disagree with some or all of my choices here. I'd love to have more discussion on this problem, as I don't feel there is a good solution yet, but here's how I'm currently handling the issues of application Go development.

### $GOPATH and Your Workspace

As per "How To Write Go Code", the `$GOPATH` environment variable must exist and must point to a Go Workspace, which is defined as a directory with `src/`, `pkg/`, and `bin/` directories. While not explicitly said, the Go Workspace is a developer-set area, meant to hold all Go code under development. This doesn't work for application development for a few reasons. First, a developer can't expect anyone to individually `go get` 20-plus different packages ([Slartibartfast has more than 10](https://github.com/jasonroelofs/slartibartfast/tree/master/src) and growing), not to mention whatever 3rd party dependencies the application uses, to work on a single application. Secondly,  after all those `go get`s, the developer's Go Workspace is now a mixture of their existing projects and the application's packages, all sitting happily next to each other. This will make any sort of development difficult, as the developer is forced to remember which packages the application uses, and which are other projects. Third, each package is expected to be in its own source repository (for easy `go get`-ing), further complicating application development.

For Slartibartfast I decided to give this project its own Workspace, with it's own `$GOPATH`, and put the entire Workspace into git. You can see this on github at [jasonroelofs/slartibarfast](https://github.com/jasonroelofs/slartibartfast). This way all development, which often happens across multiple packages, are all a part of the same history, and there's no misunderstanding of what code is Slartibartfast and what code isn't.

### 3rd Party Dependencies

Putting the entire Workspace in source control has it's own issues though, the biggest being dependencies you `go get`. `go get` will happily pull from a git, mercurial, or svn repository and drop the checked-out code right in the Workspace as per the expected directory structure. This is not a problem when the Workspace is just a directory, but when `go get` pulls a git repository into another git repository, problems abound. Git recognizes that this new repository is a "subproject" and ignores it, expecting manual handling of the directory by the developer. The code is now in a unreproducible state, and any developer checking out the repository on another machine will end up with an unbuildable project.

While `go get` probably works great for the needs inside of Google, it is a simple solution to a complex problem; a problem more people are trying to solve, though as of this writing no suggestion has made it past prototyping. Because the most important aspects of application development is build reliability, I had to look elsewhere for managing dependencies. Unfortunately the only solution that really works for me now is using [git submodules](https://github.com/jasonroelofs/slartibartfast/blob/master/.gitmodules) and rejecting `go get` altogether.

### Non Code Resources

The last major issue that an application-only Go Workspace solved for me was handling non-code resources. For a game like Slartibartfast, this includes 3d models, textures, and graphics shaders. There simply is no place to put such resources in the normal Go Workspace, as anything placed in the `src/[package]` directory gets compiled into the resulting binary and is not accessible or editable from outside. Having my full Workspace as it's own repo lets me create new directories to hold these files. Fortunately it turns out that a go executable will use `$GOPATH` when looking for files on the file system, and not finding that that will set `$GOPATH` to it's own current location, allowing these resources to live along side any binaries I build and allowing simple packaging.

So that's how I'm currently developing Slartibarfast: the full Workspace goes into git and dependencies are handled by git submodules. This ensures I always have a repository anyone can clone from github and receive the exact same code as I currently have.

I would love to hear from anyone else who's had to deal with these issues and what they've done to ensure good development practices. I doubt what I'm doing is ideal, and I feel there's a lot to be discussed yet in developing Go applications.































