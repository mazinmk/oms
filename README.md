# Sample Retail Order Management System
To design a sample **retail order management system** which has the capability to process a couple of millions orders per day.

## Technical stack 
* OS : Ubuntu-18.04
* Messaging: kafka_2.13-2.5.0
* Time series Database: Druid
* Analytics: Imply-3.3.5
* Metastore: MongoDB-3.6.3
* Dataprocessing: Spark-3.0.0
* Java: 11.0.7
* Python:3.0

**NOTE** For setup please refer to README.md file under setup folder

## Populate MongoDB with metadata
### location_details Metadata
Location Details metadata is used to enrich dataflow where the valid orders data will be enriched with name of location based on location_id present in to_location and from_location

#### Example

**location.json** can be found in **conf** folder file, which has the following metadata with respect to location.
```
{"location_id":1,
 "location":{"coordinates":[-73.856077,40.848447]},
 "logistic_id":"1",
 "name":"Morris Park Bake Shop"}
```
```
mongoimport --db=locations --collection=location_details --file=location.json




