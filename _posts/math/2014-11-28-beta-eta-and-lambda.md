---
layout: post
title: "Lambda Calculus For The Government Major"
categories: math
description: A brief dive into the basics of Lambda Calculus
tags: [lambda calculus, math]
comments: false
share: false
date: 2014-11-28T18:25:59-05:00
---


Six months ago I decided to check out my first Haskell meetup at Akami's Cambridge office. Seeing how everyone keeps singing Haskell's praises and the meetup was only a block away I was locked in. After helping myself to the obligatory slice of pizza & bottle of water I sat down and cracked open my laptop, ready to take some notes & learn me some Haskell. The first talk was simple & understandable enough; a neuroscience PHD student had used Haskell to analyze his lab results & found the software transactional memory feature made concurrency easy for him. 
Between talks I congratulated myself for following his examples in such a mysterious language & even added *What's so confusing about this* into my notes. The second talk flew way over my head, rapidly diving through Lambda Calculus (what's that??) and moving into Inference Rules & Type Systems. Needless to say this talk left me reeling but motivated to learn what all of those Greek letters actually mean. The following post attempts to explain this to a liberal arts major like myself.

### Lambda (λ) Calculus

Lambda Calculus is really little more than 8th grade algebra. It is based on the concepts of **function abstraction** - using variables to represent functions - and **function application** - assigning variable names to particular values. So while these concepts are very simple on the surface they are incredibly powerful and versatile. In fact, you can represent virtually any construct from any programming language as a lambda expression, which means any language could be represented purely via lambda expressions.

### Function Abstraction

If you're reading this then odds are you're a programmer, have a passing interest in programming, or a friend of mine I conned into reading this. Regardless of your background, you probably remember learning in Pre-Algebra that *2x + 1 = y* plots a straight line for any value of *x* and *y*. In that little example *x* &  *y* are data variables that abstract away the x & y coordinates for an arbitrary point on the line. Function abstraction is what happens when we say *f(x)=2x +1*, so the equation for a line becomes *f(x)=y*. This abstraction has removed the concrete formula & provided a name in its place. This is the backbone of higher-order functions.

### Function Application

Continuing with the example from above, *f(x)=y* gives us a lovely representation of our line, but ultimately we're going to want to plot the line & to do that we'll need to transform our formula into a series of (x,y) coordinates. The act of "specializing" a particular formula or *expression* by providing concrete values (in this case 2 _ +1) allows for eventual function evaluation. As you can see, abstraction & application work hand in hand.

### λ Expressions

A λ expression is one of three things: a variable name identifying function abstraction, a non-abstracted function, or the application of a function. The context-free grammar would be:

	Expression = Name | Function | Application
	Function = λName.Body
	Body = Expression
	Application = (Function Argument)

For example *λa.a λx.λy.y* is a function that sets a variable *a* equal to the function *λx.λy.y*, which you can tell reduces to *y*. So the example statement reduces to *y*.

When evaluating a λ expression there are two possible approaches, **applicative order** and **normal order**. Applicative order means the parameter is evaluated *before* being passed into the function, while normal order means the parameter is evaluated *after* being passed in. 

At this point you're probably thinking "great, so I can write long convoluted λ expressions. There must be a cleaner way". Thankfully, there is!

### Beta (β) Reductions

β reductions is the formal name for replacing a particular λ function with a name. So, the β reduction of *λa.a* could be *identity*, since *λa.a* is the identity function in λ calculus. So really all we've done is take the concept of function abstraction discussed above & given it a fancy name & a Greek letter. Conceptually, this is the exact same as saying *f(a) = a*.

### Alpha (α) Conversions

Now that we have the power to introduce abstraction at arbitrary places in our λ expressions, you're probably concerned about naming collisions. After all, I could easily introduce a free variable with the same name as a bound variable, so how can I tell them apart? Thankfully α conversions enforce consistent renaming across all free instances of  a variable. This is a complex name for the incredibly simple concept of choosing unique names for variables to avoid confusion.

### Eta (η) Reductions

η reductions simplify an expression by removing unnecessary bound variables from the expression for instance, a statement *λa.(expr a) * will reduce to *expr * via an η reduction. This process strips out unnecessary variables from a λ expression to clarify the meaning.


In no way have I attempted to provide a comprehensive overview of λ calculus, but I do hope this gives people enough information to prevent them from being completely lost whenever these terms pop up in conversation. Thanks for reading and please contact me with any questions or corrections.



