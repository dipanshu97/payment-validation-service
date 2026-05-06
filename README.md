# 🔐 Core Payment Integration System

> A secure, scalable payment validation and processing backend built for a real UK-based e-commerce client using Java, Spring Boot, and Microservices architecture.

---

## 📌 Project Overview

This system was built during my internship at **HulkHire Tech** as the core payment processing engine for **ToyCuddle** — a UK-based e-commerce platform targeting US and European markets.

ToyCuddle already had a live e-commerce platform with basic payment integrations. The goal of this project was to build a **robust, secure, and scalable payment engine** that could eventually integrate with multiple Payment Service Providers (PSPs) like PayPal, Stripe, Trustly, Square, and Apple Pay.

**My Role:** Java Developer Intern — involved in both design discussions and hands-on development of critical payment components.

**Team:** Product Owner, Tech Lead, 2 Senior Developers, 2 QA Engineers, and Interns.

**Methodology:** Agile Scrum — 3 sprints (2 weeks each), daily 10AM standups, weekly demos.

**Achievement:** ⭐ Awarded Star Performer of the Month for delivering all tasks on time and supporting the team to meet sprint deadlines.

---

## 🏗️ Architecture

The system is split into two independent microservices:

```
┌─────────────────────────────────────────────────────────────┐
│                    ToyCuddle E-Commerce                      │
│                  (UK-based client system)                    │
└─────────────────────────┬───────────────────────────────────┘
                          │  HmacSHA256 Signed Request
                          ▼
┌─────────────────────────────────────────────────────────────┐
│              payment-validation-service                      │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  Spring Security + HmacSHA256 Authentication        │    │
│  └──────────────────────┬──────────────────────────────┘    │
│                         ▼                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  Modular Validation Framework (Chain of Resp.)      │    │
│  │  • Duplicate Payment Detection                      │    │
│  │  • Payment Attempt Limit Check                      │    │
│  │  • Field-level Business Validations                 │    │
│  └──────────────────────┬──────────────────────────────┘    │
└─────────────────────────┼───────────────────────────────────┘
                          │  Validated Request
                          ▼
┌─────────────────────────────────────────────────────────────┐
│              payment-processing-service                      │
│                                                              │
│  • Transaction Execution                                     │
│  • Payment Status Management (INITIATED → SUCCESS/FAILED)   │
│  • Custom Error Codes & Exception Handling                   │
│  • Redis Caching for Performance                             │
└────────────────┬──────────────────────┬─────────────────────┘
                 │                      │
                 ▼                      ▼
        ┌──────────────┐       ┌──────────────────┐
        │  MySQL (RDS) │       │   Redis Cache     │
        │  AWS RDS     │       │   (AWS EC2)       │
        └──────────────┘       └──────────────────┘
```

---

## ✨ Key Features

### 🔒 Security & Authentication
- **HmacSHA256** — Generates a unique hash signature combining a secret key + request payload. Ensures every payment request from ToyCuddle is authenticated and data is not tampered with in transit.
- **Spring Security** — Secures all REST API endpoints. Unauthorized requests are rejected before reaching business logic.
- **AWS Secrets Manager** — Database credentials and API keys are stored securely. No sensitive data hardcoded in the codebase. Application fetches credentials at runtime via IAM roles.

### ✅ Modular Validation Framework
- Rule-based, plug-and-play validation engine using **Chain of Responsibility** pattern
- Business rules implemented:
  - **Duplicate Payment Detection** — Prevents the same payment from being processed twice
  - **Payment Attempt Limits** — Restricts excessive retry attempts on a single transaction
  - **Field Validations** — Amount, currency, merchant ID, and other required fields
- Adding new rules requires zero changes to the core engine

### ⚡ Performance Optimization
- **Redis Caching** — Frequently accessed data (payment status, validation lookups) served from in-memory cache
- **TTL-based cache expiry** — Stale entries automatically cleaned up
- Reduced database round trips on high-frequency queries

### 📊 Payment Status Management
- Full lifecycle tracking: `INITIATED` → `PROCESSING` → `SUCCESS` / `FAILED`
- Every transaction state is persisted and traceable
- Ensures 100% reliability and auditability of payments

### 🛡️ Error Handling
- Custom error codes for every failure scenario
- Spring Exception Handling (`@ControllerAdvice`) for consistent, structured error responses
- Comprehensive logging with Log4J / Logback for debugging and audit trails

---

## 🧱 Design Patterns

| Pattern | Where Used | Why |
|---|---|---|
| **Chain of Responsibility** | Validation Framework | Each rule is a handler; request passes through chain. Easy to add/remove rules. |
| **Factory Pattern** | Validation Rule Creation | Dynamically creates rule instances based on payment type — avoids hard-coded conditionals |
| **Builder Pattern** | Payment Request/Response Objects | Constructs complex objects with many optional fields cleanly |

---

## 🛠️ Tech Stack

| Category | Technology |
|---|---|
| Language | Java 17 |
| Framework | Spring Boot |
| Architecture | Microservices |
| Security | Spring Security, HmacSHA256 |
| Database | MySQL 8.x (AWS RDS) |
| Caching | Redis |
| Cloud | AWS EC2, RDS, Secrets Manager |
| Build Tool | Maven |
| API Testing | Postman |
| Testing | JUnit 5, Mockito, Code Coverage |
| Version Control | Git, BitBucket, SourceTree |
| Logging | Log4J / Logback |
| IDE | Eclipse STS |
| Agile | Scrum, Jira |
| Other Libraries | GSON, Lombok, SonarLint |

---

## 📁 Project Structure

```
payment-integration-system/
│
├── payment-validation-service/
│   ├── src/main/java/
│   │   ├── controller/          # REST API endpoints
│   │   ├── service/             # Business logic
│   │   ├── validation/
│   │   │   ├── framework/       # Core validation engine (Chain of Responsibility)
│   │   │   └── rules/           # Individual validation rules
│   │   ├── security/            # HmacSHA256 + Spring Security config
│   │   └── exception/           # Custom error codes & global exception handler
│   ├── src/test/                # JUnit + Mockito unit tests
│   └── pom.xml
│
├── payment-processing-service/
│   ├── src/main/java/
│   │   ├── controller/
│   │   ├── service/
│   │   ├── status/              # Payment status lifecycle management
│   │   └── cache/               # Redis integration
│   ├── src/test/
│   └── pom.xml
│
└── README.md
```

---

## 🚀 Getting Started (Local Setup)

### Prerequisites
- Java 17
- Maven
- MySQL 8.x
- Redis (Windows: [Redis Windows port](https://github.com/microsoftarchive/redis/releases))
- Eclipse STS or IntelliJ IDEA

### Local Setup

```bash
# 1. Clone the repository
git clone https://github.com/dipanshu97/payment-integration-system.git
cd payment-integration-system

# 2. Start Redis locally (Windows)
redis-server.exe

# 3. Configure application-dev.properties
spring.datasource.url=jdbc:mysql://localhost:3306/payments_db
spring.datasource.username=root
spring.datasource.password=your_password
spring.redis.host=localhost
spring.redis.port=6379

# 4. Build and run validation service
cd payment-validation-service
mvn clean package -P dev
mvn spring-boot:run

# 5. Build and run processing service
cd ../payment-processing-service
mvn clean package -P dev
mvn spring-boot:run
```

---

## ☁️ AWS Deployment

### EC2 Setup
```bash
# Connect to EC2
ssh -i your-key-pair.pem ec2-user@<your-ec2-ip>

# Install Java 17 on EC2
sudo su
yum update
yum install java-17-amazon-corretto-17.0.10+7-1.amzn2.1.x86_64

# Verify
java --version
```

### Deploy JAR to EC2
```bash
# Build locally
mvn clean package -P dev

# Upload to EC2
scp -i your-key-pair.pem ./payment-validation-service.jar ec2-user@<ec2-ip>:/home/ec2-user
scp -i your-key-pair.pem ./payment-processing-service.jar ec2-user@<ec2-ip>:/home/ec2-user

# Run on EC2 (background process)
chmod 777 *
nohup java -jar -Xms100M -Xmx150M payment-processing-service.jar > processing-service.log &
nohup java -jar -Xms100M -Xmx150M payment-validation-service.jar > validation-service.log &
```

### AWS Secrets Manager (Secure Credentials)
```bash
# Verify secret is accessible from EC2
aws secretsmanager get-secret-value --secret-id dev/payment-validation-service --region ap-south-1
```

```properties
# application-dev.properties — no hardcoded passwords
spring.config.import=aws-secretsmanager:dev/payment-validation-service
```

---

## 🧪 Testing

```bash
# Run all tests
mvn test

# Run with coverage report
mvn test jacoco:report
```

Tests cover:
- Validation framework — all business rules tested in isolation
- Mocked database and Redis dependencies using Mockito
- Edge cases: duplicate payments, exceeded attempt limits, invalid fields

---

## 📅 Project Timeline

| Phase | Duration | What happened |
|---|---|---|
| Knowledge Transfer | Week 1-2 | Architecture, domain knowledge, tools setup |
| Sprint 1 | Nov 18 – Nov 29 | Validation framework, HmacSHA256 security |
| Sprint 2 | Dec 2 – Dec 13 | Payment processing, Redis, error handling |
| Sprint 3 | Dec 16 – Dec 27 | AWS deployment, testing, sprint completion |

---

## 📬 Contact

**Deepanshu Gupta** — Java Backend Developer  
📧 dipanshu.raj989@gmail.com  
💼 [LinkedIn](https://linkedin.com/in/dev-deepanshu-gupta)  
🐙 [GitHub](https://github.com/dipanshu97)
