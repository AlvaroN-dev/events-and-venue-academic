# TiqueteraCatalogo - Event Management System

[![Java](https://img.shields.io/badge/Java-17-orange)](https://www.oracle.com/java/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.5.7-brightgreen)](https://spring.io/projects/spring-boot)
[![Architecture](https://img.shields.io/badge/Architecture-Hexagonal-blue)](https://en.wikipedia.org/wiki/Hexagonal_architecture_(software))
[![SOLID](https://img.shields.io/badge/Principles-SOLID-purple)](https://en.wikipedia.org/wiki/SOLID)

Event and venue management system implemented with **Hexagonal Architecture** and **SOLID principles**, using Spring Boot 3, JPA/Hibernate, and H2 Database.

---

## 📋 Table of Contents

- [Features](#-features)
- [Architecture](#-architecture)
- [Technologies](#-technologies)
- [Project Structure](#-project-structure)
- [Installation and Execution](#-installation-and-execution)
- [API Endpoints](#-api-endpoints)
- [Swagger Documentation](#-swagger-documentation)
- [Design Principles](#-design-principles)
- [Usage Examples](#-usage-examples)

---

## ✨ Features

- ✅ **Complete CRUD** for Events and Venues
- ✅ **Hexagonal Architecture** (Ports & Adapters)
- ✅ **SOLID principles** applied throughout the code
- ✅ **H2 Database** in-memory
- ✅ **Complete OpenAPI/Swagger documentation**
- ✅ **Validations** with Bean Validation
- ✅ **Centralized and secure error handling**
- ✅ **DTOs** for request/response
- ✅ **Mappers** for layer conversion
- ✅ **Services** for use case orchestration

---

## 🏗️ Architecture

This project implements **Hexagonal Architecture** (also known as Ports and Adapters), which separates business logic from implementation details.

### Main Layers

```
src/main/java/com/codeup/riwi/tiqueteracatalogo/
│
├── 📦 dominio/                    # DOMAIN LAYER
│   ├── models/                    # Pure domain models (framework-free)
│   │   ├── Evento.java
│   │   └── Venue.java
│   ├── ports/                     # Interfaces (contracts)
│   │   ├── in/                    # Input ports (future)
│   │   └── out/                   # Output ports
│   │       ├── EventoRepositoryPort.java
│   │       └── VenueRepositoryPort.java
│   └── excepcion/                 # Domain exceptions
│       └── RecursoNoEncontradoException.java
│
├── 📦 aplicacion/                 # APPLICATION LAYER
│   ├── usecases/                  # Use cases (business logic)
│   │   ├── evento/
│   │   │   ├── CrearEventoUseCase.java
│   │   │   ├── ObtenerEventoUseCase.java
│   │   │   ├── ListarEventosUseCase.java
│   │   │   ├── ActualizarEventoUseCase.java
│   │   │   └── EliminarEventoUseCase.java
│   │   └── venue/
│   │       └── ... (same use cases)
│   ├── services/                  # Services (orchestration)
│   │   ├── EventoService.java
│   │   └── VenueService.java
│   ├── dto/                       # Data Transfer Objects
│   │   ├── EventoRequest.java
│   │   ├── EventoResponse.java
│   │   ├── VenueRequest.java
│   │   └── VenueResponse.java
│   └── mapper/                    # Mappers (DTO ↔ Domain)
│       ├── EventoMapper.java
│       └── VenueMapper.java
│
└── 📦 infraestructura/            # INFRASTRUCTURE LAYER
    ├── controllers/               # REST controllers
    │   ├── EventController.java
    │   ├── VenueController.java
    │   └── advice/                # Exception handlers
    │       ├── GlobalExceptionHandler.java
    │       └── ErrorResponse.java
    ├── adapters/                  # Adapters (implement ports)
    │   ├── EventoRepositoryAdapter.java
    │   └── VenueRepositoryAdapter.java
    ├── repositories/              # JPA repositories
    │   ├── EventoJpaRepository.java
    │   └── VenueJpaRepository.java
    ├── entities/                  # JPA entities
    │   ├── EventoJpaEntity.java
    │   └── VenueJpaEntity.java
    └── config/                    # Configuration
        ├── OpenApiConfig.java
        └── UseCaseConfiguration.java
```

### Data Flow

```
HTTP Request
     ↓
[Controller] ← Input Adapter
     ↓
[Use Case] ← Business Logic (uses Ports)
     ↓
[Repository Port] ← Interface (Output Port)
     ↓
[Repository Adapter] ← Port Implementation
     ↓
[JPA Repository] ← Persistence
     ↓
[H2 Database]
```

---

## 🛠️ Technologies

| Technology | Version | Purpose |
|------------|---------|---------|
| Java | 17 | Programming language |
| Spring Boot | 3.5.7 | Main framework |
| Spring Data JPA | 3.5.7 | Data persistence |
| H2 Database | 2.3.232 | In-memory database |
| Hibernate | 6.6.33 | ORM |
| Springdoc OpenAPI | 2.7.0 | Swagger documentation |
| Lombok | 1.18.36 | Boilerplate reduction |
| Bean Validation | 3.0 | Validations |
| Maven | 3.9+ | Dependency management |

---

## 📁 Project Structure

### Domain (Business Core)

**Features:**
- ✅ No framework dependencies
- ✅ Pure models (POJOs)
- ✅ Defines interfaces (ports)
- ✅ Contains business exceptions

**Example:**
```java
// Pure domain model
public class Evento {
    private Long id;
    private String name;
    private LocalDateTime eventDate;
    // ... without JPA annotations
}

// Port (interface)
public interface EventoRepositoryPort {
    Evento save(Evento evento);
    Optional<Evento> findById(Long id);
    // ...
}
```

### Application (Use Cases)

**Features:**
- ✅ Contains business logic
- ✅ Depends only on ports (interfaces)
- ✅ Framework-independent
- ✅ Services orchestrate use cases

**Example:**
```java
public class CrearEventoUseCase {
    private final EventoRepositoryPort eventoRepository;
    private final VenueRepositoryPort venueRepository;
    
    public Evento ejecutar(Evento evento) {
        // Business validation
        if (!venueRepository.existsById(evento.getVenueId())) {
            throw new IllegalArgumentException("The venue does not exist");
        }
        return eventoRepository.save(evento);
    }
}
```

### Infrastructure (Implementation Details)

**Features:**
- ✅ Implements ports
- ✅ Contains technical details (JPA, REST, etc.)
- ✅ Interchangeable adapters
- ✅ Controllers, Repositories, Entities

**Example:**
```java
@Component
public class EventoRepositoryAdapter implements EventoRepositoryPort {
    private final EventoJpaRepository jpaRepository;
    
    @Override
    public Evento save(Evento evento) {
        EventoJpaEntity entity = toEntity(evento);
        EventoJpaEntity saved = jpaRepository.save(entity);
        return toDomain(saved);
    }
}
```

---

## 🚀 Installation and Execution

### Prerequisites

- Java 17 or higher
- Maven 3.9+ (included in the project as `mvnw`)

### Steps

1. **Clone the repository**
```bash
git clone https://github.com/your-user/TiqueteraCatalogo.git
cd TiqueteraCatalogo
```

2. **Compile the project**
```bash
./mvnw clean compile
```

3. **Run the application**
```bash
./mvnw spring-boot:run
```

4. **Verify it's running**
- Application: http://localhost:8080
- Swagger UI: http://localhost:8080/swagger-ui.html
- H2 Console: http://localhost:8080/h2-console

---

## 📡 API Endpoints

### Events (`/api/events`)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/events` | List all events |
| GET | `/api/events/{id}` | Get event by ID |
| GET | `/api/events/venue/{venueId}` | List events by venue |
| POST | `/api/events` | Create new event |
| PUT | `/api/events/{id}` | Update event |
| DELETE | `/api/events/{id}` | Delete event |

### Venues (`/api/venues`)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/venues` | List all venues |
| GET | `/api/venues/{id}` | Get venue by ID |
| POST | `/api/venues` | Create new venue |
| PUT | `/api/venues/{id}` | Update venue |
| DELETE | `/api/venues/{id}` | Delete venue |

---

## 📚 Swagger Documentation

Access interactive documentation at: **http://localhost:8080/swagger-ui.html**

Documentation features:
- ✅ Request/response examples
- ✅ Detailed schemas
- ✅ HTTP response codes
- ✅ Documented validations
- ✅ Live testing ("Try it out")

---

## 🎯 Design Principles

### Hexagonal Architecture ✅

1. **Domain at the center**: Business logic doesn't depend on frameworks
2. **Ports**: Interfaces that define contracts
3. **Adapters**: Interchangeable implementations
4. **Dependency inversion**: Infrastructure depends on domain

### SOLID Principles ✅

#### 1. Single Responsibility Principle (SRP)
Each class has a single responsibility:
- `CrearEventoUseCase`: Only create events
- `EventoRepositoryAdapter`: Only adapt persistence
- `EventController`: Only handle HTTP

#### 2. Open/Closed Principle (OCP)
Open for extension, closed for modification:
```java
// We can add new adapters without modifying use cases
public class EventoMongoAdapter implements EventoRepositoryPort { }
public class EventoRedisAdapter implements EventoRepositoryPort { }
```

#### 3. Liskov Substitution Principle (LSP)
Adapters are interchangeable:
```java
EventoRepositoryPort repo = new EventoRepositoryAdapter();  // JPA
EventoRepositoryPort repo = new EventoMongoAdapter();       // MongoDB
// The use case works with either
```

#### 4. Interface Segregation Principle (ISP)
Specific and cohesive interfaces:
```java
public interface EventoRepositoryPort { /* only event methods */ }
public interface VenueRepositoryPort { /* only venue methods */ }
```

#### 5. Dependency Inversion Principle (DIP)
Dependencies on abstractions:
```java
public class CrearEventoUseCase {
    private final EventoRepositoryPort repository;  // ✅ Interface
    // NOT: private final EventoRepositoryAdapter repository;  // ❌ Implementation
}
```

---

## 💡 Usage Examples

### 1. Create a Venue

**Request:**
```bash
POST http://localhost:8080/api/venues
Content-Type: application/json

{
  "name": "National Theater",
  "address": "71st Street #10-25",
  "city": "Bogotá",
  "country": "Colombia",
  "capacity": 1500
}
```

**Response (201 Created):**
```json
{
  "id": 1,
  "name": "National Theater",
  "address": "71st Street #10-25",
  "city": "Bogotá",
  "country": "Colombia",
  "capacity": 1500
}
```

### 2. Create an Event

**Request:**
```bash
POST http://localhost:8080/api/events
Content-Type: application/json

{
  "name": "Rock Concert",
  "description": "Great rock concert",
  "eventDate": "2025-12-15T20:00:00",
  "categoria": "Music",
  "venueId": 1,
  "capacity": 1000,
  "price": 80000.0
}
```

**Response (201 Created):**
```json
{
  "id": 1,
  "name": "Rock Concert",
  "description": "Great rock concert",
  "eventDate": "2025-12-15T20:00:00",
  "categoria": "Music",
  "venueId": 1,
  "capacity": 1000,
  "price": 80000.0
}
```

### 3. List Events by Venue

**Request:**
```bash
GET http://localhost:8080/api/events/venue/1
```

**Response (200 OK):**
```json
[
  {
    "id": 1,
    "name": "Rock Concert",
    "eventDate": "2025-12-15T20:00:00",
    "venueId": 1,
    ...
  }
]
```

### 4. Error Handling

**Error 404 - Resource not found:**
```json
{
  "timestamp": "2025-11-25T10:00:00",
  "status": 404,
  "error": "Not Found",
  "message": "Event with ID 999 not found",
  "path": "/api/events/999"
}
```

**Error 400 - Validation:**
```json
{
  "timestamp": "2025-11-25T10:00:00",
  "status": 400,
  "error": "Bad Request",
  "message": "Validation failed",
  "path": "/api/events",
  "details": {
    "name": "Event name is required",
    "eventDate": "Event date is required"
  }
}
```

**Error 500 - Internal error (generic message for security):**
```json
{
  "timestamp": "2025-11-25T10:00:00",
  "status": 500,
  "error": "Internal Server Error",
  "message": "An internal error has occurred. Please contact the administrator.",
  "path": "/api/events"
}
```

---

## 🔒 Security

- ✅ **Generic error messages**: SQL details and stack traces are not exposed
- ✅ **Validations**: Bean Validation on all DTOs
- ✅ **Centralized exception handling**: `GlobalExceptionHandler`

---

## 📝 Configuration

### H2 Database

The application uses H2 in-memory. Configuration in `application.properties`:

```properties
spring.datasource.url=jdbc:h2:mem:tiqueteradb
spring.datasource.driverClassName=org.h2.Driver
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```

**Access H2 Console:**
- URL: http://localhost:8080/h2-console
- JDBC URL: `jdbc:h2:mem:tiqueteradb`
- User: `sa`
- Password: (empty)

---

## 🧪 Testing

### Test with cURL

```bash
# Create venue
curl -X POST http://localhost:8080/api/venues \
  -H "Content-Type: application/json" \
  -d '{"name":"Theater","address":"1st Street","city":"Bogotá","country":"Colombia","capacity":500}'

# Create event
curl -X POST http://localhost:8080/api/events \
  -H "Content-Type: application/json" \
  -d '{"name":"Concert","description":"Show","eventDate":"2025-12-15T20:00:00","categoria":"Music","venueId":1,"capacity":500,"price":50000}'

# List events
curl http://localhost:8080/api/events
```

### Test with Swagger UI

1. Go to http://localhost:8080/swagger-ui.html
2. Select an endpoint
3. Click "Try it out"
4. Fill in the example JSON
5. Click "Execute"

---

## 📊 Benefits of this Architecture

### Maintainability
- Organized and easy-to-understand code
- Clear responsibilities
- Localized changes

### Testability
- Easy to create unit tests with mocks
- Independent use cases
- Ports allow injecting fake implementations

### Flexibility
- Easy to switch from JPA to MongoDB
- Easy to add new adapters
- Protected business logic

### Scalability
- Decoupled components
- Easy to add new features
- Architecture ready for microservices

---

## 👥 Author

**Tiquetera Team**
- Email: soporte@tiquetera.com

---

## 📄 License

This project is licensed under Apache 2.0 - see the [LICENSE](LICENSE) file for details.

---

## 🔗 Useful Links

- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [Hexagonal Architecture](https://en.wikipedia.org/wiki/Hexagonal_architecture_(software))
- [SOLID Principles](https://en.wikipedia.org/wiki/SOLID)
- [OpenAPI Specification](https://swagger.io/specification/)

---

**Questions or suggestions?** Open an issue on GitHub or contact the development team.
