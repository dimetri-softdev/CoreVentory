# CoreVentory 📦

**CoreVentory** is a modern, resilient, event-driven Microservices Inventory & Order Management System built with Java 21, Spring Boot 3, Apache Kafka, and Redis.

The system addresses core e-commerce backend challenges: distributed data consistency, high-read catalog caching, concurrent stock reservation, and idempotent payment handling.

---

## 🏗️ Architecture Overview

```
                          ┌─────────────────────────────┐
                          │     Spring Cloud Gateway    │
                          └──────────────┬──────────────┘
                                         │
       ┌─────────────────┬───────────────┼───────────────┬─────────────────┐
       │ REST            │ REST          │ REST          │ REST            │
┌──────▼────────┐ ┌──────▼────────┐     │ ┌─────────────▼┐ ┌───────────────▼┐
│   Product &   │ │ Order Service │     │ │ Payment Svc │ │ Notification  │
│ Inventory Svc │ └──────┬────────┘     │ └──────┬──────┘ │    Service    │
└──────┬────────┘        │              │        │        └───────▲───────┘
       │                 │              │        │                │
   (Redis)          (PostgreSQL)        │   (PostgreSQL)          │
       │                                │        │                │
       └────────────────────────────────┴────────┴────────────────┘
                                        │
                        ┌───────────────▼──────────────┐
                        │   Apache Kafka Event Bus     │
                        └──────────────────────────────┘
```

---

## 🛠️ Tech Stack & Technologies

* **Language & Core:** Java 21, Spring Boot 3.x
* **Data & Caching:** Spring Data JPA, PostgreSQL, Redis
* **Messaging & Events:** Apache Kafka (Spring Kafka)
* **Resilience & Fault Tolerance:** Resilience4j (Circuit Breaker, Retry, Rate Limiter)
* **API & Documentation:** SpringDoc OpenAPI (Swagger 3)
* **Testing & Quality:** JUnit 5, Mockito, Testcontainers
* **Containerization & Ops:** Docker, Docker Compose, Spring Boot Actuator

---

## 📦 Microservices Breakdown

| Service | Port | Database / Cache | Responsibility |
| :--- | :--- | :--- | :--- |
| **ApiGateway** | `8080` | N/A | Central routing, rate limiting, request validation |
| **ProductService** | `8081` | PostgreSQL + Redis | Product catalog management, read caching, optimistic locking for inventory |
| **OrderService** | `8082` | PostgreSQL | Order creation (`PENDING`, `CONFIRMED`, `CANCELLED`), Saga orchestration |
| **PaymentService** | `8083` | PostgreSQL | Idempotent payment processing, Resilience4j circuit breakers |
| **NotificationService** | `8084` | N/A | Asynchronous event listener (Kafka), transactional notification delivery |

---

## ⚡ Key Features

1. **High-Performance Caching:** Product requests are cached in Redis (`Cache-Aside` pattern) to handle heavy read traffic.
2. **Concurrent Inventory Allocation:** Inventory updates leverage optimistic locking (`@Version`) to prevent race conditions during high-concurrency checkouts.
3. **Event-Driven Asynchronous Processing:** Order state transitions publish events (`order-created`, `payment-processed`) via Kafka.
4. **Resilience & Circuit Breaking:** External REST calls wrap with Resilience4j to gracefully handle service degradation.
5. **Idempotency:** Payment events enforce unique idempotency keys to ensure payments are processed exactly once.

---

## 🚀 Getting Started

### Prerequisites

* **Java 21 JDK** or higher
* **Maven 3.8+**
* **Docker & Docker Compose**

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/CoreVentory.git
cd CoreVentory
```

### 2. Start Infrastructure Services

Spin up PostgreSQL, Redis, and Apache Kafka using Docker Compose:

```bash
docker-compose up -d
```

### 3. Build & Run Services

Build the entire project:

```bash
mvn clean package -DskipTests
```

Run services individually or using Spring Boot plugin:

```bash
# Example: Run Product Service
cd product-service
mvn spring-boot:run
```

---

## 🔗 API Endpoints Summary

### Product & Inventory Service (`:8081`)
* `GET /api/v1/products` - Fetch product catalog (Cached via Redis)
* `GET /api/v1/products/{id}` - Get product details
* `POST /api/v1/products` - Add a new product
* `PATCH /api/v1/products/{id}/stock` - Update inventory stock

### Order Service (`:8082`)
* `POST /api/v1/orders` - Place a new order (Triggers `order-created` Kafka event)
* `GET /api/v1/orders/{id}` - Fetch order details and state

### Payment Service (`:8083`)
* `POST /api/v1/payments` - Process payment manually / fallback API

---

## 🧪 Testing

Execute unit and integration tests across all microservices using **Testcontainers**:

```bash
mvn test
```

---

## 📄 License

Distributed under the MIT License. See `LICENSE` for details.
