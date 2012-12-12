---
title: Getting Started
---

### What is Sky

Sky is a behavioral database.
As opposed to most databases that only track the current state of objects, Sky tracks all state and actions about objects over time.
This makes Sky optimized for two use cases:

1. Analyzing the behavior of a single object.
1. Aggregating behavior across the entire population of objects.

Sky is built on top of solid libraries such as [LevelDB][] for storage, [LuaJIT][] for processing and [ZeroMQ][] for distributed computing.

### Supported Platforms

Currently Sky has been built and tested on the following platforms:

* Max OS X 10.7 (LLVM 3.1)
* Ubuntu 12.04 (GCC 4.6.3)

If you have Sky up and running on a different platform, please [e-mail the mailing list to let us know](mailto:sky@librelist.com) and we'll add it to the list.


### Prerequisites

ZeroMQ 3.2.x is the only dependency that needs to be externally compiled.
You can download it from [zeromq.org](http://www.zeromq.org/).

    $ wget http://download.zeromq.org/zeromq-3.2.2.tar.gz
    $ tar zxvf zeromq-3.2.2.tar.gz
    $ cd zeromq-3.2.2
    $ ./configure
    $ make
    $ sudo make install

On Ubuntu, you'll need to add `/usr/local/lib` to your `ldconfig`.


### Installing Sky

Download the [latest version of Sky from GitHub](https://github.com/skydb/sky/downloads) and follow these instructions to build it:

    $ wget http://skydb.io/sky-0.2.0.tar.gz
    $ tar zxvf sky-0.2.0.tar.gz
    $ cd sky-0.2.0
    $ make
    $ sudo make install

To run the server, simply run the `skyd` program:

    $ sudo skyd
    Sky Server v0.2.0
    Listening on 0.0.0.0:8585, CTRL+C to stop


### Ruby Client Library

Currently the only Sky client library available is for Ruby.
You can use RubyGems to install it:

    $ gem install skydb
    Successfully installed skydb-0.2.0
    1 gem installed

You can reference the [API documentation](http://rubydoc.info/gems/skydb/frames) or the [Sky D3.js Demo][] application for examples of how to use the API.

    
### Running the demo

Sky is built to analyze behavior which provides us interesting ways to  aggregate and visualize the data.
Because of this the [Sky D3.js Demo] was created as a test bed for trying out new functionality.
To get up and running on the demo, pull down the code from GitHub and use Bundler to install the dependencies:

    $ git clone https://github.com/skydb/sky-d3-demo.git
    $ cd sky-d3-demo
    $ bundle install

The demo uses the [GitHub Archive][] as its data set so you'll need to import the data first.
Make sure you have the Sky server running before attempting to import.
The `import.rb` command takes two arguments: a start date and and end date.

    $ ./import.rb 2012-10-01 2012-10-31

This example will pull down the entire month of October 2012 for processing.
The import is currently a bit slow but we're working on it.
You can try with a smaller date range if it's taking too long.
Once the import is complete start up the application server.

    $ rackup
    >> Thin web server (v1.4.1 codename Chromeo)
    >> Maximum connections set to 1024
    >> Listening on 0.0.0.0:9292, CTRL+C to stop

You can now open up [http://localhost:9292](localhost:9292) and view the interactive visualization.
You can click on different actions to drill in and see how users on GitHub interact with the site.

The drop down in the upper left allows you to select which action to start with.
The drop down in the upper right toggles between a native C implementation of the aggregation and a LuaJIT implementation.
The LuaJIT is currently 2x slower than pure optimized C.
You can view the Lua implementation by opening up the `app.rb` file.

The demo application is open source so feel free to add to it.

<br/>
<span class="pull-right"><a href="architecture.html">Next <span class="hidden-phone">(The Sky Architecture)</span> »</a></span>


  [LevelDB]: http://code.google.com/p/leveldb
  [LuaJIT]: http://luajit.org
  [ZeroMQ]: http://zeromq.org
  [Sky D3.js Demo]: https://github.com/skydb/sky-d3-demo
  [GitHub Archive]: http://www.githubarchive.org/