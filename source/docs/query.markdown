---
title: Behavioral Query Language
---

### Overview

The Sky query system is conceptually similar to SQL and includes selection, grouping and conditional sections.
However, SQL is meant to be used on relational data and Sky's query system works on behavioral data.
Because of this you also get temporal operators like `after()` that relate the relationship between actions.

Another important note is that the query engine is built into the Sky Ruby gem and built on top of the dynamic Lua execution available in the Sky server.
You can see the generated Lua at any time by running the `codegen()` method of the `SkyDB::Query` object.


### Selection

The most basic Sky query is a simple select.
Within the select you get a choice of aggregation types:

* `count()`
* `sum(property)`
* `min(property)`
* `max(property)`

For example, you could count the total number of events in your database with this query:

```ruby
SkyDB.select("count()").execute()
#=> {count:82392}
```

You can also alias fields to change the returned name:

```ruby
SkyDB.select("count() my_count, sum(purchase_amount)").execute()
#=> {my_count:82392, purchase_amount:13982.02}
```


### Grouping

Selection is useful for aggregating the data, however, it doesn't help unless you can segment the data.
To do this, you can use the `group_by()` method on the query object.
These helper methods all return the query object so you can chain them together.

Here's an example of counting the number of each type of action in your system:

```ruby
SkyDB.select("count()").group_by("action_id").execute()
#=> {1:{count:29301}, 2:{count:9123}}
```

The `group_by` method accepts multiple levels of grouping and multiple forms.
The following are all equal:

```ruby
SkyDB.select("count()").group_by("utm_source, action_id")
SkyDB.select("count()").group_by("utm_source").group_by("action_id")
SkyDB.select("count()").group_by(:utm_source, :action_id)
```


### After Condition

The 'after' condition is a powerful tool.
It lets you select from the event immediately following an action.
For example, if you're tracking page views on a web site and you want to see what page people went to immediately after visiting the `/index.html` page, you can write the following query:

```ruby
SkyDB.select("count()")
     .group_by("action_id")
     .after("/index.html")
     .execute()
```

You can chain together `after()` calls to make a funnel analysis:

```ruby
SkyDB.select("count()")
     .group_by("action_id")
     .after("/index.html")
     .after("/signup.html")
     .after("/welcome.html")
     .execute()
```

This query will show what people did immediately after viewing the home page, signing up on the web site and then being presented with a welcome message after sign up.
Each successive `after()` condition will search all actions after the previous one until a match is found within the current session.

The `select()` will only occur once all the `after()` conditions have been fulfilled.
That means that if a user only goes to `/index.html` and `/signup.html` but never goes to `/welcome.html`, they will not be counted.


### Sessions

Usually you will want to perform something like funnel analysis within the context of a session.
A session can be defined by the number of idle seconds that must occur between actions that causes one session to stop and another one to start.
You can set the session idle time by using the `Query.session()` method:

```ruby
SkyDB.select("count()")
     .group_by("action_id")
     .after("/index.html")
     .after("/signup.html")
     .after("/welcome.html")
     .session(7200)             # Sets a 2-hour idle time between sessions.
     .execute()
```

There are two important features of sessions.
First, you can use `after(:enter)` to track the start of the session:

```ruby
SkyDB.select("count()").group_by("action_id").after(:enter)
```

This shows the number of actions performed immediately upon starting a session.

The other feature is that you can see where people dropped off of a session.
In our sessionized funnel analysis above, you'll see action identifiers of `-1` returned.
The `-1` action id represents a selection where all the `after` conditions matched but the session is at the end.

In the next section, we'll talk about how to bulk load data into Sky so you can start using your own data set.

<br/>
<a href="basics.html">« Prev <span class="hidden-phone">(The Basics)</span></a>
<span class="pull-right"><a href="import.html">Next <span class="hidden-phone">(Bulk Importing Data)</span> »</a></span>
