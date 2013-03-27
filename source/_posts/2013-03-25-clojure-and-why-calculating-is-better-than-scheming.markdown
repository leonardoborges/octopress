---
layout: post
title: "Clojure and 'Why calculating is better than scheming'"
date: 2013-03-26 11:00
comments: true
categories: [clojure, functional-programming, haskell]
---

Last week while attending [Clojure/West][1] in Portland I came across a paper called [Why calculating is better than scheming][2]. In a nutshell, this paper is a critique to [Abelson][5] and [Sussman][6]'s classic textbook [SICP - Structure and Interpretation of Computer Programs][3], 
used by MIT for many years to teach their introductory programming course.

If you haven't read [SICP][3], you should. It's an amazing book. It uses [Scheme][4], a dialect of Lisp, as the vehicle to present fundamental programming concepts.

[Philip Wadler][9] - the author of this particular paper - contrasts teaching in [Scheme][4] to teaching using [KRC][7] and [Miranda][8], pointing out
four major features he considers important and lacking in [Scheme][4]. They are:

* Pattern matching
* A syntax close to traditional mathematical notation
* A static type discipline and user-defined types
* Lazy Evaluation

> Note:  [KRC][7] influenced [Miranda][8] which in turn influenced [Haskell][11].  Their syntax is similiar, so where Wadler used Miranda code snippets in the paper, I'll be using Haskell in this post.

As an aside, although the paper talks specifically of [Scheme][4], the term Lisp is used quite loosely and could lead the not-so-careful reader to be misled regarding a whole family of languages. Lisps have come a long way and modern dialects - of which I'll be focusing on [Clojure][10] - address many of the concerns raised by [Wadler][9].

Let us begin.

### Pattern Matching

Here Clojure, and most - all? - Lisps, are out of luck.

The example used in the paper is that of summing all integers in a list. First in Haskell:

```haskell
sum []   = 0
sum x:xs = x + sum xs
```

Now in Clojure:

```clojure
(defn sum [coll]
	(if (empty? coll)
		0
		(+ (first coll) (sum (rest coll)))))
```

The question here is this: Which snippet is easier to read/reason about? the Haskell code!

I must confess that I, too, miss pattern matching sometimes. However we can still improve our Clojure version to read nicer on the eyes by using [destructuring][11]:

```clojure
(defn sum [[first & rest :as coll]]
	(if (empty? coll)
		0
		(+ first (sum rest))))
```
And that's pretty much it. Without proper pattern matching, we can't get much better than that.

In addition to the Haskell snippet being easier to read, it's also easier to prove correct by structural induction, as demonstrated in Wadler's paper.

> Note: [core.match][16] adds support to pattern matching in Clojure. At the time of this writing, it's considered "alpha quality"

### Data structures

The paper continues to discuss exercise 2-27 from the [SICP][3], where the reader has to write code to represent a binary mobile, which consists of a left and right branch with each branch being a rod of certain length, from which hangs either a weight or another binary mobile.

Translating the Scheme example to Clojure, such a structure is represented using lists, like so:

```clojure
(defn make-mobile [left right] 
	(list left right))
	
(defn make-branch [length structure]
	(list length structure))
```

Wadler then contrasts this with the equivalent Miranda code, translated below to Haskell, taking advantage of [algebraic data types][17]:

```haskell
data Structure = Weight Int | Mobile Branch Branch
data Branch = Branch Int Structure
```
The first claim is that the Haskell/Miranda data type declaration makes it clearer what the data structure looks like, which is fair.

Also, the compiler can catch errors early on.

However, when writing idiomatic Clojure code, here's how I'd actually create this structure:

```clojure
(defn make-mobile [left right] 
	{:left left :right right})
	
(defn make-branch [length structure]
	{:length length :structure structure})
```

Granted, it's still not as clear and the compiler can't validate the shape of our data structure.

This is however cleaner than the previous version and drives home the point that Clojure isn't limited to lists, having literals for other data types such as the hash maps used in this example.

The second part of this claim is that through custom data types and pattern mathing, extracting values from those structures becomes simpler:

```haskell
totalWeight (Weight w) = w
totalWeight (Mobile l r) = totalWeightBranch l + totalWeightBranch r

totalWeightBranch(Branch d s) = totalWeight s
```

Once again Clojure can improve things by taking advantage of its builtin data structures and destructuring:

```clojure
(defn total-weight [{:keys [left right] :as structure}]
	(if (number? structure)
		structure
		(+ (total-weight-branch left)
		   (total-weight-branch right))))
		   
(defn total-weight-branch [{structure :structure}]
	(total-weight structure))		   
```

For a language with no pattern matching nor algebraic data types, this snippet is clear, concise and elegant - and a real improvement 
over the Scheme version discussed in the paper - which was essentially handicapped by the use of lists to simulate 'structs'.

As far as Clojure goes, this claim ends here: the next point in the paper, about changing from using `list` to using `cons`, is rendered moot since
we're using hash maps to represent our mobiles.


### Lisp lists are not self-escaping

Creating lists in Clojure goes like this:

```clojure
(list (list 1 2) nil) ;; ((1 2) nil)

'((1 2) nil) ;; ((1 2) nil)
```

Both statements above are equivalent, with the second one being clearly more concise.

The claim here is that the fact that you need to either use the `list` function or quote the form is cumbersome and can be confusing to beginners.

Clojure solves this by providing literals to another data structure - vectors:

```clojure
[[1 2] nil]
```

Simple and concise - in fact, in idiomatic Clojure code, you'll rarely see quoted lists where a vector will do.

This is possible because both lists and vectors conform to a higher level abstraction called a [Seq][12], in terms of which most list
operations are defined.

This eliminates the two following points mentioned in the paper as it allows a beginner to defer his/her understanding of quoted forms
to more advanced lessons/usages.

### Programs that Manipulate Programs - the interpreter example

Here Wadler shows a simple grammar for an interpreter in both Miranda and Scheme.

He claims that since Haskell/Miranda have free data types, representing such grammar becomes simpler:

```haskell
data Term = Var [Char]
			| Lambda Var Term
			| Apply Term Term
			| Closure Env Var Term
type Env = [(Var, Term)] 
type Var = [Char]
```

This is true in that one can easily scan the snippet above and deduce quickly what `Term` looks like.

Then, by using pattern matching, `eval` could be implemented like so:

```haskell
eval e (Var v) = lookup e v
eval e (Lambda v t) = Closure e v t
eval e (Apply t0 t1) = apply (eval e t0) (eval e t1)

...
```

I do believe this makes Haskell an excellent choice for writing interpreters and compilers. 

However, the flip side is that entering such terms in Haskell is cumbersome. Consider the term below:


> (λx.(x x)) (λx.(x x))

This is how to represent this term using the grammar defined above:


```haskell
(apply (Lambda "x" (apply (Var "x") (Var "x"))) 
       (Lambda "x" (apply (Var "x") (Var "x"))))
```

The strength in Lisp lies elsewhere. Since we have quoted forms, entering a similar term is a lot less verbose and closer to its intended representation:

```clojure
'((lambda [x] (x x)) (lambda [x] (x x)))
```

This is, of course, at the expense of making `eval` a more complicated function:

```clojure
(defn eval [e t] 
  (cond (variable? t)
        (lookup e (variable-name t))
        (lambda? t)
        (make-closure e (lambda-var t) (lambda-body t))
        (apply? t)
        (apply (eval e (apply-operator t))
               (eval e (apply-operand t)))))
               
...
```

The paper leaves out an important advantage of Lisps though:

Because we can write code for our made up language directly in its (almost)abstract syntax tree form, Lisps are the ideal choice when writing [Internal Domain Specific Languages][15].


### Lazy Evaluation

#### Lists

Haskell and Miranda are lazy languages and that yields a lot of power. This claim is more specific to the use of lazy lists - or sequences, streams - and starts off with a snippet that calculates the sum of squares of all odd numbers from 1 up to 100:

```haskell
sum [ i*i | i <- [1..100], odd i ]
```

What follows in the paper is a not-so-clear snippet of equivalent functionality using Scheme streams.

Clojure features lazy sequences and list comprehensions, making the above Haskell example trivial to write:

```clojure
(sum (for [i (range 100) :when (odd? i)] (* i i)))
```

If you're following at home with the original paper you'll see this is more readable and elegant than the equivalent Scheme example.

Another - also idiomatic - way to write the same expression is by using a combination of map/filter:

```clojure
(sum (map #(* % %) (filter odd? (range 100))))
```

Deciding which one is clearer is left as an exercise to the reader.

#### Special forms and lazy evaluation

In this section, Wadler brings another example from SICP where the reader wishes to implement his/her own `if` form.

As we know, in order to implement our own version of `if`, we need to use macros. That is because in Lisps arguments to functions are eagerly evaluated.

One might implement it like this:

```clojure
(defmacro my-if [pred then else]
  `(cond ~pred ~then
  		:else ~else))
```

In Lazy languages, such as Haskell and Miranda, this problem doesn't occur allowing such functions to be defined without the need for special and/or quoted forms:

```haskell
myIf True  t e = t
myIf False t e = e
```

However this completely dismisses the power of macros which allow you to extend the language in ways no other language allows - as is extensively demonstrated in books such as [On Lisp][18] and [Let Over Lambda][13].

As [Guy Steele][14] once put it:  *"[…] If you give someone Lisp, he has any language he pleases"*

### Conclusion

Hopefully this post doesn't come off as trying to invalidate Wadler's paper - that is not my intention.

While I do think a few of the points discussed are only applicable to the domain in which his paper was written - teaching - they are still valid and worth understanding.

I do however expect to have given you a different perspective on it, showing the strengths of modern Lisps such as Clojure and how it approaches these issues - such as by using its rich set of data structures, literals and techniques such as destructuring.


[1]: http://clojurewest.org
[2]: http://www.cs.kent.ac.uk/people/staff/dat/miranda/wadler87.pdf
[3]: http://mitpress.mit.edu/sicp/
[4]: http://en.wikipedia.org/wiki/Scheme_(programming_language)
[5]: http://en.wikipedia.org/wiki/Hal_Abelson
[6]: http://en.wikipedia.org/wiki/Gerald_Jay_Sussman
[7]: http://en.wikipedia.org/wiki/Kent_Recursive_Calculator
[8]: http://en.wikipedia.org/wiki/Miranda_(programming_language)
[9]: http://homepages.inf.ed.ac.uk/wadler/
[10]: http://clojure.org/
[11]: http://www.haskell.org/haskellwiki/Haskell
[12]: http://clojure.org/sequences#Sequences-The%20Seq%20library-Seq%20in,%20Seq%20out
[13]: http://amzn.to/WKpMZA
[14]: http://en.wikipedia.org/wiki/Guy_L._Steele,_Jr.
[15]: http://martinfowler.com/bliki/InternalDslStyle.html
[16]: https://github.com/clojure/core.match
[17]: http://www.haskell.org/haskellwiki/Algebraic_data_type
[18]: http://amzn.to/14mrrbk