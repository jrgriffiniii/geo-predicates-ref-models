# ActiveFedora
_Examples for integrating Rails data models into Samvera repositories using the [ActiveFedora](https://github.com/samvera/active_fedora) Gem.  ActiveFedora persists resources into a [Fedora repository](https://duraspace.org/fedora/) implementation (also using [Apache Solr](https://lucene.apache.org/solr/) for discovery)._

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
_:g70334915130300 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://schema.org/GeoShape> .
_:g70334915130300 <http://www.opengis.net/ont/geosparql#asWKT> "POLYGON ((10 10, 40 10, 40 40, 10 40, 10 10))"^^<http://www.opengis.net/ont/geosparql#wktLiteral> .
```

### Places
```ruby
class Place
  include ActiveTriples::RDFSource

  configure type: ::RDF::URI.new('http://schema.org/Place'), base_uri: 'http://institution.edu/georepository/'
  # [...]
  property :geo, predicate: ::RDF::URI.new('http://schema.org/geo'), class_name: 'GeoShape'
end
```
```
> place = Place.new
> place.geo = shape
> puts place.dump :ntriples
_:g70334915130300 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://schema.org/GeoShape> .
_:g70334915130300 <http://www.opengis.net/ont/geosparql#asWKT> "POLYGON ((10 10, 40 10, 40 40, 10 40, 10 10))"^^<http://www.opengis.net/ont/geosparql#wktLiteral> .
_:g70334913418520 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://schema.org/Place> .
_:g70334913418520 <http://schema.org/geo> _:g70334915130300 .
```

### (Linked) Geospatial Works
```ruby
class ScannedMapWork < ActiveFedora::Base
  include ::Hyrax::WorkBehavior

  self.indexer = ScannedMapWork
  # Change this to restrict which works can be added as a child.
  # self.valid_child_concerns = []
  validates :title, presence: { message: 'Your work must have a title.' }

  type ::RDF::URI.new('http://schema.org/Map')
  property :spatial_coverage, predicate: ::RDF::URI.new('http://schema.org/spatialCoverage'), class_name: 'Place'
  # [...]
  # This must be included at the end, because it finalizes the metadata
  # schema (by adding accepts_nested_attributes)
  include ::Hyrax::BasicMetadata
end
```
```
> linked_geo_work = ScannedMapWork.new
> linked_geo_work.title = ['My linked geo. work']
> linked_geo_work.spatial_coverage = place
> linked_geo_work.save
> persisted = linked_geo_work.reload
> persisted.resource.dump :ttl
<http://127.0.0.1:8984/rest/dev/wd/37/5w/28/wd375w28x> <http://fedora.info/definitions/v4/repository#hasParent> <http://127.0.0.1:8984/rest/dev> .
<http://127.0.0.1:8984/rest/dev/wd/37/5w/28/wd375w28x> <http://fedora.info/definitions/v4/repository#lastModified> "2018-10-17T00:12:23.082Z"^^<http://www.w3.org/2001/XMLSchema#dateTime> .
<http://127.0.0.1:8984/rest/dev/wd/37/5w/28/wd375w28x> <http://fedora.info/definitions/v4/repository#lastModifiedBy> "bypassAdmin" .
<http://127.0.0.1:8984/rest/dev/wd/37/5w/28/wd375w28x> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://fedora.info/definitions/v4/repository#Container> .
<http://127.0.0.1:8984/rest/dev/wd/37/5w/28/wd375w28x> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://schema.org/Map> .
<http://127.0.0.1:8984/rest/dev/wd/37/5w/28/wd375w28x> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/ns/ldp#RDFSource> .
<http://127.0.0.1:8984/rest/dev/wd/37/5w/28/wd375w28x> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://fedora.info/definitions/v4/repository#Resource> .
<http://127.0.0.1:8984/rest/dev/wd/37/5w/28/wd375w28x> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/ns/ldp#Container> .
<http://127.0.0.1:8984/rest/dev/wd/37/5w/28/wd375w28x> <http://schema.org/spatialCoverage> <http://127.0.0.1:8984/rest/.well-known/genid/b8/74/21/72/b8742172-e68f-4ae1-879e-ae865819dad9> .
<http://127.0.0.1:8984/rest/dev/wd/37/5w/28/wd375w28x> <info:fedora/fedora-system:def/model#hasModel> "ScannedMapWork" .
<http://127.0.0.1:8984/rest/dev/wd/37/5w/28/wd375w28x> <http://purl.org/dc/terms/title> "My linked geo. work" .
<http://127.0.0.1:8984/rest/dev/wd/37/5w/28/wd375w28x> <http://fedora.info/definitions/v4/repository#writable> "true"^^<http://www.w3.org/2001/XMLSchema#boolean> .
<http://127.0.0.1:8984/rest/dev/wd/37/5w/28/wd375w28x> <http://fedora.info/definitions/v4/repository#createdBy> "bypassAdmin" .
<http://127.0.0.1:8984/rest/dev/wd/37/5w/28/wd375w28x> <http://fedora.info/definitions/v4/repository#created> "2018-10-17T00:12:23.082Z"^^<http://www.w3.org/2001/XMLSchema#dateTime> .
<http://127.0.0.1:8984/rest/dev/wd/37/5w/28/wd375w28x> <http://www.w3.org/ns/auth/acl#accessControl> <http://127.0.0.1:8984/rest/dev/96/77/d6/44/9677d644-eeac-42e1-b98e-857ce28f818a> .
<http://127.0.0.1:8984/rest/.well-known/genid/b8/74/21/72/b8742172-e68f-4ae1-879e-ae865819dad9> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://fedora.info/definitions/v4/repository#Skolem> .
<http://127.0.0.1:8984/rest/.well-known/genid/b8/74/21/72/b8742172-e68f-4ae1-879e-ae865819dad9> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://schema.org/Place> .
<http://127.0.0.1:8984/rest/.well-known/genid/b8/74/21/72/b8742172-e68f-4ae1-879e-ae865819dad9> <http://schema.org/geo> <http://127.0.0.1:8984/rest/.well-known/genid/3c/99/b4/e2/3c99b4e2-5516-4cf4-852a-9f32c51e0e2d> .
<http://127.0.0.1:8984/rest/.well-known/genid/3c/99/b4/e2/3c99b4e2-5516-4cf4-852a-9f32c51e0e2d> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://fedora.info/definitions/v4/repository#Skolem> .
<http://127.0.0.1:8984/rest/.well-known/genid/3c/99/b4/e2/3c99b4e2-5516-4cf4-852a-9f32c51e0e2d> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://schema.org/GeoShape> .
<http://127.0.0.1:8984/rest/.well-known/genid/3c/99/b4/e2/3c99b4e2-5516-4cf4-852a-9f32c51e0e2d> <http://www.opengis.net/ont/geosparql#asWKT> "POLYGON ((10 10, 40 10, 40 40, 10 40, 10 10))"^^<http://www.opengis.net/ont/geosparql#wktLiteral> .
```
