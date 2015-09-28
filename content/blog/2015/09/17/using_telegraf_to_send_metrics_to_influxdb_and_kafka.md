---
title: Using Telegraf to send metrics to InfluxDB and Kafka
author: Cameron Sparr
date: 2015-09-17
publishdate: 2015-09-17
---

### Introduction

Telegraf is a daemon that can run on any server and collect a wide variety of metrics
from the system (cpu, memory, swap, etc.), common services
(mysql, redis, postgres, etc.), or
third-party APIs (coming soon). It is
plugin-driven for both collection and output of data so it is easily extendable.
It is also written in [Go](http://golang.org/), which means that it is a
compiled and standalone binary
that can be executed on any system with no need for external dependencies.
(no npm, pip, gem, or other package management tools required)

Telegraf was originally built as a metric-gathering agent for
[InfluxDB](https://influxdb.com), but has
recently evolved to output metrics to other data sinks as well, such as
[Kafka](http://kafka.apache.org/), [Datadog](https://www.datadoghq.com/),
and [OpenTSDB](http://opentsdb.net/). This
allows users to easily integrate InfluxDB into their existing monitoring and
logging stack, and to continue using their existing tools that consume kafka
or datadog data.

In this guide, we will cover:

* How to install and configure telegraf to collect CPU data
* Running telegraf
* Viewing telegraf data in Kafka
* Viewing telegraf data in the InfluxDB admin interface and Chronograf

The Kafka portion of this guide is entirely optional. If you only want to output
your data to InfluxDB, simply skip all the steps regarding Kafka and delete
the `outputs.kafka` section from the provided sample config.

### Pre-Requisites

Before you start, you will need the following:

* **[InfluxDB Server](https://influxdb.com/download/index.html)**
    * If you don't already have one, you can run the database from within a docker
    container, I build my influxdb docker image from the `build-docker.sh` script
    in the [influxdb repo](https://github.com/influxdb/influxdb).

* **Kafka Broker**
    * for this demo I'm going to run a kafka broker in a docker container, using the
    spotify/kafka distribution. Assuming you have docker installed, you can do this
    like so:

```bash
docker run -d -p 2181:2181 -p 9092:9092 \
    --env ADVERTISED_HOST=localhost \
    --env ADVERTISED_PORT=9092 spotify/kafka
```

NOTE: this entire guide can also be performed on OSX, just replace `localhost`
everywhere with your `boot2docker ip` or `docker-machine ip <name>`,
and install telegraf via homebrew

### Step 1 -- Installing Telegraf

Telegraf can be installed via .deb or .rpm packages available on the
[github page](https://github.com/influxdb/telegraf#installation), but for this
simplified demo I'll just use a standalone binary:

```bash
wget http://get.influxdb.org/telegraf/telegraf_linux_amd64_0.1.8.tar.gz
tar -xvf telegraf_linux_amd64_0.1.8.tar.gz
mv ./telegraf_linux_amd64 /usr/local/bin/telegraf
```

### Step 2 -- Configuring Telegraf

Telegraf provides a command for generating a sample config that includes all
plugins and outputs:
`telegraf -sample-config`, but for the purposes of this guide we will
use a more simple config file, paste the configuration found below
into a file called `~/telegraf.toml`. You will need to edit the two indicated
lines to match your environment if necessary.

```toml
[tags]
    dc = "us-east-1"

# OUTPUTS
[outputs]
[outputs.influxdb]
    # The full HTTP endpoint URL for your InfluxDB instance
    url = "http://localhost:8086" # EDIT THIS LINE
    # The target database for metrics. This database must already exist
    database = "telegraf" # required.

[outputs.kafka]
    # URLs of kafka brokers
    brokers = ["localhost:9092"] # EDIT THIS LINE
    # Kafka topic for producer messages
    topic = "telegraf"

# PLUGINS
# Read metrics about cpu usage
[cpu]
    # Whether to report per-cpu stats or not
    percpu = false
    # Whether to report total system cpu stats or not
    totalcpu = true
```

### Step 3 -- Running Telegraf

Now we can run telegraf with the `-test` flag, outputting one set of metrics
from the `cpu` plugin to stdout. One caveat is that this will not include CPU
usage percent yet, because that requires taking two measurements.
```bash
$ telegraf -config ~/telegraf.toml -test
* Plugin: cpu
> [cpu="cpu-total"] cpu_user value=5.75
> [cpu="cpu-total"] cpu_system value=4.59
> [cpu="cpu-total"] cpu_idle value=11226.01
[...]
> [cpu="cpu-total"] cpu_busy value=12.15
```

We are now ready to run telegraf on our system, it will begin sending its
measurements to the configured InfluxDB server & Kafka brokers, using the
automatic `host` tag and any tags configured in the toml file

```bash
$ telegraf -config telegraf.toml
2015/09/01 22:01:41 Starting Telegraf (version 0.1.8)
2015/09/01 22:01:41 Loaded outputs: influxdb kafka
2015/09/01 22:01:41 Loaded plugins: cpu
2015/09/01 22:01:41 Tags enabled: dc=us-east-1 host=myserver
```

As we can see, telegraf tells us that it has loaded the `influxdb` and `kafka`
output sinks, and the `cpu` collection plugin.

### Step 4 -- Viewing Kafka Data

We can now use the
[Kafka console consumer](http://kafka.apache.org/documentation.html#quickstart)
to validate that our kafka broker is receiving messages of each InfluxDB
line-protocol message emitted from telegraf.

To run the console consumer,
[download Kafka source:](http://kafka.apache.org/downloads.html)

```bash
wget http://apache.arvixe.com//kafka/0.8.2.0/kafka_2.10-0.8.2.0.tgz
tar -zxf kafka_2.10-0.8.2.0.tgz
cd kafka_2.10-0.8.2.0/
```

Then run (replace `localhost` with your kafka broker address if necessary):

```
$ ./bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic telegraf --from-beginning
[...]
cpu_percentageUser,cpu=cpu-total,dc=us-east-1,host=myserver value=0.4024144869214442
cpu_percentageSystem,cpu=cpu-total,dc=us-east-1,host=myserver value=0.8048289738428706
cpu_percentageIdle,cpu=cpu-total,dc=us-east-1,host=myserver value=98.6921529174951
cpu_percentageNice,cpu=cpu-total,dc=us-east-1,host=myserver value=0.0
cpu_percentageIowait,cpu=cpu-total,dc=us-east-1,host=myserver value=0.0
cpu_percentageIrq,cpu=cpu-total,dc=us-east-1,host=myserver value=0.0
cpu_percentageSoftirq,cpu=cpu-total,dc=us-east-1,host=myserver value=0.10060362173035882
cpu_percentageSteal,cpu=cpu-total,dc=us-east-1,host=myserver value=0.0
cpu_percentageGuest,cpu=cpu-total,dc=us-east-1,host=myserver value=0.0
cpu_percentageGuestNice,cpu=cpu-total,dc=us-east-1,host=myserver value=0.0
cpu_percentageStolen,cpu=cpu-total,dc=us-east-1,host=myserver value=0.0
cpu_percentageBusy,cpu=cpu-total,dc=us-east-1,host=myserver value=1.3078470824946893
```

As shown above, each
[InfluxDB line protocol](https://influxdb.com/docs/v0.9/write_protocols/line.html)
message that telegraf sends to our database is also getting logged to our kafka
broker, allowing us to consume the line-protocol from arbitrary tools that
interact with Kafka.

### Step 5 -- Viewing InfluxDB data in the Web UI and via Chronograf

Now if we navigate to our InfluxDB web interface at http://localhost:8083, we
can query for our `cpu_percentageBusy` data and see that telegraf is outputting
proper measurements.

```
SELECT * FROM cpu_percentageBusy WHERE host='myserver' AND dc='us-east-1'
```

![](/img/blog/influx_ui.png)

While the default InfluxDB admin interface is fairly basic, Influx also develops
a time-series visualization tool called
[Chronograf](https://influxdb.com/chronograf/index.html), which can make graphs
that look like this:

![](/img/blog/chrono.png)

This is graphing a query of the cpu busy percentage as seen below,
which can also be made using Chronograf's built-in query builder:
```
SELECT "value" FROM "telegraf".."cpu_percentageBusy" WHERE time > now() - 1h AND "cpu"='cpu-total'
```

### Next Steps:

Telegraf has a wealth of
[plugins already available](https://github.com/influxdb/telegraf#supported-plugins)
and a [few output options too](https://github.com/influxdb/telegraf#supported-outputs).
Documentation on how to configure all plugins and outputs can be found by running
`telegraf -sample-config`, or for individual plugins by running
`telegraf -usage PLUGIN_NAME`.

------

[Cameron Sparr](https://github.com/sparrc) works for InfluxDB and is the core
contributor to [Telegraf](https://github.com/influxdb/telegraf)

