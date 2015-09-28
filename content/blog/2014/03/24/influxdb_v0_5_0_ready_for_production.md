---
title: InfluxDB v0.5.0 ready for production
author: Paul Dix
date: 2014-03-24
publishdate: 2014-03-24
---

InfluxDB version 0.5.0 has been released and is ready for production! Well, ready for production depending on your comfort level. There are [still issues and features we want to address](https://github.com/influxdb/influxdb/issues?state=open). However, there are no known bugs that cause a crash or a memory leak. This is the first release that we're deeming production worthy. Here's the lowdown on what we've added, what's coming, and what promises we're making for releases going forward.

This release adds support for clustering. That means you can have your data split out across multiple servers for either high availability or scalability. This release also had a few big contributions from the community, which is really exciting for us. There's support for [ingesting data via the Graphite protocol](https://github.com/influxdb/influxdb/blob/master/src/configuration/config.toml#L26-L29) thanks to [@Dieterbe](https://github.com/Dieterbe). There's a new [EXPLAIN query](https://github.com/influxdb/influxdb/commit/e2adcf1c581dc5b6074901d11656887ffca2cdb1#diff-dcba2e0e9a976baee1338925df6ffe93R42) thanks to [@elcct](https://github.com/elcct).

Continuous queries are now supported. One great new feature with them is the ability to [interpolate column values into resulting series names](https://github.com/influxdb/influxdb/blob/master/src/integration/server_test.go#L1311-L1368). That makes it easy to denormalize series into many resulting series that are very fast to query against.

Full details can be seen in the [changelog from the first 0.5.0 RC forward](https://github.com/influxdb/influxdb/blob/master/CHANGELOG.md#v050-rc1-2014-02-25).

Does all this mean that you can run InfluxDB in production? Well yes, if you're comfortable running early software. We're running it in production in a 2-node HA setup for a few customers. However, there are still a few key features that we think are quite important:

* Move a shard from one node to another (for downed node replacement)
* Backup the database (instead of just copying all the files in /data)

Note that the current release will handle a node in a cluster going down. Reads and writes should continue, but the assumption is that you'll be able to bring the node back up. When it comes back up it will receive all the data from the other nodes in the cluster that it missed while it was down. If the node is totally lost, there's currently no way to spin up a new node and have it take the place of the downed one.

Our priority for the next few point releases is to fix some of the remaining issues and add those two key features. You can expect a point release every week or two for the next few months. Future releases should not break the underlying storage format. If they do, we'll give you a migration path. If you're running a two node cluster, you should be able to upgrade the cluster one node at a time to avoid downtime. However, the client you're using must support failover. We have this in the [InfluxDB Ruby gem](https://github.com/influxdb/influxdb-ruby) and library authors should update their libraries to support multiple hosts.

You also may have noticed that all the links in this post pointed to source code and not documentation. We'll be going through and doing a complete update on the docs to make them easier to understand and to guide you through setting up a cluster.

If you need any help drop us a line at [support@influxdb.com](mailto:support@influxdb.com), or talk to us on the [InfluxDB mailing list](https://groups.google.com/forum/#!newtopic/influxdb), or chat at us in #influxdb on freenode. If you want to start using InfluxDB, but are nervous about hosting it yourself, get in touch. We're running production clusters for some customers already and we're here to help you out.

Go to the [download page](http://influxdb.org/download/) to get the latest. Or watch this [Homebrew PR for InfluxDB](https://github.com/Homebrew/homebrew/pull/27832) to see when it's available.