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
## DelegatingFilterProxy হলো Spring-এর একটি Servlet Filter, যেটা নিজে মূল security কাজ করে না। এটি request-কে Spring-এর ApplicationContext-এর একটি bean/filter-এর কাছে delegate করে।

সহজভাবে:

DelegatingFilterProxy = Servlet container এবং Spring Security-এর মধ্যে bridge
```
Flow
Browser / Client
      ↓
Tomcat
      ↓
DelegatingFilterProxy
      ↓
FilterChainProxy
      ↓
Spring Security Filters
      ↓
DispatcherServlet
      ↓
Controller
কেন দরকার?

Tomcat সরাসরি Spring Bean সম্পর্কে জানে না।

Tomcat জানে:

Servlet
Filter
Listener

অন্যদিকে Spring জানে:

@Service
@Component
@Bean

তাই DelegatingFilterProxy এই দুইটার মধ্যে connection তৈরি করে।

Tomcat
  │
  │ Servlet Filter
  ↓
DelegatingFilterProxy
  │
  │ delegate
  ↓
Spring ApplicationContext
  │
  ↓
FilterChainProxy
  │
  ↓
Security Filters
DelegatingFilterProxy নিজে Authentication করে?

না।

এটি মূলত request-কে অন্য filter-এর কাছে পাঠায়।

Spring Security-তে সেই গুরুত্বপূর্ণ filter হলো:

FilterChainProxy

FilterChainProxy আবার একাধিক Security Filter execute করে।

যেমন:

DelegatingFilterProxy
        ↓
FilterChainProxy
        ↓
UsernamePasswordAuthenticationFilter
        ↓
BearerTokenAuthenticationFilter
        ↓
AuthorizationFilter
        ↓
Controller
একটি সহজ উদাহরণ

ধরুন client request পাঠাল:

GET /api/user
Authorization: Bearer eyJ...

তখন:

1. Tomcat request receive করে

GET /api/user

2. DelegatingFilterProxy request পায়

এটি বলে:

"আমি security-এর কাজ নিজে করব না; Spring Security-এর filter-এর কাছে request পাঠাব।"

3. FilterChainProxy request নেয়

তারপর appropriate security filters চালায়।

4. JWT filter JWT check করে

JWT
 ↓
Validate
 ↓
Authentication
 ↓
SecurityContext

5. Authorization check হয়

User has required role?
        ↓
   Yes → Controller
   No  → 403 Forbidden
সবচেয়ে গুরুত্বপূর্ণ সম্পর্ক
DelegatingFilterProxy
        ↓
     delegates
        ↓
FilterChainProxy
        ↓
Security Filter Chain
        ↓
Authentication + Authorization
```
### Servlet Container vs IoC Container
```
1. Servlet Container

Servlet Container হলো এমন একটি runtime environment যা HTTP request, response, Servlet এবং Filter manage করে।

Example:

Tomcat
Jetty
Undertow
Client
  ↓
Tomcat (Servlet Container)
  ↓
Filter
  ↓
DispatcherServlet
  ↓
Controller

Main responsibilities:

HTTP Request/Response
Servlet lifecycle
Filter management
Request handling
2. IoC Container

IoC = Inversion of Control

Spring IoC Container Java Objects/Beans তৈরি, manage এবং dependency injection করে।

Example:

ApplicationContext
       ↓
@Service
@Repository
@Controller
@Component

Spring-এর প্রধান IoC Container হলো:
```
## ApplicationContext
```
Difference
Servlet Container                       	IoC Container
Tomcat	                                  Spring ApplicationContext
HTTP request manage করে         	           Spring Bean manage করে
Servlet/Filter manage করে	                 Dependency Injection করে
Web lifecycle manage করে	                 Object lifecycle manage করে
```

### Spring Security Connection
```
HTTP Request
     ↓
Tomcat
(Servlet Container)
     ↓
DelegatingFilterProxy
     ↓
FilterChainProxy
     ↓
Security Filters
     ↓
DispatcherServlet
     ↓
Controller

Key Point:

Servlet Container → Web request, Servlet, Filter manage করে।
IoC Container → Spring Bean এবং Dependency manage করে।
DelegatingFilterProxy → Servlet Container এবং Spring Security-এর মধ্যে bridge।
```
