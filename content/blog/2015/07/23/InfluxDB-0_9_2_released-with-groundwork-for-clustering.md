---
title: InfluxDB 0.9.2 released with groundwork for clustering
author: Paul Dix
date: 2015-07-24
publishdate: 2015-07-24
---

Today we're releasing InfluxDB 0.9.2, exactly 3 weeks and a day after 0.9.1, which is just one day past what we were aiming for. Other than the addition of support for SSL, this release was focused on bug fixes and laying the groundwork for lifting the limitations on clustering. See the <a href="https://github.com/influxdb/influxdb/blob/master/CHANGELOG.md" target="_">CHANGELOG</a> for full details on what went into this 0.9.2.

### Clustering, performance and the next release

I mentioned that this release laid most of the groundwork for lifting the limitations on clustering. Most of this work was centered around refactoring the query engine, so this release will require some extra testing. The one day delay was due to a few bugs we found and fixed. We also started work on adding compression to the database and smoothing out the write response times when the WAL is flushing. This means that a bunch of great improvements will be coming in the next release.

The release of 0.9.3 on August 13th will have support for clusters of arbitrary size with replication factors that are less than the number of servers. You'll be able to join a server to an existing cluster. Practically speaking, I would expect clusters of > 100 servers to be problematic (if not some number below that), but we'll be interested in hearing what people's testing turns up. We'll be doing some testing of our own against large clusters in different clouds.

The clustering implementation will still be in alpha mode, but you'll be able to test larger clusters and look at how it scales. There are some key features we'll still need to implement that won't get started until the 0.9.4 release cycle. Specifically, replacing a downed server, rebalancing, and active anti-entropy.

Even with those features missing, testing against clustering can begin in earnest very soon. In fact, the nightly builds should have support for larger clusters sometime late next week. You can get 0.9.2 and the nightly builds on the <a href="/download/index.html" target="_">download page</a>.

### Migrating 0.8 servers

We're still working on migrating old servers. Next week we'll be releasing 0.8.9, which will have support for exporting all of your data into compressed text files that you can either read directly, or import into 0.9. The import functionality should land in a nightly build late next week.

Give this release a spin and let us know how things look.