---
layout: post
title: Mirroring kafka topic between kafka cluster
---

Kafka data migration or mirroring between kafka cluster

### 1.1. Feature
Feature:
- Mirroring kafka topic between kafka cluster
- Mirroring kafka consumer group
- Kafka Mirror Maker 2 Docker 
- Kafka Mirror Maker 2 Kubernetes (Strimzi Kafka Operator)


### Creating mirror maker 2
#### MM2 Docker
`docker-compose.yml`
{% highlight shell %}
   mirrormaker:
     image: 'izalul/mirror-maker:2.2'
     depends_on:
       - kafka1
     environment:
       - ALLOW_PLAINTEXT_LISTENER=yes
       - SOURCE=192.168.1.20:9092
       - DESTINATION=192.168.1.30:9092
       - TOPICS=.*
{% endhighlight %}

Deploy mirror maker 2 using command `docker-compose up -d`.

#### MM2 Kubernetes (Strimzi kafka operator)
`mm2-kafka.yaml`
{% highlight shell %}
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaMirrorMaker2
metadata:
  name: mirrormaker2-kafka
  namespace: kafka
spec:
  version: 3.0.0
  replicas: 1
  connectCluster: "my-target-docker-cluster"
  clusters:
  - alias: "my-source-docker-cluster"
    bootstrapServers: 192.168.1.20:9092
    config:
      ssl.endpoint.identification.algorithm: ""
  - alias: "my-target-docker-cluster"
    bootstrapServers: 192.168.1.30:9092
    config:
      ssl.endpoint.identification.algorithm: ""
      producer.max.request.size: "104857600"
      producer.buffer.memory: "104857600"
  mirrors:
  - sourceCluster: "my-source-docker-cluster"
    targetCluster: "my-target-docker-cluster"
    sourceConnector:
      tasksMax: 12
      config:
        replication.factor: 3
        offset-syncs.topic.replication.factor: 3
        sync.topic.acls.enabled: "false"
        replication.policy.separator: ""
        replication.policy.class: "io.strimzi.kafka.connect.mirror.IdentityReplicationPolicy"
    heartbeatConnector:
      config:
        heartbeats.topic.replication.factor: 3
    checkpointConnector:
      config:
        sync.topic.acls.enabled: "false"
        sync.group.offsets.enabled: "true"
        consumer.max.partition.fetch.bytes: 10485760
        max.partition.fetch.bytes: 10485760
        producer.max.request.size: "104857600"
        producer.buffer.memory: "104857600"
        replication.policy.class: "io.strimzi.kafka.connect.mirror.IdentityReplicationPolicy"        
    topicsPattern: ".*"
    groupsPattern: ".*"
  logging: 
    type: inline
    loggers:
      connect.root.logger.level: "INFO"
  readinessProbe: 
    initialDelaySeconds: 25
    timeoutSeconds: 25
  livenessProbe:
    initialDelaySeconds: 25
    timeoutSeconds: 25

{% endhighlight %}


deploy kafka cluster using command `kubectl apply -f mm2-kafka.yaml`.

if you use ACL on strimzi kafka cluster, create a user who has access to all topics and groups (super user).

#### Example super user (Strimzi kafka operator)
`super-user.yaml`
{% highlight shell %}
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaUser
metadata:
  name: super-user
  namespace: kafka
  labels:
    strimzi.io/cluster: my-cluster
spec:
  authentication:
    type: tls
  authorization:
    type: simple
    acls:
      # Example consumer Acls for topic my-topic using consumer group my-group
      - resource:
          type: topic
          name: '*'
          patternType: literal
        operation: All
      - resource:
          type: group
          name: '*'
          patternType: literal
        operation: All
        host: "*"
{% endhighlight %}


May this is help for you who implement Kafka cluster on docker or Kubernetes. 
