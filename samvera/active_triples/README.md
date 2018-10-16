# ActiveTriples

## Examples for integrating Rails models into Samvera repositories using the ActiveTriples Gem

## Models

### GeoShapes

### Places
```ruby
class Place
  include  ActiveTriples::RDFSource

  configure type: ::RDF::URI.new('http://schema.org/Place'), base_uri: 'http://institution.edu/georepository/'
  # [...]
  property :geo, predicate: ::RDF::URI.new('http://schema.org/geo'), class_name: 'GeoShape'
end
```
```
> place = Place.new
> place.geo = shape
> puts place.dump :ntriples
_:g70334930722580 <schema:geo> _:g70334881126520 .
_:g70334930722580 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <schema:Place> .
_:g70334881126520 <geo:asWKT> "POLYGON ((30 10, 40 40, 20 40, 10 20, 30 10))"^^<geo:wktLiteral> .
_:g70334881126520 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <schema:GeoShape> .
```

### (Linked) Geospatial Works
```ruby
class GeoWork
  include  ActiveTriples::RDFSource

  configure type: [::RDF::URI.new('http://pcdm.org/works#Work'), ::RDF::URI.new('http://schema.org/Map')], base_uri: 'http://institution.edu/georepository/'

  property :spatial_coverage, predicate: ::RDF::URI.new('schema:spatialCoverage'), class_name: 'Place'
end
```
```

```
