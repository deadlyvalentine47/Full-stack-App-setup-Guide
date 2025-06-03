# Phase 2: Core Configuration and Communication for E-Commerce Backend Microservices

## Objective
Set up each microservice (User, Product, Cart, Order, Payment) in separate folders, configure them with `application.properties`, and enable communication via REST (through API Gateway) and Kafka for asynchronous events. We’ll build on Phase 1, ensuring services are independent, runnable, and connected.

## 1. Core Configuration
- **What We’re Doing**: Create separate folders for each microservice, configure connections to databases, Kafka, and Eureka, and start them individually.
- **Why**: Separate folders keep microservices modular and independent, making development and scaling easier.

### Step 1.1: Set Up Separate Folders
- **Why**: Each microservice gets its own project folder for isolation and clarity.
- **How**:
  1. Open your project directory from Phase 1 (e.g., `C:\ecommerce` or `~/ecommerce`).
  2. Create five new folders:
     - `user-service`
     - `product-service`
     - `cart-service`
     - `order-service`
     - `payment-service`
  3. For each service, use Spring Initializr to create a project:
     - Open browser, go to https://start.spring.io.
     - Settings (same for all):
       - Project: Maven
       - Language: Java
       - Spring Boot: Latest stable (e.g., 3.2.x)
       - Group: `com.ecommerce`
       - Java: 17
       - Dependencies: Spring Web, Spring Data JPA, Spring Data MongoDB, Spring for Apache Kafka, Spring Security, Spring Boot Starter OAuth2 Resource Server, Spring Boot Actuator, Spring Cloud Netflix Eureka Client
     - Artifact (change for each):
       - `user-service` for User
       - `product-service` for Product
       - `cart-service` for Cart
       - `order-service` for Order
       - `payment-service` for Payment
     - Click “Generate”, download, and unzip each into its folder (e.g., unzip `user-service.zip` into `user-service`).
  4. Open IntelliJ IDEA:
     - Click “File” > “Open” for each folder (e.g., open `C:\ecommerce\user-service`).
     - Load each as a separate project.
- **Result**: Five independent folders with Spring Boot projects.

### Step 1.2: Configure Keycloak for OAuth2
- **Why**: Keycloak provides OAuth2 for secure authentication across all services.
- **How**:
  1. Start Keycloak:
     - In terminal, run:
       ```
       docker run -d -p 8080:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:22.0
       ```
     - This runs Keycloak on port 8080.
  2. Access Keycloak:
     - Open browser, go to http://localhost:8080.
     - Log in: Username `admin`, Password `admin`.
  3. Set up:
     - Click “Create realm”, name it `ecommerce`.
     - In `ecommerce` realm, click “Clients” > “Create client”.
     - Client ID: `ecommerce-app`, Client type: OpenID Connect, Save.
     - Settings: Enable “Client authentication”, set “Valid redirect URIs” to `*` (for now), Save.
     - Click “Users” > “Add user”.
     - Username: `testuser`, Save.
     - Set password: Click “Credentials” tab, set password to `test123`, disable “Temporary”, Save.
     - Click “Roles” > “Create role”, name it `user`, Save.
     - Assign: Go to “Users”, select `testuser`, click “Role mapping”, assign `user` role.
  4. Note: Issuer URI is `http://localhost:8080/realms/ecommerce`—we’ll use this for security.
- **Result**: Keycloak ready for OAuth2 authentication.

### Step 1.3: Configure Each Microservice
- **Why**: Each service needs its own `application.properties` to connect to databases, Kafka, Eureka, and secure with OAuth2.
- **How**:
  1. **User Service**:
     - Open `user-service/src/main/resources/application.properties`.
     - Add:
       ```
       server.port=8081
       spring.datasource.url=jdbc:postgresql://localhost:5432/ecommerce
       spring.datasource.username=postgres
       spring.datasource.password=your_password
       spring.datasource.driver-class-name=org.postgresql.Driver
       spring.jpa.hibernate.ddl-auto=update
       spring.jpa.show-sql=true
       spring.redis.host=localhost
       spring.redis.port=6379
       spring.kafka.bootstrap-servers=localhost:9092
       spring.kafka.consumer.group-id=ecommerce-group
       spring.kafka.consumer.auto-offset-reset=earliest
       spring.cloud.discovery.enabled=true
       eureka.client.service-url.defaultZone=http://localhost:8761/eureka/
       eureka.client.register-with-eureka=true
       eureka.client.fetch-registry=true
       management.endpoints.web.exposure.include=health,metrics,info
       management.endpoint.health.show-details=always
       spring.security.oauth2.resourceserver.jwt.issuer-uri=http://localhost:8080/realms/ecommerce
       spring.application.name=user-service
       ```
     - **Explain**:
       - `server.port=8081`: Runs on port 8081.
       - `spring.datasource.*`: Connects to PostgreSQL for user data.
       - `spring.redis.*`: For session caching.
       - `spring.kafka.*`: Links to Kafka for events.
       - `eureka.client.*`: Registers with Eureka.
       - `management.endpoints.*`: Exposes Actuator for monitoring.
       - `spring.security.oauth2.*`: Secures with Keycloak.
  2. **Product Service**:
     - Open `product-service/src/main/resources/application.properties`.
     - Add:
       ```
       server.port=8082
       spring.data.mongodb.uri=mongodb://localhost:27017/ecommerce
       spring.data.mongodb.database=ecommerce
       spring.redis.host=localhost
       spring.redis.port=6379
       spring.kafka.bootstrap-servers=localhost:9092
       spring.kafka.consumer.group-id=ecommerce-group
       spring.kafka.consumer.auto-offset-reset=earliest
       spring.cloud.discovery.enabled=true
       eureka.client.service-url.defaultZone=http://localhost:8761/eureka/
       eureka.client.register-with-eureka=true
       eureka.client.fetch-registry=true
       management.endpoints.web.exposure.include=health,metrics,info
       management.endpoint.health.show-details=always
       spring.security.oauth2.resourceserver.jwt.issuer-uri=http://localhost:8080/realms/ecommerce
       spring.application.name=product-service
       ```
     - **Explain**: Uses MongoDB for products, Redis for caching, others similar.
  3. **Cart Service**:
     - Open `cart-service/src/main/resources/application.properties`.
     - Add:
       ```
       server.port=8083
       spring.redis.host=localhost
       spring.redis.port=6379
       spring.kafka.bootstrap-servers=localhost:9092
       spring.kafka.consumer.group-id=ecommerce-group
       spring.kafka.consumer.auto-offset-reset=earliest
       spring.cloud.discovery.enabled=true
       eureka.client.service-url.defaultZone=http://localhost:8761/eureka/
       eureka.client.register-with-eureka=true
       eureka.client.fetch-registry=true
       management.endpoints.web.exposure.include=health,metrics,info
       management.endpoint.health.show-details=always
       spring.security.oauth2.resourceserver.jwt.issuer-uri=http://localhost:8080/realms/ecommerce
       spring.application.name=cart-service
       ```
     - **Explain**: Relies on Redis for cart storage, Kafka for updates.
  4. **Order Service**:
     - Open `order-service/src/main/resources/application.properties`.
     - Add:
       ```
       server.port=8084
       spring.datasource.url=jdbc:postgresql://localhost:5432/ecommerce
       spring.datasource.username=postgres
       spring.datasource.password=your_password
       spring.datasource.driver-class-name=org.postgresql.Driver
       spring.jpa.hibernate.ddl-auto=update
       spring.jpa.show-sql=true
       spring.kafka.bootstrap-servers=localhost:9092
       spring.kafka.consumer.group-id=ecommerce-group
       spring.kafka.consumer.auto-offset-reset=earliest
       spring.cloud.discovery.enabled=true
       eureka.client.service-url.defaultZone=http://localhost:8761/eureka/
       eureka.client.register-with-eureka=true
       eureka.client.fetch-registry=true
       management.endpoints.web.exposure.include=health,metrics,info
       management.endpoint.health.show-details=always
       spring.security.oauth2.resourceserver.jwt.issuer-uri=http://localhost:8080/realms/ecommerce
       spring.application.name=order-service
       ```
     - **Explain**: Uses PostgreSQL for orders, Kafka for events.
  5. **Payment Service**:
     - Open `payment-service/src/main/resources/application.properties`.
     - Add:
       ```
       server.port=8085
       spring.datasource.url=jdbc:postgresql://localhost:5432/ecommerce
       spring.datasource.username=postgres
       spring.datasource.password=your_password
       spring.datasource.driver-class-name=org.postgresql.Driver
       spring.jpa.hibernate.ddl-auto=update
       spring.jpa.show-sql=true
       spring.kafka.bootstrap-servers=localhost:9092
       spring.kafka.consumer.group-id=ecommerce-group
       spring.kafka.consumer.auto-offset-reset=earliest
       spring.cloud.discovery.enabled=true
       eureka.client.service-url.defaultZone=http://localhost:8761/eureka/
       eureka.client.register-with-eureka=true
       eureka.client.fetch-registry=true
       management.endpoints.web.exposure.include=health,metrics,info
       management.endpoint.health.show-details=always
       spring.security.oauth2.resourceserver.jwt.issuer-uri=http://localhost:8080/realms/ecommerce
       spring.application.name=payment-service
       ```
     - **Explain**: PostgreSQL for payment records, Kafka for updates.
- **Result**: Each service configured in its own folder.

### Step 1.4: Start Each Microservice
- **Why**: Run each service to ensure they connect to Eureka and are alive.
- **How**:
  1. Ensure Docker is running (PostgreSQL, MongoDB, Redis, Kafka from Phase 1).
  2. Start Eureka Server:
     - In terminal, go to `eureka-server` folder (e.g., `cd C:\ecommerce\eureka-server`).
     - Run:
       ```
       mvn spring-boot:run
       ```
     - Check: Open http://localhost:8761, see the dashboard.
  3. For each microservice:
     - Open a new terminal tab/window.
     - Navigate to folder:
       - `cd C:\ecommerce\user-service`
       - `cd C:\ecommerce\product-service`
       - `cd C:\ecommerce\cart-service`
       - `cd C:\ecommerce\order-service`
       - `cd C:\ecommerce\payment-service`
     - Run each:
       ```
       mvn spring-boot:run
       ```
     - Check: Open http://localhost:8761, see services (e.g., “USER-SERVICE”, “PRODUCT-SERVICE”) under “Instances”.
  4. Start API Gateway:
     - In terminal, go to `cd C:\ecommerce\api-gateway`.
     - Run:
       ```
       mvn spring-boot:run
       ```
     - Check: Runs on port 8080.
- **Result**: All services running and registered with Eureka.

## 2. Communication Between Microservices
- **What We’re Doing**: Enable services to talk via REST (synchronous) through API Gateway and Kafka (asynchronous) for events.
- **Why**: REST handles direct requests (e.g., get user data), Kafka handles events (e.g., order created notifies payment).

### Step 2.1: REST Communication via API Gateway
- **Why**: API Gateway routes requests to services, centralizing access and security.
- **How**:
  1. Verify API Gateway setup (from Phase 1):
     - Check `api-gateway/src/main/resources/application.properties`:
       ```
       server.port=8080
       spring.cloud.gateway.routes[0].id=user-service
       spring.cloud.gateway.routes[0].uri=lb://user-service
       spring.cloud.gateway.routes[0].predicates[0]=Path=/api/users/**
       spring.cloud.gateway.routes[1].id=product-service
       spring.cloud.gateway.routes[1].uri=lb://product-service
       spring.cloud.gateway.routes[1].predicates[0]=Path=/api/products/**
       spring.cloud.gateway.routes[2].id=cart-service
       spring.cloud.gateway.routes[2].uri=lb://cart-service
       spring.cloud.gateway.routes[2].predicates[0]=Path=/api/cart/**
       spring.cloud.gateway.routes[3].id=order-service
       spring.cloud.gateway.routes[3].uri=lb://order-service
       spring.cloud.gateway.routes[3].predicates[0]=Path=/api/orders/**
       spring.cloud.gateway.routes[4].id=payment-service
       spring.cloud.gateway.routes[4].uri=lb://payment-service
       spring.cloud.gateway.routes[4].predicates[0]=Path=/api/payments/**
       eureka.client.service-url.defaultZone=http://localhost:8761/eureka/
       spring.application.name=api-gateway
       ```
  2. Test communication:
     - Ensure all services and Eureka are running.
     - Use a tool like Postman or curl.
     - Example: Call User Service:
       - In Postman, GET http://localhost:8080/api/users (won’t work yet—no APIs, but checks routing).
     - Later, we’ll add APIs (e.g., GET /api/users/{id}) in Phase 2.
  3. Note: API Gateway uses `lb://` to find services via Eureka.
- **Result**: Services reachable via API Gateway (e.g., http://localhost:8080/api/users).

### Step 2.2: Kafka Communication for Events
- **Why**: Kafka lets services send async messages (e.g., Order Service tells Payment Service “new order created”).
- **How**:
  1. Create Kafka Topics:
     - In terminal, go to Kafka folder (where `docker-compose.yml` is).
     - Run:
       ```
       docker exec -it kafka kafka-topics --create --topic order-events --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
       ```
       ```
       docker exec -it kafka kafka-topics --create --topic payment-events --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
       ```
     - **Explain**: Creates `order-events` and `payment-events` topics for messages.
  2. Add Kafka Config to Services:
     - Already in `application.properties` (e.g., `spring.kafka.bootstrap-servers=localhost:9092`).
  3. Test Basic Kafka (we’ll expand in next phase):
     - **Order Service Producer**:
       - Open `order-service/src/main/java/com/ecommerce/orderservice`.
       - Create `KafkaProducerConfig.java`:
         ```java
         import org.springframework.context.annotation.Bean;
         import org.springframework.context.annotation.Configuration;
         import org.springframework.kafka.core.KafkaTemplate;
         import org.springframework.kafka.core.ProducerFactory;
         import org.springframework.kafka.core.DefaultKafkaProducerFactory;
         import org.apache.kafka.clients.producer.ProducerConfig;
         import java.util.HashMap;
         import java.util.Map;
         
         @Configuration
         public class KafkaProducerConfig {
             @Bean
             public KafkaTemplate<String, String> kafkaTemplate() {
                 Map<String, Object> props = new HashMap<>();
                 props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
                 props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
                 props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
                 return new KafkaTemplate<>(new DefaultKafkaProducerFactory<>(props));
             }
         }
         ```
     - **Payment Service Consumer**:
       - Open `payment-service/src/main/java/com/ecommerce/paymentservice`.
       - Create `KafkaConsumerConfig.java`:
         ```java
         import org.springframework.context.annotation.Configuration;
         import org.springframework.kafka.annotation.KafkaListener;
         
         @Configuration
         public class KafkaConsumerConfig {
             @KafkaListener(topics = "order-events", groupId = "ecommerce-group")
             public void listenOrderEvents(String message) {
                 System.out.println("Payment Service received: " + message);
             }
         }
         ```
     - **Test**:
       - Open `order-service/src/main/java/com/ecommerce/orderservice`.
       - Edit `OrderServiceApplication.java`:
         ```java
         import org.springframework.boot.SpringApplication;
         import org.springframework.boot.autoconfigure.SpringBootApplication;
         import org.springframework.context.annotation.Bean;
         import org.springframework.kafka.core.KafkaTemplate;
         
         @SpringBootApplication
         public class OrderServiceApplication {
             public static void main(String[] args) {
                 var context = SpringApplication.run(OrderServiceApplication.class, args);
                 KafkaTemplate<String, String> template = context.getBean(KafkaTemplate.class);
                 template.send("order-events", "Test order created!");
             }
         }
         ```
     - Run Eureka, Order Service, and Payment Service:
       - `cd C:\ecommerce\eureka-server && mvn spring-boot:run`
       - `cd C:\ecommerce\order-service && mvn spring-boot:run`
       - `cd C:\ecommerce\payment-service && mvn spring-boot:run`
     - Check: Look at Payment Service terminal, see “Payment Service received: Test order created!”.
- **Result**: Services communicate async via Kafka.

## Next Steps
- **Timeline**: 2-3 days
  - Day 1: Set up folders, Keycloak.
  - Day 2-3: Configure and start services, test communication.
- **Next**: Phase 2 continues—build entities, repositories, and APIs for each service.
- **Git**: Save progress:
  ```
  git add .
  git commit -m "Phase 2: Configured microservices and communication"
  ```

## Best Practices
- Run Eureka first, then services.
- Use separate terminals for each service.
- Replace `your_password` with a secure value.
- Check logs for errors if a service doesn’t start.