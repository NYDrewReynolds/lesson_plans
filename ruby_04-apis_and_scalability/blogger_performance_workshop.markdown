---
title: Blogger Performance Workshop
length: 180
tags: rails, performance, activerecord, sql, caching, redis, memcached
---

# JSBlogger Performance Dojo

### Learning Goals

* Apply the performance techniques we've been discussing to a semi-realistic example application
* Get some experience working with a large seed dataset (750k+ records)
* Use a performance profiling tool (NewRelic) to pinpoint bottlenecks

## Warmup/Setup

For this tutorial, we'll be using the blogger-advanced branch of the JS Blogger
project to practice making performance improvements to a rails
application.

To get started, [checkout the branch here](https://github.com/JumpstartLab/blogger_advanced/tree/blogger-perf-workshop)
and follow the included instructions for cloning the branch and setting
up its database and dependencies.

### Caveats before you begin

This branch includes a pre-constructed dataset which populates the DB
with a large amount of records -- 5k authors, 70k articles, 300k+
comments, and 300k+ taggings. This is intended to get us closer to the
sorts of datasets we might see in production systems, and really drive
home the impact of SQL/AREL techniques like using indices and avoiding
n+1 queries.

As it stands, the performance of the app is __seriously__ bad. You'll
likely find that several of the pages won't even finish rendering in
less than 30-60 seconds.

So it is going to take a lot of work to whip the thing into shape.

But fear not, noble optimizers! You're up to the challenge.

Dig in and remember the key tenets of performance optimization work:

* Go after the biggest wins first
* Make decisions based on data / benchmarks
* Consider which optimizations will improve specific operations in the
  app VS which might have a "halo effect" to other parts of the app.
  (e.g. caching a chunk of markup vs adding a db index)

#### Step 1 -- Articles#show

This is the only page of the app that isn't completely dying, so let's
start here. For me this page is generally rendering between 150 - 250
ms.

This is not bad but let's see if we can get it under 100ms. This
shouldn't be too difficult and can likely be accomplished with one or
more of the following techniques:

* DB indices on one or more of the foreign keys we're using
* AREL Includes to save an extra query or 2
* Counter Cache for comment counts

#### Step 2 -- Articles#Index

Now things will start to get more interesting. For me this page is
completely un-usable at the moment, and it will take a good bit of work
to even get it "under control" to the point that we can start doing more
refined optimizations.

For starters, try to think about what, in general, is making the page
slow. There are likely a number of culprits, but ultimately even with
some clever DB optimizations in place, we likely just have too much data
to be rendering all of the articles on a single page.

So for starters, look for ways to cut down the number of articles we're
rendering on each `GET /articles` request (hint hint: pagination...).

Then you should be able to look at some more fine-grained tweaks to the
page as well. This one may be more challenging than `Articles#show`, but
try to get it rendering under 200ms.

#### Step 3 -- Dashboard#show

Similar to Articles#index, this page will likely be unusable to start
with. However the major problems we need to address will likely be slightly different
in nature -- While the Index needed to churn through a huge number of
articles in a repetitive fashion, the dashboard needs to ingest data
from across the application's schema in order to produce some
statistics.

As such we'll likely need to use a combination of clever query
optimizations and data or markup caching to get this page under control.

This page is probably the hardest one of the 3, but see if you can get
it to render in under 300ms (ideally without just markup-caching the
whole thing...)

__Hint:__ One useful technique for dealing with pathologically slow
pages can be to just comment out big chunks of the markup to start.
When a page takes over 30 seconds to render it can be really challenging
to even make progress.

By temporarily removing large chunks of logic from the rendering flow,
we can gradually improve the performance of the page by focusing on
smaller pieces at a time.
