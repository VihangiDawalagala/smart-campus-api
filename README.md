# 🏫 Smart Campus API — Sensor & Room Management System

**Module:** 5COSC022W — Client-Server Architectures  
**University:** University of Westminster  
**Academic Year:** 2025/26  

---

## 📌 API Overview

This project is a RESTful API developed using **JAX-RS (Jersey)** with an embedded **Grizzly HTTP server**.  
It is designed to manage university campus **Rooms**, **Sensors**, and **Sensor Readings**.

The system supports:
- Room creation, retrieval, and deletion
- Sensor registration with room validation
- Filtering sensors using query parameters
- Sensor readings with history tracking
- Structured error handling using HTTP status codes
- Request and response logging using JAX-RS filters

All data is stored in-memory using Java collections (`ConcurrentHashMap`). No database is used.

---

## 🌐 Base URL
http://localhost:8080/api/v1

📂 API Endpoints
/api/v1
├── GET    /                              → Discovery endpoint
├── GET    /rooms                         → Get all rooms
├── POST   /rooms                         → Create a room
├── GET    /rooms/{roomId}                → Get room by ID
├── DELETE /rooms/{roomId}                → Delete a room
├── GET    /sensors                       → Get all sensors
├── GET    /sensors?type=CO2              → Filter sensors by type
├── POST   /sensors                       → Create sensor
├── GET    /sensors/{sensorId}            → Get sensor by ID
├── GET    /sensors/{sensorId}/readings   → Get reading history
└── POST   /sensors/{sensorId}/readings   → Add a new reading


## 🛠️ Technologies Used

| Technology | Details |
|---|---|
| Java | JDK 11 |
| JAX-RS | Jersey 2.41 |
| HTTP Server | Grizzly (Embedded) |
| JSON Processing | Jackson (via Jersey) |
| Build Tool | Maven |


## 🚀 How to Run the Project

### Step 1 — Clone the Repository
```bash
git clone https://github.com/VihangiDawalagala/smart-campus-api.git
cd smart-campus-api

Step 2 — Build the Project
   Option A — Terminal:    bashmvn clean package
    
   Option B — NetBeans:    Clean and Build Project (Shift + F11)

Step 3 — Run the Server
   Option A — Terminal:    java -jar target/smart-campus-api-1.0-SNAPSHOT.jar

   Option B — NetBeans:    Run Main.java (F6)

Step 4 — Confirm Server is Running
==============================================
 Smart Campus API running at: http://0.0.0.0:8080/api/v1/
 Press ENTER to stop the server.
==============================================

Step 5 — Stop the Server
Press ENTER in the terminal.

🧪 Sample curl Commands

# 1. Discovery endpoint
curl http://localhost:8080/api/v1

# 2. Create a room
curl -X POST http://localhost:8080/api/v1/rooms \
  -H "Content-Type: application/json" \
  -d '{"id":"LIB-301","name":"Library Quiet Study","capacity":50}'

# 3. Get all rooms
curl http://localhost:8080/api/v1/rooms

# 4. Create a sensor
curl -X POST http://localhost:8080/api/v1/sensors \
  -H "Content-Type: application/json" \
  -d '{"id":"CO2-001","type":"CO2","status":"ACTIVE","currentValue":0.0,"roomId":"LIB-301"}'

# 5. Get all sensors
curl http://localhost:8080/api/v1/sensors

# 6. Filter sensors by type
curl "http://localhost:8080/api/v1/sensors?type=CO2"

# 7. Add a sensor reading
curl -X POST http://localhost:8080/api/v1/sensors/CO2-001/readings \
  -H "Content-Type: application/json" \
  -d '{"value":412.5}'

# 8. Get reading history
curl http://localhost:8080/api/v1/sensors/CO2-001/readings

# 9. Delete a room with sensors (returns 409)
curl -X DELETE http://localhost:8080/api/v1/rooms/LIB-301

# 10. Create sensor with invalid roomId (returns 422)
curl -X POST http://localhost:8080/api/v1/sensors \
  -H "Content-Type: application/json" \
  -d '{"id":"FAKE-001","type":"CO2","status":"ACTIVE","currentValue":0.0,"roomId":"FAKE-999"}'

📁 Project Structure
com.smartcampus
├── Main.java
├── SmartCampusApplication.java
├── model
│   ├── Room.java
│   ├── Sensor.java
│   └── SensorReading.java
├── store
│   └── DataStore.java
├── resource
│   ├── DiscoveryResource.java
│   ├── RoomResource.java
│   ├── SensorResource.java
│   └── SensorReadingResource.java
├── filter
│   └── LoggingFilter.java
└── exception
    ├── RoomNotEmptyException.java
    ├── RoomNotEmptyExceptionMapper.java
    ├── LinkedResourceNotFoundException.java
    ├── LinkedResourceNotFoundExceptionMapper.java
    ├── SensorUnavailableException.java
    ├── SensorUnavailableExceptionMapper.java
    └── GlobalExceptionMapper.java

⚠️ Error Handling Summary

| Scenario                      | Exception Class                 | HTTP Status               |
| ----------------------------- | ------------------------------- | ------------------------- |
| Delete room with sensors      | RoomNotEmptyException           | 409 Conflict              |
| Invalid roomId                | LinkedResourceNotFoundException | 422 Unprocessable Entity  |
| Reading on MAINTENANCE sensor | SensorUnavailableException      | 403 Forbidden             |
| Unexpected error              | GlobalExceptionMapper           | 500 Internal Server Error |



📝 Coursework Report — Question Answers

Part 1.1 — JAX-RS Resource Class Lifecycle
Each incoming request in JAX-RS creates a new instance of a resource class. This behaviour is known as per-request scope, meaning resource classes are not treated as singletons. As a result, data stored in resource classes such as RoomResource or SensorResource would vanish once each request was completed.
To address this, I created a distinct DataStore class in my project that serves as the application's shared storage. All data is maintained between requests since this class remains active while the server is operating.
Additionally, since multiple requests occur at the same time, I utilized ConcurrentHashMap rather than HashMap. ConcurrentHashMap ensures that everything operates safely even with concurrent access and helps prevent problems like data corruption.

Part 1.2 — HATEOAS (Hypermedia as the Engine of Application State)
HATEOAS (Hypermedia as the Engine of Application State) is a REST principle where the API provides navigation links in responses that allow clients to dynamically discover available actions.
This project implements a GET /api/v1 discovery endpoint that returns links to the main resources such as /api/v1/rooms and /api/v1/sensors. This allows clients to start at one entry point and navigate the API without hardcoding URLs.
Compared to static documentation:

  - Links are always up to date
  - The API can be explored on the fly by clients
  - Less external documentation required
  - Makes the API self describing


Part 2.1 — Returning IDs vs Full Room Objects
When only IDs are returned, the response sizes are smaller, but the client will have to do additional requests to get full details (the N+1 problem). If you want to get multiple rooms, you would have to make additional requests for each room.
Returning full objects provides all required data in a single response, reducing network calls and simplifying client side logic but increases payload size.
In this project, we return complete Room objects, but relationships such as sensors inside rooms are stored as IDs to avoid deep nested structures and circular references.

Part 2.2 — Is DELETE Idempotent?
Yes, this implementation of DELETE is idempotent. Idempotency means that multiple calls with the same request will have the same effect on the final server's state.
In the project:
The first DELETE request deletes the room and returns 204 No Content
Repeated DELETE requests return 204 No Content even if the room no longer exists
Repeated calls do not modify the server state, satisfying idempotency. No matter how many times the request is sent, the client's intention (the room should not exist) is achieved.

Part 3.1 — @Consumes(APPLICATION_JSON) Mismatch
If a method is specified with @Consumes(MediaType.APPLICATION_JSON) it expects requests in JSON format.
If the client provides a request with a different content type for example text/plain or application/xml, then JAX-RS does content negotiation before invoking the method.
The framework returns 415 Unsupported Media Type automatically if there is no matching method for the content type. This prevents bad or unexpected data formats from reaching the application logic.

Part 3.2 — @QueryParam vs Path Segment for Filtering
Query parameters are more appropriate for filtering because they do not change the identity of the resource.
For example:

/sensors returns all sensors
/sensors?type=CO2 returns filtered sensors

Using a path like /sensors/type/CO2 incorrectly implies that "CO2" is a separate resource.
Advantages of using query parameters:

Optional filtering
Supports multiple filters (e.g., ?type=CO2&status=ACTIVE)
Keeps API design clean and consistent


Part 4.1 — Sub-Resource Locator Pattern Benefits
The Sub-Resource Locator pattern allows nested resources to be handled by separate classes.
In this project, SensorResource delegates requests for /sensors/{sensorId}/readings to a dedicated SensorReadingResource class instead of handling everything in one class.
Benefits include:

   - Separation of concerns
   - Improved code readability and maintainability
   - Avoids large, complex resource classes
   - Easier to extend and scale
   - Allows context-aware operations using sensorId


Part 5.2 — Why HTTP 422 is More Accurate than 404
HTTP 404 Not Found means that the endpoint you requested does not exist.
HTTP 422 Unprocessable Entity means the request is valid but the data is not logically correct.
In this project, when a sensor is created with a roomId that does not exist, the /sensors endpoint is valid, but the reference inside the request body is invalid. Therefore, 422 is more appropriate because the error is related to the data, not the endpoint.

Part 5.4 — Security Risks of Exposing Stack Traces
Revealing stack traces can expose sensitive internal information about your system, including class names, package hierarchies, and framework versions.
This exposes to security risks such as:

   - Revealing system architecture
   - Assisting attackers in finding vulnerabilities
   - Exposing internal logic

GlobalExceptionMapper is used in this project to catch unexpected errors and return a generic 500 internal server error response. The full stack trace will be logged internally on the server side, but not exposed to the client, which enhances security.

Part 5.5 — Why Use JAX-RS Filters for Logging
Because logging is a cross-cutting concern that applies to all API endpoints, using JAX-RS filters for logging is a good approach. You don't need to add logger.info() calls to every resource method, filters allow logging to be handled in one central place and have it apply to all requests and responses automatically.
This helps reduce code duplication and keeps resource classes clean and focused on their primary functionality. It also provides uniform logging for the entire API.
For my project I used a request filter to log the HTTP method and request URI and a response filter to log the status code of every response. This allows for easier expansion and updating of the logging later and improves maintainability.
