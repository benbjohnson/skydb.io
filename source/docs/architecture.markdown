---
title: The Sky Architecture
---

### Terminology

The best way to understand Sky is to think of the parts of speech you learned in school: *noun*, *verb*, *adjective* and *adverb*.
Most databases just store things (nouns) and their state (adjectives).
For example, you have a table of customers and each customer has fields such as name, age, & salary.

Sky tracks nouns and adjectives but also keeps track of verbs and adverbs.
For example, you may want to track when a customer makes a purchase and the purchase amount.
In this case, the purchase is an action (or verb) and the purchase amount describes the action (which makes it the adverb).

So the basic primitives of Sky are:

* The object (noun)
* Properties (adjective)
* Actions (verb)
* Action Properties (adverbs)

Every time a property changes or an action is performed it is called an *event*.
Each event is stored with a timestamp so we can track how objects change and act over time.


### Performance

While a traditional database could be used to analyze behavioral data, Sky is specifically optimized for it.
Most analytics systems are setup to pull data from the database and crunch the numbers in an external program.
This causes a lot round trip latency and serialization overhead.
Sky embeds processing in the database itself so its processing can be orders of magnitude faster.
In addition to local processing, Sky aligns the data so it can be accessed sequentially while processed.

Most processing models will struggle to crunch through tens of thousands or even hundreds of thousands of rows per second because these issues.
Sky is built to easily crunch tens of millions of events per second on a single core.
And because Sky isolates processing to a single object at a time, it can scale up linearly across multiple cores.


### Concurrency

While per-process speed is important, it doesn't mean much if it can't scale.
Sky takes advantage of an important characteristic of behavioral data: it's isolated.
Each object's timeline can be processed in complete isolation which means that it is an [embarrassingly parallel problem][] that can easily scale.
This is intuitive.
For example, when you click around on a web site your behavior is not affected by other users on the web site.

Sky currently scales linearly across multiple cores on a machine.
We'll be adding distribution across multiple machines soon.

<br/>
<a href="getting-started.html">« Prev <span class="hidden-phone">(Getting Started)</span></a>
<span class="pull-right"><a href="basics.html">Next <span class="hidden-phone">(The Basics)</span> »</a></span>


  [embarrassingly parallel problem]: http://en.wikipedia.org/wiki/Embarrassingly_parallel

