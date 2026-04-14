+++
date = '2026-04-10T11:33:35-07:00'
draft = true
title = 'Patterns in Stream Processing: Deduplication'
tags = ['flink', 'patterns', 'deduplication']
+++

I generally recommend Flink SQL to developers starting a new stream processing project because it includes a powerful collection of built-in operators. These built-in operators can be composed together to satisfy the requirements of most use cases — provided you are able to see how to decompose your use case into a combination of these building blocks.

These built-in operators can do things like transforming, enriching, or correlating events, aggregating them in windows, and looking for patterns. It's common to perform several of these operations together in one application.

The first stage of many streaming applications is deduplication. If you're going to need to deduplicate, you'll know it, but for many developers the SQL idiom for deduplication is kinda scary looking, and it obscures some interesting details. Let's dig in.

## Why on earth would I need to deduplicate? After all, Flink promises exactly-once.

Flink's exactly-once guarantees have a cost attached: increased latency. For some use cases, it makes sense to settle for at-least-once semantics in exchange for increased responsiveness — and that means living with the possibility of duplicates.

Fortunately, even with at-least-once semantics, duplicates should be rare. Typically, they will only occur in conjunction with a restart, i.e., while recovering from a failure. Because of that, duplicates should follow soon after the original event.

## How expensive is deduplication?

With Flink SQL, we're not generally too concerned about raw CPU performance. Instead, "expensive" is more likely to mean "uses a lot of state", "adds considerable latency", or "introduces a lot of complexity".

Deduplication is inherently stateful, since recognizing a duplicate requires having saved the original event when it was first processed. The worst case can be very expensive: if you need to ensure that no duplicates ever occur, then every unique event must be retained forever.

However, for most use cases, this is unnecessary. If you are confident that duplicates will arrive soon after the original event, then you can limit how long events are kept in Flink's state store.

## How to (and how not to) do deduplication

One approach is to look at deduplication as a windowing operation, where the objective is to emit only the original event, and drop any duplicates. Assuming that the original and any duplicates will share the same timestamp, they will all land in the same window — so this will work. But it's not the best way to handle this, because

- This makes deduplication a temporal operation, i.e., one that depends on watermarks. As a consequence, any overly out-of-order events will be dropped by Flink SQL.

- Windows don't emit any results until they are closed, so this approach adds unnecessary latency.

Even worse is what happens if the timestamps are assigned by the Kafka brokers. Then you won't be able to guarantee that the original and the duplicates will be assigned to the same window, and this whole strategy falls apart.

The right way to approach deduplication in Flink SQL is with an OVER aggregation. OVER is a construct in Flink SQL that groups together an incoming row with some or all previous rows in the same *partition*. It produces an output row for every input row.

## The setup I used for my experiments

This will be easier to explain in the context of an example. In order to play with this, I needed a table with some duplicates, which I created like this:

```sql
CREATE TABLE trades (
  `trade_id` STRING NOT NULL,
  `ticker` STRING NOT NULL,
  `quantity` INT NOT NULL,
  `price` DECIMAL(10,2) NOT NULL,
  `created_at` TIMESTAMP_LTZ(3) NOT NULL METADATA FROM 'timestamp',
  WATERMARK FOR `created_at` AS `created_at`
) WITH (
  'connector' = 'faker',
  'fields.trade_id.expression' = '#{Internet.UUID}',
  'fields.ticker.expression' = '#{regexify ''[A-Z]{4}''}',
  'fields.quantity.expression' = '#{Number.numberBetween ''100'',''10000''}',
  'fields.price.expression' = '#{Number.randomDouble ''2'', ''10'', ''500''}',
  'rows-per-second' = '1'
);
```

```sql
CREATE VIEW trades_with_dups_view AS (
  SELECT * FROM trades
  UNION ALL
  SELECT * FROM trades WHERE price < 100
);
```

```sql
CREATE TABLE trades_with_duplicates (
  WATERMARK FOR `created_at` AS `created_at`
)
DISTRIBUTED BY (trade_id) INTO 2 BUCKETS
AS (
  SELECT * FROM trades_with_dups_view
);
```

## The deduplication query

This is the query I described earlier as being "kinda scary looking"; it will deduplicate this trades_with_duplicates table.

```sql
SELECT
  trade_id,
  ticker,
  quantity,
  price,
  created_at
FROM (
  SELECT *, ROW_NUMBER()
    OVER (PARTITION BY trade_id
          ORDER BY created_at ASC) AS row_num
  FROM trades_with_duplicates
)
WHERE row_num = 1;
```

## Conceptually, how does this work?

Before looking at how the Flink SQL runtime actually executes this query, let's try to understand why it makes sense to model deduplication this way.

`PARTITION BY trade_id`

is effectively saying that we want to deduplicate solely on the basis of the trade_id column — the trades_with_duplicates table is being partitioned so that each slice of the table contains all of trades with a specific trade_id.

`ORDER BY created_at ASC`

Then within each partition, the Trade events with the same trade_id are to be sorted by created_at in ascending order (from oldest to most recent). (Later, we'll see that in the case of deduplication, the Flink SQL runtime isn't actually doing any sorting, but bear with me.)

The inner subquery (below) is adding a column named row_num to trades_with_duplicates with the ROW_NUMBER() for each row. This indicates that row's position within the sorted partition for that trade.

```sql
SELECT *, ROW_NUMBER()
  OVER (PARTITION BY trade_id ORDER BY created_at ASC) AS row_num
FROM trades_with_duplicates
```

For each distinct trade_id, the row_num will have the value 1 for the original event, and then it will continue counting upwards — 2, 3, 4, etc. — for any duplicates. Deduplication is simply a matter of filtering the results of this subquery:

```sql
SELECT trade_id, ticker, quantity, price, created_at
FROM (
  SELECT *, ROW_NUMBER()
    OVER (PARTITION BY trade_id ORDER BY created_at ASC) AS row_num
  FROM trades_with_duplicates
)
WHERE row_num = 1;
```

## How does this really work?

One of the core concepts in the Flink runtime is the idea of key-partitioned data streams, which involves shuffling events between workers so that, for each key, all events with that key are processed by the same worker. So it's easy to arrange for all events with the same trade_id to be processed by the same instance of the deduplication operator.

Flink SQL supports two different styles for deduplication: (1) emit the first event for each trade_id and ignore any duplicates that follow, or (2) emit the first event for each trade_id and subsequently update that result with the latest info as duplicates are processed. The first approach is the one shown in the example above, and is specified by ORDER BY ... ASC; the second approach is what the runtime does when you specify ORDER BY ... DESC instead.

If you go and read the Flink source code, you'll find the implementation of deduplication has ended up being much more complex than you might expect, because there are so many different cases being considered: which style of deduplication is being used, whether the input stream is append-only or can include updates and deletions, and so on.

When I dug into this, one thing surprised me: the attribute used in the ORDER BY clause doesn't have to be a time attribute.

Normally when Flink SQL is running in streaming mode, a query like

```sql
SELECT * FROM trades ORDER BY created_at ASC;
```

can only be executed if the primary sort order of the table is ascending on a time attribute. For general purpose sorting, this restriction makes sense — anything else would be intractably expensive.

Now the [documentation for deduplication](https://nightlies.apache.org/flink/flink-docs-release-2.2/docs/dev/table/sql/queries/deduplication/) states that

> ORDER BY time_attr [asc|desc]: Specifies the ordering column, it must be a time attribute. ... Ordering by ASC means keeping the first row, ordering by DESC means keeping the last row.

Given this (incorrect) statement in the docs, I had assumed that the deduplication operator needs to be able to sort. But that's not true. All that matters is that the column used for ordering is comparable, so that the runtime can keep track of either the smallest or largest row, as measured by this column.

So, for example

`ORDER BY created_at [ASC|DESC]`

means keeping either the first or latest row, and

`ORDER BY price [ASC|DESC]`

means keeping either the cheapest or most expensive row.

From a practical perspective this is great news, because it means that deduplication doesn't depend on watermarks, and won't drop late events.

## Using state TTL to avoid unbounded state retention

To prevent Flink from blowing up with too much state, you'll want to set an idle state timeout on the state used by deduplication. This will allow the Flink runtime to expire the state for stale keys.

This can be done with a session-level configuration setting, or inside the query using a SQL hint. I prefer the latter approach, since it's more granular, and gets all of the business logic together in one place. Here's a complete example:

```sql
INSERT INTO trades_deduped
SELECT /*+ STATE_TTL('trades_with_duplicates' = '24 HOURS') */
  trade_id,
  ticker,
  quantity,
  price,
  created_at
FROM (
  SELECT *, ROW_NUMBER()
    OVER (
      PARTITION BY trade_id
      ORDER BY created_at ASC
    ) AS row_num
  FROM trades_with_duplicates
)
WHERE row_num = 1;
```

After 24 hours (of wall-clock time) with no updates for a given trade_id, the entry for that trade will be cleared.
