# Hyrax

## Examples for integrating Rails models into Samvera repositories using the Hyrax solution bundle

## Models

### GeoShapes
```ruby
class GeoShape
  include  ActiveTriples::RDFSource

  configure type: ::RDF::URI.new('http://schema.org/GeoShape'), base_uri: 'http://institution.edu/georepository/'
  # [...]
  property :geo, predicate: ::RDF::URI.new('http://www.opengis.net/ont/geosparql#asWKT')
end
```
```
> shape = GeoShape.new
> shape.geo = RDF::Literal.new("POLYGON ((30 10, 40 40, 20 40, 10 20, 30 10))", datatype:"geo:wktLiteral")
> puts shape.dump :ntriples
_:g70334881126520 <geo:asWKT> "POLYGON ((30 10, 40 40, 20 40, 10 20, 30 10))"^^<geo:wktLiteral> .
_:g70334881126520 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <schema:GeoShape> .
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
_:g70334930722580 <schema:geo> _:g70334881126520 .
_:g70334930722580 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <schema:Place> .
_:g70334881126520 <geo:asWKT> "POLYGON ((30 10, 40 40, 20 40, 10 20, 30 10))"^^<geo:wktLiteral> .
_:g70334881126520 <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <schema:GeoShape> .
```

### (Linked) Geospatial Works
```ruby
class GenericGeoWork < ActiveFedora::Base
  include ::Hyrax::WorkBehavior

  self.indexer = GenericGeoWorkIndexer
  # Change this to restrict which works can be added as a child.
  # self.valid_child_concerns = []
  validates :title, presence: { message: 'Your work must have a title.' }

  property :spatial_coverage, predicate: ::RDF::URI.new('http://schema.org/spatialCoverage'), class_name: 'Place'
  # [...]
  # This must be included at the end, because it finalizes the metadata
  # schema (by adding accepts_nested_attributes)
  include ::Hyrax::BasicMetadata
end
```
```
> linked_geo_work = GenericGeoWork.new
> linked_geo_work.title = ['My linked geo. work']
> linked_geo_work.spatial_coverage = place
> linked_geo_work.save
> persisted = linked_geo_work.reload
> persisted.resource.dump :ntriples
<http://127.0.0.1:8984/rest/.well-known/genid/08/2c/84/21/082c8421-d113-4588-a222-f803458d52bd> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <schema:Place> .
<http://127.0.0.1:8984/rest/.well-known/genid/08/2c/84/21/082c8421-d113-4588-a222-f803458d52bd> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://fedora.info/definitions/v4/repository#Skolem> .
[...]
<http://127.0.0.1:8984/rest/dev/08/61/2n/52/08612n52b> <schema:spatialCoverage> <http://127.0.0.1:8984/rest/.well-known/genid/08/2c/84/21/082c8421-d113-4588-a222-f803458d52bd> .
[...]
<http://127.0.0.1:8984/rest/dev/08/61/2n/52/08612n52b> <info:fedora/fedora-system:def/model#hasModel> "GenericGeoWork" .
<http://127.0.0.1:8984/rest/dev/08/61/2n/52/08612n52b> <http://purl.org/dc/terms/title> "My linked geo. work" .
```
