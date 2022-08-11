---
layout: post
title: Deploy kafka cluster on docker (Docker Compose)
---

Kafka cluster on docker using docker compose

### 1.1. Kafka cluster spec
Feature:
- 3 Broker dan 3 Zookeeper
- Authentication, PLAINTEXT, SASL_PLAINTEXT, SSL


### Creating a cluster
#### Create directory for kafka and zookeeper data
Pre-Requisites
- mkdir /data/kafka-docker/zookeeper/
- mkdir /data/kafka-docker/zookeeper2/
- mkdir /data/kafka-docker/zookeeper3/
- mkdir /data/kafka-docker/kafka1/
- mkdir /data/kafka-docker/kafka2/
- mkdir /data/kafka-docker/kafka3/
- generate truststore.jks and user.p12/keystore.jks
- create sasl_plaintext password kafka_server_jaas.conf

{% highlight shell %}
KafkaServer {
  org.apache.kafka.common.security.plain.PlainLoginModule required
  username="admin"
  password="youradminpassword"
  user_admin="adminpassword"
  user_producer="producerpassword"
  user_consumer="consumerpassword"
};
{% endhighlight %}

`docker-compose.yml`
{% highlight shell %}
version: '2'
services:
  zookeeper1:
    image: zookeeper:3.7
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_SASL_ENABLED: "false"
      ZOO_LOG4J_PROP: "INFO,ROLLINGFILE"
    restart: unless-stopped
    volumes:
     - /var/run/docker.sock:/var/run/docker.sock
     - /data/kafka-docker/zookeeper/data:/data
     - /data/kafka-docker/zookeeper/log:/logs

  zookeeper2:
    image: zookeeper:3.7
    ports:
      - "22181:2181"
    environment:
      ZOOKEEPER_SASL_ENABLED: "false"
      ZOO_LOG4J_PROP: "INFO,ROLLINGFILE"
    restart: unless-stopped
    volumes:
     - /var/run/docker.sock:/var/run/docker.sock
     - /data/kafka-docker/zookeeper2/data:/data
     - /data/kafka-docker/zookeeper2/log:/logs 

  zookeeper3:
    image: zookeeper:3.7
    ports:
      - "32181:2181"
    environment:
      ZOOKEEPER_SASL_ENABLED: "false"
      ZOO_LOG4J_PROP: "INFO,ROLLINGFILE"
    restart: unless-stopped
    volumes:
     - /var/run/docker.sock:/var/run/docker.sock
     - /data/kafka-docker/zookeeper3/data:/data
     - /data/kafka-docker/zookeeper3/log:/logs 

  kafka1:
    image: wurstmeister/kafka:2.13-2.8.1
    ports:
      - "9092:9092"
      - "9093:9093"
      - "9094:9094"
    environment:
      DOCKER_API_VERSION: 1.22
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://192.168.1.20:9092,SASL_PLAINTEXT://192.168.1.20:9093,SSL://192.168.1.20:9094
      KAFKA_ADVERTISED_HOST_NAME: 192.168.1.20
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper1:2181,zookeeper2:2181,zookeeper3:2181
      # KAFKA_JMX_OPTS: "-Djava.security.auth.login.config=/etc/kafka/kafka_server_jaas.conf"
      KAFKA_OPTS: "-Djava.security.auth.login.config=/etc/kafka/kafka_server_jaas.conf"
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092,SASL_PLAINTEXT://0.0.0.0:9093,SSL://0.0.0.0:9094
      KAFKA_MESSAGE_MAX_BYTES: 15728640
      KAFKA_SSL_KEYSTORE_LOCATION: '/certs/keystore.jks'
      KAFKA_SSL_KEYSTORE_PASSWORD: 'mypass'
      KAFKA_SSL_KEY_PASSWORD: 'mypass'
      KAFKA_SSL_TRUSTSTORE_LOCATION: '/certs/truststore.jks'
      KAFKA_SSL_TRUSTSTORE_PASSWORD: 'mypass'
      KAFKA_SSL_CLIENT_AUTH: 'none'
      KAFKA_SSL_ENDPOINT_IDENTIFICATION_ALGORITHM: " "
      KAFKA_SASL_ENABLED_MECHANISMS: PLAIN
      KAFKA_SASL_MECHANISM_INTER_BROKER_PROTOCOL: PLAIN
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'true'
      KAFKA_offsets_topic_replication_factor: 3
      # ZOOKEEPER_SASL_ENABLED: "false"
      # KAFKA_SECURITY_INTER_BROKER_PROTOCOL: SASL_PLAINTEXT
      # KAFKA_AUTHORIZER_CLASS_NAME: kafka.security.auth.SimpleAclAuthorizer
      # KAFKA_ALLOW_EVERYONE_IF_NO_ACL_FOUND: "true"
      # KAFKA_SUPER_USERS: "User:admin"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /data/kafka-docker/kafka1/data:/kafka
      - /data/kafka-docker/kafka1/log:/var/log/kafka
      - /data/kafka-docker/kafka1/private/ssl:/var/private/ssl/
      - /data/kafka-docker/kafka1/certs:/certs
      - /data/kafka-docker/kafka1/private/kafka_server_jaas.conf:/etc/kafka/kafka_server_jaas.conf
    restart: unless-stopped
    depends_on:
      - zookeeper1
      - zookeeper2
      - zookeeper3

  kafka2:
    image: wurstmeister/kafka:2.13-2.8.1
    ports:
      - "19092:19092"
      - "19093:19093"
      - "19094:19094"
    environment:
      DOCKER_API_VERSION: 1.22
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://192.168.1.20:19092,SASL_PLAINTEXT://192.168.1.20:19093,SSL://192.168.1.20:19094
      KAFKA_ADVERTISED_HOST_NAME: 192.168.1.20
      KAFKA_BROKER_ID: 2
      KAFKA_ZOOKEEPER_CONNECT: zookeeper1:2181,zookeeper2:2181,zookeeper3:2181
      # KAFKA_JMX_OPTS: "-Djava.security.auth.login.config=/etc/kafka/kafka_server_jaas.conf"
      KAFKA_OPTS: "-Djava.security.auth.login.config=/etc/kafka/kafka_server_jaas.conf"
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:19092,SASL_PLAINTEXT://0.0.0.0:19093,SSL://0.0.0.0:19094
      KAFKA_MESSAGE_MAX_BYTES: 15728640
      KAFKA_SSL_KEYSTORE_LOCATION: '/certs/keystore.jks'
      KAFKA_SSL_KEYSTORE_PASSWORD: 'mypass'
      KAFKA_SSL_KEY_PASSWORD: 'mypass'
      KAFKA_SSL_TRUSTSTORE_LOCATION: '/certs/truststore.jks'
      KAFKA_SSL_TRUSTSTORE_PASSWORD: 'mypass'
      KAFKA_SSL_CLIENT_AUTH: 'none'
      KAFKA_SSL_ENDPOINT_IDENTIFICATION_ALGORITHM: " "
      KAFKA_SASL_ENABLED_MECHANISMS: PLAIN
      KAFKA_SASL_MECHANISM_INTER_BROKER_PROTOCOL: PLAIN
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'true'
      KAFKA_offsets_topic_replication_factor: 3
      # ZOOKEEPER_SASL_ENABLED: "false"
      # KAFKA_SECURITY_INTER_BROKER_PROTOCOL: SASL_PLAINTEXT
      # KAFKA_AUTHORIZER_CLASS_NAME: kafka.security.auth.SimpleAclAuthorizer
      # KAFKA_ALLOW_EVERYONE_IF_NO_ACL_FOUND: "true"
      # KAFKA_SUPER_USERS: "User:admin"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /data/kafka-docker/kafka2/data:/kafka
      - /data/kafka-docker/kafka2/log:/var/log/kafka
      - /data/kafka-docker/kafka2/private/ssl:/var/private/ssl/
      - /data/kafka-docker/kafka2/certs:/certs
      - /data/kafka-docker/kafka2/private/kafka_server_jaas.conf:/etc/kafka/kafka_server_jaas.conf
    restart: unless-stopped
    depends_on:
      - zookeeper1
      - zookeeper2
      - zookeeper3

  kafka3:
    image: wurstmeister/kafka:2.13-2.8.1
    ports:
      - "29092:29092"
      - "29093:29093"
      - "29094:29094"
    environment:
      DOCKER_API_VERSION: 1.22
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://192.168.1.20:29092,SASL_PLAINTEXT://192.168.1.20:29093,SSL://192.168.1.20:29094
      KAFKA_ADVERTISED_HOST_NAME: 192.168.1.20
      KAFKA_BROKER_ID: 3
      KAFKA_ZOOKEEPER_CONNECT: zookeeper1:2181,zookeeper2:2181,zookeeper3:2181
      # KAFKA_JMX_OPTS: "-Djava.security.auth.login.config=/etc/kafka/kafka_server_jaas.conf"
      KAFKA_OPTS: "-Djava.security.auth.login.config=/etc/kafka/kafka_server_jaas.conf"
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:29092,SASL_PLAINTEXT://0.0.0.0:29093,SSL://0.0.0.0:29094
      KAFKA_MESSAGE_MAX_BYTES: 15728640
      KAFKA_SSL_TRUSTSTORE_LOCATION: '/certs/truststore.jks'
      KAFKA_SSL_TRUSTSTORE_PASSWORD: 'mypass'
      KAFKA_SSL_KEYSTORE_LOCATION: '/certs/keystore.jks'
      KAFKA_SSL_KEYSTORE_PASSWORD: 'mypass'
      KAFKA_SSL_KEY_PASSWORD: 'mypass'
      KAFKA_SSL_CLIENT_AUTH: 'none'
      KAFKA_SSL_ENDPOINT_IDENTIFICATION_ALGORITHM: " "
      KAFKA_SASL_ENABLED_MECHANISMS: PLAIN
      KAFKA_SASL_MECHANISM_INTER_BROKER_PROTOCOL: PLAIN
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'true'
      KAFKA_offsets_topic_replication_factor: 3
      # ZOOKEEPER_SASL_ENABLED: "false"
      # KAFKA_SECURITY_INTER_BROKER_PROTOCOL: SASL_PLAINTEXT
      # KAFKA_AUTHORIZER_CLASS_NAME: kafka.security.auth.SimpleAclAuthorizer
      # KAFKA_ALLOW_EVERYONE_IF_NO_ACL_FOUND: "true"
      # KAFKA_SUPER_USERS: "User:admin"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /data/kafka-docker/kafka3/data:/kafka
      - /data/kafka-docker/kafka3/log:/var/log/kafka
      - /data/kafka-docker/kafka3/private/ssl:/var/private/ssl/
      - /data/kafka-docker/kafka3/certs:/certs
      - /data/kafka-docker/kafka3/private/kafka_server_jaas.conf:/etc/kafka/kafka_server_jaas.conf
    restart: unless-stopped
    depends_on:
      - zookeeper1
      - zookeeper2
      - zookeeper3
{% endhighlight %}

deploy kafka cluster using command `docker-compose up -d`, monitor kafka cluster using command `docker ps` or `docker-compose ps`




May this is help for you who implement Kafka cluster on docker. 
