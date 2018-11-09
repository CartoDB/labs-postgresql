## PL/Python

- Official [documentation](https://www.postgresql.org/docs/10/plpython-python23.html)

- Installation (depends on the Postgresql distribution)
```
sudo apt-get install postgresql-plpython-10.1
# must be superuser to create procedural language "plpythonu"
sudo -Hnu postgres psql -c "ALTER USER aromeu WITH SUPERUSER"
sudo -Hnu postgres psql -c "CREATE EXTENSION plpythonu"
# or for a certain DB:
sudo -Hnu postgres psql $dbname -c "CREATE EXTENSION plpythonu"
```

- Or you can trust the language by default:
```sql
SELECT * FROM pg_language;
 lanname  | lanowner | lanispl | lanpltrusted | lanplcallfoid | laninline | lanvalidator | lanacl
-----------+----------+---------+--------------+---------------+-----------+--------------+--------
 internal  |       10 | f       | f            |             0 |         0 |         2246 |
 c         |       10 | f       | f            |             0 |         0 |         2247 |
 sql       |       10 | f       | t            |             0 |         0 |         2248 |
 plpgsql   |       10 | t       | t            |         12967 |     12968 |        12969 |
 plproxy   |       10 | t       | f            |         22869 |         0 |        22870 |
 plpythonu |       10 | t       | f            |         18040 |     18041 |        18042 |

UPDATE pg_language SET lanpltrusted = true WHERE lanname LIKE 'plpythonu';
```

- Simplest example:

```sql
CREATE FUNCTION pymax (a integer, b integer)
  RETURNS integer
AS $$
  if a > b:
    return a
  return b
$$ LANGUAGE plpythonu;
```

- Data type mapping
  - NULL <-> None
  - ARRAY <-> list
  - TYPE -> dict
  - dict|Object -> TYPE

- Sharing data
  - SD (shared data per function) vs GD (global data between functions)

- Database access
  - execute
  - prepare
  - cursors

```sql
CREATE FUNCTION vvv ()
  RETURNS TEXT
AS $$
  result = plpy.execute("SELECT version();")
  return result[0]['version']
$$ LANGUAGE plpythonu;
```

- Subtransactions

- Modules

_The cleaner solution would be to set PYTHONPATH for the postgresql server process._


```sh
mkdir -p /tmp/foo
cat <<EOF > /tmp/foo/foo.py
def f():
    return 'hello from foo'
EOF
```

```sql
CREATE FUNCTION foo_module()
    RETURNS text
AS $$
    import sys
    sys.path.insert(0, '/tmp/foo')
    import foo
    return foo.f()
$$ LANGUAGE plpythonu;

select foo_module();
```