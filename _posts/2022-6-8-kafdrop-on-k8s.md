---
layout: post
title: Deploy Kafdrop on Kubernetes
---

What is Kafka?
Apache Kafka is an open-source platform. Kafka was originally developed by Linkedin and was later incubated as the Apache Project. It can process over 1 million messages per second.

Need for Kafdrop:
Kafka is an amazing platform for processing a huge number of messages very quickly. However, Kafka has one disadvantage that it does not come with an inbuilt User Interface where the users can see the information related to Kafka.

Kafdrop helps us in solving this problem. It gives us a simple, lightweight, and easy-to-use User Interface where one can not only see the required information but can also create and delete Kafka topics.

What Can It Do?
- View Kafka brokers — topic and partition assignments, and controller status
- View topics — partition count, replication status, and custom configuration
- Browse messages — JSON, plain text and Avro encoding
- View consumer groups — per-partition parked offsets, combined and per-partition lag
- Create new topics
- View ACLs

### Deploy Kafdrop on K8s
You can also install KafDrop on the Kubernetes cluster with the helm of the manifest file. Create a YAML file called kafdrop-deployment.yaml with the following content in it:

#### kafdrop-deployment.yaml

{% highlight shell %}
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kafka-kafdrop-deployment
  namespace: kafdrop
  labels:
    app: kafka-kafdrop
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kafka-kafdrop
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: kafka-kafdrop
    spec:
      volumes:
        - name: tz-config
          hostPath:
            path: /usr/share/zoneinfo/Asia/Jakarta
      containers:
        - image: obsidiandynamics/kafdrop
          imagePullPolicy: Always
          name: kafka-kafdrop
          volumeMounts:
            - name: tz-config
              mountPath: /etc/localtime
          resources:
            limits:
              cpu: 200m
              memory: 500Mi
            requests:
              cpu: 200m
              memory: 500Mi
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          ports:
            - containerPort: 5010
              name: server
            - containerPort: 5012
              name: jmx
          env:
            - name: JVM_OPTS
              value: "-Xms512M -Xms512M"
            - name: SERVER_SERVLET_CONTEXTPATH
              value: "/"
            - name: KAFKA_BROKERCONNECT
              value: "your kafka broker IP and Port"
            - name: KAFKA_PROPERTIES
              value: "your kafka config.properties encode to base64"
            - name: KAFKA_TRUSTSTORE
              value: "your kafka truestore encode to base64"
            - name: KAFKA_KEYSTORE
              value: "your kafka keystore encode to base64"
      restartPolicy: Always
{% endhighlight %}

#### kafdrop-service.yaml

{% highlight shell %}
apiVersion: v1
kind: Service
metadata:
  name: kafka-kafdrop-service
  namespace: kafdrop
  labels:
    app: kafka-kafdrop
spec:
  ports:
    - protocol: "TCP"
      port: 9000
      name: server
      nodePort: 30766
  selector:
    app: kafka-kafdrop
  type: NodePort #LoadBalancer
{% endhighlight %}

By default, the topic creation, deletion and ACL are enabled via KafDrop. If you want to disable topic creation and topic deletion, add the following in the env section of the YAML file:

{% highlight shell %}
- name: CMD_ARGS
  value: "--topic.deleteEnabled=false --topic.createEnabled=false"
{% endhighlight %}

May this is help for you who implement Kafka. 
