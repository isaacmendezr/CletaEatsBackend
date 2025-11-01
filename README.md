# CletaEats Backend

<em>A comprehensive food delivery system backend built with Kotlin and Spring Boot, managing clients, restaurants, delivery drivers, and orders with real-time status tracking and analytics.</em>

---

## Table of Contents

- [CletaEats Backend](#cletaeats-backend)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Architecture](#architecture)
  - [Features](#features)
  - [Technology Stack](#technology-stack)
  - [Ecosystem](#ecosystem)
  - [Database Model](#database-model)
  - [API Endpoints](#api-endpoints)
    - [Authentication](#authentication)
    - [Clients](#clients)
    - [Delivery Drivers](#delivery-drivers)
    - [Restaurants](#restaurants)
    - [Orders](#orders)
    - [Reports](#reports)
  - [Project Structure](#project-structure)
  - [Getting Started](#getting-started)
    - [Prerequisites](#prerequisites)
    - [Installation](#installation)
    - [Database Setup](#database-setup)
    - [Configuration](#configuration)
    - [Usage](#usage)
  - [Business Logic](#business-logic)
    - [Order Creation Flow](#order-creation-flow)
    - [Order Status Workflow](#order-status-workflow)
    - [Driver Management](#driver-management)
  - [Contact](#contact)

---

## Overview

**CletaEats Backend** is a RESTful API service for a food delivery platform (similar to Uber Eats or Rappi) that manages the complete lifecycle of food orders, from customer placement to delivery completion.

The system handles:
- Multi-role user authentication (clients, restaurants, delivery drivers, admins)
- Real-time order tracking with multiple states
- Automatic delivery driver assignment
- Dynamic cost calculation (distance-based, with holiday rates)
- Driver performance tracking (complaints, warnings, daily kilometers)
- Comprehensive analytics and reporting

This backend communicates with:
- **CletaEatsApp** (Android Mobile App): [https://github.com/isaacmendezr/CletaEatsApp](https://github.com/isaacmendezr/CletaEatsApp)

---

## Architecture

**Backend Framework:**
- **Framework:** Spring Boot
- **Language:** Kotlin
- **Database:** PostgreSQL
- **ORM:** JPA/Hibernate
- **Migration Tool:** Flyway
- **Design Pattern:** MVC (Model-View-Controller) with Service Layer
- **API Style:** RESTful JSON API

**Layered Architecture:**
```
Controllers (REST Endpoints)
    ↓
Services (Business Logic)
    ↓
Repositories (Data Access)
    ↓
Database (PostgreSQL)
```

---

## Features

| Category | Description |
| :-------- | :----------- |
| 🔐 **Authentication** | Multi-role login system supporting clients, delivery drivers, restaurants, and admins |
| 📦 **Order Management** | Complete order lifecycle: creation, preparation, in transit, delivery, and suspension |
| 🚗 **Smart Assignment** | Automatic delivery driver assignment based on availability and performance metrics |
| 💰 **Cost Calculation** | Dynamic pricing with IVA (13%), distance-based transport costs, and holiday rate differential |
| ⚠️ **Driver Monitoring** | Complaint and warning system with automatic deactivation at 4+ warnings |
| 📊 **Analytics** | Comprehensive reports: revenue by restaurant, peak hours, top customers, driver performance |
| 🔄 **State Management** | Controlled state transitions for clients, drivers, and orders with role-based permissions |
| 🍽️ **Menu System** | Restaurant combo management with embedded pricing |

---

## Technology Stack

**Core Dependencies:**
- **Spring Boot** - Application framework
- **Kotlin** - Primary language
- **PostgreSQL** - Relational database
- **Spring Data JPA** - Data persistence and ORM
- **Flyway** - Database version control and migrations
- **Jackson Kotlin Module** - JSON serialization/deserialization
- **Gradle** - Build automation tool

**Configuration:**
- **Server Port:** 8080
- **Database:** `jdbc:postgresql://localhost:5432/cletaeats`
- **JVM Target:** Java 21
- **Flyway Baseline:** Enabled on migrate

---

## Ecosystem

The **CletaEats Backend** is part of an integrated food delivery application ecosystem:

| Component | Description | Repository |
|-----------|-------------|------------|
| 💻 **CletaEats Backend** | RESTful API service (this repository) | [CletaEatsBackend](https://github.com/isaacmendezr/CletaEatsBackend) |
| 📱 **CletaEatsApp** | Android mobile application for clients, restaurants, and delivery drivers | [CletaEatsApp](https://github.com/isaacmendezr/CletaEatsApp) |

The mobile app consumes all API endpoints to provide a complete user experience across different roles (clients placing orders, restaurants managing their orders, drivers delivering, and admins monitoring the system).

---

## Database Model

**Main Entities:**

| Entity | Description | Key Fields |
|--------|-------------|------------|
| **Cliente** | Registered customers | cedula, nombre, estado (active/suspended), numeroTarjeta |
| **Repartidor** | Delivery drivers | cedula, estado (available/busy/inactive), amonestaciones, quejas[], kmRecorridosDiarios |
| **Restaurante** | Food establishments | cedulaJuridica, tipoComida, combos[] |
| **Pedido** | Orders | clienteId, restauranteId, repartidorId, precio, distancia, costoTransporte, iva, total, estado, combos[] |
| **Admin** | System administrators | nombreUsuario, contrasena |

**Embedded Collections:**
- `RestauranteCombo`: Menu items within restaurants (numero, nombre, precio)
- `PedidoCombo`: Order line items
- `Repartidor.quejas`: List of complaints per driver

**Database Schema:** Managed via Flyway migration `V1__create_cletaeats_schema.sql` with pre-loaded test data.

---

## API Endpoints

### Authentication

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/auth/login` | Authenticate user (returns user type and data) |

**Request Body:**
```json
{
  "cedula": "123456789",
  "contrasena": "password1234"
}
```

**Response:**
```json
{
  "type": "success",
  "user": {
    "type": "cliente",
    "cliente": { /* cliente data */ }
  }
}
```

---

### Clients

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/clientes` | Get all clients |
| GET | `/clientes/active` | Get active clients only |
| GET | `/clientes/suspended` | Get suspended clients |
| POST | `/clientes/register` | Register new client |
| PUT | `/clientes/{cedula}` | Update client profile |

---

### Delivery Drivers

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/repartidores` | Get all delivery drivers |
| GET | `/repartidores/no-amonestations` | Get drivers with 0 warnings |
| POST | `/repartidores/register` | Register new driver |
| PUT | `/repartidores/{cedula}` | Update driver profile |
| POST | `/repartidores/{repartidorId}/quejas` | Add complaint/warning to driver |
| POST | `/repartidores/reset` | Reset all drivers to available status |

**Add Complaint Request:**
```json
{
  "queja": "Late delivery",
  "addAmonestacion": true
}
```

---

### Restaurants

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/restaurantes` | Get all restaurants |
| POST | `/restaurantes/register` | Register new restaurant |
| PUT | `/restaurantes/{cedulaJuridica}` | Update restaurant profile |

---

### Orders

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/pedidos` | Get all orders |
| POST | `/pedidos` | Create new order |
| POST | `/pedidos/{orderId}/status` | Update order status |

**Create Order Request:**
```json
{
  "clienteId": "1",
  "restauranteId": "1",
  "distancia": 5.0,
  "combos": [
    {
      "numero": 1,
      "nombre": "Pizza Margherita",
      "precio": 4000.0
    }
  ]
}
```

**Update Status Request:**
```json
{
  "newStatus": "en camino",
  "userType": "RestauranteUser",
  "userId": "1"
}
```

**Order States:**
- `en preparación` → `en camino` / `suspendido` (Restaurant can change)
- `en camino` → `entregado` (Driver can change)

---

### Reports

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/pedidos/reports/revenue/restaurant` | Total revenue grouped by restaurant |
| GET | `/pedidos/reports/restaurant/most-orders` | Restaurant with most orders |
| GET | `/pedidos/reports/restaurant/least-orders` | Restaurant with least orders |
| GET | `/pedidos/reports/repartidor/quejas` | Complaints grouped by driver |
| GET | `/pedidos/reports/cliente/pedidos` | Orders grouped by client |
| GET | `/pedidos/reports/cliente/most-orders` | Client with most orders |
| GET | `/pedidos/reports/pedidos/hora-pico` | Peak hour for orders |

---

## Project Structure

```sh
CletaEatsBackend/
├── build.gradle.kts
├── settings.gradle.kts
├── gradlew
├── gradlew.bat
├── gradle/
│   └── wrapper/
│       └── gradle-wrapper.properties
└── src/
    ├── main/
    │   ├── kotlin/
    │   │   └── com/example/cletaeatsbackend/
    │   │       ├── CletaEatsBackendApplication.kt
    │   │       ├── config/
    │   │       │   └── JacksonConfig.kt
    │   │       ├── controller/
    │   │       │   ├── AdminController.kt
    │   │       │   ├── AuthController.kt
    │   │       │   ├── ClienteController.kt
    │   │       │   ├── PedidoController.kt
    │   │       │   ├── RepartidorController.kt
    │   │       │   └── RestauranteController.kt
    │   │       ├── model/
    │   │       │   ├── Admin.kt
    │   │       │   ├── Cliente.kt
    │   │       │   ├── Pedido.kt
    │   │       │   ├── Repartidor.kt
    │   │       │   ├── Restaurante.kt
    │   │       │   └── UserType.kt
    │   │       ├── repository/
    │   │       │   ├── AdminRepository.kt
    │   │       │   ├── ClienteRepository.kt
    │   │       │   ├── PedidoRepository.kt
    │   │       │   ├── RepartidorRepository.kt
    │   │       │   └── RestauranteRepository.kt
    │   │       └── service/
    │   │           ├── AdminService.kt
    │   │           ├── AuthService.kt
    │   │           ├── ClienteService.kt
    │   │           ├── PedidoService.kt
    │   │           ├── RepartidorService.kt
    │   │           └── RestauranteService.kt
    │   └── resources/
    │       ├── application.properties
    │       └── db/
    │           └── migration/
    │               └── V1__create_cletaeats_schema.sql
    └── test/
        └── kotlin/
            └── com/example/cletaeatsbackend/
                └── CletaEatsBackendApplicationTests.kt
```

---

## Getting Started

### Prerequisites

- **Java 21** or higher
- **PostgreSQL 14+** (running on localhost:5432)
- **Gradle 8.x** (included via wrapper)
- **Git** for cloning the repository

### Installation

1. Clone the repository:
   ```sh
   git clone https://github.com/isaacmendezr/CletaEatsBackend.git
   cd CletaEatsBackend
   ```

2. Ensure PostgreSQL is running and accessible.

### Database Setup

1. Create the database:
   ```sh
   psql -U postgres
   CREATE DATABASE cletaeats;
   \q
   ```

2. Flyway will automatically run migrations on first startup, creating:
   - All tables (clientes, repartidores, restaurantes, pedidos, admins)
   - Element collection tables (combos, quejas)
   - Test data (3 clients, 3 drivers, 3 restaurants, 4 sample orders, 1 admin)

### Configuration

Update `src/main/resources/application.properties` if needed:

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/cletaeats
spring.datasource.username=postgres
spring.datasource.password=root
server.port=8080
```

### Usage

1. Run the application:
   ```sh
   ./gradlew bootRun
   ```
   Or on Windows:
   ```sh
   gradlew.bat bootRun
   ```

2. The API will be available at `http://localhost:8080`

3. Test authentication with pre-loaded data:
   ```sh
   curl -X POST http://localhost:8080/auth/login \
     -H "Content-Type: application/json" \
     -d '{"cedula":"123456789","contrasena":"password1234"}'
   ```

4. Create an order:
   ```sh
   curl -X POST http://localhost:8080/pedidos \
     -H "Content-Type: application/json" \
     -d '{
       "clienteId": "1",
       "restauranteId": "1",
       "distancia": 5.0,
       "combos": [{"numero": 1, "nombre": "Pizza Margherita", "precio": 4000.0}]
     }'
   ```

5. View all orders:
   ```sh
   curl http://localhost:8080/pedidos
   ```

---

## Business Logic

### Order Creation Flow

1. **Validation:**
   - Client must be active (not suspended)
   - Restaurant must exist
   - All combos must belong to the selected restaurant

2. **Driver Assignment:**
   - System selects random available driver
   - Driver must have < 4 warnings
   - Driver status changes to "busy"

3. **Cost Calculation:**
   - `precio` = sum of combo prices
   - `costoTransporte` = distance × cost per km (weekday/holiday rate)
   - `iva` = price × 0.13
   - `total` = price + transport cost + IVA

4. **Order Creation:**
   - Initial status: "en preparación"
   - Timestamp recorded: `horaRealizado`

### Order Status Workflow

```
en preparación (Restaurant) → en camino (Restaurant) → entregado (Driver)
       ↓
   suspendido (Restaurant) → en preparación / en camino
```

**Permissions:**
- **Restaurants** can change their own orders between preparation, in-transit, and suspended
- **Drivers** can only mark orders as delivered when in-transit
- On delivery: driver returns to "available", kilometers added to daily total

### Driver Management

**Warning System:**
- Drivers can receive complaints
- Warnings (amonestaciones) count independently
- At 4+ warnings → status changes to "inactive" (cannot receive orders)
- Reset endpoint available to return drivers to "available" status (if < 4 warnings)

---

## Contact

**Developer:** Isaac Méndez  
**Repository:** [https://github.com/isaacmendezr/CletaEatsBackend](https://github.com/isaacmendezr/CletaEatsBackend)

---

**Note:** This project includes pre-loaded test data for immediate testing. Default admin credentials: `cedula: "1"`, `contrasena: "password1234"`
