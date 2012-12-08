---
layout: post
title: "Monads in small bites - Part IV - Monads"
date: 2012-12-08 17:36
comments: true
categories: [clojure, functional-programming, haskell]
---

This is Part IV of my Monads tutorial. Make sure you read the previous parts:

* [Part I   - Functors](/2012/11/30/monads-in-small-bites-part-i-functors/) 

* [Part II  - Applicative Functors](/2012/12/02/monads-in-small-bites-part-ii-applicative-functors/)

* [Part III - Monoids](/2012/12/05/monads-in-small-bites-part-iii-monoids/)

* Part IV  - Monads (this post)


### A quick recap

In [Part I](/2012/11/30/monads-in-small-bites-part-i-functors/) we learned about *Functors*, which are things that can be mapped over using a normal function - `fmap` is used for that. 

[Part II](/2012/12/02/monads-in-small-bites-part-ii-applicative-functors/) tought us that when our Functors themselves contain functions and we want them applied to the values contained in other Functors, *Applicatives* come to the rescue - and bring theirs friends `pure` and `<*>`.

[Part III](/2012/12/05/monads-in-small-bites-part-iii-monoids/) introduced Monoids which model a special type of relationship involving binary functions and their identity values.

Now it's time for what I hope is the post you have all been waiting for :)

### Monads

#### A word on context

So far I've said things such as *wrapping* stuff in Functors, *unwrapping* functions from Applicatives and putting results into minimal Functors. All this really means is that [Applicative]Functors - and Monads - have associated contexts that model some sort of computation.

For lists, for example, this means they represent computations that can have several results - non-determinism. 

These computations can have much greater implications though - they can represent failure (or not!), do IO and even launch nuclear missiles. The point is: when we combine Functors/Applicatives/Monads, we carry their context with us to the end - they are essentially *sequenced* together.

This will become clearer with an example. For once I won't start with lists - w00t! - so get ready for it!

### The Maybe Monad

The Maybe monad models computations that can fail. Let's have a look at an example.

Say you have an e-commerce system. When placing an order, a few things need to get done:

- gather information about the order;
- calculate shipping rates;
- apply discount codes, if any, and;
- finally place the order.

The code below shows the supporting functions that will be orchestrated in order to achieve this:

```clojure
(defn calculate-shipping-rate [address]
    (if (= (:country address) "Australia")
        10.0
        nil))

(defn apply-shipping-costs [order shipping-rate]
    (assoc order :total (+ (:total order) shipping-rate)))

(defn lookup-discount-code [code]
    (if (= code "XMAS2012")
        5.0
        nil))

(defn apply-discount-code [order discount]
    (assoc order :total (- (:total order) discount)))

(defn place [order]
    (prn (str "Off you go! Order total: $" (:total order))))
```

Note that based on the code above, we can *only* ship to Australia and there is *only one* active discount code. Keep this in mind - you'll see why later on.


Now let's place an order for some Jalapeño sauce:

```clojure 
(def order {
    :items [{:name "Jalapeño sauce" :price 20.0}]
    :address {:country "Australia"}
    :discount-code "XMAS2012"
    :total 20.0
})

(def shipping-rate (calculate-shipping-rate (:address order)))
(def discount (lookup-discount-code (:discount-code order)))

(-> order
    (apply-shipping-costs shipping-rate)
    (apply-discount-code discount)
    (place))
;; "Off you go! Order total: $25.0"
```

Great! Soon I'll be receiving some hot sauce to go with my burritos!


But wait, what if I had mistakenly set my address to somewhere other than Australia? How would this code behave?

```clojure
(def another-order {
    :items [{:name "Jalapeño sauce" :price 20.0}]
    :address {:country "Brazil"}
    :discount-code "HACKERZ"
    :total 20.0
})


(def shipping-rate (calculate-shipping-rate (:address another-order)))
(def discount (lookup-discount-code (:discount-code another-order)))

(-> another-order
    (apply-shipping-costs shipping-rate)
    (apply-discount-code discount)
    (place))
;; NullPointerException   [trace missing]
```

**Oops**! Your e-commerce system just crashed! Not cool. But hey, this is easy to fix, right? We could just change our *apply-shipping-costs* function to something like this:

```clojure
(defn apply-shipping-costs [order shipping-rate]
    (if shipping-rate
        (assoc order :total (+ (:total order) shipping-rate))
        order))


;;Remember we only support one discount code so the same problem could happen again
;;We need to change the apply-discount-code function as well

(defn apply-discount-code [order discount]
    (if discount
        (assoc order :total (- (:total order) discount))
        order))
```

Now let's see what happens:

```clojure
(def shipping-rate (calculate-shipping-rate (:address another-order)))
(def discount (lookup-discount-code (:discount-code another-order)))

(-> another-order
    (apply-shipping-costs shipping-rate)
    (apply-discount-code discount)
    (place))
;; "Off you go! Order total: $15.0"
```

Well, it doesn't *crash* but we can't ship to Brazil anyway! So the code is *still* incorrect! What we really want is a way to halt the whole computation - placing an order - if any of those steps fail.

Of course we could fix it with a couple more *if* forms before trying to call the *place* function but you see where this is going.

Essentially our nice little functions became burdened with *context*: each of them is now aware that they can fail and need to cater for it.

### Enter the Monad

I'll jump straight to how the code could look like if we had monads - it won't work now because we haven't actually implemented the monad yet, but this should whet your appetite. 

Also, assume we reversed the changes from before - the functions don't have the *if* forms checking its arguments any longer, just like in the original version. Here's the code:

```clojure
(domonad maybe-monad
    [order order
     shipping-rate (calculate-shipping-rate (:address order))
     discount (lookup-discount-code (:discount-code order))]
     (-> order
        (apply-shipping-costs shipping-rate)
        (apply-discount-code discount)
        (place)))
;;"Off you go! Order total: $25.0"

(domonad maybe-monad
    [order another-order
     shipping-rate (calculate-shipping-rate (:address order))
     discount (lookup-discount-code (:discount-code order))]
     (-> order
        (apply-shipping-costs shipping-rate)
        (apply-discount-code discount)
        (place)))
;; nil
```
`domonad` receives the monad you want to operate on, a vector of bindings and an expression that's the final result of the whole thing.

Is your mind blown yet? :) Somehow the whole operation fails and yields `nil` in the second call to *domonad* above - without any *if* forms and without crashing! To see why that is, I'll now explain the monad type class from Haskell.

### The Monad Type Class

Here's the Haskell definition of the Monad type class (I left the `fail` function out so we can focus on the core of it):

```haskell
class Monad m where  
    return :: a -> m a  
  
    (>>=) :: m a -> (a -> m b) -> m b  
  
    (>>) :: m a -> m b -> m b  
    x >> y = x >>= \_ -> y
```

Let's distill those bad ass type signatures:

> **return** - much like `pure` from [Applicative Functors](/2012/12/02/monads-in-small-bites-part-ii-applicative-functors/), `return` is responsible for wrapping a value of type `a` into a minimum context Monad that yields a value of type `a` - referred to as a *monadic value*.

> **(>>=)** - often called `bind` - is a function of two arguments. The first is a *monadic value* of type `a` and the second is a function that receives a value of type `a` and returns a monadic value of type `m b` which is also the overall result of the function. 

> In other words: `bind` *runs* the monad `m a`, feeding the yielded `a` value into the function it received as an argument - any context carried by that monad will be taken into account.

> **(>>)** - often called `then` - This function receives two monads, `m a` and `m b`, and returns a monad of type `m b`. It is generally used when you're interested in the side effects - the context - carried out by the monad `m a` but doesn't care about the value `a` it yields.  It's rarely implemented in specific monads because the type class provides a default implementation:

> It applies `bind` to the monad `x` and a function that ignores its argument (`\_ -> y`) - which by convention is represented by an *underscore* - and simply yields the monad `y`: that's the final result of the computation. 

I won't be implementing `then` in Clojure though - I'll focus on `return` and `bind`, since `then` is essentially a helper function you could write yourself.

#### The Maybe Monad - Clojure edition

With definitions out of the way, let's implement the Clojure version of the Maybe monad.

```clojure
(def maybe-monad {
    :return (fn [v] v)
    :bind (fn [mv f] 
            (if mv
                (f mv)
                nil))})
```

Yup. That's *it*.

For the maybe monad, all its context needs to represent is a single value or the absence of value. We do this inside `bind` by checking if the monadic value `mv` is `nil`. If it isn't, we apply `f` to it, which will yield another monadic value. If, on the other hand, `mv` IS `nil`, we just return `nil`, bypassing the function application entirely.

`return`, as we saw, wraps a value into a minimal monad. In this case this is the value itself, so we just return it untouched.

This is how one may go about using it:

```clojure
(-> another-order
    ((:bind maybe-monad)
     (fn [order]
       (-> (calculate-shipping-rate (:address order))
           ((:bind maybe-monad)
            (fn [shipping-rate]
              (-> (lookup-discount-code (:discount-code order))
                  ((:bind maybe-monad)
                   (fn [discount]
                     ((:return maybe-monad)
                      (-> order 
                        (apply-shipping-costs shipping-rate) (
                        apply-discount-code discount) 
                        (place))))))))))))
;; nil
```

*WOW!* That is awful! And I won't blame you for not wanting to read through this aberration. But trust me, it does the job.

However, you're probably thinking: that looks *nothing* like the nice little `domonad` notation we saw earlier!

Well, you're right. That's because `domonad` is a [macro](http://clojure.org/macros) - it gives us some syntactic sugar that expands into the real code shown above. In order to be able to use the `domonad` notation, paste the following into your REPL:

```clojure
(defn monad-steps
    ([monad steps expr]
        (if (seq steps)
            (let [fst (first steps)
                  snd (second steps)]
                  `((:bind ~monad) 
                    (fn [~(symbol fst)]
                        (->  ~snd ~(monad-steps monad (subvec steps 2) expr)))))
            expr)))


(defmacro domonad [monad steps expr]
    (let [args (map first (partition 2 steps))
          forms (map second (partition 2 steps))
          new-steps (subvec (vec (interleave (cons nil args) forms)) 2)]
          `(let [m# ~monad]  
            (-> ~(second steps)
                ~(monad-steps monad new-steps 
                    `((:bind ~monad) (fn [~(symbol (last args))] ((:return ~monad) ~expr))))))))
```

All set! Now you should be able to run the examples that use `domonad` without any hiccups. Give it a shot:

```clojure
(domonad maybe-monad
    [order another-order
     shipping-rate (calculate-shipping-rate (:address order))
     discount (lookup-discount-code (:discount-code order))]
     (-> order
        (apply-shipping-costs shipping-rate)
        (apply-discount-code discount)
        (place)))
;; nil
```

> **Note:** macros can be daunting at times so don't worry too much about its implementation. It's way more important to me that you understand the end result than it is to be able to implement the macro yourself - but by all means dissect this implementation if you feel inclined to do so :)

Now that's way better. The *maybe* monad abstracted away the logic behind computations that can fail so you don't have to worry about it in your functions  - you can just focus on writing them. 

In the end I also believe it aids readability once you get used to it.

### Don't break the law

Monads have laws of their own too! Let's have a look at them.


#### Right unit

> Binding a monadic value `m` to `return` should be equal to `m` itself

In Haskell speak:

```haskell
m >>= return =  m
```

The proof in Clojure:

```clojure
(def m 10)
(def >>= (:bind maybe-monad))
(def return (:return maybe-monad))

;; given the above, this...
(>>= m return) ;; 10

;;is the same as
m ;; 10
```

#### Left unit

> Applying `return` to `x` and then applying `>>=` to the resulting value and `f`should be the same as applying `f` directly to `x`

In Haskell speak:

```haskell
return x >>= f =  f x 
```

The proof in Clojure:

```clojure
(def x 10)
(def >>= (:bind maybe-monad))
(def return (:return maybe-monad))
(def f (fn [v] (* v 2)))

;; given the above, this...
(-> (return x) (>>= f)) ;; 20

;;is the same as
(f x) ;; 20
```

#### Associativity

> Binding `m` to `f` and then applying `>>=` to the result and `g` should be the same as applying `>>=` to `m` and a function of argument `x` that first applies `f` to `x` and then binds it to `g`.

Phew...another mouthful, huh? Code should make it clearer. As usual, Haskell comes first:

```haskell
(m >>= f) >>= g  =  m >>= (\x -> f x >>= g)
```
And now let's prove it in Clojure:

```clojure
(def >>= (:bind maybe-monad))
(def return (:return maybe-monad))

(def m 10)
(def f (fn [v] (* v 2)))
(def g (fn [v] (+ v 10)))

;; given the above, this...
(-> (>>= m f) (>>= g)) ;; 30

;;is the same as
(>>= m
     (fn [x]
        (>>= (f x) g))) ;; 30
```

Alright, we're getting to the end now! Hold on just a little longer!

### One last thing - The List Monad


Yeah, I'm sure you saw this coming. Lists are monads too! I'll make this quick and show its implementation and usage in Clojure - bear with me one last time.

```clojure
(def list-monad {
    :return (fn [v] [v])
    :bind (fn [mv f] 
        (if (seq mv)
            (apply concat (map f mv))
            []))
    })

;;let's play with it

(domonad list-monad
    [a [1 2]
     b [a, (- a)]]
     (* 3 b))
;; (3 -3 6 -6)

(domonad list-monad
    [a [1 2]
     b []]
     (* 3 b))
;; () - an empty list. 
```

This should look familiar if you've used [list comprehensions](http://clojuredocs.org/clojure_core/clojure.core/for) in Clojure or other languages such as Python:

```clojure
(for [a [1 2]
     b [a, (- a)]]
     (* 3 b))
;; (3 -3 6 -6)
```

See? You've been using monads all along and didn't even know it! How awesome is that?

Also note that we didn't need to re-implement `domonad` for the list monad. It's a generic macro that will work with any monads you throw at it!

It's interesting to see how the list and the maybe monads differ. This time, `return` puts the value `v` inside a list and returns it because for lists, a minimum monad is a list with a single element.

`bind` is a bit more interesting. It first checks to see if `mv` is empty, in which case it returns an empty list, causing the whole computation to stop. If, however, `mv` is NOT empty, it maps `f` over every element in `mv`. 

The resulting list is potentially a list of lists, since functions fed to monads - such as `f` in this case - have to return monadic values. That's why we then apply `concat` to the resulting list, effectively flattening it.


### Final words

Hopefully you now have a much better understanding of Monads and should start seeing in your code use cases and/or opportunities for the monads shown here. 

You'll notice that this Clojure implementation of monads used only normal functions - that was by design since I wanted this implementation to be as close as possible to Clojure's [core.algo.monads](https://github.com/clojure/algo.monads) library. You should have a look at it.

Also, bear in mind that this tutorial is by no means exhaustive - there's **a lot** more about monads that I could possibly cover in a blog - it was hard enough ending it here! But if you want to study more about them, I'd recommend starting with these resources:

- [Learn You a Haskell for Great Good](http://learnyouahaskell.com/) - this book is an excellent intro to Haskell and it was the approach found there that made me grok monads - highly recommended and freely available online.

- [The Monads Section on the Haskell wikibook](http://en.wikibooks.org/wiki/Haskell/Understanding_monads) - another free online resource

That's it from me. I hope you enjoyed the read and if you made it until here, a big *thank you*.