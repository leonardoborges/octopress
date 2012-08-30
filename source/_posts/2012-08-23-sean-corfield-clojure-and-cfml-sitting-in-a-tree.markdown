---
layout: post
title: "Sean Corfield: Clojure and CFML sitting in a tree"
date: 2012-08-23 00:03
comments: true
categories: [clojure, functional-programming, clj-syd]
---

Last night I attended the Adobe User Group here in Sydney. That might strike some of you as a big surprise
given my relationship with Adobe is pretty much limited to fiddling with Photoshop/Lightroom to get my photos looking nice.

However the reason for which I attended the meetup is that [Sean Corfield](https://twitter.com/seancorfield) - a prolific member of the Clojure community - gave a presentation on how he introduced and migrated most of his backend code at [World Singles](http://worldsingles.com/ws2010/index.cfm) from [CFML](http://en.wikipedia.org/wiki/ColdFusion_Markup_Language) to [Clojure](http://clojure.org/) - hence my interest.

What follows is a mix of my own notes and what I could remember from the slides:

### Going faster

The original platform was built in CFML between 2001 and 2008. It was essentially monolithic procedural code. It was rewritten in 2009 - when they introduced OO and a new version of CFML

### Brief Stats

* 3 million members
* 1 million emails sent every day
* 2k concurrent users 24x7, on average

### Clojure and WorldSingles

* Tried .NET - ran into all sorts of trouble. Didn't work well in production.
* Tried Scala - Memory leaks in the built-in actor library were a deal breaker. The type system also wasn't a good cultural fit.
* Evaluated Clojure in 2010 - seemed like a good language to squeeze performance out of lower level components
* Clojure version released in production in 2011
	* Rewrote remaining Scala Code
	* 6.3k LOC - 1.5k Tests LOC
	* equivalent to 4x the CFML LOC
	* Clojure is essentially used in the model (as in MVC)
	
### Why add Clojure?

* It's fast - compiles down to JVM bytecode (and it's faster than CFML)
* Immutability (automatic thread safety)
	* they found several thread safety bugs in the third party ColdFusion libraries being used
* Built-in concurrency / parallelism support - [future](http://clojuredocs.org/clojure_core/clojure.core/future), [pmap](http://clojuredocs.org/clojure_core/clojure.core/pmap), [pvalues](http://clojuredocs.org/clojure_core/clojure.core/pvalues) etc.
* Lazy sequences - being able to work with potentially ininite collections without bringing your server down.
* All high quality, production ready Java libraries easily accessible via [java interop](http://clojure.org/java_interop).

### What do they use it for?

* Email
	* html generation & sending
	* tracking & log file analysis
* Geolocation (rest/json)
* i18n
* Reporting (parallel queries) - he showed a bit of this code. Heavy use of [futures](http://clojuredocs.org/clojure_core/clojure.core/future) + [deref](http://clojuredocs.org/clojure_core/clojure.core/deref)
* Search engine integration (json/xml)
	* This breaks down to two Clojure components:
	* One feeds the search engine based on changes to the users profiles
	* The other then runs against the search engine 24x7 trying to find new matches for users
* (lightweight?) Persistence layer
	* no ORM - thin framework over sql instead
	* Same interface for both Mysql & MongoDB
	* MongoDB being used after hitting performance issues in Mysql
	
### Clojure ecosystem/community

Having an active Community is crucial for language adoption and the clojure community got this right:

* Community
	* 6700 developers on the [mailing list](https://groups.google.com/forum/?fromgroups#!forum/clojure)
	* ~300 developers on IRC 24x7
* Active library development
	* \#23 language on Github
	* Over 7k projects on github
	* Nearly 2k libs on [Clojars](https://clojars.org/)
	
### The Future

* Looking into [Cascalog](https://github.com/nathanmarz/cascalog) for big data processing
* All new back-end/model code written in Clojure
* Views/Controllers will remain in CFML
	* This ties up with Sean's answer to 'What would you not use Clojure for?'
	* He mentioned HTML rendering as the major reason. You can't hand [hiccup](https://github.com/weavejester/hiccup) code to a designer. 
	* Similarly, while designers can work on [Enlive](https://github.com/cgrand/enlive) templates, updating the dynamic bits in Clojure can be cumbersome.


### Summary

It was great to hear the many good things Sean had to say about Clojure. WorldSingles seem pretty happy with the decision of migration their heavy-lifting code to it and hopefully these slide notes will give you some insight into the sort of things Clojure is good at.

~~Sean is likely to put his slides up somewhere so I'll link to it as soon as I have it.~~ [Here they are.](http://corfield.org/articles/WorldSinglesWeb.pdf)

If you want to know more about Clojure and be involved in the community, you should come to the next [clj-syd meetup](http://www.meetup.com/clj-syd/).
	


