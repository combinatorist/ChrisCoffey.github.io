---
layout: post
title: "Building on Lists"
modified:
categories: programming
description: Lists are the fundamental building block for many algorithms
tags: [FP, data structures, Scala]
image:
  feature:
  credit:
  creditlink:
comments:
share:
date: 2014-12-15T06:24:26-05:00
---


Functional data structure are kind of a big deal, but what exactly makes them "functional"? After all, isn't any data structure that works functional? Sorry, bad joke. In fact, a functional data structure is one that, like functional code itself, does not mutate state. Instead, functional data data structures are modified by copying the affected cells of a data structure and making changes in the copy. Okay you say, what does this buy me over a Vector?

To illustrate some of the advantages to functional data structures lets walk through the linked list. Linked lists are, as Chris Okasaki points out in his excellent book [Purely Functional Data Structures](http://www.amazon.com/Purely-Functional-Structures-Chris-Okasaki/dp/0521663504), stacks. That is to say, you can add elements to the head of a list (push), or read off the head (pop). Both structures are head-optimized. So, what does a basic linked list implementation actually look like in Scala? 

{% highlight scala %}
abstract class List_

case object Nil_ extends List_

case class Cons_[A](head: A, tail: List_) extends List_

object List_{
  def::[A](data: A, ls: List_) = Cons_(data, ls)

  def isEmpty(ls: List_) = ls match{
    case Nil_ => true
    case _ => false
  }

  def head(ls: List_) = ls match{
    case Cons_(h, t) => Some(h)
    case _ => None
  }

  def tail(ls: List_) = ls match{
    case Cons_(_, t: Cons_) => Some(t)
    case _ => None
  }
}
{% endhighlight %}

In fairly idiomatic Scala, you'll start with a base interface List, then provide the empty case Nil. After that, every List is simply a bit of data added to the head of another List, aka Cons. Now lets imagine we want to append two lists together. With a mutable list you can just take the tail element of the list & set it as the head of the second list. However, with a functional list we don't want to mutate the other list. So, how do we concatenate both lists without mutating either list?

{% highlight scala %}
  
  def append(as: List_, bs: List_): List_ = as match{
    case Nil => bs
    case Cons_(h, t) => Cons_(h, append(t,bs))
  } 

{% endhighlight %}

As you can see, instead of modifying either list I've walked both lists & constructed a third list from their elements. It's the order of the traversal, from list A's head -> list B's tail that ensures the concatenation is in the correct order. There are a few other helper functions like *foldLeft* and *reverse* that we'll need in a moment, so lets add them as well:

{% highlight scala %}
  
 def foldLeft[T, A](ls: List_, seed: T)(f: (A, T)=> T): T =
    ls match{
      case Nil_ => seed
      case Cons_(h: A, t) => foldLeft(t, f(h, seed))(f)
      case _ => seed
    }

  def reverse[A](ls: List_) = ls match{
    case Nil_ => ls
    case Cons_(h:A, t) => foldLeft[List_, A](ls, Nil_)((x, seed) =>{
      seed match{
        case Nil_ => Cons_[A](h, Nil_)
        case Cons_(a:A, _) => Cons_[A](a, seed)
      }
    })
  }

{% endhighlight %}

Aside from the ugly syntax (easy enough to fix!) this should look somewhat similar to the standard library. Fold Left is still an eager tail-recursive fold, and reverse returns the same elements in opposite order.

Now that we have a solid linked list implementation, lets move one to creating everyone's favorite FIFO structure, the queue.

 Unlike stacks, which are great for parsing & similar algorithms, queues are well-suited for algorithms requiring time-ordering such as processing instructions in the order they were issued. In an imperative setting Queue's are often implemented using an Array or Vector with pointers to the current head & tail held internally. However, since this implementation relies exclusively on mutation, it's not going to suffice for our functional queue. Instead, we'll implement the queue as a pair of lists. The following code illustrates how to construct a basic functional queue:

{% highlight scala %}

case class Queue_[A](f: List_, r: List_)

object Queue_{

  def isEmpty(q: Queue_[_]) = q match{
    case Queue_(Nil_, Nil_) => true
    case _ => false
  }

  def enQ[A](a: A, q: Queue_[A]) = Queue_(q.f, Cons_(a, q.r))

  def deQ[A](q: Queue_[A]): (Option[A], Queue_[A]) = {
    q match{
      case x if isEmpty(x) => (None, q)
      case Queue_(_Nil, x) => {
        val newF = List_.reverse(x)
        (List_.head(newF), Queue_(List_.tail(newF), Nil_))
      }
      case Queue_(f, r) =>
    }
  }

}


{% endhighlight %}

Now that's certainly more interesting than just a plain linked list. Thanks to the extended List implementation I'm able to model the queue as two lists which swap places whenever the front (f) is empty. This means either *enQ* or *deQ* will be *O(n)* in the worst case since it will require a full traversal to preserve ordering. In my example, I chose to provide an *O(1)* insertion and leave *O(n)* worst case for removal. You'd need to profile or at least understand your workload better to determine whether this makes sense for a particular use case. 

For more reading I really can't recomend Chris Okasaki's book enough, but [Functional Programming In Scala](http://www.manning.com/bjarnason/) is also a great resource. Good luck & feel free to send me an email with any questions.