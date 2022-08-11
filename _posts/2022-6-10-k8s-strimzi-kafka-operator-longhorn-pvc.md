---
layout: post
title: Deploy kafka on Kubernetes using strimzi kafka operator and longhorn
---

Strimzi simplifies the process of running Apache Kafka in a Kubernetes cluster.

This guide provides instructions for evaluating a working environment of Strimzi. The steps describe how to get a Strimzi deployment up-and-running as quickly as possible.

Before trying Strimzi, it is useful to understand its capabilities and how you might wish to use it. This chapter introduces some of the key concepts behind Kafka, and also provides a brief overview of the Strimzi Operators.

Operators are a method of packaging, deploying, and managing a Kubernetes application. Strimzi Operators extend Kubernetes functionality, automating common and complex tasks related to a Kafka deployment. By implementing knowledge of Kafka operations in code, Kafka administration tasks are simplified and require less manual intervention.

### 1.1. Kafka capabilities
- The underlying data stream-processing capabilities and component architecture of Kafka can deliver:
- Microservices and other applications to share data with extremely high throughput and low latency
- Message ordering guarantees
- Message rewind/replay from data storage to reconstruct an application state
- Message compaction to remove old records when using a key-value log
- Horizontal scalability in a cluster configuration
- Replication of data to control fault tolerance

### 1.2. Kafka use cases
Kafka’s capabilities make it suitable for:

- Event-driven architectures
- Event sourcing to capture changes to the state of an application as a log of events 
- Message brokering
- Website activity tracking
- Operational monitoring through metrics
- Log collection and aggregation
- Commit logs for distributed systems
- Stream processing so that applications can respond to data in real time

### 1.3. How Strimzi supports Kafka
Strimzi provides container images and Operators for running Kafka on Kubernetes. Strimzi Operators are fundamental to the running of Strimzi. The Operators provided with Strimzi are purpose-built with specialist operational knowledge to effectively manage Kafka.

Operators simplify the process of:

- Deploying and running Kafka clusters
- Deploying and running Kafka components
- Configuring access to Kafka
- Securing access to Kafka
- Upgrading Kafka
- Managing brokers
- Creating and managing topics
- Creating and managing users

### 1.4. Operators
Strimzi provides Operators for managing a Kafka cluster running within a Kubernetes cluster.

##### Cluster Operator
Deploys and manages Apache Kafka clusters, Kafka Connect, Kafka MirrorMaker, Kafka Bridge, Kafka Exporter, Cruise Control, and the Entity Operator

##### Entity Operator
Comprises the Topic Operator and User Operator

##### Topic Operator
Manages Kafka topics

##### User Operator
Manages Kafka users

The Cluster Operator can deploy the Topic Operator and User Operator as part of an Entity Operator configuration at the same time as a Kafka cluster.

### Installing Strimzi
Create a new kafka namespace for the Strimzi Kafka Cluster Operator.

{% highlight shell %}
kubectl create ns kafka
{% endhighlight %}
Modify the installation files to reference the kafka namespace where you will install the Strimzi Kafka Cluster Operator.
{% highlight shell %}
sed -i 's/namespace: .*/namespace: kafka/' install/cluster-operator/*RoleBinding*.yaml
{% endhighlight %}
Edit the install/cluster-operator/060-Deployment-strimzi-cluster-operator.yaml file and set the STRIMZI_NAMESPACE environment variable to the namespace kafka.
{% highlight shell %}
# ...
env:
- name: STRIMZI_NAMESPACE
  value: my-kafka-project
# ...
{% endhighlight %}
Give permission to the Cluster Operator to watch the kafka namespace.
{% highlight shell %}
kubectl create -f install/cluster-operator/020-RoleBinding-strimzi-cluster-operator.yaml -n kafka
{% endhighlight %}
{% highlight shell %}
ubectl create -f install/cluster-operator/031-RoleBinding-strimzi-cluster-operator-entity-operator-delegation.yaml -n kafka
{% endhighlight %}
The commands create role bindings that grant permission for the Cluster Operator to access the Kafka cluster.

Deploy the CRDs and role-based access control (RBAC) resources to manage the CRDs.

{% highlight shell %}
kubectl create -f install/cluster-operator/ -n kafka
{% endhighlight %}

### Creating a cluster
- Use persistent-claim storage
- Expose the Kafka cluster outside of the Kubernetes cluster using an external listener configured to use a nodeport.
{% highlight shell %}
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: my-cluster
  namespace: kafka
spec:
  kafka:
    template:
      # kafkaContainer:
      #   env:
      #   - name: TZ
      #     value: Asia/Jakarta
      pod:
        securityContext:
          runAsUser: 0
          fsGroup: 0
        affinity:
          nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
              nodeSelectorTerms:
              - matchExpressions:
                - key: kubernetes.io/hostname
                  operator: In
                  values:
                  - k8s-node-01
    version: 3.0.0
    replicas: 3
    listeners:
      - name: plain
        port: 9092
        type: internal
        tls: false
      - name: backup
        port: 9095
        type: internal
        tls: {}
      - name: tls
        port: 9093
        type: internal
        tls: true
      - name: external
        port: 9094
        type: nodeport
        tls: true
        authentication:
          type: tls
        configuration:
          bootstrap:
            nodePort: 30056
          # brokers:
          # - broker: 0
          #   nodePort: 32282
          # - broker: 1
          #   nodePort: 30989
          # - broker: 2
          #   nodePort: 32282
    config:
      auto.create.topics.enable: "true"
      offsets.topic.replication.factor: 3
      transaction.state.log.replication.factor: 3
      transaction.state.log.min.isr: 2
      default.replication.factor: 3
      inter.broker.protocol.version: "3.0"
      message.max.bytes: 15728640
      replica.fetch.max.bytes: 15728640
    authorization:
      type: simple
    storage:
      type: persistent-claim
      size: 2000Gi
      class: longhorn
      deleteClaim: false
    resources:
      requests:
        memory: 64Gi
        cpu: "8"
      limits:
        memory: 64Gi
        cpu: "12"
    jvmOptions:
      -Xms: 9192m
      -Xmx: 9192m
  zookeeper:
    template:
      pod:
        affinity:
          nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
              nodeSelectorTerms:
              - matchExpressions:
                - key: kubernetes.io/hostname
                  operator: In
                  values:
                  - k8s-node-01
        securityContext:
          runAsUser: 0
          fsGroup: 0
    replicas: 3
    storage:
      type: persistent-claim
      size: 50Gi
      class: longhorn
      deleteClaim: false
    resources:
      requests:
        memory: 8Gi
        cpu: "2"
      limits:
        memory: 8Gi
        cpu: "2"
    jvmOptions:
      -Xms: 4096m
      -Xmx: 4096m
    logging:
      type: inline
      loggers:
        zookeeper.root.logger: "INFO"
  entityOperator:
    topicOperator: {}
    userOperator: {}
  kafkaExporter:
    topicRegex: ".*"
    groupRegex: ".*"
{% endhighlight %}

When your cluster is ready, create a topic to publish and subscribe from your external client.
{% highlight shell %}
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaTopic
metadata:
  name: my-topic
  namespace: kafka
  labels:
    strimzi.io/cluster: my-cluster
spec:
  partitions: 1
  replicas: 3
  config:
    max.message.bytes: '15728640'
{% endhighlight %}

When your cluster is ready, create a topic to publish and subscribe from your external client.




May this is help for you who implement Kafka on Kubernetes. 
