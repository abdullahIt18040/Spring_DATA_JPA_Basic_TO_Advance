## JPA — Database Mapping and Rules (Specification)

```
✔️ JPA (Jakarta Persistence API) একটি specification, মানে এটি শুধু “নিয়ম/গাইডলাইন” দেয়

কিভাবে—

জাভা ক্লাসকে ডাটাবেস টেবিলের সাথে map করবে

কিভাবে insert/update/delete হবে

কিভাবে transaction handle হবে

কিভাবে caching হবে

কিভাবে query লিখতে হবে (JPQL)

📌 JPA নিজে কোন কোড চালায় না, এটি শুধু API interface তৈরি করে।
কাজ করে আসলে Hibernate, EclipseLink বা OpenJPA—যারা এই JPA rules follow করে।

📌 JPA কী ধরনের নিয়ম দেয়?
1️⃣ Object → Table mapping

জাভার class → database table হবে
field → column হবে
@Id → primary key হবে
@OneToMany → relation হবে
@EmbeddedId → composite key

2️⃣ Persistence Context

একটি transaction-এর ভেতরে:

EntityManager একই object track করবে

Dirty checking করবে

One-time select cache রাখবে

3️⃣ Entity Lifecycle Rules

Entity হতে পারে:

New

Managed

Detached

Removed

4️⃣ JPQL Query rules

SQL না, Object ভিত্তিক query:

SELECT u FROM User u WHERE u.age > 20

🌿 Example: JPA Specification Explaining with a Real Entity

ধরুন আপনি একটি জাভা ক্লাস লিখলেন:

@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "full_name", nullable = false)
    private String name;

    @Column(unique = true)
    private String email;
}


👉 এখানে JPA শুধু rules define করছে:

@Entity → This class is a DB table

@Table(name="users") → table name users

@Id → id is primary key

@GeneratedValue → auto increment হবে

@Column → name, email কলামের নিয়ম

➡️ এগুলো JPA-এর rules (specification)।
➡️ কিন্তু execution কে করবে? Hibernate।

🏗️ How Hibernate Implements JPA Specification (Example Step)
Step 1 — আপনি entity লিখলেন

→ JPA define করে দেয় mapping rules।

Step 2 — Hibernate এই mapping পড়ে

→ Hibernate নিজে SQL বানায়:

insert into users (full_name, email) values (?, ?)

Step 3 — Hibernate database-এ send করে

→ Data insert হয়

Step 4 — Hibernate object state track করে (Dirty Checking)

আপনি লিখলেন:

user.setName("Changed");


Spring save() না করলেও Hibernate update চালায়:

update users set full_name=? where id=?


➡️ এটাও JPA-এর rule, Hibernate বাস্তবে execute করে।

🎯 Simplified Understanding
Level	Meaning
JPA	Only rules (what to do)
Hibernate	The real engine (how to do)
Spring Data JPA	Shortcut interface (easy CRUD)
🧩 Diagram
JPA (Rules)
   ↓
Hibernate (Implementation)
   ↓
Spring Boot (Uses Hibernate)
   ↓
Your Code (Entity/Repo)
   ↓
Database

🧠 Summary (One Line)

JPA শুধু বলে দেবে “mapping কীভাবে হবে”, “query কেমন হবে”, “entity কী” —
কিন্তু Hibernate সেই নিয়ম follow করে SQL চালায় এবং DB কাজ সম্পন্ন করে।
```
## Spring Data JPA এবং JPA এর সম্পর্ক

```
✔️ JPA = Rules / Specification

এটা শুধু guideline দেয়:

কিভাবে Entity mapping হবে

কিভাবে relationship হবে (@OneToMany, @ManyToOne)

কিভাবে JPQL query হবে

EntityManager কিভাবে কাজ করবে

Transaction কিভাবে behave করবে

➡️ JPA real code execute করে না, শুধু interface ও নিয়ম দেয়।

✔️ Hibernate = JPA Implementation

Hibernate JPA-এর সব নিয়মকে বাস্তবে execute করে:

SQL তৈরি করে

DB তে query চালায়

Entities load করে

Dirty Checking করে

Caching করে

➡️ Hibernate = Real Engine

✔️ Spring Data JPA = Abstraction layer on top of JPA

Spring Data JPA:

Repository interface তৈরি করে

Hibernate-এর EntityManager ব্যবহার করে

CRUD মেথড auto-generate করে

Query methods auto-create করে (findByName, findByEmail)

Pagination / Sorting সহজ করে

➡️ Spring Data JPA = Shortcut + Automation

🎯 Three-Layer Relationship (Very Important)
Spring Data JPA (Easy API / Repository)
            ↓
       JPA (Rules)
            ↓
Hibernate (Engine)
            ↓
Database (MySQL/Postgres)


এটাই exact relationship.

🌿 Example: একই Entity তিনভাবে কাজ করে
📌 1. JPA Level (rules)
@Entity
public class User {
    @Id
    Long id;
    String name;
}


JPA rules:

ক্লাসটি টেবিল

id primary key

name column

📌 2. Hibernate Level (execution)

Hibernate behind the scenes SQL create করে:

insert into user (id, name) values (?, ?)


Hibernate object ↔ row mapping করে।

📌 3. Spring Data JPA Level (your code)
public interface UserRepo extends JpaRepository<User, Long> {}


Now you can:

userRepo.save(new User());


এই save() →

Spring Data JPA → EntityManager কে call করে

EntityManager → Hibernate কে call করে

Hibernate → SQL execute করে DB তে

আপনি কোন SQL দেখতে পেলেন না — Spring Data JPA automatically সব handle করে দিল।

🌺 Relationship Summary (Short & Clear)
Layer	কাজ
JPA	Rules, annotations, entity mapping
Hibernate	Real database operations (SQL generation, caching)
Spring Data JPA	Developer-friendly repository API
🎯 One-Line Understanding

Spring Data JPA uses JPA rules, but relies on Hibernate to actually execute SQL in the database.
```
What is AbstractPlatformTransactionManager (Super Simple Explanation)

এটা Spring-এর transaction system-এর মা/বাবা ক্লাস।

Spring এ যত ধরনের transaction manager আছে:

JpaTransactionManager

DataSourceTransactionManager

HibernateTransactionManager

JtaTransactionManager

সবাই এই ক্লাসকে extend করে।

কারণ transaction কীভাবে শুরু হবে, commit হবে, rollback হবে—এই common rules আগে থেকেই এখানে লেখা থাকে।

🌿 Why does Spring need this class?

কারণ Spring চায়:

All transaction managers → Same rules follow করুক

Code repeat না হোক

Transaction start/commit/rollback workflow এক জায়গায় control করা যাক

এটি থাকায় Spring খুব সহজে বিভিন্ন ধরনের DB / JPA / JDBC transaction manage করতে পারে।

🌳 Simple Analogy (Very Easy)

ধরুন আপনি একটি কোম্পানি চালান।

Company Rules:

অফিস কখন খুলবে

কখন বন্ধ হবে

ছুটি কীভাবে হবে

এগুলো Company লিখে রাখে।
এখন যেই Manager (HR, Sales, Finance) আসুক না কেন—
সব Manager কে সেই rules follow করতে হবে।

AbstractPlatformTransactionManager = Company Rules
JpaTransactionManager = HR Manager
DataSourceTransactionManager = Sales Manager
HibernateTransactionManager = Finance Manager

সব Manager নিজের কাজ করবে, কিন্তু rules একটি common place থেকে follow করবে।


What is TransactionDefinition?

TransactionDefinition হলো Spring-এর transaction properties বর্ণনা করার interface।

Spring যখন একটি নতুন transaction শুরু করতে চায়, তখন ঠিক করে:

Propagation কী হবে?

Isolation level কী হবে?

Timeout কত?

Read-only হবে কিনা?

এই সব তথ্য TransactionDefinition থেকে নেওয়া হয়।

Spring এ এটি প্রধানত ব্যবহার হয়:

@Transactional

TransactionTemplate

PlatformTransactionManager এ

🌿 TransactionDefinition Contains 4 Main Things

Spring-এর transaction start করার আগে TransactionDefinition এই ৪টি জিনিস define করে:

1️⃣ Propagation Behavior (Propagation)

Transaction কীভাবে আচরণ করবে?

Seven types:

Propagation	Meaning
REQUIRED	Already transaction থাকলে join করবে, না থাকলে নতুন transaction তৈরি করবে
REQUIRES_NEW	চলতি transaction suspend, নতুন transaction শুরু
NESTED	nested transaction (savepoint)
SUPPORTS	থাকলে join, না থাকলে non-transaction
NOT_SUPPORTED	চাইলেও transaction চালানো যাবে না
MANDATORY	transaction না থাকলে error
NEVER	transaction থাকলে error

Example:

@Transactional(propagation = Propagation.REQUIRED)

2️⃣ Isolation Level

(For consistency control)

Defines: When multiple transactions run at the same time, how data will behave?

Level	Problems Prevented
READ_UNCOMMITTED	No prevention (dirty read possible)
READ_COMMITTED	Prevents dirty read
REPEATABLE_READ	Prevents dirty + non-repeatable read
SERIALIZABLE	Full isolation (most strict)

Example:

@Transactional(isolation = Isolation.REPEATABLE_READ)

3️⃣ Timeout

Transaction কত সেকেন্ড পর্যন্ত চলতে পারবে?

Default = no timeout

Example:

@Transactional(timeout = 5)   // 5 seconds


Timeout exceed ➝ rollback.

4️⃣ Read-Only Flag

Transaction শুধুই read করবে?
If yes:

Hibernate unnecessary update checking বন্ধ করে

কিছু DB optimization কাজ করে

Example:

@Transactional(readOnly = true)
public List<User> getUsers() { ... }


Read-only হলে update/insert করলে Spring error দিতে পারে।

🌱 Combined Example
@Transactional(
    propagation = Propagation.REQUIRES_NEW,
    isolation = Isolation.SERIALIZABLE,
    timeout = 10,
    readOnly = false
)
public void saveOrder() {
   ...
}

## What is DefaultTransactionDefinition? (Super Simple)
```
DefaultTransactionDefinition হলো Spring-এর transaction এর default settings রাখা একটি সাধারণ class।

এর কাজ:

Transaction-এর default behavior define করা

কোন transaction কোন property পাবে তা supply করা

Spring যখন নতুন transaction শুরু করে → এই class এর information পড়ে

এটা এমন এক class যেটা বলে দেয়:

Propagation → REQUIRED (default)

Isolation → DEFAULT (DB default)

Timeout → no limit

ReadOnly → false

Name → null

এই default settings ব্যবহার করে Spring transaction start করে।

🌿 Why is it Needed?

কারণ Spring যখন কোন transaction শুরু করে, তখন কিছু default rule দরকার হয়।

যদি আপনার কোডে কখনো না থাকে:

@Transactional(...)


তাহলে Spring internally DefaultTransactionDefinition ব্যবহার করে default settings দিয়ে transaction চালায়।

🌳 DefaultTransactionDefinition = Default Transaction Rules

এটি মূলত TransactionDefinition interface-এর default implementation।

✔️ Default Values inside DefaultTransactionDefinition
Property	Default Value
Propagation	REQUIRED
Isolation	DEFAULT
Timeout	TIMEOUT_DEFAULT = -1
Read Only	false (means read-write)
Name	null
🍀 Example (Very Simple)

Spring যখন আপনি simply লিখেন:

@Transactional
public void saveData() { }


আপনি কোন সেটিং দেননি → Spring বলবে:

"তাহলে default rule দিয়ে transaction চালাই।"

এই default rule আসে:

→ DefaultTransactionDefinition থেকে।
💡 Programmatic Example (Manual)
DefaultTransactionDefinition def = new DefaultTransactionDefinition();

TransactionStatus status =
    transactionManager.getTransaction(def);

try {
    // business logic
    transactionManager.commit(status);
} catch (Exception e) {
    transactionManager.rollback(status);
}


এখানে def ব্যবহার করে Spring transaction start করে।

Spring internally:

Propagation = REQUIRED
Isolation = DEFAULT
Timeout = none
ReadOnly = false


এই rule apply করে।

🌸 SUPER SIMPLE SUMMARY

DefaultTransactionDefinition হলো Spring-এর transaction-এর default settings holder ক্লাস।
Spring কোনো custom rules না পেলে এই default settings দিয়ে transaction চালায়।
```
