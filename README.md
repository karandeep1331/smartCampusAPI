# 5COSC022W Client-Server Architectures Coursework 

Name: Karandeep Singh Jalf

Student ID: W2074910

# Video Link



# overview
I have Developing a API for a Smart Campus system using Java and JAX-RS, the application manages resources such as rooms and sensors through HTTP methods like GET and POST. It requires setting up a Maven project, creating model and resource classes, and ensuring proper handling of requests with suitable responses and validation. And testing the system of the GET and POST with postman application.

# Discovery endpoint
```bash
curl -X GET http://localhost:8080/SmartcampusAPI-CSA/api/v1
```
# Create a room
```bash
curl -X POST http://localhost:8080/smartCampusAPI-CSA/api/v1/rooms
-H "Content-Type: application/json" \
-d '{ "id": "LIB-301","name": "Library Quiet Study", "capacity": 100 }'
```
# Get all rooms
```bash
curl -X GET http://localhost:8080/smartCampusAPI-CSA/api/v1/rooms
```
# Get room by ID
```bash
curl -X GET http://localhost:8080/SmartCampusAPI-CSA/api/v1/rooms/LIB-301
```

# Delete a room
```bash
curl -X DELETE http://localhost:8080/SmartCampusAPI-CSA/api/v1/rooms/LIB-301
```
# Create a sensor
```bash
curl -X POST http://localhost:8080/smartCampusAPI-CSA/api/v1/sensors
-H "Content-Type: application/json" \
-d '{"id": "LABSEN-01", "type": "Temperature","status": "ACTIVE","currentValue": 22.5,
  "roomId": "LAB-01"}'
```
# Get all sensors
```bash
curl -X GET http://localhost:8080/smartCampusAPI-CSA/api/v1/sensors
```
# Filter Sensor
```bash
curl -X  GET http://localhost:8080/SmartCampusAPI-CSA/api/v1/sensors?type=Temperature
```
# Add Sensor Reading

```bash
curl -X  POST http://localhost:8080/SmartCampusAPI-CSA/api/v1/sensors/LABSEN-01/readings
-H "Content-Type: application/json" \
-d '{"id":reading-001, "reading-002","timestamp": 1710001200000,"value": 25.4}'
```
# Get Sensor Reading
```bash
curl -X GET http://localhost:8080/SmartCampusAPI-CSA/api/v1/sensors/LABSEN-01/readings
```




# Report
# Part 1: Service Architecture & setup

Q1.1 Project & Application Configuration

Resource classes in JAX-RS is a per-request lifecycle,a new object will be instantiated on each request rather than maintaining a single global object throughout the application. This approach eliminates many possible problems associated with multithreading since each request is processed by its object and not affects any other request directly.In the majority of applications, shared data structures such as maps or lists ( containing rooms or sensor values) are employed. As a result, the application may handle multiple requests at once, and more than one user may attempt to read or modify a shared piece of information simultaneously. In this case, the issue of race condition arises, and you need to take appropriate measures to protect your data against potential overwriting and inconsistency. Using thread-safe implementations of collections such as ConcurrentHashMap should be considered or applying proper synchronization techniques.

Q1.2 The ”Discovery” Endpoint

Hypermedia is considered a hallmark of advanced RESTful design because it transforms an API from a set of static endpoints into a dynamic, navigable system where the server actively guides the client’s next possible actions through links included in responses. This approach benefits client developers by providing a self-discoverable and always up-to-date view of the API, reducing the need to rely on static documentation that can quickly become outdated. Instead of hardcoding URLs, clients can follow the links provided by the server, making the system more flexible and easier to work with.

# Part 2: Room Management

Q2.1 Room Resource Implementation

By returning just the IDs of the rooms, it will be able to minimise the network bandwidth as the message will be much shorter, which would make the API more efficient. The downside is that this method will make the client request other messages to get other information on the rooms. If it return the full objects of the rooms, then all the necessary information will already be provided to the client with one response message, which makes it easier for the user since no other API requests will be needed.

Q2.2 Room Deletion & Safety Logic

The DELETE command calls result in the same final state of the system.  When the client makes a DELETE call to delete a particular room DELETE /rooms/{roomId}, the system first checkes if their the room has any sensors attached to it. In case when sensors are attached, the system does not allow the room to be deleted of the room and returns a 409 Conflict error code. However, when there are no sensors attached, the room is successfully deleted from the system. In case when the same DELETE request is made for the same room, but the room was previously deleted, then the system does nothing and returns a 404 Not Found error message

# Part 3: Sensor Operations & Linking

Q3.1 Sensor Resource & Integrity

The @Consumes(MediaType.APPLICATION_JSON) annotation specifies that the POST endpoint only accepts request bodies in JSON format. If a client sends data in a different format, such as text/plain or application/xml, the request is automatically rejected with an HTTP 415 Unsupported Media Type response. This occurs before the method logic is executed, as JAX-RS cannot match the request to a method that supports the given media type. JAX-RS uses message body readers to convert incoming data into Java objects, and if no suitable reader exists for the provided content type, the request fails.

Q3.2 Filtered Retrieval & Search

Using @QueryParam for filtering /api/v1/sensors?type=CO2) is generally considered superior because it aligns with RESTful principles for querying collections. Query parameters are specifically designed for filtering, sorting, and searching within a resource collection without changing the identity of the resource itself. In this case, /api/v1/sensors still represents the same collection, and the query parameter simply refines the result set. In contrast, em  bedding the filter in the path (e.g., /api/v1/sensors/type/CO2) treats the filter as a sub-resource, which can lead to less flexible and less scalable designs. Query parameters allow multiple filters to be combined easily (e.g., ?type=CO2&status=ACTIVE), whereas path-based filtering would require increasingly complex and rigid URL structures. Additionally, query parameters are more intuitive for clients and better supported by web standards for caching and bookmarking filtered results. Therefore, using @QueryParam results in a cleaner, more flexible, and more maintainable API design.

# Part 4: Deep Nesting with Sub – Resources

Q4.1 The Sub-Resource Locator Pattern

Sub-Resource Locator gives us several architectural advantages. Design pattern increases the  scalability of the RESTful API development process. Instead of dealing with nested routes in one huge controller class, you can transfer responsibility for deeper paths ("/sensors/{sensorId}/readings") to another class were it  will be SensorReadingResource. This means that the SensorResource class will only deal with its particular functionality and not have to take care of any additional logic. It would become difficult to manage, harder to test, and  to get errors as the API grows.

# Part 5: Advanced Error Handling & Exception Mapping

Q5.2 Dependency Validation (422 Unprocessable Entity)

The use of HTTP 422 Unprocessable Entity is more correct than 404 Not Found. The HTTP 422 Unprocessable occurs when the request could not be processed due tTO errors. The HTTP 404 Not Found status shows that the server cannot find a matching URL. The client calls the POST /sensors endpoint correctly, and the request is correct JSON. One of the fields of the roomId, contains a reference to a resource that does not exist. The request itself can be understood by the server, its contents are incorrect. The server returns HTTP 422 Entity response code to indicate that the request could not be processed correctly.

Q5.4 The Global Safety Net (500)

Providing external access to internal stack traces from Java presents several security threats since it gives away too much sensitive information related to how the program works internally. For example, a trace will contain class names, package names, file locations  As a result, malicious users gain valuable information about how an application is organised and may find potential vulnerabilities in its architecture to exploit. There is also the danger that stack traces give away other details such as frameworks, library packages, and server configurations, allowing for further targeted attacks using existing exploits. Sometimes error messages contain fragments of data that should not be there at all because of poor handling of the information. Using generic HTTP 500 Internal Server Error messages removes those threats.
