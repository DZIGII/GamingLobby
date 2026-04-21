# NotificationService

A Spring Boot microservice responsible for handling and dispatching notifications within the **GamingLoby** platform. It listens for asynchronous events from other services via ActiveMQ, sends email notifications to users, and persists notification records to a PostgreSQL database.

---

## Overview

The NotificationService is one of several microservices that make up the GamingLoby ecosystem:

| Service | Role |
|---|---|
| EurekaServer | Service discovery & registry |
| GamingLobbyApiGateway | API Gateway — routes external traffic |
| UserService | User management & authentication |
| GamingService | Gaming session management |
| **NotificationService** | **Email notifications & notification history** |

---

## Features

- Consumes notification events from an **ActiveMQ** queue (`notification.queue`)
- Sends **email notifications** via Gmail SMTP for the following event types:
  - `SESSION_INVITE` — a user has been invited to a gaming session
  - `USER_ACTIVATION` — account activation email with a one-time link
  - `SESSION_CANCELLED` — the gaming session a user joined has been cancelled
- Persists every notification (with status `SENT` / `FAILED`) to PostgreSQL
- Exposes a paginated **admin REST API** to retrieve notification history
- Secured with **JWT** (HS512) using an AOP-based role-check mechanism
- Registers with **Eureka** for service discovery

---

## Tech Stack

| Category | Technology |
|---|---|
| Language | Java 11 |
| Framework | Spring Boot 2.2.2 |
| Service Discovery | Spring Cloud Netflix Eureka |
| Messaging | Apache ActiveMQ (JMS) |
| Persistence | Spring Data JPA + PostgreSQL |
| Email | Spring Mail (Gmail SMTP) |
| Security | JWT (JJWT 0.11.5) |
| Build | Maven |

---

## Prerequisites

- **Java 11**
- **Maven 3.6+**
- **PostgreSQL** running on port `5433`
- **Apache ActiveMQ** broker running on port `61616`
- **EurekaServer** running on port `8761`
- **UserService** running on port `8081`
- Gmail account with an App Password configured

---

## Configuration

All configuration lives in `src/main/resources/application.properties`.

```properties
# Server
server.port=8083

# Eureka
eureka.client.service-url.defaultZone=http://localhost:8761/eureka

# Database
spring.datasource.url=jdbc:postgresql://localhost:5433/notificationservice
spring.datasource.username=postgres
spring.datasource.password=password

# ActiveMQ
spring.activemq.broker-url=tcp://localhost:61616

# Email (Gmail SMTP)
spring.mail.host=smtp.gmail.com
spring.mail.port=587
spring.mail.username=<your-gmail>
spring.mail.password=<your-app-password>

# JWT
jwt.secret=<256-character-secret>
```

---

## Running the Service

```bash
# Clone the repository
git clone git@github.com:DZIGII/GamingLoby.git
cd GamingLoby/NotificationService

# Build
mvn clean install

# Run
mvn spring-boot:run
```

Make sure PostgreSQL, ActiveMQ, and EurekaServer are running before starting this service.

---

## REST API

### `GET /notifications`

Returns a paginated list of all notifications. Requires a valid JWT with the `ADMIN` role.

**Headers:**
```
Authorization: Bearer <token>
```

**Query Parameters:**

| Parameter | Type | Description |
|---|---|---|
| page | int | Page number (0-based) |
| size | int | Page size |

**Response:**
```json
{
  "content": [
    {
      "id": 1,
      "userId": 42,
      "type": "SESSION_INVITE",
      "content": "...",
      "status": "SENT",
      "sentAt": "2024-01-01T12:00:00"
    }
  ],
  "totalElements": 100,
  "totalPages": 10
}
```

---

## Architecture

```
 GamingService ──┐
                 │  ActiveMQ (notification.queue)
 UserService ────┼──────────────────────────────► NotificationListener
                 │                                        │
                                                          ├── EmailService ──► Gmail SMTP
                                                          │
                                                          └── NotificationRepository ──► PostgreSQL

 API Gateway ──► NotificationControllerAdmin (GET /notifications)
```

---

## Project Structure

```
src/main/java/raf/sk/notificationservice/
├── controller/
│   └── NotificationControllerAdmin.java   # Admin REST endpoint
├── listener/
│   └── NotificationListener.java          # ActiveMQ JMS listener
├── service/
│   ├── NotificationService.java           # Service interface
│   ├── NotificationServiceImpl.java       # Business logic
│   └── EmailService.java                  # Email sending
├── model/
│   └── Notification.java                  # JPA entity
├── repository/
│   └── NotificationRepository.java        # Spring Data repository
├── security/
│   ├── CheckSecurity.java                 # Custom security annotation
│   ├── SecurityAspect.java                # AOP JWT validation
│   └── TokenServiceImpl.java             # JWT implementation
├── dto/
│   ├── NotificationEventDto.java          # Incoming queue message
│   ├── NotificationResponseDto.java       # API response
│   └── UserDto.java                       # User data from UserService
└── client/
    └── UserServiceClientConfiguration.java # RestTemplate config
```

---

## License

This project is part of the GamingLoby academic/portfolio project.
