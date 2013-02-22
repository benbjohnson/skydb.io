---
title: Skycap - An HTTP Tracking Server
---

### Overview

The Sky gem provides one way to send events to the Sky server but it can be too intrusive for some applications.
If you are tracking events for your web site, you may want to send events through a JavaScript interface.

Skycap is a simple server for receiving events over HTTP and sending those events to Sky.


### Running Skycap

Skycap is simple to use.
First, make sure your Sky server is running in one window:

```bash
$ sudo skyd
```

And then simply install the `skycap` gem and run it using the command line.
You'll need to specify the table you want to Skycap to use.

```bash
$ gem install skycap
$ skycap --table=my_table
== Sinatra/1.3.4 has taken the stage on 10001 for development with backup from Thin
>> Thin web server (v1.5.0 codename Knife)
>> Maximum connections set to 1024
>> Listening on 0.0.0.0:10001, CTRL+C to stop
```

### Using Skycap From Your Web Page

Once the Skycap server is running, you can include the `skycap.js` file and begin identifying users and tracking events.

```html
<script src="/skycap.js"></script>
<script>
  skycap.identify("user123", {first_name:"Bob", salary:10000, is_friendly:true});
  skycap.track("/signup.html", {plan:"premium"});
</script>
```

### Skycap API

#### skycap.identify(identifier, data)

This method identifies a user and associates user-level state with them.
For example, you may want to identify the user by a `user_id` in your database or by their e-mail address.
*You only need to call this method once.*

The `data` object represents state about the user itself.
This could be their name or whether they are a paid user.
User-level data is data that changes over time and exists outside the context of an action the user is performing.

#### skycap.track(action, data)

This method tracks a specific action that has occurred.
This could be the web page that the user is on or it could be a link that is clicked.

The first argument is the name of the action.
The second argument is action-level data.
Action-level data is any data that exists only within the context of this specific action.
For example, a `purchase` action could have `purchase_amount`.


<br/>
<a href="skybox.html">« Prev <span class="hidden-phone">(Exploring Your Data with Skybox)</span></a>
