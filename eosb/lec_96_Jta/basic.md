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

```
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
## Distributed System-এ কেন JTA ব্যবহার করা হয় (বাংলায় ব্যাখ
```
Distributed system-এ যখন একাধিক system / service / database একসাথে একটি transaction manage করে, তখন আমরা JTA (Java Transaction API) ব্যবহার করি।

সমস্যা কী?

ধরুন:

এক service-এ Order save

আরেক service-এ Payment cut

এগুলো আলাদা আলাদা system।

যদি:

Order save হয়ে যায় ✅

Payment fail হয় ❌

➡️ তাহলে data inconsistent হয়ে যাবে।

JTA কী করে?

JTA নিশ্চিত করে:

সব system একসাথে সফল হবে, নাহলে সব system একসাথে rollback হবে

এটাই Distributed Transaction Management।

JTA কিভাবে কাজ করে? (Two-Phase Commit – 2PC)
Phase 1: Prepare Phase

Transaction Manager সব system-কে জিজ্ঞেস করে:

“তুমি কি commit করতে ready?”

Database A → YES

Database B → YES / NO

Phase 2: Commit / Rollback Phase

যদি সবাই YES দেয় → সব জায়গায় commit

যদি একজনও NO দেয় → সব জায়গায় rollback

JTA-র মূল Components

Transaction Manager

পুরো transaction control করে

Resource Manager

Database, Message Queue ইত্যাদি

XA Protocol

Distributed coordination এর জন্য

বাস্তব উদাহরণ
Order Service  → MySQL Database
Payment Service → Oracle Database


Order + Payment = এক transaction

Payment fail হলে → Order rollback

JTA এটা নিশ্চিত করে

কোথায় JTA ব্যবহার করা হয়?

✔ Monolithic বা Enterprise Application
✔ Multiple Database transaction
✔ Bank / Financial System

কখন JTA ব্যবহার করা উচিত না?

❌ Microservices architecture
❌ High scalability দরকার হলে
❌ Long-running transaction

এক্ষেত্রে ব্যবহার করা হয়:
➡️ Saga Pattern (Eventual Consistency)

জনপ্রিয় JTA Implementation

Narayana (JBoss)

Atomikos

Bitronix

সংক্ষেপে

JTA = Distributed Transaction Management

Multiple system এক transaction-এ কাজ করলে ব্যবহার হয়

ACID property বজায় রাখে

Two-Phase Commit ব্যবহার করে

চাও তো আমি এটা diagram দিয়ে, অথবা JTA vs Saga comparison-ও বাংলায় বুঝিয়ে দিতে পারি।
```
