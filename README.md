# Schema Exporter for Confluent Cloud
Exporter scripts for Confluent Schema Registry.

A tool to export all schemas from a Confluent Cloud Schema Registry to another.
This app supports two modes: `batchExport` and `sync`

- `batchExport` will do a one time migration between schema registries, then it will reset the destination registry to `READWRTIE` mode.
- `sync` will continuously sync newly registered schemas into the destination registry

If you are looking to migrate schemas between On-Prem and Confluent Cloud, check out 
[Confuent Replicator](https://docs.confluent.io/current/connect/kafka-connect-replicator/index.html).

The exporter expects the following variables to be set in the environment to make the necessary calls:

- `SRC_SR_URL` : The URL for the source Schema Registry
- `SRC_API_KEY` : The API KEY to be used to make calls to the source Schema Registry
- `SRC_API_SECRET` : The API SECRET to be used to make calls to the source Schema Registry
- `DST_SR_URL` : The URL for the destination Schema Registry
- `DST_API_KEY` : The API KEY to be used to make calls to the destination Schema Registry
- `DST_API_SECRET` : The API SECRET to be used to make calls to the destination Schema Registry

It is also possible to define the credentials through command flags. If both are defined, the flags take precedence.

## Build
````
git clone https://github.com/abraham-leal/ccloud-schema-exporter
cd ccloud-schema-exporter
go build ./cmd/ccloud-schema-exporter/ccloud-schema-exporter.go 
````

## Docker Run
````
docker run \
  -e SRC_SR_URL=$SRC_SR_URL \
  -e SRC_API_KEY=$SRC_API_KEY \
  -e SRC_API_SECRET=$SRC_API_SECRET \
  -e DST_SR_URL=$DST_SR_URL \
  -e DST_API_KEY=$DST_API_KEY \
  -e DST_API_SECRET=$DST_API_SECRET \
  abrahamleal/ccloud-schema-exporter:latest
````

A sample docker-compose is also provided at the root of this directory.

The docker image is built to handle the case of `-sync` continuous sync. For a one time export, it is recommended
to use a release binary.

For Docker, the `latest` tag will build directly from master. The master branch of this project is kept non-breaking;
However, for stable images tag a release.

## Run
- `./ccloud-schema-exporter -batchExport` : Running the app with this flag will perform a batch export.
- `./ccloud-schema-exporter -sync` : Running the app with this flag will start a continuous sync 
between source and destination schema registries.

When both flags are applied, `sync` mode prevails.

### Options

````
Usage of ./ccloud-schema-exporter:

  -batchExport
    	Perform a one-time export of all schemas
  -deleteAllFromDestination
    	Setting this will run a delete on all schemas written to the destination registry
  -dest-sr-key string
    	API KEY for the Destination Schema Registry Cluster
  -dest-sr-secret string
    	API SECRET for the Destination Schema Registry Cluster
  -dest-sr-url string
    	Url to the Source Schema Registry Cluster
  -scrapeInterval int
    	Amount of time ccloud-schema-exporter will delay between schema sync checks (default 60)
  -src-sr-key string
    	API KEY for the Source Schema Registry Cluster
  -src-sr-secret string
    	API SECRET for the Source Schema Registry Cluster
  -src-sr-url string
    	Url to the Source Schema Registry Cluster
  -sync
    	Sync schemas continuously
  -timeout int
    	Timeout, in seconds, to use for all REST call with the Schema Registries (default 60)
  -usage
    	Print the usage of this tool
  -version
    	Print the current version and exit
````

#### Source Schema Deletion

This application does *NOT* auto-delete schemas in the destination registry. This is to take a cautious approach to
schema preservation. Operators should decide when a schema should be either soft or permanently deleted from any given 
Schema Registry cluster.

#### Example Usage 
````
export SRC_SR_URL=XXXX
export SRC_API_KEY=XXXX
export SRC_API_SECRET=XXXX
export DST_SR_URL=XXXX
export DST_API_KEY=XXXX
export DST_API_SECRET=XXXX
./ccloud-schema-exporter <-sync | -batchExport>
````


