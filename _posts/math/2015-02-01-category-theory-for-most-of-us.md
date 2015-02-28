---
layout: post
title: "Learning Category Theory"
modified:
categories: math
description:
tags: [math, category theory]
image:
  feature:
  credit:
  creditlink:
comments:
share:
date: 2015-02-01T14:39:45-05:00
---

Eugene Yakota's [Learning ScalaZ](http://eed3si9n.com/learning-scalaz/) series of articles are some of the best resources available for folks starting to learn functional programming. I loved following his path through the ScalaZ library & Learn You a Haskell For Great Good, and it was one of the most valuable bits of reading I've done. However, I've been spending more and more time playing around with Haskell and it seems like in order to really understand what's going I should learn the theory behind it all. Category Theory to be exact.

## What Is Category Theory?

Stanford defines Category Theory as:

>  ... it is the general mathematical theory of structures and systems of structures.

Wikipedia prefers:

> Category theory is a mathematical structure ... What makes category theory different from the study of other structures is that in a sense the concept of category is an abstraction of a kind of mathematics. (This cannot be made into a precise mathematical definition!) This makes category theory unusually self-referential and capable of treating many of the same questions that mathematical logic treats. In particular, it provides a language that unifies many concepts in different parts of math.

There are dozens of other slight variations on this theme, but I think that's because Category Theory is still a fairly new branch of mathematics. At it's heart it's the study of the relationship between different areas of math.

But what is a category?

Categories have two main components:
- Objects
   - Objects are points/elements within the category
- Arrows
   - Arrows are a directed connection between objects in a given category.

<figure align="center">
<img src="/images/Scala Category.svg" alt="Categories are Awesome!">
<figcaption> The Category of Scala Types </figcaption>
</figure>

Imagine we have three types, *A, B, C* within a Scala program, and the ability to map from *C → B*. This could be something as simple as this:

{% highlight scala %}

object Test {
	type C = Int
	type B = BigDecimal 
	type A = String

	def c_→_b(c: C): B = BigDecimal(c)
	def b_→_a(b: B): A = b toString 
}

{% endhighlight %}

Obviously the actual category of Scala types is infinite, but in this simple example you can see category *T* contains a few objects and some arrows between them. Those arrows are each functions from object *O₀* to *O₁* and *O₁* to *O₂*, and as functions this means we can compose them together. Composition is awesome, not in the least because it allows us to add another implied arrow from *C* → *A* by composing *C → B → A*.

So now our category actually looks like this:

<figure align="center">
<img src="/images/Scala Category implied arrow.svg" alt="OOOhh, arrows can compose. This looks interesting!">
<figcaption>Composing Arrows</figcaption>
</figure>

Because arrows are also functions and we can compose this, you can legally write {% raw %} b_→_a compose  c_→_b {% raw %}, effectively creating a new function from *C → A*! Extending your code without actually having to write any new code is pretty awesome, and we know this is safe because the arrows in category T prove it (and the compiler + type system prove it in Scala).

## Arrow Composition Laws

Composing arrows follows the same basic laws as composing functions. That is to say, arrows are associative and have an identity.

1. Associativity
  -  Arrows composed together do not need to parenthesis. Ordering of arrows remains important, but where the parenthesis are placed doesn't matter.
  -     *a • b • c = a •(b * c) = (a • b) • c*

2. Identity
  -  All objects in a category have a self-referential arrow. This arrow provides a unit/zero/identity for elements in the category, just like 0 is the identity for addition/subtraction or 1 across multiplication/division. In a category, the identity arrow is an important unit of composition because it is both always present on any given object & provides a valid starting point when performing composition.

These composition laws allow us to work with Categories independently of any understanding about what the objects actually are. Instead, you're able to reason about how objects within a category relate to each other via their arrows. 
