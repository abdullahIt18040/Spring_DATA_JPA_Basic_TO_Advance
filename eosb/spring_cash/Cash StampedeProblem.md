### Cache Stampede হলো এমন একটি সমস্যা যেখানে cache expire হওয়ার পর একসাথে অনেক request database বা backend server এ চলে যায়, ফলে server এ হঠাৎ খুব বেশি load পড়ে যায়।
```
সহজ ভাষায়:
আগে data cache এ ছিল, তাই fast response দিচ্ছিল।
কিন্তু cache expire হওয়ার পর সবাই একই data চাইলে সব request একসাথে database এ hit করে। এটাকেই cache stampede বলে।

Real Life Example

ধরো তোমার website এ homepage data Redis cache এ রাখা আছে।

Cache expiry: 5 মিনিট
প্রতি সেকেন্ডে 500 user homepage open করছে

৫ মিনিট পরে cache expire হলো।

এখন:

প্রথম user cache miss পেল
দ্বিতীয় userও cache miss পেল
তৃতীয় userও cache miss পেল
এভাবে 500 user একসাথে DB call করলো

ফলে:

Database overload
Slow response
Server crash পর্যন্ত হতে পারে
Simple Flow
Normal Situation
User Request
    ↓
Cache Hit
    ↓
Fast Response
Cache Stampede Situation
User Request
    ↓
Cache Expired
    ↓
Thousands of DB Calls
    ↓
Database Overload
Why It Happens?

মূল কারণগুলো:

একই সময় cache expire হওয়া
High traffic
Expensive database query
Multiple servers একই data request করা
Solutions of Cache Stampede
1. Mutex Lock (Most Common)

একজন request DB থেকে data আনবে, বাকিরা wait করবে।

Request 1 → DB call করছে
Request 2 → wait
Request 3 → wait
Example Logic
if(cache exists){
   return cache;
}

if(lock acquired){
   data = db.getData();
   cache.set(data);
   release lock;
   return data;
}else{
   wait and retry;
}

এটা Redis lock দিয়েও করা যায়।

2. Cache Expiry Randomization

সব cache যেন একই সময়ে expire না হয়।

খারাপ:

All cache expire at 10:00

ভালো:

User cache → 10:01
Product cache → 10:03
Order cache → 10:05

Example:

expiry = 300 + random(0,60);
3. Cache Prewarming

Cache expire হওয়ার আগেই background job দিয়ে refresh করা।

Cron Job → Refresh Cache

তাহলে user request এ DB hit কম হবে।

4. Never Let Cache Fully Expire

পুরোনো data কিছু সময় serve করো while new data generate হচ্ছে।

এটাকে বলে:

Stale Cache
Stale While Revalidate
Redis Example

ধরো Redis use করছো।

Application
   ↓
Redis Cache
   ↓ miss
Database

Cache miss হলে Redis distributed lock ব্যবহার করা যায়।

Interview Definition

Cache stampede is a situation where many requests simultaneously try to rebuild an expired cache entry, causing sudden heavy load on the backend/database.

Difference Between Cache Miss vs Cache Stampede
Cache Miss	Cache Stampede
One/few requests DB এ যায়	Huge number of requests DB এ যায়
Normal	Dangerous
Performance একটু কমে	System crash হতে পারে
Spring Boot Real Example

তুমি যদি Spring Boot + Redis use করো:

@Cacheable use করলে basic caching হবে
কিন্তু high traffic এ extra protection দরকার
এজন্য:
Redis Lock
Redisson
Background refresh
Random TTL ব্যবহার করা হয়
Short Summary

Cache stampede =

Cache expired
        +
Many concurrent requests
        =
Database overload

এটা avoid করতে সাধারণত:

Locking
Random expiry
Background refresh
Stale cache

ব্যবহার করা হয়।
```
