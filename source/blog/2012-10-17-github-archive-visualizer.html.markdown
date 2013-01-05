---
title: GitHub Archive Visualizer
date: 2012-10-17 00:00 -00:00
layout: blog
---

### Overview

Today I'm releasing code and a video for the first [Sky](/) demo app.
Sky is built to aggregate user actions and state over time so I wanted to find a data set and a visualization that fellow developers would relate to.
And so was born: **The GitHub Archive Visualizer!**

<div class="row">
  <div class="span4">
    The visualizer uses Sky to step through aggregate paths of user actions.
    So you can walk through and see what users do immediately following an action such as <code>Create Repository</code>.
    The <a href="http://en.wikipedia.org/wiki/Sankey_diagram">Sankey diagram</a> shows the total number of users who performed each particular path.
    It's also interactive so you can drill down to as many levels deep as you want.
  </div>
  
  <div class="span4" style="margin:10px 0px">
    <a class="thumbnail" href="/images/github-archive-visualizer/githubarchiveviz.png">
      <img src="/images/github-archive-visualizer/githubarchiveviz_thumb.png">
    </a>
  </div>
</div>

It's important to note that all the results are computed in real-time.
There's no aggregation happening.
Every click on the UI recomputes the entire dataset.
In the demo, Sky maxes out at about **60MM events per second**.
That's fast enough to crunch through the [monthly pages views of StackOverflow](http://highscalability.com/blog/2011/3/3/stack-overflow-architecture-update-now-at-95-million-page-vi.html) in about a second and a half.
On my laptop.
On a single thread.

### Video Walkthrough

Sky is built to import a lot of events per second but to only allow a handful of large queries at a time.
That's typically how analytics are used within organizations.
The downside of this is that putting a public demo up and having tens of thousands of people querying tens of millions of events each is just not feasible.

So I created a short walkthrough of the demo app that you can view below.

<div class="row" style="margin:18px 0">
  <div class="span6 offset1">
    <a target="_blank" href="https://vimeo.com/51629936"><img src="/images/github-archive-visualizer/video_thumb.png"/></a>
  </div>
</div>

The video is six and a half minutes but the first half is a short history on Sky and some information on the dataset.
If you don't have time to spare, you can skip to 3:20 to actually see the visualizer in action.

### Installing the Demo

If the video does satisfy you need for data crunching, you can install the app locally on your machine.
The [sky-d3-demo project](https://github.com/skydb/sky-d3-demo) project on GitHub has details on how to get up and running.
If you have any issues getting it installed, feel free to add an <a href="https://github.com/skydb/sky-d3-demo/issues">issue on GitHub</a>.

I'll be adding more demos in the future.
If you have any suggestions for behavioral analytics you'd like to see (like *dynamic cohort analysis* or *predictive behavioral analytics*), add a comment below to suggest it.


### Special Thanks

The demo wouldn't be possible without these awesome tools and services:

* [The GitHub Archive](http://www.githubarchive.org/) from [Ilya Grigorik](http://www.igvita.com/).
* [D3.js Visualization Library](http://d3js.org/) from [Mike Bostock](http://bost.ocks.org/mike/)