# Non-relational capabilities

- Array
- JSONB > JSON
- HStore
- Range types, Geometry, XML, etc.

## Array

Official docs:

- <https://www.postgresql.org/docs/10/arrays.html>
- <https://www.postgresql.org/docs/10/functions-array.html>

Useful for lists of values

```sql
CREATE TABLE students (
    name text,
    scores int[][],
    contacts varchar ARRAY -- or varchar[]
);
```

```sql
\d students

Table "public.students"
  Column  |        Type         | Modifiers
----------+---------------------+-----------
 name     | text                |
 scores   | integer[]           |
 contacts | character varying[] |
 ```

```sql
INSERT INTO students
VALUES ('student_name',
        '{{44,93,82,42},{59,74,73,67},{43,54,59,77},{46,45,68,98}}',
        array ['(555)480-9941','(555)738-3707']);
```

### Indexing

```sql
CREATE INDEX idx_scores ON students USING GIN(scores); -- GIN Index (array)
CREATE INDEX idx_contacts ON students USING GIN(contacts); -- GIN Index (array)
```

Special operators -> `<@, @>, =, &&`

```sql
EXPLAIN ANALYZE SELECT name, contacts FROM students WHERE contacts @> '{(555)738-3707}';
```

```text
                                QUERY PLAN
---------------------------------------------------------------------------
 Bitmap Heap Scan on students  (cost=8.01..12.02 rows=1 width=72)
   Recheck Cond: (contacts @> '{(555)738-3707}'::character varying[])
   ->  Bitmap Index Scan on idx_contacts  (cost=0.00..8.01 rows=1 width=0)
         Index Cond: (contacts @> '{(555)738-3707}'::character varying[])
```

```sql
SELECT name, scores FROM students WHERE scores @> '{97}';
```

```text
                        QUERY PLAN
-----------------------------------------------------------
 Seq Scan on students  (cost=0.00..19.00 rows=1 width=234)
   Filter: (scores @> '{97}'::integer[])
```

```sql
CREATE EXTENSION intarray;
```

```sql
CREATE INDEX idx_scores_intarray ON students USING GIN(scores gin__int_ops);
```

```text
                                    QUERY PLAN
----------------------------------------------------------------------------------
 Bitmap Heap Scan on students  (cost=8.00..12.02 rows=1 width=234)
   Recheck Cond: (scores @> '{97}'::integer[])
   ->  Bitmap Index Scan on idx_scores_intarray  (cost=0.00..8.00 rows=1 width=0)
         Index Cond: (scores @> '{97}'::integer[])
```

### [Functions](https://www.postgresql.org/docs/10/functions-array.html)

- Accessor [] (from 1 to n, supports ranges)
- array_dims
- array_length
- array_append
- array_to_string
- array_agg
- unnest

```sql
EXPLAIN (analyze, verbose, costs off, buffers)
 SELECT count(*)
  FROM students
 WHERE 90 < ANY(scores);
```

```sql
SELECT score, count(*)
  FROM students, unnest(scores) as t(score)
GROUP BY score
ORDER BY count DESC
  LIMIT 10;
```

## JSONB > JSON

Official docs:

- <https://www.postgresql.org/docs/10/datatype-json.html>
- <https://www.postgresql.org/docs/10/functions-json.html>

When to use JSONB

- Unstructured data (Event tracking, gaming data, external data sources, etc.)
- Defer design

When not to use JSONB

- When disk is scarce (larger table footprint)
- Slow for aggregations (postgresql statistics do not work)
- Upgrade from JSON lgeacy apps (not backwards compatible)

When to use JSONB over Mongo

- Constraints, JOINs, transactions, operational database, etc.
- Team skills

How we loaded the origin table:

```bash
cd tmp
wget https://github.com/gtmanfred/pg_shard_example/raw/master/customer_reviews_nested_1998.json.gz
gzip -d customer_reviews_nested_1998.json.gz
```

```sql
CREATE TABLE reviews(review jsonb);
\copy reviews FROM 'customer_reviews_nested_1998.json'
VACUUM ANALYZE reviews;
```

For the exercise:

```sql
create table review_sample as select *
       from "commitconf-01".reviews
tablesample system(20);
```

### Data access

```sql
CREATE INDEX on review_sample ((review #>> '{product,category}'));
```

```sql
SELECT
    review #>> '{product,title}' AS title,
    avg((review #>> '{review,rating}')::int)
FROM review_sample
WHERE review #>> '{product,category}' = 'Fitness & Yoga'
GROUP BY 1 ORDER BY 2;
```

```text
                       title                       |        avg
---------------------------------------------------+--------------------
 Kathy Smith - New Yoga Challenge                  | 1.6666666666666667
 Pumping Iron 2                                    | 2.0000000000000000
 Kathy Smith - New Yoga Basics                     | 3.0000000000000000
 Men Are from Mars, Women Are from Venus           | 4.0000000000000000
 Kathy Smith - Functionally Fit - Peak Fat Burning | 4.5000000000000000
 Kathy Smith - Pregnancy Workout                   | 5.0000000000000000
(6 rows)
```

```sql
CREATE INDEX on review_sample USING GIN (review);
```

- JSON @> JSON is a subset
- JSON ? TEXT contains a value
- JSON ?& TEXT[] contains all the values
- JSON ?| TEXT[] contains at least one value

```sql
SELECT
    review #>> '{product,title}' AS title,
    avg((review #>> '{review,rating}')::int)
FROM review_sample
WHERE review @> '{"product": {"category": "Fitness & Yoga"}}'
GROUP BY 1 ORDER BY 2;
```

Smaller and faster INDEX (better for `@>`)

```sql
CREATE INDEX on review_sample USING GIN (review jsonb_path_ops);
```

### Constraints

```sql
ALTER TABLE review_sample
ADD CONSTRAINT
"validate_review_rating" CHECK ((review #>> '{review,rating}')::int <= 5);

INSERT INTO review_sample
VALUES ('{"review": {"rating": 10}}');
```

## HStore

Non-hierarchical key-value strings.

Used extensively in OpenStreetMap.

Official docs:

- <https://www.postgresql.org/docs/10/hstore.html>

JSONB vs HStore

![http://mateuszmarchel.com/blog/2016/06/29/jsonb-vs-hstore-performance-battle/](http://mateuszmarchel.com/images/table_big2.png)

## References

- <https://momjian.us/main/writings/pgsql/non-relational.pdf>
- <https://www.citusdata.com/blog/2016/07/14/choosing-nosql-hstore-json-jsonb/>
- <http://www.craigkerstiens.com/2013/07/03/hstore-vs-json/>
- <https://blog.2ndquadrant.com/jsonb-type-performance-postgresql-9-4/>
- <https://www.enterprisedb.com/es/blog/illustration-jsonb-capabilities-postgres-95>
- <https://www.compose.com/articles/faster-operations-with-the-jsonb-data-type-in-postgresql/>
- <https://www.compose.com/articles/take-a-dip-into-postgresql-arrays/>
- <https://tapoueh.org/blog/2018/04/postgresql-data-types-arrays/>
