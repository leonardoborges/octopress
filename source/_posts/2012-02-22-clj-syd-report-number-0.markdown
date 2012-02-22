---
layout: post
title: "clj-syd report #0"
date: 2012-02-22 21:14
comments: true
categories: [clojure, clj-syd]
---

Last night we held the first ever meetup for the [Sydney Clojure User Group][13]!

When I decided to start running the meetups I had no idea we'd end up with **37**
people on the first night! What a great turn out!

As for the content, here's what you missed:

* **Running Clojure apps on Heroku** - Lincoln Stoll ([@lstoll][5])

Linc works for Heroku but ended up not spending a whole lot of time talking about
that - instead he showed us how  to build a simple web application using
[Compojure][6], a small Clojure web framework.

He then evolved the example by implementing it using [Noir][7] - yet another web
framework that builds on top of [Compojure][6], adding some helpful macros. 

To top it off, the UI was implemented in [ClojureScript][8] - so that was all Clojure
from end to end!

* **Paredit (in Eclipse) for the IDE junkie** - Matt Quail ([@spudbean][3])

If you're into lisps and use emacs - if not you **should** anyway - you learned to
love [paredit][9].

However, not everyone gets to write Clojure for a living and a lot of us end up in
some sort of IDE. If you're in Java land, a popular choice is Eclipse and Matt showed
us how you can get the paredit goodness right there using the Eclipse plugin [counterclockwise][10].

* **[Building a better Clojure REPL][2]** - Cosmin Sterejean ([@offbytwo][4])

Frustrated with the Clojure REPL? Wish you had more useful shortcuts? Decent line
editing? Saving that nice function you've been working on right from the REPL? Then
make sure you check out both the [presentation][2] and [IClojure][11], Cosmin's
project.

Definetly worth a try.

* **Where do I put my DependentStrategyTemplateAbstractFactory?** - Bayan Khalili

Bayan was in one of our internal meetups. Some people raised the fact that we don't
see a lot of the design patterns made popular by the [GoF book][12] in languages such as Ruby, Python or Clojure
and the overall opinion is that in those languages, some of the problems these
patterns solve in, say, Java, simply don't exist.

Bayan decided to dig up a few and show what's the alternative in Clojure. Interesting
perspective.

## Want more? ##

Join us on [Google Groups][13], submit a talk proposal on the [wiki][14] and RSVP to
the [next meetup][15]!

See you next time!



[2]: http://offbytwo.com/presentations/building-better-repl.pdf
[3]: https://twitter.com/#!/spudbean
[4]: https://twitter.com/#!/offbytwo
[5]: https://twitter.com/#!/lstoll
[6]: https://github.com/weavejester/compojure/wiki
[7]: http://webnoir.org/
[8]: https://github.com/clojure/clojurescript
[9]: http://www.emacswiki.org/emacs/ParEdit
[10]: http://code.google.com/p/counterclockwise/
[11]: https://github.com/cosmin/IClojure
[12]: http://amzn.to/wdq6Lr
[13]: http://groups.google.com/group/clj-syd
[14]: https://github.com/clj-syd/clj-syd/wiki
[15]: http://www.meetup.com/clj-syd/
