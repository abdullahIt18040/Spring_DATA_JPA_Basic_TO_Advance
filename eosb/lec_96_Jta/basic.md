## AbstractPlatformTransactionManager – 
```
এটি Spring Framework-এর core abstract class

PlatformTransactionManager interface-এর partial implementation

Spring-এর সব প্রধান transaction manager এর base class

মূল কাজগুলো:

Transaction শুরু, commit, rollback নিয়ন্ত্রণ

Propagation (REQUIRED, REQUIRES_NEW ইত্যাদি) handle করা

Rollback rules প্রয়োগ করা

Transaction synchronization ও Thread-local binding করা

Nested transaction / savepoint support

যারা এটি extend করে:

DataSourceTransactionManager (JDBC)

JpaTransactionManager (JPA)

HibernateTransactionManager

JtaTransactionManager

@Transactional এর সাথে সম্পর্ক:

@Transactional ব্যবহার করলে Spring ভিতরে ভিতরে এই ক্লাস দিয়েই transaction পরিচালনা করে

```
## Distributed Transaction কীভাবে Globally Manage করা হয় — সহজ বাংলায়

Distributed Transaction মানে এমন একটি transaction
যা একাধিক service / database / resource জুড়ে কাজ করে
এবং সবগুলোকে একসাথে commit বা rollback করাতে হয়।

এটি globally manage করতে মূলত ২টি প্রধান approach ব্যবহার করা হয় 👇

🔹 1️⃣ JTA + Global Transaction Manager (2PC)
📌 কীভাবে কাজ করে?

Spring বা Java EE তে:

Global Transaction Manager (যেমন: Narayana, Atomikos)

JTA (Java Transaction API)

XA-enabled resources

🧩 Two-Phase Commit (2PC)

Phase 1 – Prepare

সব DB / resource কে জিজ্ঞেস করা হয়
👉 “তুমি commit করতে পারবে?”

Phase 2 – Commit / Rollback

সবাই OK → Commit

কেউ Fail → Rollback

👉 পুরো coordination করে
Global Transaction Manager

🧠 Spring Example
@Bean
public PlatformTransactionManager transactionManager() {
    return new JtaTransactionManager();
}

✅ সুবিধা

Strong consistency (ACID)

Automatic rollback

❌ সমস্যা

Performance slow

Deadlock risk

Complex setup

👉 Microservice-এ কম ব্যবহার হয়

🔹 2️⃣ Saga Pattern (Modern Approach)
📌 কীভাবে কাজ করে?

Long-running transaction কে
👉 ছোট ছোট local transaction এ ভাগ করা হয়

Failure হলে compensation transaction চালানো হয়

🧩 Example Flow

Order Service → Order created

Payment Service → Payment done

Inventory Service → Stock reduced

যদি Step-2 fail হয়:

Step-1 cancel (compensation)

🔸 Saga দুইভাবে হয়:
🟢 Choreography Saga

Event-driven (Kafka / RabbitMQ)

No central controller

🔵 Orchestration Saga

Central Saga Orchestrator

Step-by-step control

✅ সুবিধা

High performance

Microservice-friendly

Scalable

❌ সমস্যা

Eventual consistency

Compensation logic লিখতে হয়

🔹 JTA vs Saga (Quick Comparison)
বিষয়	JTA (2PC)	Saga
Consistency	Strong	Eventual
Performance	Slow	Fast
Microservice	❌	✅
Complexity	Config-heavy	Logic-heavy
🔹 কখন কোনটা ব্যবহার করবেন?

✅ Monolith / Few DB
→ JTA + Global Transaction Manager

✅ Microservice / Cloud
→ Saga Pattern
```
