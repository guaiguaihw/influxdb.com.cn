---
title: "Visualizing Bitcoin post frequency on HN with InfluxDB and Rickshaw"
description: "Visualizing Bitcoin post frequency on HN with InfluxDB and Rickshaw"
author: "Todd Persen"
date: "2013-11-19"
publishdate: "2013-11-19"
categories:
  - "Visualization"
  - "InfluxDB"
tags: [ "rickshaw", "bitcoin", "hacker news", "ruby" ]
---

Based on casual observation, the crowd at HackerNews seems to be totally obsessed 
with Bitcoin and it seems to be reaching a fever pitch. What better way to 
look at the HN Bitcoin obsession than with a visualization? In this post we'll 
use InfluxDB and Rickshaw to create a visualization of the number of posts with 
Bitcoin in the title on HN.

InfluxDB has a straightforward and snappy HTTP API that makes it easy to
pull your time series data out in real-time for use in user interfaces and
visualizations. This makes it a great pairing with [D3](http://d3js.org),
a data visualization library written in JavaScript.

First, we need to load InfluxDB with the data of Bitcoin posts over time. We'll use an 
open [Hacker News API](https://www.hnsearch.com/api) to get a list of the last 1,000 
posts containing the phrase `bitcoin` in the title. Here's a quick ruby script that 
also leverages the InfluxDB rubygem for writing data into the database:

```ruby
require "rubygems"
require "net/http"
require "net/https"
require "uri"
require "json"
require "time"
require "influxdb"

QUERY = "bitcoin"

http = Net::HTTP.new("api.thriftdb.com", 443)
http.use_ssl = true

influxdb = InfluxDB::Client.new "bitcoin", {
  :host => "sandbox.influxdb.org",
  :port => 9061,
  :username => "todd",
  :password => "password"
}

(0..9).each do |count|
  params = {
    "q" => QUERY,
    "start" => 100*count,
    "limit" => 100,
    "sortby" => "create_ts desc",
    "weights[title]" => "1.0",
  }

  request = Net::HTTP::Get.new "/api.hnsearch.com/items/_search?" +
    URI.encode_www_form(params)
  response = http.request(request)
  data = JSON.parse(response.body)

  data["results"].each do |result|
    influxdb.write_point("posts", {
      :message => result["item"]["title"],
      :time => Time.parse(result["item"]["create_ts"]).to_i * 1000
    })
  end
end
```

If you don't have a local installation of InfluxDB handy, head on over
to our [InfluxDB Playground](http://play.influxdb.org) and create a free
database to experiment with. Once we have our data available, we can turn to the fun part - visualization.
To make things easier, we're going to use a wrapper for D3
called [Rickshaw](http://code.shutterstock.com/rickshaw/), which was written
by the gang over at Shutterstock.

We'll just use the [InfluxDB Javascript Library](https://github.com/influxdb/influxdb-js)
to fetch the data, and then feed that right into a simple line chart in Rickshaw.

```javascript
$(function() {
  var influxdb = new InfluxDB("sandbox.influxdb.org", 9061, "todd", "password", "bitcoin");

  influxdb.query("SELECT COUNT(message) FROM posts WHERE time > now() - 365d GROUP BY time(24h);", function(points) {
    var data = points.map(function(point) {
      return { x: point.time / 1000, y: point.count };
    }).reverse();

    var graph = new Rickshaw.Graph({
      element: document.querySelector("#chart"),
      width: 640,
      height: 200,
      renderer: 'line',
      series: [{ data: data, color: 'steelblue' }]
    });

    var xAxis = new Rickshaw.Graph.Axis.Time({ graph: graph });
    var yAxis = new Rickshaw.Graph.Axis.Y({
      graph: graph,
      orientation: 'left',
      element: document.getElementById('y_axis')
    });

    xAxis.render();
    yAxis.render();
    graph.render();
  });
});
```

Since InfluxDB lets us easily query the time series data, all we need is a simple
transformation and then it's ready to feed directly into Rickshaw.

You can view, run, and modify the entire thing on this JSFiddle:

<iframe width="100%" height="270" style="margin-bottom: 20px;" src="http://jsfiddle.net/toddpersen/46ZRj/15/embedded/result,js,html,css" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Looking at the frequency over time it definitely seems to be picking up along with Bitcoin's price.
