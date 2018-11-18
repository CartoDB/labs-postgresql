## psql

Official [documentation](https://www.postgresql.org/docs/10/app-psql.html)

### psql short cheatsheet

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

### Config .psqlrc

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

**A .psqlrc example**

```
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

-- Border around results
\pset border 2

-- To complete SQL keywords uppercase
\set COMP_KEYWORD_CASE upper

-- Display query times
\timing

-- Size of previously executed queries
\set HISTSIZE 2000
\set HISTCONTROL ignoredups
\set HISTFILE ~/.psql_history- :DBNAME

-- Expanded table format
\x auto

-- verbose, default, terse
\set VERBOSITY verbose

-- shortcuts
\set version 'SELECT version();'
\set extensions 'select * from pg_available_extensions;'

-- Welcome message
\echo 'Welcome to PostgreSQL\n'
\echo 'Type :version to see the PostgreSQL version. \n'
\echo 'Type :extensions to see the available extensions. \n'
\echo 'Type \\q to exit. \n'

\unset QUIET
```

### Open a session

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
