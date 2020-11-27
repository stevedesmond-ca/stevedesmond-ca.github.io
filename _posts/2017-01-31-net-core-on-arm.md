---
title: .NET Core on ARM
description: Announcing nightly builds of the .NET Core Runtime for Ubuntu 16.04 on ARM!
date: 2017-01-31
comments: https://twitter.com/stevedesmond_ca/status/826515547504381957
---

**TL;DR**: Nightly builds of the .NET Core Runtime for Ubuntu 16.04 on ARM are available [here](https://github.com/stevedesmond-ca/dotnet-arm/releases/). These are "Hello World" tested on my Raspberry Pi 2 and Chromebook. Nightly builds of the SDK are expected in the next week or so.

Also, should probably have a disclaimer: these are unofficial builds using the master branch of the dotnet code repositories. This is about as rare / raw / blue as you can get!

Update: official nightlies are available [here](https://github.com/dotnet/core-setup#daily-builds)!

---

Well this one has been a long time coming! Check the commit history and you'll see I've been working on this on-and-off in my spare time since September!

I first discovered the need for .NET Core on ARM when buying a new laptop. I found that we had 2 x64 devices in the house (one desktop, one laptop), and 6 ARM devices (3 phones, 2 tablets, 1 Chromebook). If the vision of ["Any Developer, Any App, Any Platform"](https://developer.microsoft.com/en-us/) is to be realized, it must include ARM.

### Wait, what's an ARM?

Similar to how we have different operating systems, there are [many types of CPU architectures](https://en.wikipedia.org/wiki/List_of_CPU_architectures). These different architectures have different instruction sets, so just like you can't run a Windows app natively on OS X, you can't necessarily run code from one architecture natively on another.

Most desktops, laptops, and servers run on something called x86-64 (or "x64" for short): the 64-bit version of x86. x86 has been around pretty much since the dawn of personal computing.

ARM, on the other hand, is much newer, focusing on mobile and embedded systems, and therefore prioritizing cost, physical footprint, and power efficiency over raw performance.

There's an almost 100% chance your smartphone runs on an ARM processor. Other well-known devices include the Raspberry Pi (and most other IoT T's) and many newer entry-level laptops (formerly called "netbooks"), like my Chromebook. Current estimates say there are around 20 billion active ARM devices worldwide. Twenty billion.

### OK, what does this all mean?

<figure>
<img src="/assets/blog/dotnet_arm_chromebook_pi.jpg" alt="Chromebook"/>
<figcaption>.NET Core running on both my Chromebook and Raspberry Pi</figcaption>
</figure>

In short, you can download the .NET Core Runtime [here](https://github.com/stevedesmond-ca/dotnet-arm/releases/), install it on your Raspberry Pi or Chromebook or any other ARM device on Ubuntu 16.04, and run your .NET Core apps with dotnet <app>.dll!

Once an ARM SDK is available (again, hopefully in the next week or so), and one can run Visual Studio Code and build .NET Core apps on ARM hardware, this significantly decreases the barrier to entry for new .NET developers worldwide, especially in developing markets where low-end devices are much more mainstream.

### Some background

Way back when DNX was still a thing, DNX ran fine on ARM, but during the "dotnet" platform consolidation, this was something that was not so easily accomplished. After the 1.0 release, when I started seeing Rich Lander mentioning in responses to various questions that it was unlikely to make it into 1.1, I decided to take things into my own hands.

**Ha! Easier said than done.**

Many compilers, unit test frameworks, etc. use their own tool to build/test their tool -- dogfooding inception, if you will -- and likewise, .NET Core uses .NET Core to build .NET Core. But if .NET Core doesn't exist on ARM, how does one build .NET Core on ARM?

This was the first major roadblock I hit, and while the answer is "cross-compiling," neither I nor the codebases (yes, plural) were prepared for that.

### .NET Core Components

It turns out there are many different components that make up .NET Core. I've tried to lay this out, slightly simplified, below:

![.NET SDK dependencies](/assets/blog/dotnet-dependencies.png)

Let's start at the right and left sides: the Runtime is what allows you to type dotnet <app>.dll to run your app, whereas the SDK provides the developer workflow functionality for dotnet new, dotnet restore, dotnet build, dotnet run, dotnet publish, etc.

The runtime is made up of:

- **CoreCLR**: the Common Language Runtime for .NET Core, takes the IL generated by the compiler and translates it into code your CPU can understand
- **CoreFX**: the .NET Core Framework, containing pretty much everything in the System. namespace. This is the part that is growing massively between .NET Core (and Standard) 1.x and 2.x
- **Roslyn**: the .NET compiler platform
- **libuv**: the async I/O library that Kestrel (ASP.NET Core's server) runs

The others:

- **core-setup**: build tools to pull in all of the above and combine it into the .NET Core Runtime
- **CLI**: the .NET Command Line Interface, so you can dotnet whatever as part of the SDK

As you can see from the diagram, the .NET Core SDK is basically the .NET Core Runtime + CLI.

### Cross-Compiling

By using CPU emulation -- yes, the same thing that lets you play classic video games on modern hardware -- we can run applications for other CPU architectures. But even without emulation, just because one architecture can't run another's code doesn't mean it can't know what that code should be. The process of compiling code for a different target (architecture, platform) other than the one it is being built on is called cross-compilation.

Side note: remember how I mentioned performance was not ARM's strong point? Compiling CoreCLR on a Raspberry Pi takes about an hour. By cross-compiling it for ARM on my x64 desktop (i7-4770K, 32GB RAM), it builds in about 3 minutes. That's 20x faster!

As more of the .NET Core components became cross-compile-able, the dream of .NET Core on ARM slowly became more believable, but it really wasn't until the last month or so that it was able become a reality.

Which brings me to the following:

### Credits

One thing I want to do is put the spotlight on everyone who has been working on getting .NET Core running on ARM. I think that unless you're knee-deep in the development of a big feature like this, it's hard to recognize how much effort goes into something so fundamental. While I've largely been on the sidelines -- I've only submitted a few ARM-related PRs -- others have toiled tirelessly over the past few months on getting official support up and running.

Obviously, Microsoft has many teams working on different aspects of .NETCore. Here are those who have helped with ARM PRs:

- Andy Ayers
- Senthil Chellappan
- Bruce Forstall
- Davis Goodin
- Pawel Kadluczka
- Russ Keldorph
- Guarav Khanna
- Jan Kotas
- Rahul Kumar
- Lubomir Litchev
- Peter Marcu
- Mike McLaughlin
- Matt Mitchell
- Geoff Norton
- Lakshmi Priya Sekar
- Jarret Shook
- Eric St. John
- Swaroop Sridhar
- Brian Sullivan
- Koundinya Veluri
- Jan Vorlicek

As mentioned during Connect this year, Samsung is working on having .NET
Core be a target for their Tizen-based smart TV apps. These are all the
people at Samsung who have ARM-related PRs merged:

-   Prajwal Aithal
-   Ivan Baravy
-   Dmitri Botcharnikov
-   Hyung-Kyu Choi
-   Dongyun Jin
-   MyungJoo Ham
-   Sujin Kim
-   Andrey Kvochko
-   Hanjoung Lee
-   Geunsik Lim
-   Viacheslav Nikolaev
-   Hyeongseok Oh
-   Jonghyun Park
-   SaeHie Park
-   Evgeny Pavlov
-   Jiyoung Giuliana Yun *

\* I specifically want to call out Jiyoung Giuliana Yun here, who laid out [the process she was using](https://github.com/dotnet/core-setup/issues/725#issuecomment-273677348) for her own builds, which ended up being the base for turning my scripts from a hacky mess into something more resilient.

Of course, I would be remiss if I did not mention the "volunteer" community. I was hoping that there would be more of this type of participation on the ARM side of things, given how spoiled we seem to be in ASP.NET Core land, but still some very important mentions:

- **Frederik Carlier, of Quamotion in Belgium**, was the one that got libuv (the underlying library of Kestrel, ASP.NET Core's server) ARM CI builds working
- **Ben Pye, CS student at University of Bristol**, (last but not certainly not least) was instrumental in getting CoreCLR up and running on ARM from early 2015 through early 2016. I'm positive I wouldn't be writing this blog post right now if it weren't for his early efforts.

I apologize if I've unintentionally left anyone out. Thank you for your great work!

One benefit of software being developed as open-source (aside from being able to mutually benefit off each others' work) is that it allows us to see the effort that goes into the tools we use every day. It truly does take a village...