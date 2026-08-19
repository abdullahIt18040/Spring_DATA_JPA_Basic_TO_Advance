## Spring Security মূলত Servlet Filter-এর উপর ভিত্তি করে কাজ করে, যখন আপনি Spring MVC / Spring Boot-এর Servlet-based application ব্যবহার করেন।
```
Request Flow
Client
   ↓
Tomcat / Servlet Container
   ↓
Servlet Filter Chain
   ↓
Spring Security Filter Chain
   ↓
DispatcherServlet
   ↓
Controller
Spring Security কীভাবে Filter ব্যবহার করে?

Spring Security সাধারণত DelegatingFilterProxy ব্যবহার করে।

HTTP Request
     ↓
DelegatingFilterProxy
     ↓
FilterChainProxy
     ↓
Spring Security Filters
     ↓
Authentication / Authorization
     ↓
DispatcherServlet
     ↓
Controller

DelegatingFilterProxy → Spring Security-এর FilterChainProxy-এর কাছে request পাঠায়।

FilterChainProxy → Spring Security-এর বিভিন্ন filter manage করে।

যেমন:

UsernamePasswordAuthenticationFilter
BasicAuthenticationFilter
BearerTokenAuthenticationFilter
AnonymousAuthenticationFilter
AuthorizationFilter
```
