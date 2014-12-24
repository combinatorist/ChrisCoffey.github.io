---
layout: post
title: "More State Machines"
modified:
categories: programming
description: **A** brief introduction and walkthrough of how to create FSM's with side effects (business logic) in the transitions
tags: [fsm, scala]
image:
  feature:
  credit:
  creditlink:
comments:
share:
date: 2014-12-23T15:35:27-05:00
---


In my last post about state machines I talked about how they can help provide simplifying assumptions for modeling your applications. However, all the simplifying assumptions aren't very helpful if you can't actually change your data. In this post I'll discuss guarded transitions & how to model them in Scala. 

##### The Basics
A basic state machine, especially the DFA we modeled in the previous post, transitions directly from the current state to the new state upon an input. However, how do we model this if we need to perform a calculation/ operation that may fail during the transition from state **A** -> **B**? The way I see it, you have to options:

- Hide the side effect in the transition and make the transition synchronous.
- Introduce an intermediate state **B'** representing the intent or attempt to transition from **A** -> **B**. 

In the first option, there's a big advantage in keeping your conceptual model clean. This makes it easier for others to consume. Unfortunately, as soon as you introduce guarded transitions this conceptual "clarity" actually turns into a muddy pool of non-determinism. This is because as soon as the possibility of failure is introduced you have the possibility that a transition from **A** -> **B** is no longer a DFA, but an NDFA (Non-Deterministic Finite Automata) because if the transition fails, then you'll actually wind up back in **A** (or a failure state), otherwise the machine transitions to **B**. As you can imagine, this is more difficult to reason about. In a Scala implementation this also means you don't know that the transition was requested in the event of a complete failure.

The alternative approach using an intermediate state means your diagram becomes much more complicated, with each intermediate state that may fail protected by a guard state. However daunting this pattern may seem initially, it's actually incredibly easy to reason about so long as developers all understand the pattern (just like any other design pattern). The major advantages here are that the FSM is always a DFA, i.e. you always know exactly the state you're in. If there is input that triggers a guarded transition it immediately transitions into the guard state, then returns. Yup, this means that you're always transitioning immediately then handling the forward/backward transition in a callback. The only allowed transitions from **B'** should be to **A** on failure or **B** on success. 

##### A Simple Scala Example
Bank accounts. Most of us have them (some prefer a shoe-box under the mattress though), and all of us have probably either used one as an example or read examples using them before so the mechanics should be pretty straight forward. 

To start with, lets think about what we'll need to model a DFA. To start with, we need the transition table and a domain to work with: 

{% highlight scala %}

trait DFAState

trait Result
trait Request

trait DFA[T, Input] {
	
	def transition(in: Input)(state: T): (Result, T)

}

trait TranRequest{
  val amount: Int
}

case class Credited(amount: Int) extends Result
case class Debited(amount: Int) extends Result
case class OverdrawApproved(amount: Int) extends Result
case class OverdrawRejected(amount: Int) extends Result

case class ApproveOverdraw(amount: Int) extends Request with TranRequest
case class RejectOverdraw(amount: Int) extends Result with TranRequest
case class Credit(amount: Int) extends Request with TranRequest
case class Debit(amount: Int) extends Result with TranRequest

trait AccountState extends DFAState
case object Open extends AccountState
case object Overdrawn extends AccountState

{% endhighlight %}

There's pretty much nothing actually going on here, but I've defined the basic structures we'll need for working with our bank account. We can attempt to either Credit or Debit the account, which will result in a credit or debit to the account. If we try to take too much money out of the account it triggers an overdraw, which needs to ask a hypothetical overdraw controller if we're allowed to do this. Given that we don't want to immediately allow the overdraw, hand out the money, then come back and decide "oops, probably shouldn't have done that", we'll need some intermediate state between Open & Overdrawn.

To that end I've created a simple wrapper state, *Guarded[T]* which wraps any other state and the initial state we were in. Pretty easy to rollback when you're carrying the initial state around w/ you, isn't it! Okay, lets look at some code:


{% highlight scala %}

case class Guarded[A <: AccountState, B <: AccountState](s: A, prev: B)

/**
 * A simple case class representing a bank account.
 *
 */
case class BankAccount(balance: Int, state: AccountState)

object BankAccount extends DFA[BankAccoun, TransactionRequest]{

	def transition(in: Input)(state: T): (Result, T)={
 		state.state match{
      case Open => {
      in match {
        case Credit(x) => (Credited(x), BankAccount(balancePostTransaction(state, in), Open))
        case Debit(x) if balancePostTransaction(state, in) >=0 => (Debited(x), BankAccount(balancePostTransaction(state, in), Open))
        case Debit(x) => (Debited(x), BankAccount(state.balance , Guarded(Overdrawn, Open)))
       }
      }
        case Guarded(Overdrawn, prev) => in match {
          // Allow the guarded transition to proceed
          case ApproveOverdraw(amt) => (OverdrawApproved(amt), BankAccount(balancePostTransaction(state, in), Overdrawn)
            // Reject the guarded transition and move back to prior state
            case RejectOverdraw(amt) => (OverdrawRejected(amt), state.copy( state = prev))
        }
        case Overdrawn => {
          case Credit(x) => ???
        } 
  }
  }

  private def balancePostTransaction(acct: BankAccount, tran: TranRequest): Int = {
    acct.balance + tran.amount 
  }


}

{% endhighlight %}

I tend to have a bad habit of throwing a lot of code up before speaking to it, and this is no exception. To start with, we have the *Guarded[A,B]* type described above, followed by a trivial context (state carrier) and the transition table. Most of the transitions are unrelated to what we're actually concerned with here, but they help to add some ambiance (or I got distracted and wrote them during this flight...). 

The really interesting piece and the entire reason for this post is the use of Guarded. You'll notice halfway through the transition table that attempts to overdraw don't result in any change to the account balance and they move the state to Guarded. This pattern allows a quick and easy rollback in the event of a failed overdraw, or a successful transition to Overdrawn with the new amount computed. 

What does this buy us? I'll explain.

##### Why use the *Guarded Transition* Pattern?
When compared to the alternative of wrapping a possibly failing computation up into a transition, the Guarded pattern has two main advantages. First, it forces a complete separation of concerns between the logic and the DFA model. For instance, a possible implementation of this may involve a parent service that orchestrates actions based on the DFA's response following a transition. This allows the service to wrap the transition up in a Future, execute it synchronously, or do anything else with it, all unbeknown to the DFA. Additionally if events are being persisted real-time w/ something like [Akka Persistence](link required) you'll know exactly what computation was in-progress if the system crashes. Secondly, the guarded transition pattern provides a simple structured error handling & rollback approach for potentially failing transitions. This is especially important if that transition involves a 3rd party service for something like authentication, lookup, etc...

