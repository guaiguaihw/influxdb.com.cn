---
title: InfluxDB 0.9.1 and Telegraf 0.1.2 released with new documentation
author: Paul Dix
date: 2015-07-02
publishdate: 2015-07-02
---

Today we're releasing InfluxDB v0.9.1 and Telegraf 0.1.2. The InfluxDB release comes 3 weeks to the day after the 0.9.0 release, which is exactly what we were aiming for: regular release cadence every 3 weeks. Telegraf releases will probably come more frequently as contributors add plugins to support more services. Read on for details on the InfluxDB and Telegraf releases and how to get involved in Telegraf development.

### InfluxDB v0.9.1

The 0.9.1 release includes many bug fixes, some of which are significant. If you're using 0.9.0 or testing against it, we highly recommend that you start running 0.9.1. It's more stable, has more features, and is a drop in replacement for 0.9.0 so upgrading should take you only a minute.

For bug fixes, the list is too large to enumerate here, but there were many that caused the server to panic. There were also issues with error feedback given to users that was not helpful. Full details can be seen in the [InfluxDB CHANGELOG](https://github.com/influxdb/influxdb/blob/master/CHANGELOG.md#v091-2015-07-02).

There were only a few features added in this release, but some of them are big ones for our community. The top one on many people's list was better support for the Graphite Carbon protocol. We've expanded the support for feeding in Graphite metrics and either passing them through to InfluxDB or parsing them into measurements and tags. You can read about how to take advantage of these new features in the [InfluxDB Graphite Readme](https://github.com/influxdb/influxdb/blob/master/services/graphite/README.md).

Some users logged issues about a large number of IOPS during heavy write loads. To address this we implemented a write ahead log, or WAL. No migration is required to take advantage of this feature and it is available as soon as you upgrade. It's worth noting here that if you are testing a high write load scenario, you will get occasional pauses of up to 3 seconds during WAL flushes (depending on hardware and schema). Set your timeouts accordingly. We'll work on smoothing this out over time in future 0.9 point releases. But for now, overall throughput of 0.9.1 should be significantly better than 0.9.0.

The other big user facing feature is the addition of a new Administration Web UI. It has been simplified from the old version, but it's functional and will give users that aren't comfortable using the [Influx CLI](http://influxdb.com/docs/v0.9/tools/shell.html) a friendlier interface to get started.

We've now started on the next release cycle, which will end when we ship 0.9.2 on July 23rd. I'll post more about what's going into that release cycle and more of our future release planning in an upcoming post. In the meantime you can download 0.9.1 or the nightly builds of the upcoming 0.9.2 release from the [InfluxDB download page](https://influxdb.com/download/index.html).

### Telegraf 0.1.2 and contributing future features

We announced [Telegraf, an agent for collecting metrics from servers, Docker, services, and third-party APIs](https://influxdb.com/blog/2015/06/19/Announcing-Telegraf-a-metrics-collector-for-InfluxDB.html), two weeks ago. So far we've had 6 external contributors: thanks [sherifzain](https://github.com/sherifzain), [voxxit](https://github.com/voxxit), [fromYukki](https://github.com/fromYukki), [jipperinbham](https://github.com/jipperinbham), [nkatsaros](https://github.com/nkatsaros), and [EmilS](https://github.com/EmilS)! Thanks to those contributions we're releasing Telegraf v0.1.2 today.

There were some bug fixes and minor tweaks in this release. First, we fixed an issue where global tags, like host, weren't getting passed through to InfluxDB. Yukki also added a new plugin for memcached. You can see the full details in the [Telegraf CHANGELOG](https://github.com/influxdb/telegraf/blob/master/CHANGELOG.md#v012-2015-07-01). You can find instructions for [downloading Telegraf on the README](https://github.com/influxdb/telegraf#linux-packages-for-debianubuntu-and-rhelcentos).

Going forward with Telegraf we have many ideas that we're looking for help from the OSS community to implement. The most obvious things are plugins for just about any well known service you can think of: Hadoop, ElasticSearch, and Cassandra spring immediately to mind. But we can also have plugins for third party APIs. Think of SaaS platforms like MailChimp, NewRelic, and Google Analytics where you may want to export data from and be able to do ad-hod queries against in InfluxDB.

The next big feature we'd like to add is support for other output formats. Specifically, we want Telegraf to be able to send metrics to places other than InfluxDB. Adding support for [sending metrics to Kafka](https://github.com/influxdb/telegraf/issues/38) or [Riemann](https://github.com/influxdb/telegraf/issues/34) are two immediate targets.

The last big feature we're looking to add is [StatsD style aggregation inside Telegraf](https://github.com/influxdb/telegraf/issues/39). With this additional feature, your entire metrics stack becomes Telegraf, InfluxDB, and Grafana. This is part of our push to make things simpler for developers.

With all of these Telegraf issues, we're hoping the community can help out. The code base is much smaller and more accessible than the InfluxDB code base so even if you're just getting started with Go, it's a good place to begin.

### New Documentation

Finally, we've pushed up new documentation. This is thanks to the awesome InfluxDB support team. It's still rough around the edges and very much a work in progress, but we'll be continually improving it as we put out more releases of InfluxDB and Telegraf.

The [InfluxDB and Telegraf docs are open source](https://github.com/influxdb/influxdb.com) so we'd welcome any corrections or additions.