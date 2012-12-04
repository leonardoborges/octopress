---
layout: post
title: "Monads in small bites - Part III - Monoids"
date: 2012-12-05 09:25
comments: true
categories: [clojure, functional-programming, haskell]
---

This is Part III of my Monads tutorial. Make sure you read the previous parts:

* [Part I   - Functors](/2012/11/30/monads-in-small-bites-part-i-functors/)

* [Part II  - Applicative Functors](/2012/12/02/monads-in-small-bites-part-ii-applicative-functors/)

* Part III - Monoids (this post)

* Part IV  - Monads


### Monoids

Simply put, Monoids describe types containing a [binary function](http://en.wikipedia.org/wiki/Binary_function) and an identity value.

When applied to the identity value and a random value `x`, said function leaves its argument `x` *untouched*, returning it as a result.

This short description should be enough to get the conversation started.

Here's how Haskell defines a Monoid:

```haskell
class Monoid m where
    mempty :: m
    mappend :: m -> m -> m
    mconcat :: [m] -> m
    mconcat ms = foldr mappend mempty ms
```

This type introduces three new functions so let's walk through each one of them:

> **mempty** - I started with a lie since `mempty` isn't actually a function. You can think of it as a constant of the same type of the Monoid `m`. It is this monoid's identity value.


> **mappend** - A poorly named function, `mappend` is the binary function I mentioned earlier. It receives two arguments of type `m` and returns a value of type `m`

> **mconcat** - It receives a list of Monoids `m` and reduces them to a single Monoid of type `m`. What's interesting about this snippet is that the Monoid type class provides a default implementation for `mconcat`: it simply calls *[foldr](http://www.haskell.org/haskellwiki/Foldr_Foldl_Foldl')* with the binary function `mappend`, a starting value of `mempty` and the list of Monoid values `ms`

Enough Haskell! Let's have a look at a few examples.

Did you know that, in Clojure,  the functions `*` and `+` are monoids? Yup. But don't take my word for it. Let me prove it to you:

```clojure
(def  mempty (+)) ;; 0
(def  mappend +)
(defn mconcat [ms] 
    (reduce mappend mempty ms))

(mappend 3 4) ;; 7

(mconcat [2 3 4]) ;; 9
```

Whoa!  What happened here? Am I just making this stuff up?

Not really. I only defined the same haskell names to their Clojure counterparts for clarity. Totally overkill. The code above is the same as:

```clojure
(+) ;; 0

(+ 3 4) ;; 7

(reduce + [2 3 4]) ;; 9
```

Did you notice that on the second call to `reduce` we did not provide an initial value? That's because `reduce` will attempt to get its initial accumulator by calling the reducing function without arguments - hence `mempty == (+)`.

So that means we don't even need an `mconcat` function since in Clojure,  `reduce` works with monoids as well!

But how the hell do you create a monoid in Clojure then? I'm glad you asked. Let's create our own *plus-monoid*!

#### Your first monoid

In [Part I](/2012/11/30/monads-in-small-bites-part-i-functors/) I implemented Functors using [protocols](http://clojure.org/protocols) and [records](http://clojuredocs.org/clojure_core/clojure.core/defrecord). In [Part II](/2012/12/02/monads-in-small-bites-part-ii-applicative-functors/) I showed how Applicative Functors could be implemented using [multimethods](http://clojure.org/multimethods).

This time around I won't be using any of these. I'll implement Monoids using pure functions:

```clojure
(defn plus-monoid 
    ([]
        0)
    ([a b]
        (+ a b)))

(plus-monoid) ;; 0 - same as mempty

(plus-monoid 3 4) ;; 7 - same as mappend

(reduce plus-monoid [2 3 4]) ;; 9 - when working with monoids, reduce is the same as mconcat
```

We start by defining a function with multiple arities. The first body receives no arguments, so we just return the identity value for summation, which is *0 (zero)*. The second body receives two arguments so we can just add them up. Multiplication can be implemented in a similar fashion but obviously with the identity value of *one*.

Easy, huh?

Oh, by the way, lists are Monoids too! Who'd have thought?

Here's its Clojure implementation:

```clojure
(defn list-monoid 
    ([]
        '())
    ([a b]
        (concat a b)))

(list-monoid) ;; () - remember, same as mempty

(list-monoid [1 2 3] [4 5 6]) ;; (1 2 3 4 5 6) - remember, same as mappend

(reduce list-monoid [[1 2 3] [4 5 6] [7 8 9]]) ;; (1 2 3 4 5 6 7 8 9) - mconcat in action
```

Same rules apply but for lists `mappend` is achieved by using `concat` inside our monoid function. 

Also, since our binary function concatenates two lists together it makes sense that `mempty` is `()` (the empty list). Remember `mempty` is supposed to be an identity value so if we stitch `()` and `[1 2 3]` together, we're left with `[1 2 3]` which is exactly what we'd expect.

> You can see now why I said `mappend` was poorly named. While it makes sense when you think about lists, `mappend` doesn't do any appending in our *plus-monoid* and in fact most monoids don't append anything. Just keep this in mind if you see any haskell code using it: `mappend` is just a binary function.

### Don't break the law

You saw this coming, huh? Monoids also come with a couple of laws. You know the drill. Let's prove they both hold.

#### Identity

> Applying `mappend` to `mempty` and a monoid `x` should be the same as the original `x` monoid.

In Haskell:

```haskell
mappend mempty x = x
mappend x mempty = x
``` 

And the proof in Clojure:

```clojure
;; first, the plus-monoid
(def mempty (plus-monoid))

(def x 10)

;; This...
(plus-monoid mempty x) ;; 10

;; ...is the same as:
(plus-monoid x mempty) ;; 10

;;now, the list-monoid

(def mempty (list-monoid))
(def x [1 2 3])

;; This...
(list-monoid mempty x) ;; (1 2 3)

;; ...is the same as:
(list-monoid x mempty) ;; (1 2 3)
```

#### Associativity

> Applying `mappend` to a monoid `x` and the result of applying `mappend` to the monoids `y` and `z` should be the same as first applying `mappend` to the monoids `x` and `y` and then applying `mappend` to the resulting monoid and the monoid `z`


In Haskell:

```haskell
mappend x (mappend y z) = mappend (mappend x y) z
```

And the proof in Clojure - remember that calling the monoid function with two arguments is equivalent to `mappend` in haskell:

```clojure
;; first, the plus-monoid

(def x 10)
(def y 25)
(def z 40)

;; This...
(plus-monoid x (plus-monoid y z)) ;; 75

;; ...is the same as:
(plus-monoid (plus-monoid x y) z) ;; 75

;;now, the list-monoid

(def x [40])
(def y [10 25])
(def z [50])

;; This...
(list-monoid x (list-monoid y z)) ;; (40 10 25 50)

;; ...is the same as:
(list-monoid (list-monoid x y) z) ;; (40 10 25 50)
```


### Almost there...

This puts an end to Part III. It's time to head to the pub. 

When you're back look for the final post in these series - Part IV - where we will conclude our journey by finally introducing Monads!