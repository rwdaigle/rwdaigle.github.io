---
layout: post
title:  "Time-Series Database Design with InfluxDB"
date:   2013-08-13
author: ryan
tags:
  - time series
  - influxdb
category: blog
blog: true
---

Here at [Spreedly](https://spreedly.com/) we've recently started using the time series database [InfluxDB](http://influxdb.com/) to store a variety of customer activity metrics. As with any special purpose database, using and designing for a time-series database is quite different than what you may be used to with structured (SQL) databases. I'd like to describe our experience designing our InfluxDB schema, the mistakes we made, and the conclusions we've come to based on those experiences.

## The mark

Consider the following scenario, closely resembling Spreedly's: You run a service that lets your customers transact against a variety of payment gateways. You charge for this service on two axes – by the number of gateways provisioned and the number of credit cards stored. For any point in time you want to know how many of both each of your customers has for their account.

Initially we setup two series (InfluxDB's term for a collection of measurements, organizationally similar to a SQL database table) to store the total number of each item per account:

* **gateway.account.sample**
* **payment-method.account.sample**

On some regular interval we'd collect the number of gateways and payment methods (credit cards) for each account and store it in the respective series. Each measurement looked like:

```json
{
  "time": 1400803300,
  "value": 2,
  "account_key": "abc123"
}
```

`time` is required by InfluxDB and is the [epoch time](http://en.wikipedia.org/wiki/Unix_time) of the measurement. The `value` is our value of the measurement at that time, and `account_key` is an additional property of that measurement.

Simple enough. This approach felt good and we went to production with this schema. That's when we learned our first lesson...

## Time-scoped queries

The first app that used the data in InfluxDB was our customer Dashboard product. It displays all your transactions and a simple view of your current billing counts (number of gateways and number of stored payment methods). Dashboard simply queried for the most recent measurement from each series for the current account:

```sql
select value
  from gateway.account.sample
  where account_key = 'abc123'
  limit 1
```

*Since results in InfluxDB are ordered by default most recent first, the `limit 1` clause ensures only the most recent measurement is returned for that customer (account).*

All was fine initially, but as our dataset grew into the hundreds of thousands entries for each series we noticed our queries were taking quite some time to complete - about a constant 5s for every account. It turns out that these queries were incurring a full table scan, hence the constant (poor) performance.

> Avoid a full table scan by always time-scoping your queries

In InfluxDB, the non-time fields aren't indexed, meaning any queries that filter based on them require a full table scan (even if you're only fetching a single result). The way to avoid a full table scan is to always time-scope your queries. Knowing this we modified our queries to only query against the previous 2 days worth of data (enough time to capture the most recent input):

```sql
select value
  from gateway.account.sample
  where time > now() - 2d
    and account_key = 'abc123'
  limit 1
```

Adding the `where time > now() - 2d` clause ensures that the query operates against a manageable set of data and avoids a full table scan. This dropped our query times from 5s (and growing) down to a steady 100ms - 200ms. (Keep in mind this is a remote instance of InfluxDB, meaning the bulk of that is in connection setup and network latency.)

![](http://f.cl.ly/items/2b2A2T1k1P381v04170V/Image%202014-08-07%20at%205.17.05%20PM.png)

*InfluxDB response time reduction using time-scoped queries. Y-axis truncated for maximum obfuscation.*

Obviously your use-case may differ wildly from ours. If your data is collected at unknown intervals, or in real-time, you don't have the luxury of limiting your queries to a known window of time. In these situations it is wise to think about how to segment your data into series for optimal performance.

## Series granularity

How many series should you have? How much data should you store in each series? When should you break out queries into their own series? These are all common questions when designing your time-series schema and, unfortunately, there is no concrete right or wrong answer. However, there are some good rules of thumb to keep in mind when structuring your data.

Continuing from our previous example: We were now using time-scoped queries to get the total number of gateways and cards for each account. While we were seeing good performance, each query was operating against a single series that contained data for all accounts. The query's `account_key` condition was responsible for filtering the data by account:

```sql
select value
  from gateway.account.sample
  where time > now() - 2d
    and account_key = 'abc123'
  limit 1
```

As even this already time-scoped set of data grows, querying against a non-indexed field will start to become an issue. Queries whose conditions eliminate a large percentage of the data within the series should be extracted out into their own series. E.g., in our case we have a query that gets a single account's count of stored gateways to the exclusion of all the other accounts. This is an example of a query that filters out the majority of the data in a series and should be extracted so each account has its own series.

> Series are cheap. Use them liberally to isolate highly conditional data access.

If you're coming from a SQL-based mindset, the thought of creating one series per account might seem egregious. However, it's perfectly acceptable in time-series land. So that's what we did - we starting writing data from each account into its own series (with each series' name including the account key). Now, when querying for an account's total number of stored gateways we do:

```sql
select value
  from account-abc1234.gateway.sample
  where time > now() - 2d
    ...
```

Since you have to know the key in question to access the right series, this type of design is most common with primary (or other well-known keys). But... not only can series be segmented by key, segmenting by time period is also possible. While not useful in our specific situation, [you can imagine](http://en.wikiquote.org/wiki/Billy_Madison#Dialogue) segmenting data into monthly series, e.g., `201407.gateway.sample` or some other period, depending on your access pattern.

![](http://f.cl.ly/items/3v2r0h123I2p2v0Q2q0R/chris-farley-you-could-imagine-o.gif)

*You can imagine...*

## Multi-purpose data

At this point your series are lean and efficient, well-suited for accessing a single type of query and data. However, sometimes life isn't that clean and you have one set of data that needs to be accessed in many different ways.

For instance, at Spreedly, we'd like to have a business-level set of metrics available that shows the total number of gateways and payment-methods *across all customers*. We could just dump summary-level data into a new series (not a terrible idea), but we're already collecting this data on a customer-level. It'd be nice not to have to do two writes per measurement.

> Use continuous queries to re-purpose broad series by access pattern

Fortunately, InfluxDB has a feature called [continuous queries](http://influxdb.com/docs/v0.8/api/continuous_queries.html) that lets you modify and isolate data from one series into one or more other dependent series. Continuous queries are useful when you want to "rollup" time-series data by time period (e.g., get the 99th percentile service times across 5, 10 and 15 minute periods) and also to isolate a subset data for more efficient access. This latter application is perfect for our use-case.

To use continuous queries to support both summary and account-specific stats we need to create the parent series that contain measurements for each account.

```json
{
  "time": 1400803300,
  "value": 2,
  "account_key": "abc123"
},
{
  "time": 1400803300,
  "value": 7,
  "account_key": "def456"
}
```

We can access this series directly to obtain the business-level stats we need across all customers:

```sql
select sum(value)
  from gateway.account.sample
  where time > now() - 1d
```

With continuous queries we can also use this parent series to spawn several "fanout" queries that isolate the data by account (replicating the account-specific series naming scheme from earlier):

```sql
select value
  from gateway.account.sample
  into account-[account_key].gateway.sample;
```

Notice the `[account_key]` interpolation syntax? This creates one series per account and stores the value field from each measurement into the new account-specific series (retaining the original measurement's time):

```json
{
  "time": 1400803300,
  "value": 2
}
```

```json
{
  "time": 1400803300,
  "value": 7
}
```

With this structure we:

* Only write the data one time into the parent series `gateway.account.sample`
* Can perform summary level queries against this parent series
* Have access to the highly efficient, constantly updated, account-specific, data series `account-def456.gateway.sample` etc...

This is a great use of fanout continuous queries. Also available are regular continuous queries which operate by precomputing expensive `group by` queries. I'll skip over them for now since we're not yet using them at Spreedly, but I encourage you to look at them for your use cases.

## Naming and structure

Series naming and packet structure is a tough topic due to personal preferences, differences in client languages and highly varied access patterns. I'm not going to label the following as best-practices, instead I'll present what we've found at Spreedly, our motivations, and let you decide whether it makes sense for you to apply.

* Come up with a naming structure that conveys both the purpose of the series and the type of data contained within. At Spreedly it's something like (and still evolving): `[key].measured-item.[grouping].measurement-type`. For instance, the series that contains the count of all gateways stored by account is `gateway.account.sample`. The account-specific version is: `account-abc123.gateway.sample`. The `measurement-type` component is highly influenced by the [l2met logging conventions](https://github.com/ryandotsmith/l2met/wiki/Usage#logging-convention) and deserves further discussion.
  * `count` series record a specific number of times something happened in a specific period of time as an integer. Counts can be summed with other counts in the same series to perform time-based aggregations (rollups). Measured number of requests or transactions per minute are an example of a `count` series.
  * `sample` series take a point in time measurement of some metric that supercedes all previous samples of the same series. Sum totals are a good example of this type of series, e.g., total revenue to date, or total number of payment methods. With each measurement in the series, previous measurements are no longer relevant, though they may still be used to track trends over time.
  * `measure` series are similar to `count` series except that instead of being a simple representation of the number of times something happen, they can represent any unit of measure such as ms, Mb etc...  Measurements are be mathmatically operable and can be summed, percentiled, averaged etc... CPU load and response times are examples of `measure` series.
* Often there is a single value that represents the thing being measured, with the rest of the fields being meta-data or conditions. To facilitate re-usable client parsing we've found it nice to use the same field name across all series to represent the value of the measurement. Unsurprisingly, we chose `value`. All our series data contains a `value` field that contains the measurement value. This makes it easy to retrieve, which is especially useful in queries that select across multiple series or even [merge results from multiple series](http://influxdb.com/docs/v0.8/api/query_language.html#merging-series) into a single result set.

There's a lot of subjectivity that goes into database design, independent of the storage [paradigm](http://adland.tv/commercials/verizon-boxymoron-2002-030-usa). While SQL has been around for awhile and has well-known patterns, alternative databases, including time-series databases, are a bit more of a wild west. I'm hoping that by sharing our experiences we can prevent some common mistakes, freeing you up to create all new ones of your own!

*Many thanks to [Paul](https://twitter.com/pauldix) and [Todd](https://twitter.com/toddpersen) and the rest of InfluxDB for their tireless guidance on the subject*
