---
title: "Developing Desktop Applications"
date: 2025-10-10T12:00:00-04:00
draft: true
---

During the pandemic I served as my church's treasurer, and through that experience I got to experience how weird the world of accounting software is. Our church, a non-profit, uses Quickbooks Desktop to handle its books and I wanted to know if there was something better, because boy is Quickbooks a bad experience, so much so that it is my quintessencial example of software that is grown by features and committees. It's excruciatingly hard to get your data out of the system, and we even relied on hand copy-and-pasting from Quickbooks into Google Sheets to make the reports we needed. Also, when a CPA tells you "don't use Quickbooks Online", you know something has gone horribly wrong.

What I found was a myriad of services and applications, all tailored and built around the needs of for-profit corporations, so much so that to make said systems work for my church I would need to first undo so many layers of accounts and pre-built reports, and even then the day-to-day operations of tracking expenses and income was going to be difficult at best. So we stayed with Quickbooks. Entering in the data is easy, getting your data out is almost impossible, but we made it work.

As a software engineer, I couldn't help but think "it cannot be this hard to make good accounting software". At its core, proper accounting is easy: a write-only double-entry ledger. Every transaction consists of at least one debit from an account and at least one credit of another account. To verify the numbers are correct, you add up all debits, add up all credits, and if you get the same number, you've done it right. How hard could it be to build up accounting software based on the double-entry ledger core princple?

I set a couple of requirements for this software. First, it was going to be a desktop application. Using it should not require an internet connection, though I would make sure to build in a backup system and have plans for syncing the database via a cloud service if people want to pay for that. Second, the data is the users and only the users. It would store everything in a local SQLite database and I would publish the schema so anyone could do their own querying. Third, it needed to be cross-platform. I myself regularly use Mac and Windows, so it would need to run on at least those two. Finally, language choice means a lot to me. I don't want to right low-level like Rust or C++, and I want a language that's staticly typed (I've written Ruby for the past twenty years, but, I kind of want static typing now. I'll write more about that in a future article). Also it needs to be packageable into a standalone bundle or executable.

After two years of research, experiments, starts, stops, restarts, and trying so many different options, I think I've finally found the toolkit that checks all of the boxes: [Kotlin Multiplatform](https://kotlinlang.org/docs/multiplatform.html) with [Compose Multiplatform](https://www.jetbrains.com/compose-multiplatform/), though I've been keeping a very close eye on [Slint](https://slint.dev/).

This article is a run down of every technology I've looked at, an explanation of what it provides, why to use it, and why I chose not to, in an effort to help make this decision easier for the next person wanting to make desktop applications in 2025 and after. Because boy this was quite the trip.

[Probably a table of contents]

Before I dive into the options, I want to provide a set of requirements that helped me figure out if this option was right for me. I wanted the following:

## Required

* Supports system menus (desktop applications have File, Edit, etc).
* I don't want to write Rust or C++.
* Has full, or improving, support for accessibility tooling (screen readers).
* Can be packaged into a standalone unit for the platform (e.g. macOS bundle, Windows installer).


## Qt

[Qt](https://www.qt.io) (pronounced "cute" though I always default to saying the letters) is the standard tooling for cross platform desktop applications. It has been around a long time and probably contains most everything you could possibly need. It is written in C++ and is old enough to have it's own entire build system (qmake, though that is finally getting sunset for cmake in Qt 7). Qt also maintains bindings to Python and Javascript, and there's been a myriad of other bindings in various state of maintenance across many languages.

### QML

Qt has historically been an imperative UI library (called [QtWidgets](https://doc.qt.io/qt-6/qtwidgets-index.html)), where every UI element is written out in code, along with layout logic and the like. There is an XML format that is used by QtDesigner but it's not really meant for human use. They also provide a more declarative and reactive form called [QML or QtQuick](https://doc.qt.io/qt-6/qtqml-index.html). QtQuick is implemented as a Javascript framework that gets compiled into a final binary with Qt's own Javascript runtime.

You really can't go wrong with Qt, but they are a business and licenses can be expensive. For open source or personal use, make sure you read and understand their licensing requirements before you go with this framework. There are also people who have stayed with Qt 5 which has more permissive licensing.

## Slint

[Slint](https://slint.dev/) (previously SixtyFPS) was started by a couple of people who used to work at Qt and have otherwise spent a lot of time in the Qt space. Slint is a framework that takes the QML idea and rebuilds it from the ground up as it's own system and language with the ability to use different rendering backends (which defaults to Qt) with an initial focus on the embedded space. They support writing applications in Rust, C++, Python, and Javascript all powered by the Slint language frontend and tooling. There is an increasing focus on Desktop support.

## Electron

If you want to use web technologies (Javascript, CSS, HTML) to build your desktop application, [Electron](https://www.electronjs.org/) is the 800 pound gorilla. Electron powers a ton of very popular software and while there are regular complaints from more tech-focused circles about its resource usage, most users don't care about that. From VS Code, Spotify, Slack, and Discord, some of the most popular software in the world today is running on Electron. By bundling Chromium into the application itself you are guarenteed a consistent look and feel regardless of the platform, and by using web technology you get to use an enormous ecosystem of tools and libraries for whatever you could possibly want.

## Wails

With all three major operating systems now providing browser rendering natively, there is an increasing number of libraries that want to provide the Electron experience without the bloat. [Wails](https://wails.io/) is one such attempt, letting you build your backend in [Go](https://go.dev/). I actually started implementing my application in Wails and can highly recommend it. It has a great setup for interacting between the web frontend and Go backend and spends a lot of effort on properly interacting with the underlying OS (e.g. really good system menu support).

## Tauri

This may not be entirely fair, but [Tauri](https://tauri.app/) is like Wails, but in Rust (Tauri started first). Tauri is a much bigger project than Wails and its use is definitely gaining momentum. If you like Rust and you want an application using web technologies for the front-end, Tauri is the way to go. Also, the Tauri project has modularized a lot of their core logic to the point where other similar libraries actually use Tauri tools e.g. [wry](https://github.com/tauri-apps/wry) and [tao](https://github.com/tauri-apps/tao). Providing better abstractions on top of the sometimes arcane and esoteric OS windowing APIs is a benefit to everyone.

## .NET, Avalonia, Uno Platform

I'm admittedly clustering a lot of possible solutions into this one bucket but that is because it is an ecosystem that I have effectively zero experience with. Microsoft itself has released so many UI libraries, WPF, WinUI, WinUI2, WinUI3, MAUI, though from what I've read at this point it's best to stay far away from Microsoft official libraries because it will get replace with the next incomplete, buggy mess in three years time.

That said, if you already are well versed in the .NET ecosystem, especially if you are already familiar with WPF and XAML, then Avalonia or Uno are great choices. Coming from the outside, and working on a Mac, I found them too difficult to get started with, and for both frameworks even following the provided tutorials gave me an app that didn't even build, and I had no idea where to go from there.

## Flutter

[Flutter](https://flutter.dev/) is Google's attempt to solve this problem, going even as far as creating a whole programming language first for it ([Dart](https://dart.dev/)). Flutter has mainly been focused on mobile support (iOS and Android) and only recently has pushed harder into supporting the Desktop environment. Flutter does all its own rendering, giving it full control over look-and-feel, but also needing to implement everything itself.

## React Native

## Fyne, and others

## Godot, Unity, Unreal, and other game engines

