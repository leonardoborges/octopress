---
layout: post
title: "Functional composition with Monads, Kleislis and Functors"
date: 2014-06-17 11:00
comments: true
categories: [functional-programming, scala]
---

I've been learning Scala for my current client project and I find writing to be a great tool to test my understanding of any given topic. This means there might be a few Scala posts coming up soon as I keep learning interesting things. 

Today I'll be exploring a few different ways in which you can compose programs. I'll be using [Scalaz][1] in this post.

The examples that follow all deal with Vehicles - more specifically makes and parts:

```scala
  case class Make(id: Int, name: String)
  case class Part(id: Int, name: String)
```

Next we have a couple of functions which interact with these case classes:

```scala
  val make: (Int) => Make = (_) => Make(1, "Suzuki")

  val parts: Make => List[Part] = {
    case Make(1, _) => List(Part(1, "Gear Box"), Part(2, "Clutch cable"))
  }
```

So we have a function from `Int` to `Make` and then a function from `Make` to `List[Part]`. From set theory we know this implies we must have a function from `Int` to `List[Part]`. This is nothing more than simple function composition:

```scala
  val f = parts compose make
  f(1)
  // List[Part] = List(Part(1,Gear Box), Part(2,Clutch cable))
  
 // alternatively you can use 'andThen' which works like compose, but with the arguments flipped:
 val g = make andThen parts
 g(1)
 // List[Part] = List(Part(1,Gear Box), Part(2,Clutch cable))
```

Pretty boring stuff.

A more realistic example accounts for failure in our functions. One way we can encode this is using the `Option` data type:

```scala
val make  = (x: Int) => (x == 1).option(Make(1, "Suzuki"))

val parts = (x: Make) =>
  (x.id == 1).option(NonEmptyList(Part(1, "Gear Box"), Part(2, "Clutch cable")))
```

Now we have a function `make: Int => Option[Make]` and a function `parts: Make => Option[NonEmptyList[Part]]`. Based on our first example we should have a way to create a function from `Int` to `Option[NonEmptyList[Part]]`. This isn't immediately obvious however. 

While `make` does return a `Make`, it is wrapped inside an `Option` so we need to account for a possible failure. This leads to our first attempt:

```scala
  val f: Option[Make] => Option[NonEmptyList[Part]] = {
    case Some(m) => parts(m)
    case _ => None
  }
  val g = f compose make
  g(1) // Some(NonEmptyList(Part(1,Gear Box), Part(2,Clutch cable)))  
```

While this works, we had to manually create the plumbing between the two functions.  You can imagine that with different return and input types, this plubming would have to be rewritten over and over. 

All the function `f` above is doing is serving as an *adapter* for `parts`. It turns out there is a couple of ways in which this pattern can be generalised.

## Monadic bind

`Option` is a [monad][2] so we can define *f* using a for comprehension:

```scala
  val f = (x: Int) => for {
    m <- make(x)
    p <- parts(m)
  } yield p
  f(1) // Some(NonEmptyList(Part(1,Gear Box), Part(2,Clutch cable)))
```

Which is simply syntactic sugar for:

```scala
val g = make(_:Int) flatMap (m =>
    parts(m).map(p => p))
g(1)
// Some(NonEmptyList(Part(1,Gear Box), Part(2,Clutch cable)))

// you can also use the symbolic alias for 'bind', which makes it a lot nicer
val h = make(_:Int) >>= parts
h(1)
// Some(NonEmptyList(Part(1,Gear Box), Part(2,Clutch cable)))
```

The reason this is better is that `make` and `parts` could operate under a different monad but the client code would not need to change. In the example below, we're operating under the `List` monad:

```scala
val words: (String) => List[String] = _.split("""\s""").toList
val chars: String => List[Char] = _.toList

val f = (phrase: String) => for {
  m <- words(phrase)
  p <- chars(m)
} yield p

f("Motorcycles are fun to ride!")
// List(M, o, t, o, r, c, y, c, l, e, s, a, r, e, f, u, n, t, o, r, i, d, e, !)

// or even:
val g = words(_:String) flatMap (w =>
    chars(w).map(c => c))
g("Motorcycles are fun to ride!")
// List(M, o, t, o, r, c, y, c, l, e, s, a, r, e, f, u, n, t, o, r, i, d, e, !)
```

We used the exact same *for* comprehension syntax to compose these operations. This works because both `Option` and `List` are monads.

Notwithstanding, this still feels like unnecessary plumbing. All we are doing with the *for* comprehenstion / `flatMap` is extracting the values from their respective monads to simply put them back in. It would be nice if we could simply do something like `make compose parts` as we did in our first example.

## Kleisli Arrows

A [Kleisli arrow][3] is simply a *wrapper* for a function of type `A => F[B]`. This is the same type of the second argument to the monadic *bind* as [defined][4] in Scalaz:

```scala
bind[A, B](fa: F[A])(f: A => F[B]): F[B]
```

By creating a Kleisli arrow from a function, we are given a function that knows how to extract the value from a Monad `F` and feed it into the underlying function, much like bind does, but without actually having to do any binding yourself.

To use a concrete example, let's create a kleisli arrow from our `parts` function:

```scala
kleisli(parts)
// scalaz.Kleisli[Option,Make,scalaz.NonEmptyList[Part]]
```

You can read this type as being a function which knows how to get a value of type `Make` from the `Option` monad and will ultimately return an `Option[NonEmptyList[Part]]`. Now you might be asking, why would we want to wrap our functions in a kleisli arrow?  

By doing so, you have access to a number of useful functions defined in the Kleisli trait, one of which is `<==<` (aliased as `composeK`):

```scala
  val f = kleisli(parts) <==< make
  // same as   kleisli(parts) composeK make

  f(1) // Some(NonEmptyList(Part(1,Gear Box), Part(2,Clutch cable)))
```

This gives us the same result as the version using the *for* comprehension but with less work and with code that looks similar to simple function composition.

## Not there yet

One thing that was bugging me is the return type for `parts` above:

```scala
Make => Option[NonEmptyList[Part]]
```

Sure this works but since lists already represent non-deterministic results, one can make the point that the Option type there is reduntant since, for this example, we can treat both `None` and the empty List as the *absence of result*. Let's update the code:

```scala
val make  = (x: Int) => (x == 1).option(Make(1, "Suzuki"))

val parts: Make => List[Part] = {
  case Make(1, _) => List(Part(1, "Gear Box"), Part(2, "Clutch cable"))
  case _ => Nil
}
```

It seems we're in worse shape now! As before, `parts`'s input type doesn't line up with `make`'s return type. Not only that, they aren't even in the same monad anymore!

This clearly breaks our previous approach using a kleisli arrow to perform the composition. On the other hand it makes room for another approach: [Functor][5] *lifting*.


## Lifting

In Scala - and category theory - monads are [functors][5]. As such both `Option` and `List` have access to a set of useful functor combinators. The one we're interested in is called [lift][6].

Say you have a function `A => B` and you have a functor `F[A]`. Lifting is the name of the operation that transforms the function  `A => B` into a function of type `F[A] => F[B]`.

This sounds useful. Here are our function types again:

```scala
make: Int => Option[Make]
parts: Make => List[Part]
```

We can't get a function `Int => List[Part]` because `make` returns an `Option[Make]` meaning it can fail. We need to propagate this possibility in the composition. We can however *lift* `parts` into the `Option` monad, effectively changing its type from `Make => List[Part]` to `Option[Make] => Option[List[Part]]`:

```scala
val f = Functor[Option].lift(parts) compose make
f(1)
// Some(List(Part(1,Gear Box), Part(2,Clutch cable)))
```
`f` now has the type `Int => Option[List[Part]]` and we have once again successfully composed both functions without writing any plumbing code ourselves.

[Mark][7] pointed out to me that `lift` is pretty much the same as `map` but with the arguments reversed. So the example above can be more succintly expressed as:

```scala
val g = make(_:Int).map(parts)
g(1)
// Some(List(Part(1,Gear Box), Part(2,Clutch cable)))
```


## Summary

If you are only now getting to this whole Functor/Monad/Kleisli *thing* this was probably quite heavy to get through. The point I am trying to make here is that learning at least *some* of the abstractions provided by Scalaz is certainly worthwhile. They encode common patterns which we would otherwise keep re-writing a lot of the time. 

Special Thanks to [Mark Hibberd][7] who kindly reviewed an early draft of this post.

[1]: https://github.com/scalaz/scalaz
[2]: http://www.leonardoborges.com/writings/2012/12/08/monads-in-small-bites-part-iv-monads/
[3]: http://www.haskell.org/haskellwiki/Arrow_tutorial#Kleisli_Arrows
[4]: https://github.com/scalaz/scalaz/blob/scalaz-seven/core/src/main/scala/scalaz/Bind.scala#L16
[5]: http://www.leonardoborges.com/writings/2012/11/30/monads-in-small-bites-part-i-functors/
[6]: https://github.com/scalaz/scalaz/blob/scalaz-seven/core/src/main/scala/scalaz/Functor.scala#L31
[7]: https://twitter.com/markhibberd