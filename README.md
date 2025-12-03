# TiqueteraCatalogo

Events and Venues Catalog (Spring Boot 3) with in-memory storage, OpenAPI/Swagger documentation, and layered architecture Controller → Service → Repository.

## Table of Contents
- [Overview](#overview)
- [Architecture](#architecture)
- [How to Run](#how-to-run)
- [Swagger / OpenAPI](#swagger--openapi)
- [Endpoints](#endpoints)
- [Error Handling](#error-handling)
- [Classes and Code Explained Line by Line](#classes-and-code-explained-line-by-line)
  - [TiqueteraCatalogoApplication](#tiqueteracatalogoapplication)
  - [OpenApiConfig](#openapiconfig)
  - [DTOs](#dtos)
    - [EventDTO](#eventdto)
    - [VenueDTO](#venuedto)
  - [Repositories (in-memory simulation)](#repositories-in-memory-simulation)
    - [EventRepository](#eventrepository)
    - [VenueRepository](#venuerepository)
  - [Services](#services)
    - [EventService](#eventservice)
    - [VenueService](#venueservice)
  - [Controllers](#controllers)
    - [EventController](#eventcontroller)
    - [VenueController](#venuecontroller)
  - [Exceptions and Errors](#exceptions-and-errors)
    - [ErrorResponse](#errorresponse)
    - [GlobalExceptionHandler](#globalexceptionhandler)
    - [ResourceNotFoundException](#resourcenotfoundexception)
- [Quick Tests with cURL](#quick-tests-with-curl)
- [Notes and Future Improvements](#notes-and-future-improvements)

## Overview
Java 17 project with Spring Boot 3.5.7. Implements:
- Complete CRUD for Events and Venues.
- In-memory storage during runtime (no database).
- OpenAPI/Swagger documentation with examples and schemas.
- Global error handling (400/404/500) and validations (Bean Validation).
- Layered architecture (Controller → Service → Repository).

## Architecture
```
controller/   → Receives HTTP, validates, maps HTTP codes and documents Swagger.
service/      → Business logic, orchestrates repositories.
repository/   → Simulates persistence in ArrayList, generates IDs (AtomicLong).
DTO/          → Validated and documented transfer models.
exception/    → Exceptions, error model and global handler.
config/       → OpenAPI/Swagger configuration.
```

## How to Run
Requirements: Java 17. From PowerShell on Windows:

```powershell
# Compile
cd c:\Users\anonimo\Videos\TiqueteraCatalogo\TiqueteraCatalogo
./mvnw.cmd clean compile

# Run
./mvnw.cmd spring-boot:run
```

- App: http://localhost:8080
- Swagger UI: http://localhost:8080/swagger-ui.html
- OpenAPI JSON: http://localhost:8080/v3/api-docs

> If PowerShell doesn't recognize `./mvnw.cmd`, use:
> ```powershell
> & ".\mvnw.cmd" spring-boot:run
> ```

## Swagger / OpenAPI
The `config/OpenApiConfig.java` class defines metadata (title, version, contact, license) and servers. Controllers use `@Operation`, `@ApiResponses`, `@Parameter` annotations and examples for request/response. DTOs have `@Schema` per field.

## Endpoints
Events `/api/events` and Venues `/api/venues` with operations: GET (all, by id), POST, PUT, DELETE and search by relationship (events by venue).

## Error Handling
- 400 Bad Request: validation and incorrect types.
- 404 Not Found: resource not found.
- 500 Internal Server Error: generic.

Unified `ErrorResponse` format with `timestamp`, `status`, `error`, `message`, `path` and optional `details`.

---

## Classes and Code Explained Line by Line

> Note: For brevity, imports are explained as a complete block. Line-by-line detail focuses on the body of classes and methods.

### TiqueteraCatalogoApplication
File: `src/main/java/.../TiqueteraCatalogoApplication.java`
```java
@SpringBootApplication          // Enables auto-configuration, component scanning and configuration
public class TiqueteraCatalogoApplication {

    public static void main(String[] args) {
        SpringApplication.run(TiqueteraCatalogoApplication.class, args); // Starts the Spring Boot app
    }
}
```

### OpenApiConfig
File: `config/OpenApiConfig.java`
```java
@Configuration                       // Marks as Spring configuration class
@Bean                                // Exposes an OpenAPI bean for springdoc
public OpenAPI customOpenAPI() {
    return new OpenAPI()
        .info(new Info()
            .title("Events and Venues Catalog API") // Title displayed in Swagger UI
            .version("1.0.0")                       // API version
            .description("REST API to manage catalog...") // General description
            .contact(new Contact().name("Tiquetera Team").email("soporte@tiquetera.com"))
            .license(new License().name("Apache 2.0").url("https://www.apache.org/licenses/LICENSE-2.0.html")))
        .servers(List.of(                           // List of servers published in the doc
            new Server().url("http://localhost:8080").description("Development Server"),
            new Server().url("http://localhost:8081").description("Testing Server")));
}
```

### DTOs
#### EventDTO
File: `DTO/EventDTO.java`
```java
@Schema(description = "Event information")                      // Documents the resource schema
public class EventDTO {
    @Schema(..., accessMode = READ_ONLY) private Long id;        // Auto-generated ID by repo (read-only)

    @NotBlank(message = "The name...")                           // Validation: required
    @Schema(description = "Name...", example = "Rock Concert 2025", required = true)
    private String name;

    @Schema(description = "Description...")
    private String description;                                   // Optional descriptive text

    @NotNull(message = "The date...")                            // Validation: required
    @Schema(description = "Date and time...", example = "2025-12-15T20:00:00", required = true)
    private LocalDateTime eventDate;

    @NotNull(message = "The venue ID...")                        // Validation: required
    @Schema(description = "Venue ID...", example = "1", required = true)
    private Long venueId;

    @Positive(message = "The capacity...")                       // Validation: > 0
    @Schema(description = "Maximum capacity...", example = "1000", required = true)
    private Integer capacity;

    @Positive(message = "The price...")                          // Validation: > 0
    @Schema(description = "Ticket price", example = "80000.00", required = true)
    private Double price;

    // Standard Getters/Setters for JSON serialization and binding
}
```

#### VenueDTO
File: `DTO/VenueDTO.java`
```java
@Schema(description = "Venue/location information")
public class VenueDTO {
    @Schema(..., accessMode = READ_ONLY) private Long id;        // Auto-generated ID by repo

    @NotBlank(message = "The name...")
    @Schema(description = "Venue name", example = "National Theater", required = true)
    private String name;

    @NotBlank(message = "The address...")
    @Schema(description = "Address...", example = "71st Street #10-25", required = true)
    private String address;

    @NotBlank(message = "The city...")
    @Schema(description = "City...", example = "Bogotá", required = true)
    private String city;

    @NotBlank(message = "The country...")
    @Schema(description = "Country...", example = "Colombia", required = true)
    private String country;

    @Positive(message = "The capacity...")
    @Schema(description = "Maximum capacity...", example = "1500", required = true)
    private Integer capacity;
}
```

### Repositories (in-memory simulation)
#### EventRepository
File: `repository/EventRepository.java`
```java
@Repository                                        // Detected as data access bean
public class EventRepository {
    private final List<EventDTO> events = new ArrayList<>();     // In-memory storage
    private final AtomicLong idGenerator = new AtomicLong(1);    // Thread-safe ID sequence

    public List<EventDTO> findAll() {                            // Read all
        return new ArrayList<>(events);                          // Defensive copy
    }

    public Optional<EventDTO> findById(Long id) {                // Search by ID
        return events.stream().filter(e -> e.getId().equals(id)).findFirst();
    }

    public List<EventDTO> findByVenueId(Long venueId) {          // Query by relationship
        return events.stream().filter(e -> e.getVenueId().equals(venueId)).toList();
    }

    public EventDTO save(EventDTO event) {                       // Insertion
        if (event.getId() == null) event.setId(idGenerator.getAndIncrement());
        events.add(event);
        return event;
    }

    public EventDTO update(EventDTO event) {                      // In-place update
        return findById(event.getId()).map(db -> {
            db.setName(event.getName());
            db.setDescription(event.getDescription());
            db.setEventDate(event.getEventDate());
            db.setVenueId(event.getVenueId());
            db.setCapacity(event.getCapacity());
            db.setPrice(event.getPrice());
            return db;                                           // Returns updated reference
        }).orElse(null);                                         // Simple convention for this example
    }

    public boolean deleteById(Long id) {                          // Deletion by ID
        return events.removeIf(e -> e.getId().equals(id));
    }

    public boolean existsById(Long id) {                          // Existence check
        return events.stream().anyMatch(e -> e.getId().equals(id));
    }

    public long count() { return events.size(); }                 // Total count
}
```

#### VenueRepository
File: `repository/VenueRepository.java` (same pattern as `EventRepository`)

### Services
#### EventService
File: `service/EventService.java`
```java
@Service                                         // Service bean (business logic)
public class EventService {
    private final EventRepository eventRepository;               // Constructor injection

    public EventService(EventRepository eventRepository) {
        this.eventRepository = eventRepository;                  // Assigns dependency
    }

    public List<EventDTO> getAllEvents() { return eventRepository.findAll(); }
    public Optional<EventDTO> getEventById(Long id) { return eventRepository.findById(id); }
    public EventDTO createEvent(EventDTO dto) { return eventRepository.save(dto); }

    public Optional<EventDTO> updateEvent(Long id, EventDTO dto) {
        if (!eventRepository.existsById(id)) return Optional.empty();
        dto.setId(id);                                           // Ensures idempotency by ID
        return Optional.ofNullable(eventRepository.update(dto)); // Can be null if doesn't exist
    }

    public boolean deleteEvent(Long id) { return eventRepository.deleteById(id); }
    public List<EventDTO> getEventsByVenueId(Long venueId) { return eventRepository.findByVenueId(venueId); }
}
```

#### VenueService
File: `service/VenueService.java` (identical pattern to `EventService`).

### Controllers
#### EventController
File: `controller/EventController.java`
```java
@RestController                              // Marks REST controller (JSON by default)
@RequestMapping("/api/events")               // Prefix for all endpoints
@Tag(name = "Events", description = "API for event management") // Groups in Swagger
public class EventController {
    private final EventService eventService; // Service injection

    public EventController(EventService eventService) { this.eventService = eventService; }

    @Operation(summary = "Get all...", description = "Returns complete list...")
    @ApiResponses({ @ApiResponse(responseCode = "200", ... ) })
    @GetMapping
    public ResponseEntity<List<EventDTO>> getAllEvents() {
        return ResponseEntity.ok(eventService.getAllEvents());
    }

    @Operation(summary = "Get event by ID", description = "...")
    @ApiResponses({
      @ApiResponse(responseCode = "200", ...),
      @ApiResponse(responseCode = "404", content = @Content(schema = @Schema(implementation = ErrorResponse.class)))
    })
    @GetMapping("/{id}")
    public ResponseEntity<EventDTO> getEventById(@Parameter(...) @PathVariable Long id) {
        return eventService.getEventById(id)
            .map(ResponseEntity::ok)
            .orElseThrow(() -> new ResourceNotFoundException("Event", id)); // Throws 404 if not exists
    }

    @Operation(summary = "Create new event", description = "...")
    @ApiResponses({
      @ApiResponse(responseCode = "201", ...),
      @ApiResponse(responseCode = "400", content = @Content(schema = @Schema(implementation = ErrorResponse.class)))
    })
    @PostMapping
    public ResponseEntity<EventDTO> createEvent(@Valid @RequestBody EventDTO eventDTO) {
        EventDTO created = eventService.createEvent(eventDTO);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }

    @Operation(summary = "Update event", description = "...")
    @PutMapping("/{id}")
    public ResponseEntity<EventDTO> updateEvent(@PathVariable Long id, @Valid @RequestBody EventDTO eventDTO) {
        return eventService.updateEvent(id, eventDTO)
            .map(ResponseEntity::ok)
            .orElseThrow(() -> new ResourceNotFoundException("Event", id));
    }

    @Operation(summary = "Delete event", description = "...")
    @ApiResponses({ @ApiResponse(responseCode = "204", description = "Event deleted") })
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteEvent(@PathVariable Long id) {
        if (!eventService.deleteEvent(id)) throw new ResourceNotFoundException("Event", id);
        return ResponseEntity.noContent().build();
    }

    @Operation(summary = "Get events by venue", description = "...")
    @GetMapping("/venue/{venueId}")
    public ResponseEntity<List<EventDTO>> getEventsByVenueId(@PathVariable Long venueId) {
        return ResponseEntity.ok(eventService.getEventsByVenueId(venueId));
    }
}
```

#### VenueController
File: `controller/VenueController.java` (same pattern as `EventController`).

### Exceptions and Errors
#### ErrorResponse
File: `exception/ErrorResponse.java`
```java
@Schema(description = "Standard API error response")
public class ErrorResponse {
    private LocalDateTime timestamp = LocalDateTime.now();  // Timestamp when constructed
    private int status;                                     // HTTP code (400/404/500)
    private String error;                                   // Short text: Bad Request/Not Found/...
    private String message;                                 // Readable message of the problem
    private String path;                                    // Endpoint that failed
    private List<String> details;                           // Details (e.g. invalid fields)

    // Standard constructors + getters/setters
}
```

#### GlobalExceptionHandler
File: `exception/GlobalExceptionHandler.java`
```java
@RestControllerAdvice                             // Global exception catcher
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationExceptions(...) {
        // Iterates FieldError and builds "field: message" list
        // Returns 400 with ErrorResponse + details
    }

    @ExceptionHandler(MethodArgumentTypeMismatchException.class)
    public ResponseEntity<ErrorResponse> handleTypeMismatch(...) {
        // Incorrect type param in path/query → 400 with message "parameter X must be of type Y"
    }

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFound(...) {
        // Returns 404 with exception message
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(...) {
        // Unexpected failure → 500 generic message
    }
}
```

#### ResourceNotFoundException
File: `exception/ResourceNotFoundException.java`
```java
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) { super(message); }
    public ResourceNotFoundException(String resource, Long id) {
        super(String.format("%s with ID %d not found", resource, id));
    }
}
```

---

## Quick Tests with cURL

### 1. Create a Venue first (required for events)
```bash
curl -X POST http://localhost:8080/api/venues \
  -H "Content-Type: application/json" \
  -d '{"name":"National Theater","address":"71st Street #10-25","city":"Bogotá","country":"Colombia","capacity":1500}'
```
**Expected response (201 Created):**
```json
{"id":1,"name":"National Theater","address":"71st Street #10-25","city":"Bogotá","country":"Colombia","capacity":1500}
```

### 2. Create an Event (uses venueId=1 from previous step)
```bash
curl -X POST http://localhost:8080/api/events \
  -H "Content-Type: application/json" \
  -d '{"name":"Rock Concert","description":"Great rock show","eventDate":"2025-12-15T20:00:00","venueId":1,"capacity":1000,"price":80000}'
```
**Expected response (201 Created):**
```json
{"id":1,"name":"Rock Concert","description":"Great rock show","eventDate":"2025-12-15T20:00:00","venueId":1,"capacity":1000,"price":80000.0}
```

### 3. List all resources
```bash
# List events
curl http://localhost:8080/api/events

# List venues
curl http://localhost:8080/api/venues
```

### 4. Get by ID (now that they exist)
```bash
curl http://localhost:8080/api/events/1
curl http://localhost:8080/api/venues/1
```

### 5. Update a Venue (PUT with ALL fields)
```bash
curl -X PUT http://localhost:8080/api/venues/1 \
  -H "Content-Type: application/json" \
  -d '{"name":"Renovated National Theater","address":"71st Street #10-25","city":"Bogotá","country":"Colombia","capacity":2000}'
```
**⚠️ IMPORTANT:** PUT requires all mandatory fields (`name`, `address`, `city`, `country`, `capacity`).

### 6. Update an Event
```bash
curl -X PUT http://localhost:8080/api/events/1 \
  -H "Content-Type: application/json" \
  -d '{"name":"Updated Rock Concert","description":"Renovated show","eventDate":"2025-12-15T21:00:00","venueId":1,"capacity":1200,"price":90000}'
```

### 7. Delete resources
```bash
curl -X DELETE http://localhost:8080/api/events/1
curl -X DELETE http://localhost:8080/api/venues/1
```
**Expected response:** 204 No Content (no body)

### 8. Search events by venue
```bash
curl http://localhost:8080/api/events/venue/1
```

## ❌ Common Errors and How to Avoid Them

### Error 404: "Event with ID X not found"
**Cause:** You're trying to access/update/delete a resource that doesn't exist.
```bash
# ❌ WRONG - ID 999 doesn't exist
curl http://localhost:8080/api/events/999
```
**Solution:** Verify that the resource exists first with GET /api/events

### Error 400: Validation failed
**Cause 1:** Missing mandatory fields in PUT
```bash
# ❌ WRONG - Missing mandatory fields
curl -X PUT http://localhost:8080/api/venues/1 \
  -H "Content-Type: application/json" \
  -d '{"name":"Only name"}'
```
**Solution:** Send ALL required fields.

**Cause 2:** Invalid values
```bash
# ❌ WRONG - negative capacity
curl -X POST http://localhost:8080/api/venues \
  -H "Content-Type: application/json" \
  -d '{"name":"Theater","address":"1st Street","city":"Bogotá","country":"Colombia","capacity":-100}'
```
**Solution:** Use positive values for `capacity` and `price`.

**Cause 3:** Empty fields
```bash
# ❌ WRONG - empty name
curl -X POST http://localhost:8080/api/venues \
  -H "Content-Type: application/json" \
  -d '{"name":"","address":"1st Street","city":"Bogotá","country":"Colombia","capacity":500}'
```
**Solution:** Don't send empty strings in `@NotBlank` fields.

## 🔍 View Error Responses

### Example 404 error:
```json
{
  "timestamp": "2025-10-28T11:04:57",
  "status": 404,
  "error": "Not Found",
  "message": "Event with ID 1 not found",
  "path": "/api/events/1"
}
```

### Example 400 error (validation):
```json
{
  "timestamp": "2025-10-28T11:11:26",
  "status": 400,
  "error": "Bad Request",
  "message": "Validation error in submitted data",
  "path": "/api/venues/1",
  "details": [
    "address: Address is required",
    "country: Country is required",
    "city: City is required"
  ]
}
```

## 💡 Recommendation: Use Swagger UI

Instead of cURL, use Swagger UI for easier testing:
1. Go to http://localhost:8080/swagger-ui.html
2. Expand an endpoint (e.g.: POST /api/venues)
3. Click on "Try it out"
4. Fill in the example JSON
5. Click on "Execute"
6. See the response immediately with the HTTP code

Swagger validates data before sending and shows clear errors.

## Notes and Future Improvements
- Real persistence with JPA/Hibernate and database.
- Pagination, sorting and filters.
- Unit and integration tests.
- Security (Spring Security, JWT).
- Additional validations (date range, positive price with BigDecimal)
# events-and-venue-academic