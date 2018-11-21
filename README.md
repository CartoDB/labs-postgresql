# PostgreSQL Workshop

## Foreword

This repository contains a number of exercises about some not so well-knonw PostgreSQL features. It may be interesting for application developers and database administrators.

Authors:

* [Alberto Romeu](https://github.com/alrocar) · Full Stack developer at CARTO
* [Jorge Sanz](https://github.com/jsanz) · Solutions engineer at CARTO

## Contents

* Workshop set up
* [plsql](./workshop/psql.md) tricks demo
* [Visual EXPLAINs](./workshop/pev.md) exercise
* [PostGIS](./workshop/postgis.md) exercise
* [Expression Based Indexes](./workshop/indexes-on-expressions.md) exercise
* [Full Text Search](./workshop/full-text-search.md) exercise
* [PL/Python - Python Procedural Language](./workshop/plpython.md) demo
* [Non Relational Data](./workshop/non-relational.md) exercise
* [Declarative Table Partitioning](./workshop/partitioning.md) exercise
* [TOAST](./workshop/toast.md) exercise

## Further reading

When we brainstormed this workshop we came accross other topics that we decided to not include, this is the list of topics and some further reading resources we reserve for future editions of the workshop where they may be included.

* [Foreign Data Wrappers](https://www.postgresql.org/docs/current/postgres-fdw.html)
* [`LISTEN`](https://www.postgresql.org/docs/current/sql-listen.html) & [`NOTIFY`](https://www.postgresql.org/docs/current/sql-notify.html) 
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

## Editions

This workshop has been delivered at:

* 2018-11-24 · [CommitConf 2018](https://2018.commit-conf.com/) · [Workshop page](https://github.com/CartoDB/labs-postgresql/tree/commitconf18)