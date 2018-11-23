# PL/Python

- Official [documentation](https://www.postgresql.org/docs/10/plpython-python23.html)

- Installation (depends on the Postgresql distribution)

```sh
$ sudo apt-get install postgresql-plpython-10.1
# must be superuser to create procedural language "plpythonu"
$ sudo -Hnu postgres psql -c "ALTER USER aromeu WITH SUPERUSER"
$ sudo -Hnu postgres psql -c "CREATE EXTENSION plpythonu"
# or for a certain DB:
$ sudo -Hnu postgres psql $dbname -c "CREATE EXTENSION plpythonu"
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

- [Subtransactions](https://www.postgresql.org/docs/9.1/plpython-subtransaction.html)

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

## Demo

Create a trigger to calculate the full address of a pair of coordinates.

- First we are creating a configuration table to store available geocoders metadata:

```sql
CREATE TABLE geocoders(name TEXT, url TEXT, reverse_url TEXT);
INSERT INTO geocoders
VALUES('nominatim',
       NULL,
       'https://nominatim.openstreetmap.org/reverse');
```

- This is the business table with coordinates and addresses:

```sql
CREATE TABLE locations(lon NUMERIC, lat NUMERIC, full_address TEXT);
```

- A request to the Nominatim URL with a couple of coordinates gives us the full address:

```text
https://nominatim.openstreetmap.org/reverse?format=jsonv2&lat=40.420108&lon=-3.705786&zoom=18
```

```json
{"place_id":"154055937","licence":"Data © OpenStreetMap contributors, ODbL 1.0. https:\/\/osm.org\/copyright","osm_type":"way","osm_id":"344130176","lat":"40.41985725","lon":"-3.70581199957834","place_rank":"30","category":"place","type":"square","importance":"0","addresstype":"place","name":"Plaza de Callao","display_name":"Plaza de Callao, Sol, Centro, Madrid, Área metropolitana de Madrid y Corredor del Henares, Community of Madrid, 28001, Spain","address":{"address29":"Plaza de Callao","pedestrian":"Plaza de Callao","suburb":"Sol","city_district":"Centro","city":"Madrid","county":"Área metropolitana de Madrid y Corredor del Henares","state":"Community of Madrid","postcode":"28001","country":"Spain","country_code":"es"},"boundingbox":["40.419483","40.4202199","-3.7061354","-3.705431"]}{
  "place_id": "154055937",
  "licence": "Data © OpenStreetMap contributors, ODbL 1.0. https://osm.org/copyright",
  "osm_type": "way",
  "osm_id": "344130176",
  "lat": "40.41985725",
  "lon": "-3.70581199957834",
  "place_rank": "30",
  "category": "place",
  "type": "square",
  "importance": "0",
  "addresstype": "place",
  "name": "Plaza de Callao",
  "display_name": "Plaza de Callao, Sol, Centro, Madrid, Área metropolitana de Madrid y Corredor del Henares, Community of Madrid, 28001, Spain",
  "address": {
    "address29": "Plaza de Callao",
    "pedestrian": "Plaza de Callao",
    "suburb": "Sol",
    "city_district": "Centro",
    "city": "Madrid",
    "county": "Área metropolitana de Madrid y Corredor del Henares",
    "state": "Community of Madrid",
    "postcode": "28001",
    "country": "Spain",
    "country_code": "es"
  },
  "boundingbox": [
    "40.419483",
    "40.4202199",
    "-3.7061354",
    "-3.705431"
  ]
}
```

- A Python function that requests to a geocoder REST API for a full address:

```sql
CREATE OR REPLACE FUNCTION reverse_geocode(geocoder TEXT, lon NUMERIC, lat NUMERIC)
    RETURNS TEXT
AS $$
    import requests

    # Let's use the shared cache to avoid preparing the geocoder metadata
    if geocoder in SD:
        plan = SD[geocoder]
    # A prepared statement from Python
    else:
        plan = plpy.prepare("SELECT reverse_url AS url FROM geocoders WHERE name = $1", ["text"])
        SD[geocoder] = plan

    # Execute the statement with the geocoder param and limit to 1 result
    rv = plpy.execute(plan, [geocoder], 1)
    url = rv[0]['url']

    # Make the request to the geocoder API
    payload = {"lon": lon, "lat": lat, "format": "jsonv2", "zoom": 18}
    r = requests.get(url, params=payload)

    # Return the full address or nothing if not found
    if r.status_code == 200:
      return r.json()["display_name"]
    else:
      return ''
$$ LANGUAGE plpythonu;
```

- Don't go crazy and use each language for what it's best. Let's use a regular SQL trigger procedure to automatically geocode new rows:

```sql
CREATE OR REPLACE FUNCTION fill_full_address() RETURNS trigger AS $$
    BEGIN
        -- NOTE we are calling the reverse_geocode Python function defined above
        NEW.full_address := reverse_geocode('nominatim', NEW.lon, NEW.lat);
        RETURN NEW;
    END;
$$ LANGUAGE plpgsql;
```

```sql
CREATE TRIGGER fill_full_address
BEFORE
INSERT
OR
UPDATE ON locations
FOR EACH ROW EXECUTE PROCEDURE fill_full_address();
```

- Let's INSERT a new location getting in the same execution the result of the trigger using [`RETURNING`](https://www.postgresql.org/docs/current/dml-returning.html):

```sql
INSERT INTO locations 
     VALUES (-3.705786, 40.420108) 
  RETURNING lon, lat, full_address;
```

```text
    lon    |    lat    |                                                         full_address
-----------+-----------+-------------------------------------------------------------------------------------------------------------------------------
 -3.705786 | 40.420108 | Plaza de Callao, Sol, Centro, Madrid, Área metropolitana de Madrid y Corredor del Henares, Comunidad de Madrid, 28001, España
 ```
