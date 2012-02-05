---
layout: post
title: "Project Euler: problem 4 in clojure"
date: 2012-02-05 13:29
comments: true
categories: [clojure, algorithms, project-euler]
---

I solved a few of the problems on [Project Euler][1] in the past, both in Java and Ruby, and thought it would be useful to redo them in Clojure, thus improving my skills on the language's core functions and libraries. Today I'll share [problem 4][2]. 

Go ahead and read it but here's the meat of it: 

*"Find the largest palindrome made from the product of two 3-digit numbers."*

From this statement we can tell two things: (1) we'll need a function that can tell whether a number is a [palindrome][3] or not and (2) that the largest palindrome is given by the product of two numbers between 100 and 999, inclusive.

Let's tackle number one first:

``` clojure
(defn palindrome? [n]
  (= (->> n str reverse (apply str))
     (str n)))
```

With our utility function in hand, one possible solution might be as follows:

``` clojure
(defn largest-palindrome []
  (apply max (filter #(palindrome? %)
                     (for [x (range 100 (inc 999))
                           y (range 100 (inc 999))]
                       (* x y)))))
(time
 (largest-palindrome))
;;"Elapsed time: 1405.358 msecs
```

While this works, I wasn't happy with a couple of things in this solution. First, I thought I could do without using *filter*. Second, we have unnecessary multiplications going on, leading to poor performance - it takes ~1.4secs to finish.

You see, when we begin multiplying the numbers, we'll see multiplications such as _100 \* 100, 100 \* 101, 100 \* 102 ..._ and then again, after the first loop is exhausted, _102 \* 100, 102 \* 101, 102 \* 102 ..._

That led me to take a closer look at [for][4], Clojure's list comprehension macro. It's a very powerful construct, providing 3 useful modifiers: _let_, _while_ and _when_.

With that in mind, I refactored my first solution to look like this:

``` clojure
(defn largest-palindrome-1 []
  (apply max (for [x (range 100 1000)
                   y (range 100 1000)
                   :while (>= x y)
                   :let [z (* x y)]
                   :when (palindrome? z)]
               z)))

(time
 (largest-palindrome-1))
;;"Elapsed time: 689.262 msecs"
```

Here, the _while_ modifier makes sure we aren't wasting any time with unnecessary multiplications. The _when_ modifier lets us get rid of the outer _filter_ call.  And as you can see, the solution is about twice as fast as its first version. 

On top of that, it's still pretty concise. Not bad.

[1]: http://projecteuler.net/
[2]: http://projecteuler.net/problem=4
[3]: http://en.wikipedia.org/wiki/Palindrome
[4]: http://clojuredocs.org/clojure_core/clojure.core/for