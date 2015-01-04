---
layout: post
title: "Primitive Power"
modified:
categories: math
description:
tags: []
image:
  feature:
  credit:
  creditlink:
comments:
share:
date: 2015-01-04T16:15:01-06:00
---

Flying back from a nice 2 week vacation in Texas & I had a chance to think about primative operators. Specifically in this post I'll be exploring what a primative opeartor actually is and whether there's such a thing as "more powerful" primatives.

To get started I'll be working with **map**a function we all hold near and dear. **Map**'s signature takes a value of some type **A** and a function **A => B**. So the full signature would be:

{% highlight scala %}
def map[A, B](x: A)(f: A => B) : B
{% endhighlight %} 

This function has as it's range and domain all possible morphisms across two types. For instance, based on signature alone **map** can transform an **Int** into a **Dinosaur** via some magical function **f(i: Int)**. However, what it cannot do is transform an **Int** and a **String** into a **Dinosaur**. You could accomplish this via a **Tuple[Int, String]** and an **f(t: (Int, String))** but because you are still acting on a single parameter you're unable to partially apply **map** to the tuple. Really this means we've been forced into evaluating all or none of the parameters with no ability to choose between them. That seems like an unnecessary constraint for a morphism.

This suggests there exists some more "powerful" version of **map** that allows us to partially apply across multiple parameters and evaluate them independently. For lack of a better name lets call this **map2**. **Map2** accepts two types of parameters & returns a third, so we know it's signature looks like this:

{% highlight scala %}
def map2[A,B,C](a: A, b: B)(f: (A, B) => C): C
{% endhighlight %} 

**Map2** is able to handle anything **map** can do by providing a no-op type as **B**'s type argument. For example, here's **map** implemented via **map2**:

{% highlight scala %}
def map[A, B](a: A)(f: A => B): B = 
	map2(a, NoOp)((x, y) => f(x))
{% endhighlight %}

This on it's own buys us alot of power because we no longer need to worry about actually applying the computation for the morphism in **map**, instead we can defer it into **map2**. The next question I have about **Map2** is whether we can create a function **Map3** that is again more powerful than **Map2**? Logically it would seem that since **Map2**'s domain and range are strictly supersets of the domain and range for **Map** that it holds **Map2** should be a subset of **Map3**, right?

It turns out this isn't quite the case on the implemetation level. You can actually implement all other arity mapping functions in terms of **Map2** by using a trick simmilar to have a right fold operates. That is to say via building up a chain of function calls to provide the proper arity. Here's **Map3**:


{% highlight scala %}
def map3[A, B, C, D](a: A, b: B, c: C)(f: (A, B, C) => D): D = 
	map2(a, b)((x, y) => map2(c, NoOp)(f(a, b, c)))
{% endhighlight %}

This ends up creating a chain of map2 calls that can be composed together as **f • g • ...** to generate the full chain. 

As I think about it, it is actually possible to use the same approach to construct a similarly deep function call chain for **map**, allowing us to use **map** as the base primative and implement all other arity **map**'s in terms of the base primitive.

With standard Map it's still impossible to use a single function call to support >1 parameter, where as **map** with arity > 1 inherently supports this

I'll update this post once I've had a chance to do some more research on which of the two functions is actually the base primitive..
