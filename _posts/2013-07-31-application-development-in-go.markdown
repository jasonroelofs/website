---
layout: post
title: Application Development in Go
published: true
tags:
- software
- slartibartfast
- golang
---

Getting started with Go is surprisingly easy. Not only is the [official docs](http://golang.org/doc/) detailed and comprehensive, but the Go team has also put together [A Tour of Go](http://tour.golang.org/#1), an entire set of tutorials you can run right in the browser. Go's syntax is simple and should be very familiar to anyone who's already comfortable in any C-esque language. Then, for writing quick scripts, Go comes with the `go run` command, which compiles and runs a single go file as you would any "scripting" language like Ruby, Python or Perl.

When you're ready to start putting together libraries and more complicated command line tools, [How To Write Go Code](http://golang.org/doc/code.html) is ready and waiting (seriously, please familiarize yourself with everything in this document, it's how Go wants to work). And from there, [Effective Go](http://golang.org/doc/effective_go.html) is full of hints, tips, and suggestions on how to write good, readable Go code.

Now I should probably take a step back. Go is a strongly opinionated language, and by proxy so is the community. There's only one way to write Go code. Brackets go on the same line, `if` always needs brackets, if you're handling one return value, you must handle them all, and so on. Go code doesn't even compile if these rules are not followed. Even unused package imports and unused local variables are compiler errors. So from this, if you're writing libraries and command line tools and not using an environment as described in [How To Write Go Code](http://golang.org/doc/code.html), you're going to have a bad time. I know I did.

There's one aspect of development though that the Go docs do not cover, and that's any project bigger than a library or a command line tool, what I usually call an Application. Any software where you'll have multiple packages working together, probably a number of 3rd party libraries, and often non-source-code files and data. In Go terms, this would be any software that no-one would ever `go get` and use in their own project. For this iteration of [Project Slartibartfast](http://jasonroelofs.com/2013/07/01/third-times-a-charm/) I spent a lot of time reading the docs, playing with various ideas, and discussing my problems on Freenode's #golang IRC channel and came up with the structure I have now. I'm sure others in the Go community will disagree with some or all of my choices here, and I'd love to have more discussion on this problem, as I don't feel it's a solved one yet. With that, here's the aspects of Go development I had issues with and the decision I made.

### $GOPATH and Your Workspace

As per "How To Write Go Code", the $GOPATH must exist and must point to a Go Workspace, which is defined as a directory with `src/`, `pkg/`, and `bin/` directories, and while not explicitly said, it's expected that there's one Go Workspace on the machine, containing all Go code. This doesn't work for application development for a few reasons. First, a developer can't expect anyone to individually `go get` 20-plus different packages (Slartibartfast has 14 and growing), not to mention whatever 3rd party dependencies the application uses, to work on a single application. Secondly is that after all those `go get`s, that developer's Go Workspace is now a quagmire of their existing projects sitting right next to the application's package code, which makes remembering what packages exist for who, and basic development of any package, very difficult. Third, each package is expected to be in its own repository (for easy `go get`-ing), further complicating application development.

For Slartbartfast I decided to give this project its own Workspace, setting up an `environment` file to help me set the `GOPATH` appropriately, and putting everything Slartibartfast needs into git. You can see this on github at [jasonroelofs/slartibarfast](https://github.com/jasonroelofs/slartibartfast).

### 3rd Party Dependencies

Putting the entire Workspace in source control has a number of issues though, the biggest of them being `go get` for any dependencies. `go get` will happily pull from a git, mercurial, or svn repository and drop the checked-out code right in the Workspace as per the expected directory structure. Not a problem when the Workspace is just a directory, but when `go get` pulls a git repository into another git repository (itself), problems abound. Git recognizes that this new repository is a "subproject" and ignores it, expecting manual handling of the directory by the developer. The code is now in a non-reproducible state, and any developer who then checks out the repository on another machine will find an unrunnable project.

While `go get` works great for the needs inside of Google, it is a simple solution to a complex problem, a problem more people are trying to solve, though as of this writing no suggestion has made it past prototyping. As one of the most important aspects of application development is build reliability, I had to look elsewhere for managing dependencies. Unfortunately the only solution that really works for me now is some [git submodules](https://github.com/jasonroelofs/slartibartfast/blob/master/.gitmodules) and rejecting `go get` altogether.

### Non Code Resources

The last major issue that a consolidated Go Workspace solved for me was non code resources. For a game like Slartibartfast, this includes 3d models, textures, and graphics shader programs. There simply is no place to put such resources in the normal Go Workspace, as anything placed in the `src/[package]` directory gets compiled into the resulting binary and is not accessible or editable from outside the program (as is rather important in game development). Having my full Workspace as it's own repo let me create new directories to hold this information. Fortunately it turns out that a compiled binary executable will use `$GOPATH` when looking for files on the file system, and barring that will set `$GOPATH` to it's own current location, allowing these resources to live along side any binaries I build.

So that's how I'm currently developing Slartibarfast: the full Workspace goes into git, I've got helper scripts to manage my packages, extra directories for my non-code resources, and my dependencies are pulled in via git submodules. This collectively gives me a repository anyone can clone from github and receive the exact same state of code as I currently have.

I would love to hear from anyone else who's had to deal with these issues, and what they've done to ensure good development practices. I doubt what I'm doing is ideal, and I feel there's a lot to be discussed yet in developing Go applications.































