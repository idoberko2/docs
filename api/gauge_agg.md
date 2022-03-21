# gauge_agg() <tag type="toolkit" content="Toolkit" /><tag type="experimental" content="Experimental" />
Produces a `GaugeSummary` that can be used to accumulate gauge data for further
calculations. 
```sql
gauge_agg (
    ts TIMESTAMPTZ,
    value DOUBLE PRECISION
) RETURNS GaugeSummary
```

<highlight type="warning">
Experimental features could have bugs. They might not be backwards compatible,
and could be removed in future releases. Use these features at your own risk,
and do not use any experimental features in production.
</highlight>

For more information about counter and gauge aggregation functions, see the
[hyperfunctions documentation][hyperfunctions-counter-agg].

## Required arguments

|Name|Type|Description
|-|-|-|
|`ts`|`TIMESTAMPTZ`|The time at each point|
|`value`|`DOUBLE PRECISION`|The value at that timestamp|

Only `DOUBLE PRECISION` values are accepted for the `value` parameter. For gauge
data stored as other numeric types, cast it to `DOUBLE PRECISION` when using the
function.

<highlight type="note">
Both `ts` and `value` can be `NULL`, but the aggregate is not evaluated on
`NULL` values. If the aggregate receives only a `NULL` value, it returns
`NULL`. If it receives both non-`NULL` and `NULL` values, it ignores the `NULL`
values. Both `ts` and `value` must be non-`NULL` for the row to be included.
</highlight>

## Optional arguments

|Name|Type|Description|
|-|-|-|
|`bounds`|`TSTZRANGE`|The largest and smallest possible times that can be input to the aggregate. Calling with `NULL`, or leaving out the argument, results in an unbounded `GaugeSummary`|

<highlight type="important">
Bounds are required for extrapolation, but not for other accessor functions.
</highlight>

## Returns

|Column|Type|Description|
|-|-|-|
|`gauge_agg`|`GaugeSummary`|A `GaugeSummary` object that can be passed to accessor functions or other objects in the gauge aggregate API|

The returned `GaugeSummary` can be used as an input to the same accessor
functions as `CounterSummary`, except for `num_resets`.

## Sample usage
Create a gauge summary from time-series data that has a timestamp, `ts`, and a
gauge value, `val`. Get the instantaneous rate of change from the last 2 time
intervals using the `irate_right` accessor:
```sql
WITH t as (
    SELECT
        time_bucket('1 day'::interval, ts) as dt,
        gauge_agg(ts, val) AS gs
    FROM foo
    WHERE id = 'bar'
    GROUP BY time_bucket('1 day'::interval, ts)
)
SELECT
    dt,
    irate_right(gs)
FROM t;
```

[hyperfunctions-counter-agg]: timescaledb/:currentVersion:/how-to-guides/hyperfunctions/counter-aggregation/