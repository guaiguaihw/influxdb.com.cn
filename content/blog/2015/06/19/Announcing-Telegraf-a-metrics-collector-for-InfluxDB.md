---
title: Announcing Telegraf, a metrics collector for InfluxDB
author: Paul Dix
date: 2015-06-19
publishdate: 2015-06-19
---

Today we're releasing the first version of [Telegraf, an open source agent for collecting metrics and writing them to InfluxDB](https://github.com/influxdb/telegraf). It's written in Go and has the code organized to allow [plugins to be written to add support for new services and third-party APIs](https://github.com/influxdb/telegraf#plugins). As of today, Telegraf can collect metrics for system (CPU, memory, network, etc), Docker, Redis, MySQL, and PostgreSQL.

We'll be adding support for more services and third-party APIs. We'll also take pull requests from the community and be active in managing the official plugins that Telegraf supports.

### Why write another metrics collector?

There's no shortage of open source agents for collecting metrics. Many people are using CollectD or Diamond along with InfluxDB. If those exist and already have support for hundreds of different services, why would we go to the trouble of creating yet another metrics collector?

The primary reason is that we wanted to have a metrics collector that was designed around InfluxDB's data model: measurements and tags. We also wanted something that would be easy to setup and work with InfluxDB.

The last reason is that we plan on adding more features to Telegraf as time goes on. We could add support for a metrics aggregator for InfluxDB like StatsD, or we could add support for monitoring and alerting at the collection point. The point is that we wanted a project that we could move forward that was specifically designed to work with InfluxDB.

### Does this mean InfluxDB won't support the Graphite, CollectD or OpenTSDB protocols?

InfluxDB will continue to support other protocols and third-party collection tools. In fact, next week we'll be working on updating our support for the Graphite protocol to make it more flexible. Hopefully this work will be included in the 0.9.1 release on July 2nd.

We'll continue to support integration with other tools. We think it's important to have the ability to select your tools and this will ultimately lead to a stronger community around InfluxDB.

### TICK stack: for building applications around time series data

Even though we want to support third party tools and as many integrations as possible, we also have a plan to create tools for working with time series data. Telegraf is the first addition to what will be a larger stack that includes InfluxDB. There are four parts to it: Telegraf, InfluxDB, C (to be announced), and K (to be announced). These four parts together create the TICK stack, a platform for working with time series data.

It's our effort to create a suite of tools that are easy to setup and use for developers that are building applications around time series data. Use cases include server monitoring tools, application performance monitoring, real-time analytics and business intelligence, and applications for sensor data.

The name TICK is a reference to what people in finance refer to as a single point in a time series. We'll be announcing more on the TICK stack soon.

In the meantime, see the [Telegraf README](https://github.com/influxdb/telegraf/blob/master/README.md) for instructions on how to install it and write plugins for new services and APIs.

Finally, I'd like to give special thanks to [Evan Phoenix](https://twitter.com/evanphx) and his company [Vektra](https://github.com/vektra) for building the initial version of Telegraf. His code is a great starting point for what will hopefully be a long-lived and vibrant OSS project.