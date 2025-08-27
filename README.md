# Kafka Consumer Guide

This README documents key concepts, benefits, working model, consumer logs, and troubleshooting for **Apache Kafka consumers**.  
It is compiled from multiple discussion points for easy reference.

---

## 📑 Table of Contents
1. [Consumer Behavior Logs](#1-consumer-behavior-logs)
2. [Common Kafka Errors](#2-common-kafka-errors)
3. [Consumer application Log execution flow](#3-consumer-application-log-execution-flow)
4. [Back to Top](#-back-to-top)

---

## 1. Consumer Behavior Logs
Below are typical Kafka **consumer-related logs** and their meaning:

- **Consumer group rebalancing started**  
  → Kafka is redistributing partitions among consumers.  

- **Partition assigned to consumer**  
  → The consumer got a specific partition for processing.  

- **Polling messages from broker**  
  → Consumer is actively fetching records from Kafka.  

- **Committed offset successfully**  
  → Acknowledges that messages were processed and marked as safe.  

- **Rebalancing completed**  
  → Consumer group rebalancing finished; stable state achieved.  

- **Consumer heartbeat**  
  → Consumer is alive and communicating with Kafka.  

- **Consumer disconnected / session expired**  
  → Consumer left group, crashed, or lost connection.  

---

## 2. Common Kafka Errors
- **OffsetOutOfRangeException**  
  → Offset doesn’t exist (either too old or deleted).  
  🔹 Fix: Reset offset (`earliest` / `latest`).  

- **Rebalance In Progress**  
  → Happens when consumers join/leave group.  
  🔹 Fix: Ensure proper `session.timeout.ms` and `max.poll.interval.ms`.  

- **Consumer lag increasing**  
  → Messages are produced faster than consumed.  
  🔹 Fix: Scale consumer group or optimize processing.  

- **Serialization errors**  
  → Message format mismatch.  
  🔹 Fix: Use the same serializer/deserializer in producer & consumer.  

---

## 3. Consumer application Log execution flow

1. **Initializing Spring DispatcherServlet**

   * Log:
     `Initializing Spring DispatcherServlet 'dispatcherServlet'`
   * Explanation:
     Spring Boot is setting up the main servlet (`dispatcherServlet`) that handles all HTTP requests in the application. This is the entry point where REST APIs are routed to the correct controllers.

2. **Servlet Initialization Completed**

   * Log:
     `Completed initialization in 4 ms`

   * Explanation:
     Confirms that the servlet was successfully set up and ready to handle web requests in just **4 milliseconds**, which means the web layer is initialized correctly.
3. **Kafka Producer Configuration Loaded**
   * Log:
     `ProducerConfig values: ...`
   * Explanation:
     Kafka producer configuration is printed, showing all key parameters:
     * **bootstrap.servers=localhost:9092** → Producer connects to Kafka broker on local machine.
     * **acks=-1** → Ensures strong durability (leader + all replicas must acknowledge).
     * **enable.idempotence=true** → Prevents duplicate messages (exactly-once semantics).
     * **retries=2147483647** → Infinite retries for sending messages until success.
     * **key.serializer=StringSerializer & value.serializer=JsonSerializer** → Defines how keys and values are serialized before sending to Kafka.

4. **Idempotent Producer Created**
   * Log:
     `[Producer clientId=producer-1] Instantiated an idempotent producer.`
   * Explanation:
     Confirms Kafka producer is created with **idempotence enabled**, ensuring **no duplicate messages** are stored even in retries or failures.

5. **Kafka Version and Start Info**
   * Logs:
     ```
     Kafka version: 3.4.1  
     Kafka commitId: 8a516edc2755df89  
     Kafka startTimeMs: 1756278640572

     ```

   * Explanation:
     Kafka client library version is **3.4.1**. The commitId uniquely identifies the Kafka build version. The startTimeMs shows when the Kafka client started.

6. **Metadata Reset for Partitions**

   * Logs:

     ```
     Resetting the last seen epoch of partition vmc-demo-0 ...
     Resetting the last seen epoch of partition vmc-demo-4 ...

     ...

     ```

   * Explanation:
     Producer refreshed metadata for topic **`vmc-demo`** partitions (0 to 4).
     * "epoch reset" means Kafka detected a new **topicId** (unique identifier for a topic in Kafka) and reset its metadata for correct routing of messages.
     * This ensures the producer sends messages to the correct leader partition.



7. **Cluster ID Assigned**
   * Log:
     `Cluster ID: lOEwIfjZRfSHBvyTp4mtfA`
   * Explanation:
     Producer successfully connected to the Kafka cluster and identified it with a unique **Cluster ID**. This confirms the producer is now part of the cluster communication.


8. **Transaction Manager Initialized**
   * Log:
     `[Producer clientId=producer-1] ProducerId set to 2000 with epoch 0`
   * Explanation:
     Kafka transaction manager assigned **ProducerId=2000** and **epoch=0**, which is crucial for idempotent and transactional messaging. This ensures reliable message delivery and consistency.

9. **Message Successfully Sent**

   * Logs:

     ```
     Sent message=[vamshi : 0] with offset=[2]  
     Sent message=[vamshi : 1] with offset=[3]

     ```
   * Explanation:
     * Two messages (`vamshi : 0` and `vamshi : 1`) were successfully published to Kafka.
     * The assigned **offsets (2 and 3)** confirm the exact position of the messages in the Kafka partition log.
     * This ensures message ordering and allows consumers to read from a specific offset for replay.

---

✅ So overall:

* Spring Boot initialized web layer (`dispatcherServlet`).
* Kafka producer was created with **safe & idempotent configuration**.
* Producer connected to the Kafka cluster, refreshed metadata, and got assigned a **ProducerId**.
* Finally, messages were successfully sent to the topic with specific offsets.


---


 

---

## 🔼 Back to Top
[Go to Table of Contents](#-table-of-contents)
