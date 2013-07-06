---
layout: post
title: "Clojure, core.async and the Lisp advantage"
date: 2013-07-06 18:50
comments: true
categories: [clojure, concurrency]
---

Long gone are the days when systems needed to do only one thing at a time.

Concurrency is the word but it often leads to complex code, dealing with locks, mutexes etc…

There are several different abstractions which allows us to both model and think about asynchronous code in a more sane fashion: futures, promises and events/callbacks are but a few of them.

I won't get into the merits - or lack thereof - of these alternatives in this post but rather focus on a different alternative: [Communicating Sequential Processes - CSP](http://en.wikipedia.org/wiki/Communicating_sequential_processes).

### CSP and Go

CSP isn't new. It was first described in 1978 by [Tony Hoare](http://en.wikipedia.org/wiki/C._A._R._Hoare) and languages such as [Occam](http://bit.ly/14rEwxU) and [Erlang](http://bit.ly/14rEAh7) stem from it. 

It has however gained momentum by being natively supported by the [Go programming language](http://bit.ly/11ZvMzj).

I haven't read Hoare's paper so I'll use a little bit of what I know about Go's implementation of CSP.

Go introduced the concept of a `goroutine`. It *looks* like a function call:

```go
// doing some stuff...
go myFunction("argument") //does stuff in the background...
//continuing about my business...
```

What this does is it creates a *lightweight* process and returns control immediately to the caller. 

It is *lightweight* because it doesn't map 1-1 to native OS threads.

The reasoning behind it is that creating too many threads can bring your machine (or VM) down due to the amount of stack allocated to each one.

`goroutines` are cheap to create so you can have hundreds of thousands of them, and the runtime will multiplex them into a thread pool.

The immediate advantage is that it is dead simple to achieve a higher degree of concurrency. 

So far it sounds awfully similar to using futures with a pre-configured thread pool and a bit of syntactic sugar. But this is not the end of it.

### Communication

`goroutines` really shine when your several lightweight processes need to talk to each other. This is where a new abstraction comes into play:`channels`. 

Channels are first-class citizens - meaning you can pass them as arguments to functions as well as the return value of functions. 

Using them is straightforward:

```go
c := make(chan string)
go func() { 
  time.Sleep(time.Duration(5000) * time.Millisecond)
  c <- "Leo" 
}()
fmt.Printf("Hello: %s\n", <-c) //this will block until the channel has something to give us 
```

The code above creates a goroutine from an anonymous-executing function that will, in the background, sleep for 5 seconds and then send the string `Leo` to the channel `c`. Since control is returned immediately after that, the call blocks on the next line where it's trying to read a value from the channel using the `<-c` statement.

### Lisp

"But What does all this have to do with Lisp?" - ah! I'm glad you asked.

It actually has more to do with [Clojure](http://clojure.org/) - and by extension, Lisp.

[Clojure](http://clojure.org/) is a modern Lisp for the JVM, built for concurrency. The core team recently released [core.async](https://github.com/clojure/core.async), a new library that adds support for asynchronous programming much in the same way Go does with goroutines.

To highlight the similarities, I'll show and translate a couple of the examples from Rob Pike's presentation, [Go Concurrency patterns](http://talks.golang.org/2012/concurrency.slide#1).

#### Setting the scene

Say you're Google. And you need to write code that takes user input, - a search string - hits 3 different search services, - web, images and video - aggregates the results and presents them to the user.

Since they are three different services, you wish to do this concurrently.

To simulate these services, Rob presented a function that has unpredictable performance based of a random amount of sleep, shown below:

```go
var (
    Web   = fakeSearch("web")
    Image = fakeSearch("image")
    Video = fakeSearch("video")
)

type Search func(query string) Result

func fakeSearch(kind string) Search {
        return func(query string) Result {
              time.Sleep(time.Duration(rand.Intn(100)) * time.Millisecond)
              return Result(fmt.Sprintf("%s result for %q\n", kind, query))
        }
}
```

This is one way we could write such function in Clojure:

```clojure
(defn fake-search [kind]
  (fn [query]
    (Thread/sleep (int (* (java.lang.Math/random) 1000)))
    (str kind " result for " query)))

(def web   (fake-search "Web"))
(def image (fake-search "Image"))
(def video (fake-search "Video"))
```

#### Google Search 2.0

The first example is the simple case: we hit the services concurrently, wait for them to respond and then return the results:

> This example is from [slide #46](http://talks.golang.org/2012/concurrency.slide#46).

```go
func Google(query string) (results []Result) {
    c := make(chan Result)
    go func() { c <- Web(query) } ()
    go func() { c <- Image(query) } ()
    go func() { c <- Video(query) } ()

    for i := 0; i < 3; i++ {
        result := <-c
        results = append(results, result)
    }
    return
}
```

And here's the Clojure version:

```clojure
(defn google2-0 [query]
  (let [c (chan)]
    (go (>! c (web query)))
    (go (>! c (image query)))
    (go (>! c (video query)))
    (reduce (fn [results _]
              (conj results (<!! (go (<! c)))))
            []
            (range 3))))

(google2-0 "Clojure")
;; prints ["Video result for Clojure" "Web result for Clojure" "Image result for Clojure"]
```

The [`>!`](http://clojure.github.io/core.async/#clojure.core.async/%3E!) operator puts a value into a channel inside a `go` form. The function then uses [`<!!`](http://clojure.github.io/core.async/#clojure.core.async/<!!) to block on the `c` channel until it gets a value we can use.

#### Google Search 2.1

This is virtually the same example but this time we do not wish to wait on slow servers. So we'll return whatever results we have after a pre-defined timeout.

> This example is from [slide #47](http://talks.golang.org/2012/concurrency.slide#47).

```go
c := make(chan Result)
go func() { c <- Web(query) } ()
go func() { c <- Image(query) } ()
go func() { c <- Video(query) } ()

timeout := time.After(80 * time.Millisecond)
for i := 0; i < 3; i++ {
    select {
    case result := <-c:
        results = append(results, result)
    case <-timeout:
        fmt.Println("timed out")
        return
    }
}
return
```

You'll notice the use of `select` here.

`select` waits on multiple channels and returns as soon as *any* of them has something to say. 

The trick of this example is that one of these channels times out, at which point you get the message "timed out", effectively moving on to the next iteration and ignoring that slow server(s) response.

We can express the same intent in Clojure as well:

```clojure
(defn google2-1 [query]
  (let [c (chan)
        t (timeout 500)]
    (go (>! c (web query)))
    (go (>! c (image query)))
    (go (>! c (video query)))
    
    (reduce (fn [results _]
              (conj results (first (alts!! [c t]))))
            []
            (range 3))))


(google2-1 "Clojure")
;; prints ["Video result for Clojure" nil nil]
```
Everything looks the same but we're using [`alts!!`](http://clojure.github.io/core.async/#clojure.core.async/alts!!) to wait on the channels `c` and `t` (the timeout channel). This is analogous to Go's `select` form in that it waits for any channel to receive a value or, in this case, to timeout.

Note the `nil` values. Those came from servers which did not respond in time and were simply ignored.

Effectively what this means is that each time you run this function you'll likely get different results, depending on how long the `fake-search` function takes to run.

Amazing, huh?

### The big deal

But here's the *big deal* about this: although [core.async](https://github.com/clojure/core.async) looks like it's deeply integrated into the language, it is *just* a library!

It's not a separate compiler. It's not a new language. And it's not a special version of Clojure.

Since Clojure supports macros - like all Lisps - the core team was able to create the syntax required to easily use [core.async](https://github.com/clojure/core.async). And that's the beauty of it!

*The Lisp advantage, once again.*

#### Clojure's advantage

Now one thing I haven't mentioned is that Clojure is particularly well suited for this - and in a way even more so than Go: Clojure is opinionated and favours immutability.

That means that when using channels - and in fact any type of concurrent programming - you can safely share your data structures between concurrent units of work. Since they're immutable, you can't shoot yourself in the foot.

One last thing: [core.async](https://github.com/clojure/core.async) states as one of its goals Clojurescript compatibility, bringing channel based concurrent programming to the browser. Exciting stuff.

### More on core.async

[core.async](https://github.com/clojure/core.async) is still in alpha but you are encouraged to take it for a spin. Documentation is still lacking so I recommend you look at:

- [Rich's blog post about it](http://clojure.com/blog/2013/06/28/clojure-core-async-channels.html)
- [The source code on github](https://github.com/clojure/core.async)
- [The walkthrough namespace](https://github.com/clojure/core.async/blob/master/examples/walkthrough.clj), which showcases its features.


Also, the full Clojure code I used here can be seen in [this gist](https://gist.github.com/leonardoborges/5924461).








