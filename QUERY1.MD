# Query 1: Frequent Routes

The goal of the query is to find the top 10 most frequent routes during the last 30 minutes. A route is represented by a starting grid cell and an ending grid cell. All routes completed within the last 30 minutes are considered for the query. The output query results must be updated whenever any of the 10 most frequent routes changes. The output format for the result stream is:

```
pickup_datetime, dropoff_datetime, start_cell_id_1, end_cell_id_1, ... , start_cell_id_10, end_cell_id_10, delay
```

where pickup_datetime, dropoff_datetime are the timestamps of the trip report that resulted in an update of the result stream, start_cell_id_X the starting cell of the Xth-most frequent route, end_cell_id_X the ending cell of the Xth-most frequent route. If less than 10 routes can be identified within the last 30 min, then NULL is to be output for all routes that lack data.

The attribute “delay” captures the time delay between reading the input event that triggered the output and the time when the output is produced. Participants must determine the delay using the current system time right after reading the input and right before writing the output. This attribute will be used in the evaluation of the submission.

The cells for this query are squares of 500 m X 500 m. The cell grid starts with cell 1.1, located at 41.474937, -74.913585 (in Barryville). The coordinate 41.474937, -74.913585 marks the center of the first cell. Cell numbers increase towards the east and south, with the shift to east being the first and the shift to south the second component of the cell, i.e., cell 3.7 is 2 cells east and 6 cells south of cell 1.1. The overall grid expands 150km south and 150km east from cell 1.1 with the cell 300.300 being the last cell in the grid. All trips starting or ending outside this area are treated as outliers and must not be considered in the result computation.


# 1. Local Deployment

## TODO: Local Deployment Architecture

## 1-1. Prerequisites for Local Deployment

* [Maven](http://brewformulas.org/Maven)
* [Spring Cloud Data Flow Local Server](http://repo.spring.io/libs-milestone/org/springframework/cloud/spring-cloud-dataflow-server-local/1.0.0.RC1/spring-cloud-dataflow-server-local-1.0.0.RC1.jar)
* [Spring Cloud Data Flow Shell](http://repo.spring.io/libs-milestone/org/springframework/cloud/spring-cloud-dataflow-shell/1.0.0.RC1/spring-cloud-dataflow-shell-1.0.0.RC1.jar)
* [RabbitMQ](http://brewformulas.org/Rabbitmq)

## 1-2. Start Data Flow Server and Shell

### 1-2-1. Start RabbitMQ

```
rabbitmq-server
```

### 1-2-2. Start Data Flow Server

```
java -jar spring-cloud-dataflow-server-local-1.0.0.RC1.jar --binder=rabbit
```

### 1-2-3. Start Data Flow Shell

```
java -jar spring-cloud-dataflow-shell-1.0.0.RC1.jar
```

## 1-3. Register Spring Cloud Stream Apps to Data Flow Server (Sending data to GemFire)

### 1-3-1. Register HTTP Source

In Dataflow Shell

```
app unregister --name http:source

app register --name http --type source --uri http://repo.spring.io/libs-milestone/org/springframework/cloud/stream/app/http-source-rabbit/1.0.0.RC1/http-source-rabbit-1.0.0.RC1.jar
```

Or you can build the HTTP Source from source code.
https://github.com/spring-cloud/spring-cloud-stream-app-starters

### 1-3-2. Register Flat2Tuple Processor

```
app unregister --name flat2tuple:processor

app register --name flat2tuple --type processor --uri https://rawgit.com/komushi/spring-cloud-stream/master/spring-cloud-stream-processor-flat2tuple-0.0.1-SNAPSHOT.jar
```
Or you can build the HTTP Source from source code.
https://github.com/komushi/spring-cloud-stream-processor-flat2tuple

### 1-3-3. Register GemFire Sink

```
app unregister --name gemfire:sink

app register --name gemfire --type sink --uri http://repo.spring.io/libs-milestone/org/springframework/cloud/stream/app/gemfire-sink-rabbit/1.0.0.RC1/gemfire-sink-rabbit-1.0.0.RC1.jar
```

Or you can build the GemFire Sink from source code.
https://github.com/spring-cloud/spring-cloud-stream-app-starters

## 1-4. Register Spring Cloud Stream Apps to Data Flow Server (Receiving data from GemFire)

### 1-4-1 Register GemFire Source

```
app unregister --name gemfire:source

app register --name gemfire --type source --uri http://repo.spring.io/libs-milestone/org/springframework/cloud/stream/app/gemfire-source-rabbit/1.0.0.RC1/gemfire-source-rabbit-1.0.0.RC1.jar
```

Or you can build the GemFire Sink from source code.
https://github.com/spring-cloud/spring-cloud-stream-app-starters

### 1-4-2 Register Stomp Sink

```
app unregister --name stomp:sink

app register --name stomp --type sink --uri https://rawgit.com/komushi/spring-cloud-stream/master/StompSink-0.0.1-SNAPSHOT.jar
```

Or you can build the GemFire Sink from source code.
https://github.com/komushi/spring-cloud-stream-sink-stomp

## 1-5 Start GemFire

Several options for GemFire

* [Embedded as Spring Boot App](https://github.com/komushi/spring-boot-gemfire-server)
* [Official](http://gemfire.docs.pivotal.io/docs-gemfire/getting_started/15_minute_quickstart_gfsh.html)

Here I will help you go through the steps with GemFire embedded as Spring Boot App.

### 1-5-1. Download & Build

```
git clone https://github.com/komushi/spring-boot-gemfire-server.git
cd spring-boot-gemfire-server
mvn clean package -DskipTests
```

### 1-5-2. Patch

Check details from https://github.com/komushi/spring-boot-gemfire-server

```
./patch-jar.sh
```

### 1-5-3. Start GemFire as Spring Boot App

```
java -jar target/spring-boot-gemfire-server-0.0.1-SNAPSHOT.jar
```

## 1-6. Create Stream on Spring Data Flow Server

## 1-6-1 Receive data from HTTP and send to GemFire

```
stream create http2gem --definition "http --server.port=9000 | flat2tuple --outputType=application/json | gemfire --gemfire.region.regionName=RegionRaw --gemfire.json=true --gemfire.keyExpression=payload.getField('uuid')" --deploy
```

## 1-6-2 Receive data from GemFire and send to Stomp(WebSocket)

```
stream create gem2stomp --definition "gemfire --gemfire.region.region-name=RegionTopTen | stomp --stomp.withSockJS=true --logging.level.io.pivotal.spring.cloud.stream.sink=TRACE" --deploy
```

## 1-7. Start Browser

```
```

#2 Local deployment without Spring Cloud Data Flow Server

## 2-1. Prerequisites for Local Deployment

* [Maven](http://brewformulas.org/Maven)
* [RabbitMQ](http://brewformulas.org/Rabbitmq)

## 2-2. Start GemFire

### 2-2-1. Start GemFire as Spring Boot App

```
java -jar target/spring-boot-gemfire-server-0.0.1-SNAPSHOT.jar
```

## 2-3. Start Spring Cloud Stream Applications to receive data from HTTP and send to GemFire

### 2-3-1. Start Http Source

```
java -jar http-source-rabbit-1.0.0.RC1.jar --server.port=9000 --spring.cloud.stream.bindings.output.destination=http2tuple

```

### 2-3-2. Start Flat2Tuple Processor

```
java -jar target/spring-cloud-stream-processor-flat2tuple-0.0.1-SNAPSHOT.jar --server.port=9090 --outputType=application/json --spring.cloud.stream.bindings.output.destination=tuple2gem --spring.cloud.stream.bindings.input.destination=http2tuple
```

### 2-3-3. Start GemFire Sink

```
java -jar gemfire-sink-rabbit-1.0.0.RC1.jar --server.port=9999 --gemfire.region.region-name=RegionRaw --gemfire.json=true --gemfire.key-expression="payload.getField('uuid')" --spring.cloud.stream.bindings.input.destination=tuple2gem

```

## 2-4. Start Spring Cloud Stream Applications to receive data from GemFire and send to Stomp(WebSocket)

### 2-4-1. Start GemFire Source

```
java -jar gemfire-source-rabbit-1.0.0.RC1.jar --server.port=8000 --gemfire.cacheEventExpression=newValue --gemfire.region.region-name=RegionTopTen --spring.cloud.stream.bindings.output.destination=topten
```

####

```
java -jar StompSink-0.0.1-SNAPSHOT.jar --server.port=8008 --stomp.withSockJS=true --logging.level.io.pivotal.spring.cloud.stream.sink=TRACE --spring.cloud.stream.bindings.input.destination=topten
```

### 2-4-3. Start Log Sink

```
java -jar log-sink-rabbit-1.0.0.RC1.jar --server.port=8888 --spring.cloud.stream.bindings.input.destination=topten
```