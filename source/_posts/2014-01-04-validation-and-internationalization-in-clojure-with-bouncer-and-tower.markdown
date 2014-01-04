---
layout: post
title: "Validation and Internationalization in Clojure with bouncer &amp; tower"
date: 2014-01-04 19:00
comments: true
categories: [clojure]
---

I released [bouncer](https://github.com/leonardoborges/bouncer) in April last year and since then it has had a small but steady growth in usage. 

So much so that I received some community feedback in the form of emails and pull requests which is simply *great*!

The latest and most substantial pull request, submitted by [Vadim Platonov](https://github.com/dm3), added the ability to customise validation messages in anyway you like, as you can see in the section [Internationalization and customised error messages](https://github.com/leonardoborges/bouncer#internationalization-and-customised-error-messages) of the [README](https://github.com/leonardoborges/bouncer/blob/master/README.md).

However the documentation only gives a simple example and while I believe it should show users how this opens up several different usage patterns, I thought I'd expand that in this post and integrate [bouncer](https://github.com/leonardoborges/bouncer) with the excellent Internationalization library [tower](https://github.com/ptaoussanis/tower) to show how validation and I18n could work together in this configuration.

> If you've never used bouncer before, I'd recommend you have a quick look at the [Basic Validations](https://github.com/leonardoborges/bouncer#basic-validations) section of the [README](https://github.com/leonardoborges/bouncer/blob/master/README.md) before continuing.

## Setup

The first thing you'll need to do is create a new leiningen project and add both [bouncer](https://github.com/leonardoborges/bouncer) and [tower](https://github.com/ptaoussanis/tower) as dependencies in your `project.clj`:

```clojure
[bouncer "0.3.1-beta1"]
[com.taoensso/tower "2.0.1"]
```

Next, in your `core` namespace - or wherever, really - require both libs and set up a dictionary to be used in the examples:

```clojure
(ns bouncer+tower.core
  (:require [taoensso.tower :as tower
             :refer (with-locale with-tscope t *locale*)]
            [bouncer.core :refer [validate]]
            [bouncer.validators :as v]))

(def my-tconfig
  {:dev-mode? true
   :fallback-locale :en
   :dictionary
   {:en
    {:person  {:name {:required "A person must have a name"}
               :age  {:number   "A person's age must be a number. You provided '%s'"}
               :address {:postcode {:required "Missing postcode in address"}}}
     :missing  "<Missing translation: [%1$s %2$s %3$s]>"}
    :pt-BR
    {:person  {:name {:required "Atributo Nome para Pessoa é obrigatório."}
               :age  {:number   "Atributo Idade para Pessoa deve ser um número. Valor recebido foi '%s'"}
               :address {:postcode {:required "Endereço deve ter um código postal"}}}
     :missing  "<Tradução ausente: [%1$s %2$s %3$s]>"}}})
```

Also, we need someting to validate so go ahead and create a map representing a person:

```clojure
(def person {:age "NaN"})
```

## Customising bouncer

Since `0.3.1-beta1`, [bouncer](https://github.com/leonardoborges/bouncer)'s `validate` function receives an optional first argument called *message-fn*. As the name implies, it is a function from *error metadata* to, most likely, a string. This is our opportunity to use tower's features to translate our error messages.

In order to accomplish that, we'll be using this message function:

```clojure
(defn message-fn
  "Receives a locale, tscope and a map containing error metadata.

  Uses this information to return a I18n'ed string"
  [locale tscope {:keys [path value]
                  {validator :validator} :metadata}]
  (let [tr-key (->> (flatten [path validator])
                    (map name)
                    (clojure.string/join "/")
                    keyword)]
    (with-tscope tscope
      (t locale my-tconfig tr-key value))))
```

In order to understand the function above, let's validate the person map using `identity` so we can inspect the error metadata that will be the third argument to this function:

```clojure
(validate identity
          person
          :name v/required)
          
;; [{:name
;;   ({:path [:name],
;;     :value nil,
;;     :args nil,
;;     :metadata
;;     {:optional false,
;;      :default-message-format "%s must be present",
;;      :validator :bouncer.validators/required},
;;     :message nil})}
;;  {:age "NaN",
;;   :bouncer.core/errors
;;   {:name
;;    ({:path [:name],
;;      :value nil,
;;      :args nil,
;;      :metadata
;;      {:optional false,
;;       :default-message-format "%s must be present",
;;       :validator :bouncer.validators/required},
;;      :message nil})}}]
```
As we're only validating the name in this case, that's all we get in the return value of `validate`. However you can see how we have all sorts of useful information now - hopefully this makes the `message-fn` code above easier to understand.

## Take it for a spin

We're now ready for a couple of examples, using two different locales. Let's get on with it:

```clojure
(validate (partial message-fn :en :person)
          person
          :name v/required
          :age  v/number
          [:address :postcode] v/required)


;; [{:address {:postcode ("Missing postcode in address")},
;;   :age ("A person's age must be a number. You provided 'NaN'"),
;;   :name ("A person must have a name")}
;;  {:age "NaN",
;;   :bouncer.core/errors
;;   {:address {:postcode ("Missing postcode in address")},
;;    :age ("A person's age must be a number. You provided 'NaN'"),
;;    :name ("A person must have a name")}}]
```

Now let's get some messages in portuguese, shall we?

```clojure
(validate (partial message-fn :pt-BR :person)
          person
          :name v/required
          :age  v/number
          [:address :postcode] v/required)


;; [{:address {:postcode ("Endereço deve ter um código postal")},
;;   :age
;;   ("Atributo Idade para Pessoa deve ser um número. Valor recebido foi 'NaN'"),
;;   :name ("Atributo Nome para Pessoa é obrigatório.")}
;;  {:age "NaN",
;;   :bouncer.core/errors
;;   {:address {:postcode ("Endereço deve ter um código postal")},
;;    :age
;;    ("Atributo Idade para Pessoa deve ser um número. Valor recebido foi 'NaN'"),
;;    :name ("Atributo Nome para Pessoa é obrigatório.")}}]
```

In case you're too lazy to do this all from scratch, I created a github repo containing this example ready for you to play with. [Go get it](https://github.com/leonardoborges/bouncer-tower).

## Conclusion

Hopefully this gives you a little taste of what you can do with the latest version of [bouncer](https://github.com/leonardoborges/bouncer). Remember this is but one of many ways you could integrate the library so get creative :)

Once again, thanks to [Vadim Platonov](https://github.com/dm3) for submitting this pull request.