# Full text search

It's very common to perform full text search on your data. This can go quickly South if your requirements are to enable flexible queries getting into concepts like tokenization and lexemes. In this scenario you can think on going to NoSQL options like Elastic Search or Sphinx. You shouldn't!! Postgres full text search capabilities include:

* Special data type to store your tokenized text, with support for indexing
* Simple language to generate search queries
* Ranking
* Highlighting

Full documentation [here](https://www.postgresql.org/docs/current/textsearch.html).

## Some example queries

Let's follow some example queries from this [excellent blog post](https://www.compose.com/articles/mastering-postgresql-tools-full-text-search-and-phrase-search/#entertsquery) to learn how to create a tokenized field, build a query and check using the `@@` operator if they intersect:

```sql
SET default_text_search_config = 'pg_catalog.spanish';
SELECT to_tsvector(
   'En un lugar de la Mancha, de cuyo nombre no quiero acordarme, ' ||
   'no ha mucho tiempo que vivía un hidalgo de los de lanza en astilleros, ' ||
   'adarga antigua, rocín flaco y galgo corredor.'
) @@ to_tsquery('lugar');
```

As you can see we set a configuration value to tell Postgres we are using the Spanish language. This means that we can use `lugar` or `lugares` and they both work. Of course capitalization does not affect and `mancha` and `MaNcHa` they all intersect.

You can also run more detailed queries

* `hidalgo | caballero` for `or` type of queries
* `hidalgo & lugar` for `and` type of queries
* `!caballero` for negation
* `galgo <-> corredor` to search for combined words
* `lugar <3> mancha` even a defined number of words apart
* `!caballero & mucho & (rocín <-> flacos)` and combinations


## Query performance

To demonstrate the performance of this data structure, and following the guidelines described on this [article](https://blog.lateral.io/2015/05/full-text-search-in-milliseconds-with-postgresql/) we've set up a `streets` table with the centroids and names of the Spanish roads and streets coming from [OSM](https://osm.org) database. That's a three million records table with the following structure

```text
## Schema

+----------------------+-------------------------+
| attribute            | type                    |
+----------------------+-------------------------+
| cartodb_id           | bigint                  |
| the_geom             | geometry(Geometry,4326) |
| the_geom_webmercator | geometry(Geometry,3857) |
| fid                  | integer                 |
| osm_id               | text                    |
| name                 | text                    |
| fclass               | text                    |
| tsv                  | tsvector                |
+----------------------+-------------------------+

## Indexes

+----------------------------------+----------------------+------------+
| index_name                       | column_name          | index_type |
+----------------------------------+----------------------+------------+
| streets_name_idx                 | name                 | btree      |
| streets_pkey                     | cartodb_id           | btree      |
| streets_the_geom_idx             | the_geom             | gist       |
| streets_the_geom_webmercator_idx | the_geom_webmercator | gist       |
| streets_tsv_idx                  | tsv                  | gin        |
+----------------------------------+----------------------+------------+
```

As you can see there is a normal index on the `name` field in case you want to try searching on that field directly, but also an index on the `tsv` field so you can run queries like this one:


```sql
set default_text_search_config = 'pg_catalog.spanish';
select name, tsv
  from "commitconf-01".streets,
       plainto_tsquery('YOUR QUERY') as q1
 where (tsv @@ q1)
```

where `YOUR QUERY` can be for example:

* `lola <-> flores`
* `leyenda <-> zelda`
* ...
`

**Note**: you may get repeated rows because OSM data is per street section, not the full one.

If you want to get unique street names and ordered by the rank result you can run this variation:

```sql
set default_text_search_config = 'pg_catalog.spanish';
  select name,
         ts_rank_cd(s.tsv, plainto_tsquery('adolfo <-> suárez')) as rank
    from (
          select name, tsv
            from "commitconf-01".streets,
                 plainto_tsquery('adolfo <-> suárez') as q1
           where (tsv @@ q1)
         ) s
group by 1,2
order by 2 desc
```

Finally, if you want to get the results highlighted you may want to change the `name` by a call to `ts_headline`:

```sql
set default_text_search_config = 'pg_catalog.spanish';
  select ts_headline(name ,plainto_tsquery('adolfo <-> suárez')) as headline,
         ts_rank_cd(s.tsv, plainto_tsquery('adolfo <-> suárez')) as rank
    from (
          select name, tsv
            from "commitconf-01".streets,
                 plainto_tsquery('adolfo <-> suárez') as q1
           where (tsv @@ q1)
         ) s
group by 1,2
order by 2 desc
```

All these queries should run against that three million records table (1GB) in less than **ten milliseconds** on the testing database.
