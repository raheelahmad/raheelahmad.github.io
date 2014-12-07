---
layout: post
title: The case against Cocoapods and AFNetworking
---

In the past few years, the iOS/Mac open source community has gone from a few developers maintaining a few libraries, to a thriving community [representing](https://github.com/trending) Objective-C (and now Swift) prominently on GitHub. [CocoaPods](http://cocoapods.org) has supported the community with the ease-of-integration for libraries that Apple left out of its platform.

However, the adoption of CocoaPods and 3rd party libraries, has not seen a balanced approach that I believe that any new (or old) technology deserves, specially on long-term projects. My thoughts below are issues that I do not hear being discussed often enough in the community.

---

## Is it worth it?

**Managing another dependancy.** Including a new tool in your development workflow add its own dependency.  As such, you should balance its worth with the tool's failures: how painful they are, how long  it takes to fix them, and do they block your development process, your build process, or something else.  Half of my CocoaPods problems have been ruby and rubygems issues, which usually left me hoping that `gem install cocoapods` was actually `brew install cocoapods`. Or the fact that CocoaPods may [modify your project file](https://github.com/CocoaPods/CocoaPods/wiki/Generate-ASCII-format-xcodeproj) to a different format on `pod install` (blame Apple here for using an undocument format here, though); a merge conflict in this scenario would have you curl up and cry. You must talk to at least a couple of developers who have used CocoaPods or through the years, and measure if their successes stand up to their ordeals. In the same vein, using a library ties you to all the issues that the library may have, which brings us to being...

**Tied to a specific implementation**. Very often you would like to keep part of a library's approach, but replace others. You would probably create a fork if your approach is fundamentally different from the author's approach, but it does not warrant replacing the library just yet. However, syncing forks is often a losing battle, as the impedance and velocity mismatch is usually too much to cope with. If you would be making contributions to the main repository (and you most probably will be, if your project lasts longer than a year), consider how robust the implementation is, how collaborative/combative their pull request policy is, how well does the library keep up with the iOS releases, how well are the difficult bugs being tracked (I have noticed some libraries lose focus of edge-case bugs, which might be very importatnt to you), and anything else that would matter to you in the long run.

**Be left out of Apple's innovation.** For example, with CocoaPods, the support for Swift is still incoming, as it is a particularly challenging problem. Another example is the support for Xcode Bots, which took a long time (I believe it is functional now). You might say that you wouldn't want to use these features in your measured reliance on CocoaPods, but ideally you would want to keep that choice to yourself, as the sole platform vendor makes major changes every year. A similar case can be made against most libraries.

**You get the kitchen sink.** This is really what I believe to be the most important issue, that could use a more balanced approach than what I often see. Let's take AFNetworking: most people need a small subset of what this sizable library provides. And, it is trivially simple to write a basic HTTP client! I am no [John Carmack](http://blog.wilshipley.com/2013/12/my-doom-20th-anniversary-stories.html), but I have written my own networking stack for nearly all my apps, sometimes reusing a previous version, but often writing it from scratch. I would start with a simple client that would make GET requests, write unit tests for it, and call it a day. Until I have to do user authentication, and then need POST requests; so I write a POST request handler. PUT often comes in close behind (editing the user profile). But even at this stage, I hold these advantages:
	- my (tested) client code is consolidated in 3 - 4 small and simple classes that any team mate could dig in to and modify. It has probably taken me 2 to 3 hours to write. 
	- writing this client has given me a great appreciation for networking. I am always looking for opportunities to delve deeper into the foundations of computing, as time permits.
	- it serves as a code kata for me, as I refine my previous approach
	- adding features such as canceling/revoking requests, caching control, and counting requests in flight, are all fairly trivial. And, for any sophisticated and long-lived app, at least one developer on the team should be intimately aware of this implementation.
	- growing code organically, with just the right bits, is an [important design principle](https://www.vitsoe.com/gb/about/good-design). There is a [concept in Urdu poetry](http://publishing.cdlib.org/ucpressebooks/view?docId=ft10000326) called *rabt* that is related to Dieter Rams' last principle: to have a couplet be written with such care, that there is nothing superfluous left in its two lines — even if it is intricately complicated or delightful in its simplicity. I strongly believe that our code should capture this principle.
	- If I need to add, modify, or remove a feature in my networking code, I know exactly where to go. Pull requests can be argued and accepted on my team's schedule. 
	
Adding AFNetworking in situations like this would be like using Microsoft Word when TextEdit would suffice.
	
**An abstraction on top of an abstraction.** Let's take [MagicalRecord](https://github.com/magicalpanda/MagicalRecord) as an example: Core Data is the Apple framework everyone loves and hates. MR provides a lot of helpers around Core Data: for fetching and manipulating objects, setting up the Core Data stack, easing the use of its concurrency idioms, etc. It is a well-written and well-supported library that is inteded to ease you in to the murky world of Core Data.I would argue that Core Data being the giant abstraction that it is, should make you want to avoid another giant abstraction on top of it. For e.g., very soon after you start using the MR multi-threaded support, you would do something stupid. Your resolution would be to find the right way to use MR, but also the right way to use Core Data, and *why*. Similarly when you have to manipulate the fine details in the stack setup: importing, resetting, migration, multi-threading set up, multi-store set up, optimizations, and so on. MR might support all of your edge cases, but by then your experience would convince you to replace MR with your own lean, well-understood stack setup and helper code. Your time will be well spent understanding how Core Data works intimately. The framework is responsible for managing your data, and ultimately a bug in your model layer is the worst bug of all.

A different example of hiding abstractions is the behavior of [mogenerator](https://github.com/rentzsch/mogenerator) (its core feature, really), where it puts all your dynamic properties in a super class. If there is one thing I want to look at in my model class, it is its properties. With mogenerator, now I have to look at `_Project` for the dynamic properties, and `Project` for my synthesized properties and methods. Balance this constant, day-to-day friction in your code reading and writing, against the primary friction it relieves you of: not having to write down the properties that you add in your `xcdatamodel` in your `.h, .m` (well, only one file/one line with Swift now). I have never had much issue with simply adding these requisite properties in manually every time I add them to the model. You get a helpful crash in the next 5 minutes if you don't do it right. But, it is the same crash if you forget to run mogenerator on the command line after making the model changes. *(I miss Xcode 4's menu command that would allow you to copy/paste these attribute from your model to your implementation files)*.

Another example is Cocoapods hiding from you the build process between your app and the pods. It's a process that a developer on a long-term project will do well to learn. 

Indeed, these are open source projects, so they are not necessarily hiding details from you (at least the Magical Record and Cocoapods examples). However, looking up the internals of a significantly sized library brings the overhead we discussed earlier.

**Do we really need it?** The reason why other languages have a library manager is because they don't have Cocoa. Rarely does a language have such a mature set of frameworks, that spans the spectrum of what a developer needs. (For example, try looking for a canonical, system-provided approach to parsing a JSON string to native objects in .NET.)  So, in general the need for having a thriving ecosystem of libraries and a strong community of contributors around them should be reduced in Cocoa. On the other hand, iOS and Mac open source libraries rarely rise to the complexity of libraries in other languages (for e.g., say data science libraries in Python). There are notable exceptions, as we will see later, but this gives us more context when comparing platforms and their 3rd-party library ecosystems.

Sugar libraries, which provide nice little helpers for onerous tasks, probably rank the lowest in my needs. Histroically, these are the ones that often go stale, are superseded by a Cocoa framework, hide abstractions for the least important reason, or can (and probably should) be the most easily replicated in your own style. This holds not just for Cocoa, but for all platforms. Elsewhere, there are excpetions to this, where the sugar gives you a comprehensive and sane layer over a pathological base implementation (see [CoffeeScript](http://coffeescript.org)).

## The case for using 3-rd party libraries

There are some of the scenarios where using a library would be worth the trade offs above:

**Using small libaries.** All of my arguments are void when the library in question is a well-written, useful, and a *small* one. The one reason I would use AFNetworking is for its support for multi-part uploads. This is a non-trivial task, and it is difficult to get all the edge cases covered reliably. However, instead of the kitchen sink approach, I would rather use a library that does this one task well (possibly using NSURLSession internally and providing hooks for customizing behavior). The UNIX approach of writing and using modular binaries has advantages that have been amply covered elsewhere. *Related*: you should take the same approach and split up your own project code base into modules (both figuratively, and the Xcode kind). This has much win, including speeding up your Swift project compile times.

**Small projects and prototypes.** I feel very strongly for the above rationale on long-term projects — those you would be working on for more than a year and a half. If your intention is to work on a very small project or a prototype, using a library might be worth the speed gains. For e.g., using a library to do your JSON to native objects mapping, as this might not be something you want to hand craft on a prototype project.

**Specialized libraries.** A library can be tremendously useful when it caters to a specialized need that Cocoa does not cover. A few examples: ReactiveCocoa, a graphing library, Crypto, and CLI option parsing.

## Post Script. Contribution to the open source community.

The biggest challenge writing this was my guilt of making light of the work of those who work on Cocoapods, and the libraries mentioned. I have learnt a lot from the code that is available thanks to these folks, and their work has undoubtedly done a service to the Cocoa community.

Also, if you want to contribute to the Cocoa community but hold the same opinion as me, there are always avenues besides Cocoapods and the libraries I mentioned above:

- solve a tough and unsolved problem
	- a Core Audio wrapper
	- support for a natural language's script that is not yet supported by Apple
	- a library for prosidy recognition

Also, the best community support you can provide is to teach people. You can teach not just how to write solutions, but how to write good solutions. For a lot of us, the struggle is not how to use Cocoa, but how to write code that scales with complexity. And, when others learn from your approach, they can take the work that they like, and leave out the bits they don't.
	
## Post Post Script. Carthage as another option

[Carthage](https://github.com/Carthage/Carthage) is an alternative to Cocoapods, that may be better suited to your needs if you are targetting iOS 8 onwards:

- no git submodules, but git sources are still specified
- versioning (tags-based)
- easy access to the built frameworks, instead of hunting in DerivedData
- you keep your project file organization

Also, consider frameworks—what Carthage uses—as the modern way to distribute code. It is also useful for your own project's code architecture.

*Please consider these arguments below as food for thought, and not a source for having an argument with me, or a Cocoapods lover. Even if YMMV, rarely would understanding well made arguments against your beliefs *not* make you better suited for your job. Also, critical understanding of contrarion approaches and viewpoints that one may never agree with, are a hallmark of an excellent community (Neil Postman [makes a better case](https://w2.eff.org/Net_culture/Criticisms/informing_ourselves_to_death.paper) for this than I possibly can).*
