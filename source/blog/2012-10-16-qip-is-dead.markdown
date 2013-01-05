---
title: Qip is Dead, Long Live Qip
date: 2012-10-16 00:00 -00:00
layout: blog
---

### A Short History

When I first started writing Sky I wanted to make a language that was as fast as C but was dynamic and integrated into the data model.
So at the beginning of May 2012, I started writing a language called EQL (Event Query Language).
As it turns out, there's already a language with this name so the name was changed to Qip.

After five months of development, Qip turned out to be pretty awesome.
By using LLVM it was as fast as C.
It used metadata to dynamically generate things like MessagePack serialization methods for classes.
And it integrated into Sky pretty nicely by binding structs directly to the underlying data file.

But last week I realized that I had to take Qip out of the first release of Sky.


### Too Much, Too Soon

Two things made me realize that Qip need to be removed:

1. 80% of behavioral analysis can be rolled into prebuilt, parameterized functions.
2. Early releases need to focus on stability, not flexibility.

Behavioral analysis typically centers around understanding what someone is going to do next, how their actions flow over time and segmentation.
People do this with DAGs, Sankey diagrams, cohort analysis, etc.
These types of analysis are relatively easy to understand and don't require a new language to compute.
These generic features are being built directly in Sky as parameterized functions.

The other problem with Qip is complexity.
Building a database from scratch is complex enough -- you don't need to add anymore.
Especially not for the initial release.


### The Future of Qip

Qip is not dead.
It's just in an indefinite slumber.
Once Sky stabilizes more and the module API gets fleshed out then I will reconsider integrating it.
Qip is not the only option on the table though.
Google's V8 JavaScript engine could integrate well and Lua could be used instead.

However, for now, Sky is moving forward with a simpler API and no Qip.


