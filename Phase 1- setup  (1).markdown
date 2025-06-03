# Step-by-Step Guide for Phase 1: Initial Setup for E-Commerce Backend

## Objective

We’re starting the e-commerce backend by setting up everything needed to build our microservices with Java Spring Boot. This guide walks you through every tiny step of Phase 1 from our plan, like holding your hand, to get the environment, project, and core services ready. Let’s do this slowly and clearly!

---

## 1. Environment Preparation

**What We’re Doing**: We need to install all the tools and services (Java, Maven, Docker, databases, etc.) so our backend can run and connect to databases and messaging systems.

### Step 1.1: Install Java 17

- **Why**: Spring Boot needs Java to run, and version 17 is a stable, long-term support (LTS) version.
- **How**:
  1. Go to https://adoptium.net (a trusted source for Java).
  2. Click “Latest Release” or find “Temurin 17 (LTS)”.
  3. Download the installer for your system (e.g., Windows, macOS, Linux).
  4. Run the installer:
     - Follow prompts (click “Next”, accept terms, choose install location).
     - Default location is fine (e.g., C:\\Program Files\\Eclipse Adoptium).
  5. Set the `JAVA_HOME` environment variable:
     - **Windows**:
       - Right-click “This PC” &gt; “Properties” &gt; “Advanced system settings” &gt; “Environment Variables”.
       - Under “System variables”, click “New”.
       - Variable name: `JAVA_HOME`
       - Variable value: Path to Java (e.g., `C:\Program Files\Eclipse Adoptium\jdk-17.0.9+9`).
       - Find “Path” in System variables, click “Edit”, add `%JAVA_HOME%\bin`.
     - **macOS/Linux**:
       - Open terminal.
       - Edit your shell file (e.g., `nano ~/.bash_profile` or `nano ~/.zshrc`).
       - Add: `export JAVA_HOME=/path/to/java` (e.g., `/usr/lib/jvm/temurin-17-jdk`).
       - Add to path: `export PATH=$JAVA_HOME/bin:$PATH`.
       - Save and run: `source ~/.bash_profile` (or `~/.zshrc`).
  6. Verify:
     - Open a terminal (Command Prompt on Windows, Terminal on macOS/Linux).
     - Type: `java -version`
     - Look for: “17.0.9” or similar—means it’s working!
- **Result**: Java 17 is installed and ready.

### Step 1.2: Install Maven

- **Why**: Maven builds our Spring Boot projects, manages dependencies (like libraries), and runs our code.
- **How**:
  1. Go to https://maven.apache.org/download.cgi.
  2. Download the latest “Binary zip archive” (e.g., `apache-maven-3.9.6-bin.zip`).
  3. Unzip it to a folder (e.g., `C:\Program Files\Maven` on Windows, or `~/maven` on macOS/Linux).
  4. Set environment variables:
     - **Windows**:
       - Right-click “This PC” &gt; “Properties” &gt; “Advanced system settings” &gt; “Environment Variables”.
       - Under “System variables”, click “New”.
       - Variable name: `M2_HOME`
       - Variable value: Path to Maven (e.g., `C:\Program Files\Maven\apache-maven-3.9.6`).
       - Find “Path”, click “Edit”, add `%M2_HOME%\bin`.
     - **macOS/Linux**:
       - Open terminal.
       - Edit shell file: `nano ~/.bash_profile` or `nano ~/.zshrc`.
       - Add: `export M2_HOME=/path/to/maven` (e.g., `/home/user/maven/apache-maven-3.9.6`).
       - Add: `export PATH=$M2_HOME/bin:$PATH`.
       - Save and run: `source ~/.bash_profile` (or `~/.zshrc`).
  5. Verify:
     - Open a terminal.
     - Type: `mvn -version`
     - Look for: “Apache Maven 3.9.6” or similar—means it’s ready!
- **Result**: Maven is installed for building projects.

### Step 1.3: Install Docker

- **Why**: Docker runs our databases (PostgreSQL, MongoDB, Redis) and Kafka in containers, making setup easy and consistent.
- **How**:
  1. Go to https://www.docker.com.
  2. Download “Docker Desktop” for your system (Windows, macOS, or Linux).
  3. Run the installer:
     - Follow prompts (click “Next”, accept terms, install).
     - On Windows, enable WSL 2 if prompted (a feature for running containers).
  4. Start Docker Desktop:
     - Open the app; it runs in the background.
  5. Verify:
     - Open a terminal.
     - Type: `docker --version`
     - Look for: “Docker version 24.0.7” or similar—means it’s working!
- **Result**: Docker is ready to run our services.

### Step 1.4: Set Up Databases and Kafka

- **Why**: We need PostgreSQL for users/orders, MongoDB for products, Redis for caching/carts, and Kafka for messaging between services.
- **How**:
  1. **PostgreSQL**:
     - **Purpose**: Stores structured data (e.g., user IDs, order details).
     - Open a terminal.
     - Run:

       ```
       docker run -d -p 5432:5432 --name ecom_db -e POSTGRES_PASSWORD=password postgres:15
       ```
     - **Explain**:
       - `docker run`: Starts a container.
       - `-d`: Runs in background.
       - `-p 5432:5432`: Maps port 5432 on your machine to the container.
       - `--name postgres`: Names it “postgres”.
       - `-e POSTGRES_PASSWORD=your_password`: Sets password (replace “your_password” with something secure, e.g., “mysecret123”).
       - `postgres:15`: Uses PostgreSQL version 15.
     - Check: Type `docker ps` to see it running.
  2. **MongoDB**:
     - **Purpose**: Stores flexible product data (e.g., name, price, description).
     - In terminal, run:

       ```
       docker run -d -p 27017:27017 --name mongodb mongo:6
       ```
     - **Explain**:
       - `-p 27017:27017`: Maps MongoDB’s default port.
       - `--name mongodb`: Names the container.
       - `mongo:6`: Uses MongoDB version 6.
     - Check: Type `docker ps` to confirm it’s running.
  3. **Redis**:
     - **Purpose**: Fast storage for carts and caching product lists.
     - In terminal, run:

       ```
       docker run -d -p 6379:6379 --name redis redis:7
       ```
     - **Explain**:
       - `-p 6379:6379`: Maps Redis’s default port.
       - `--name redis`: Names the container.
       - `redis:7`: Uses Redis version 7.
     - Check: Type `docker ps` to see it running.
  4. **Kafka**:
     - **Purpose**: Sends messages (e.g., “order created”) between services.
     - Create a file named `docker-compose.yml`:
       - Open a text editor (e.g., Notepad, VS Code).
       - Copy and paste:

         ```yaml
         version: '3'
         services:
           zookeeper:
             image: confluentinc/cp-zookeeper:latest
             ports:
               - "2181:2181"
             environment:
               ZOOKEEPER_CLIENT_PORT: 2181
               ZOOKEEPER_TICK_TIME: 2000
           kafka:
             image: confluentinc/cp-kafka:latest
             depends_on:
               - zookeeper
             ports:
               - "9092:9092"
             environment:
               KAFKA_BROKER_ID: 1
               KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
               KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
               KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
         ```
       - Save it in a folder (e.g., `C:\ecommerce` or `~/ecommerce`).
     - In terminal, go to that folder: `cd C:\ecommerce` (or your path).
     - Run:

       ```
       docker-compose up -d
       ```
     - **Explain**:
       - `zookeeper`: A service Kafka needs to manage brokers.
       - `ports: "2181:2181"`: Maps Zookeeper’s port.
       - `kafka`: The messaging system.
       - `depends_on: zookeeper`: Starts Zookeeper first.
       - `ports: "9092:9092"`: Maps Kafka’s port.
       - Check: Type `docker ps` to see both running.
- **Result**: PostgreSQL, MongoDB, Redis, and Kafka are running in Docker.

### Step 1.5: Install IDE

- **Why**: IntelliJ IDEA helps us write, debug, and run Spring Boot code easily.
- **How**:
  1. Go to https://www.jetbrains.com/idea.
  2. Download the Community Edition (free) or Ultimate (paid, has extra features).
  3. Run the installer:
     - Click “Next”, accept terms, choose install path (default is fine).
     - Install and open IntelliJ IDEA.
  4. Start a new project later—we’ll use it soon!
- **Result**: IDE ready for coding.

### Step 1.6: Install Git

- **Why**: Git tracks our code changes so we can save and manage versions.
- **How**:
  1. Go to https://git-scm.com.
  2. Download the installer for your system.
  3. Run it:
     - Click “Next” through prompts, use default settings.
  4. Verify:
     - Open a terminal.
     - Type: `git --version`
     - Look for: “git version 2.43.0” or similar—means it’s working!
- **Result**: Git is installed for version control.

---

## 2. Project Initialization

- **What We’re Doing**: We’ll create a Spring Boot parent project and sub-modules for our microservices (User, Product, Cart, Order, Payment).

### Step 2.1: Use Spring Initializr

- **Why**: Spring Initializr gives us a ready-to-go project with the right setup.
- **How**:
  1. Open your browser.
  2. Go to https://start.spring.io.
  3. Fill in the form:
     - **Project**: Select “Maven”.
     - **Language**: Select “Java”.
     - **Spring Boot**: Choose the latest stable (e.g., “3.2.x”).
     - **Group**: Type `com.ecommerce`.
     - **Artifact**: Type `microservices-parent`.
     - **Java**: Select “17”.
     - **Dependencies**: Click “Add Dependencies” and search/add:
       - Spring Web (for REST APIs)
       - Spring Data JPA (for PostgreSQL)
       - Spring Data MongoDB (for MongoDB)
       - Spring for Apache Kafka (for messaging)
       - Spring Security (for security)
       - Spring Boot Starter OAuth2 Resource Server (for OAuth2)
       - Spring Boot Actuator (for monitoring)
       - Spring Cloud Netflix Eureka Client (for service discovery)
  4. Click “Generate”.
  5. Download the zip file (e.g., `microservices-parent.zip`).
  6. Unzip it to a folder (e.g., `C:\ecommerce` or `~/ecommerce`).
  7. Open IntelliJ IDEA:
     - Click “File” &gt; “Open”.
     - Select the unzipped folder (e.g., `microservices-parent`).
     - Click “OK” to load.
- **Result**: Base project is ready.

### Step 2.2: Create Sub-Modules

- **Why**: Each microservice (User, Product, etc.) gets its own module for independence.
- **How**:
  1. In IntelliJ, right-click the `microservices-parent` project in the Project pane.
  2. Select “New” &gt; “Module”.
  3. Choose “Maven” and click “Next”.
  4. For each module, set:
     - **ArtifactId**: `user-service`
     - Click “Next”, then “Finish”.
  5. Repeat for:
     - `product-service`
     - `cart-service`
     - `order-service`
     - `payment-service`
  6. Check: You’ll see folders like `user-service`, `product-service`, etc., under `microservices-parent`.
  7. For each module:
     - Ensure a structure like `src/main/java/com/ecommerce/userservice` (adjust for each service).
     - IntelliJ auto-creates basic files (e.g., `UserServiceApplication.java`).
- **Result**: Five sub-modules ready for development.

### Step 2.3: Update Parent pom.xml

- **Why**: The parent project needs dependencies and module links for all services.
- **How**:
  1. Open `microservices-parent/pom.xml` in IntelliJ.
  2. Replace the `<dependencies>` section with:

     ```xml
     <dependencies>
       <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-web</artifactId>
       </dependency>
       <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-data-jpa</artifactId>
       </dependency>
       <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-data-mongodb</artifactId>
       </dependency>
       <dependency>
         <groupId>org.springframework.kafka</groupId>
         <artifactId>spring-kafka</artifactId>
       </dependency>
       <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-security</artifactId>
       </dependency>
       <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
       </dependency>
       <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-actuator</artifactId>
       </dependency>
       <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
       </dependency>
     </dependencies>
     <dependencyManagement>
       <dependencies>
         <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-dependencies</artifactId>
           <version>2023.0.3</version>
           <type>pom</type>
           <scope>import</scope>
         </dependency>
       </dependencies>
     </dependencyManagement>
     ```
  3. Add a `<modules>` section before `</project>`:

     ```xml
     <modules>
       <module>user-service</module>
       <module>product-service</module>
       <module>cart-service</module>
       <module>order-service</module>
       <module>payment-service</module>
     </modules>
     ```
  4. Save the file.
- **Result**: Parent project links to all sub-modules with shared dependencies.

### Step 2.4: Initialize Git

- **Why**: Git saves our progress and tracks changes.
- **How**:
  1. Open a terminal.
  2. Go to the project folder: `cd C:\ecommerce\microservices-parent` (or your path).
  3. Run:

     ```
     git init
     ```
     - This starts a Git repository.
  4. Add all files:

     ```
     git add .
     ```
  5. Commit changes:

     ```
     git commit -m "Initial project setup"
     ```
  6. Check: Run `git status` to see files tracked.
- **Result**: Code is under version control.

---

## 3. Service Discovery & Gateway

- **What We’re Doing**: Set up Eureka so microservices can find each other, and an API Gateway to route requests.

### Step 3.1: Eureka Server

- **Why**: Eureka is a discovery server—microservices register here so others can find them.
- **How**:
  1. Open https://start.spring.io.
  2. Fill in:
     - Project: Maven
     - Language: Java
     - Spring Boot: Latest stable (e.g., 3.2.x)
     - Group: `com.ecommerce`
     - Artifact: `eureka-server`
     - Java: 17
     - Dependency: Spring Cloud Netflix Eureka Server
  3. Click “Generate”, download, and unzip `eureka-server.zip`.
  4. Open in IntelliJ:
     - File &gt; Open &gt; Select `eureka-server` folder &gt; OK.
  5. Edit the main class:
     - Open `src/main/java/com/ecommerce/eurekaserver/EurekaServerApplication.java`.
     - Replace with:

       ```java
       import org.springframework.boot.SpringApplication;
       import org.springframework.boot.autoconfigure.SpringBootApplication;
       import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;
       
       @SpringBootApplication
       @EnableEurekaServer
       public class EurekaServerApplication {
           public static void main(String[] args) {
               SpringApplication.run(EurekaServerApplication.class, args);
           }
       }
       ```
     - This enables Eureka.
  6. Configure:
     - Open `src/main/resources/application.properties`.
     - Add:

       ```
       server.port=8761
       eureka.client.register-with-eureka=false
       eureka.client.fetch-registry=false
       ```
     - **Explain**:
       - `server.port=8761`: Runs Eureka on port 8761.
       - `eureka.client.register-with-eureka=false`: Stops Eureka from registering itself.
       - `eureka.client.fetch-registry=false`: No need to fetch other services.
  7. Run:
     - In terminal, go to `eureka-server` folder: `cd C:\ecommerce\eureka-server` (or your path).
     - Type:

       ```
       mvn spring-boot:run
       ```
  8. Check:
     - Open browser, go to http://localhost:8761.
     - Look for: Eureka dashboard (a webpage showing “Instances currently registered”).
- **Result**: Eureka server runs on port 8761.

### Step 3.2: API Gateway

- **Why**: The API Gateway is the entry point—routes requests (e.g., /api/users) to the right microservice.
- **How**:
  1. Open https://start.spring.io.
  2. Fill in:
     - Project: Maven
     - Language: Java
     - Spring Boot: Latest stable (e.g., 3.2.x)
     - Group: `com.ecommerce`
     - Artifact: `api-gateway`
     - Java: 17
     - Dependencies: Spring Cloud Gateway, Spring Cloud Netflix Eureka Client
  3. Click “Generate”, download, and unzip `api-gateway.zip`.
  4. Open in IntelliJ:
     - File &gt; Open &gt; Select `api-gateway` folder &gt; OK.
  5. Edit the main class:
     - Open `src/main/java/com/ecommerce/apigateway/ApiGatewayApplication.java`.
     - Replace with:

       ```java
       import org.springframework.boot.SpringApplication;
       import org.springframework.boot.autoconfigure.SpringBootApplication;
       import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
       
       @SpringBootApplication
       @EnableDiscoveryClient
       public class ApiGatewayApplication {
           public static void main(String[] args) {
               SpringApplication.run(ApiGatewayApplication.class, args);
           }
       }
       ```
     - **Explain**: `@EnableDiscoveryClient` connects to Eureka.
  6. Configure:
     - Open `src/main/resources/application.properties`.
     - Add:

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
     - **Explain**:
       - `server.port=8080`: Gateway runs on port 8080.
       - `spring.cloud.gateway.routes`: Routes requests (e.g., /api/users/\*\* goes to user-service).
       - `uri=lb://user-service`: Uses Eureka to find services (“lb” means load balance).
       - `eureka.client.service-url.defaultZone`: Connects to Eureka.
       - `spring.application.name`: Names this service.
  7. Run:
     - In terminal, go to `api-gateway` folder: `cd C:\ecommerce\api-gateway` (or your path).
     - Type:

       ```
       mvn spring-boot:run
       ```
  8. Check:
     - Ensure Eureka is running (http://localhost:8761).
     - Gateway runs on 8080, but we’ll test routing later with microservices.
- **Result**: API Gateway runs on port 8080, ready to route.

---

## Next Steps

- **Timeline**: Aim for 1-2 days.
  - **Day 1**: Install tools, set up databases and Kafka.
  - **Day 2**: Initialize project, set up Eureka and API Gateway.
- **Next Phase**: Move to Phase 2—set up Keycloak and configure `application.properties` for each microservice.
- **Git**: Save progress:

  ```
  git add .
  git commit -m "Phase 1 complete: initial setup
  ```