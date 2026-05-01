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

radis 
<img width="1076" height="601" alt="image" src="https://github.com/user-attachments/assets/70a7489b-d08a-4e1f-9295-1fe88130d1fc" />


