# Indexes on expressions

* Official [documentation](https://www.postgresql.org/docs/current/indexes-expressional.html)

Expression-based index are just as regular field-based indexes but working on the execution of an arbitrary expression. If your application is using them on `WHERE` or `GROUP BY` clauses you may want to set up them to speed their execution.

## Set up

Just open `https://betis.carto.io` and configure a CARTO account to point to your `commitconf-XX` user and check that the table `bkmappluto` exists and has the three default indexes:

* `bkmappluto_pkey`
* `bkmappluto_the_geom_idx`
* `bkmappluto_the_geom_webmercator_idx`

### For the lecturers

Remember to remove the exercise index if present.

```sql
drop index bkmappluto_gist_centroid_idx;
```

## Indexed check

Let's start with a simple query that returns the blocks of the `bkmappluto` table that are inside a bounding box. We will use `ST_MakeEnvelope` to define the box and the `&&` operator that PostGIS converts into an indexed check.

```sql
explain analyze
select count(*)
  from "commitconf-01".bkmappluto
  where ST_MakeEnvelope(-73.968962,40.678067,-73.948545,40.686650) && the_geom
```

```text
Aggregate  (cost=10411.78..10411.79 rows=1 width=8) (actual time=13.925..13.925 rows=1 loops=1)
  ->  Bitmap Heap Scan on bkmappluto  (cost=132.48..10402.68 rows=3638 width=0) (actual time=1.977..13.503 rows=3881 loops=1)
        Recheck Cond: ('0103000000010000000500000046443179037E52C098BF42E6CA56444046443179037E52C032E6AE25E4574440327216F6B47C52C032E6AE25E4574440327216F6B47C52C098BF42E6CA56444046443179037E52C098BF42E6CA564440'::geometry && the_geom)
        Heap Blocks: exact=3562
        ->  Bitmap Index Scan on bkmappluto_the_geom_idx  (cost=0.00..131.57 rows=3638 width=0) (actual time=1.522..1.522 rows=3881 loops=1)
              Index Cond: ('0103000000010000000500000046443179037E52C098BF42E6CA56444046443179037E52C032E6AE25E4574440327216F6B47C52C032E6AE25E4574440327216F6B47C52C098BF42E6CA56444046443179037E52C098BF42E6CA564440'::geometry && the_geom)
Planning time: 0.263 ms
Execution time: 14.009 ms
```

As you can see the request runs in just 14 milliseconds and taking the leverage of the geometry index `bkmappluto_the_geom_idx`.

### Non indexed check

Now let's try a slightly different query, instead of checking against the stored geometry, let's do it with their centroids.

```sql
explain analyze
select count(*)
  from "commitconf-01".bkmappluto
  where ST_MakeEnvelope(-73.968962,40.678067,-73.948545,40.686650) && ST_Centroid(the_geom)
```

```text
Finalize Aggregate  (cost=120218.54..120218.55 rows=1 width=8) (actual time=241.474..241.474 rows=1 loops=1)
  ->  Gather  (cost=120218.12..120218.53 rows=4 width=8) (actual time=241.422..241.467 rows=5 loops=1)
        Workers Planned: 4
        Workers Launched: 4
        ->  Partial Aggregate  (cost=119218.12..119218.13 rows=1 width=8) (actual time=217.983..217.984 rows=1 loops=5)
              ->  Parallel Seq Scan on bkmappluto  (cost=0.00..119183.53 rows=13836 width=0) (actual time=0.440..217.826 rows=749 loops=5)
                    Filter: ('0103000000010000000500000046443179037E52C098BF42E6CA56444046443179037E52C032E6AE25E4574440327216F6B47C52C032E6AE25E4574440327216F6B47C52C098BF42E6CA56444046443179037E52C098BF42E6CA564440'::geometry && st_centroid(the_geom))
                    Rows Removed by Filter: 54594
Planning time: 0.109 ms
Execution time: 253.985 ms
```

This query runs in 254 milliseconds, which is almost **20 times** slower.

### Create the index over the function

Now, if we create the same

```sql
create index bkmappluto_gist_centroid_idx on "commitconf-01".bkmappluto using gist(ST_Centroid(the_geom))
```

### Expression based indexed query

```sql
explain analyze
select count(*)
  from "commitconf-01".bkmappluto
  where ST_MakeEnvelope(-73.968962,40.678067,-73.948545,40.686650) && ST_Centroid(the_geom)
```

```text
Finalize Aggregate  (cost=51119.48..51119.49 rows=1 width=8) (actual time=33.977..33.977 rows=1 loops=1)
  ->  Gather  (cost=51119.06..51119.47 rows=4 width=8) (actual time=10.244..33.966 rows=5 loops=1)
        Workers Planned: 4
        Workers Launched: 4
        ->  Partial Aggregate  (cost=50119.06..50119.07 rows=1 width=8) (actual time=1.999..1.999 rows=1 loops=5)
              ->  Parallel Bitmap Heap Scan on bkmappluto  (cost=1933.19..50084.47 rows=13836 width=0) (actual time=0.501..1.915 rows=749 loops=5)
                    Recheck Cond: ('0103000000010000000500000046443179037E52C098BF42E6CA56444046443179037E52C032E6AE25E4574440327216F6B47C52C032E6AE25E4574440327216F6B47C52C098BF42E6CA56444046443179037E52C098BF42E6CA564440'::geometry && st_centroid(the_geom))
                    Heap Blocks: exact=3442
                    ->  Bitmap Index Scan on bkmappluto_gist_centroid_idx  (cost=0.00..1919.36 rows=55343 width=0) (actual time=1.736..1.736 rows=3745 loops=1)
                          Index Cond: ('0103000000010000000500000046443179037E52C098BF42E6CA56444046443179037E52C032E6AE25E4574440327216F6B47C52C032E6AE25E4574440327216F6B47C52C098BF42E6CA56444046443179037E52C098BF42E6CA564440'::geometry && st_centroid(the_geom))
Planning time: 0.130 ms
Execution time: 37.802 ms
```

Now the query runs in 37 milliseconds, an almost 7x improvement.

## Other use cases

* Using casts on your filter to convert between strings and numbers: `where rooms::int = 3`
* Case insesitive queries where you use `lower(name) = "peter"`

## Note

To be able to create this type of indexes your function call needs to bemarked as `IMMUTABLE`, otherwise their results would depend on the client connected to the database. This is the case for functions that are dependant of the locale settings (time and time zone conversions, currency, etc).
