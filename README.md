# Optimizing Swift build times
Collection of advice on optimizing compile times of Swift projects.

Swift is constantly improving. In its current state (Swift 4), though, long compile times are one of the biggest hindrance when working on medium-to-large apps. The goal of this project is to gather tips and tricks that will help you shorten build times in one place.

👷🏻 Maintainer: [Arek Holko](https://twitter.com/arekholko)]. Contributions always welcomed through issues and pull requests.
## Table of contents
TODO: add links
## Showing build times in Xcode
First, to be able to actually know whether your build times are improving, you should enable showing them in Xcode’s UI. To do that, run this from the command line:

```
$ defaults write com.apple.dt.Xcode ShowBuildOperationDuration -bool YES
```

Once done, after building a project (cmd-B) you should see:
[image:87401C97-3EBF-4B53-916B-C50BFFCE2D3D-41531-0000D5A04F107FE4/Untitled.png]

I recommend comparing build times under same conditions each time, e.g.
1. Quit Xcode.
2. Clear Derived Data (`$ rm -rf ~/Library/Developer/Xcode/DerivedData`).
3. Open your project in Xcode.
4. Start a build either immediately after Xcode opens or after indexing phase completes. First approach seems to be more representative because starting with Xcode 9 building also performs indexing.

📖 Sources:
- [How to enable build timing in Xcode? - Stack Overflow](https://stackoverflow.com/a/2801156/1990236)

## Functions and expressions
Swift build times are slow mostly because of expensive type checking. 

📖 Sources:
- [Guarding Against Long Compiles](http://khanlou.com/2016/12/guarding-against-long-compiles/)
- [Measuring Swift compile times in Xcode 9 · Jesse Squires](https://www.jessesquires.com/blog/measuring-compile-times-xcode9/)

## Slowly compiling files
Previous section described TODO

📖 Sources:
* [Diving into Swift compiler performance](https://koke.me/2017/03/24/diving-into-swift-compiler-performance/)


## Third-party dependencies
There are two ways you can embed third-party dependencies in your projects:
1. as a source that gets compiled each time you perform a clean build of your project (examples: CocoaPods, git submodules, copy-pasted code, internal libraries in subprojects that the app target depends on)
2. as a prebuilt framework/library (examples: Carthage, static library distributed by a vendor that doesn’t want to provide the source code)

CocoaPods being the most popular [dependency manager](https://twitter.com/arekholko/status/923989580948402177) for iOS by design leads to longer compile times, as the source code of 3rd-party libraries gets compiled each time you perform a clean build. In general you shouldn’t have to do that often but in reality you do (e.g. because of switching branches, Xcode bugs, etc.).

Carthage, even though it’s harder to use, is a better choice if you care about build times. You build frameworks only when you change something in the dependency list (add a new framework, update a framework to a newer version, etc.). That may take 5 or 15 minutes to complete but you do it a lot less often than building code embedded with CocoaPods.

📖 Sources:
- time spent waiting for Xcode to finish builds 😅

## Build only what you need
TODO: About scheme settings and many schemes overall

📖 Sources:

## Whole Module Optimization
TODO: no warnings about modified files during building

📖 Sources:

## Modularization
Incremental compilation in Swift isn’t perfect. I know about projects where changing one string somewhere causes almost a whole project to be recompiled during an incremental build. It’s something that can be debugged and improved but a good tooling for that isn’t available.

To avoid issues like that, you should consider splitting your app into modules. In iOS these are either: dynamic frameworks or static libraries (support for Swift was added in Xcode 9).

Let’s say your app target depends on an internal framework called `DatabaseKit`. The main guarantee of this approach is that when you change something in your app project, `DatabaseKit` **won’t** get recompiled during an incremental build.

📖 Sources:
- [Technical Note TN2435 – Embedding Frameworks In An App](https://developer.apple.com/library/content/technotes/tn2435/_index.html)
- [uFeatures](https://github.com/microfeatures/guidelines)

## XIBs
XIBs/storyboards vs. code. 🔥 It’s as hot a topic as they go but let’s not discuss it fully here. What’s interesting is that when you change the contents of a file in Interface Builder, only that file gets compiled to NIB format (with `ibtool`). On the other hand, Swift compiler may decide to recompile bigger part of your project e.g. when you change a single line in a public method in a `UIView`. 

📖 Sources:
- [(…) in a large project incremental build is much faster if only a .xib was changed (vs. only a line of Swift UI code)](https://twitter.com/MichalCiuba/status/925326831074643968)
