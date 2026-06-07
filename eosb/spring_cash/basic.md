## spring cash
Latency (Response Time) :

User sends request → Spring controller → DB → then give response to the user.means Time taken for one request.

 ## Throughput (Capacity)

Throughput = how many requests your system can handle per time

👉 Example:
Your Spring Boot app handles 1000 requests per second (RPS)
## Tomcate
Apache Tomcat হলো একটি Java Web Server + Servlet Container, যা দিয়ে Java (Spring, JSP, Servlet) অ্যাপ রান করা হয় .thread contain into tomcat । Apache Tomcat প্রতিটা request handle করার জন্য thread ব্যবহার করে।

🔹 সহজভাবে বুঝলে

👉 Tomcat এমন একটা সার্ভার, যা
Browser থেকে request নেয় → Java code চালায় → response পাঠায়

## distributed cash in macharnism 
```
“Distributed caching mechanism” শুনতে জটিল লাগলেও ধারণাটা খুব straightforward—
👉 একাধিক server/app একসাথে একই cache share করে, যাতে data দ্রুত পাওয়া যায়

🔹 আগে বুঝি: Cache কী?

👉 Cache = fast temporary storage

উদাহরণ:

DB থেকে data আনতে 500ms লাগে
Cache থেকে আনতে 5ms লাগে
🔹 Distributed Cache কী?

👉 যখন তোমার system একাধিক server এ চলে:

App Server 1
App Server 2
App Server 3

👉 তখন যদি প্রতিটা server আলাদা cache রাখে:

Data mismatch হতে পারে ❌

👉 তাই সবাই মিলে shared cache ব্যবহার করে
👉 এটাকেই বলে Distributed Cache

🔹 Architecture
        User Request
             ↓
   ┌───────────────┐
   │  App Server 1 │
   │  App Server 2 │
   │  App Server 3 │
   └───────┬───────┘
           ↓
   Distributed Cache (Redis)
           ↓
         Database

👉 সব server একই cache use করে

🔹 Popular Tools
Redis (most popular)
Memcached
Hazelcast
Ehcache (cluster mode)
🔹 কাজ কীভাবে করে
Step-by-step:
Client request দেয়
App → cache check করে
যদি cache-এ থাকে → direct return ✅
না থাকলে → DB থেকে নেয় → cache-এ রাখে
```
radis 
<img width="1076" height="601" alt="image" src="https://github.com/user-attachments/assets/70a7489b-d08a-4e1f-9295-1fe88130d1fc" />

## cash Aside Pattern:
```
Cache Aside Pattern কী?

👉 Application নিজে cache control করে
👉 আগে cache check করে, না থাকলে DB থেকে এনে cache-এ রাখে

🔹 কীভাবে কাজ করে (Step-by-step)
1. Request আসে
2. Cache check
   → Data থাকলে → return ✅
   → না থাকলে → DB call
3. DB থেকে data এনে
4. Cache-এ save করে
5. Response return
🔹 Flow Diagram
Client
  ↓
Application
  ↓
Cache (Redis)
  ↓ (miss হলে)
Database
🔹 Example (Spring Boot)

ধরো তুমি Redis ব্যবহার করছো:

@Cacheable("users")
public User getUser(Long id) {
    return userRepository.findById(id).orElse(null);
}

👉 First call:

Cache miss ❌ → DB call → Cache save

👉 Second call:

Cache hit ✅ → Direct return
🔹 Update হলে কী হবে? (Important)

👉 Cache Aside এ update manually handle করতে হয়

@CacheEvict(value = "users", key = "#id")
public void updateUser(Long id, User user) {
    userRepository.save(user);
}

👉 মানে:

আগে DB update
তারপর cache clear (evict)
```

## Spring Cash: 
MAP of MAP: product key value map which contain key object value object .

<img width="1130" height="605" alt="image" src="https://github.com/user-attachments/assets/65df632b-12a0-47d0-88e2-21f7e774165b" />

## SPEL
```
SPeL এর পূর্ণরূপ হলো Spring Expression Language।
এটি Spring Framework এর একটি powerful expression language, যেটা দিয়ে runtime এ object এর value read, modify, method call, condition check ইত্যাদি করা যায়।

কেন SPeL ব্যবহার করা হয়?

SPeL ব্যবহার করা হয় যখন dynamic ভাবে value evaluate করতে হয়।

যেমন:

property access করা
method call করা
condition check করা
bean access করা
collection filter করা
Basic Syntax
#{expression}

বা annotation এর ভিতরে:

@Value("#{expression}")
```
## Unless and condition 
```
unless Redis caching এ ব্যবহার করা হয় method execute হওয়ার পরে decide করার জন্য যে result cache এ রাখা হবে কি না।

তোমার comment টা একদম ঠিক:

// condition -> evaluated before method invoke
// unless -> evaluated after method invoke

এখন সহজভাবে বুঝি।

condition vs unless
বিষয়	condition	unless
কখন check হয়	Method call এর আগে	Method call এর পরে
কী check করে	method run হবে/cache use হবে কি না	result cache এ save হবে কি না
result access করতে পারে?	না	হ্যাঁ (#result)
unless কেন ব্যবহার করা হয়?

অনেক সময় method execute হবে, কিন্তু সব result cache করতে চাই না।

যেমন:

null data cache করতে চাই না
empty list cache করতে চাই না
admin user এর data cache করতে চাই না
বড় object cache করতে চাই না

এগুলোর জন্য unless ব্যবহার হয়।

Example 1: null হলে cache করবে না
@Cacheable(
    cacheNames = "product",
    key = "#id",
    unless = "#result == null"
)
public Product getProduct(Integer id) {
    return productRepos.findById(id).orElse(null);
}

এখানে:

method execute হবে
result আসবে
যদি result null হয় → cache করবে না
null না হলে → cache করবে
Example 2: Admin হলে cache করবে না
@Cacheable(
    cacheNames = "product",
    key = "#id",
    unless = "#result.price > 10000"
)

যদি product price 10000 এর বেশি হয় → cache হবে না।

তোমার কোডে কোথায় useful?

তুমি CacheLogic class বানিয়েছো:

class CacheLogic{
    public boolean isCacheable(){
        var auth = SecurityContextHolder.getContext().getAuthentication();

        return auth.getAuthorities()
                .stream()
                .anyMatch(a -> !"ROLE_ADMIN".equals(a.getAuthority()));
    }
}

এখানে তুমি বলতে চাচ্ছো:

Admin হলে cache use করবে না
Normal user হলে cache করবে

এটা condition দিয়েও করা যায়:

@Cacheable(
    cacheNames = "product",
    key = "#id",
    condition = "@cacheLogic.isCacheable()"
)

এখানে:

method call এর আগেই check হবে
admin হলে cache use হবে না
কিন্তু unless হলে কী হতো?
@Cacheable(
    cacheNames = "product",
    key = "#id",
    unless = "#result == null"
)

এখানে:

আগে method execute হবে
পরে result দেখে decide করবে cache save করবে কি না
খুব সহজে মনে রাখো
condition

👉 "Method execute করার আগেই check করো"

unless

👉 "Result পাওয়ার পরে check করো"

Flow Diagram
condition
Request
   ↓
Condition check
   ↓
true হলে cache use
false হলে cache skip
unless
Request
   ↓
Method execute
   ↓
Result পাওয়া গেল
   ↓
Unless check
   ↓
false → cache save
true → cache save না
Real-life Use Case

সবচেয়ে common:

unless = "#result == null"

কারণ null cache করলে পরে real data আসলেও সমস্যা হতে পারে।
```
