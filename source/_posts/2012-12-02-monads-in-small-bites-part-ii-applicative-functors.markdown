---
layout: post
title: "Monads in small bites - Part II - Applicative Functors"
date: 2012-12-02 21:43
comments: true
categories: [clojure, functional-programming, haskell]
---

This is Part II of my Monads tutorial. Make sure you read the previous parts:

* [Part I   - Functors](/2012/11/30/monads-in-small-bites-part-i-functors/)

* Part II  - Applicative Functors (this post)

* Part III - Monoids

* Part IV  - Monads

### Applicative Functors

In [Part I](/2012/11/30/monads-in-small-bites-part-i-functors/) I talked a little about Haskell type signatures and introduced Functors, which provide a way to map standard functions over values which are *wrapped* inside a Functor - we used `fmap` for that. You might want to [skim through it]() again as a refresher.

Now suppose you have Functors that *wrap* functions and that you want to apply those *wrapped* functions to other Functors, maybe even composing new functions on the way! 

What then?

Well you're in luck! Applicative Functors do just that! They're **Functors on steroids**.


Here's how Haskell defines the Applicative data type:

```haskell
class (Functor f) => Applicative f where
    pure :: a -> f a
    (<*>) :: f (a -> b) -> f a -> f b
```

Based on our previous knowledge of Haskell's type signatures, we can infer from this definition that in order for it to be an Applicative Functor, `f` *must* already be a Functor.

Let's break this down and have a closer look at the two new functions this type introduces:

> **pure** is a function that takes a value `a` and *wraps* it into a minimal Functor `f`.

> __<*>__ is a function that takes two arguments: the first is a Functor `f` that wraps a function of type `a -> b`. The second argument is a Functor `f` that wraps a value - which could be a function! - of type `a`. The final result is a Functor `f` that wraps some value of type `b` - which was obtained by somehow applying the function `(a -> b)` to the Functor `f a`.

`pure` has a straightforward explanation whereas `<*>` is a bit more involved. 

To clear things up, I'll show the type signatures again but this time as if they only worked with the List Functor that we've been working on:

```haskell
pure :: a -> [a]
(<*>) :: [(a -> b)] -> [a] -> [b]
```

Let's revisit those definitions:

> **pure** is a function that takes a value `a` and *puts* it into an empty list, returning the resulting single element list.

> __<*>__ is a function that takes two arguments: the first is a list containing one or more functions of type `a -> b`. The second argument is a list of one or more values - or functions! - of type `a`. The final result is a list of one or more values of type `b` - which was obtained by somehow applying the function `(a -> b)` to the Functor `f a`.

Enough definitions though! Let's extend our List Functor and make it an Applicative as well. 


While we'll still be using the List Functor we implemented in [Part I](), this time I'll implement its Applicative version using [multimethods](http://clojure.org/multimethods) for a change.  Here's the code:

```clojure
;; it dispatches on the record type since we could have implementations of pure for List, Maybe, Either etc...
(defmulti pure (fn [f _] f)) 
(defmethod pure List [_ v] 
    "Wraps value v in a list"
    (List. [v]))

;; it dispatches on the class of the Functor instance passed in the 1st argument
(defmulti <*> (fn [fs _] (class fs))) 
(defmethod <*> List [fs xs] 
    "Unwraps the functions in fs, applies them to the Functors in xs, wrapping the result at the end"
    (List. (for [f (:wrapped fs)
                 x (:wrapped xs)]
                (f x))))
```

By focusing on the List as an Applicative Functor we can more easily understand what these functions do. From the code above, `pure`'s job is a simple one: all it does is *wrap* it's argument `v` into a minimal List functor which in our case means a Functor wrapping a one element list.

`<*>` on the other hand is responsible for somehow unwrapping the functions brought in by the Applicatives in `fs` and applying them to the [Applicative] Functors in `xs`. It does that by using [list comprehensions](http://clojuredocs.org/clojure_core/clojure.core/for) and *wraps* the result into a new List Functor.

Study this code carefully. It *can* be tricky.

> **Note**: When I first encountered **<\*>** I had no idea what this function was called. I asked the twittersphere and it seems it's called `apply`. In the process of figuring this out I was enlightened [by this conversation](https://twitter.com/leonardo_borges/status/267777875367841792). It turns out `<*>` has several names. Can you guess which one is my favorite? :)

With the Applicative functions defined for our List, let's take it for a spin:

```clojure
(def fs (List. [#(* 2 %) #(+ 10 %)]))
(def xs (List. [1 2 3]))

(<*> fs xs) ;; List{:wrapped (2 4 6 11 12 13)}


(def g (pure List #(* 50 %)))
(<*> g xs) ;; List{:wrapped (50 100 150)}
```

There should have been no surprises here. Read the code again and make sure it's all fresh before moving along.

### Don't break the law

Just as Functors, Applicative Functors also need to obey some laws:


#### Identity

> Feeding a function `f` to `pure` and applying the resulting Applicative to the Functor `v` should be the same as directly mapping `f` over the Functor `v`

In Haskell speak:

```haskell
pure f <*> v = fmap f v
```

And this is the proof, in Clojure:

```clojure
(def f #(+ 2 %))
(def v (List. [10]))

;; given the above, this...
(<*> (pure List f) v) ;; List{:wrapped (12)}


;; ...is the same as:
(fmap v f) ;; List{:wrapped (12)}

```

#### Composition 

> The result of *applying* an Applicative Functor that yields the **function composition** operator to the Applicative `u`, then apply the resulting Functor to `v` and finally applying that result to the final Applicative `w` should be the same as *applying* `v` to `w` and then *applying* `u` to the resulting *Applicative*.

That was a mouthful! Let's see how Haskell tells this story:

In Haskell:

```haskell
pure (.) <*> u <*> v <*> w = u <*> (v <*> w)
```

I needed to cheat a bit in Clojure to prove this law since functions are not [curried by default like they are in Haskell](http://www.haskell.org/haskellwiki/Currying). But this code should still clearly show how this law holds:

```clojure
(def u (List. [#(* 2 %)]))
(def v (List. [#(+ 10 %)]))
(def w (List. [1 2 3]))

;; Given the above, this...
(-> (pure List (fn [x] (partial comp x))) 
  (<*> u)
  (<*> v)
  (<*> w)) ;; List{:wrapped (22 24 26)}

;; ...is the same as:
(<*> u (<*> v w)) ;; List{:wrapped (22 24 26)}
```


#### Homomorphism

> The result of applying the `pure` value of `f` to the `pure` value of `x` should be the same as applying `f` directly to `x` and then feeding that into `pure`.

In Haskell:

```haskell
pure f <*> pure x = pure (f x)
```

And in Clojure:

```clojure
(def f #(* 2 %))
(def x 10)

;; given the above, this...
(-> (pure List f) 
  (<*> (pure List x))) ;; List{:wrapped (20)}

;; ...is the same as:
(pure List (f x)) ;; List{:wrapped (20)}
```


#### Interchange

> The result of applying an Applicative Functor `u` to the `pure` value of `y` should be the same as taking the Applicative obtained by calling `pure` with a function that applies its argument to `y` and then applying that to `u`

In Haskell:

```haskell
u <*> pure y = pure ($ y) <*> u
```

This type signature presents new syntax so before proving the law in Clojure, I want to explain what `($ y)` means. 

In Haskell, `$` is the function application operator. So if we give `y` a value of *10*, I can show you that in this law `$` essentially translates to a single argument function that applies its argument to *10*:

```haskell
let double a = a * 2 -- helper function. it doubles its argument

-- given the above, this...
(($ 10) double) -- 20

-- ...is the same as:
let dollarTen = (\a -> (a 10))  -- this is Haskell's lambda syntax. It's equivalent to ($ 10)
((dollarTen) double) -- 20
```

Now, to the proof in Clojure:

```clojure
(def u (pure List #(+ 10 %)))
(def y 50)

;; given the above, this...
(<*> u (pure List y)) ;; List{:wrapped (60)}

;; ...is the same as:
(def dollar-y #(% y)) ;; it's called dollar-y to show the correlation with the explanation above
(<*> (pure List dollar-y) u) ;; List{:wrapped (60)}
```

This brings us to the end or Part II. Two down and two to go.

I hope you're still with me but go home now. 

Or better yet go to the gym lift some weights and think about these Functors on steroids. When you're back, look out for Part III.