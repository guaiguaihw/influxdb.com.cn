---
title: InfluxDB v0.5.0-rc1 released with all new clustering and features.
author: Paul Dix
date: 2014-02-25
publishdate: 2014-02-25
---

InfluxDB version 0.5.0-rc1 is out! This release adds significant improvements to clustering, eviction of old data, and 99.99th percentile write performance. We've also added a cool new feature to continuous queries and given the admin interface some love. The goal of this RC is to put it through serious testing with different loads and failure scenarios. We'd like the 0.5.0 line to be ready for production use with a few caveats. Read on for all the details on this big new release.

### Breaking release

All the changes to clustering and the distribution of data break formats between previous releases and this one. If you're upgrading you'll have to blow away your old Raft and DB directories and re-import all of your data. We anticipate that this will be the last breaking change we make for a while. Future breaking changes after the 0.5.0 release will come with a migration tool so that databases can be upgraded without having to re-import everything.

### New clustering and data distribution

The previous version used a consistent hashing algorithm to distribute data across a cluster. This version changes that completely to a sharding style that distributes data based on time. Data is now distributed into shards, which are contiguous blocks of time. There are two levels of storage: long term and short term. You're able to simply drop entire shards for either level whenever you'd like. This is a very efficient operation as it just deletes the entire LevelDB instance backing the shard. For more see [this detailed writeup of how sharding and clustering work in InfluxDB](https://groups.google.com/forum/#!msg/influxdb/3jQQMXmXd6Q/cGcmFjM-f8YJ) going forward.

This new style enables things like writing in high precision short term data, then automatically writing down-sampled data into longer term storage, and dropping the high precision data after some arbitrary amount of time. The down-sampling can be done automatically with continuous queries. This also makes it simple to expand clusters for new data. Simply add a server, and create upcoming shards on the new server. As the new data is written in it will go to the new servers while the old ones are still available for read queries.

### Write buffering

We've added a write ahead log (or WAL) to this version. It fills two purposes: buffering writes to the local datastore, and keeping a short term log of writes for any server that goes down that will later need that data replayed. The write buffering to the local datastore should solve the issue with writes that take too long because of LevelDB compactions. The buffer size is configurable and will need to be larger for high volume situations.

### Denormalizing into many series with continuous queries

Continuous queries received an upgrade that give them the ability to create new series from column values. For example, if you're writing in a series called `events` that has data like this:

```json
{
  "type": "click",
  "user": 23,
  "location": "/foo.html"
}
```

You can now denormalize that data into streams for each user or event type with continuous queries like this:

```sql
select * from events into events.[user]

-- or this
select * from events into events.[user].[type]
```

Since not all characters make valid series names, the values will be cleaned. `/` turns into `.` while spaces turn into `_` and other invalid characters are removed. Once we enable continuous queries to start from a given timestamp, you'll be able to backfill these indexes with old data.

### We want to hear from you!

We'll be testing this release extensively in both single server and clustered modes under different write loads and failure scenarios. If you see anything in your testing like high memory or CPU usage, crashes, or other odd behavior, let us know on the [InfluxDB mailing list](https://groups.google.com/forum/#!forum/influxdb) or at [support@influxdb.com](mailto:support@influxdb.com).

We're also looking for case studies of InfluxDB in use. If you're using InfluxDB we'd love to hear about it. We'll send you a shirt, answer questions and give support, or generally sing your praises.

### Caveats on production use

I mentioned there would be some caveats to production use. In addition to more general testing and live use, there are a few features that we consider fairly important for using InfluxDB in production. The first is the ability to move shards between servers, which enables the replacement of downed nodes in a cluster. Version 0.6.0 will add this feature, but you could certainly run in production before this. The second is the ability to backup a database, which is also slated for the next release.

That being said, if you're comfortable using early software, we'd love to help you out. Let us know if there's anything you need.

### Getting the release

The latest releases have been updated for Ubuntu, Debian, RedHat, CentOS, and the source tarballs. Head to the [InfluxDB download page](http://influxdb.org/download/) to grab them. If you're on OSX, watch [this Homebrew pull request](https://github.com/Homebrew/homebrew/pull/27012). Once it's merged in do:

```shell
brew update && brew install influxdb --devel
````

If you're upgrading from a previous version you'll need to delete your Raft and DB directories.