# w1974413-smart-campus-jaxrs-api

## API Overview
This project is a RESTful API built in Java using JAX-RS for a Smart Campus scenario. The API is designed to manage rooms across a university campus, the sensors assigned to those rooms, and the historical readings produced by those sensors.

The system uses in-memory Java data structures rather than a database. The API supports room creation, room retrieval, room deletion with safety checks, sensor registration, sensor filtering by type, nested sensor reading history, custom exception handling, and request/response logging.

The main resources in the API are:

- **Room**
- **Sensor**
- **SensorReading**

The base path for the API is:

`http://localhost:8080/api/v1`

## Build and Run

1. Open the project in NetBeans.
2. Allow Maven to download all required dependencies.
3. Run the `Main` class.
4. The server should start locally.
5. The API will be available at:

`http://localhost:8080/api/v1`

## Sample curl Commands
1. Discovery Endpoint - curl -X GET http://localhost:8080/api/v1
   
2. Creating a Room:
curl -X POST http://localhost:8080/api/v1/rooms \
-H "Content-Type: application/json" \
-d "{\"id\":\"LIB-301\",\"name\":\"Library Quiet Study\",\"capacity\":40}"

3. Create a sensor:
curl -X POST http://localhost:8080/api/v1/sensors \
-H "Content-Type: application/json" \
-d "{\"id\":\"CO2-01\",\"type\":\"CO2\",\"status\":\"ACTIVE\",\"currentValue\":0.0,\"roomId\":\"LIB-301\"}"

4. Filter sensors by type:
curl -X GET "http://localhost:8080/api/v1/sensors?type=CO2"

5. Add a reading to a sensor:
curl -X POST http://localhost:8080/api/v1/sensors/CO2-01/readings \
-H "Content-Type: application/json" \
-d "{\"value\":650.5}"

## Coursework Questions

### Part 1.1
JAX-RS resource classes are request-scoped by default, so a new instance is created for each request. This means instance fields can't reliably store shared data and anything saved there would be lost between requests. To solve this, I created a separate DataStore class using in-memory collections like maps and lists. Data persists there for as long as the server runs. I also used thread-safe collections to avoid race conditions when multiple requests access shared state at the same time.

### Part 1.2
Hypermedia makes an API self-describing. Rather than forcing clients to know every endpoint in advance, the API returns links that show what is available and where to go next. My discovery endpoint returns basic metadata alongside the main resource paths. A developer can follow the responses to navigate the API without relying entirely on external documentation. It also makes things more adaptable so if paths change later, clients guided by the API can adjust easily than ones with everything hardcoded.

### Part 2.1
Returning only IDs keeps responses lightweight but forces the client to make extra requests for room details. Returning full objects is heavier but far more convenient beacuse the client gets everything in one go. For this project, full objects made more sense. It keeps the API simple and testing straightforward. In a larger system, returning partial data would probably be the better call.

### Part 2.2
Yes, DELETE is idempotent in my implementation. The first call removes the room and any subsequent call returns a 404 because the room no longer exists.
The response differs but the system state doesn't keep changing so the room is gone either way.

### Part 3.1
This annotation tells JAX-RS the method expects a JSON request body. If a client sends a different format, the framework can't map it to the expected Java object and rejects the request usually with a 415 Unsupported Media Type response. It enforces a clear contract so the server only processes what it's built to handle.

### Part 3.2
Using /sensors?type=CO2 is cleaner because the client is still requesting the same collection. Putting the type in the path like /sensors/type/CO2 makes it look like a separate resource entirely. Query parameters are also more flexible therfore if you want to filter by multiple criteria later, they combine naturally.

### Part 4.1
This pattern separates distinct responsibilities into their own classes. SensorResource handles sensor-level operations, while SensorReadingResource deals specifically with readings. Putting everything in one class would make it bloated and hard to maintain. Splitting by sub-resource keeps things organised and easier to extend, for readings especially, it captures the relationship to the sensor without cluttering the main class.

### Part 5.2
The issue isn't with the URL, it's with the data in the request body. So when a client sends valid JSON but references a roomId that doesn't exist, the server understands the request but can't fulfil it because the referenced room is missing. A 404 implies the endpoint wasn't found, which isn't the case here. A 422 communicates the actual problem far more accurately.

### Part 5.4
Stack traces can reveal package names, class names, method names, and line numbers which is enough for an attacker to start mapping the application's internal structure and look for weak points. I implemented a global exception mapper that returns a generic 500 message instead to inform the client, without exposing how the server actually works.

### Part 5.5
Logging applies to the whole API, not any specific piece of business logic. Writing Logger.info() calls in every resource method means repeating the same code constantly and risking inconsistent coverage if an endpoint gets missed. Filters centralise everything in one place.
