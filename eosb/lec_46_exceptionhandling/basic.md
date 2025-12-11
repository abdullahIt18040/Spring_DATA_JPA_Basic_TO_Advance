Error Handling 
## Request Flow in Spring Boot (Step-by-step)

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

<img width="858" height="388" alt="image" src="https://github.com/user-attachments/assets/b8714547-72e8-4c4e-bbaa-cae62e66d4f9" />

