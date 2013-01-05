---
title: Data Processing Using Lua
---

### How Lua Fits into Sky

In the last section we looked at a simple aggregation program in Lua that computes our total number of purchases and the total amount spent.
In this section we'll look at how Sky integrates with Lua at a lower level.
Sky is built to be simple.
It stores events and reads them back in order.
Lua was added to flexibly handle any analytical complexity you want to throw at it.

This documentation won't cover how to code in Lua.
For an introduction to [Lua], you can find resources on the [Lua documentation] page.
You can also read up on [LuaJIT] to understand how to load and call C functions or to optimize your Lua code.


### The Basic Structure

The basic Lua aggregation program looks like this:

```lua
function aggregate(cursor, data)
  -- Initialize your own data structures here. Also, grab a reference to
  -- the current event state. The event's property values will change on
  -- each iteration below but it's reference stays the same.
  event = cursor:event()

  -- Loop over every event that occurred with the object.
  while cursor:next() do
    -- Do something with the current event data.
  end
end
```

### Breaking It Down

It might look complicated at first but it's fairly intuitive once you dive in.
The `aggregate()` function starts by providing you a `cursor` argument and a `data` argument.
This function gets called once for every object in your table.
The `cursor` lets you chronologically iterate over each event that belongs to the object.

To access the current state of the object, use the `event` variable.
You can access object and action properties by name.
For example, if you had an `age` property on your table you can access it for the current object within the current event by using `event.age`.
The action identifier can be found by using `event.action_id` and the timestamp of the event is `event.timestamp`.
Timestamps are stored as seconds since midnight, January 1, 1970 UTC.

To move to the next event in the object, you just need to call the `next()` method on the `cursor`.
When there are no more events then the loop will stop and the function will exit.

*Important Note:* Strings are treated differently.
See the "Gotchas" section below to learn how to use them.


### Where The Data Goes

Once all the data is aggregated it is returned to Ruby.
You can structure your Lua data structures however you want and they'll be encoded in MessagePack so your Ruby structure will look identical.
So if your Lua data is a hash of hashes then your Ruby data will be a hash of hashes.

One caveat is that Lua doesn't have a separate concept of hashes and arrays -- Lua just has *tables*.
Because of this and because consistency is important, all tables are encoded as hashes.


### Gotchas

Processing using Sky and Lua works about how you'd expect it to.
There are some things to remember though:

1. Don't write to any global variables!
   The Lua script is executed in multiple contexts so globals won't be consistent.
   One exception is if you have read-only global data.
   That's fine.

1. The `event` object returned by `cursor:event()` doesn't change -- only it's properties do.
   That means that you can copy properties from the `event` object but you can't save a copy of the whole object.

1. String properties are treated differently than integers, booleans and floats.
   To use a string, you need to use a wrapper function.
   That means instead of calling `event.first_name` you need to use `event:first_name()`.
   Note the colon instead of a dot and the parentheses at the end.


### LuaJIT API

Sky uses LuaJIT to compile Lua into machine code at runtime.
It also allows you to extend the Lua language by using C structs for performance and using an FFI interface to work with C libraries.
However, use this at your own risk!
Sky doesn't have any way to catch memory access errors or problems in external libraries.
That means that LuaJIT can crash Sky if used improperly.

You can learn more about this technique by referencing the [LuaJIT FFI reference](http://luajit.org/ext_ffi.html).

<br/>
<a href="basics.html">« Prev <span class="hidden-phone">(The Basics)</span></a>
<span class="pull-right"><a href="sessions.html">Next <span class="hidden-phone">(Aggregating Sessions)</span> »</a></span>


  [Lua]: http://www.lua.org/
  [Lua documentation]: http://www.lua.org/docs.html
  [LuaJIT]: http://luajit.org/
