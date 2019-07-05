# Kafka

## Installation

To install Kafka in our cluster we use the setup provided by the community GitHub repository [Yolean/kubernetes-kafka](https://github.com/Yolean/kubernetes-kafka).

**Modifications:**
* `kafka/50kafka.yml`, `zookeeper/50pzoo.yml` and `zookeeper/51zoo.yml`: Remove `storageClassName: standard`

Install Kafka with its dependencies:
```bash
kubectl apply -f 00-namespace.yml && kubectl apply -k ./variants/dev-small/
```

## Access to Kafka

Kafka is available under `bootstrap.kafka:9092` inside the cluster.

Access to the Kafka Broker:
```bash
KAFKA_BROKER=$(kubectl get pods -l=app=kafka -n kafka -o jsonpath="{.items[*].metadata.name}")
kubectl -n kafka exec -it $KAFKA_BROKER /bin/bash
```

**Relevant commands (run inside of a Kafka broker):**

List topics:
```bash
/opt/kafka/bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
```

List consumer groups:
```bash
/opt/kafka/bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
```

Delete topic:
```bash
/opt/kafka/bin/kafka-topics.sh --bootstrap-server localhost:9092 --delete --topic transform-result
```

Consumer:
```bash
/opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --group mico --topic transform-result --from-beginning
```

Producer:
```bash
/opt/kafka/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic transform-request
```

Sample Cloud Events messages are documented [here](https://mico-docs.readthedocs.io/en/latest/messaging/cloudevents.html).

Example:
```json
{"specversion":"0.2","type":"io.github.ust.mico.result","source":"/router","id":"A234-1234-1234","time":"2019-05-08T17:31:00Z","contenttype":"application/json","data":{"key":"value"}}
```

## kafkacat

kafkacat is a command line utility that you can use to test and debug Apache Kafka® deployments. You can use kafkacat to produce, consume, and list topic and partition information for Kafka. Described as "netcat for Kafka", it is a swiss-army knife of tools for inspecting and creating data in Kafka.

It is similar to Kafka Console Producer (`kafka-console-producer`) and Kafka Console Consumer (`kafka-console-consumer`), but even more powerful.

kafkacat is an open-source utility, available at [https://github.com/edenhill/kafkacat](https://github.com/edenhill/kafkacat). It is not included in Confluent Platform.

Deploy *kafkacat* to Kubernetes:
```bash
kubectl apply -f 01-test-namespace.yml && kubectl apply -f ./kafka/test/kafkacat.yml
```

Access to kafkacat:
```bash
KAFKACAT=$(kubectl get pods -n test-kafka -l test-target=kafka-client-kafkacat -o=jsonpath={.items..metadata.name})
kubectl -n test-kafka exec -it $KAFKACAT /bin/bash
```

Listing topics:
```bash
kubectl -n test-kafka exec -it $KAFKACAT -- kafkacat -b bootstrap.kafka:9092 -L
```

Consuming topic 'test-kafkacat':
```bash
kubectl -n test-kafka exec -it $KAFKACAT -- kafkacat -b bootstrap.kafka:9092 -C -t test-kafkacat
```

Producing message to topic 'test-kafkacat':
```bash
kubectl -n test-kafka exec -it $KAFKACAT -- kafkacat -b bootstrap.kafka:9092 -P -t test-kafkacat
```

---

Consuming topic 'transform-result' with group id 'mico':
```bash
kubectl -n test-kafka exec -it $KAFKACAT -- kafkacat -b bootstrap.kafka:9092 -G mico -C -t transform-result
```

Producing message to topic 'transform-request' with group id 'mico':
```bash
kubectl -n test-kafka exec -it $KAFKACAT -- kafkacat -b bootstrap.kafka:9092 -G mico -P -t transform-request
```

## Kafka Manager

Install Kafka Manager (Yahoo):
```bash
kubectl apply -f ./yahoo-kafka-manager
```

Access Kafka Manager via port forwarding:
```bash
kubectl port-forward svc/kafka-manager -n kafka 30969:80
```

Open [http://localhost:30969](http://localhost:30969/).

Configure Kafka Manager so it's able to access ZooKeeper:
1. Add Cluster
2. Cluster Name: `MICO-Kafka`
3. Cluster Zookeeper Hosts: `zookeeper.kafka:2181`
4. brokerViewThreadPoolSize: 2
5. offsetCacheThreadPoolSize: 2
6. kafkaAdminClientThreadPoolSize: 2
