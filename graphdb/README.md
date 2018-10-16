# GraphDB
## Running these examples using the GraphDB graph database
_Note: For the purposes of this demonstration, this outlines the usage of the free distribution of GraphDB_

## Installing GraphDB
### Mac OS Environments

## Querying with GeoSPARQL
### Over the HTTP using the cURL
```
curl -G --data-urlencode query@[PATH_TO_GIT_REPO]/queries/geo_sparql_scanned_map.rq -H "Accept: application/sparql-results+xml" http://localhost:7200/repositories/[GRAPH_REPO_ID]
```
