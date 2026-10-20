# kafka 4.3.1 
Essential Guide to Using Kafka

## 1. Generate a Cluster UUID and Format Storage
```Bash
# Generate a random UUID
❯ KAFKA_CLUSTER_ID="$(./bin/kafka-storage.sh random-uuid)"
```

## 2. Format Storage 
```Bash
# Format the storage using the KRaft configuration file
# -t or --cluster-id : specifiles the unique ID for the Kafka cluster.
# -c or --config
❯ ./bin/kafka-storage.sh format -t $KAFKA_CLUSTER_ID -c ./config/server.properties --standalone
```

## 3. Start the Kafka Sever
```Bash
❯ ./bin/kafka-server-start.sh ./config/server.properties
```

## 4. Check Existing Topics
 List all available topics in the Kafka cluster
* --list : tells Kafka to return a list of all existing topics.
* --bootstrap-server : specifies the Kafka broker address to connect to.
```Bash
❯ ./bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

## 5. Create a New Topic
 Create a new topic using the standard naming convention
* --create : specifies that you want to create a new topic.
* --topic : defines the name of the topic.
* --partitions : the number of partitions (rooms) for the topic (1 for local testing).
* --replication-factor : the number of copies of data (1 for standalone mode).
```Bash
❯ ./bin/kafka-topics.sh --create --topic catalog.product.registered --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
```

## 6. Verify Topic Details (Describe)
 Inspect the detailed configuration and status of a specific topic
* --describe : shows the details of the topic (partitions, leader, replicas, ISR).
* --topic : specifies the name of the topic you want to inspect.
```Bash
❯ ./bin/kafka-topics.sh --describe --topic catalog.product.registered --bootstrap-server localhost:9092

❯ ./bin/kafka-topics.sh --describe --topic catalog.product.registered --bootstrap-server localhost:9092
Topic: catalog.product.registered	TopicId: Pvb_brNNTmSCPBAoI0BM1Q	PartitionCount: 1	ReplicationFactor: 1	Configs: min.insync.replicas=1,segment.bytes=1073741824
	Topic: catalog.product.registered	Partition: 0	Leader: 1	Replicas: 1	Isr: 1	Elr: 	LastKnownElr:
```

## 7. Start a Producer to Send Messages

Start the Kafka Console Producer to manually type and send (publish) messages to your topic. 
 Start the console producer to publish messages
* --bootstrap-server : specifies the Kafka broker address (replaces the deprecated --broker-list).
* --topic : specifies the target topic where messages will be sent.
```bash

./bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic catalog.product.registered
```

## 8. Start a Consumer to Read Messages

Start the Kafka Console Consumer to read (subscribe to) messages from your topic. 
It is highly recommended to run this command in a **new terminal window** alongside your Producer, so you can watch messages arrive in real-time.

Start the console consumer to fetch and display messages
* --bootstrap-server : specifies the Kafka broker address to connect to.
* --topic : specifies the target topic to read data from.
* --from-beginning : tells the consumer to read all historical messages from the start of the topic.
```bash

./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic catalog.product.registered --from-beginning
```
