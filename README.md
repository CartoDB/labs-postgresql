# CommitConf 2018 PostgreSQL Workshop

<p style="font-size:1.5em">
<a href="http://bit.ly/commitconf18-postgres">bit.ly/commitconf18-postgres</a>
</p>

![commit banner](./banner.png)

## Foreword

This repository contains a number of exercises about some not so well-knonw PostgreSQL features. It may be interesting for application developers and database administrators.

### Authors

* [Alberto Romeu](https://github.com/alrocar) · Full Stack developer at CARTO
* [Jorge Sanz](https://github.com/jsanz) · Solutions engineer at CARTO


### About CARTO

![](https://github.com/CartoDB/labs-postgresql/raw/master/workshop/imgs/logo_CARTO_positive_90.png)

Founded in 2012 by a team of experts in geospatial development, big data analytics, and visualization techniques, [CARTO](https://carto.com) is based in New York City and Spain, with additional locations in Washington DC, and Estonia. CARTO has a team of 100 employees, a portfolio of 1,200 customers including BBVA, BCG, NYC, and Twitter and more than 200,000 users over the globe. The company is backed by investors such as Accel and Salesforce Ventures.

[CARTO](https://carto.com) leads the world of location intelligence, empowering any organization and individual to discover and predict key insights through location data. With CARTO’s intuitive location intelligence platform, analysts and developers build self-service location based apps that help optimize operational performance, strategic investments, and everyday decisions.

### We are hiring!!

We are looking for a **UX Designer** for our **Madrid** office, more details [here](https://carto.workable.com/j/3103A90892).

## Contents

* Workshop set up
* [psql](https://github.com/CartoDB/labs-postgresql/blob/master/workshop/psql.md) tricks demo
* [Visual EXPLAINs](https://github.com/CartoDB/labs-postgresql/blob/master/workshop/pev.md) exercise
* [PostGIS](https://github.com/CartoDB/labs-postgresql/blob/master/workshop/postgis.md) exercise
* [Expression Based Indexes](https://github.com/CartoDB/labs-postgresql/blob/master/workshop/indexes-on-expressions.md) exercise
* [Full Text Search](https://github.com/CartoDB/labs-postgresql/blob/master/workshop/full-text-search.md) exercise
* [PL/Python - Python Procedural Language](https://github.com/CartoDB/labs-postgresql/blob/master/workshop/plpython.md) demo
* [Non Relational Data](https://github.com/CartoDB/labs-postgresql/blob/master/workshop/non-relational.md) exercise
* [Declarative Table Partitioning](https://github.com/CartoDB/labs-postgresql/blob/master/workshop/partitioning.md) exercise
* [TOAST](https://github.com/CartoDB/labs-postgresql/blob/master/workshop/toast.md) exercise

## Further reading

When we brainstormed this workshop we came accross other topics that we decided to not include, this is the list of topics and some further reading resources we reserve for future editions of the workshop where they may be included.

* [Foreign Data Wrappers](https://www.postgresql.org/docs/current/postgres-fdw.html)
* [`LISTEN`](https://www.postgresql.org/docs/current/sql-listen.html) & [`NOTIFY`](https://www.postgresql.org/docs/current/sql-notify.html)
* Using [`RETURNING`](https://www.postgresql.org/docs/current/dml-returning.html) to get data from modified rows
* [Window functions](https://www.postgresql.org/docs/current/tutorial-window.html)
* Continuous archiving of Postgres operations with [wal-e](https://github.com/wal-e/wal-e)
* Table [inheritance](https://www.postgresql.org/docs/current/ddl-inherit.html)
* A convenient connection pooling system with [pgBouncer](https://pgbouncer.github.io/features.html)

Apart from those topics and the infinite number of resources you'll find out there we recommend also these resources:

* [CARTO Engineering blog](https://carto.com/blog/inside)
* [Paul Ramsey's blog](http://blog.cleverelephant.ca/), mostly about PostGIS
* Abel's excellent [PostGIS/PostgreSQL tips & tricks](https://abelvm.github.io/sql/sql-tricks/)
* [Is PostgreSQL good enough?](http://renesd.blogspot.com/2017/02/is-postgresql-good-enough.html?m=1), great write up on many of the nice features we covered
* Commit 2016 talk: [the ten most powerful queries](https://www.youtube.com/watch?v=ZLvT8lQit80) (Spanish)
* [Modern SQL](https://modern-sql.com/), you probably already knew this one :smile:

## Data

You can get access to the datasets used on this workshop on this [shared Google Drive folder](https://drive.google.com/drive/folders/1xx8jCt_JgYq5g1WDDvrr6nmHPqo9CWXb?usp=sharing). They are mostly stored in a format called `Geopackage`, which is a customized SQLite database you can manage with GIS software.

* [Brooklyn MapPLUTO building footprints](https://www1.nyc.gov/site/planning/data-maps/open-data/dwn-pluto-mappluto.page) by the NYC Planning department
* Natural Earth [countries](https://www.naturalearthdata.com/downloads/10m-cultural-vectors/10m-admin-0-countries/) and [populated places](https://www.naturalearthdata.com/downloads/10m-cultural-vectors/10m-populated-places/) by Nathaniel Vaughn Kelso et. al.
* The streets dataset is derived from the Spanish [OpenStreetMap](https://osm.org) dump by geofabrik available [here](http://download.geofabrik.de/europe/spain.html)
