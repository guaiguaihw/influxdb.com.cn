---
title: Announcing Telegraf 0.1.9 with Clustering, OpenTSDB and AMQP support
author: Cameron Sparr
date: 2015-09-23
publishdate: 2015-09-23
---

We are excited to announce that Telegraf 0.1.9, InfluxDB’s native, open source data collecting agent, is now available!

## What’s new

### Support for InfluxDB Clusters

[PR #143](https://github.com/influxdb/telegraf/issues/143): Telegraf can now write data to InfluxDB clusters  by listing out your instances in the URL parameter of the Telegraf config file. Telegraf will randomly select servers until it successfully makes a write.

### Support for filtering telegraf outputs on the CLI

[PR #217](https://github.com/influxdb/telegraf/pull/217): Telegraf will now allow filtering of output sinks on the command-line using the `-outputfilter` flag, much like how the `-filter` flag works for plugins.

### Support for filtering on config-file creation

[PR #217](https://github.com/influxdb/telegraf/pull/217): Telegraf now supports filtering to -sample-config command. You can now run `telegraf -sample-config -filter cpu -outputfilter influxdb` to get a config file with only the cpu plugin defined, and the influxdb output defined.

### Makefile GOBIN support

[PR #181](https://github.com/influxdb/telegraf/pull/181): `make` will now work despite a custom bin directory, courtesy of @Vye.

### AMQP output plugin

[PR #200](https://github.com/influxdb/telegraf/pull/200): Telegraf now supports AMQP output, courtesy of @ekini.

### OpenTSDB output plugin

[PR #182](https://github.com/influxdb/telegraf/pull/182): Telegraf now supports output to OpenTSDB using telnet mode, courtesy of @rplessl.

### Retry failed server connections

[PR #187](https://github.com/influxdb/telegraf/pull/187): Telegraf can now be configured to retry the connection to InfluxDB if the initial connection fails.

### Add a port tag to the Apache plugin

[PR #220](https://github.com/influxdb/telegraf/pull/220): You can now select your Apache port number depending on the URL scheme, courtesy of @neezgee.

## Release notes

### Config change

InfluxDB output config change: `url` is now `urls`, and is a list. Config files will still be backwards compatible if only `url` is specified.

### `-test` flag output
The -test flag will now output two metric collections

### Breaking change: CPU collection plugin

The CPU collection plugin has been refactored to fix some bugs and outdated dependency issues. At the same time, I also decided to fix a naming consistency issue, so `cpu_percentageIdle` will become `cpu_usage_idle`. Also, all CPU time measurements now have it indicated in their name, so `cpu_idle` will become `cpu_time_idle`. Additionally, `cpu_time` measurements are going to be dropped in the default config.

### Breaking change: Memory plugin

The memory plugin has been refactored and some measurements have been renamed for consistency. Some measurements have also been removed from being outputted. They are still being collected by `gopsutil`, and could easily be re-added in a "verbose" mode if there is demand for it.

...plus 11 bug fixes! Get the full story by checking out the official [0.1.9 changelog](https://github.com/influxdb/telegraf/blob/master/CHANGELOG.md) on GitHub.

### Get your InfluxDB hoodie while they last!

Does your company have a cool app or product that uses InfluxDB in production? We'd love to feature it on influxdb.com and give you a shout out on social media. As a "thank you" gift, we'll send you an InfluxDB hoodie and a pack of stickers. Just [fill out this simple form](https://influxdb.com/testimonials/) and we'll let you know when your entry is live and the hoodie is in the mail. Drop us a line at contact@influxdb.com if you have any questions. Thanks for the support!
