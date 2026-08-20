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
## my code implement spring security filter 
```
package com.sil.springsecurityapp.config;

import jakarta.servlet.Filter;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.boot.security.autoconfigure.SecurityProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.Order;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;
import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

@Configuration
public class SecurityConfig {
    private final RateLimitFilter rateLimitFilter = new RateLimitFilter();
    @Bean
    public SecurityFilterChain defaultFilterChain() {
        return new SecurityFilterChain() {
            @Override
            public boolean matches(HttpServletRequest request) {

                return request.getRequestURI().startsWith("/api");
            }

            @Override
            public List<Filter> getFilters() {
                return List.of(new ApiKeyFilter(),
                        new simpleFilter(),
                        rateLimitFilter);
            }
        };
    }

    static class ApiKeyFilter extends OncePerRequestFilter {
        private static final String API_KEY = "SIMLE_API_KEY";

        @Override
        protected void doFilterInternal(HttpServletRequest request,
                                        HttpServletResponse response,
                                        FilterChain filterChain) throws ServletException, IOException {

            String apiKey = request.getHeader("X-API-KEYPRO");
//            if (apiKey == null) {
//                filterChain.doFilter(request,response);
//                return;
//            }
             if(API_KEY.equals(apiKey))
             {
            filterChain.doFilter(request,response);
             }else {
                 response.setStatus(HttpStatus.FORBIDDEN.value());
                 response.setContentType(MediaType.APPLICATION_JSON_VALUE);
                 response.getWriter().println("""
                         {
                         "error message ": "hey bro! you are not permited to perform this operation ",
                          "STATUS ":404,
                           "ERROR ":"FORBIDDEN"
                         }
                         
                         
                         """);
             }


        }



    }
    static class simpleFilter extends OncePerRequestFilter {

        @Override
        protected void doFilterInternal(HttpServletRequest request,
                                        HttpServletResponse response,
                                        FilterChain filterChain) throws ServletException, IOException {

//            System.out.println("this is my first filter in sdlc pro ");
//            response.getWriter().println("hellow bro ! ");
            filterChain.doFilter(request, response);

        }
    }
    static class RateLimitFilter extends OncePerRequestFilter{
          private static final int MaxRequest  = 5;
        private final Map<String, Integer> requestCounter = new ConcurrentHashMap<>();

        @Override
        protected void doFilterInternal(HttpServletRequest request,
                                        HttpServletResponse response,
                                        FilterChain filterChain) throws ServletException, IOException {

            String clientIp = request.getRemoteAddr();

          int count=  requestCounter.merge(clientIp,1,Integer::sum);

            System.out.println(
                    "RateLimitFilter: "
                            + clientIp
                            + " -> "
                            + count
            );

            if(count>MaxRequest)
            {
                response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
                response.setContentType(MediaType.APPLICATION_JSON_VALUE);
                response.getWriter().println("""
                    {
                        "status": 429,
                        "error": "TOO_MANY_REQUESTS",
                        "message": "Too many requests. Please try again later."
                    }
                    """);
                return;
            }
            filterChain.doFilter(request,response);



        }
    }
}



```
