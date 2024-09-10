# 💥 Docker Compose Configuration Explanation

This document explains the configuration of your Docker Compose file, which sets up a Kafka ecosystem with Zookeeper and Kafka services. Each service is defined with its respective image, hostname, environment variables, ports, health checks, and volume mappings. The network and volumes used by the services are also outlined.

👉 [NOTION LIKE](https://rogue-consonant-00a.notion.site/Deploy-Apache-Kafka-Latest-fc3224b06fea4270b4649b9331ce0a53?pvs=4)

# ⭕ Services

### 1️⃣ Zookeeper Service
👉 The Zookeeper service is responsible for managing the configuration of the Kafka broker(s) and providing synchronization between them.

- Image: Uses `confluentinc/cp-zookeeper:7.4.6`, which is a Confluent Zookeeper image compatible with the Confluent Kafka setup.
  
- Hostname & Container Name: The Zookeeper service is named "zookeeper" for both the hostname and container name, ensuring easy communication between services.

- Ports:
  - `2181:2181`: The container's port 2181 (Zookeeper client port) is mapped to the host's port 2181.

- Environment Variables:
  - `ZOOKEEPER_CLIENT_PORT`: Port used by clients to connect to Zookeeper.
  - `ZOOKEEPER_TICK_TIME`: The basic time unit in milliseconds used by Zookeeper for its heartbeat and session timeouts.
  - `ALLOW_ANONYMOUS_LOGIN`: Allows anonymous logins to Zookeeper.

- Healthcheck:
  - A health check is configured using `nc` (netcat) to verify that Zookeeper is listening on port 2181. It checks the connection every 10 seconds, retries 3 times, and times out after 5 seconds if Zookeeper is not ready.

- Volumes:
  - `zookeeper-data:/var/lib/zookeeper/data`: Maps a persistent volume for storing Zookeeper data.

- Networks:
  - Zookeeper is connected to the `kafka-network`, ensuring it can communicate with Kafka.

### 2️⃣ Kafka Service
👉 Kafka is the core messaging system that handles producing and consuming messages.

- Image: Uses `confluentinc/cp-kafka:7.4.6`, which is a Confluent Kafka image compatible with the specified Zookeeper version.

- Hostname & Container Name: The Kafka service is named "kafka" for both the hostname and container name.

- Depends on:
  - The Kafka service depends on Zookeeper being healthy before starting, as defined by the `service_healthy` condition.

- Ports:
  - `9092:9092`: Maps the container's port 9092 (Kafka listener port) to the host's port 9092.

- Environment Variables:
  - `KAFKA_BROKER_ID`: Sets the unique broker ID for this Kafka instance.
  - `KAFKA_ZOOKEEPER_CONNECT`: Specifies the connection details to Zookeeper (`zookeeper:2181`), which is the hostname of the Zookeeper service.
  - `KAFKA_ADVERTISED_LISTENERS`: Defines the listener address for external communication. Replace `<server-ip>` with the actual IP of the server where the service is running.
  - `KAFKA_LISTENERS`: Configures Kafka to listen on all interfaces (`0.0.0.0`) for port 9092.
  - `KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR`: Sets the replication factor for the `__consumer_offsets` topic. Since there is only one broker, this is set to `1`.
  - `KAFKA_CONFLUENT_SUPPORT_METRICS_ENABLE`: Disables Confluent support metrics.
  - `KAFKA_MESSAGE_MAX_BYTES`: Sets the maximum message size to 2GB.
  - `KAFKA_REPLICA_FETCH_MAX_BYTES`: Defines the maximum fetch size for replicated messages, also set to 2GB.
  - `KAFKA_REPLICA_MAX_LOG_SIZE`: Sets the maximum log size for replicated messages to 2GB.
  - `KAFKA_AUTO_CREATE_TOPICS_ENABLE`: Automatically creates topics when needed.
  - `KAFKA_LOG_RETENTION_HOURS`: Logs are retained for 168 hours (7 days).
  - `KAFKA_LOG_RETENTION_BYTES`: Logs are retained up to 10GB of disk space.

- Healthcheck:
  - Similar to Zookeeper, Kafka has a health check that uses `nc` (netcat) to verify that Kafka is listening on port 9092. It checks every 10 seconds with 3 retries and starts after a 30-second delay.

- Volumes:
  - `kafka-data:/var/lib/kafka/data`: Maps a persistent volume for Kafka data storage.

- Networks:
  - Kafka is also connected to the `kafka-network`, allowing it to communicate with Zookeeper.

# ⭕ Networks

### ⁉️ kafka-network
👉 A custom bridge network named `kafka-network` is created to allow Zookeeper and Kafka to communicate internally. By using a Docker bridge network, services can refer to each other by their container names (e.g., `zookeeper`, `kafka`) rather than needing explicit IP addresses.

# ⭕ Volumes

### ⁉️ zookeeper-data
👉 A volume named `zookeeper-data` is created and mounted to `/var/lib/zookeeper/data` in the Zookeeper container to persist Zookeeper data across container restarts.

### ⁉️kafka-data
👉 A volume named `kafka-data` is created and mounted to `/var/lib/kafka/data` in the Kafka container to persist Kafka data across container restarts.

# ⭕ Summary
This Docker Compose configuration sets up a Kafka ecosystem with Zookeeper and Kafka. Zookeeper provides service coordination, while Kafka handles message brokering. Both services use persistent volumes to ensure data persistence, and health checks are configured for service reliability.
