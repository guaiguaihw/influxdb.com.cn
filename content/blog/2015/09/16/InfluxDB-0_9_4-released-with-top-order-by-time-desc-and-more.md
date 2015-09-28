---
title: InfluxDB 0.9.4 released with top function, order by time desc and more
author: Paul Dix
date: 2015-09-16
publishdate: 2015-09-16
---

Today we're releasing InfluxDB v0.9.4. This release has a few big features and some bug fixes. Full details are in the [InfluxDB changelog](https://github.com/influxdb/influxdb/blob/master/CHANGELOG.md#v094-2015-09-14), but here are some of the highlights.

The [top aggregate](https://github.com/influxdb/influxdb/issues/1821) has been brought into the 0.9 release line. This means that for a given period of time you can query for the top N values. This is useful for a ranked list, like seeing which counters were highest, or which hosts had the highest CPU, or which sensors had the highest readings.

You can now [order by time descending](https://github.com/influxdb/influxdb/issues/2022). The most common use case for this is to pair it with a `LIMIT 1` in the query to find the most recent data point in a time series.

You can now [backreference measurement names in continuous queries](https://github.com/influxdb/influxdb/issues/2555). By using this and the `GROUP BY *` shortcut in a query, you can now define continuous queries that aggregate all of your data into lower and longer retention policies.

We've added two queries that will provide instrumentation information about InfluxDB. From now on you'll be able to run:

```sql
-- basic stats information
show stats

-- diagnostics information
show diagnostics
```

We don't have statistics instrumentation for every part of the system yet, but we'll adding to it over time. By default, the server will write the stats information into a database called `_internal`. This data is only kept around for 7 days, but it'll be the first stop when we're helping users troubleshoot problems with InfluxDB.

One final note is that this is the first version of InfluxDB that's built with Go 1.5, so the performance characteristics may be slightly different.

### What's next: write throughput and clustering

Our number one priority for the next release is to improve write throughput and the on disk size of data in the database. To that end, we're working on a new storage engine designed specifically for our needs. This won't require any sort of data migration and the next release will still be a drop in upgrade.

Preliminary tests have shown significant improvements in sustained write throughput and on-disk size of the data. We'll be making an announcment when it's available in a nightly build for testing. Or you can [track this issue to get notified when the new storage engine is available for testing](https://github.com/influxdb/influxdb/issues/4086).

We're also continuing work on clustering with bug fixes for replication as we find them, performance improvements, and wiring up every query to work within a cluster.

### Want to give it a try?

You can <a href="https://influxdb.com/download/" target="_">download InfluxDB</a> or sign up for a <a href="https://customers.influxdb.com/" target="_">free trial of a manged InfluxDB server</a> in the cloud.

We also offer expert professional services for help with InfluxDB, feature requests, performance tuning, and advanced configurations. Contact us at [support@influxdb.com](mailto:support@influxdb.com) to learn more about how we can help accelerate your project.
