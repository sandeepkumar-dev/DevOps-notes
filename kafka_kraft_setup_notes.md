# 📘 Apache Kafka 4 Setup Guide (KRaft Mode)

This guide explains how to set up **Apache Kafka 4.x** in **KRaft mode** on Windows without ZooKeeper.

---

# 🚀 Kafka 4 Installation & Setup (KRaft Mode)

## 📥 Step 1: Download Kafka

Download Kafka from the official website:

https://kafka.apache.org/downloads

Extract Kafka to:

```txt
C:\kafka\kafka_2.13-4.2.0
```

> ⚠️ Important: Avoid spaces in the folder path.

Example ❌

```txt
C:\Program Files\Kafka
```

Example ✅

```txt
C:\kafka\kafka_2.13-4.2.0
```

---

# ⚙️ Step 2: Configure Kafka

Open the file:

```txt
config/server.properties
```

Replace the configuration with:

```properties
process.roles=broker,controller
node.id=1

controller.quorum.voters=1@localhost:9093

listeners=PLAINTEXT://localhost:9092,CONTROLLER://localhost:9093
advertised.listeners=PLAINTEXT://localhost:9092

inter.broker.listener.name=PLAINTEXT
controller.listener.names=CONTROLLER

listener.security.protocol.map=PLAINTEXT:PLAINTEXT,CONTROLLER:PLAINTEXT

log.dirs=./logs
num.partitions=1

offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
```

---

# ▶️ Step 3: Start Kafka

## 1️⃣ Generate Cluster ID

Run the following command inside the Kafka folder:

```cmd
bin\windows\kafka-storage.bat random-uuid
```

Example output:

```txt
hd9d4_4uTAW-Wtg13LnwUQ
```

---

## 2️⃣ Format Kafka Storage

Use the generated Cluster ID:

```cmd
bin\windows\kafka-storage.bat format -t hd9d4_4uTAW-Wtg13LnwUQ -c config\server.properties
```

---

## 3️⃣ Start Kafka Server

```cmd
bin\windows\kafka-server-start.bat config\server.properties
```

If Kafka starts successfully, the server will begin running on:

```txt
localhost:9092
```

---



# 🧵 Step 4: Create a Kafka Topic

Create a topic named `order-events`:

```cmd
bin\windows\kafka-topics.bat --create ^
--topic order-events ^
--bootstrap-server localhost:9092 ^
--partitions 1 ^
--replication-factor 1
```

Expected output:

```txt
Created topic order-events.
```

---

# ✅ Step 5: Verify Kafka Installation

List all topics:

```cmd
bin\windows\kafka-topics.bat --list --bootstrap-server localhost:9092
```

---

# ✉️ Step 6: Test Kafka Producer

Start the producer:

```cmd
bin\windows\kafka-console-producer.bat --topic order-events --bootstrap-server localhost:9092
```

Now type messages:

```txt
Hello Kafka
Order Created
Payment Successful
```

---

# 📩 Step 7: Test Kafka Consumer

Open another terminal and run:

```cmd
bin\windows\kafka-console-consumer.bat --topic order-events --from-beginning --bootstrap-server localhost:9092
```

You should see all produced messages displayed in the consumer console.

---

# 📌 Useful Kafka Commands

## 🔹 List Topics

```cmd
bin\windows\kafka-topics.bat --list --bootstrap-server localhost:9092
```

## 🔹 Describe Topic

```cmd
bin\windows\kafka-topics.bat --describe --topic order-events --bootstrap-server localhost:9092
```

## 🔹 Delete Topic

```cmd
bin\windows\kafka-topics.bat --delete --topic order-events --bootstrap-server localhost:9092
```

---

# 🛠 Troubleshooting

## ❌ Port Already in Use

If port `9092` is busy:

```cmd
netstat -ano | findstr 9092
```

Kill the process:

```cmd
taskkill /PID <PID> /F
```

---

## ❌ Java Not Found

Verify Java installation:

```cmd
java -version
```

Kafka 4 requires Java 17+.

---

# 📚 Summary

✅ Download Kafka  
✅ Configure `server.properties`  
✅ Generate Cluster ID  
✅ Format Storage  
✅ Start Kafka Server  
✅ Create Topics  
✅ Test Producer & Consumer  

---

# 🎯 Kafka Architecture (KRaft Mode)

```txt
Producer → Kafka Broker → Consumer
```

Kafka 4 uses **KRaft mode**, which removes the need for ZooKeeper and simplifies setup 🚀
