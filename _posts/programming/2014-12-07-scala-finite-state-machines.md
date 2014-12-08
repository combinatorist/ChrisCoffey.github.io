---
layout: post
title: "Scala Finite State Machines"
modified:
categories: programming
description: Creating simple finite state machines using Scala's match syntaxt
tags: [design, Scala]
image:
  feature:
  credit:
  creditlink:
comments:
share:
date: 2014-12-07T16:19:59-05:00
---

I'm assuming the reader is already familiar with finite state machines, otherwise I strongly recommend reading about them. Personally, I find them easiest to read as diagrams rather than tables or strings. For example, here's a really simple diagram of a baby's life:

<figure>
<img src="/images/babyState.png" alt="">
<figcaption> Living the life... </figcaption>
</figure>

Babies only have a couple of states: Happy, Hungry, Sitting on someone's lap, or Sleeping. Different events may occur in the world that cause the baby to change states. Other events may not change the baby from one state to another, so the baby just continues along as it was.

Implementing this type of structure is really easy in Scala thanks to pattern matching & strong typing. Taking our baby example & translating it to code, we end up with this:

{% highlight scala %}
abstract class FsmState[T <: Enumeration](val state: T)

object BabyState extends Enumeration{
  type State = Value
  val Happy, Hungry, Sitting, Sleeping = Value
}
case class Baby(name: String, babyState: BabyState.State) extends FsmState(state =  babyState)

trait Event

case class Cried(id: String) extends Event
case class Ate(id: String) extends Event
case class Sat(id: String) extends Event
case class LaidDown(id: String) extends Event

object Baby{

  private def cloneNewState(d: Baby, s: BabyState.State) =
    d.copy(babyState = s)

  def transition(d: Baby, e: Event): Baby = {
    d.babyState match {
      case BabyState.Happy =>
        e match {
          case Cried(x) => cloneNewState(d, BabyState.Hungry)
          case Sat(x) => cloneNewState(d, BabyState.Sitting)
          case _ => d
        }
      case BabyState.Hungry =>
        e match {
          case Ate(x) => cloneNewState(d, BabyState.Happy)
          case _ => d
        }
      case BabyState.Sitting =>
        e match{
          case LaidDown(x) => cloneNewState(d, BabyState.Sleeping)
          case Cried(x) => cloneNewState(d, BabyState.Hungry)
          case _ => d
        }
      case BabyState.Sleeping =>
        e match{
          case Cried(x) => cloneNewState(d, BabyState.Happy)
          case _ => d
        }
      case _ => d
    }
  }

}
{% endhighlight %}

Okay, so that's a little more than some circles & arrows, but hopefully not too difficult to look at. To start with, I've defined an abstract trait  {% highlight scala %} FsmState[T <: Enumeration] {% endhighlight %} which specifies that you'll need to define a state enumeration & that the FSM state may only ever be in one state (pretty obvious). Next I define the basic case class to contain our state & have it extend {% highlight scala %} FsmState[T <: Enumeration] {% endhighlight %}.

Now that I have a domain object & some states, its time to build the actual state machine. Like I mentioned above, pattern matching on the Event classes make it incredibly simple to construct the full machine. First, you match on the current {% highlight scala %} BabyState.State {% endhighlight %}, then either allow a transition to a new state or drop through to the current state if no transition is allowed. Conversely you could return the error-handling type of your choice in the fall-through case. 

This machine is trivially simple & doesn't contain any business logic, but I'll dive into how to add business logic to the transitions in the next post.


* Special thanks to my 4-month old niece for the FSM inspiration.