# GraphDB
## Running these examples using the GraphDB graph database
_Note: For the purposes of this demonstration, this outlines the usage of the free distribution of GraphDB_

## Installing GraphDB
* Download the binary from https://ontotext.com/free-graphdb-download/
* Running the server, one should be able to access the administrative interface using http://localhost:7200/

## Creating an RDF repository
* Create a new repository using http://localhost:7200/repository/create
* (For the purposes of this demonstration, we are using the repository ID of "test1")
* Ensure that this repository is connected (please see http://graphdb.ontotext.com/documentation/free/quick-start-guide.html#create-a-repository)

## Upload the RDF files to the repository
* Upload RDF files using http://localhost:7200/import#user
* Upload the file "scanned_map.ttl" from the "data/" directory in the Git repository
* Select the "Import" button
* Import the RDF into the default graph

## Query the data using GeoSPARQL
* Craft the following SPARQL query into the interface at http://localhost:7200/sparql
```
PREFIX ogc: <http://www.opengis.net/ont/geosparql#>
PREFIX ogcf: <http://www.opengis.net/def/function/geosparql/>
PREFIX schema: <http://schema.org/>

SELECT ?resource ?polygon
WHERE
{
  ?resource schema:spatialCoverage ?fPlace .
  ?fPlace schema:geo ?fGeom .
  ?fGeom ogc:asWKT ?polygon .
  FILTER (ogcf:sfWithin(?polygon,"POLYGON ((0 0, 50 0, 50 50, 0 50, 0 0))"^^ogc:wktLiteral))
}
```
*Please freely try the other GeoSPARQL queries in the [queries directory](https://github.com/jrgriffiniii/geo-predicates-ref-models/tree/master/graphdb/queries) directory*
