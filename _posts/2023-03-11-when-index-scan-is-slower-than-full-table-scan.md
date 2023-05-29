---
title: When index scan is slower than full table scan
date: 2023-05-20 9:30:00 +0700
categories: [SQL]
tags: [database]
---

Yesterday, I woke up early, as I do on many other days, and performed some checks on an alert system. While some alerts were annoying, a critical one appeared right in front of me - a SQL query that examined the entire table, retrieving ONLY 8 rows! Here is the log containing some information I obtained, but I've omitted certain details for privacy reasons.

```
# Query_time: 11.676028  
# Lock_time: 0.000002 
# Rows_sent: 8  Rows_examined: 2276066
use production;
SET timestamp=1684378801;
SELECT p.user_id
FROM products AS p   
WHERE p.active
GROUP BY p.user_id;
```

Almost time, I will assume that the query was executed on a non-indexed column, specifically either `p.user_id` or `p.active`. However, to my surprise, this time was different! I noticed that we already had an index on the `p.user_id` column, yet we were still examining over two million rows!

Ah, something unusual was happening. I quickly grabbed my coffee and began a more thorough investigation.

## Preparing

- The SQL database I am working with is MySQL version  8.0.28
- The table size at the time I performed my check was

    ```SQL
    SELECT count(id) FROM products
    -- count(id): 2281099
    ```

## Explain the query

First of all, to ensure that the SQL query utilized the index on column `p.user_id`, I executed a simple `EXPLAIN` query with

```SQL
EXPLAIN SELECT p.user_id
FROM products AS p   
WHERE p.active
GROUP BY p.user_id;
``` 

And I obtained the output in the format specified by [MySQL Explain Output](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html)

```json
{
  "id" : 1,
  "select_type" : "SIMPLE",
  "table" : "p",
  "partitions" : null,
  "type" : "index",
  "possible_keys" : "IX_products_user_id",
  "key" : "IX_products_user_id",
  "key_len" : "146",
  "ref" : null,
  "rows" : 2252020,
  "filtered" : 90.0,
  "Extra" : "Using where"
}
```

The interesting thing is that we used the index key `IX_products_user_id`, but despite that, more than two million rows were examined and filtered, resulting in a 90 percent reduction. This means that we loaded the entire table into memory and then filtered out 90 percent of the records to obtain a smaller set.

## Explain the query with analyzer

To obtain a better understanding of what the database has done with the data, I proceeded to use the same query but with the `ANALYZE` option

```SQL
EXPLAIN ANALYZE SELECT p.user_id
FROM products AS p   
WHERE p.active
GROUP BY p.user_id;
``` 

Then it took me approximately ... 15 seconds to obtain an explanation. Something is clearly going wrong with that query.

```
-> Limit: 200 row(s)  (cost=5086.80 rows=200) (actual time=31904.827..37635.793 rows=8 loops=1)
    -> Group (no aggregates)  (cost=5086.80 rows=33925) (actual time=31904.825..37635.788 rows=8 loops=1)
        -> Filter: (0 <> p.active)  (cost=1587.60 rows=33925) (actual time=28788.416..37635.374 rows=825 loops=1)
            -> Index scan on p using IX_products_user_id  (cost=1587.60 rows=37694) (actual time=38.097..37491.597 rows=2281100 loops=1)
```

> Limit: 200 row(s) is what DBeaver added to my query by default and you can ignore it 
{: .prompt-info }

The output format `actual time=38.097..37491.597` indicates the time taken to retrieve the first row and the time taken to retrieve all rows. This outcome aligns with our expectations since we are fetching over two million rows. However, it is not desirable, as we anticipated a faster execution time considering the use of an index. Taking more than 15 seconds to fetch rows with an index is indeed suboptimal.

Let me summarize the key points we have gathered so far:

- We are utilizing an index on a column of a table
- We are fetching the entire set of rows from the table because of business need (assuming it based on the original query)
- The execution time appears reasonable but doesn't align with our expectations

Interestingly, this situation reminds you of a book I read a long time ago called [Use The Index, Luke](https://use-the-index-luke.com/sql/where-clause/the-equals-operator/slow-indexes-part-ii)

## Explain and analyze without index

We will explain everything without utilizing the index on the column `p.user_id`. After did some googling, I found a method to disable or ignore the index, as discussed on [StackExchange](https://dba.stackexchange.com/a/110713)

Explain: 

```SQL
EXPLAIN SELECT p.user_id
FROM products AS p   
WHERE p.active
GROUP BY p.user_id;

-- {
--   "id" : 1,
--   "select_type" : "SIMPLE",
--   "table" : "p",
--   "partitions" : null,
--   "type" : "ALL",
--   "possible_keys" : null,
--   "key" : null,
--   "key_len" : null,
--   "ref" : null,
--   "rows" : 2252027,
--   "filtered" : 90.0,
--   "Extra" : "Using where; Using temporary"
-- }
``` 

Explain Analyze:

```SQL
EXPLAIN ANALYZE SELECT p.user_id
FROM products AS p USE INDEX ()      
WHERE p.active 
GROUP BY p.user_id;

-- -> Limit: 200 row(s)  (cost=446934.14..446936.62 rows=200) (actual time=1152.400..1152.402 rows=8 loops=1)
--     -> Table scan on <temporary>  (cost=0.01..25337.80 rows=2026824) (actual time=0.002..0.003 rows=8 loops=1)
--         -> Temporary table with deduplication  (cost=446934.14..472271.92 rows=2026824) (actual time=1152.399..1152.401 rows=8 loops=1)
--             -> Filter: (0 <> p.active)  (cost=244251.70 rows=2026824) (actual time=0.285..1151.343 rows=825 loops=1)
--                 -> Table scan on p  (cost=244251.70 rows=2252027) (actual time=0.071..1021.024 rows=2281105 loops=1)
```

Indeed, it is interesting to observe that the actual time for fetching both a single row (`0.071`) and all rows (`1021.024`) without utilizing the index is smaller compared to the query that makes use of the index.

## Why?

The key factor here is the need to fetch a large number of rows, such as the entire rows of the table in this case. In my opinion, two factors contribute to making a full table scan faster:

- With an index scan, the database needs to traverse the B-Tree structure to retrieve the row's ID and then fetch the actual data. This process involves two steps, and the access pattern is random, which can be slower compared to a full table access pattern.
- On the other hand, with a full table scan, the database only needs to perform a single step, which is a sequential scan of the table.

## Solution

Based on the results from the statistics, it appears that in this particular case, forcing the database to ignore the index can improve the query execution time.

## Notes

Why aren't we using `DISTINCT` in this case? And here is the explanation for this type of query

```SQL
EXPLAIN ANALYZE SELECT DISTINCT p.user_id
FROM products AS p
WHERE p.active 

-> Limit: 200 row(s)  (cost=446934.71..446937.19 rows=200) (actual time=1151.773..1151.775 rows=8 loops=1)
    -> Table scan on <temporary>  (cost=0.01..25337.83 rows=2026827) (actual time=0.002..0.003 rows=8 loops=1)
        -> Temporary table with deduplication  (cost=446934.71..472272.52 rows=2026827) (actual time=1151.772..1151.774 rows=8 loops=1)
            -> Limit table size: 200 unique row(s)
                -> Filter: (0 <> p.active)  (cost=244252.00 rows=2026827) (actual time=0.286..1150.798 rows=825 loops=1)
                    -> Table scan on p  (cost=244252.00 rows=2252030) (actual time=0.074..1020.530 rows=2281108 loops=1)
```

