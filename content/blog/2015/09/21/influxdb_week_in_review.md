---
title: InfluxDB Week in Review - September 21, 2015
author: Todd Persen
date: 2015-09-21
publishdate: 2015-09-21
---

In this post we’ll recap the most interesting InfluxDB related content you may have missed in the last week or so.

### Blogs

* [Using telegraf to send metrics to InfluxDB and Apache Kafka](https://influxdb.com/blog/2015/09/17/using_telegraf_to_send_metrics_to_influxdb_and_kafka.html)
* [InfluxDB 0.9.4 released with top function, order by time desc and more](https://influxdb.com/blog/2015/09/16/InfluxDB-0_9_4-released-with-top-order-by-time-desc-and-more.html)

### Videos

* [How the new InfluxDB 0.9.3 Write Ahead Log (WAL) works](https://vimeo.com/138574472)

### New and Updated Documentation

* [Frequently Encountered Issues (Troubleshooting Tips)](https://influxdb.com/docs/v0.9/troubleshooting/frequently_encountered_issues.html)
* [Writing data to InfluxDB with the HTTP API](https://influxdb.com/docs/v0.9/guides/writing_data.html)
* [Understanding time-series databases for the SQL DBA](https://influxdb.com/docs/v0.9/concepts/crosswalk.html)

### Events

Watch this space! InfluxDB will be presenting at Meetups in San Francisco, Denver and Chicago very soon. Dates will be announced shortly.

### TL;DR Tech Tips

### [Getting an HTTP URL to return JSON data without using curl](https://groups.google.com/forum/#!topic/influxdb/GzpFy3-5118)

**Q:** Is there a way to get a URL to return JSON data without using the curl command?

**A:** Yes, just make sure to URL encode the string so it’s a valid hyperlink. For example:

http://localhost:8086/query?db=dbname&q=SHOW%20MEASUREMENTS

### [Aggregations over specific time periods](https://groups.google.com/forum/#!topic/influxdb/knurn-BgabM)

**Q:** I need to do aggregations over events grouped by specific time periods (weekdays, months and hours) to get the most popular dayweek for an event. How do I do this?

**A:** Custom grouping periods are currently not supported, so a good workaround for now is to add new tags to the data model such as weekday (with values 1-7), month (1-12) and dayhour (0-23). This allows you to do GROUP BYs over these tags.

### [Shutting down InfluxDB gracefully](https://groups.google.com/forum/#!topic/influxdb/Gz4zaRMyvX4)

**Q:** How do I shutdown InfluxDB gracefully?

**A:** If installed as a service, execute:

```
service influxdb stop
```

...otherwise send SIGTERM to the `influxd` process.

### Announcements

Get your InfluxDB hoodie while they last!

Does your company have a cool app or product that uses InfluxDB in production? We'd love to feature it on influxdb.com and give you a shout out on social media. As a "thank you" gift, we'll send you an InfluxDB hoodie and a pack of stickers. Just [fill out this simple form](https://influxdb.com/testimonials/) and we'll let you know when your entry is live and the hoodie is in the mail. Drop us a line at contact@influxdb.com if you have any questions. Thanks for the support!
