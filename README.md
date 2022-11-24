# Simple example of Confluent Audit logs

## Goal

The goal of this example is to configure a very simple cluster (one zookeeper + one broker) with Confluent Audit logs enabled (using property `KAFKA_AUTHORIZER_CLASS_NAME`). Once running, we can verify the default configuration and how the events are sent to the topic `confluent-audit-log-events`.

## How to run

1. Clone repo and start docker

```shell 
    git clone git@github.com:tomasalmeida/confluent-audit-logs-example.git 
    cd confluent-audit-logs-example
    docker-compose up -d
```

2. Check (and wait) until the topic `confluent-audit-log-events` is created:

```shell
    kafka-topics --bootstrap-server localhost:19092 --command-config configs/host.properties --list
```


3. Start consuming from this topic. For convenience, we use `jq` to format the json events in a human readable format

```shell
    kafka-console-consumer --bootstrap-server localhost:19092 --consumer.config configs/host.properties --topic confluent-audit-log-events | jq
```

4. Create a new topic and after it, produce and consume from it.

```shell
    kafka-topics --bootstrap-server localhost:19092 --command-config configs/host.properties --create --topic test
    kafka-console-producer --bootstrap-server localhost:19092 --producer.config configs/host.properties --topic test
    kafka-console-consumer --bootstrap-server localhost:19092 --consumer.config configs/host.properties --topic test
```

5. You can see the events are created by Confluent Audit logs for the topic creation


5.1. Creation event for topic in the cluster

```json
{
    "specversion": "1.0",
    "id": "e5ababff-c6b5-4f19-9e03-be3f12e81643",
    "source": "crn:///kafka=dAkAmyKqTTSosZSZzR-r3g",
    "type": "io.confluent.kafka.server/authorization",
    "datacontenttype": "application/json",
    "subject": "crn:///kafka=dAkAmyKqTTSosZSZzR-r3g",
    "time": "2022-11-24T08:28:32.577593Z",
    "route": "confluent-audit-log-events",
    "data": {
        "serviceName": "crn:///kafka=dAkAmyKqTTSosZSZzR-r3g",
        "methodName": "kafka.CreateTopics",
        "resourceName": "crn:///kafka=dAkAmyKqTTSosZSZzR-r3g",
        "authenticationInfo": {
            "principal": "User:hostclient"
        },
        "authorizationInfo": {
            "granted": true,
            "operation": "Create",
            "resourceType": "Cluster",
            "resourceName": "kafka-cluster",
            "patternType": "LITERAL"
        },
        "request": {
            "correlation_id": "3",
            "client_id": "adminclient-1"
        },
        "requestMetadata": {
            "client_address": "/172.28.0.1"
        }
    }
}
```

5.2. Creation event for topic with resourceName as the topic itself

```json

{
    "specversion": "1.0",
    "id": "3ad21961-13be-4e71-a583-d1b29e488ff6",
    "source": "crn:///kafka=dAkAmyKqTTSosZSZzR-r3g",
    "type": "io.confluent.kafka.server/authorization",
    "datacontenttype": "application/json",
    "subject": "crn:///kafka=dAkAmyKqTTSosZSZzR-r3g/topic=test",
    "time": "2022-11-24T08:28:32.579313Z",
    "route": "confluent-audit-log-events",
    "data": {
        "serviceName": "crn:///kafka=dAkAmyKqTTSosZSZzR-r3g",
        "methodName": "kafka.CreateTopics",
        "resourceName": "crn:///kafka=dAkAmyKqTTSosZSZzR-r3g/topic=test",
        "authenticationInfo": {
            "principal": "User:hostclient"
        },
        "authorizationInfo": {
            "granted": true,
            "operation": "DescribeConfigs",
            "resourceType": "Topic",
            "resourceName": "test",
            "patternType": "LITERAL"
        },
        "request": {
            "correlation_id": "3",
            "client_id": "adminclient-1"
        },
        "requestMetadata": {
            "client_address": "/172.28.0.1"
        }
    }
}
```