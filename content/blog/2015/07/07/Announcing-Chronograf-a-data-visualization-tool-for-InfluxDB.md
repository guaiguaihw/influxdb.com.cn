---
title: Announcing Chronograf, a data visualization tool for InfluxDB
author: Paul Dix
date: 2015-07-07
publishdate: 2015-07-07
---

Today we're announcing Chronograf, a data visualization tool for InfluxDB. One of our project goals for Chronograf is to create a web based visualization tool that is easy to use for ad-hoc exploration of data in InfluxDB. We wanted something that could be used by non-programmers to quickly get answers to questions about their time series data. Here's a look at building a query against data fed in from <a href="/blog/2015/06/19/Announcing-Telegraf-a-metrics-collector-for-InfluxDB.html" target="_">Telegraf</a> using Chrongraf:

![Chronograf demo](/img/blog/chronograf.gif)

This release is an initial preview of Chronograf. It's a web application written in Go, which means deploying it is as simple as installing a `deb` or `rpm` package. The set of features on this initial release is small, but it's the starting point for an application and toolkit for data visualization of time series data from InfluxDB.

We're looking for early feedback and users to work with on defining and prioritizing future features. Chronograf will not only be the application, it will also be a set of tools and a framework for building custom visualizations for data from InfluxDB inside your own applications.

<a href="/chronograf/index.html" target="_">Get the initial Chronograf release here</a> or read on for more details on our goals with Chronograf and why we're building it.

### Why build yet another visualization application?

I was certain that this question is one of the first people would ask: why build yet another data visualization tool? It's a fair question, but for us it all goes back to one of our driving goals as a company: to optimize for developer happiness. That means building tools that are easy to use that integrate seamlessly together.

That means making the entire platform around InfluxDB simple to collect, process, store, and visualize time series data. Because of that goal, we need to build tools outside of just the core database. With Chronograf we want to give users something that is targeted not just at DevOps users, or developers, but also non-programmers in other areas of an organization.

If InfluxDB is the core of a central operational metrics platform, many people inside an organization will want to ask questions about the time series data and metrics being stored there. The value of InfluxDB as a platform will be even greater if users in other areas of an organization can take advantage of getting real-time insight into their systems, users, and business processes.

Chronograf will be the friendly user interface on top of the InfluxDB platform. It's the `C` in the TICKstack that I made reference to a few weeks ago in the <a href="/blog/2015/06/19/Announcing-Telegraf-a-metrics-collector-for-InfluxDB.html" target="_">Telegraf announcement</a>.

The last reason to have our own team working on another data visualization tool is that their experience building Chronograf will help them write open source libraries and frameworks that we can share with application developers to enable them to build applications around InfluxDB quickly.

### Shouldn't you be focused on InfluxDB?

The question of whether we should be singularly focused on InfluxDB came up when we made the Telegraf announcement so I figured it would be worthwhile to address that here. There's an old saying in software development from <a href="https://en.wikipedia.org/wiki/Brooks%E2%80%99_law" target="_">Brook's Law</a>: "nine women can't make a baby in a month." That is, adding more developers to a project doesn't necessarily mean that you'll get a linear improvement in time to release. In fact, you'll often delay things even further.

I'll add to this classic phrase the following obvious corollary: "nine women can make nine babies in nine months." We have separate teams inside InfluxDB and each of these is working on a project. They're able to iterate and improve while the others do their work. Having the Chronograf team as users of InfluxDB also helps us strengthen the usability and API of Influx itself.

We're still pushing forward on InfluxDB and there are more developers on it than on any other project we have going. We're also still <a href="https://boards.greenhouse.io/influxdb#.VZwzdhNVhBc" target="_">hiring for InfluxDB core</a> so if you're interested in helping out let us know.

### How will Chronograf be licensed?

Chronograf will follow a freemium model. It'll be free to use in an unlimited fashion for a single user. If you want to enable features around multiple users, teams, and security, it'll be licensed per user, per year. However, the initial releases for the next few months will be completely free to use as we build up the feature set and improve things.

### What does this mean for Grafana?

The existence of Chronograf shouldn't affect Grafana. We still want InfluxDB and Grafana to work well together and fully expect that most InfluxDB users will also be Grafana users. We'll still be deploying Grafana as part of our managed hosting platform for the foreseeable future.

In short, we love Grafana and want to ensure that InfluxDB and Chronograf users can also be happy Grafana users. However, we also needed the ability to iterate on ideas for visualization tools ourselves. We're compelled to build tools that help developers use InfluxDB as not just a DevOps tool, but as a backend for their own custom time series applications. Having a team working on visualization tools means we can not only develop Chronograf the application, but also future open source libraries that will make visualizing time series data quick and easy.

[Go sign up and install Chronograf](/chronograf/index.html).