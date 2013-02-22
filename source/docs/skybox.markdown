---
title: Exploring Your Data with Skybox
---

### Overview

While Sky can aggregate millions of events per second, it isn't a typical database so it can be difficult to query for the data you want.
Skybox is the official UI for Sky and allows you to explore your data in depth and perform behavioral queries through an easy HTML5 interface.

Click the ["Demo"](http://demo.skydb.io) link in the navigation bar to view Skybox in action.


### Understanding Skybox

Skybox is a simple interface over your behavioral data.
It currently can only show a graph of how actions flow through your system (but more features are coming!).
For example, if you are tracking users on a web site, you might see that users start their session by entering onto your home page (`/index.html`).
From there they might flow to an `/about.html` page or a `/signup.html` page.

Each time you click on an action, Skybox will use Sky to calculate the number of users who did the action immediately after the previous one.
Since Sky recomputes the entire dataset each time you click, there is no limit to how far through a session you can view.


### Running Skybox

You can get up and running with Skybox easily.
First, make sure your Sky server is running in one window:

```bash
$ sudo skyd
```

And then simply install the `skybox` gem and run it using the command line:

```bash
$ gem install skybox
$ skybox 
== Sinatra/1.3.4 has taken the stage on 10000 for development with backup from Thin
>> Thin web server (v1.5.0 codename Knife)
>> Maximum connections set to 1024
>> Listening on 0.0.0.0:10000, CTRL+C to stop
```

Then you can open your browser to [http://localhost:10000/](http://localhost:10000/) to use Skybox.


### Upcoming Features

The current release of Skybox is limited to simple flow analysis.
However, upcoming releases will include more complex interaction such as filtering, segmentation and rolling up event properties.
If you have specific use cases you'd like to use Skybox for, please add an Issue to the [Sky GitHub Issues page](https://github.com/skydb/sky/issues).


<br/>
<a href="import.html">« Prev <span class="hidden-phone">(Bulk Importing Data)</span></a>
<span class="pull-right"><a href="skycap.html">Next <span class="hidden-phone">(Skycap: An HTTP Tracking Server)</span> »</a></span>
