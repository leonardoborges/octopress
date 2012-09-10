---
layout: post
title: "Clojure, leiningen 2 and Heroku: AOT compilation gotchas"
date: 2012-09-10 10:41
comments: true
categories: [clojure, functional-programming]
---

Recently I upgraded the [clojure](http://clojure.org/) project I'm working on to [Leiningen 2](http://leiningen.org/) in order to start using [nrepl](https://github.com/clojure/tools.nrepl) - since [swank-clojure](https://github.com/technomancy/swank-clojure) is now [deprecated](http://technomancy.us/163). Little did I know this would lead me to a small debugging adventure. 

### Heroku

I use [Heroku](http://www.heroku.com/) as my deployment platform and my project had been running on it for a few weeks without any
issues. I also use Heroku's PostgreSQL solution.

However, by upgrading to Leiningen 2, my project started throwing some weird exceptions during deployment -  it couldn't connect to my database any longer. Everything was fine on my local environment though.

Upon inspecting the deployment logs on Heroku, I realized that the [Heroku Clojure Buildpack](https://github.com/heroku/heroku-buildpack-clojure) tries to perform [Ahead of Time (AOT) compilation](http://clojure.org/compilation) if it identifies the project is running Leiningen 2.x

This means that after receiving a git push Heroku runs this command: 

```bash 
lein with-profile production compile :all
```

At first this might seem harmless. But step back for a second and think about what this means for code that depends on environment variables.

### Case in point: Lobos

[Lobos](https://github.com/budu/lobos) is a clojure library for manipulating database schemas akin [ActiveRecord migrations](http://api.rubyonrails.org/classes/ActiveRecord/Migration.html) in Ruby land.

The [real-world](https://github.com/budu/lobos#real-world-example) example on their github page recommends this code snippet for configuring a database connection:


```clojure
(ns lobos.config
  (:use lobos.connectivity))

(def db
  {:classname "org.postgresql.Driver"
   :subprotocol "postgresql"
   :user "test"
   :password "test123"
   :subname "//localhost:5432/test"})

(open-global db)
```

More often than not this isn't ideal. You'll probably want to customise your database configuration based on environment variables. Something along these lines:


```clojure
(ns lobos.config
  (:use lobos.connectivity))

(def db
     {:classname "org.postgresql.Driver"
      :subprotocol "postgresql"
      :user (System/getenv "DB_USER")
      :password (System/getenv "DB_PASSWORD")
      :subname (System/getenv "DATABASE_URL")})

(open-global db)
```

And you're good to go - until Heroku decides to AOT compile your code.

### Compilation

The Clojure compilation process caught me by surprise. As stated by [Rich Hickey](https://twitter.com/richhickey) on [this IRC log entry](http://clojure-log.n01se.net/date/2008-11-12.html#16:07), "a side effect of compiling Clojure code is loading the namespaces in order to make macros and functions they use available".

This means that during compilation, any top level function calls - such as `(open-global db)` from above - will get executed. The same applies to macros that expand to top-level function calls.

If said function call has no side effects, you're probably ok. 

The trouble with *open-global* though is that it effectively attempts a connection to the database! 

**BAM!**

On Heroku the environment variables you rely on - such as *DATABASE_URL* - aren't available on compile time meaning a connection will be attempted to an empty url. *All sorts of badness*.

After having understood why the compilation process was effectively executing top level function calls, I then wrapped that call to *open-global* in a function declaration:

```clojure
(defn init []
  (open-global (db)))
```

This will prevent eager evaulation during compilation since all `defn` does is bind the given function body to a *var*.

Then, in the code that needs migrations to happen - such as in my [midje](https://github.com/marick/Midje) tests setup - I initialize the database like this:

```clojure
(background
 (before :contents (lobos.config/init))
 (before :facts (migrate)
         :after (rollback :all)))
```

I spent some time in the #clojure IRC channel on freenode and it is accepted that top level function calls should be avoided anyway - especially if you're dealing with side effects.

I also applied the same principle to other initialization code I have such as the configuration for [Korma](http://sqlkorma.com/).

### Happy ending?

Not quite. After having spent quite some time to get to this point, this was the new error that was stealing my sleep: 

```bash
Caused by: java.lang.NullPointerException
    at java.util.concurrent.ConcurrentHashMap.hash(ConcurrentHashMap.java:332)
    at java.util.concurrent.ConcurrentHashMap.get(ConcurrentHashMap.java:987)
    at clojure.lang.Namespace.find(Namespace.java:188)
    at clojure.core$find_ns.invoke(core.clj:3659)
    at clojure.core$the_ns.invoke(core.clj:3691)
    at clojure.core$ns_name.invoke(core.clj:3698)
    at my_app.env__init.load(Unknown Source)
    at my_app.env__init.<clinit>(Unknown Source)
```

This isn't an unknown error. It basically means you're trying to find a namespace that doesn't exist. Or does it?

Once again this only happened if the code got compiled Ahead Of Time. Puzzling.

A couple of debugging hours later and I tracked it down to be a problem with [clj-logging-config](https://github.com/malcolmsparks/clj-logging-config) - a logging configuration utility for clojure. This is the offending code:

```clojure
(ns my-app.env
  (:use 
    clojure.tools.logging
    clj-logging-config.log4j))
(set-logger!) ;; this call causes the exception
```

`set-logger!` is a macro that makes use of the **\*ns\*** var. **\*ns\*** contains the current namespace and, due to what I'm assuming to be special semantics regarding this particular var, it shouldn't be [*unquoted*](http://clojuredocs.org/clojure_core/clojure.core/unquote) within a macro - which is why it was failing in AOT mode.

I [opened an issue](https://github.com/malcolmsparks/clj-logging-config/issues/15) in the clj-logging-config github page. I eventually forked the project and fixed the issue in [this pull request](https://github.com/malcolmsparks/clj-logging-config/pull/16). That was then [merged last Friday](https://github.com/malcolmsparks/clj-logging-config/commit/ec8a08535daad01eb9f23e92771b623b5902c8c9).

My code now works perfectly - regardless of AOT compilation.

### Final thoughts

This was a nice little journey and I'm actually glad I went through it. I have a much better understanding of Clojure's compilation process, worked out some quirks on Heroku and even got my first Clojure related open-source contribution accepted. Life is good.

Hopefully this will save other people a lot of time and effort debugging similar issues.