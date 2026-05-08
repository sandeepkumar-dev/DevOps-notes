# 📘 Spring Boot Kafka Producer Example

This project demonstrates how to publish messages to Apache Kafka using:

- Spring Boot
- Spring Kafka
- REST API
- Kafka Producer

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
spring.application.name=orderserviceexample

server.port=8081

spring.kafka.bootstrap-servers=localhost:9092

spring.kafka.producer.properties.spring.json.add.type.headers=false
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JacksonJsonSerializer
```

---

# 📁 Project Structure

```txt
src
 └── main
      ├── java
      │    └── com.orderserviceexample
      │           ├── config
      │           │      └── KafkaProducerConfig.java
      │           ├── controller
      │           │      └── OrderController.java
      │           ├── dto
      │           │      └── OrderEvent.java
      │           ├── kafka
      │           │      └── OrderProducer.java
      │           └── OrderserviceexampleApplication.java
      │
      └── resources
             └── application.properties
```

---

# ⚙️ Kafka Producer Configuration

Create:

```txt
src/main/java/com/orderserviceexample/config/KafkaProducerConfig.java
```

```java
package com.orderserviceexample.config;

import com.orderserviceexample.dto.OrderEvent;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.serialization.StringSerializer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.core.ProducerFactory;
import org.springframework.kafka.support.serializer.JacksonJsonSerializer;

import java.util.HashMap;
import java.util.Map;

@Configuration
public class KafkaProducerConfig {

    @Bean
    public ProducerFactory<String, OrderEvent> producerFactory() {

        Map<String, Object> config = new HashMap<>();

        config.put(
                ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,
                "localhost:9092"
        );

        config.put(
                ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
                StringSerializer.class
        );

        config.put(
                ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,
                JacksonJsonSerializer.class
        );

        // IMPORTANT FIX
        config.put(
                JacksonJsonSerializer.ADD_TYPE_INFO_HEADERS,
                false
        );

        return new DefaultKafkaProducerFactory<>(config);
    }

    @Bean
    public KafkaTemplate<String, OrderEvent> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }
}
```

---

# 📦 OrderEvent DTO

Create:

```txt
src/main/java/com/orderserviceexample/dto/OrderEvent.java
```

```java
package com.orderserviceexample.dto;

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

# 📤 Kafka Producer Service

Create:

```txt
src/main/java/com/orderserviceexample/kafka/OrderProducer.java
```

```java
package com.orderserviceexample.kafka;

import com.orderserviceexample.dto.OrderEvent;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Service;

@Service
public class OrderProducer {

    private final KafkaTemplate<String, OrderEvent> kafkaTemplate;

    private static final String TOPIC = "order-events";

    public OrderProducer(KafkaTemplate<String, OrderEvent> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    public void sendOrderEvent(OrderEvent event) {

        kafkaTemplate.send(TOPIC, event);

        System.out.println("Event Sent : " + event);
    }
}
```

---

# 🌐 REST Controller

Create:

```txt
src/main/java/com/orderserviceexample/controller/OrderController.java
```

```java
package com.orderserviceexample.controller;

import com.orderserviceexample.dto.OrderEvent;
import com.orderserviceexample.kafka.OrderProducer;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/orders")
public class OrderController {

    private final OrderProducer producer;

    public OrderController(OrderProducer producer) {
        this.producer = producer;
    }

    @PostMapping
    public String placeOrder(@RequestBody OrderEvent event) {

        producer.sendOrderEvent(event);

        return "Order Event Published Successfully";
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
http://localhost:8081
```

---

# 📮 Test API Using Postman

## Endpoint

```http
POST http://localhost:8081/orders
```

## Request Body

```json
{
  "orderId": 101,
  "email": "test@gmail.com",
  "status": "CREATED"
}
```

## Response

```txt
Order Event Published Successfully
```

---

# 📩 Verify Kafka Consumer

Start Kafka consumer:

```cmd
bin\windows\kafka-console-consumer.bat ^
--topic order-events ^
--from-beginning ^
--bootstrap-server localhost:9092
```

You should receive:

```json
{"orderId":101,"email":"test@gmail.com","status":"CREATED"}
```

---

# 🎯 Flow Diagram

```txt
REST API → Kafka Producer → Kafka Topic(order-events) → Consumer
```

---

# ✅ Summary

✔ Spring Boot Kafka Producer Setup  
✔ KafkaTemplate Configuration  
✔ REST API Integration  
✔ JSON Message Publishing  
✔ Kafka Topic Communication  

---

# 🔥 Technologies Used

- Java
- Spring Boot
- Apache Kafka
- REST API
- Maven
