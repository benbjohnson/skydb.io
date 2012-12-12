---
title: The Basics
---

### Collecting Events

The Sky client library is simple.
You can store events and you can aggregate the events.
To store an event using the Ruby library simply call `add_event()`:

    SkyDB.table_name = 'customers'
    
    SkyDB.add_event(
      new Event(
        :object_id => 10,
        :timestamp => DateTime.now(),
        :action => {
          :name => 'purchase',
          :purchase_amount => 9.95,
          :referrer_url => 'google.com'
        },
        :data => {
          :first_name => 'John',
          :last_name => 'Doe',
          :rewards_member => true
        }
      )
    )

In this example, we're tracking a purchase just made by `John Doe` who's customer id is `10` in our database.
The `object_id` can be any number but typically it is the primary key from your production database table.
The `action` object shows that this is a `purchase` for $9.95 and the customer was referred from Google.
The `name` is the only action field required.
The other action properties will be added automatically if they don't already exist.

The `data` object tracks information specifically about our customer. 
This customer's name is `John Doe` and he is currently a rewards member.
Sky is smart about saving this data so if Sky already knew that this customer's name was John Doe at this point in time then it wouldn't save it again.
However, if Sky detects that this customer was previously not a rewards member then Sky will store the `rewards_member` property value for this customer at this point in time.
Sky only saves the data deltas for object properties.


### Property Types

Each property on a table has a data type and is marked as being an action property or an object property.
Object properties will maintain their state over time whereas the value of action properties only exist at the time of the action.

The following property data types exist:

* String
* Integer
* Double
* Boolean

When you add an event with a property that doesn't exist, Sky will automatically add it and set it's property data type based on the value provided.
If you'd like more control over properties and actions, you can use the `add_action()` and `add_property()` methods on the `SkyDB` object.


### Aggregating Data

Once you have events in your database you can aggregate them using the handy Lua map/reduce framework.
[Lua][] is a simple, embeddable language that is compiled and optimized into machine code on the fly making it extremely fast.

In this example, we'll start with something simple.
We'll aggregate the total number of purchases and the total amount of purchases.
We can pass this script into `SkyDB.lua_map_reduce()` and the result from the `reduce()` function below will be returned to Ruby:

    function map(cursor, data)
      data.purchase_amount = data.purchase_amount or 0
      data.purchase_count = data.purchase_count or 0

      while not cursor:eof() do
        event = cursor:event()

        -- The 'purchase' action id needs to be retrieved before generating the script.
        -- Let's assume that we know it is '2'.
        if event.action_id == 2 then
          data.purchase_amount = data.purchase_amount + event.purchase_amount
          data.purchase_count = data.purchase_count + 1
        end

        cursor:next()
      end
    end

    function reduce(results, data)
      result.purchase_amount = result.purchase_amount || 0
      result.purchase_count = result.purchase_count || 0
      
      for k,v in pairs(data) do
        results.purchase_amount = results.purchase_amount + data.purchase_amount
        results.purchase_count = results.purchase_count + data.purchase_count
      end
      return results
    end

In this example, our `map()` function will retrieve a `cursor` for each object in the table.
The `data` object passed in is a Lua table that lets us store a hash of data.
Here we are storing the total purchase amount and the total number of purchases.

Next, we loop over the cursor.
Each time we loop in the cursor we get the next event for an object so we can process the full history of state and actions for an object.
In this example, we're hard coding the action identifier for the `purchase` action.
In your application you would typically look this up and insert it into the Lua script.
If so, we'll add the purchase amount to the total and we'll increment our counter on the `data` object.

The table is split up into groups and each object is processed independently.
The `reduce()` function takes care of combining all that data together.
The `results` argument keeps a running total across all `reduce()` calls.
The `data` argument here is the `data` variable calculated in the `map()` function.
Here we simply sum up all the purchase amounts and counts from the map functions into a final `results` function.

This `results` variable is then serialized and sent to the Ruby client.
It can be any data structure you want: a hash, an array, a hash of hashes, etc.

In the next section, we'll talk more in detail about how Lua integrates with Sky.

<br/>
<a href="architecture.html">« Prev <span class="hidden-phone">(The Sky Architecture)</span></a>
<span class="pull-right"><a href="lua.html">Next <span class="hidden-phone">(Data Processing Using Lua)</span> »</a></span>


  [Lua]: http://www.lua.org/

