# Simple Retail Order Management System
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

## STEP 1: Set up the enviroment 
Refer to README.md file under **setup** folder

## STPE 2: Populate MongoDB with metadata
### fraud_id Metadata
This metadata is used in **validate_orders** data flow where each ipaddress in raw orders is checked against the fraud_ip collection if it is found or not. If found order is not processed further.

####
* **generate_random_ip.py** which will help us in generating the fraud_ip.json is found in **helper-kit** folder
* **fraud_id.json** can be found in **config/mongodb/* folder, which has the following structure.
#### Import data onto mongodb
```
$ mongoimport --db=locations --collection=fraud_ip --file=fraud_ip_address.json
```

### location_details Metadata
Location Details metadata is used in **enricher** dataflow where the valid orders data will be enriched with name of location based on location_id present in to_location and from_location

**location.json** can be found in **config/mongodb/** folder file, which has the following structure with respect to location.
```
{"location_id":1,
 "location":{"coordinates":[-73.856077,40.848447]},
 "logistic_id":"1",
 "name":"Morris Park Bake Shop"}
```
#### Import data onto mongodb
```
$ mongoimport --db=locations --collection=location_details --file=location.json
```

### product_details Metadata
This metadata is used in **router** data flow where based on the route_id the order is routed to internal or external order processing.
* **generate_random_products.py** which will help us in generting random product names is found in **helper-kit** folder
* **product_details.json** can be found in **config/mondodb** folder,which has the following structure.

```
{'product_id': 1000, 'product_name': 'skimpy-indigo-frigatebird', 'route_id': 1, 'category': 'Grocery'}
```
#### Import data onto mongodb
```
$ mongoimport --db=locations --collection=product_details --file=product_details.json
```
## STEP 3: Enable Druid Kafka Ingestion
**Kafka Supervisor specs are found under configs/druid-imply folder**
We will use Druid's Kafka indexing service to ingest messages from our all the topic related to all workflows and aggregators to create visibility across all the transformation phase in the pipeline. To start the service, we will need to submit all upervisor spec to the Druid overlord by running the following from the Imply home directory.
```
$ cd /home/druid/imply
$ curl -XPOST -H'Content-Type: application/json' -d @quickstart/wikipedia-kafka-supervisor.json http://localhost:8090/druid/indexer/v1/supervisor
```
## Step 4: Create logs dir
Create the logs dir where the workflows and aggregators python program files are located. If you want to have separate logs dir you can do it
by changing the LOG_FILE_NAME variable.
```
$ mkdir logs
```

## STEP 5: Starting the work flows  and aggregators
**Workflows and aggregators program files are found in workflow and aggregators directory**

### Start workflows pipeline
```
$ python3 validate_order.py &
$ python3 enrich_order_location.py &
$ python3 route_order.py &
```
#### Check the log files
```
$ tail -f logs/validate_order.logs
$ tail -f logs/enrich_order_location.log
$ tail -f logs/route_order.log

```
### Start the aggregators
In **two separate consoles** start the two spark jobs as it is been set to print on screen to see the computation value
```
$ python3 aggregate_product_price.py
$ python3 aggregate_to_location.py
```

### Start Order generation
Once all the workflows and aggregators are started, execute following program to generate random orders
```
$ python generate_order.py
```
#### Check the log files
```
$ tail -f logs/genrerate_order.log

```
