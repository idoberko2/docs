# 8. Create a data retention policy

An intrinsic part of working with time-series data is that the relevance of
highly granular data diminishes over time. New data is accumulated and old data
is rarely, if ever, updated. It is therefore often desirable to delete old raw
data to save disk space.

<highlight type="tip"> 
In practice, old data is often downsampled first, such
that a summary of it is retained (for example in a continuous aggregate), while
the raw data points are then discarded via data retention policies. 
</highlight>

Just like for continuous aggregates and compression, TimescaleDB provides an
automated policy to drop data according to a schedule and defined rules.
Automated data retention policies (along with compression and continuous
aggregates) give you more control over the amount of data you retain and its
granularity for specific time periods. These policies are "set it and forget it"
in nature, meaning less hassle for maintenance and upkeep.

For example, here is a data retention policy which drops chunks consisting of
data older than 25 years from the hypertable `weather_metrics`:

```sql
-- Data retention policy
-- Drop data older than 25 years
SELECT add_retention_policy('weather_metrics', INTERVAL '25 years');
```


And just like with continuous aggregates and compression policies, you can see
information about retention policies and their job statistics from the
following informational views:

```sql
-- Informational view for policy details
SELECT * FROM timescaledb_information.jobs;
-- Informational view for stats from run jobs
SELECT * FROM timescaledb_information.job_stats;
```


## Manual data retention

Dropping chunks is also useful when done on a one-off basis. One such case is
deleting large swathes of data from tables—this can be costly and slow if done
row-by-row using the standard `DELETE` command. Instead, TimescaleDB provides a
`drop_chunks` function that quickly drops data at the granularity of chunks
without incurring the same overhead.

```sql
-- Manual data retention
SELECT drop_chunks('weather_metrics', INTERVAL '25 years');
```


This drops all chunks from the hypertable that only contain data older than 25
years. It does not delete individual rows within chunks. In other words, if a
chunk contains a mix of data older and newer than 25 years, all its rows are
retained. 

## Downsampling

We can combine continuous aggregation and data retention policies to implement
downsampling on our data. We can downsample high-fidelity raw data into summaries
via continuous aggregation and then discard the underlying raw observations from
our hypertable, while retaining our aggregated version of the data.

We can also take this a step further, by applying data retention policies (or
using `drop_chunks`) on continuous aggregates themselves, since they are a special
kind of hypertable. The only restriction at this time is that you cannot apply
compression or continuous aggregation to these hypertables.

## Learn more about data retention

For more details and best practices on data retention and automated data retention
policies, see the [Data Retention docs][data-retention].

[data-retention]: /how-to-guides/data-retention