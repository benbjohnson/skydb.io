---
title: Aggregating Sessions
---

### What are Sessions?

Sessions are a common term in web applications but the idea applies to any kind of behavior analysis.
We typically want to view how someone acts within a block of time where they are active.
By constraining our analysis to a block of active time we can see what actions caused someone to stop being active (e.g. leaving your web site).


### How to Use Sessions

Session analysis is simple in Sky.
You just wrap your cursor in an extra loop and that becomes your context.
Here's the basic flow of session analysis in Lua:

```lua
function aggregate(cursor, data)
  event = cursor:event()
  
  -- Set up how long a user must be idle (e.g. have no events) 
  -- before their session is considered done. Let's set our
  -- session for two hours (7,200 seconds).
  cursor:set_session_idle(7200)
  
  -- This outer loop will be executed once for every session.
  while cursor:next_session() do
    -- This inner loop will be executed once for every individual
    -- event in the session. It will automatically exit once the
    -- session is over.
    while cursor:next() do
      -- Do something with the current event data.
    end
  end
end
```


<br/>
<a href="lua.html">« Prev <span class="hidden-phone">(Data Processing in Lua)</span></a>


  [Lua]: http://www.lua.org/
  [Lua documentation]: http://www.lua.org/docs.html
  [LuaJIT]: http://luajit.org/
