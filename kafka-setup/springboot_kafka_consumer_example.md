# 📘 Spring Boot Kafka Consumer Example

This project demonstrates how to consume messages from Apache Kafka using:

- Spring Boot
- Spring Kafka
- Kafka Consumer
- REST Microservice Architecture

---

# 🚀 Project Setup

## 📦 Required Dependencies

Add the following dependencies in `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webmvc</artifactId>
</dependency>

<!-- Kafka -->
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

---

# ⚙️ application.properties

Create:

```txt
src/main/resources/application.properties
```

Add:

```properties
spring.application.name=notification-service

server.port=8082

spring.kafka.bootstrap-servers=localhost:9092

spring.kafka.consumer.group-id=notification-group
spring.kafka.consumer.auto-offset-reset=earliest

spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer

spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JacksonJsonDeserializer

spring.kafka.consumer.properties.spring.json.trusted.packages=*

spring.kafka.consumer.properties.spring.json.value.default.type=com.notification_service.dto.OrderEvent
```

---

# 📁 Project Structure

```txt
src
 └── main
      ├── java
      │    └── com.notification_service
      │           ├── dto
      │           │      └── OrderEvent.java
      │           ├── kafka
      │           │      ├── KafkaConsumerConfig.java
      │           │      └── NotificationConsumer.java
      │           └── NotificationServiceApplication.java
      │
      └── resources
             └── application.properties
```

---

# 📦 OrderEvent DTO

Create:

```txt
src/main/java/com/notification_service/dto/OrderEvent.java
```

```java
package com.notification_service.dto;

public class OrderEvent {

    private Long orderId;
    private String email;
    private String status;

    public Long getOrderId() {
        return orderId;
    }

    public void setOrderId(Long orderId) {
        this.orderId = orderId;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getStatus() {
        return status;
    }

    public void setStatus(String status) {
        this.status = status;
    }
}
```

---

# ⚙️ Kafka Consumer Configuration

Create:

```txt
src/main/java/com/notification_service/kafka/KafkaConsumerConfig.java
```

```java
package com.notification_service.kafka;

import com.notification_service.dto.OrderEvent;
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.core.*;
import org.springframework.kafka.support.serializer.JacksonJsonDeserializer;

import java.util.HashMap;
import java.util.Map;

@Configuration
public class KafkaConsumerConfig {

    @Bean
    public ConsumerFactory<String, OrderEvent> consumerFactory() {

        Map<String, Object> config = new HashMap<>();

        config.put(
                ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG,
                "localhost:9092"
        );

        config.put(
                ConsumerConfig.GROUP_ID_CONFIG,
                "notification-group"
        );

        JacksonJsonDeserializer<OrderEvent> deserializer =
                new JacksonJsonDeserializer<>(OrderEvent.class);

        deserializer.addTrustedPackages("*");

        return new DefaultKafkaConsumerFactory<>(
                config,
                new StringDeserializer(),
                deserializer
        );
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, OrderEvent>
    kafkaListenerContainerFactory() {

        ConcurrentKafkaListenerContainerFactory<String, OrderEvent> factory =
                new ConcurrentKafkaListenerContainerFactory<>();

        factory.setConsumerFactory(consumerFactory());

        return factory;
    }
}
```

---

# 📩 Kafka Consumer Service

Create:

```txt
src/main/java/com/notification_service/kafka/NotificationConsumer.java
```

```java
package com.notification_service.kafka;

import com.notification_service.dto.OrderEvent;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Service;

@Service
public class NotificationConsumer {

    @KafkaListener(
            topics = "order-events",
            groupId = "notification-group"
    )
    public void consume(OrderEvent event) {

        System.out.println("Event Received : " + event.getEmail());

        String subject = "Order Status Update";

        String body =
                "Order ID : " + event.getOrderId() +
                        "\\nStatus : " + event.getStatus();

        System.out.println(subject);
        System.out.println(body);
    }
}
```

---

# ▶️ Run the Application

Run the Spring Boot application:

```bash
mvn spring-boot:run
```

Application will start on:

```txt
http://localhost:8082
```

---

# 📨 How It Works

1. Kafka Producer publishes message to:

```txt
order-events
```

2. Notification Consumer listens to the topic.

3. Consumer receives JSON message automatically.

4. Event data is processed inside:

```java
consume(OrderEvent event)
```

---

# 📮 Sample Kafka Message

```json
{
  "orderId": 101,
  "email": "test@gmail.com",
  "status": "CREATED"
}
```

---

# 📩 Expected Console Output

```txt
Event Received : test@gmail.com

Order Status Update

Order ID : 101
Status : CREATED
```

---

# 🎯 Kafka Message Flow

```txt
Order Service → Kafka Topic(order-events) → Notification Service
```

---

# ✅ Summary

✔ Spring Boot Kafka Consumer Setup  
✔ KafkaListener Integration  
✔ JSON Message Deserialization  
✔ Kafka Consumer Group Configuration  
✔ Event-Driven Communication  

---

# 🔥 Technologies Used

- Java
- Spring Boot
- Apache Kafka
- Kafka Consumer
- Maven
