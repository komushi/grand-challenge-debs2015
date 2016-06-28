# Query 1: Frequent Routes

The goal of the query is to find the top 10 most frequent routes during the last 30 minutes. A route is represented by a starting grid cell and an ending grid cell. All routes completed within the last 30 minutes are considered for the query. The output query results must be updated whenever any of the 10 most frequent routes changes. The output format for the result stream is:

```
pickup_datetime, dropoff_datetime, start_cell_id_1, end_cell_id_1, ... , start_cell_id_10, end_cell_id_10, delay
```

where pickup_datetime, dropoff_datetime are the timestamps of the trip report that resulted in an update of the result stream, start_cell_id_X the starting cell of the Xth-most frequent route, end_cell_id_X the ending cell of the Xth-most frequent route. If less than 10 routes can be identified within the last 30 min, then NULL is to be output for all routes that lack data.

The attribute “delay” captures the time delay between reading the input event that triggered the output and the time when the output is produced. Participants must determine the delay using the current system time right after reading the input and right before writing the output. This attribute will be used in the evaluation of the submission.

The cells for this query are squares of 500 m X 500 m. The cell grid starts with cell 1.1, located at 41.474937, -74.913585 (in Barryville). The coordinate 41.474937, -74.913585 marks the center of the first cell. Cell numbers increase towards the east and south, with the shift to east being the first and the shift to south the second component of the cell, i.e., cell 3.7 is 2 cells east and 6 cells south of cell 1.1. The overall grid expands 150km south and 150km east from cell 1.1 with the cell 300.300 being the last cell in the grid. All trips starting or ending outside this area are treated as outliers and must not be considered in the result computation.

# 1. Local Deployment Architecture 
 


# 2. Local Deployment Steps 

## 2-1. Prerequisites for local

* [Maven](http://brewformulas.org/Maven)
* [Spring Cloud Data Flow Local Server](http://repo.spring.io/libs-milestone/org/springframework/cloud/spring-cloud-dataflow-server-local/1.0.0.RC1/spring-cloud-dataflow-server-local-1.0.0.RC1.jar)
* [Spring Cloud Data Flow Shell](http://repo.spring.io/libs-milestone/org/springframework/cloud/spring-cloud-dataflow-shell/1.0.0.RC1/spring-cloud-dataflow-shell-1.0.0.RC1.jar)
* [RabbitMQ](http://brewformulas.org/Rabbitmq)

## 2-2. Start Data Flow Server and Shell

### 2-2-1. Start RabbitMQ

```
rabbitmq-server
```

### 2-2-2. Start Data Flow Server

```
java -jar spring-cloud-dataflow-server-local-1.0.0.RC1.jar --binder=rabbit
```

### 2-2-3. Start Data Flow Shell

```
java -jar spring-cloud-dataflow-shell-1.0.0.RC1.jar
```

## 2-3. Stream to transfer data from simulator to GemFire 

### 2-3-1. Register HTTP Source

In Dataflow Shell

```
app unregister --name http:source

app register --name http --type source --uri http://repo.spring.io/libs-milestone/org/springframework/cloud/stream/app/http-source-rabbit/1.0.0.RC1/http-source-rabbit-1.0.0.RC1.jar
```

### 2-3-2. Register Flat2Tuple Processor

```
app unregister --name flat2tuple:processor

app register --name flat2tuple --type processor --uri https://rawgit.com/komushi/spring-cloud-stream/master/Flat2TupleTransformer-0.0.1-SNAPSHOT.jar
```

### 2-3-3. Register GemFire Sink

```
app unregister --name flat2tuple:processor

app register --name flat2tuple --type processor --uri https://rawgit.com/komushi/spring-cloud-stream/master/Flat2TupleTransformer-0.0.1-SNAPSHOT.jar
```

```
stream create http2gem --definition "http --server.port=9000 | flat2tuple --outputType=application/json | gemfire --regionName=RegionRaw --json=true --keyExpression=payload.getField('uuid')" --deploy
```
