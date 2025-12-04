# Kafka Debezium MongoDB to Parquet Configuration

## Complete Setup Guide

---

## 1. Docker Compose File

**File:** `docker-compose.yaml`

```yaml
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    container_name: debezium-zookeeper
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
      ZOOKEEPER_MIN_SESSION_TIMEOUT: 4000      # Minimum 4 seconds
      ZOOKEEPER_MAX_SESSION_TIMEOUT: 20000     # Maximum 20 seconds 
    volumes:
      - ./zookeeper-data/data:/var/lib/zookeeper/data      # ✅ Persist data
      - ./zookeeper-data/log:/var/lib/zookeeper/log       # ✅ Persist logs
    

  kafka1:
    image: confluentinc/cp-kafka:7.5.0
    container_name: kafka-broker-1
    ports:
      - "9092:9092"
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_INTERNAL:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092,PLAINTEXT_INTERNAL://kafka1:29092
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092,PLAINTEXT_INTERNAL://0.0.0.0:29092
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT_INTERNAL
      
      # Replication & Durability
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      KAFKA_MIN_INSYNC_REPLICAS: 2
      
      # Performance Tuning
      KAFKA_NUM_NETWORK_THREADS: 16
      KAFKA_NUM_IO_THREADS: 16
      KAFKA_SOCKET_SEND_BUFFER_BYTES: "2097152"
      KAFKA_SOCKET_RECEIVE_BUFFER_BYTES: "2097152"
      KAFKA_SOCKET_REQUEST_MAX_BYTES: "20971520"
      
      # Compression
      KAFKA_COMPRESSION_TYPE: snappy
      KAFKA_LOG_CLEANUP_POLICY: delete
      KAFKA_HEAP_OPTS: "-Xmx6g -Xms6g"  
      
      # Log Settings
      KAFKA_LOG_RETENTION_HOURS: 168
      KAFKA_LOG_SEGMENT_BYTES: "2147483647"
      KAFKA_LOG_RETENTION_BYTES: "107374182400"
      KAFKA_LOG_RETENTION_MS: "604800000"
      KAFKA_LOG_FLUSH_INTERVAL_MESSAGES: "1000000"
      KAFKA_LOG_FLUSH_INTERVAL_MS: "1000"
      
      # Auto Create Topics
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
      KAFKA_NUM_PARTITIONS: 10
    volumes:
      - ./kafka1_data:/var/lib/kafka/data
    healthcheck:
      test: ["CMD", "kafka-broker-api-versions", "--bootstrap-server", "localhost:9092"]
      interval: 10s
      timeout: 10s
      retries: 10
      start_period: 60s
    restart: on-failure
  kafka2:
    image: confluentinc/cp-kafka:7.5.0
    container_name: kafka-broker-2
    ports:
      - "9093:9093"
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 2
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_INTERNAL:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9093,PLAINTEXT_INTERNAL://kafka2:29093
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9093,PLAINTEXT_INTERNAL://0.0.0.0:29093
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT_INTERNAL
      
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      KAFKA_MIN_INSYNC_REPLICAS: 2
      
            # Performance Tuning
      KAFKA_NUM_NETWORK_THREADS: 16
      KAFKA_NUM_IO_THREADS: 16
      KAFKA_SOCKET_SEND_BUFFER_BYTES: "2097152"
      KAFKA_SOCKET_RECEIVE_BUFFER_BYTES: "2097152"
      KAFKA_SOCKET_REQUEST_MAX_BYTES: "20971520"
      
      KAFKA_COMPRESSION_TYPE: snappy
      KAFKA_HEAP_OPTS: "-Xmx6g -Xms6g"
      
      # Log Settings
      KAFKA_LOG_RETENTION_HOURS: 168
      KAFKA_LOG_SEGMENT_BYTES: "2147483647"
      KAFKA_LOG_RETENTION_BYTES: "107374182400"
      KAFKA_LOG_RETENTION_MS: "604800000"
      KAFKA_LOG_FLUSH_INTERVAL_MESSAGES: "1000000"
      KAFKA_LOG_FLUSH_INTERVAL_MS: "1000"
      KAFKA_LOG_CLEANUP_POLICY: delete

      
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
      KAFKA_NUM_PARTITIONS: 10
    volumes:
      - ./kafka2_data:/var/lib/kafka/data
    healthcheck:
      test: ["CMD", "kafka-broker-api-versions", "--bootstrap-server", "localhost:9093"]
      interval: 10s
      timeout: 10s
      retries: 10
      start_period: 60s
    restart: on-failure

  kafka3:
    image: confluentinc/cp-kafka:7.5.0
    container_name: kafka-broker-3
    ports:
      - "9094:9094"
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 3
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_INTERNAL:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9094,PLAINTEXT_INTERNAL://kafka3:29094
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9094,PLAINTEXT_INTERNAL://0.0.0.0:29094
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT_INTERNAL
      
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      KAFKA_MIN_INSYNC_REPLICAS: 2
      
      # Performance Tuning
      KAFKA_NUM_NETWORK_THREADS: 16
      KAFKA_NUM_IO_THREADS: 16
      KAFKA_SOCKET_SEND_BUFFER_BYTES: "2097152"
      KAFKA_SOCKET_RECEIVE_BUFFER_BYTES: "2097152"
      KAFKA_SOCKET_REQUEST_MAX_BYTES: "20971520"
      
      KAFKA_COMPRESSION_TYPE: snappy
      KAFKA_HEAP_OPTS: "-Xmx6g -Xms6g"
      
      # Log Settings
      KAFKA_LOG_RETENTION_HOURS: 168
      KAFKA_LOG_SEGMENT_BYTES: "2147483647"
      KAFKA_LOG_RETENTION_BYTES: "107374182400"
      KAFKA_LOG_RETENTION_MS: "604800000"
      KAFKA_LOG_FLUSH_INTERVAL_MESSAGES: "1000000"
      KAFKA_LOG_FLUSH_INTERVAL_MS: "1000"
      KAFKA_LOG_CLEANUP_POLICY: delete
      
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
      KAFKA_NUM_PARTITIONS: 10
    volumes:
      - ./kafka3_data:/var/lib/kafka/data
    healthcheck:
      test: ["CMD", "kafka-broker-api-versions", "--bootstrap-server", "localhost:9094"]
      interval: 10s
      timeout: 10s
      retries: 10
      start_period: 60s
    restart: on-failure


  kafka4:
    image: confluentinc/cp-kafka:7.5.0
    container_name: kafka-broker-4
    ports:
      - "9095:9095"
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 4
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_INTERNAL:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9095,PLAINTEXT_INTERNAL://kafka4:29095
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9095,PLAINTEXT_INTERNAL://0.0.0.0:29095
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT_INTERNAL

      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      KAFKA_MIN_INSYNC_REPLICAS: 2

      # Performance Tuning (same as others)
      KAFKA_NUM_NETWORK_THREADS: 16
      KAFKA_NUM_IO_THREADS: 16
      KAFKA_SOCKET_SEND_BUFFER_BYTES: "2097152"
      KAFKA_SOCKET_RECEIVE_BUFFER_BYTES: "2097152"
      KAFKA_SOCKET_REQUEST_MAX_BYTES: "20971520"

      KAFKA_COMPRESSION_TYPE: snappy
      KAFKA_HEAP_OPTS: "-Xmx6g -Xms6g"

      # Log Settings
      KAFKA_LOG_RETENTION_HOURS: 168
      KAFKA_LOG_SEGMENT_BYTES: "2147483647"
      KAFKA_LOG_RETENTION_BYTES: "107374182400"
      KAFKA_LOG_RETENTION_MS: "604800000"
      KAFKA_LOG_FLUSH_INTERVAL_MESSAGES: "1000000"
      KAFKA_LOG_FLUSH_INTERVAL_MS: "1000"
      KAFKA_LOG_CLEANUP_POLICY: delete

      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
      KAFKA_NUM_PARTITIONS: 10
    volumes:
      - ./kafka4_data:/var/lib/kafka/data
    healthcheck:
      test: ["CMD", "kafka-broker-api-versions", "--bootstrap-server", "localhost:9095"]
      interval: 10s
      timeout: 10s
      retries: 10
      start_period: 60s
    restart: on-failure
  kafka5:
    image: confluentinc/cp-kafka:7.5.0
    container_name: kafka-broker-5
    ports:
      - "9096:9096"
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 5
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_INTERNAL:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9096,PLAINTEXT_INTERNAL://kafka5:29096
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9096,PLAINTEXT_INTERNAL://0.0.0.0:29096
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT_INTERNAL

      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      KAFKA_MIN_INSYNC_REPLICAS: 2

      KAFKA_NUM_NETWORK_THREADS: 16
      KAFKA_NUM_IO_THREADS: 16
      KAFKA_SOCKET_SEND_BUFFER_BYTES: "2097152"
      KAFKA_SOCKET_RECEIVE_BUFFER_BYTES: "2097152"
      KAFKA_SOCKET_REQUEST_MAX_BYTES: "20971520"

      KAFKA_COMPRESSION_TYPE: snappy
      KAFKA_HEAP_OPTS: "-Xmx6g -Xms6g"

      KAFKA_LOG_RETENTION_HOURS: 168
      KAFKA_LOG_SEGMENT_BYTES: "2147483647"
      KAFKA_LOG_RETENTION_BYTES: "107374182400"
      KAFKA_LOG_RETENTION_MS: "604800000"
      KAFKA_LOG_FLUSH_INTERVAL_MESSAGES: "1000000"
      KAFKA_LOG_FLUSH_INTERVAL_MS: "1000"
      KAFKA_LOG_CLEANUP_POLICY: delete

      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
      KAFKA_NUM_PARTITIONS: 10
    volumes:
      - ./kafka5_data:/var/lib/kafka/data
    healthcheck:
      test: ["CMD", "kafka-broker-api-versions", "--bootstrap-server", "localhost:9096"]
      interval: 10s
      timeout: 10s
      retries: 10
      start_period: 60s
    restart: on-failure

  kafka6:
    image: confluentinc/cp-kafka:7.5.0
    container_name: kafka-broker-6
    ports:
      - "9097:9097"
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 6
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_INTERNAL:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9097,PLAINTEXT_INTERNAL://kafka6:29097
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9097,PLAINTEXT_INTERNAL://0.0.0.0:29097
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT_INTERNAL

      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      KAFKA_MIN_INSYNC_REPLICAS: 2

      KAFKA_NUM_NETWORK_THREADS: 16
      KAFKA_NUM_IO_THREADS: 16
      KAFKA_SOCKET_SEND_BUFFER_BYTES: "2097152"
      KAFKA_SOCKET_RECEIVE_BUFFER_BYTES: "2097152"
      KAFKA_SOCKET_REQUEST_MAX_BYTES: "20971520"

      KAFKA_COMPRESSION_TYPE: snappy
      KAFKA_HEAP_OPTS: "-Xmx6g -Xms6g"

      KAFKA_LOG_RETENTION_HOURS: 168
      KAFKA_LOG_SEGMENT_BYTES: "2147483647"
      KAFKA_LOG_RETENTION_BYTES: "107374182400"
      KAFKA_LOG_RETENTION_MS: "604800000"
      KAFKA_LOG_FLUSH_INTERVAL_MESSAGES: "1000000"
      KAFKA_LOG_FLUSH_INTERVAL_MS: "1000"
      KAFKA_LOG_CLEANUP_POLICY: delete

      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
      KAFKA_NUM_PARTITIONS: 10
    volumes:
      - ./kafka6_data:/var/lib/kafka/data
    healthcheck:
      test: ["CMD", "kafka-broker-api-versions", "--bootstrap-server", "localhost:9097"]
      interval: 10s
      timeout: 10s
      retries: 10
      start_period: 60s
    restart: on-failure




  connect:
    image: confluentinc/cp-kafka-connect:7.5.0
    container_name: debezium-connect
    ports:
      - "8083:8083"
    depends_on:
      schema-registry:
        condition: service_healthy
    environment:
      # CONNECT_BOOTSTRAP_SERVERS: 'debezium-kafka:9092'
      CONNECT_BOOTSTRAP_SERVERS: 'kafka1:29092,kafka2:29093,kafka3:29094,kafka4:29095,kafka5:29096,kafka6:29097'
      CONNECT_REST_ADVERTISED_HOST_NAME: 'localhost'
      CONNECT_GROUP_ID: debezium
      CONNECT_CONFIG_STORAGE_TOPIC: connect-configs
      CONNECT_OFFSET_STORAGE_TOPIC: connect-offsets
      CONNECT_STATUS_STORAGE_TOPIC: connect-status
      CONNECT_KEY_CONVERTER: org.apache.kafka.connect.json.JsonConverter
      CONNECT_VALUE_CONVERTER: org.apache.kafka.connect.json.JsonConverter
      CONNECT_INTERNAL_KEY_CONVERTER: org.apache.kafka.connect.json.JsonConverter
      CONNECT_INTERNAL_VALUE_CONVERTER: org.apache.kafka.connect.json.JsonConverter
      CONNECT_REST_PORT: 8083
      CONNECT_PLUGIN_PATH: /usr/share/java,/usr/share/confluent-hub-components
      CONNECT_CONFIG_STORAGE_REPLICATION_FACTOR: 3
      CONNECT_OFFSET_STORAGE_REPLICATION_FACTOR: 3
      CONNECT_STATUS_STORAGE_REPLICATION_FACTOR: 3

      # performance
      # 🚀 PERFORMANCE OPTIMIZATIONS (NEW/UPGRADED)
      CONNECT_OFFSET_FLUSH_INTERVAL_MS: "1000"           # 10x faster offset commits
      CONNECT_PRODUCER_COMPRESSION_TYPE: snappy
      CONNECT_PRODUCER_BATCH_SIZE: "65536"               # 2x bigger batches
      CONNECT_PRODUCER_LINGER_MS: "20"                   # 2x batching delay
      CONNECT_PRODUCER_MAX_REQUEST_SIZE: "2097152"       # 2x message size
      CONNECT_CONSUMER_MAX_POLL_RECORDS: "10000"         # 20x bigger polls!
      CONNECT_CONSUMER_FETCH_MAX_BYTES: "10485760"       # 10MB Kafka fetches
      CONNECT_CONSUMER_FETCH_MIN_BYTES: "1048576"        # 1MB min fetch
      CONNECT_CONSUMER_MAX_POLL_INTERVAL_MS: "600000"
      CONNECT_TASK_POLL_TIMEOUT_MS: "100"  

      # 🚀 JVM Heap (CRITICAL for high throughput)
      KAFKA_HEAP_OPTS: "-Xms4g -Xmx4g"    

      AWS_ACCESS_KEY_ID: # you AWS ACCESS ID
      AWS_SECRET_ACCESS_KEY: # your AWS SECRET KEY
      AWS_DEFAULT_REGION: # your REGION
    volumes:
    - ./plugin/debezium-connector-mongodb:/usr/share/confluent-hub-components/debezium-connector-mongodb
    restart: on-failure  # ✅ ADD THIS - Auto-restart when brokers become healthy

    

  schema-registry:
    image: confluentinc/cp-schema-registry:7.5.0
    container_name: schema-registry
    depends_on:
      kafka1:
        condition: service_healthy
      kafka2:
        condition: service_healthy
      kafka3:
        condition: service_healthy
      kafka4:
        condition: service_healthy
      kafka5:
        condition: service_healthy
      kafka6:
        condition: service_healthy
    ports:
      - "8081:8081"
    environment:
      SCHEMA_REGISTRY_HOST_NAME: schema-registry
      # SCHEMA_REGISTRY_KAFKASTORE_BOOTSTRAP_SERVERS: 'debezium-kafka:9092'
      SCHEMA_REGISTRY_KAFKASTORE_BOOTSTRAP_SERVERS: 'kafka1:29092,kafka2:29093,kafka3:29094,kafka4:29095,kafka5:29096,kafka6:29097'
      SCHEMA_REGISTRY_KAFKASTORE_TOPIC_REPLICATION_FACTOR: 3
      SCHEMA_REGISTRY_KAFKASTORE_INIT_TIMEOUT_MS: 90000
      SCHEMA_REGISTRY_LISTENERS: http://0.0.0.0:8081
    networks:
      - default
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8081/"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
    restart: on-failure  # ✅ ADD THIS - Auto-restart when brokers become healthy

  minio:
    image: minio/minio:latest
    container_name: debezium-minio
    ports:
      - "9000:9000"
      - "9001:9001"
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    command: server /data --console-address ":9001"
    volumes:
      - ./minio_data:/data


```

---

## 2. Debezium MongoDB Source Connector

**Configuration Command:**

```bash
curl -X POST -H "Content-Type: application/json" http://localhost:8083/connectors \
-d '{
  "name": "debezium-mongo-optimized",
  "config": {
    "connector.class": "io.debezium.connector.mongodb.MongoDbConnector",
    "tasks.max": "1",
    "mongodb.connection.string": "YOUR MONGO CONNECTION STRING",
    "collection.include.list": "gst_staging.sales_financial_data", ### your db.collection
    "snapshot.mode": "initial",
    "snapshot.fetch.size": "10240",
    "capture.mode": "change_streams_update_full",
    "key.converter": "io.confluent.connect.avro.AvroConverter",
    "key.converter.schema.registry.url": "http://schema-registry:8081",
    "value.converter": "io.confluent.connect.avro.AvroConverter",
    "value.converter.schema.registry.url": "http://schema-registry:8081",
    "transforms": "unwrap",
    "transforms.unwrap.type": "io.debezium.connector.mongodb.transforms.ExtractNewDocumentState",
    "transforms.unwrap.add.fields": "op,ts_ms",
    "max.queue.size": "100000",
    "max.queue.size.in.bytes": "104857600",
    "max.batch.size": "8192",
    "poll.interval.ms": "100",
    "producer.compression.type": "snappy",
    "producer.linger.ms": "20",
    "producer.batch.size": "131072",
    "producer.buffer.memory": "134217728",
    "producer.max.request.size": "4194304",
    "producer.acks": "1",
    "producer.max.in.flight.requests.per.connection": "5"
  }
}'

```

**Key Settings:**
- `snapshot.mode: initial` - Captures existing data + future changes
- `ExtractNewDocumentState` - Flattens MongoDB documents
- `add.fields: op,ts_ms` - Adds operation type and timestamp
- Avro converter with Schema Registry for structured data

---

## 3. S3 Parquet Sink Connector

**Configuration Command:**

```bash
curl -X POST -H "Content-Type: application/json" http://localhost:8083/connectors \
-d '{
  "name": "s3-sink-optimized",
  "config": {
    "connector.class": "io.confluent.connect.s3.S3SinkConnector",
    
    "tasks.max": "18", ##18 paritions so 18 tasks for parallel execution
    "topics": "mongo.gst_staging.sales_financial_data",
    
    "s3.bucket.name": "dwh-dev-data-bucket", ##bucket name
    "s3.region": "ap-south-1", ## aws region 
    "topics.dir": "sales_financial_data", ## your collection
    
    "format.class": "io.confluent.connect.s3.format.parquet.ParquetFormat",
    "storage.class": "io.confluent.connect.s3.storage.S3Storage",
    "parquet.codec": "snappy",
    
    "transforms": "unwrap,flatten",
    "transforms.unwrap.type": "io.debezium.transforms.ExtractNewRecordState",
    "transforms.unwrap.add.fields": "op,ts_ms",
    "transforms.flatten.type": "org.apache.kafka.connect.transforms.Flatten$Value",
    "transforms.flatten.delimiter": "_",
    
    "key.converter": "io.confluent.connect.avro.AvroConverter",
    "key.converter.schema.registry.url": "http://schema-registry:8081",
    "value.converter": "io.confluent.connect.avro.AvroConverter",
    "value.converter.schema.registry.url": "http://schema-registry:8081",
    
    "flush.size": "50000",
    "rotate.interval.ms": "60000",
    "rotate.schedule.interval.ms": "60000",
    
    "partitioner.class": "io.confluent.connect.storage.partitioner.TimeBasedPartitioner",
    "path.format": "'\''year'\''=YYYY/'\''month'\''=MM/'\''day'\''=dd/'\''hour'\''=HH",
    "partition.duration.ms": "3600000",
    "timestamp.extractor": "Record",
    "locale": "en-US",
    "timezone": "Asia/Kolkata",
    
    "schema.compatibility": "NONE",
    
    "consumer.max.poll.records": "10000",
    "consumer.max.poll.interval.ms": "600000",
    "consumer.fetch.min.bytes": "1048576",
    "consumer.fetch.max.wait.ms": "500",
    "consumer.max.partition.fetch.bytes": "10485760",
    
    "s3.part.size": "10485760",
    "s3.object.tagging": "true",
    "behavior.on.null.values": "ignore",
    "errors.tolerance": "all",
    "errors.log.enable": "true",
    "errors.log.include.messages": "true"
  }
}'
```

**Key Settings:**
- `format.class: ParquetFormat` - Writes Parquet files
- `flush.size: 10000` - Flushes after 10k records
- `rotate.interval.ms: 300000` - Rotates files every 5 minutes
- `parquet.codec: snappy` - Compression codec

---

## 4. Startup Commands

### Initial Setup

```bash
# 1. Start all services
docker-compose up -d

# 2. Wait for Kafka Connect to be ready
sleep 120

# 3. Create Kafka internal topics with correct cleanup policy
docker exec kafka-broker-1 kafka-topics --bootstrap-server kafka1:29092 \
  --create --topic mongo.gst_staging.sales_financial_data \
  --partitions 18 \
  --replication-factor 3 \
  --config min.insync.replicas=2 \
  --config retention.ms=604800000 \
  --config compression.type=snappy



# 4. Create Debezium MongoDB connector from above


# 5. Wait for snapshot to complete (5-15 minutes depending on data size)
sleep 600

# 6. Create S3 Parquet sink connector from above

```

---

## 5. Monitoring Commands

### Check Connector Status

```bash
# Check Debezium connector status
curl http://localhost:8083/connectors/debezium-mongo-optimized/status | jq

# Check S3 sink connector status
curl http://localhost:8083/connectors/s3-sink-optimized/status | jq

# List all connectors
curl http://localhost:8083/connectors | jq
```

### Monitor Data Flow

```bash
# Check Kafka topic offsets
docker exec -it debezium-kafka kafka-run-class kafka.tools.GetOffsetShell \
  --broker-list localhost:9092 \
  --topic atlas1.newtestdb.test_collection

# Check schema registry subjects
curl http://localhost:8081/subjects | jq

# Monitor S3 sink consumer group lag
docker exec -it debezium-kafka kafka-consumer-groups \
  --bootstrap-server localhost:9092 \
  --describe \
  --group connect-s3-parquet-sink-final
```

### View Sample Data

```bash
# Consume 1 Avro message from topic
docker exec -it debezium-kafka kafka-avro-console-consumer \ ##{make suitable changes here}
  --bootstrap-server localhost:9092 \
  --topic  \
  --from-beginning \
  --max-messages 1 \
  --property schema.registry.url=http://schema-registry:8081


```

---

## 6. Management Commands

### Delete Connectors

```bash
curl -X DELETE http://localhost:8083/connectors/debezium-mongo-optimized
curl -X DELETE http://localhost:8083/connectors/s3-sink-optimized
```

### Stop Services

```bash
docker-compose down -v
```

### Clean Data




---

## 7. Key Features

✅ **MongoDB Change Data Capture** - Real-time data sync via Debezium  
✅ **Avro Serialization** - Schema-managed data format  
✅ **Schema Registry Integration** - Centralized schema management  
✅ **Flattened Documents** - MongoDB nested documents become flat Parquet columns  
✅ **Parquet Output** - Optimized columnar format for analytics  
✅ **S3/MinIO Storage** - Cloud-native data lake storage  
✅ **Operation Tracking** - `__op` field (c/u/d/r), `__ts_ms` timestamp  
✅ **Snapshot Mode** - Captures existing + future data  

---

## 8. Data Flow

```
MongoDB Database
    ↓
Debezium MongoDB Connector (snapshot + CDC)
    ↓
Kafka Topic: atlas1.newtestdb.test_collection (Avro format)
    ↓
Schema Registry: Stores Avro schemas
    ↓
S3 Parquet Sink Connector
    ↓
Parquet Files in S3: sales_financial_data/mongo.gst_staging.sales_financial_data/year=2025/month=12/day=04/hour=11/
```

---

## 9. Troubleshooting

### Snapshot Not Starting
- Delete `connect-offsets` topic
- Ensure `cleanup.policy=compact` on all Connect topics
- Restart Connect service

### Avro Deserialization Errors
- Check if topic has old JSON data (delete and recreate)
- Verify Schema Registry is running
- Ensure converters match (Avro source → Avro sink)

### S3 Sink Not Writing Files
- Check S3 credentials in AWS CLI
- Verify bucket exists and is writable
- Check sink connector status: ``

### Connector Not Producing Data
- Verify MongoDB connection string is correct
- Check if MongoDB is accessible from Connect container
- Review connector logs: `docker logs debezium-connect`



my commands...........




##install s3 sink


docker exec -it debezium-connect confluent-hub install --no-prompt confluentinc/kafka-connect-s3:latest

#mongo-connector already installed

via command line and put in plugin folder
