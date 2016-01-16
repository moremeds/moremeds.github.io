---
title: Grafana Stack with InfluxDB
date: 2016-01-03 19:22:37
tags: [grafana, influxdb, statsd]
categories: [tutorial]
---

Collecting and analyzing metrics are the key to understand a system’s runtime behaviors and thus to provide a better guideline of optimization. As the user of a metrics system, you might want to collect metric and event with simple APIs, retrieve historical metric data, and view visualized metrics statistics.

This article shows you how to build a metric system with [StatsD](https://github.com/etsy/statsd), [influxdb](https://influxdata.com/), and [Grafana](http://grafana.org/), as an example metrics system that composes of a client, a data storage, and visualization UI. There are many available open source packages.

<img src="/images/grafana/LogicFlow.png" width="60%" height="60%">

<!--more-->

The figure above shows you the data flow. User configures the application, and sends metrics to StatsD server. StatsD stores the metrics data in InfluxDB, a time-series database, where Grafana retrieves data and displays the visualized metrics.

Let’s briefly discuss the software collection before moving on to configuration.
* StatsD is a daemon that collects and aggregates metrics data. With the well-accepted UDP protocol, [clients library](https://github.com/etsy/statsd/wiki) for almost all major programming language. In the meantime, StatsD can also serves as a local aggregator, providing a extensible backend architecture.
* [Grafana](http://docs.grafana.org/installation/rpm/) is a metrics dashboard that provides data visualization and generates statistic graph. You can choose the most suitable approach to install. Configuration file can be found [here](http://docs.grafana.org/installation/configuration/).
* [InfluxDB](https://docs.influxdata.com/influxdb/v0.9/introduction/overview/) is a time-series database for storing the metrics data.

## StatsD
When the StatsD is placed in a suitable location, please edit `config.js` to configure StatsD to send data to InfluxDB. Possible example is as below:

``` js
1.{
2.    influxdb: {
3.        host: '127.0.0.1', // InfluxDB host. (default 127.0.0.1)
4.        port: 8086, // InfluxDB port. (default 8086)
5.        database: 'demo',  // InfluxDB database instance. (required)
6.        username: 'root', // InfluxDB database username. (required)
7.        password: 'root', // InfluxDB database password. (required)
8.        flush: {
9.            enable: true // Enable regular flush strategy. (default true)
10.        },
11.        proxy: {
12.            enable: false, // Enable the proxy strategy. (default false)
13.            suffix: 'raw', // Metric name suffix. (default 'raw')
14.            flushInterval: 1000 // Flush interval for the internal buffer.
15.                // (default 1000)
16.        }
17.    },
18.    port: 8125, // StatsD port.
19.    backends: ['./backends/console’, 'statsd-influxdb-backend'],
20.    debug: true,
21.    legacyNamespace: false,
22.    keyNameSanitize: false
23.}
```
Please note that the key word **backends**, which specifies the destination of the metrics data, and the dependency could be installed as below:

``` bash
1.$ cd statd
2.
3.$ npm install statsd-influxdb-backend -d
4.
5.$ npm install

```

`flushInterval` is set to send batch of information to InfluxDB every 10s by default. `keyNameSanitize`, please set it to `false` when you are connecting InfluxDB or have some special characters in the name tags.

## Grafana

Configuration for Grafana mainly contains two sections, one is for `Data Sources`, and another is for templating/query. I will focus on configure the data source in the article.

Since the DataSource can only be configured after organisation is created, it would be better to sign-up as root and create a default organisation. Then change the anonymous auth settings with your newly-created org. This will allow users without login view the stats information, but they won't be able to modify the charts'.

If you created the root user (you may call it whatever you like), and login with the detail, you will find the DataSource section looks like this snapshot:

![](/images/grafana/dataSource.png)

Select InfluxDB 0.9 as the latest (or whatever your influxDB version is), and then enter details you set for your InfluxDB instance. In the dashboard you may create a new graph point to your new data source.

## InfluxDB
Once database service is started, the default port for Admin Web UI is `8083`, and DataSource Port is `8086`.

In the admin UI, the dropdown menu shows some template query for basic operation. As mentioned before influxDB offers a [SQL-like query style](https://docs.influxdata.com/influxdb/v0.9/concepts/crosswalk/). Simply put, `MEASUREMENTS` is equivalent to `TABLE`, `TAGS` is equivalent to `COLUMNS`, and `POINTS` is equivalent to `ROW`. The above link shows a comprehensive induction on how to query InfluxDB.

This [link](https://docs.influxdata.com/influxdb/v0.9/tools/shell/) shows how to connect to InfluxDB with command-line, if you prefer a geeky way. To find a full document on how InfluxDB works in detail, please click here (Google Search always points to older version).

Once influxDB is installed, it would fairly easy to use. No much settings required.

![](/images/grafana/influxdbSetting.png)

The `localhost` is the server where you installed the influxDB, and `8086` is the default port for querying data.

**Please leave a comment if you have any questions or correct me if there is an update on the new change for these applications.**