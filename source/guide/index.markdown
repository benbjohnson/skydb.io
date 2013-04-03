---
title: The Guide
---

<h3><a name="overview" href="#overview">Overview</a></h3>

This guide is meant to give you an understanding of how Sky works and how you can use it.
It covers the basics of how data is stored and how you can use Sky's query engine for advanced analytics.

#### Partially Transient Hashes

Sky is a database for tracking the state of partially transient hashes over time.
These hashes are partially transient because the individual properties of the hash can be set to persist values over time or only hold the value for a particular moment.
These data structures are especially useful for tracking the behavior of things.

Let's look at an example.
Say you want to know what users are doing on your e-commerce web site.
First you want to track properties about these users such as gender and place of residence.
These are called *permanent properties* and their value will persist from the time it is set until it is changed.

Next you want to track properties about actions that these users are doing such as visiting your home page, adding a product to a cart and checking out.
These properties are called *transient properties* and their value will only exist for a single moment in time.

Each time you update the value of one or more properties it is called an *event* which as an associated timestamp and these events are all combined into a *timeline* for each hash.
Hashes are also referred to as *objects* in Sky.
Because these events are all organized into a single timeline for each user, Sky can query over this data extremely fast.

#### Internal Representation & Data Types

Sky is composed of *tables* which are just collections of *objects* that adhere to an ad hoc schema that is defined by *properties*.
Properties can be defined as transient or permanent and have a data type associated with them.
There are five data types currently supported: `string`, `integer`, `float`, `boolean`, & `factor`.

Factor types work the same as strings except that they are represented more efficiently when the property has a limited number of possible values.
They work great for categorical data such as gender, state or action names.


<h3><a name="query-system" href="#query-system">The Query System</a></h3>

Because Sky stores non-relational data, SQL doesn't make sense as a query language.
Instead Sky has a simple query language composed of a few basic primitives: *queries*, *selections* & *conditions*.
Sky uses a RESTful JSON over HTTP API so we'll show queries in their JSON format.

#### Query

The *query* primitive is the root of all queries and contains a list of `steps` (either conditions or selections).
The query engine works by iterating over each event for each object in chronological order like a cursor in a database.
At each event, all the steps in the query are executed.

If you want to break timelines into sessions you can use `sessionIdleTime` to specify the number of seconds between two contiguous events to delineate a session.


#### Selections

A selection is simply a sample that is taken whenever the query engine reaches it.
The selection can specify an optional name, optional dimensions and one or more fields.
The selection fields are named and have an expression using one of the aggregation functions: `count()`, `sum(field)`, `min(field)`, `max(field)`.

##### Simple Count Query

Here's an example of a simple query to count every event in a table:

```json
{
  "steps":[
    {
      "type":"selection",
      "fields":[
        {"name":"count", "expression":"count()"}
      ]
    }
  ]
}
```

Running this query will output the following result (assuming you have 381,293 events in your table):

```json
{"count":381293}
```


##### Multi-Dimensional Query

A more complicated query involves breaking out these sample by a dimension of each object.
For example, let's assume you want to see the breakdown events that occur by `gender` and `country`.
We'll also name our selection "myStats":

```json
{
  "steps":[
    {"type":"selection", "name":"myStats", "dimensions":["gender","country"], "fields":[
      {"name":"count", "expression":"count()"}
    ]}
  ]
}
```

This query would return the following results:

```json
{
  "myStats":{
    "gender":{
      "male":{
        "country":{
          "US":{
            "count":19382
          },
          "Mexico":{
            "count":10302
          }
        }
      },
      "female":{
        "country":{
          "US":{
            "count":20183
          },
          "England":{
            "count":3023
          }
        }
      }
    }
  }
}
```

#### Conditions

Basic aggregation is not too interesting so that's why we have conditions.
Conditions are like mini-queries that will execute only if a given expression evaluates to true.
Conditions are also a way to move the query system's cursor forward by using the `within` property.
Another useful feature of conditions is that they can be nested inside each other.

##### Simple Conditioned Selection

Here's an example query to count the number of events where the "action" property is equal to "checkout" and sum the total purchase price property at each checkout:

```json
{
  "steps":[
    {"type":"condition", "expression":"action == 'checkout'", "steps":[
      {"type":"selection", "fields":[
        {"name":"count", "expression":"count()"},
        {"name":"total", "expression":"sum(purchaseAmount)"}
      ]}
    ]}
  ]
}
```

Result:

```json
{"count":128712, "total":2910232.23}
```

##### Funnel Analysis

The real power of the Sky query system is when you combine and nest these primitives.
In this example, we'll use the `within` property which is a range that specifies the number of steps that a condition must occur within in order for it to execute.
For example, specifying a `within` of `[0,1]` means the condition must evaluate to true in the current event or in the next event.
A `within` of `[1,1]` means that the condition must evaluate true in the next event.

By nesting conditions, each subcondition only executes if its parent executes.
We can perform a funnel analysis by combining nested conditions and selections.
Let's say we want to track the number of users who visit our home page, go to our pricing page and finally go to our sign up page.
We also want to count the number of users who make it to each step.

We'll number each step of the way ("0", "1", "2").
Finally we want to breakdown what action users performed immediately after the sign up so we'll dimension our final selection.

```json
{
  "sessionIdleTime":7200,
  "steps":[
    {"type":"condition", "expression":"action == '/index.html'", "steps":[
      {"type":"selection","name":"0","fields":[{"name":"count","expression":"count()"}]},
      {"type":"condition", "expression":"action == '/pricing.html'", "within":[1,1], "steps":[
        {"type":"selection","name":"1","fields":[{"name":"count","expression":"count()"}]},
        {"type":"condition", "expression":"action == '/signup.html'", "within":[1,1], "steps":[
          {"type":"selection","name":"2","fields":[{"name":"count","expression":"count()"}]},
          {"type":"condition", "expression":"true", "within":[1,1], "steps":[
            {"type":"selection","name":"3","dimensions":["action"],"fields":[{"name":"count","expression":"count()"}]}
          ]}
        ]}
      ]}
    ]}
  ]
}
```

This query will result in a nice set of steps so we can see how our funnel breaks down:

```json
{
  "0":{"count":10292},
  "1":{"count":7382},
  "2":{"count":2731},
  "3":{
    "/welcome.html":{"count":726},
    "/contact_us.html":{"count":529},
    "/index.html":{"count":128}
  }
}
```

We read these results like this:

1. 10,292 users made it to the home page.
2. 7,382 of those users then clicked through to the pricing page.
3. 2,731 of those users then went to the sign up page.
4. Finally, only 726 of those users actually signed up and made it to our welcome page. 529 users went to our contact page and then 128 users went back to our home page.
5. We can also deduce that any user who didn't make it to a next step must have left the web site. That means 1,348 users left the site (`2731-(726+529+128)`).


<h3><a name="clients" href="#clients">Client Libraries & Tools</a></h3>

#### Client Libraries

Sky uses a simple RESTful JSON over HTTP API so it's easy to interact with the server from any language.
However, there are also several client libraries that are available:

* Go - [https://github.com/skydb/sky.go](https://github.com/skydb/sky.go)
* Ruby - [https://github.com/skydb/sky.rb](https://github.com/skydb/sky.rb)
* Node.js - [https://github.com/sandfox/skynode](https://github.com/sandfox/skynode)


#### Tools

Below is a list of open source tools built to be used with Sky:

* [Skybox](https://github.com/skydb/skybox) - A frontend for visualizing Sky data.
* [Sky GitHub Archive Importer](https://github.com/skydb/sky-gharchive-importer) - Import public GitHub events into Sky for analysis.

Please let us know if you create any tools for Sky and we'll add them here.


<h3><a name="help" href="#help">Additional Help & Questions</a></h3>

Please join the [skydb Google Group](https://groups.google.com/d/forum/skydb) if you have questions or want to keep up on the latest Sky news.

If you are experience bugs in Sky or Sky-related projects you can also submit issues to the central [Sky GitHub Issues](https://github.com/skydb/sky/issues) page.