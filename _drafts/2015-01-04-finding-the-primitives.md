---
layout: post
title: "Finding the Primitives"
modified:
categories: programming
description:
tags: [design, functional programming]
image:
  feature:
  credit:
  creditlink:
comments:
share:
date: 2015-01-02T22:09:20-05:00
---

*"A combinator library is a library that offers functions that combine fuctions together to make bigger functions"* 
- History of Haskell

fairly unclear how that maps to the real world

*"... a domain-specific language for describing values of a particular type"*

Combinators are sometimes called Domain specific embedded languages, because they are implemented/embeded within a particular "host" programming language.

Key advantage is separation of problem description from implementation of solution.

Once a DSL is implemented and bootstrapped in the initial host language it can be translated into additional languages without any loss in expressiveness, provided the host type system is injectie w/ respect to the target type system.

DSEL allows you to break y
