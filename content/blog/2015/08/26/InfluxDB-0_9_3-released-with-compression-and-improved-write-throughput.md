---
title: InfluxDB 0.9.3 released with compression, improved write throughput and migration from 0.8
author: Paul Dix
date: 2015-08-26
publishdate: 2015-08-26
---

Today we're releasing InfluxDB v0.9.3. This version features significant improvements in write throughput, compression, migration from 0.8, and many bug fixes and smaller features. There were a few breaking changes that made it in as well, so please read the <a href="https://github.com/influxdb/influxdb/blob/master/CHANGELOG.md#v093-unreleased" target="_">CHANGELOG</a> carefully before upgrading. The work on this release produced a better than 2x improvement in peak write throughput and better than 50% reduction in disk space.

### Write performance and compression

Write performance has been significantly improved. Running a test on my new Macbook Pro, writing 100M data points across 100k unique series (1,000 points per series) posting to the HTTP endpoint in batches of 5,000 points per request show the following numbers from the 0.9.2 release:

Worst response times:

```
3.75s
3.71s
3.71s
```

With an average response time of `525ms` the test took 1,052 seconds to run with an average insertion rate of around 95k points per second. The worst case response times on writes in the 0.9.2 release were causing many problems for people including write timeouts and failures for continuous queries. We decided that the 0.9.3 release should prioritize addressing these issues by pulling the write ahead log out of BoltDB.

The 0.9.3 release had these numbers:

Worst response times:

```
889 ms
687 ms
644 ms
```

With an average response time of `205ms` the test took 428 seconds to run with an average insertion rate around 233k points per second. We also ran this on an i2.xlarge instance on EC2 and saw > 500k points per second writing over 250M points.

The <a href="https://github.com/influxdb/influxdb/blob/master/cmd/influx_stress/influx_stress.go" target="_">InfluxDB Stress tool</a> was used to run the test. We'll be expanding this over time to stress test scenarios with different data shapes and running queries while write load is simulated.

While these numbers are starting to look good, I should note that this is only for burst performance. Sustained write load at this level will overwhelm the server eventually as the BoltDB index won't be able to keep up.

We also added compression in this release. The previous test with 100M data points had the BoltDB on disk size of 7GB on 0.9.2 while the 0.9.3 release had an on disk size of 4GB. Because of how BoltDB works, the increments after 1GB are always in 1GB steps, so 7GB and 4GB aren't entirely accurate representations of their size. More data could be written in while the on-disk size remains the same.

We'll be working to improve compression and write performance over time. This is only the first step in a long journey.

### Upgrading from previous 0.9 versions

If you're upgrading from a previous 0.9 version, you may need to update your configuration file. Specifically, the <a href="https://github.com/influxdb/influxdb/blob/master/etc/config.sample.toml#L198" target="_">sections for UDP and Graphite inputs</a>. We also changed the line protocol, so if you're writing integers, you'll need to update your client along with this InfluxDB upgrade.

You won't see the gains in write performance and storage immediately. 0.9.3 will run both the old and new storage engine simultaneously. After upgrading, the old storage engine will be used until a new shard for the next block of time is created. This could take up to 7 days to happen. Until then you'll be on the same old storage engine that 0.9.2 used.

Do this upgrade with caution. There were large changes to the underlying storage engine so it's best to backup your data and tread lightly.

### Upgrading from 0.8

With this release we have a migration path from 0.8 to 0.9.3. The process is to upgrade to 0.8.9, run an export tool, then run an import into 0.9.3. Full details are in the <a href="https://github.com/influxdb/influxdb/blob/master/importer/README.md" target="_">readme on upgrading from 0.8</a>.

I should note that there are a few missing features in 0.9 that may delay some people from ugprading. Specifically, we need to add the <a href="https://github.com/influxdb/influxdb/issues/1821" target="_">TOP function</a>, <a href="https://github.com/influxdb/influxdb/issues/2022" target="_">order by time descending</a>, and enable <a href="https://github.com/influxdb/influxdb/issues/2555" target="_">continuous queries to downsample across many measurements</a>.

We're starting the work for these three features in the 0.9.4 cycle. They should ship either with that release on September 10th or the 0.9.5 release on October 1st.

### Where is clustering?

Some work on clustering made it into this release, but it's not ready yet. You now should be able to spin up clusters larger than 3 nodes with varied replication factors. However, not all of the queries have been wired up to be distributed. We're also missing some features for moving shard data around a cluster and replacing servers. We're starting this work in the 0.9.4 cycle. Clustering will soon be in a fully testable state. However, anti-entropy won't start until the 0.9.5 release cycle, which is a required feature for automatic recovery in long outage scenarios.

### Conclusion

You can <a href="https://influxdb.com/download/" target="_">download InfluxDB</a> or sign up for a <a href="https://customers.influxdb.com/" target="_">free trial of a manged InfluxDB server</a> in the cloud.

We also offer expert professional services for InfluxDB help, feature requests, performance and advanced configurations. Contact us at support@influxdb.com to learn more about how we can help accelerate your project.
