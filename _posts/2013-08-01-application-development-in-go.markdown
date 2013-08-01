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

When it is time to build libraries and more complicated command line tools, [How To Write Go Code](http://golang.org/doc/code.html) is ready and waiting (I highly recommend spending the time to read the whole thing, it is how Go wants to work). And from there, [Effective Go](http://golang.org/doc/effective_go.html) is full of hints, tips, and suggestions on how to write good, readable Go code.

As a fair warning: Go is a strongly opinionated language, and by proxy so is the community. There is only one way to write Go code. Brackets go on the same line, `if` always needs brackets, if you are handling one return value, you must handle them all, and so on. Go code does not even compile if these rules are not followed. Unused package imports and unused local variables are also compiler errors. So if you are building libraries and command line tools and not using an environment as described in [How To Write Go Code](http://golang.org/doc/code.html), it is going to feel harder than it should, and questions to the mailing list will probably result in "You aren't following the Go way" with directions back to this document.

The one aspect of development that the Go docs do not cover, however, is application development: any software with multiple packages, probably a number of 3rd party libraries, and often non-source-code files and data. In Go terms, this would be any software that no-one would ever `go get` for use in their own project. I spent a lot of time reading the docs, experimenting, and discussing this problem on #golang. I have a solution now, though it is far from ideal and I am sure others in the Go community will disagree with some or all of my choices here. I would love to have more discussions on this problem, as I do not feel there is a good solution yet, but here is how I am currently handling the issues of application development in Go.

### $GOPATH and Your Workspace

As per "How To Write Go Code", the `$GOPATH` environment variable must exist and must point to a Go Workspace, which is defined as a directory with `src/`, `pkg/`, and `bin/` directories. While not explicitly said, the Go Workspace is a developer-set area, meant to hold all Go code under development. This does not work for application development for a few reasons. First, a developer can not expect anyone to individually `go get` 20-plus different packages ([Slartibartfast has more than 10](https://github.com/jasonroelofs/slartibartfast/tree/master/src) and growing), not to mention whatever 3rd party dependencies the application uses, to work on a single application. Secondly,  after all those `go get`s, the developer's Go Workspace is now a mixture of their existing projects and the application's packages, all sitting happily next to each other. This will make any sort of development difficult, as the developer is forced to remember which packages the application uses, and which are other projects. Third, each package is expected to be in its own source repository (for easy `go get`-ing), further complicating application development.

For Slartibartfast I decided to give this project its own Workspace, with its own `$GOPATH`, and put the entire Workspace into git. You can see this on github at [jasonroelofs/slartibarfast](https://github.com/jasonroelofs/slartibartfast). This way all development, which often happens across multiple packages, happens together and there is no misunderstanding of what code is Slartibartfast and what code isn't.

### 3rd Party Dependencies

Putting the entire Workspace in source control has its own issues though, the biggest being managing dependencies. `go get`, Go's built-in package manager, is really nothing more than a source repository front-end to git, mercurial, svn, and some others. Running `go get` for a package clones the repository at its current HEAD/master/trunk, with no way of specifying a version or commit you might want. Thus even in the normal case it is currently impossible to know if another developer can get the exact same code of your library and its dependencies.

When the entire Workspace is itself a repository, it gets worse. When `go get` pulls a git repository into another git repository, git recognizes that this new repository is a "subproject" and ignores it, expecting manual handling of the directory by the developer. Now, not only do other developers who pull your code not have a full project, but they can not even use `go get -u` to update the packages for themselves, and thus the whole repository is basically broken.

`go get` probably works great for the needs inside of Google; it is a simple solution to a complex problem. While I agree with the arguments that versioned dependencies often lead to version hell, for `go get` to be relevant there has to at least be some way to mark a repository at a given point so we can tell Go that "this is good code, you want this stuff and nothing else". In the end, the only solution I could get working at all was to use [git submodules](https://github.com/jasonroelofs/slartibartfast/blob/master/.gitmodules) and reject `go get` altogether.

### Non Code Resources

The last major issue that an application-only Go Workspace solved for me was handling non-code resources. For a game like Slartibartfast, this includes 3d models, textures, and graphics shaders. There is no place to put such resources in the normal Go Workspace, as anything placed in the `src/[package]` directory gets compiled into the resulting binary and is not accessible or editable from outside. Having my full Workspace as its own repo lets me create new directories to hold these files. Fortunately it turns out that a go executable will use `$GOPATH` when looking for files on the file system, and not finding that that will set `$GOPATH` to its own current location, allowing these resources to live along side any binaries I build and allowing simple packaging.

So that is how I am currently developing Slartibarfast: the full Workspace goes into git and dependencies are handled by git submodules. This ensures I always have a repository anyone can clone from github and receive the exact same code as I currently have.

I would love to hear from anyone else who has had to deal with these issues and what they have done to ensure good development practices. I doubt what I am doing is ideal, and I feel there is a lot to be discussed yet in developing Go applications.































