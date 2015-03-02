---
layout: post
title: "Parsing SQL DDL Schema With Parboiled2"
modified:
categories: programming
description:
tags: []
image:
  feature:
  credit:
  creditlink:
comments:
share:
date: 2015-02-28T13:48:06-05:00
---

Its been a long time since I've written anything, we just started a pivot at Leaf so things have been pretty crazy. But like any radical change, there have been a ton of new things to explore and learn from as well, so I suppose there's a silver lining in there.

As we're converting our code over to a new platform we've switched persistence from Kafka to Postgres which means there's a whole world of serialization code we need to write & test. Normally SQL databases are difficult to unit test and can only be tested by running integration tests that rely on programs running outside the scope of your project. For testing behavior, I still can't think of a better way but a colleauge's off-hand remark that its impossible to test *any* of the SQL code got me thinking on my commute home.

First off, what would the requirements be for testing SQL? What can you reasonably test given some code & the DDL for schema creation?

1. Needs to ensure column names are serialized correctly
2. Can't rely on any files outside of the project

That seems like a manageable set of requirements to start with. To start with, since the only thing I'm comparing are the column names, lets take a look at an example file:

{% highlight sql %}

--comment comment

create table tables (
  id int identity not null,
  label varchar(15) not null,
  location int not null
)

create table locations(
  id int identity not null,
  name varchar(15) not null,
  owner varchar(50) not null
)

-- more comments

{% endhighlight %}

Anyone would agree this is a really simple SQL schema, but does that mean it's really simple to parse? To help answer this question lets investigate the basic grammar for that SQL statement. I'm going to drop the **<** and **>** around all of the identifiers because I think they clutter the grammar. 

{% highlight scala %}
DDL         ::= Statements | Gap
Statements  ::= Statement | Gap Statement Statements
Gap         ::= Whitespace | Newline
Statement   ::= Operator "("  Arguments ")"
Operator    ::= "create table"
Arguments   ::= Arg | Arg "," Arguments
Arg         ::= Name DataType Nullable
Name        ::= text_literal
DataType    ::= Category | Category "(" Size ")"
Category    ::= "int" | "varchar"
Nullable    ::= "null" | "not null"
Whitespace  ::= Space | Tab
Space       ::= " "
Tab         ::= "\t"
NewLine     ::= "\n" | "\r"
{% endhighlight %}

This grammar says that a DDL file consists of 0 or more Statements, each separated by a gap of whitespace or a line break. Each statement is composed of one or more arguments, which maps directly onto our intuition that a schema consists of one or more tables, each made up of one or more columns. This isomorphism is valuable for understanding how exactly we'll parse this statement into something we can use to help with our unit testing. To start with, I'll need an object that mimics what I want for my column name testing and some of the tokens I'll be looking for.

{% highlight scala  %}

case class tbl(name: String, columns: Seq[String])

val Return = "\r"
val NewLine = "\n"
val Comma = ","
val Space = " "
val CreateTable = "table"

{% endhighlight %}

I chose to represent **CreateTable** as just the word *table* rather than *create table* as an optimization that I'll delve into later.

Now that the tokens are in place, its time to start building the elements of my SQL DDL grammar. [Parboiled2](https://github.com/sirthias/parboiled2), the excellent Scala parsing library by Mathias Doenitz & co. allows me to write code that closely mirrors the BNF grammar, which means as you write your grammar you can actually test it out real-time & the code should read just like the grammar. Pretty cool stuff, now lets write some code!

{% highlight scala %}

  case class DdlParser(input: ParserInput, columnStart: String) extends Parser {
    def DDL           = rule { Statements.*  }
    def Statements    = rule { Ignore ~ Table }
    def Table         = rule { TableFlag ~ TableName ~ Ignore ~ Arguments ~> tbl }
    def TableName     = rule { capture(!EndName ~ ANY).+ ~> (_.mkString("")) ~ EndName}
    def Arguments     = rule { Arg.*.separatedBy(Ignore) }
    def Arg           = rule { columnStart ~ capture(!Space ~ ANY).+ ~>(_.mkString("")) ~ Space}
    def TableFlag     = rule { CreateTable ~ Space }
    def EndName       = rule { Space | "(" }
    def Ignore        = rule { (! (CreateTable | Space ~ Space)  ~ ANY).+ }
  }

{% endhighlight %}

It's a fair bit uglier than a real BNF grammar definition, but given that we need to define functions & chain calls together Parboiled2 is a great step in the right direction. So what's going on here? Lets break this down line by line.

The `DdlParser` case class extends `Parser` which is the base class of all Parboiled2 parsers & requires a `ParserInput` val named input. Since this is a case class, you probably inferred that each time a document is parsed a new parser is created. Thankfully these are lightweight objects.

`DDL` is the top level rule just like in the grammar itself. The `rule` macro is called with `Statements.*` in its scope. This is a dense line, but under the covers it's generating a parser rule that matches on *0-n* `Statements` as a sequence. `Statements` in turn are just an ignored range and a `Statement` (that's what the `~` combinator means). The parser continues walking down the DDL parse tree passing over elements (we haven't added anything to the stack yet) until it reaches the `TableName` rule in `Table` where you'll notice `capture(!EndName ~ ANY).+`. This statement pushes characters onto the stack until it reaches `EndName`, which triggers a parser failure & backtracking to its parent rule, `TableName`. Once one or more characters have been pushed, its time to turn them into a string using Parboiled2's `~>` action combinator, which is a function of the prior rule's result/side effects (i.e. elements pushed onto stack). In the `DdlParser` we want the full table name so it's simply turning the sequence of characters pushed into a string. 

After a table name has been matched & pushed onto the stack as a string, the parser will begin processing `Arguments`, or column names in this example. Each `Arg` is preceded by some separator that's been passed into the parser at construction time. In our case this is two spaces, "  ". So, just like we did with the table name, the parser will push column name characters onto the stack until it reaches a terminating space then transform those characters into a string & push it back onto the stack. Once the arguments have been parsed & pushed, the stack should look like this: 

{% highlight scala %}
 
  
[TOP OF STACK]  Vector(id, label, location)
[  Next Cell ]  tables

{% endhighlight %}

Parboiled2 has a great feature where when data fits a case class, as this does `tbl`, you can use an action rule to transform the stack elements directly into the case class, which is what's happening in `Table`! Once the parser has walked through all the provided characters it returns a `Try[T]` with your expected result or a `ParserError` indicating where you went wrong. 

Hopefully after reading this & playing around with the code you're as excited about what Parboiled2 can do! (I'm not going to show you how to test your serialization code)
