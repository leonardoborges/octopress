---
layout: post
title: "CUFP/ICFP 2013"
date: 2013-09-25 13:17
comments: true
categories: [clojure, functional-programming, haskell]
---

I'm sitting in the Lobby of the Hilton in Boston and since my flight back to Australia isn't for a few hours I thought I'd write my experience report while everything is still fresh in my mind.

{% img left /assets/images/posts/cufp2013-badge.jpg 318 400 %}

[CUFP - Commercial Users of Functional Programming -](http://cufp.org/) is a Workshop-like conference targeting the practically-minded functional programming community.

As it's stated on their website, *"The CUFP workshop is a place where people can see how others are using functional programming to solve real world problems […]"*.

One of the things that make the event special is that it runs together with [ICFP - International Conference on Functional Programming](http://icfpconference.org/icfp2013/) - which is an event on the far opposite side of the spectrum with language designers, professors, compiler implementors getting together and thinking about the future of their languages and fields. The diversity of the event is astonishing. 

CUFP itself runs for three days and is divided into a traditional conference format day with several talks and two tutorial days. 

I was there for all three days and on the last one I delivered my own tutorial about writing macros in Clojure - more on that later.

As far as the talks of day 1 go, someone already did a great job of summarising them. I highly recommend [you go read it](http://www.syslog.cl.cam.ac.uk/2013/09/22/liveblogging-cufp-2013/).

### Day 2

In total there were 9 tutorials being offered - two of which made up a two-day Haskell tutorial. That's the one I decided to attend.

In the instructions, the instructors mentioned that we could use an online Haskell IDE to follow the course should we choose not to install the Haskell platform on our laptops.

I decided to give it a go. The tool is called FP Haskell Center and has been developed by the awesome guys at [FP Complete](https://www.fpcomplete.com).

It's important to note this is an online IDE - but the editor isn't the only thing being offered though. The Haskell Centre offers a complete deployment solution as well - though I didn't have the chance to play with it.

Back to the tutorial, I used the FP Haskell Centre for day one and it worked great as far as online IDEs go. Compilation and inspecting types do suffer from the round trip over the web and by the end of the day I was feeling a little frustrated with all the waiting. The tool is great and if they offered an offline version, I'm sure the experience would have been improved tenfold. 

Day one was taught by [Andres Löh](http://www.well-typed.com/people/andres) from [Well-Typed](http://www.well-typed.com/), a Haskell consultancy. 

It was full of exercises in the various basics of Haskell such as expressions, functions, IO, pattern matching and even Monads. I had a lot of fun working through them and it reinforced my opinion about Haskell being as practical a language as any other - but with several advantages.

### Day 3

Given my frustration with FP Haskell Center being a bit slow I decided to install the Haskell platform on my new laptop and configure emacs with haskell-mode. I was *much* happier with this setup. haskell-mode has a lot of nifty features that were extremely useful during the tutorial.

The second day of the Haskell tutorial gave way to [Simon Marlow](http://community.haskell.org/~simonmar/), a software engineer from Facebook UK and author of  [Parallel and Concurrent Programming in Haskell](http://community.haskell.org/~simonmar/pcph/) - also available freely [online](http://chimera.labs.oreilly.com/books/1230000000929/index.html).

Not surprisingly, his half of the tutorial was about Concurrency. We started with several exercises on IO involving more Monadic functions that we hadn't learned the previous day. We then moved on to study the basic Haskell concurrency constructs and primitives. All very interesting stuff.

If you have the chance to attend a tutorial by these guys, do yourself a favour and go for it.

### Day 3 - 2pm onwards

Unfortunately I missed the second half of the second day as I, too, had to deliver my own tutorial/workshop.

Titled *"Bending Clojure to your will: Macros and Domain Specific Languages"*, the tutorial had participants work their way through several difference exercises aimed at teaching the various nuances of writing macros.

The tutorial has failing tests for all exercises so it's dead easy to know when you have arrived at a solution - all participants seemed to have had a great time learning this stuff and I even saw a couple of positive tweets about it - I'm really happy with how everything went.

The best tweet though was the one saying that someone from the Haskell tutorial decided to switch to mine half-way through. So I see that as a win :) [Here's said tweet](https://twitter.com/prasincs/status/382585215694413824).

I've [pushed the exercises to github](https://github.com/leonardoborges/clojure-macros-workshop) should you want to try them by yourself. [Slides are also available](http://www.slideshare.net/borgesleonardo/clojure-macros-workshop-lambdajam-2013-cufp-2013) though they will likely not make sense out of context.

Enjoy! :)