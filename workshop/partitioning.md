# Table partitioning

With Postgres 10 and above [declarative partitioning](https://www.postgresql.org/docs/current/ddl-partitioning.html) is supported. That means that a main table can be defined with a partitioning field, then when creating the inherited tables, only the range, list, or hash condition of accepted values need to be specified and that's it. This makes creating partitioned tables very easy and convenient. Partitions can be attached and detached at any time.

On this exercise we take the `reviews` table that stores `JSONB` objects and we are going to generate a partitioned table on our account (schema) using the rating, an internal value of the JSON field, as our criteria.

## Set up the partitioning master table

Let's first create the master table with a `LIST` type of partitioning as we will store one different rating on each partition. Additionaly, we can use `RANGE` for partitions that are based on dates for example and `HASH` for partitions to be spread using field `MODULUS`.

```sql
drop table if exists reviews_part;

create table reviews_part(
  review jsonb
) partition by list(cast(review #>> '{review,rating}' as int));
```

## Creating the partitions

Our ratings are just values from `1` to `5`, let's create five partitions then:

```sql
create table reviews_part_1 partition of reviews_part for values in (1);
create table reviews_part_2 partition of reviews_part for values in (2);
create table reviews_part_3 partition of reviews_part for values in (3);
create table reviews_part_4 partition of reviews_part for values in (4);
create table reviews_part_5 partition of reviews_part for values in (5);
```

And the corresponding expression based indexes:

```sql
create index on reviews_part_1(cast(review #>> '{review,rating}' as int));
create index on reviews_part_2(cast(review #>> '{review,rating}' as int));
create index on reviews_part_3(cast(review #>> '{review,rating}' as int));
create index on reviews_part_4(cast(review #>> '{review,rating}' as int));
create index on reviews_part_5(cast(review #>> '{review,rating}' as int));
```

## Populate the partition table

Let's populate our partitioned table using a 20% of the `"commitconf-01".reviews` table for the sake of this exercise.

```sql
insert into reviews_part(review)
     select review
       from "commitconf-01".reviews
tablesample system(20)
```

## Check the partitions

We can run this query to check the size of the different partitions.

```sql
select * from (
  select count(*) as rows, '1' as part from reviews_part_1 union
  select count(*) as rows, '2' as part from reviews_part_2 union
  select count(*) as rows, '3' as part from reviews_part_3 union
  select count(*) as rows, '4' as part from reviews_part_4 union
  select count(*) as rows, '5' as part from reviews_part_5
) s
order by rows
```

Feel free to run different queries on `reviews_part` like grouping by the partitioning criteria

```sql
explain analyze
  select count(*) as counts,
         cast(review #>> '{review,rating}' as int) as rating
    from reviews_part
group by 2
```

A final note about partitioning with Postgres 11. There's been many improvements on the last release for this feature like:

* we can have a `DEFAULT` partition
* creating an index on the main table will automatically create them on the partitions.
* we can run `UPDATE` on partition rows and they will be moved if needed
* partitions by hash

More details [here](https://pgdash.io/blog/partition-postgres-11.html).
