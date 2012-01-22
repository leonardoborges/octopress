---
layout: post
title: "Backlog: Ola Bini on Clojure/conj"
date: 2012-01-22 22:01
comments: true
categories: [clojure, clj-syd]
---

As I promised in my [previous post][1], here are the highlights of what [Ola Bini][2] had to say about [Clojure/conj][3]. 

Clojure/conj is the largest gathering of clojure programmers, hackers and enthusiasts, with a single track of talks spanning three days.
Ola selected his favorite talks and summarised each one of them.

The actual slides for each one of the talks can be found on [github][4].

You can find Ola's slides [here][5].
 
 
### Fun Data Structures ###
 
This is certainly a very interesting topic. The highlight here is the approach Clojure takes to handling immutable data structures efficiently: persistent data structures through structural sharing - maybe a good short talk for the next meetup?
 
However, even though efficient, there are cases where the space complexity of known algorithms gets compromised.
More specifically, algorithms that rely on the ability of changing data structures in-place - such as the QuickSort - will suffer from this approach.
 
I asked Ola about it and the short answer is that you should use a different algorithm that takes advantages of trees instead, since they often don't rely on in-place changing.
 
The long answer is a book called [Purely Functional Data Structures][6] - another one added to my wish list.
 
 
### Logic/constraint programming ###
 
This is a topic that's completely new to me and it sounds fascinating at first. It also ended up being the topic of the rest of the tech night, with Ola, Sarah and Jo - also ThoughtWorkers - giving us a Logic Programming 101 crash course using Prolog. Lots to read up on.
 
In the Clojure world, logic programming can be achieved via [core.logic][7].
 
The book [The Reasoned Schemer][8] was also mentioned as a good read on the subject.
 
 
### Cascalog ([github repo][9]) ###
 
Haven't had the time to play with it yet but it seems extremely useful. It's a Clojure-based DSL for processing "Big Data" on top of Hadoop.
 
 
There you have it. Come [join us][10] for our next meeting!


[1]: http://bit.ly/clj-syd
[2]: http://olabini.com/blog/
[3]: http://clojure-conj.org/
[4]: https://github.com/relevance/clojure-conj
[5]: http://db.tt/lTFsOKpZ
[6]: http://amzn.to/zXbvtu
[7]: https://github.com/clojure/core.logic
[8]: http://amzn.to/y6vhT5
[9]: https://github.com/nathanmarz/cascalog
[10]: http://bit.ly/yLfPTr