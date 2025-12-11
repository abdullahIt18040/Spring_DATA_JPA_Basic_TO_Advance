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
HTTP Status Codes — 5 Main Categories

## HTTP status কোড ৫টি গ্রুপে ভাগ করা:
```
🔵 1xx — Informational (তথ্য প্রদান)

Server বলছে:
“Request পেয়েছি, কাজ করছি।”

👉 সাধারণত API-তে বেশি ব্যবহার হয় না।

100 Continue

101 Switching Protocols

🟢 2xx — Success (সফল হয়েছে)

Client-এর request সফল হলে server এই কোড দেয়।

✔ 200 OK

সবচেয়ে common
Example: Data successfully returned.

✔ 201 Created

Server-এ নতুন resource তৈরি হয়েছে
Example: নতুন user created

✔ 204 No Content

Request সফল, কিন্তু body নেই
Example: Delete সফল হলে

🟡 3xx — Redirection (দিক নির্দেশনা)

Client-কে অন্য জায়গায় যেতে বলা হয়।

✔ 301 Moved Permanently

URL স্থায়ীভাবে পরিবর্তন হয়েছে

✔ 302 Found

Temporary redirect

✔ 304 Not Modified

Cache data valid আছে, নতুন data দরকার নেই

🔴 4xx — Client Error (Client-এর ভুল)

ইঙ্গিত দেয় client ভুল request পাঠিয়েছে।

✔ 400 Bad Request

Request ভুলভাবে পাঠানো
Example: ভুল JSON, missing fields

✔ 401 Unauthorized

Client login করে নি
→ Authentication needed

✔ 403 Forbidden

Client login আছে কিন্তু অনুমতি নেই
→ Authorization problem

✔ 404 Not Found

URL / Resource পাওয়া যায় নি

✔ 409 Conflict

Data conflict
Example: duplicate email, version mismatch

⚫ 5xx — Server Error (Server-এর সমস্যা)

Server-এর ভিতরে কোনো error হয়েছে।

✔ 500 Internal Server Error

Generic server error
Programmer-এর bug হলে সাধারণত এটি পাওয়া যায়।

✔ 502 Bad Gateway

Server অন্য সার্ভারের থেকে invalid response পেয়েছে

✔ 503 Service Unavailable

Server down বা overloaded

✔ 504 Gateway Timeout

Upstream server সময়মতো response দেয়নি

⭐ Practical Examples (Spring Boot)
✔ 200 OK Example
return ResponseEntity.ok(user);

✔ 201 Created Example
return ResponseEntity.status(HttpStatus.CREATED).body(savedUser);

✔ 400 Bad Request Example
throw new IllegalArgumentException("Invalid input!");

✔ 404 Not Found Example
throw new ResourceNotFoundException("User not found");

🎯 Summary Table
সেকশন	মানে	উদাহরণ
1xx	তথ্য	Continue
2xx	সফল	200 OK
3xx	রিডাইরেক্ট	301, 302
4xx	Client ভুল	400, 401, 404
5xx	Server ভুল	500, 503

```
