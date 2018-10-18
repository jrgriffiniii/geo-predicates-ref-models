# ActiveTriples
_Examples for integrating Rails data models into applications using the [ActiveTriples](https://github.com/ActiveTriples/ActiveTriples) Gem.  ActiveTriples persists resources into any number of RDF graphs stores using [RDF::Repository](https://www.rubydoc.info/github/ruby-rdf/rdf/RDF/Repository).  Please see [the documentation](https://github.com/ActiveTriples/ActiveTriples#repositories-and-persistence) for further information._

## Models

### GeoShapes
```ruby
class GeoShape
  include ActiveTriples::RDFSource

  configure type: ::RDF::URI.new('http://schema.org/GeoShape'), base_uri: 'http://institution.edu/georepository/'
  # [...]
  property :geo, predicate: ::RDF::URI.new('http://www.opengis.net/ont/geosparql#asWKT')
end
```
```
> shape = GeoShape.new
> shape.geo = RDF::Literal.new("POLYGON ((10 10, 40 10, 40 40, 10 40, 10 10))", datatype: "http://www.opengis.net/ont/geosparql#wktLiteral")
> puts shape.dump :ntriples
_:g70334896391740 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://schema.org/GeoShape> .
_:g70334896391740 <http://www.opengis.net/ont/geosparql#asWKT> "POLYGON ((10 10, 40 10, 40 40, 10 40, 10 10))"^^<http://www.opengis.net/ont/geosparql#wktLiteral> .
```

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
_:g70334881935420 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://schema.org/Place> .
_:g70334881935420 <http://schema.org/geo> _:g70334896391740 .
_:g70334896391740 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://schema.org/GeoShape> .
_:g70334896391740 <http://www.opengis.net/ont/geosparql#asWKT> "POLYGON ((10 10, 40 10, 40 40, 10 40, 10 10))"^^<http://www.opengis.net/ont/geosparql#wktLiteral> .
```

### (Linked) Geospatial Work Resources
```ruby
class ScannedMap
  include ActiveTriples::RDFSource

  configure type: [::RDF::URI.new('http://pcdm.org/works#Work'), ::RDF::URI.new('http://schema.org/Map')], base_uri: 'http://institution.edu/georepository/'

  property :spatial_coverage, predicate: ::RDF::URI.new('schema:spatialCoverage'), class_name: 'Place'
end
```
```
> linked_geo_work = ScannedMap.new
> linked_geo_work.spatial_coverage = place
> puts linked_geo_work.dump :ntriples
_:g70334900080040 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://pcdm.org/works#Work> .
_:g70334900080040 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://schema.org/Map> .
_:g70334900080040 <http://schema.org/spatialCoverage> _:g70334881935420 .
_:g70334881935420 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://schema.org/Place> .
_:g70334881935420 <http://schema.org/geo> _:g70334896391740 .
_:g70334896391740 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://schema.org/GeoShape> .
_:g70334896391740 <http://www.opengis.net/ont/geosparql#asWKT> "POLYGON ((10 10, 40 10, 40 40, 10 40, 10 10))"^^<http://www.opengis.net/ont/geosparql#wktLiteral> .
```
