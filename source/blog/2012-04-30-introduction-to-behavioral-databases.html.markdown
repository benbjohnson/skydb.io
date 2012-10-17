---
title: Introduction to Behavioral Databases
date: 2012-04-30 00:00 -00:00
layout: blog_layout
---

### What is Behavioral Data?

We traditionally think of behavior in terms of actions that people perform but
it's more than that. It doesn't have to be restricted to people -- your dog
has behavior and even inanimate objects like your phone behave (or misbehave).
The person or thing performing an action is called the Actor.

The definition for behavioral data, in the context of behavioral databases, can
be formalized as:

> A series of events related to an actor that describe the actions the actor
> performed or changes in the actor's state across time.

Events are at the heart of behavioral data. An event exists at a specific point
in time and can either be an action such as "Sign Up For Web Site" or it can
be a state change such as "My hair color is now blue".


#### Visualizing Individual Event Paths

Behavior for a single object can be thought of as events occurring sequentially
over time like this e-commerce web site example:

<center>
  <img src="/images/introduction-to-behavioral-databases/simple_behavior.png">
</center>

This is simple enough. A person comes to a web site and likes what they see so
they sign up. Then they find a product to add to their shopping cart and finally
they check out.


#### Visualizing Aggregate Event Paths

But looking at one series of events at a time is tedious. Behavioral data is
about taking every user's series of events and aggregating them to see the big
picture. When we aggregate events we produce something called a directed graph
that shows the different paths that everyone can take.

<center>
  <img src="/images/introduction-to-behavioral-databases/directed_graph.png">
</center>

This graph may look daunting but it tells us some important things about how
users behave on this web site. It shows that users coming from a Google search
arrive on the home page as well directly land on the product page. We also find
that some people that contact tech support will then cancel their account but
we also see that contacting tech support can encourage users to eventually buy
a product.

This is a contrived example but gives you an idea of how you can visualize
behavior. In a real example, you could find the number of people who
performed a series of actions or find the percentage of people who left the
site after adding a product to the cart but before checking out.


#### Applied Behavioral Analysis

We can also aggregate behavioral data to answer questions such as:

1. How are people using my web site?
1. What types of people purchase what products?
1. How do people's behavior change over time?
1. What information can I derive about users based on how they act?
1. Can I predict what a user will do based on their past actions?

These types of questions are critical to how businesses and organizations make
decisions and how they interact with their customers.


### Why Another Database?

You may be wondering why you couldn't use an existing database technology to
store this type of information. Relational databases have been around for years,
BigTable implementations are able to scale to use large sets of data, and
software like Hadoop is flexible enough to batch process and query any type of
data.

These are all options. However, they all suffer from a common problem: *they
are too general*. Behavioral data has some special properties that traditional
databases can't take advantage of and behavioral data is a common enough use
case that it deserves its own database.

#### Transactions & Locks = Slow

Relational databases require transactions and locks to ensure that multiple
processes don't update the same data at the same time. In a sense, a relational
database is the data authority for many software systems because data change
doesn't happen until the database says it does. 

Behavioral data is different. Actions stored in the database have already
happened which means that events are independent of one another. Because of
this, behavioral databases can remove costly transactions and locks.


#### Optimized Storage

Some NoSQL databases do not require transactions or locks but are slow for 
behavioral data processing because of something called
[Spacial Locality](http://en.wikipedia.org/wiki/Locality_of_reference).
Spacial locality means that related data is stored nearby. It is important
because of how computers retrieve data from the hard disk and memory.

Below is a diagram of how a traditional database would store multiple events
for a single actor on a hard disk.

<center>
  <img src="/images/introduction-to-behavioral-databases/traditional_db_storage.png">
</center>

Events are be stored where ever there was room at the time they were added. This
means that when you want to query the events for an actor later on, the database
has search several places to find all the relevant events.

However, in a behavioral database, events are clustered together so data access
is a single lookup:

<center>
  <img src="/images/introduction-to-behavioral-databases/behavioral_db_storage.png">
</center>


#### Actions as a First Class Citizen

Some databases such as Google's BigTable do cluster related data together. This
makes data access fast but these databases are still only meant to track state
change. There is no concept of an action. Because of this, there is no language
to describe or query actions and there is no standard way to store an action.

Many people use Hadoop to process log files (which are in a sense just a list of
actions). Hadoop is extremely flexible but also requires advanced programming
knowledge to write complex behavioral queries or to track actor state over time.
A behavioral database, on the other hand, makes it natural to deal with these
types of queries and to work with actor state-in-time.


#### Behavioral Data is Big Business

It's obvious that there is demand for behavioral analytics. Google Analytics is
used by [55.8% of all web
sites](http://w3techs.com/technologies/overview/traffic_analysis/all). Companies
pay thousands of dollars to companies like [Omniture](http://www.omniture.com),
[MixPanel](http://mixpanel.com) or [KISSMetrics](http://kissmetrics.com/) to
find out what their users are doing.

But behavioral analytics is still in its infancy. Tools are typically limited to
showing basic user flows and some segmentation. Furthermore, while some
companies such as [MixPanel](http://mixpanel.com) have built their own data
stores for behavioral data, there has not been an open source implementation
available.


### Sky, The Open Source Behavioral Database

[Sky](https://github.com/skylandlabs/sky) is an in-development implementation
of a behavioral database. It has two primary goals. It's first goal is to
provide high performance insertion and aggregation of evented data. The project
is still early in development but it can already aggregate a
[directed graph](http://en.wikipedia.org/wiki/Directed_graph) of
events at the rate of **45 million events per second on a single core**. That's
snappy.

The second goal of the Sky behavioral database is to provide a query language
to aggregate and extract knowledge from event paths easily. The language needs
to be fast enough to process the millions (or billions) of events that
organizations see generated every day. It also needs to be flexible enough to
work with events and their relation to other events in order to derive
meaningful information. Sky's language is called *EQL* (Event Query Language) and
is currently in development.


### Next Steps

The Sky database is still an early project but there are exciting things coming.
This blog is being maintained as development progresses to allow people to
better understand the function of behavioral databases as well as to allow them
to contribute their ideas back.

An alpha version of the server will be released soon and client libraries will
be provided soon after. The EQL language will be implemented next and then
multi-core and multi-node distribution will follow.

We'll be writing about some interesting topics on Sky in the future including:

1. Linear Scalability
1. Query Deltas
1. Real-time Queries
1. Implicit Actions & Implicit State
1. Event Aging
1. Applying Machine Learning to Behavioral Analysis
1. Low-Level Implementation Details
1. Applications for Behavioral Analysis

To see the status of Sky or to contribute to it, please visit the [Sky GitHub
Page](https://github.com/skylandlabs/sky).