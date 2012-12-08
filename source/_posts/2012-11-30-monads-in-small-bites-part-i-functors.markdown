---
layout: post
title: "Monads in small bites - Part I - Functors"
date: 2012-11-30 01:06
comments: true
categories: [clojure, functional-programming, haskell]
---

Today I join the already bloated group of people who wrote monad tutorials. It's a bit of a ritual, really. 


Different than most tutorials though I aim to take a different approach. The good news is that I won't be comparing monads to burritos :)

People say one needs to have his/her own epiphany in order to understand Monads and reading explanations from others is of little help. My goal is to disprove that.

To that end, this tutorial will be split in four parts:

* Part I   - Functors (this post)

* [Part II  - Applicative Functors](/2012/12/02/monads-in-small-bites-part-ii-applicative-functors/)

* [Part III - Monoids](/2012/12/05/monads-in-small-bites-part-iii-monoids/)

* [Part IV  - Monads](/2012/12/08/monads-in-small-bites-part-iv-monads/)

You might want to bookmark this page - once the other parts are up, I'll update the list above with the links to them.

### Before we start

I know what you're thinking: Do I really need to know Applicative Functors just to grasp Monads? 

Well, no. However, I found that gradually building your knowledge from Parts I, II and III will allow you to fully grasp monads without the need for burritos or elephants.

You should also be familiar with a functional programming language. Any language should be fine but you'll get the most out of this tutorial if you're familiar with Haskell and/or Clojure. 

If you're not familiar with Clojure, fear not - Clojure is a small language and the code snippets should still make sense if you put your mind to it - they're all short and sweet. I also encourage you to re-implement the examples in your language of choice to gain a deeper understanding on the subject.

Ready then? Let's dive in.

## Just enough Haskell

This is not a Haskell tutorial but trust me when I tell you that learning just enough about its type signatures will make all the difference in the world in understanding the concepts I'm about to present.

Although I'll be using a little bit of Haskell syntax, I'll also provide implementations in Clojure. They are in order so you should be able to paste all code sample in the REPL and follow along if you wish.

### Type signatures

I'll make this quick. Say you have a function called `mk-vec` that creates a 2D vector with `x` and `y` coordinates. Such function could easily be coded in Clojure like this:

```clojure
(defn mk-vec [x y]
  {:x x :y y})

;; using it
(def my-vec (mk-vec 10 15))
(:x my-vec) ;; 10
```

As you can see, this function takes two arguments - x and y - and returns a map - or hash if you come from Ruby - that wraps those values in it, providing an easy way to retrieve them.

Now if this function had been implemented in haskell this would be its type signature - read the comments:

```haskell 
import qualified Data.Map as Map -- just giving Data.Map an alias
mkVec :: a -> a -> Map.Map [Char] a -- this is the type signature

mkVec x y = Map.fromList [("x", x), ("y", y)] -- this is the implementation. You can ignore this part.

-- using it
myVec = mkVec 10 15
Map.lookup "x" myVec -- Just 10 
--
-- ignore the actual value that is returned. Read it as 10 for now.
```

Whatever comes after the `::` is part of the function type signature. Read it like this:

> This is a function that receives two arguments of type `a` - which means any type - and returns a Map of key/value pairs where the key is of type `[Char]` - technically a list of `Char` values but for all effects and purposes just read it as `String` - and the value is of type `a` - the same type as its arguments. 

It might help to see the type signature in this light:

```haskell
mkVec :: a -> a -> (Map.Map [Char] a)
```

It highlights the return value in parenthesis.

Now let's have a look at a built in function, `*`. As you know, it performs multiplication and this is its type signature:

```haskell
(*) :: Num a => a -> a -> a
```
Can you guess now what it means? I'm sure you can. There's one small difference though: this function has a type constraint - it's that **Num a => ...** part of the signature. Read it like this:

> This is a function that receives two arguments of type `a` and returns a value of the same type, as long as the type of `a` is an instance of `Num`

And that's it for now. Read it again to make sure it's fresh in your mind and then continue.

As we encounter more type signatures, I'll walk you through each one of them - but if you got it up until now, you'll easily grasp the other type signatures.

### Functors

In a nutshell Functors are things that can be mapped over.

Haskell defines Functors like this:

```haskell
class Functor f where
  fmap :: (a -> b) -> f a -> f b
```

Let's dissect that type signature:

> **fmap** is a function that receives two arguments: the first is a function that receives an argument of type `a` and returns a value of type `b` and the second is a Functor that contains a value of type `a` - represented by `f a`. The result of calling `fmap` is a Functor of same type - `f` - containing a value of type **b**, which is the result of applying the function to **a**.

Too much? Let's have a look at an example: the **List** Functor.

If we rewrite the `fmap` type signature as if it only worked on lists, this is what we'd come up with:

```haskell
  fmap :: (a -> b) -> [a] -> [b]
```

This new type signature allows us to rewrite that last definition:

> **fmap** is a function that receives two arguments: the first is a function that receives an argument of type `a` and returns a value of type `b` and the second is a List of zero or more values of type **a**. The result of calling `fmap` is a List of zero or more values of type **b**, each of which is the result of applying the function to each `a` element.

Does this sound familiar to you? It should, because this is essentially what the `map` function available in most functional-*ish* languages does! It takes a function and a list, applies the function to every element in the list while putting the results into a new list, finally returning it.

In fact, the `fmap` implementation of the List Functor from Haskell is implemented in terms of `map`.

Let's see how that could be done in Clojure. I'll use [protocols](http://clojure.org/protocols) and [records](http://clojure.org/datatypes) but they are not strictly required for this.

Here's our Functor protocol:

```clojure
(defprotocol Functor 
    (fmap [functor f] "Maps fn over the functor f"))
```

And now our List Functor:

```clojure
(defrecord List [wrapped] 
    Functor 
    (fmap [functor f] 
        (List. (map f (:wrapped functor)))))
```

In the snippet above all we're saying is that the List record must satisfy the Functor protocol, which makes sense. `fmap` then is responsible for *unwrapping* the value contained in the list functor and *mapping* `f` over it.

Give it a go!

```clojure 
(def my-list-functor (List. [1 2 3])) ;; List{:wrapped (1 2 3)}

(fmap my-list-functor #(* 2 %)) ;; List{:wrapped (2 4 6)}
```

We can now `map` arbitrary functions over the values `wrapped` in a Functor! Awesome!

> **Note**: In our Clojure version of the List Functor, it is implemented as a Record that wraps a primitive Clojure list/vector - `[]`. As I mentioned this is not necessary but I chose to do it here to explicitly show the relationship with the Haskell types.

### Don't break the law

Now that we got a feel for what Functors are, it's worth noting that they must obey a few laws to be considered full-fledged Functors.

#### Identity

> Mapping an identity function over a Functor is the same as applying identity to the Functor itself


This is how Haskell puts this law:

```haskell
fmap id functor = id functor
```

Translating that to Clojure:

```clojure
;;This...
(fmap my-list-functor identity) ;; List{:wrapped (1 2 3)}

;;is the same as:
(identity my-list-functor) ;; List{:wrapped [1 2 3]}
```

#### Composition

> If you compose the functions `f` and `g` and map the resulting function over the Functor, that is the same as first mapping `g` over the Functor and `then` mapping `f` over the resulting Functor.


From the description you can see that this law involves [function composition](http://en.wikipedia.org/wiki/Function_composition). In Clojure, that's achieved with the [comp](http://clojuredocs.org/clojure_core/clojure.core/comp) function.

Again, let's see how this law is defined in Haskell:

```haskell 
fmap (f . g) functor = fmap f (fmap g functor)
```

> **Note**: The `.` operator denotes function composition in Haskell

And the proof in Clojure:

```clojure
(def f #(+ 10 %))
(def g #(* 2 %))

;; given the above, this...
(fmap my-list-functor (comp  f g)) ;; List{:wrapped (12 14 16)}

;; is the same as:
(-> my-list-functor (fmap g) (fmap f)) ;; List{:wrapped (12 14 16)}
```

> **Note**: make sure you're familiar with [Clojure's threading macro: **->** ](http://clojuredocs.org/clojure_core/clojure.core/-%3E). I'll be using it in most code snippets.


Nice! All laws hold so we can sleep peacefully in the knowledge that our Functor works as expected.

Now go get a drink and stay tuned for [Part II](/2012/12/02/monads-in-small-bites-part-ii-applicative-functors/), where I'll introduce *Applicative Functors*.