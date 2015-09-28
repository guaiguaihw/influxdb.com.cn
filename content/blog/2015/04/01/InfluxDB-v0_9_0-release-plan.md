---
title: InfluxDB v0.9.0 release plan
author: Paul Dix
date: 2015-04-01
publishdate: 2015-04-01
---

Happy April 1st everyone! As you can see, March has come and gone and we haven't yet released the official version of 0.9.0 of InfluxDB. Let's call this our elaborate April fool's joke. Seriously though, we're very hard at work on 0.9.0 and we want to have it released as soon as possible. However, there are some key things we need to get done and I wanted to share what that road map looks like publicly so people can have a better feel for how things are developing on the 0.9.0 release front.

First, we've reached a stage where we're aggressively cutting features from 0.9.0. Most features are being pushed to go into a point release (0.9.1) after 0.9.0. Our goal with 0.9.0 is to have a stable implementation with reasonable performance that has a clustering implementation that can be used in production. Here are the key features we're working on that are must haves for 0.9.0:

* Distributed queries [work already started](https://github.com/influxdb/influxdb/pull/2116). Basically, the engine is already designed for it, we just needed to wire up the network portion. That should drop in an RC next week.
* Chunked responses and streaming raw queries [work already started](https://github.com/influxdb/influxdb/pull/2107). This should go into an RC by the end of this week. Will allow queries that return large chunks of raw data without having to buffer everything in memory.
* Ability to run servers as dedicated brokers and data nodes [#1934](https://github.com/influxdb/influxdb/issues/1934#issuecomment-88547824) and [work started](https://github.com/influxdb/influxdb/pull/2128). This will make larger clusters possible and let us reach higher write scale.
* Node replacement in a cluster. We've started on this and it should be in an RC soon.
* Convert series IDs to uint64 from uint32 (this will be a breaking data change). Should be in an RC next week.
* Truncating broker logs and having data nodes get shards from other data nodes [#1722](https://github.com/influxdb/influxdb/issues/1722) and [#1948](https://github.com/influxdb/influxdb/issues/1948)
* Write back pressure if the data nodes are falling behind on replication [#1946](https://github.com/influxdb/influxdb/issues/1946)

That's it for features. It's possible we'll get to more, but those are our focus. Other than that we're doing extensive testing. We're reaching the stage where we're going to be locked down and only doing testing and bug fixes. Once we get to that point, we'll announce it since that'll be a true release candidate.

We've set up [milestones in Github to track the InfluxDB release](https://github.com/influxdb/influxdb/milestones). The 0.9.0 milestone has a bunch of stuff that may not be in there so it's not a completely accurate view of what's left. It should give you some idea as to how things are progressing though. We're cutting new RCs every couple of days. The [CHANGELOG](https://github.com/influxdb/influxdb/blob/master/CHANGELOG.md) is the best place to track how things are moving for the new releases.