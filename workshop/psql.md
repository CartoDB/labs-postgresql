# psql

Official [documentation](https://www.postgresql.org/docs/10/app-psql.html)

## psql short cheatsheet

| Command | Description |
| - | - |
| `\?` | Help |
| `\h statement` | Help on a specific SQL `statement` (e.g. `\h ALTER TABLE`) |
| `\l` | List databases |
| `\c databasename`| Switch to a `databasename` |
| `\dt` | List tables |
| `\d` table_name | Describe a table |
| `\dn` | List schemas |
| `\df` | List functions |
| `\dv` | List views |
| `\du` | List users |
| `\dx` | List extensions |
| `\g` | Execute the previous command |
| `\s` | Command history |
| `\s filename` | Save command history to `filename` |
| `\i` | Execute commands from a file |
| `\e` | Edit query on editor |
| `\ef function name` | Edit `function name` on editor |
| `\q` | Quit psql |

## Config .psqlrc

```bash
# system wide
pg_config --sysconfdir
```

```bash
# user specific
touch ~/.psqlrc
```

```bash
# versioning
touch ~/.psqlrc-9.1
touch ~/.psqlrc-9.3
```

### A .psqlrc example

```text
\set QUIET 1

-- %M refers to the database server's hostname -- is "[local]" if the connection is over a Unix domain socket
-- %> refers to the listening port
-- %n refers to the session username
-- %/ refers the current database
-- %R refers to whether you're in single-line mode (^) or disconnected (!) but is normally =
-- %# refers to whether you're a superuser (#) or a regular user (>)
-- %x refers to the transaction status -- usually blank unless in a transaction block (*)

\set PROMPT1 '%M:%> %n@%/%R%#%x '

-- PROMPT2 is used when unfinished queries (i.e. you miss the semicolon)
\set PROMPT2 '[more] %R > '

-- Colors
\set PROMPT1 '%M:%[%033[1;31m%]%>%[%033[0m%] %n@%/%R%#%x '

-- NULLs
\pset null '[null]'

-- Nicer tables
\pset linestyle unicode

-- To complete SQL keywords uppercase
\set COMP_KEYWORD_CASE upper

-- Display query times
\timing

-- Size of previously executed queries
\set HISTSIZE 200000
\set HISTCONTROL ignoredups
\set HISTFILE ~/.psql_history- :DBNAME

-- Expanded table format
\x auto

-- verbose, default, terse
\set VERBOSITY verbose

-- shortcuts
\set version 'SELECT version();'
\set extensions 'select * from pg_available_extensions;'
\set menu '\\i ~/.psqlrc'
\set settings 'select name, setting,unit,context from pg_settings;'
\set conninfo 'select usename, count(*) from pg_stat_activity group by usename;'
\set activity 'select datname, pid, usename, application_name,client_addr, client_hostname, client_port, query, state from pg_stat_activity;'
\set waits 'SELECT pg_stat_activity.pid, pg_stat_activity.query, pg_stat_activity.waiting, now() - pg_stat_activity.query_start AS \"totaltime\", pg_stat_activity.backend_start FROM pg_stat_activity WHERE pg_stat_activity.query !~ \'%IDLE%\'::text AND pg_stat_activity.waiting = true;'
\set dbsize 'SELECT datname, pg_size_pretty(pg_database_size(datname)) db_size FROM pg_database ORDER BY db_size;'
\set tablesize 'SELECT nspname || \'.\' || relname AS \"relation\", pg_size_pretty(pg_relation_size(C.oid)) AS "size" FROM pg_class C LEFT JOIN pg_namespace N ON (N.oid = C.relnamespace) WHERE nspname NOT IN (\'pg_catalog\', \'information_schema\') ORDER BY pg_relation_size(C.oid) DESC LIMIT 40;'
\set uselesscol 'SELECT nspname, relname, attname, typname, (stanullfrac*100)::int AS null_percent, case when stadistinct >= 0 then stadistinct else abs(stadistinct)*reltuples end AS \"distinct\", case 1 when stakind1 then stavalues1 when stakind2 then stavalues2 end AS \"values\" FROM pg_class c JOIN pg_namespace ns ON (ns.oid=relnamespace) JOIN pg_attribute ON (c.oid=attrelid) JOIN pg_type t ON (t.oid=atttypid) JOIN pg_statistic ON (c.oid=starelid AND staattnum=attnum) WHERE nspname NOT LIKE E\'pg\\\\_%\' AND nspname != \'information_schema\' AND relkind=\'r\' AND NOT attisdropped AND attstattarget != 0 AND reltuples >= 100 AND stadistinct BETWEEN 0 AND 1 ORDER BY nspname, relname, attname;'
\set uptime 'select now() - pg_postmaster_start_time() AS uptime;'

-- Taken from https://github.com/heroku/heroku-pg-extras
-- via https://github.com/dlamotte/dotfiles/blob/master/psqlrc
\set bloat 'SELECT tablename as table_name, ROUND(CASE WHEN otta=0 THEN 0.0 ELSE sml.relpages/otta::numeric END,1) AS table_bloat, CASE WHEN relpages < otta THEN ''0'' ELSE pg_size_pretty((bs*(sml.relpages-otta)::bigint)::bigint) END AS table_waste, iname as index_name, ROUND(CASE WHEN iotta=0 OR ipages=0 THEN 0.0 ELSE ipages/iotta::numeric END,1) AS index_bloat, CASE WHEN ipages < iotta THEN ''0'' ELSE pg_size_pretty((bs*(ipages-iotta))::bigint) END AS index_waste FROM ( SELECT schemaname, tablename, cc.reltuples, cc.relpages, bs, CEIL((cc.reltuples*((datahdr+ma- (CASE WHEN datahdr%ma=0 THEN ma ELSE datahdr%ma END))+nullhdr2+4))/(bs-20::float)) AS otta, COALESCE(c2.relname,''?'') AS iname, COALESCE(c2.reltuples,0) AS ituples, COALESCE(c2.relpages,0) AS ipages, COALESCE(CEIL((c2.reltuples*(datahdr-12))/(bs-20::float)),0) AS iotta FROM ( SELECT ma,bs,schemaname,tablename, (datawidth+(hdr+ma-(case when hdr%ma=0 THEN ma ELSE hdr%ma END)))::numeric AS datahdr, (maxfracsum*(nullhdr+ma-(case when nullhdr%ma=0 THEN ma ELSE nullhdr%ma END))) AS nullhdr2 FROM ( SELECT schemaname, tablename, hdr, ma, bs, SUM((1-null_frac)*avg_width) AS datawidth, MAX(null_frac) AS maxfracsum, hdr+( SELECT 1+count(*)/8 FROM pg_stats s2 WHERE null_frac<>0 AND s2.schemaname = s.schemaname AND s2.tablename = s.tablename) AS nullhdr FROM pg_stats s, ( SELECT (SELECT current_setting(''block_size'')::numeric) AS bs, CASE WHEN substring(v,12,3) IN (''8.0'',''8.1'',''8.2'') THEN 27 ELSE 23 END AS hdr, CASE WHEN v ~ ''mingw32'' THEN 8 ELSE 4 END AS ma FROM (SELECT version() AS v) AS foo) AS constants GROUP BY 1,2,3,4,5) AS foo) AS rs JOIN pg_class cc ON cc.relname = rs.tablename JOIN pg_namespace nn ON cc.relnamespace = nn.oid AND nn.nspname = rs.schemaname AND nn.nspname <> ''information_schema'' LEFT JOIN pg_index i ON indrelid = cc.oid LEFT JOIN pg_class c2 ON c2.oid = i.indexrelid) AS sml ORDER BY CASE WHEN relpages < otta THEN 0 ELSE bs*(sml.relpages-otta)::bigint END DESC;'
\set blocking 'select bl.pid as blocked_pid, ka.query as blocking_statement, now() - ka.query_start as blocking_duration, kl.pid as blocking_pid, a.query as blocked_statement, now() - a.query_start as blocked_duration from pg_catalog.pg_locks bl join pg_catalog.pg_stat_activity a on bl.pid = a.pid join pg_catalog.pg_locks kl join pg_catalog.pg_stat_activity ka on kl.pid = ka.pid on bl.transactionid = kl.transactionid and bl.pid != kl.pid where not bl.granted;'
\set cache_hit 'SELECT ''index hit rate'' as name, (sum(idx_blks_hit)) / sum(idx_blks_hit + idx_blks_read) as ratio FROM pg_statio_user_indexes union all SELECT ''cache hit rate'' as name, sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) as ratio FROM pg_statio_user_tables;'
\set index_size 'SELECT relname AS name, pg_size_pretty(sum(relpages*1024)) AS size FROM pg_class WHERE reltype=0 GROUP BY relname ORDER BY sum(relpages) DESC;'
\set index_usage 'SELECT relname, CASE idx_scan WHEN 0 THEN ''Insufficient data'' ELSE (100 * idx_scan / (seq_scan + idx_scan))::text END percent_of_times_index_used, n_live_tup rows_in_table FROM pg_stat_user_tables ORDER BY n_live_tup DESC;'
\set index_usage_adv 'SELECT * FROM (SELECT stat.relname AS table, stai.indexrelname AS index, CASE stai.idx_scan WHEN 0 THEN ''Insufficient data'' ELSE (100 * stai.idx_scan / (stat.seq_scan + stai.idx_scan))::text || ''%'' END hit_rate, CASE stat.idx_scan WHEN 0 THEN ''Insufficient data'' ELSE (100 * stat.idx_scan / (stat.seq_scan + stat.idx_scan))::text || ''%'' END all_index_hit_rate, ARRAY(SELECT pg_get_indexdef(idx.indexrelid, k + 1, true) FROM generate_subscripts(idx.indkey, 1) AS k ORDER BY k) AS cols, stat.n_live_tup rows_in_table FROM pg_stat_user_indexes AS stai JOIN pg_stat_user_tables AS stat ON stai.relid = stat.relid JOIN pg_index AS idx ON (idx.indexrelid = stai.indexrelid)) AS sub_inner ORDER BY rows_in_table DESC, hit_rate ASC;'
\set locks 'select pg_stat_activity.pid, pg_class.relname, pg_locks.transactionid, pg_locks.granted, substr(pg_stat_activity.query,1,30) as query_snippet, age(now(),pg_stat_activity.query_start) as "age" from pg_stat_activity,pg_locks left outer join pg_class on (pg_locks.relation = pg_class.oid) where pg_stat_activity.query <> ''<insufficient privilege>'' and pg_locks.pid=pg_stat_activity.pid and pg_locks.mode = ''ExclusiveLock'' order by query_start;'
\set long_running_queries 'SELECT pid, now() - pg_stat_activity.query_start AS duration, query AS query FROM pg_stat_activity WHERE pg_stat_activity.query <> ''''::text AND now() - pg_stat_activity.query_start > interval ''5 minutes'' ORDER BY now() - pg_stat_activity.query_start DESC;'
\set ps 'select pid, application_name as source, age(now(),query_start) as running_for, waiting, query as query from pg_stat_activity where query <> ''<insufficient privilege>'' AND state <> ''idle'' and pid <> pg_backend_pid() order by 3 desc;'
\set seq_scans 'SELECT relname AS name, seq_scan as count FROM pg_stat_user_tables ORDER BY seq_scan DESC;'
\set total_index_size 'SELECT pg_size_pretty(sum(relpages*1024)) AS size FROM pg_class WHERE reltype=0;'
\set unused_indexes 'SELECT schemaname || ''.'' || relname AS table, indexrelname AS index, pg_size_pretty(pg_relation_size(i.indexrelid)) AS index_size, idx_scan as index_scans FROM pg_stat_user_indexes ui JOIN pg_index i ON ui.indexrelid = i.indexrelid WHERE NOT indisunique AND idx_scan < 50 AND pg_relation_size(relid) > 5 * 8192 ORDER BY pg_relation_size(i.indexrelid) / nullif(idx_scan, 0) DESC NULLS FIRST, pg_relation_size(i.indexrelid) DESC;'
\set missing_indexes 'SELECT relname, seq_scan-idx_scan AS too_much_seq, case when seq_scan-idx_scan > 0 THEN ''Missing Index?'' ELSE ''OK'' END, pg_relation_size(relname::regclass) AS rel_size, seq_scan, idx_scan FROM pg_stat_all_tables WHERE schemaname=''public'' AND pg_relation_size(relname::regclass) > 80000 ORDER BY too_much_seq DESC;'

-- Development queries

\set sp 'SHOW search_path;'
\set clear '\\! clear;'
\set ll '\\! ls -lrt;'

-- Welcome message
\echo 'Welcome to PostgreSQL\n'
\echo 'Administrative queries:'
\echo
\echo '  :version               PostgreSQL version.'
\echo '  :extensions            Available extensions.'
\echo '  :activity              Server activity.'
\echo '  :bloat                 Show table and index bloat in your database ordered'
\echo '                         by most wasteful.'
\echo '  :blocking              Display queries holding locks other queries are'
\echo '                         waiting to be released.'
\echo '  :cache_hit             Calculates your cache hit rate (effective databases'
\echo '                         are at 99% and up).'
\echo '  :conninfo              Server connections.'
\echo '  :dbsize                Database Size.'
\echo '  :index_size            Show the size of the indexes, descending by size.'
\echo '  :index_usage           Calculates your index hit rate per table (effective'
\echo '                         databases are at 99% and up).'
\echo '  :index_usage_adv       Calculates your index hit rate per index (effective'
\echo '                         databases are at 99% and up).'
\echo '  :locks                 Display queries with active locks.'
\echo '  :long_running_queries  Show queries taking longer than five minutes.'
\echo '  :ps                    View active queries with execution time.'
\echo '  :seq_scans             Show the count of seq_scans by table.'
\echo '  :settings              Server settings.'
\echo '  :tablesize             Tables size.'
\echo '  :total_index_size      Show the total size of the indexes in MB.'
\echo '  :unused_indexes        Show unused and almost unused indexes, ordered by'
\echo '                         their size relative to the number of index scans.'
\echo '                         Excludes indexes of very small tables (< 5 pages).'
\echo '  :missing_indexes       Show big tables that have too many seq_scans.'
\echo '  :uptime                Server uptime.'
\echo '  :uselesscol            Useless columns (must be run as superuser).'
\echo '  :waits                 Waiting queries.'
\echo
\echo 'Development queries:'
\echo
\echo '  :sp     Current search path.'
\echo '  :clear  Clear screen.'
\echo '  :ll     List.'
\echo
\echo
\echo 'Type \\q to exit. \n'

\unset QUIET
```

## Open a session

```bash
# as postgres
psql -U postgres
```

```bash
# as a user to a database
psql -d database -U  user -W
```

```bash
# to a host
psql -h host -d database -U user -W
```

```bash
# force ssl
psql -U user -h host "dbname=db sslmode=require"
```

**Password file**:

To avoid introducing passwords

```bash
# in user's home directory
echo "hostname:port:database:username:password" > ~/.pgpass
chmod 0600 ~/.pgpass
```

```bash
# via PGPASSFILE ENV variable
PGPASSFILE=/etc/.pgpass psql -h host -d database -U user
```

**Running commands from shell**:

```bash
psql -U user -d database -c '\d'
psql -U user -d database -c 'SELECT version();'
```