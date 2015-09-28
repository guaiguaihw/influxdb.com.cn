---
title: One year of InfluxDB and the road to 1.0
author: Paul Dix
date: 2014-09-26
publishdate: 2014-09-26
---

I'm sitting in a Starbucks in Tokyo as I write this. I'm here because [Shuhei Tanuma](https://github.com/chobie), a developer at GREE, has invited me to give a talk about our experiences developing InfluxDB in Golang at a [GREE tech event](https://atnd.org/events/55464). Shuhei is also one of the 47 contributors we've had issue pull requests to the core of InfluxDB in the last year. He's using InfluxDB to store server and application performance data at GREE, a mobile social gaming company that's publicly traded on the Tokyo stock exchange.

It's hard to imagine that only a year ago this project didn't exist.

![first commit](/img/blog/first_commit.png)

Now that a year has gone by I wanted to take the opportunity to reflect on our path and look ahead to what we need to accomplish to get to version 1.0 of InfluxDB.

### The tale of a pivot

The genesis of InfluxDB starts well before that first commit. In the fall of 2012, Todd Persen and I applied to Y Combinator for the W13 batch. We were accepted for a project we were working on called Errplane, a real-time metrics and monitoring SaaS application. As part of the YC application process you're asked to list any other ideas you might want to work on. Our only other idea was "an open source time series database."

We got into YC and spent the entire time working on Errplane. Afterwards, we raised a seed round of funding and continued to work on Errplane until that first InfluxDB commit. However, we had to build something like InfluxDB before we could build Errplane. The evolution of the Errplane API is one of the things that led us directly to InfluxDB.

The first version of Errplane from 2012 was built on top of web services written in Scala using Cassandra to store metrics and time series data. This is a common use case for Cassandra and it's not the first time I've built a "time series database" on top of it. Around the end of 2012 we had the idea that we'd want to do on-premise deployments of the application. We figured we'd need to have an architecture that was a little more self contained so we started looking at rewriting it in Go.

Version 2.0 of the Errplane API was written Go using LevelDB as the underlying storage engine (two technologies that InfluxDB relies on).

![errplane api commit](/img/blog/errplane_api_commit.png)

Over the next 10 months we improved the Errplane API and pushed out two major versions. In its original design the API consisted of three separate services. One for data collection with a pub-sub mechanism, one for counting and aggregating (like StatsD), and a service for answering queries. The API was somewhat RESTful and could store both metrics and events data.

While we thought the API was cool, Errplane as a product wasn't taking off like we hoped it would. We knew there was a lot of work to do and features that users were expecting, but we were resource constrained and trying to take on too many things at the same time. However, we did have some people paying us for the service that were very enthusiastic about what we were doing. So we dug into how they were using the product. As it turned out, they were using our API like a time series database.

### Open source time series databases are a ghetto

Late in the summer of 2013 we started thinking about making a drastic shift in our strategy. The time series database angle looked interesting so we checked out what other people were doing. In the closed source world there are countless examples. In fact, almost any monitoring, APM, metrics, or analytics company has had to roll their own time series solution from scratch. Most of these are exposed as APIs that customers can use directly outside of the actual product, but very few developers outside the companies that produce these APIs use them. 

In the open source world we found Graphite and OpenTSDB. While the approach each project takes is a little different, both have some great features. Graphite also looked like it had a fairly large user base that was continuing to grow.

Despite Graphite's popularity and the growing need in many organizations to handle time series data, neither of these projects were being pushed forward at any level approaching other open source projects like Cassandra, Riak, Mongo, or Hadoop. I talked to users of Graphite and the two most common complaints I heard were that it was a pain to install and that it didn't scale well. The few OpenTSDB users I talked to complained about having to run an HBase cluster and that it was too easy to create hot spots that would kill performance.

We started seriously considering the open source time series database idea we had put in our YC application. The final moment of realization came for me in Berlin in September of 2013 at the [Monitorama conference](http://monitorama.eu). I signed up to attend and even had Errplane sponsor the event so I could present something new we were working on. I thought it might be a good place to meet potential customers.

What I found instead was that half of the attendees were employees and entrepreneurs at monitoring, metrics, DevOps, and server analytics companies. Most of them had a story about how their metrics API was their key intellectual property that took them years to develop. The other half of the attendees were developers at larger organizations that were rolling their own DevOps stack from a collection of open source tools. Almost all of them were creating a "time series database" with a bunch of web services code on top of some other database or just using Graphite.

When everyone is repeating the same work, it's not key intellectual property or a differentiator, it's a barrier to entry. Not only that, it's something that is hindering innovation in this space since everyone has to spend their first year or two getting to the point where they can start building something real. It's like building a web company in 1998. You have to spend millions of dollars and a year building infrastructure, racking servers, and getting everything ready before you could run the application. Monitoring and analytics applications should not be like this.

If so many people had the time series use case it seemed there was an opportunity to create a standard open platform. So I traveled back from Berlin and nervously announced to Todd and John that I thought we should start the open source project. They were both concerned about making such a drastic move, but we had a little bit of wiggle room. We figured that we could test the idea out in about two months. We had to make the choice to either continue pushing a boulder uphill with Errplane or take a risk that the open source time series database was something people needed and there was room for another player.

### InfluxDB gets announced to the world

We decided to take some lessons we learned from the Errplane API and start a fresh project. We worked on InfluxDB without really telling anyone other than our investors what we were doing for about 4 weeks. After that I thought we were far enough along to start talking about it and get some feedback on the API. I arranged to give talks at the [NYC Ruby Meetup](http://www.meetup.com/NYC-rb/events/141323452/) and the [NY Open Statistical Programming Meetup](http://www.meetup.com/nyhackr/events/148609252/). Word got out about the project. O'Reilly posted a link on their [Radar Blog](http://radar.oreilly.com/2013/11/four-short-links-5-november-2013.html). Someone posted the documentation site to Hacker News where it stayed on the front page for most of the day.

Since then I've received an incredible amount of encouragement on what we're building. I've been invited to give talks at the NY League of Professional Systems Administrators, Boston Ruby, Dropbox, a meetup in Charlottesville organized by [Vivid Cortex](https://vividcortex.com/), SF Data Engineers, Pivtol Tech Talks, Square, Data Dog, CloudFlare, Monitorama, GREE, Paris Data Geeks, and probably a few others I'm forgetting. I think I've given about 22 talks about InfluxDB in the past 12 months.

Luckily, I wasn't alone. Members of the growing InfluxDB community have given talks around the world including ones in [Kyoto](https://speakerdeck.com/smly/influxdb-and-leveldb-inside-out), [Sydney](http://www.meetup.com/devops-sydney/events/118488982/), [Cologne](http://www.colognerb.de/topics/zeitreihendaten-mit-influxdb), and [Dublin](https://2014.nosql-matters.org/dub/abstracts/#abstract_5279382692). The interest in InfluxDB has been overwhelming and it's encouraging to us to continue to push the project forward. Here are some interesting stats around the project.

* Over 2,500 InfluxDB servers reported running in the last 24 hours
* Over 3,000 [stars on Github](https://github.com/influxdb/influxdb)
* 47 contributors to core
* Over 17 client libraries written by external contributors for almost every language
* 4 command line interfaces for InfluxDB (Node, Perl, Ruby, and Go)

Despite how young InfluxDB is as an open source project, companies and other open source projects are forging ahead by deploying it into production and integrating Influx into their stack. Some of the more notable examples include [Heroku building a metrics dashboard for dynos](https://blog.heroku.com/archives/2014/8/5/new-dashboard-and-metrics-beta#heroku-metrics), [Gilt building a monitoring stack](http://tech.gilt.com/post/98337737919/slideshows-and-photos-from-last-nights-dublin-scala-ug), [Dieter Plaetinick working to replace Graphite at Vimeo](http://dieter.plaetinck.be/influxdb-as-graphite-backend-part2.html), [Google's cAdvisor adding InfluxDB support](https://github.com/google/cadvisor), and the [OpenStack Monasca project adding InfluxDB support](https://wiki.openstack.org/wiki/Monasca).

### The road to 1.0

With all this interest in the project we've had a bunch of feedback. We've encountered and fixed bugs and tried to be as helpful as possible with questions and feature requests. Even though there are already people running Influx in production, a 1.0 version would give more people confidence in running Influx as a key part of their stack. Here's a list of some of the priorities we'll be focusing on to get there:

* A robust and scalable clustering implementation
* Tools for devops on Influx production clusters like hot backups and node replacement
* Revisiting the API to match more closely with the metrics use case
* Taking some of the common patterns that are emerging and building them into the API

Basically, we're focusing on stability, production use, and making sure the API is mature enough that it won't have major changes for a while. We've already started much of this work and I"ll be writing more here and on the mailing list about changes we'll be making to the API.

If you're an InfluxDB user, fan, or simply interested in what we're doing, thank you. It's been an incredible year and the next 12 months are going to be even better. We'll keep shipping code and pushing to make Influx a core piece of infrastructure that delights developers and saves time and effort when building any kind of analytics or monitoring product.