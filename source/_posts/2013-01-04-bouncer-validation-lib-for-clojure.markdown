---
layout: post
title: "Announcing bouncer, a validation library for Clojure Apps"
date: 2013-01-04 17:39
comments: true
categories: [clojure, functional-programming]
---

Today I'm releasing bouncer, which was extracted from a project I've been working on.

It's a validation library for Clojure apps and it lets you write code like this:

```clojure
(def person {:name "Leo"})

(validate person
    :name required
    :age  [required number])
```

If you'd like to see more examples and a detailed guide check out the [github repository](http://github.com/leonardoborges/bouncer). The README should get you started.

This post however isn't only about announcing bouncer. It's also about the motivation and implementation details behind it.


There are a couple of Clojure validation libraries already out there so why would I write a new one? 
Well...

- Writing Clojure is fun! (who knew? :P)

- Because I believe this problem can be solved more elegantly with the use of Monads

If you've been following me for a while, you'll know that I spent most of 2012 deepening my knowledge about functional programming.

In that journey, the unavoidable subject of monads came about - and it was both interesting and enlightening enough that made me [write a whole series of posts about it](http://www.leonardoborges.com/writings/2012/11/30/monads-in-small-bites-part-i-functors/).


After learning what they are and then thinking about the validation problem for a while, I couldn't help but notice that the problem had a lot in common with the [State Monad](http://www.haskell.org/haskellwiki/State_Monad). 

In order to explain how the two relate, I'll have to digress for a moment. It'll all make sense in the end - or so I hope

## Purity

In pure functional languages, such as Haskell, functions can't have side effects. These include performing IO, changing global variables or launching missiles.

Because of that, functions in Haskell are pure: if you repeatedly call a function *f* with the same argument `x` over time you will *always* get the same result back.

Pure functions are not a feature of Haskell though. We, too, can write pure functions if we wish:

```clojure
(defn double [x]
    (+ x x))
```

The function `double` above is pure. If we call it with 10, we can be sure the result will always be 20.

This leaves us with a question though: If we were to write our programs with pure functions only, how would we perform computations that need to carry state - state that needs to change over time - around?

A good example of such computation is generating random numbers.

Most programming languages provide generators capable of creating random numbers on demand. Using them is usually trivial. Here's an example in Java:

```java
Random gen = new Random();

gen.nextDouble(); // 0.0037635726242281065
gen.nextDouble(); // 0.15821091918430885
```

**Impurity alert!**

The function `nextDouble` above is obviously *not* pure. Multiple invocations of it with the same argument - in this case, none - returns different results. 

`nextDouble`  is keeping some sort of global state between function calls.

This is where the State Monad comes in. It allows such functions to remain pure.

## The State Monad

The State Monad provides a way to abstract *state* from the function that needs to operate on it.

Sounds confusing? Hopefully an example will clear things up.

Let's have a look at the clojure function `rand`:

```clojure
(rand)
;; 0.04388682005715605
(rand)
;; 0.43057496371080517
```

`rand` suffers from the same problem as `nextDouble` we saw above. It keeps it's own state that is shared across calls, therefore being an impure function.

Now let's write a pure version of `rand`. We'll call it `pure-rand`:

```clojure
(def gen (java.util.Random.))

(defn pure-rand [g]
    [(.nextDouble g) g])

(pure-rand gen)
;; [0.5783608063218478 #<Random java.util.Random@7f30ab6>]

(pure-rand gen)
;; [0.9251968987499839 #<Random java.util.Random@7f30ab6>]
```

This is interesting. `pure-rand` now takes a generator as an argument and returns a two-element vector containing the random number itself - the result we're actually interested in - and the generator that was passed in.

Recall however that the generator returned, albeit the same object, is in a new sate.

By re-writing the function like this, we've regained purity, as demonstrated below:

```clojure
(pure-rand (java.util.Random. 100))
;; [0.7220096548596434 #<Random java.util.Random@bb14fe1>]

(pure-rand (java.util.Random. 100))
;; [0.7220096548596434 #<Random java.util.Random@bb14fe1>]
```

As you can see, as long as we provide the same generator - the same argument - we get the same result back.

If you're wondering why I'm returning a two element vector from our little function, the answer lies in the State Monad implementation as found in the [algo.monads](https://github.com/clojure/algo.monads/) library.

From its docstring:

> **State Monad**: Monad describing stateful computations. The monadic values have the structure: 
    (fn [old-state] [result new-state]).

It expects a function that receives its old state and returns the result and the new state - these are called monadic values.

By designing the function to follow this contract, we can leverage the `domonad` macro - think of it as syntactic sugar for working with monads:


```clojure
(require '[clojure.algo.monads :as m])

((m/domonad m/state-m
    [a pure-rand
     b pure-rand
     c pure-rand]
     [a b c])
     (java.util.Random. 100))

;; [[0.7220096548596434 0.19497605734770518 0.6671595726539502] 
;; #<Random java.util.Random@358ddfd6>]
```

In the example above, we're using our `pure-rand` function in the context of the State Monad to generate 3 random numbers - based on some initial state - and returning them as a vector.

As we've seen before the result is itself in a vector alongside the new state.

This is where the State Monad and the validation problem meet: 

In [bouncer](http://github.com/leonardoborges/bouncer), each validation function is designed to be compatible with the State Monad, just like `pure-rand` above: 

It receives an initial state - at first, the map to be validated - and returns a vector with the map of errors and the new state: the original map augmented with any errors from previous validators.

The end result, should one or more validations fail, is a map with all errors that might have happened, plus our new state.

Now if you head to the [github repository](http://github.com/leonardoborges/bouncer) and read the examples by keeping the State Monad and the above explanation in mind, the similarities should be obvious.

## Wrapping up

As I mentioned in the beginning of the article, there are [other](https://github.com/r0man/validation-clj) validation [libraries](https://github.com/michaelklishin/validateur) for Clojure and at the time of this writing they have more features than [bouncer](http://github.com/leonardoborges/bouncer) - by all means have a look at them.

However I will keep maintaining [bouncer](http://github.com/leonardoborges/bouncer) for a couple of reasons:

- That's what I'm using in my current side project
- It takes a fundamentally different implementation approach that is in itself worthy of exploration
- If nothing else, this is yet another example of where Monads can be useful.

#### Acknowledgments 

Thanks to [Steve](https://twitter.com/stevebuik) and [Julian](https://twitter.com/juliansgamble) for reviewing early drafts of this post as well as [Nick](https://twitter.com/nick_s_drew) for being such a PITA :) - our discussions led to a considerably nicer design. 

As usual, let me know your thoughts.