## Servlet কী?
```
Servlet নিজে কোনো server/container নয়।

Servlet হলো একটি specification/API, যা বলে দেয় web request কীভাবে handle হবে এবং কী কী interface/class থাকতে হবে।

Examples:

Filter
FilterChain
Servlet
ServletRequest
ServletResponse
2. Tomcat কী?

Tomcat হলো Servlet Container, যা Servlet specification-এর implementation প্রদান করে।

Jakarta Servlet Specification
            ↓
        Servlet API
            ↓
     Implemented by
            ↓
          Tomcat
3. Example: FilterChain

Servlet API:

public interface FilterChain {

    void doFilter(
        ServletRequest request,
        ServletResponse response
    );
}

Tomcat-এর implementation:

org.apache.catalina.core.ApplicationFilterChain

অর্থাৎ:

FilterChain (Specification/API)
            ↓
ApplicationFilterChain (Tomcat implementation)
4. Request Flow
Client
  ↓
Tomcat
  ↓
Servlet Container
  ↓
ApplicationFilterChain
  ↓
Servlet Filters
  ↓
DispatcherServlet
  ↓
Controller

Spring Security থাকলে:

Client
  ↓
Tomcat
  ↓
ApplicationFilterChain
  ↓
DelegatingFilterProxy
  ↓
FilterChainProxy
  ↓
SecurityFilterChain
  ↓
Spring Security Filters
  ↓
DispatcherServlet
  ↓
Controller
5. Important Point
Servlet Specification
       ↓
     Rules
       ↓
Servlet API
       ↓
Implementation
       ↓
Tomcat / Jetty / Undertow

মনে রাখবেন:

Servlet = Specification/API
Tomcat = Servlet Container + Servlet specification implementation

আর ApplicationFilterChain হলো Tomcat-এর internal implementation, jakarta.servlet.FilterChain হলো standard Servlet API interface।
```
