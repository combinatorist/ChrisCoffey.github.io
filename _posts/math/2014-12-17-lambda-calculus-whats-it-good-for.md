---
layout: post
title: "Lambda Calculus: What's It Good For?"
modified:
categories: math
description:
tags: [math, lambda calculus, functional programming]
image:
  feature:
  credit:
  creditlink:
comments:
share:
date: 2014-12-17T19:50:10-05:00
---

"What's the point of λ Calculus?"
	- Matt


I started this blog off with [Lambda Calculus for Government Majors](http://chriscoffey.github.io/math/beta-eta-and-lambda/) which covered the basic concepts & definitions of lambda calculus. But honestly, knowing what λ means doesn't really get you anywhere. My colleague Matt pointed this out after reading the piece, and he's right. So what can you do with λ calculus? I mean really, what's the point of this fancy algebra? Hopefully the following article explains why people get so excited about this stuff.

### Refresher

λa.a  =>  λ indicates this is a lambda expression/function
β reduction => replace a given λ expression with a common name
α conversion => create unique names for each variable to prevent collisions
η reduction => strip out unnecessary bound variables

For more info on these just take a peek back at my earlier post.

### Addition

Before getting into what you can do with λ calculus, I should probably show an example of how it actually works once it's been expanded. Expanding a λ expression is essentially replacing bound variables with provided arguments. This can end up throwing a ton of λ and parens at you, but it helps clarify the relationship between a reduced funtion & any data passed in. To start with, I'll take advantage of the fact that λ calculus is simple. So simple that it has no concept of numbers, addition, or anything else. This poses a bit of a problem if I'm trying to illustrate using λ for addition. I guess we'll just start at the beginning and define the natural numbers.

We know that for all natural numbers  *x* >= 0 that *x* + 1 is also a natural number. This means the notation we're all use to actually use for, say 5, is equivalent to 0+1+1+1+1+1. I'll use Church encoding (named after Alonzo Church, the inventor of λ calculus) to encode this property into valid λ functions.

{% highlight bash %}
	zero = λa.λn.n
	one = λa.λn.a n
	two = λa.λn.a (a n)
	three = λa.λn.a (a (a  n))
	....
{% endhighlight %}

What's really interesting about these numbers and pretty much every other λ expression you'll see is that it's a higher order function, or a function that takes another function. Those lambda expressions will grow pretty large pretty quickly, so I'll just be using normal numbers instead (developers are lazy after all). 

Are we ready for addition yet? We know how to create numbers, so what else do we need? It turns out not much:

{% highlight bash %}
	add = λx.λy.λa.λn.m a (y a x)
{% endhighlight %}

So now that I have a whole bunch of λ's & some parens, what's actually going on here? Let's use 1 + 1 as an example:

{% highlight bash %}
	add( one one) = ((λx.λy.λa.λn.x a (y a n) ) λc.λd.c d ) λ.e.λf.e f
	=> I performed an α conversion on the line above to avoid shadowing on 'one'
	((λc.λd.c d) a (y a n)) λe.λf.e f
	((λc.λd.c d) a ((λ.e.λf.e f) a n))
	c ((λe.λf.e f) a n)
	a (a n)
{% endhighlight %}

Who knew something as simple as 1 + 1 would expand into that! Thankfully this is the equivalent of doing binary arithmetic in assembler, i.e. something you won't need to do again. On the positive side this illustrates the power of function expansion using higher order functions. It takes a while to work through, and I'd strongly recommend working through the same example using 1 + 2 since you already know the answer is λa.λn.a (a (a n)).

### What Use Is This?
One word. Compilers. When you step back and think about it, λ Calculus is actually Turing Complete (hopefully that's more apparent after our foray into Church encoding), so it's a good fit for trying things out quickly. Anything that can compile down to λ Calculus (any Turing complete language) can then be translated up into another language for execution. So whenever someone says "every language is the same" you can show off your knowledge of λ calculus & isomorphisms between languages!

Hopefully this helped, but if you want to learn more check out these resources:

[Wikipedia](http://en.wikipedia.org/wiki/Lambda_calculus)

[Compiling Up to Lambda Calculus](http://matt.might.net/articles/compiling-up-to-lambda-calculus/)
 
