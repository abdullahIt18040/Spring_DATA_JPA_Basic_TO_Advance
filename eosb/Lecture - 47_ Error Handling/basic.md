## if we want to implement custom error controller or error handler  without use BasicErrorController
```
1.implements ErrorController 
2.override errorHtml(),and error() mehtod method name can be change ;
What is ErrorAttributes?
```
## ErrorAttributes is a Spring Boot interface that provides error-related data when an error happens.

👉 It supplies information like:

HTTP status

Error message

Exception name

Request path

Timestamp

Spring Boot internally uses this to build:

Error JSON (REST)

Error Model (HTML pages)

🔹 Default Error Attributes (Provided by Spring Boot)

When an error occurs, Spring Boot creates a map like this:

{
  "timestamp": "2025-12-14T10:30:15.123+06:00",
  "status": 404,
  "error": "Not Found",
  "message": "No handler found",
  "path": "/api/test"
}


This data comes from DefaultErrorAttributes.
