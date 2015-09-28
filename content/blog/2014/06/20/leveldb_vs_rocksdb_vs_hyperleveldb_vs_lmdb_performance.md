---
title: Benchmarking LevelDB vs. RocksDB vs. HyperLevelDB vs. LMDB Performance for InfluxDB
author: Paul Dix
date: 2014-06-20
publishdate: 2014-06-20
---

For quite some time we've wanted to test the performance of different storage engines for our use case with InfluxDB. We started off using LevelDB because it's what we had used on earlier projects and RocksDB wasn't around yet. We've finally gotten around to running some basic tests against a few different engines. Going forward it looks like RocksDB might be the best choice for us.

However, we haven't had the time to tune any settings or refactor things to take advantage of specific storage engine characteristics. We're open to suggestions so read on for more detail.

Before we get to results, let's look at the test setup. We used a Digital Ocean droplet with 4GB RAM, 2 Cores, and 60GB of SSD storage.

The next release of InfluxDB has a clearly defined interface for adding different storage engines. You'll be able to choose LevelDB, RocksDB, HyperLevelDB, or LMDB. Which one you use is set through the [configuration file](https://github.com/influxdb/influxdb/blob/master/config.sample.toml#L74-L132).

Under the covers LevelDB is a [Log Structured Merge Tree](http://en.wikipedia.org/wiki/Log-structured_merge-tree) while LMDB is a mmap copy on write [B+Tree](http://en.wikipedia.org/wiki/B%2B_tree). RocksDB and HyperLevelDB are forks of the LevelDB project that have different optimizations and enhancements.

Our tests used a [benchmark tool that isolated the storage engines](https://github.com/influxdb/influxdb/tree/master/src/tools/benchmark-storage) for testing. The test does the following:

1. Write N values where the key is 24 bytes (3 ints)
2. Query N values (range scans through the key space in ascending order and does compares to see if it should stop)
3. Delete N/2 values
4. Run compaction
5. Query N/2 values
6. Write N/2 values

At various steps we checked what the on disk size of the database was. We went through multiple runs writing anywhere from 1 million to 100 million values. Which implementation came out on top differed depending on how many values were in the database.

For our use case we want to test on databases that have more values rather than less so we'll focus on the results for the biggest run. We're also not benchmarking `put` operations on keys that already exist. It's either inserts or deletes, which is almost always the use case with time series data.

The keys consist of 3 unsigned integers that are converted into big endian bytes. The first is an id that would normally represent a time series column id, the second is a time stamp, and the third is a sequence number. The benchmark simulates values written into a number of different ids (the first 8 bytes) and increasing time stamps and sequence numbers. This is a common load pattern for InfluxDB. Single points written to many series or columns at a time.

Writes during the test happen in batches of 1,000 key/value pairs. Each key/value pair is a different series column id up to the number of series to write in the test. The value is a serialized protobuf object. Specifically, it's a [FieldValue](https://github.com/influxdb/influxdb/blob/master/src/protocol/protocol.proto#L3-L9) with an `int64` set.

Here are the results of a run on 100 million values spread out over 500k columns:

<style>
table tr td {
  font-weight: normal;
}
.green {
  color: green;
  font-weight: bold;
}
.red {
  color: red;
  font-weight: bold;
}
</style>
<table>
  <thead>
    <th>Test step</th>
    <th>LeveLDB</th>
    <th>RocksDB</th>
    <th>HyperLevelDB</th>
    <th>LMDB</th>
  </tr>
  <thead>
  <tr>
    <th>Write 100M values</th>
    <td>36m8.29s</td>
    <td>21m18.60s</td>
    <td class="green">10m45.41</td>
    <td class="red">1h13m21.30s</td>
  </tr>
  <tr>
    <th>DB Size</th>
    <td class="green">2.7G</td>
    <td>3.2G</td>
    <td>3.2G</td>
    <td class="red">7.6G</td>
  </tr>
  <tr>
    <th>Query 100M values</th>
    <td>2m55.37s</td>
    <td class="green">2m44.99s</td>
    <td class="red">13m49.01s</td>
    <td>5m24.80s</td>
  </tr>
  <tr>
    <th>Delete 50M values</th>
    <td>3m47.64s</td>
    <td class="green">1m53.84s</td>
    <td>6m0.38s</td>
    <td class="red">6m15.98s</td>
  </tr>
  <tr>
    <th>Compaction</th>
    <td>3m59.87s</td>
    <td class="green">3m20.27s</td>
    <td>6m33.36s</td>
    <td class="red">1.548us</td>
  </tr>
  <tr>
    <th>DB Size</th>
    <td class="green">1.4G</td>
    <td>1.6G</td>
    <td>1.6G</td>
    <td class="red">7.6G</td>
  </tr>
  <tr>
    <th>Query 50M values</th>
    <td>12.12s</td>
    <td>13.59s</td>
    <td class="red">23.98s</td>
    <td class="green">8.48s</td>
  </tr>
  <tr>
    <th>Write 50M values</th>
    <td>3m5.28s</td>
    <td class="green">1m26.9s</td>
    <td>1m54.56s</td>
    <td class="red">3m25.96s</td>
  </tr>
  <tr>
    <th>DB Size</th>
    <td class="green">673M</td>
    <td>993M</td>
    <td>928M</td>
    <td class="red">2.5G</td>
  </tr>
</table>

The test was run on the default configuration for each of the storage engines. If anyone wants to test out variations, we'd love to use the best defaults. You can play around with those in the `new` method of each of the [storage engines](https://github.com/influxdb/influxdb/tree/master/src/datastore/storage).

A few interesting things come out of these results. LevelDB is the winner on disk space utilization, RocksDB is the winner on reads and deletes, and HyperLevelDB is the winner on writes. On smaller runs (30M or less), LMDB came out on top on most of the metrics except for disk size. This is actually what we'd expect for B-trees: they're faster the fewer keys you have in them.

I've marked the LMDB compaction time as a loser in red because it's a no-op and deletes don't actually reclaim disk space. On a normal database where you're continually writing data, this is ok because the old pages get used up. However, it means that the DB will ONLY increase in size. For InfluxDB this is a problem because we create a separate database per time range, which we call a shard. This means that after a time range has passed, it probably won't be getting any more writes. If we do a delete, we need some form of compaction to reclaim the disk space.

On disk space utilization, it's no surprise that the Level variants came out on top. They compress the data in blocks while LMDB doesn't use compression.

Overall it looks like RocksDB might be the best choice for our use case. However, __there are lies, damn lies, and benchmarks__. Things can change drastically based on hardware configuration and settings on the storage engines.

We tested on SSD because that's where things are going (if not already there). Rocks won't perform as well on spinning disks, but it's not the primary target hardware for us. You could also potentially create a configuration with smaller shards and use LMDB for screaming fast performance.

Here's a [gist of more of the results from different benchmark runs](https://gist.github.com/pauldix/db7dca9595e5c359ceb8).

We're open to updating settings, benchmarks, or adding new storage engines. In the meantime we'll keep iterating and try to get to the best possible performance for the use case of time series data.