---
title: Sky v0.1.0 Alpha Preview
date: 2012-08-20 00:00 -00:00
layout: blog_layout
---

### Overview

The hardest part about any software project is getting to a minimum viable
product. There are many pieces that have to fit together to make even the
basics of the program function. However, once the basics work it is easy to
iterate quickly and add new features.

Today the Sky database is being released as an alpha preview. The data store
and the language are both working end-to-end and can be used for basic
behavioral analytics.  I'd like to give you an overview of where Sky is at now
and what the next steps are.


### Where It's At

At the heart of Sky is a simple idea: give people the ability to understand
how things behave and then make decisions based on that understanding. The
database stores actions and state about things. That's all. Simplicity is a
basic principle of Sky. Too many other databases try to do too much.

Simplicity may sound limiting though. There are many ways to analyze data so
Sky is built with a flexible processing language called Qip (pronounced
'quip'). Qip allows you to perform complex analytics but is run next to the
data it analyzes so it is blazingly fast.


### Intro to Qip

I'll be adding documentation for the Qip language as it formalizes but I
wanted to give you a short introduction. Here's a quick list of high level
features:

* Qip looks like Java but feels like C.
* Strongly typed, class-based language (but has no inheritance).
* It works entirely on stack-based memory to keep it extremely fast.
* LLVM-backed so it runs at C speed.
* JIT compiled at runtime.
* Explicit support for interfacing with C libraries.


#### Example Source

It's probably best to give you a short example of how you might use Qip to
analyze something like clickstream data. Here's a short example of analyzing
the number of hits each page on your web site received:

<pre class="prettyprint" style="margin-bottom:9px;">
[Hashable("actionId")]
class Result {
  public Int actionId;
  public Int count;
}

Cursor cursor = path.events();
for each(Event event in cursor) {
  Result result = data.get(event.actionId);
  result.count = result.count + 1;
}
</pre>

#### Example Walkthrough

Let's take this example step by step:

1. We declare a class called `Result` that will store our data. We mark it as
`Hashable` so that we can store it in a hash map. The class contains an
`actionId` which stores which action we're storing a hit count for and a
`count` property which stores the actual hit count.

1. Qip programs are executed within the context of a single path of events. A
path is simply a list of ordered events that occur for, in our case, a user on
our web site. The `path` variable is passed to our program each time it
executes and the program will execute once for each user on our web site. The
result data is carried over across each iteration so we can maintain our hit
count.

1. Once we have the `cursor` we can use it to loop over the events in our
`path`. For each event in our cursor we grab a `Result` from our hash map
called `data`. The `data` variable is similar to `path` in that it is passed
in implicitly. The variable is of type `Map<Int,Result>` meaning that it is a
hash map with an integer key (we use `actionId`) matching a `Result` value.
The Qip hash map will retrieve an existing `Result` object if one already
exists or will build a new one if it doesn't.

1. Finally we simple increment the `count` for each `actionId` found.


#### Where Does the Data Go?

Below is a diagram of how queries and data flow through Sky. The client starts
by sending a query to the server where it gets parsed and compiled by the
Query Compiler. Next, the compiled module is passed to the Query Engine where
it runs it for each path retrieved from the the Path Iterator. Finally, once
all paths are exhausted, the results are passed to the serializer which
converts everything to [MessagePack](http://msgpack.org/) format and the data
is returned to the client.

<center>
  <img src="/images/sky-v1.0.0-alpha-preview/qip_flow.png">
</center>

This flow allows you to easily integrate with any language that supports
MessagePack (C, Java, PHP, JavaScript, Go, etc) and it ensures that you are
not limited by any particular data structure provided by Sky.


#### Plugin Architecture

Because Qip is a fairly simple language it was an explicit goal to make
integration of external code as easy as possible. There are so many great data
processing libraries for machine learning and classification that I wanted to
be able to pull from. To integrate an external call, simply add an
`[External]` metadata tag to the class method.

<pre class="prettyprint" style="margin-bottom:9px;">
class SentimentAnalyzer {
  [External(name="my_analyzer_function")]
  public String analyze(String string);
}

SentimentAnalyzer analyzer;
analyzer.analyze(some_text);
</pre>

This example here will call out to the following C function:

<pre class="prettyprint" style="margin-bottom:9px;">
struct sentiment_analyzer_t {
  ...
};

char *my_analyzer_function(struct sentiment_analyzer *analyzer, char *string);
</pre>



### Roadmap

The alpha preview is still a rough cut. The next steps are to stabilize the
server, API and the language. After that some additional language features
will be added to make queries simpler and easier. A declarative syntax will be
layered on top of the Qip procedural form that will change the web page hit
query to this:

<pre class="prettyprint" style="margin-bottom:9px;">
data.map(path, :count++);
</pre>

I'll be writing more about the declarative syntax as it gets finalized.

After Qip improvements, Sky will be adding real-time queries meaning that you
can submit the query above and retrieve not only the results back but also a
continuous stream of delta updates as new events come into the server and
match your query. This is useful not only in analytics but also can be used
for real-time risk analysis or to analyze purchase intent of a customer while
they're clicking through your web site.

Finally, Sky will be adding multicore and multinode distributed processing. A
single threaded Sky server can already analyze tens of millions of events per
second. However, sometimes you need to analyze billions of events per second.

Some exciting things are coming.
Please visit the [Sky GitHub project page](https://github.com/skydb/sky) to contribute.
Or join the mailing list by sending an e-mail to [sky@librelist.com](mailto:sky@librelist.com) to get updates.<br/><br/>

[Discuss this post on Hacker News](http://news.ycombinator.com/item?id=4409506)
