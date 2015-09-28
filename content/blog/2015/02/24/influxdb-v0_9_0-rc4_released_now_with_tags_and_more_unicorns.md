---
title: InfluxDB v0.9.0-rc4 released, now with tags and more unicorns
author: Paul Dix
date: 2015-02-24
publishdate: 2015-02-24
---

After months of hard work we're very excited to announce the first early testing build of InfluxDB v0.9.0, 0.9.0-rc4([latest here](/download#latest)). There are some details on the 0.9.0 release in the [changelog](https://github.com/influxdb/influxdb/blob/master/CHANGELOG.md), but this deserves a much larger post. We've added some major new features, like support for tags, and made breaking API changes. The new API should reflect what the 1.0 release of InfluxDB later this year will look like. There may be additions, but we are unlikely to make any breaking API changes after 0.9.0 is released. Read on to get a quick preview of some of the new functionality and for more details on what's still on the TODO list before we do the full 0.9.0 release.

### Tags and the new query API

The biggest feature in this release is the addition of tags. Data in InfluxDB is now organized by databases, measurements, tags, and fields. The combination of a measurement and a tag set represents a unique series. For instance if we have the following points written in:

```
cpu region=uswest host=serverA 23.2
cpu region=uswest host=serverB 25.1
```

In this example the first part is the measurement name `cpu`, followed by the tags, which are key value pairs, followed by a value. In JSON we'd write it like this:

```json
{
  "database" : "foo",
  "retentionPolicy" : "bar",
  "points": [
    {
      "name": "cpu",
      "tags": {
        "host": "serverA",
        "region": "uswest"
      },
      "timestamp": "2009-11-10T23:00:00Z",
      "fields": {
        "value": 23.2
      }
    },
    {
      "name": "cpu",
      "tags": {
        "host": "serverB",
        "region": "uswest"
      },
      "timestamp": "2009-11-10T23:00:00Z",
      "fields": {
        "value": 25.1
      }
    }
  ]
}
```

We have the tag keys of `host` and `region` and we're writing a single field called `value`. Measurements, tags, and fields are defined when you write the data in. Databases and retention policies must be created ahead of time. Fields are defined on measurements. Once a field has been written to a data point within a measurement, its type is set. You can have multiple fields of either float64, bool, or string (and soon bytes and int64).

With the addition of tags queries like this are very fast:

```sql
SELECT * from cpu where host = 'serverA'
```

No more encoding metadata in the name, now you can use tags. We've also added a bunch of new query syntax to get at the tags and metadata. Here are some example queries that showcase some of what's possible:

```sql
-- get a single series of the mean of cpu across a region
-- this merges all the individual series on the fly
SELECT mean(value) FROM cpu
WHERE region = 'uswest'
  AND time > now() - 4h
GROUP BY time(5m)

-- get a series for each region of the mean of cpu across that region
-- this merges all the individual series of each region on the fly
SELECT mean(value) FROM cpu
WHERE time > now() - 4h
GROUP BY time(5m), region

-- get every series from the region (i.e don't merge)
SELECT mean(value) FROM cpu
WHERE region = 'uswest'
  AND time > now() - 1h
GROUP BY time(60s), *

-- get a series for cpu for everything matching the where clause
SELECT mean(value) FROM cpu
WHERE region = 'uswest'
  AND app = 'paulapp'
  AND time > now() - 4h
GROUP BY time(5m)

-- get the second 10 series from the region 
SELECT mean(value) FROM cpu
WHERE region = 'uswest'
  AND time > now() - 4h
GROUP BY time(5m), *
LIMIT 10
OFFSET 10

-- get the second 10 series from the region 
SELECT mean(value) FROM cpu
WHERE app =~ '.*someapp.*'
  AND time > now() - 4h
GROUP BY time(5m), *
LIMIT 10
OFFSET 10

-- query against the time as an RFC3339Nano
SELECT * from cpu
WHERE time >= "2015-02-24T00:00:00Z"
  AND time < "2015-02-25T00:00:00Z"

-- query against the time as an epoch
SELECT * from cpu
WHERE time > 1424823603s
  AND time < 1424823663s

-- query the data from a retention policy other than the default
SELECT * from "6 months".cpu
WHERE time > "2015-02-24T00:00:00Z"

-- show all databases in the server
SHOW DATABASES

-- show all measurements in the passed in database
SHOW MEASUREMENTS

-- find out what measurements we're taking for redis
SHOW MEASUREMENTS WHERE service = 'redis'

-- show measurements against a regex
SHOW MEASUREMENTS where app =~ '.*paulapp.*'

-- show series (unique tag sets) on the cpu measurement
SHOW SERIES FROM cpu

-- show series from cpu for a given host
SHOW SERIES FROM cpu WHERE host = 'serverA'
SHOW SERIES FROM cpu WHERE host = 'serverA' OR host = 'serverB'

-- show all measurements and their series for a given host
SHOW SERIES WHERE host = 'serverA'

-- show what tag keys we have
SHOW TAG KEYS

-- show what tag keys we have for a given measurement
SHOW TAG KEYS FROM cpu

-- show the tag values for a given key across all measurements
SHOW TAG VALUES WITH KEY = host

-- show the tag values for a given measurement and tag key
SHOW TAG VALUES FROM cpu WITH KEY = host

-- drop a series by id
DROP SERIES 1

-- drop all series matching a where clause
DROP SERIES WHERE host = 'serverA'

-- drop all series from cpu matching a where clause
DROP SERIES from cpu WHERE region = 'uswest'

-- drop an entire measurement (coming soon)
DROP MEASUREMENT cpu

-- drop a database
DROP DATABASE mydb
```

The combination of these new features makes InfluxDB not just a time series database, but also a database for time series discovery. It's our solution for making the problem of dealing with hundreds of thousands or millions of time series tractable.

<a href="http://twitter.com/home?status=The new @InfluxDB API just blew my mind out of my face! Best time series database EVAR: http://influxdb.com/blog/2015/02/24/influxdb-v0_9_0-rc4_released_now_with_tags_and_more_unicorns.html" target="_">Tell your friends about our new API on Twitter</a>

### Continuous queries

Previously, continuous queries wouldn't work with lagged data. You'd also only get the results after a time window had passed, so you wouldn't see incremental results. Now, you have configuration options to have CQs run multiple times for each window of time giving you incremental results and picking up any data that lagged behind.

You can find more details on how the continuous query configuration operations work in the [InfluxDB config source code](https://github.com/influxdb/influxdb/blob/v0.9.0-rc4/cmd/influxd/config.go#L109-L136). We'll be updating docs over the coming weeks with more information.

The syntax for creating continuous queries has also been changed. You can find details on the new CQ syntax in the [InfluxQL Language Spec](https://github.com/influxdb/influxdb/blob/v0.9.0-rc4/influxql/INFLUXQL.md#create-continuous-query)

### Clustering

In this RC you can only operate clusters where the replication factor on the retention policy is equal to the number of servers in your cluster. This release is primarily so people can spin up a single server and test out the API and start updating libraries. We'll have full clustering support wired up within a few weeks. There's a little bit about [the new clustering model in the docs](http://influxdb.com/docs/v0.9/advanced_topics/clustering.html#brokers-and-data-nodes).

### Grafana

[Grafana](http://grafana.org/), everyone's favorite dashboarding tool, needs an update to work with the new version. The good news is that we've already done some work on this and you can find it in our [Grafana fork](https://github.com/influxdb/grafana/tree/influx-0.9rc4). We'll work with the Grafana team to get it updated to support InfluxDB 0.9.0 in the coming weeks.

### Pure Go Implementation

InfluxDB v0.9.0 is written entirely in Go. There are no longer any C or C++ based build dependencies. This means it should be trivial to build the project for any platform that Go supports. Windows and ARM users rejoice! We're not yet putting out builds for these, but if anyone is interested in helping out we'd gladly make them available if you want to build them.

To achieve this goal we've changed the underlying storage engine to [BoltDB](https://github.com/boltdb/bolt), a pure Go COW B+Tree implementation. We also wrote the query language parser in pure Go. We think the result is [code that's easier to understand and update](https://github.com/influxdb/influxdb/tree/master/influxql).

### Getting to the general release

This release is not for production use. There are still things we have to do and we haven't even started looking at performance yet. With this build we'd like to get feedback on the API and start getting some broader testing in the community. Here's a quick hit list of big things that we still have to do:

* Hot database backups
* Node replacement
* Move shard from one server to another
* Distribute queries (the framework is already in place)
* Nodes that act only as data nodes
* Truncate topic logs on broker nodes (they grow unbounded right now)
* Implement fanout continuous queries
* Missing functions/features?
* TESTING, TESTING, TESTING!

The best place to look is on [waffle.io/influxdb/influxdb](https://waffle.io/influxdb/influxdb). Everything in the Ready column and to the right of it are issues that we're working on to get 0.9.0 completed.

If there's some feature you'd like or a problem you've run into, please [search on issues first](https://github.com/influxdb/influxdb/issues). If you can't find it there, please start a discussion on the [InfluxDB Google Group](https://groups.google.com/forum/#!forum/influxdb).

### Documentation

We're still working on updating the documentation, but you can find out more in the [InfluxDB 0.9.0 Getting Started Guide](http://influxdb.com/docs/v0.9/introduction/getting_started.html). The [InfluxQL Language Specification](https://github.com/influxdb/influxdb/blob/master/influxql/INFLUXQL.md) is another good source of information.

Finally, we wrote a test script that shows [writing and querying data in InfluxDB to get you started](https://gist.github.com/pauldix/dd6ba8e3d114e2ceb0a8). Let us know how it looks and spread the word!

<a href="http://twitter.com/home?status=The new @InfluxDB API just blew my mind out of my face! Best time series database EVAR: http://influxdb.com/blog/2015/02/24/influxdb-v0_9_0-rc4_released_now_with_tags_and_more_unicorns.html" target="_">Tell your friends about our new API on Twitter</a>
