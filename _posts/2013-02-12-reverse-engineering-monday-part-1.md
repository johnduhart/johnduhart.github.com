---
layout: post
title: "Reverse Engineering Monday (Part 1)"
category: reverse
---

Inspired by [this thread](http://facepunch.com/showthread.php?t=1166226) in the Facepunch programming forum, I thought I'd try my hand in reverse engineering some kid's shitty key logger.

Since running malicious programs on my main PC would be a really stupid idea, I set up a virtual machine running Windows 7 and loaded it with:

* [JetBraind dotPeek](http://www.jetbrains.com/decompiler/)
* [Telerik JustDecompile](http://www.telerik.com/products/decompiler.aspx)
* Sysinternals' <a href="http://technet.microsoft.com/en-us/sysinternals/bb896653.aspx">Process Explorer</a> and <a href="http://technet.microsoft.com/en-us/sysinternals/bb896645.aspx">Process Monitor</a>
* [Wireshark](http://www.wireshark.org/)
* Some hex editor
* [Visual Studio 2012 Express](http://www.microsoft.com/visualstudio/eng/products/visual-studio-express-products)
* Sublime Text 2

Also make sure to take snapshots or whatever of your VM before you start.

With all of that out of the way, let's get to the good stuff!

### Finding a victim
<img src="/images/2013-02-12/victims.png" />

Finding one of these trojans on YouTube isn't hard at all. The hard part about it is finding one that doesn't make you fill out a fucking survey to download or get the password to the zip file. Tricking people into downloading malware AND profiting from it? That's pretty harsh man.

After some searching I managed to [find one](http://www.youtube.com/watch?v=4HQ80W-GmI8) that doesn't require filling out a "survey" to download.

<img src="/images/2013-02-12/dontLeakThis.png" class="center" />

Don't leak this you guys!!!

<img src="/images/2013-02-12/sadChrome.png" class="center" />

Even Google Chrome is not too happy about this file. I'm sure if I was running an actual anti-virus in the virtual machine it would be going off aswell.

### Analysis
For this part, we're going to break rule #1 (don't run malware you idiot) for science. Discovering what the program does be beneficial to ultimately reverse engineering it. To do that we're going to open up Process Monitor to log what the application does.

<img src="/images/2013-02-12/filter.png" class="center" />

Before we go any further I'll say this one last time. All of this is running inside of a closed environment. Running malicious code on your main PC or even a PC connected to your home network is a very bad idea. Don't fucking do that. You will regret it.

And now, finally, it's time to run it! Double-click it and...nothing?

Huh? Sorry about that guys this must be abad application, let's go look for another one.

<img src="/images/2013-02-12/stillRunning.png" class="center" />

Haha, just kidding. The application is still running in the background and has a whole slew of events in process monitor. [I've attached the process monitor log file if you'd like to follow along](/assets/reverseeng1/Logfile.PML). Let's take a look to see if we find anything of interest, shall we?

<img src="/images/2013-02-12/dotNetLoading.png" class="center" />

Okay, it's reading .NET libaries. Must be a .NET app.

<a href="/images/2013-02-12/readFile.png"><img src="/images/2013-02-12/readFile.png" class="center" /></a>

It's reading itself after loading? There must be something tacked on to the .exe

<a href="/images/2013-02-12/childSpawn.png"><img src="/images/2013-02-12/childSpawn.png" class="center" /></a>

This one is a bit interesting. I noticed this happening during execution in Process Explorer, the intial process spawns a another process of itself, and then this original one dies shortly after. Not sure what is gained by doing that.

<a href="/images/2013-02-12/defenseless.png"><img src="/images/2013-02-12/defenseless.png" class="center" /></a>

This part is my favorite. If you look closely you can see it replacing windows defender with itself and adding it to the startup list. Nice.

<a href="/images/2013-02-12/cookies.png"><img src="/images/2013-02-12/cookies.png" class="center" /></a>

Stealing cookies (from the cookie jar).

Other interesting things found in this log:
* A (failed) attempt to write to the hosts file
* Windows version and computer name fetched from the registry
* A large amount of thread creations and exits

But most importantly

<a href="/images/2013-02-12/phoneHome.png"><img src="/images/2013-02-12/phoneHome.png" class="center" /></a>

We now know how it phones home. Great. Where exactly is that data going? Well,
* __173.194.75.109__ belongs to Google
* __Port 587__ is used for encrypted SMTP

I betcha can't figure out what that means! (It's sending email to gmail ya dingus)

One last thing,

<a href="/images/2013-02-12/oops.png"><img src="/images/2013-02-12/oops.png" class="center" /></a>

If you're writing malware, make sure it doesn't error like this. To keep it fair I'm going to ignore this message box although it does give me some good clues.

### That's all for now
I'm going to be splitting this series up into separate posts so I can release it quicker. In the next part we'll actually get into reverse engineering and less detective work like in this article.

Please let me know what you think of my posts so far.