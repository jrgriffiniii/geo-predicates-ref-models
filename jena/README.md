# Apache Jena
## Running these examples using the Jena graph database

## Installing Jena

### Downloading the binary
Retrieve the binary distribution of Jena with Fuseki from https://jena.apache.org/download/index.cgi

### Decompress the GZip-compressed TAR into a directory
```
mkdir -p /opt/jena/releases
tar -xvf apache-jena-fuseki-3.x.x.tar.gz -C /opt/jena/releases/
```

### Run the server from the binary
```
cd /opt/jena/releases/apache-jena-fuseki-3.x.x
./fuseki-server
```

### Create the Fuseki dataset within the admin. dashboard
* Visit http://localhost:3030/index.html
* Create a new dataset by accessing http://localhost:3030/manage.html
* Select "add new dataset" with a dataset name
* For the purposes of this demonstration "test1" is used

## Insert data using SPARQL UPDATE statements
```
./bin/s-update --service http://localhost:3030/test1/update @[PATH_TO_GIT_REPO]/updates/insert_data_scanned_map.rq
```

## Querying for the data
```
./bin/s-query --service http://localhost:3030/test1/query @[PATH_TO_GIT_REPO]/queries/jena_spatial_scanned_map.rq --output=xml
```

## Delete the data
```
./bin/s-update --service http://localhost:3030/test1/update @[PATH_TO_GIT_REPO]/delete_data_jena_scanned_map.rq
```
