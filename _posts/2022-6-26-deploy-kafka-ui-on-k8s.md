---
layout: post
title: Monitoring and manage kafka multi cluster
---

Kafka user interface
UI for Apache Kafka is a simple tool that makes your data flows observable, helps find and troubleshoot issues faster and deliver optimal performance. Its lightweight dashboard makes it easy to track key metrics of your Kafka clusters - Brokers, Topics, Partitions, Production, and Consumption.


### 1.1. Feature
Feature:
- Multi-Cluster Management — monitor and manage all your clusters in one place
- Performance Monitoring with Metrics Dashboard — track key Kafka metrics with a lightweight dashboard
- View Kafka Brokers — view topic and partition assignments, controller status
- View Kafka Topics — view partition count, replication status, and custom configuration
- View Consumer Groups — view per-partition parked offsets, combined and per-partition lag
- Browse Messages — browse messages with JSON, plain text, and Avro encoding
- Dynamic Topic Configuration — create and configure new topics with dynamic configuration
- Configurable Authentification — secure your installation with optional Github/Gitlab/Google OAuth 2.0

https://github.com/provectus/kafka-ui

### Creating kafka ui
#### MM2 Docker
`kafka-ui.yaml`
create namespace `kubectl create ns kafka-ui`

{% highlight shell %}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kafka-ui
  namespace: kafka-ui
  labels:
    app: kafka-ui
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kafka-ui
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: kafka-ui
    spec:
      containers:
        - image: provectuslabs/kafka-ui:latest
          imagePullPolicy: Always
          name: kafka-ui
          resources:
            limits:
              cpu: 200m
              memory: 1Gi
            requests:
              cpu: 200m
              memory: 1Gi
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          ports:
            - containerPort: 8080
              name: server
          env:
            - name: KAFKA_CLUSTERS_0_NAME
              value: "(PROD) kafka-cluster"
            - name: KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS
              value: "192.168.1.20:9092"
            - name: KAFKA_CLUSTERS_0_ZOOKEEPER
              value: "192.168.1.20:2181"
            - name: KAFKA_CLUSTERS_1_NAME
              value: "(BACKUP) kafka-cluster"
            - name: KAFKA_CLUSTERS_1_BOOTSTRAPSERVERS
              value: "192.168.1.30:9092"
            - name: KAFKA_CLUSTERS_1_ZOOKEEPER
              value: "192.168.1.30:2181"
      restartPolicy: Always

---
apiVersion: v1
kind: Service
metadata:
  name: kafka-ui-service
  namespace: kafka-ui
  labels:
    app: kafka-ui
spec:
  ports:
    - protocol: "TCP"
      port: 8080
      name: server
      nodePort: 30777
  selector:
    app: kafka-ui
  type: NodePort

{% endhighlight %}

access kafka ui `HOSTIP:30777`


May this is help for you who implement Kafka cluster on docker or Kubernetes. 
