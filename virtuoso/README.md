# Virtuoso
## Running these examples using the Virtuoso graph database

## Installing Virtuoso
### Mac OS Environments

#### With Homebrew
```
$ brew install virtuoso
...
$ cd /usr/local/Cellar/virtuoso/[VERSION]
$ bin/virtuoso-t +foreground +configfile var/lib/virtuoso/db/virtuoso.ini
```

Now one can access Virtuoso on http://localhost:8890/

## Loading the procedures
```
$ bin/isql 1111
OpenLink Virtuoso Interactive SQL (Virtuoso)
Version 07.20.3229 as of Aug 19 2018
Type HELP; for help and EXIT; to exit.
Connected to OpenLink Virtuoso
Driver: 07.20.3229 OpenLink Virtuoso ODBC Driver
SQL> LOAD [PATH_TO_GIT_REPO]/virtuoso/procedures/rdfloader.sql;
Done. -- 1 msec.

Done. -- 0 msec.

[...]

Done. -- 0 msec.
SQL>
```

## Bulk-loading data
```
$ mkdir tmp
$ cp [PATH_TO_GIT_REPO]/data/scanned_map.ttl tmp/scanned_map.ttl
$ cat >> tmp/scanned_map.ttl.graph

http://institution.edu/georepository#
$ bin/isql 1111
SQL> ld_dir ('tmp', 'scanned_map.ttl', 'http://institution.edu/georepository/');
SQL> rdf_loader_run ();
```

### Verifying the loaded graph
The following should generate the URIs for the triples which have been persisted into the graph store:
```
SQL> SPARQL
SELECT *
FROM <http://institution.edu/georepository/>
WHERE
  {
    ?s ?p ?o
  };
```
...yielding:
```
LONG VARCHAR                                                                      LONG VARCHAR                                                                      LONG VARCHAR
_______________________________________________________________________________

http://maps.org/gazetteer/boundingBox1                                            http://www.w3.org/1999/02/22-rdf-syntax-ns#type                                   http://schema.org/GeoShape
[...]
http://schema.org/spatialCoverage                                                 http://maps.org/gazetteer/place1
```

## Loading data using SPARQL UPDATE
Please see http://vos.openlinksw.com/owiki/wiki/VOS/VirtGraphProtocolCURLExamples

## Querying using GeoSPARQL:

*Limits*
Virtuoso does not currently support GeoSPARQL.  It instead supports the following custom functions within a namespace prefixed with `bif`:

(Please see http://vos.openlinksw.com/owiki/wiki/VOS/VirtGeoSPARQLEnhancementDocs)

Further, many of these functions are not mature enough to support more complex spatial structures:
```
PREFIX geo: <http://www.opengis.net/ont/geosparql#>
PREFIX schema: <http://schema.org/>
SELECT ?resource ?polygon
FROM <http://institution.edu/georepository/>
WHERE
{
  ?resource schema:spatialCoverage ?fPlace .
  ?fPlace schema:geo ?fGeom .
  ?fGeom geo:asWKT ?polygon .
  FILTER (
    bif:st_contains (
	     bif:st_geomfromtext ( "POLYGON ((0 0, 50 0, 50 50, 0 50, 0 0))" ),
       ?polygon
    )
  ) .
}
```
...will trigger the following errors:
```
*** Error 42000: [Virtuoso Driver][Virtuoso Server]GEO..: for geo contains, only "shape contains point" case is supported in current version
```

### Inserting data
#### Scanned Maps
```
curl --digest --user dba:dba --verbose --url "http://localhost:8890/sparql-graph-crud-auth?graph-uri=http://institution.edu/georepository/" -T [PATH_TO_GIT_REPO]/data/scanned_map.ttl
```

#### Raster Data Sets
```
curl --digest --user dba:dba --verbose --url "http://localhost:8890/sparql-graph-crud-auth?graph-uri=http://institution.edu/georepository/" -T [PATH_TO_GIT_REPO]/data/raster_data_set.ttl
```

#### Vector Data Sets
```
curl --digest --user dba:dba --verbose --url "http://localhost:8890/sparql-graph-crud-auth?graph-uri=http://institution.edu/georepository/" -T [PATH_TO_GIT_REPO]/data/vector_data_set.ttl
```

### Querying for data over the HTTP using the cURL:
#### Scanned Maps
```
curl -G  --data-urlencode "default-graph-uri=http://institution.edu/georepository/scanned_map.ttl"  --data-urlencode "query@[PATH_TO_GIT_REPO]/virtuoso/queries/virtuoso_spatial_scanned_map.rq" "http://localhost:8890/sparql"
```

#### Raster Data Sets
```
curl -G  --data-urlencode "default-graph-uri=http://institution.edu/georepository/raster_data_set.ttl" --data-urlencode "query@[PATH_TO_GIT_REPO]/virtuoso/queries/virtuoso_spatial_raster_data_set.rq" http://localhost:8890/sparql
```

#### Vector Data Sets
```
curl -G  --data-urlencode "default-graph-uri=http://institution.edu/georepository/vector_data_set.ttl" --data-urlencode "query@[PATH_TO_GIT_REPO]/virtuoso/queries/virtuoso_spatial_vector_data_set.rq" http://localhost:8890/sparql
```

### Deleting data
```
curl --digest --user dba:dba --verbose --url "http://localhost:8890/sparql-graph-crud-auth?graph-uri=http://institution.edu/georepository/scanned_map.ttl" -X DELETE
curl --digest --user dba:dba --verbose --url "http://localhost:8890/sparql-graph-crud-auth?graph-uri=http://institution.edu/georepository/raster_data_set.ttl" -X DELETE
curl --digest --user dba:dba --verbose --url "http://localhost:8890/sparql-graph-crud-auth?graph-uri=http://institution.edu/georepository/vector_data_set.ttl" -X DELETE
```
