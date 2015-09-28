---
title: InfluxDB v0.9.0 released with developer and production support
author: Paul Dix
date: 2015-06-11
publishdate: 2015-06-11
---

Today we're very excited to announce the release of InfluxDB version 0.9.0 and the immediate availability of [developer and production support plans](/support/). This release is the culmination of over 7 months of effort from our development team with feedback from hundreds in the community. This is a limited release with some caveats, but the most important part is what it signifies in the development of InfluxDB: a promise that going forward in the 0.9.x series of releases we will not make breaking API changes or breaking changes to the underlying data storage. It's also a return to a faster cycle of continuous incremental improvement.

### Stability

We've made a number of large changes over the past month. The most important of which is laying the [foundation for our clustering implementation](/blog/2015/06/03/InfluxDB_clustering_design.html). However, we haven't yet had the time to do the amount of testing we'd like to do. We've fixed all known bugs that cause the server to panic, but more importantly, we've fixed any known issues around data corruption.

Backup and restore functionality exists in the database, so it's now possible to recover from a catastrophic failure. If you care about your data, we recommend you regularly take backups and test to ensure the restores work.

Each future point release of the 0.9 line will add features, fix bugs, and improve overall stability. We felt it was important to get this release out now so that users in the community could start to update their libraries and integrations to work with the 0.9 API. Point releases in the 0.9 line will be drop in replacements for the older 0.9 versions and won't require any sort of migration.

### Clustering

The clustering implementation in 0.9.0 should be considered an alpha release. It should only be run in a 3 server fully replicated configuration. That is your replication factor should be 3. Future point releases of 0.9 will lift these limitations and allow you to scale out.

From the [InfluxDB clustering design doc](/blog/2015/06/03/InfluxDB_clustering_design.html) we've implemented the write path to replicate data to other nodes. We've implemented the per write request consistency levels and hinted handoff, ensuring that server reboots and short outages are quickly recovered from and consistency is restored.

What's missing are anti-entropy, distributed queries, joining new servers to a cluster, and tools for replacing servers in a cluster. We'll start development on some of these features in the 0.9.1 release cycle and more in the 0.9.2 cycle. They will be merged into releases once they're stable and tested.

What's in place in 0.9.0 should give InfluxDB users the option of running an HA setup with 3 servers. You'll be able to do rolling upgrades of those servers to get future 0.9 point releases deployed in your production environment.

### Developer and Production Support

Starting today we're officially supporting InfluxDB. Developer support plans are a one time purchase for $350 for 30 days of support. This will ensure you get quick answers to any questions you have while in the development phase of a project. Production support comes in three different tiers offering different SLAs. You can [purchase developer support directly from the site or read about the support tiers](/support/).

### Upgrading from 0.8.8

Currently there is no upgrade path to 0.9 from 0.8.8. We'll start development of the migration tool in the 0.9.1 release cycle with the goal of making it available in that release 3 weeks from now.

### Future releases and planning

Going forward we will be on a regular release cadence for the next 3 to 4 months. We'll cut a new point release in the 0.9 line every 3 weeks. This will include 2 weeks of feature development and bug fixes and 1 week of testing. Anything that passes testing will be included in the new release.

Each of the releases in the 0.9 line will be drop in replacements for the previous ones. You'll be able to upgrade without any breaking changes to the API or data storage. At the beginning of each cycle we'll tell you what we're starting development on and you'll be able to track those issues to see when they get merged in. Anything not merged in after two weeks will be pushed to the next release.

For any show stopper bugs we'll issue patch releases in the middle of the cycle. However, these would only be very serious bugs that cause a server to lose or be unable to query data. Anything related to a missing feature or something that can be worked around wouldn't be eligible for a patch release.

Our goal moving from 0.9 to 1.0 will be primarily filling in any missing features and getting more users in stable production environments. Once we've reached a level of maturity on features and testing that we feel comfortable with, we'll start a short release cycle for 1.0. This means that we're hoping to make 1.0 like the last point release in the 0.9 line, meaning that it will be a drop in replacement. We need to get further down the path of users interacting with the 0.9 API to be sure that we can make this promise, but that's the goal we have.

### Client Libraries

Currently, we only have the client library for Go updated. In the coming weeks we're hoping that library authors will update to work with the new line protocol and query language for creating databases, retention policies, and users. If you update or implement a client library, let us know and we'll link to it in the docs.

### Managed Hosting

Our managed hosting platform hasn't been updated yet to spin up 0.9.0 servers. In the coming weeks we'll update it to deploy 0.9.0 InfluxDB servers along with Grafana 2.0 servers to visualize the data. When 0.9.1 is released we'll begin work to migrate existing customers from 0.8 to 0.9.1. In the meantime, feel free to [sign up for managed InfluxDB hosting](https://customers.influxdb.com/) and we'll let you know when 0.9.0 is ready for use there.

### Conclusion

We're very excited about the foundational work we've laid out in InfluxDB 0.9.0. The API is a significant overhaul from what was previously available. There are great [new features that enable all sorts of interesting queries to do discovery and exploration](/blog/2014/12/08/clustering_tags_and_enhancements_in_0_9_0.html) of the data in InfluxDB in addition to the basic time series queries.

There's still a massive amount of work to do, but we're on a good path. And we're back to regular improvements along with a stable code base that members in the community should be able to contribute to. We've already had fantastic contributions during the last 4 weeks and we're looking forward to many more. In the coming months we'll document some key areas of the code base that will help bring contributions in areas like new query functions and input plugins.

We're looking forward to improving things quickly over the next 4 months and laying the groundwork for 1.0.
