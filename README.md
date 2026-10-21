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

❯ ./bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic catalog.product.registered
```

## 8. Start a Consumer to Read Messages

Start the Kafka Console Consumer to read (subscribe to) messages from your topic. 
It is highly recommended to run this command in a **new terminal window** alongside your Producer, so you can watch messages arrive in real-time.

Start the console consumer to fetch and display messages
* --bootstrap-server : specifies the Kafka broker address to connect to.
* --topic : specifies the target topic to read data from.
* --from-beginning : tells the consumer to read all historical messages from the start of the topic.
```bash

❯ ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic catalog.product.registered --from-beginning
```


## 9. Start a Consumer with a Consumer Group

To utilize Kafka's powerful load balancing and offset tracking features, you should run your consumers as part of a **Consumer Group**. 

Start a consumer and assign it to a specific consumer group
* --group : specifies the name of the consumer group (e.g., my-first-group).
```bash

❯ ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic catalog.product.registered --group my-first-group
```


## 10. Download and Install MariaDB Connector Plugins

Kafka Connect is already included in your Kafka installation. However, to connect to MariaDB, you must download a **Connector Plugin** and the **MariaDB JDBC Driver**.

**Step 1: Download the required files**
1. **JDBC Connector Plugin:** Go to the [Confluent Hub](https://www.confluent.io/hub/confluentinc/kafka-connect-jdbc) and download the **latest version** of the `kafka-connect-jdbc` plugin (provided as a `.zip` file).
2. **MariaDB JDBC Driver:** Go to the [MariaDB official website](https://mariadb.com/downloads/connectors/connectors-data-access/java8-connector/) and download the latest MariaDB Java Client (`.jar` file).

**Step 2: Create a plugins directory**
```bash
# Create a dedicated folder for your plugins
❯ mkdir -p ~/kafka/plugins
```
**Step 3: Extract and move the files
1.Unzip the confluentinc-kafka-connect-jdbc-x.x.x.zip file.
2.Inside the unzipped folder, find the lib directory. Copy all the .jar files from this lib directory into your ~/kafka/plugins folder.
3.Move the downloaded mariadb-java-client-x.x.x.jar file directly into the ~/kafka/plugins folder as well.

Step 4: Update the Connect configuration
Open ./config/connect-standalone.properties in your text editor and specify the plugin path at the bottom:

Properties
# Set the plugin path to the directory you created
plugin.path=/Users/kafka/plugins

## 11. Start Kafka Connect (Distributed Mode)

In production and most tutorials, Kafka Connect is run in **Distributed Mode**. You start the Connect worker first, and then add connectors dynamically via REST API (Postman/curl).

**Step 1: Configure the distributed worker**
Open `./config/connect-distributed.properties` and add your plugin path at the bottom:
```properties
# Add this to the very bottom
plugin.path=/Users/kafka/plugins
```

** Step 2: Start the Distributed Worker
Run the worker. It will start an API server on port 8083 listening for your connector configurations.

```Bash
❯ ./bin/connect-distributed.sh ./config/connect-distributed.properties
```

## 11-1. Create Target Table in MariaDB
```sql
CREATE TABLE users (
    id BIGINT AUTO_INCREMENT,
    email VARCHAR(50) NOT NULL UNIQUE,
    name VARCHAR(50) NOT NULL,
    encrypted_pwd VARCHAR(255) NOT NULL,
    user_id VARCHAR(255) NOT NULL UNIQUE,
    PRIMARY KEY (id)
);
```

## 12. Create MariaDB Source Connector via REST API
With the Connect worker running, use curl (or an API client like Bruno/Postman) to send a JSON configuration to Connect's REST API (Port 8083). This JSON tells the worker to start pulling data from MariaDB.

Run this in a Bruno or Postman
**Step 1: Create a New Request in Bruno**
1. Bruno를 열고 새로운 Request를 생성합니다.
2. **Name**: `Create MariaDB Source Connector` (원하는 이름 지정)
3. **Method**: `POST`
4. **URL**: `http://localhost:8083/connectors`

**Step 2: Configure the JSON Body**
1. Request 설정 화면에서 **Body** 탭을 클릭합니다.
2. 포맷을 **JSON**으로 선택합니다. (Bruno가 자동으로 `Content-Type: application/json` 헤더를 설정해 줍니다.)
3. 아래의 JSON 설정값을 그대로 복사하여 붙여넣습니다. (DB 정보와 테이블명은 본인 환경에 맞게 수정하세요.)

```json
{
  "name": "my-source-connect",
  "config": {
    "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
    "connection.url": "jdbc:mariadb://192.168.0.4:3306/mydb",
    "connection.user": "root",
    "connection.password": "test1357",
    "mode": "incrementing",
    "incrementing.column.name": "id",
    "catalog.pattern": "mydb",
    "table.whitelist": "mydb.users",
    "topic.prefix": "my_topic_",
    "tasks.max": "1"
  }
}
```

## 13. Verify Connector and Task Status
Endpoint: GET http://localhost:8083/connectors/my-source-connect/status

## 14. Insert Test Data
```sql
INSERT INTO users (email, name, encrypted_pwd, user_id) 
VALUES ('test@example.com', 'Soochang', 'password123', 'user01');
```

## 15. Verify Kafka Topic and Consume Data

```Bash
# 1. 자동 생성된 토픽 목록 확인 (my_topic_mydb.users 확인)
./bin/kafka-topics.sh --bootstrap-server localhost:9092 --list

# 2. Console Consumer를 통해 실시간 데이터 유입 확인
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic my_topic_users --from-beginning
```












