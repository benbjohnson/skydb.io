---
title: The Basics
---

### Collecting Events

The Sky client library is simple.
You can store events and you can aggregate the events.
To store an event using the Ruby library simply call `add_event()`:

```ruby
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
```

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

In the next section, we'll talk about how to query the data once it's in your database.

<br/>
<a href="architecture.html">« Prev <span class="hidden-phone">(The Sky Architecture)</span></a>
<span class="pull-right"><a href="query.html">Next <span class="hidden-phone">(Behavioral Query Language)</span> »</a></span>
