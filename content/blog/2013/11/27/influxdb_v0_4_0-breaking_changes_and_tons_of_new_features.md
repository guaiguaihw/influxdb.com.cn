---
title: InfluxDB v0.4.0 - Breaking changes and tons of new features
author: Paul Dix
date: 2013-11-27
publishdate: 2013-11-27
---

InfluxDB has some new features, support for custom dashboards, and there are a bunch of new client libraries that have come out since we announced the project. Most importantly, we're planning a release next week that changes something on the underlying data storage. That means that without a migration, all your data will go away. Read on for more details and how to potentially avoid this fate.

### Breaking Release

Next week we'll be releasing v0.4.0 of InfluxDB. It changes a few things about the underlying storage so anyone upgrading will need to blow away their old database. If you're already running InfluxDB in production and have data you don't want to lose, drop us a line at [influxdb@errplane.com](mailto:influxdb@errplane.com) and let us know. We can work with you to create a migration so that you'll keep your old data. Actually, even if you don't care about the data, drop us a line to let us know you're running in production! We'd love to advertise which companies are using InfluxDB.

### New Features

We've had  a ton of bug fixes and new features in the last few weeks. Most notably: first, last, and histogram aggregate functions, table aliases, regex = and ! operators on the where clause, and a functioning Homebrew recipe. Just `brew update && brew upgrade influxdb` to get it running locally. More details on the changes can be found in the [InfluxDB changelog](https://github.com/influxdb/influxdb/blob/master/CHANGELOG.md).

### New Libraries

People have jumped into contributing libraries and integrations to InfluxDB. Here's a list of some of them (and let us know if you've written something you want to share!)

* [A command line interface (CLI)](https://github.com/FGRibreau/influxdb-cli)
* [Node.js](https://github.com/bencevans/node-influx), [Python](https://github.com/influxdb/influxdb-python), [PHP](https://github.com/crodas/InfluxPHP), and [Clojure](https://github.com/olauzon/capacitor) libraries
* [Tasseo](https://github.com/obfuscurity/tasseo#influxdb), a real-time dashboard, has support for InfluxDB
* [StatsD backend](https://github.com/bernd/statsd-influxdb-backend)
* [CollectD proxy](https://github.com/bpaquet/collectd-influxdb-proxy)
* Plenty of options for [deployment](http://influxdb.org/docs/deployment.html) including Chef, Puppet, and Docker

### Custom Interfaces & Dashboards

We've set up a structure for everyone to create their own custom dashboards and potentially share them with the community. Find out about about it on the InfluxDB docs for creating [custom interfaces](http://influxdb.org/docs/interfaces/).

### Hackfest

We've started a [Meetup for NYC users of InfluxDB](http://www.meetup.com/nyc-influxdb-user-group). We've also scheduled the first InfluxDB hackfest for [Monday, December 2nd](http://www.meetup.com/NYC-InfluxDB-User-Group/events/150732352/). If you're in the city, join the group and RSVP! We'll be there to answer any questions, and write sweet code. We're thinking we can write some data collection stuff (maybe framework integrations?), and maybe a new interface or two.

Thanks for reading this far. If you're a fan of what we're doing or just want to see this project continue to grow, tell your friends! Tweet, write blog posts, give lightning talks, and write sweet code to share with the world.
