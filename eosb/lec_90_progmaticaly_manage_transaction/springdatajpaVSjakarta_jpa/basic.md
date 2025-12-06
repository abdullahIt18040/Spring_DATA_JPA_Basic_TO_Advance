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
