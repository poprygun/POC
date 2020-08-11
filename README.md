# Create demo flows to illustrate main steps.

## Prerequisites.  Refer to [installation page](https://dataflow.spring.io/docs/installation/local/manual/) for details

- [Install Spring Cloud Data Flow components](https://dataflow.spring.io/docs/installation/local/manual/)

```bash
wget https://repo.spring.io/release/org/springframework/cloud/spring-cloud-dataflow-server/2.6.0/spring-cloud-dataflow-server-2.6.0.jar -P bin \
wget https://repo.spring.io/release/org/springframework/cloud/spring-cloud-dataflow-shell/2.6.0/spring-cloud-dataflow-shell-2.6.0.jar -P bin  \
wget https://repo.spring.io/release/org/springframework/cloud/spring-cloud-skipper-server/2.5.0/spring-cloud-skipper-server-2.5.0.jar -P bin  \

```

- Make sure Docker is installed and running.

## Starting Server Components (RabbitMQ and MongoDB)

```bash
docker run -d --hostname rabbitmq --name rabbitmq -p 15672:15672 -p 5672:5672 rabbitmq:3.7.14-management
docker run -d -p 27017:27017 --name mongodb -d mongo
docker run -it --rm -p 8081:8081 --link [mongo container id]:mongo mongo-express

docker run -it --rm mongo mongo --host mongodb test

java -jar spring-cloud-skipper-server-2.5.0.jar \
java -jar spring-cloud-dataflow-server-2.6.0.jar \
```

[Dashboard](http://localhost:9393/)
[MongoDb](http://localhost:8081/)

Create MongoDb database=chachkies, collection=anything

## Starting Shell

```bash
java -jar spring-cloud-dataflow-shell-2.6.0.jar
```

## File to log - Using prebuilt apps

### Register apps

```bash
java -jar bin/spring-cloud-dataflow-shell-2.6.0.jar

app register --name file-source --type source --uri https://repo.spring.io/snapshot/org/springframework/cloud/stream/app/file-source-rabbit/2.1.4.BUILD-SNAPSHOT/file-source-rabbit-2.1.4.BUILD-SNAPSHOT.jar

app register --name file-processor --type processor --uri https://repo.spring.io/snapshot/org/springframework/cloud/stream/app/transform-processor-rabbit/2.1.4.BUILD-SNAPSHOT/transform-processor-rabbit-2.1.4.BUILD-SNAPSHOT.jar
 
app register --name logging-sink --type sink  --uri https://repo.spring.io/snapshot/org/springframework/cloud/stream/app/log-sink-rabbit/2.1.5.BUILD-SNAPSHOT/log-sink-rabbit-2.1.5.BUILD-SNAPSHOT.jar

```

### Create and deploy stream

```bash
stream create --name file-to-log --definition "file-source --file.directory=/Users/ashumilov/projects/raytheon/jpss/poc/data/in | file-processor --transformer.expression=#jsonPath(payload,'$') | logging-sink"
```

```bash
stream deploy --name file-to-log
```

Copying file to data/in directory will trigger a flow and content of the file will be logged.

## File to transformer to mongodb - using custom built apps

[Look here](https://vmware.slack.com/archives/C06PZDJDV/p1597162232110000) before going further and customize apps. 

Clone needed app [from repo](https://github.com/spring-cloud/stream-applications/tree/master/applications).

Build and install the application that was cloned in previous step and customized as needed.

Register app custom app.  Note added metadata-uri arg.

```bash
app register --name file-source-custom --type source --uri file:///Users/ashumilov/.m2/repository/io/microsamples/dataflow/file-source/3.0.0-SNAPSHOT/file-source-3.0.0-SNAPSHOT.jar --metadata-uri file:///Users/ashumilov/.m2/repository/io/microsamples/dataflow/file-source/3.0.0-SNAPSHOT/file-source-3.0.0-SNAPSHOT-metadata.jar
```

### Define MongoDb stream

```bash
stream create --name ftm-custom --definition "file-source-custom --directory=/Users/ashumilov/projects/raytheon/jpss/poc/data/in | file-processor --transformer.expression=#jsonPath(payload,'$') | mongodb-sink --uri=mongodb://localhost:27017/chachkies --collection=anything"

stream deploy --name fcm-custom
```


Copying file to data/in directory will trigger a flow and content of the file will be in [MongoDb](http://localhost:8081/).
