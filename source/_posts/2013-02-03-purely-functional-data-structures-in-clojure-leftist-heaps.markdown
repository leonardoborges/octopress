---
layout: post
title: "Purely functional data structures in Clojure: Leftist Heaps"
date: 2013-02-03 12:41
comments: true
categories: [clojure, functional-programming, functional-data-structures]
---

> This post is part of a series about Chris Okasaki's [Purely Functional Data Structures](http://amzn.to/UcIidh). You can see all posts in the series by visiting the [functional-data-structures](http://www.leonardoborges.com/writings/tags/functional-data-structures/) category in this blog.

* * *


Last year I started reading a book called [Purely Functional Data Structures](http://amzn.to/UcIidh). It's a fascinating book and if you've ever wondered how Clojure's persistent data structures work, it's mandatory reading.

However, all code samples in the book are written in [ML](http://bit.ly/YqYjtt) - with [Haskell](http://bit.ly/YqYmp6) versions in the end of the book. This means I got stuck in Chapter 3, where the ML snippets start.

I had no clue about Haskell's - much less ML's! - syntax and I was finding it very difficult to follow along. What I did notice is that their syntaxes are not so different from each other. 

So I put the book down and read [Learn You a Haskell For Great Good!](http://amzn.to/VuD3jT) with the hopes that learning more about haskell's syntax - in particular, learning how to read its type signatures - would help me get going with *Puretly Functional Data Structures*. 

Luckily, I was right - and I recommend you do the same if you're not familiar with either of those languages. [Learn You a Haskell For Great Good!](http://amzn.to/VuD3jT) is a great book and I got a lot out of it. [My series on Monads](http://www.leonardoborges.com/writings/2012/11/30/monads-in-small-bites-part-i-functors/) is a product of reading it.

Enough background though.


The purpose of this post is two-fold: One is to share the [github repository](https://github.com/leonardoborges/purely-functional-data-structures) I created and that will contain the Clojure versions of the data structures in the book as well as most solutions to the exercises - or at least as many as my time-poor life allows me to implement.  

The other is to walk you through some of the code and get a discussion going. Hopefully we will all learn something - as I certainly have when implementing these. Today, we'll start with Leftist Heaps.

### Leftist Heaps

[Leftist Heaps](http://en.wikipedia.org/wiki/Leftist_tree) - or trees - are a variant of [binary heaps](http://en.wikipedia.org/wiki/Binary_heap) that can be used as priority queues. On top of the standard invariants of binary heaps, it obeys the leftist property:

- Every node has a *rank*, which is the distance from its right spine to the nearest leaf
- A node's left child has a rank at least as large as its right child

In a nutshell, these are the operations we need to be able to perform on a leftist heap:

- insert a value into an existing heap
- merge two heaps
- find the minimum value in a heap
- delete the minimum value, returning a new heap

Since the book uses ML/Haskell, it starts with a data type definition for Heaps that exposes these and a couple of other auxiliary functions. I decided to take a stab at writing the solution using Clojure's protocols and records:

```clojure
(defprotocol Heap
  (is-empty?   [this])
  (insert      [this v])
  (merge       [this other])
  (rank        [this])
  (find-min    [this])
  (delete-min  [this]))

(defrecord LeftistHeap [rank value left right])
``` 

When implementing the algorithms the base case for the recursive solutions will involve dealing with *nil* values which at first seems like it wouldn't be a problem. However, protocol functions dispatch on the type of its first argument so what happens if I call the function *is-empty?* on *nil*?

Luckily, Clojure allows us to extend a protocol to core types:

```clojure
(extend-protocol Heap
  nil
  (rank [_] 0)
  (merge [_ other] other)
  (is-empty? [_] true)

  LeftistHeap
  (is-empty? [this]
    (nil? this))

  (rank [this]
    (:rank this))
  
  (merge [{val-this :value left-this :left right-this :right :as this}
          {val-other :value left-other :left right-other :right :as other}]
    (cond
     (is-empty? other) this
     (<= val-this val-other) (ensure-leftist left-this
                                             (merge right-this other)
                                             val-this)
     :else (ensure-leftist left-other
                           (merge this right-other)
                           val-other)))
  
  (insert [this v]
    (merge (->LeftistHeap 1 v nil nil)
           this))

  (find-min [{v :value}] v)
  
  (delete-min [{left :left right :right}]
    (merge right left)))
```

Note how I extended a few of the protocol functions to the nil data type, allowing me to continue with this implementation with no nasty hacks.

There's one last bit missing: a function that will ensure each heap retains the leftist property:

```
(defn ensure-leftist
 [this other v]
 (let [rank-this (rank this)
       rank-other (rank other)]
   (if (>= rank-this rank-other)
     (->LeftistHeap (inc rank-other) v this other)
     (->LeftistHeap (inc rank-this) v other this))))
```

The reason this function is isolated is that the Heap protocol defined above is fairly generic and could be used for defining other types of heaps - and I didn't feel it warranted its own interface.

We can now play with it and create a new leftist heap:

```clojure
(-> (->LeftistHeap 1 3 nil nil)
                   (insert 2)
                   (insert 7)
                   (insert 4)
                   (insert 10)
                   (insert 1)
                   (insert 20))
```

While I quite like this approach, I thought I'd also implement this solution using Clojure's core data types - maps in this case - and no protocols. The code is shown below:

```clojure
(defn mk-heap [rank value left right]
  {:rank rank :value value :left left :right right})

(defn heap-rank [heap]
  (if (nil? heap)
    0
    (:rank heap)))

(defn ensure-leftist-heap [value heap-a heap-b]
  (let [rank-a (heap-rank heap-a)
        rank-b (heap-rank heap-b)]
    (if (>= rank-a rank-b)
      (mk-heap (inc rank-b) value heap-a heap-b)
      (mk-heap (inc rank-a) value heap-b heap-a))))

(defn merge-heaps [{val-a :value left-a :left right-a :right :as heap-a}
                   {val-b :value left-b :left right-b :right :as heap-b}]
  (cond
   (nil? heap-a) heap-b
   (nil? heap-b) heap-a
   (<= val-a val-b) (ensure-leftist-heap val-a
                                         left-a
                                         (merge-heaps right-a heap-b))
   :else (ensure-leftist-heap val-b
                              left-b
                              (merge-heaps heap-a right-b))))

(defn heap-insert [value heap]
  (merge-heaps (mk-heap 1 value nil nil)
               heap))

(defn heap-find-min [{v :value}] v)
  
(defn heap-delete-min [{left :left right :right}]
  (merge-heaps right left))
```

Using it is equally simple:

```clojure
(->> (mk-heap 1 3 nil nil)
                    (heap-insert 2)
                    (heap-insert 7)
                    (heap-insert 4)
                    (heap-insert 10)
                    (heap-insert 1)
                    (heap-insert 20))
```

That's it for now.

As I implement more of the book's code and exercises I'll add them to the [github repo](https://github.com/leonardoborges/purely-functional-data-structures) - it also includes tests for all implementations.

Enjoy :)
