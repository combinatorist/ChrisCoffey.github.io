---
layout: post
title: "Thoughts on Micro-Services"
modified:
categories: musings
description: Are Micro-Services a new Idea?
tags: [design, musings]
image:
  feature:
  credit:
  creditlink:
comments:
share:
date: 2015-01-11T22:09:20-05:00
---

# Microservices: Unix-style toolchains for the Internet age

A very good question if you ask me. Like monads, it seems that most everyone has a slightly different understanding of what a microservice actually is. So, while I'm going to stay out of the monad-definition cottage industry for now, I think its important to explore the overloaded & overly-buzzwordy world of microservices.

Wikipedia defines microservices as:

>... microservices is a software architecture design pattern in which complex applications are composed of small, independent processes communicating with each other using language-agnostic APIs. 
> These services are small, highly decoupled, and focus on doing a small task.

Like any good buzzword definition, this one is littered with other confusing or ambiguous terms. What exactly is a "software architecture design pattern" in this context? We've all read the GoF design patterns book but I'm pretty sure we never saw anything about microservices or SingletonVisitorMicroserviceFactory in there, so is this the same type of pattern? In this context I'm taking the term design pattern to mean more of a design philosophy about decoupling software. That fits nicely with the language-agnostic APIs, which is also confusing but I'm pretty sure it just means REST, HTTP, Thrift, or some other custom communication protocol. The second sentence is significantly clearer and offers a recipe for creating a microservice: keep it small, decoupled & focused. Perfect!

So is this a microservice?

{% highlight scala %}
object MyFirstService {
	
	def strToFoo(s: String): Future[Foo] = ???

}

{% end_highlight%}

Its small, decoupled, communicates via raw strings (assuming its fronted by some REST interface) & certainly focuses on doing just one small task. Unfortunately no, I don't think this is a microservie, or if it is then it has a truly awful API. This is really just a single function exposed to the world, which means it's either a *God* function that does everything or its too narrowly focused, and either way I think that means this isn't a real micro service. 

I think we need some more view points, so lets turn to Martin Fowler & the good folks at Thoughtworks for [their take on microservices](http://martinfowler.com/articles/microservices.html):

> The term "Microservice Architecture" has sprung up over the last few years to describe a particular way of designing software applications as suites of independently deployable services. While there is no precise definition of this architectural style, there are certain common characteristics around organization around business capability, automated deployment, intelligence in the endpoints, and decentralized control of languages and data.

This brief definition doesn't to his full article justice, but at its core it shows that micro services help manage complexity in some areas while lifting a significant amount of complexity from a classic 3-tier monolith's code into the operations/ Dev-Ops level. I think this is a spectacular step in the right direction, but I take umbrage with the idea that microservices are a relatively recent phenomenon. 

I don't pretend to be a software historian, but I do happen to know that Unix & Linux are built on top of a very old & very specialized set of tools. The -nix tool-chain adheres to the "do one thing well" philosophy & often exposes their results as a standardized output buffer. These outputs coupled with the venerable **|** allow a collection of specialized tools to be composed together in a clear and logical manner in order to accomplish a larger goal. These specific tools have all been around for ~30 years, and the philosophy goes back even further. 

And that hits on my own definition of microservices, which is roughly equivalent to Martins but hopefully much clearer: 
> Microservices are internet-connected tools adhering to the Unix "do one thing well" philosophy & sharing a common input/output language



