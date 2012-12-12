---
title: Data Processing Using Lua
---

### How Lua Fits into Sky

In the last section we looked at a simple map/reduce program in Lua that computes our total number of purchases and the total amount spent.
In this section we'll look at how Sky integrates with Lua at a lower level.
Sky is built to be simple.
It stores events and reads them back in order.
Lua was added to flexibly handle any analytical complexity you want to throw at it.

This documentation won't cover how to code in Lua.
For an introduction to [Lua], you can find resources on the [Lua documentation] page.
You can also read up on [LuaJIT] to understand how to load and call C functions or to optimize your Lua code.


### The Basic Structure

The basic Lua map/reduce program looks like this:

    function map(cursor, data)
      -- Initialization for each object belongs here.
      
      while not cursor:eof() do
        event = cursor:event()

        -- Do something with the event.
        
        cursor:next()
      end
    end
    
    function reduce(results, data)
      -- Combine the data argument into the results argument here.
      return results
    end

### Start With map()

It might look complicated at first but it's fairly intuitive once you dive in.
The `map()` function starts by providing you a `cursor` argument and a `data` argument.
This function gets called once for every object in your table.
The `cursor` lets you chronologically iterate over each event that belongs to the object.

To access the current state of the object, use the `event` variable.
You can access object and action properties by name.
For example, if you had a `first_name` property on your table you can access it for the current object within the current event by using `event.first_name`.
The action identifier can be found by using `event.action_id` and the timestamp of the event is `event.timestamp`.
Timestamps are stored as microseconds since midnight, January 1, 1970 UTC.

To move to the next event in the object, you just need to call the `next()` method on the `cursor`.
When there are no more events then the cursor's `eof()` function will return `false`.


### Finish With reduce()

Finally, the `reduce()` function will be called to combine groups of `map()` calls together.
The map functions are grouped together in the background for performance reasons so the reduce function only gets called a few times.
The `results` argument is a Lua table that you can combine the map data into.
The `data` argument is the same as the `data` argument passed to `map()`.

Once all the data has been reduced, Sky will MessagePack encode the `results` variable and send it back to the client.
That means that you can structure your results in whatever way you want.
One caveat is that since Lua doesn't have a separate hash and array construct, MessagePack will encode tables with all numeric keys as arrays.
You can add a dummy string key to your table to force it to always be a map.


### Gotchas

Processing using Sky and Lua works about how you'd expect it to.
There are some things to remember though:

1. Don't write to any global variables!
   The Lua script is executed in multiple contexts so globals won't be consistent.
   One exception is if you have read-only global data.
   That's fine.

1. The `event` object returned by `cursor:event()` doesn't change -- only it's properties do.
   That means that you can copy properties from the `event` object but you can't save a copy of the whole object.


### LuaJIT API

Sky uses LuaJIT to compile Lua into machine code at runtime.
It also allows you to extend the Lua language by using C structs for performance and using an FFI interface to work with C libraries.
However, use this at your own risk!
Sky doesn't have any way to catch memory access errors or problems in external libraries.
That means that LuaJIT can crash Sky if used improperly.

You can learn more about this technique by referencing the [LuaJIT FFI reference](http://luajit.org/ext_ffi.html).

<br/>
<a href="basics.html">« Prev <span class="hidden-phone">(The Basics)</span></a>


  [Lua]: http://www.lua.org/
  [Lua documentation]: http://www.lua.org/docs.html
  [LuaJIT]: http://luajit.org/
