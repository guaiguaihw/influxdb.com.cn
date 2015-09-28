---
title: Clustering, tags, and enhancements to come in 0.9.0
author: Paul Dix
date: 2014-12-08
publishdate: 2014-12-08
---

We've been heads down working on InfluxDB v0.9.0 for a while and I'd like to update the community on what we're working on and what the goals are for the upcoming release. It started out as a release to completely rewrite the clustering implementation, but it has turned into something much bigger. Not only are we significantly improving the clustering capabilities of Influx, we're adding support for tags, cleaning up the API, rewriting a bunch of underlying implementation, changing storage engines, and moving to a pure Go codebase.

In short, it's the biggest release of InfluxDB since we started the project just over a year ago. Read on for details or jump [here to sign up for early testing](#signup).

### Clustering

The 0.9.0 release has a new clustering design. At its core is a new [Streaming Raft implementation optimized for our use case](https://github.com/influxdb/influxdb/tree/master/raft). Clusters of machines will be split out into two roles: brokers and data nodes. The brokers represent a streaming Raft consensus group. The data nodes get replicated all of the raw data and keep everything indexed to answer queries.

In the most simple highly available cluster, you'd have three servers all acting as both brokers and data nodes. In that setup if you had a replication factor of 3, you'd be able to sustain a single server failure and have things stay up for both reads and writes.

In a larger setup you'd have 3-7 dedicated brokers and have the remainder be data nodes. The number of brokers you have is influenced by the number of failures you want to be able to sustain among the brokers. In a setup with 3, you can have 1 broker failure and your cluster will still be available for writes. With 5 brokers you can have 2 broker failures and with 7 you can have 3 failures.

The number of data node failures you can have is dependent on the replication factor you set. The raw time series data will be split among replication groups in the data nodes. Thus if you have 10 data nodes and a replication factor of 2, a copy of each series will exist on 2 of the data nodes in the cluster.

Our goals on this clustering release include:

* Create cluster that can handle 1-2M values written per second
* Give users ability to add servers to the cluster to expand storage capacity and load (up to the 2M ceiling)
* Give users ability to quickly replace downed servers
* Automatic recovery from server restarts and temporary outages

The 1-2M ceiling for max throughput is for this release only. Later releases will lift that ceiling and make the entire thing horizontally scalable beyond that upper bound.

### Tags

After seeing how people are using InfluxDB, we realized that people expected string columns to be tags. They wanted to put metadata about their measurements in the columns. So we've decided to add support for tags.

Tags will basically be like indexed columns. Each name and tagset will be a unique series. You'll be able to merge and join those together on the fly. We'll also be indexing the tag values and adding new query types to access information about what series have which tags.

There are two primary goals with adding tag support to InfluxDB: aid in discovery, and have great performance with millions of unique series in the database.

With discovery, we want to answer queries like:

* What hosts do I have in this data center?
* Which hosts are running MySQL?
* What sensors exist in this building?
* What measurements am I taking?

On the performance side of things, the current InfluxDB can actually store millions of unique series. The tricky part comes when you try to merge a pattern of them together or list only a sub-group of them. To make those operations fast we need an index structure for the metadata describing series and that's what tags give us.

### API cleanup

The 0.9.0 release will have some cleanup of the API and part of this will be a few breaking changes. Some of them are minor, like moving some HTTP endpoints to lie under the database as opposed to the cluster. There are two changes that I'd like to talk about specifically here: retention policies and continuous queries.

The current version of InfluxDB has a feature called *Shard Spaces*. The goal of this feature was to give the user the ability to set retention policies for how long certain bits of data would be kept around. This works in the current release. Unfortunately, many users have found it confusing and it is too easy to get into a state where you're not sure where data is going.

That's why in the 0.9.0 release of Influx, Shard Spaces will go away and we'll have a simple high level concept of *Retention Policies*. Each database will have a default retention policy that data is written to or queried from. On any write or query, the user can override the default and specify which retention policy to hit.

This is a move from an implicit assignment to a retention policy to something more explicit. You'll still be able to have a high precision area and have continuous queries aggregate and down sample from that high precision retention policy into the longer term retention policies.

The update to continuous queries will include a few things. First, we'll be changing the syntax for how to define a continuous query. It will look a little bit like how you [define triggers in SQL](http://msdn.microsoft.com/en-us/library/ms189799.aspx).

The second part is that continuous queries will now actually run continuously. Previously, continuous queries were run at each time interval for the last period of time. This meant that if you had data collection that lagged behind or if you were loading historical data, it wouldn't be included in the output of a continuous query. The 0.9.0 version will fix this.

### Storage engine

Over the summer I wrote about our [testing with different storage engines](http://influxdb.com/blog/2014/06/20/leveldb_vs_rocksdb_vs_hyperleveldb_vs_lmdb_performance.html). We released 0.8.x with support for 4 engines: LevelDB, RocksDB, HyperLevelDB, and LMDB. This ended up causing a lot more trouble than it was worth. We found different bugs in different storage engines, the build process for Influx became much more difficult and complicated, and we didn't see any real performance increase by using one over another.

We could have achieved a much better performance increase by simply refactoring our own code and optimizing for a single storage engine. And that's exactly what we're doing in the 0.9.0 release. We're moving to [BoltDB](https://github.com/boltdb/bolt) as the only supported storage engine. We're also removing Protobufs as an extra serialization layer for data in the database, which should reduce CPU consumption and improve performance on queries.

This will make it much easier (and possible in some cases) to do things like hot database backups, move a shard from one server to another, and take advantage of the operating system caching.

Another big win is that it will significantly reduce the number of open files InfluxDB will need to run. This is probably the biggest source of bug reports on the current version. People aren't able to up their open file limits and bad things happen when Influx hits that ceiling. It will make the out of the box experience much better.

It should make reads significantly faster as well, given that BoltDB is based on a data structure optimized for reads. However, we still have write target performance of at least 20k values per second on a single server (ideally we can optimize for 2-5x that).

### Pure #golang

In the 0.9.0 release we've ripped out all C and C++ code. That means it's pure Go. That makes it much easier to build. We've also tried to refactor the entire code base to be more idiomatic. We're hoping that it'll be easier for members of the community to contribute to core.

A pure Go code base also means that it will be trivial to build for any Go supported OS or architecture. That means ARM and Windows builds will be easy to produce. We may not put them out ourselves, but anyone should be able to build on their platform of choice in seconds.

### Testing

For this release we want to have a serious round of testing before we cut the actual 0.9.0. We've already lined up early partners that will do extensive testing against 0.9.0. We'll be putting out release candidates in January with the goal of getting as many people testing them as possible. Once we've had significant stress testing including load, size of database, and failure scenarios, we'll cut the official 0.9.0 release.

If you want to be involved with early testing and work closely with us on it, please email [support@influxdb.com](mailto:support@influxdb.com).

### Migration

With all these new features and a new storage engine, users will have to migrate their data from 0.8 to 0.9. We'll provide a tool do this that will be able to run while the new server is hot for writes.

The migration tool will give you the option to either convert string columns into tags, or to convert series names into tags. If using the latter option it will assume your series names follow the form:

```
<tagName>.<tagValue>.<tagName>.<tagValue>.<measurement>
# for example
az.us-west-1.host.serverA.cpu
# or any number of tags
building.2.temperature
```

You'll be able to specify if you separator is a `.` or some other character like `_`.

### Conclusion

We're very excited about the 0.9.0 release. It should improve stability, decrease memory consumption, significantly speed up queries, and be a more solid InfluxDB overall. With this release we're laying the foundation for getting to InfluxDB 1.0.

<a id="signup"></a>Sign up to our newsletter to get notified of early 0.9.0 release candidates:

<!-- Begin MailChimp Signup Form -->
<div id="newsletter_signup">
<form action="http://errplane.us5.list-manage.com/subscribe/post?u=4d17b6adac2728b1ea6e4926b&amp;id=08af34971b" method="post" id="mc-embedded-subscribe-form" name="mc-embedded-subscribe-form" class="validate" target="_blank" novalidate>

  <input type="email" value="" name="EMAIL" class="email" id="mce-EMAIL" placeholder="email address" required>
  <div class="clear"><input type="submit" value="Subscribe" name="subscribe" id="mc-embedded-subscribe" class="button radius"></div>
</form>
</div>

<!--End mc_embed_signup-->
