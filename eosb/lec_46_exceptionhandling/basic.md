Error Handling 
## Request Flow in Spring Boot (Step-by-step)
```
1. Embedded Server Receives the Request
   >>
   
a) server assign a thread to handle request.

Spring Boot uses an embedded server like:

Tomcat (default)

Jetty

Undertow

👉 The request first arrives at Tomcat (usually).

2. Servlet Container → DispatcherServlet

Tomcat hands the request to the Servlet container, and Spring Boot uses:

➡️ DispatcherServlet (Main Front Controller)

This is the first Spring component that receives the request.
it has request mapping handler mapping whicun select a specific method for corresponding url  like   /api/getuser
DispatcherServlet সবার আগে সেটি রিসিভ করে এরপর এটি equestMappingHandlerMapping

এটাই Spring MVC-র সেই component যেটা:

সব Controller স্ক্যান করে

তাদের @RequestMapping, @GetMapping, @PostMapping এনোটেশন পড়ে

কোন URL কোন method-এর সাথে match করে তা খুঁজে বের করে. 

then  pass n request to the controller 
It acts like a traffic police officer.


DispatcherServlet → ডাকে RequestMappingHandlerMapping

🔥 ধাপ 

RequestMappingHandlerMapping সব controller method গুলো scan করে দেখবে:
then pass request to the controller
```
<img width="858" height="388" alt="image" src="https://github.com/user-attachments/assets/b8714547-72e8-4c4e-bbaa-cae62e66d4f9" />


## Spring Boot Default Error Handling (Whitelabel Error)

Spring Boot-এর নিজস্ব built-in error page আছে।

URL না মিললে অথবা exception হলে default error JSON ফিরে আসে।
```
উদাহরণ:

{
  "timestamp": "2025-12-10T09:53:23.484+00:00",
  "status": 404,
  "error": "Not Found",
  "path": "/api/test"
}

```
## Which Controller Is Called in Spring Boot Default Error Handling?

যখন Spring Boot-এ কোনো error ঘটে
(যেমন 404, 400, 500, exception ইত্যাদি)
এবং তুমি কোনো custom handler তৈরি করো নি,

তখন Spring Boot স্বয়ংক্রিয়ভাবে কল করে:

⭐ org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController

এটাই Spring Boot-এর default error controller।

📌 Default Error URL Mapping

BasicErrorController mapping করা থাকে:

/error

যখন কোনো exception → DispatcherServlet → ErrorController → BasicErrorController তে যায়।

📌 Flow Diagram (Simple)
Exception Occurs
      ↓
DispatcherServlet
      ↓
BasicErrorController  ←— (built-in Spring Boot)
      ↓
Creates JSON Error Response
      ↓
Returns to Client

📌 Proof (Internal Structure)

BasicErrorController implements:

public interface ErrorController {
    String getErrorPath();
}


Spring Boot automatically maps:

/error → BasicErrorController
```
📌 Example: Default JSON
{
  "timestamp": "2025-12-11T10:40:10.234+00:00",
  "status": 404,
  "error": "Not Found",
  "message": "",
  "path": "/api/test"
}

⭐ Final Answer
**Spring Boot default error হলে যে controller কল হয়:

👉 BasicErrorController*
```
<img width="838" height="419" alt="image" src="https://github.com/user-attachments/assets/fbe9a6b4-3d93-4267-ac9b-2735614368a3" />



