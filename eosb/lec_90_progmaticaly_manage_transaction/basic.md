## PlatformTransactionManager
```
PlatformTransactionManager কী? (বাংলায় ব্যাখ্যা)

PlatformTransactionManager হলো Spring Framework–এর ট্রান্সঅ্যাকশন পরিচালনার মূল ইন্টারফেস।
এটি বলে দেয় — একটি ট্রান্সঅ্যাকশন কогда শুরু হবে, কিভাবে কমিট হবে, বা কিভাবে রোলব্যাক হবে।

যেমন:
আপনি যদি ডাটাবেজে একাধিক কাজ করেন —
➤ User save
➤ Order save
➤ Payment entry

এগুলো যেন একই ট্রান্সঅ্যাকশনের মধ্যে চলে এবং কোনো একটা ব্যর্থ হলে সবকিছু রোলব্যাক হয় —
সেই দায়িত্ব PlatformTransactionManager এর।

✅ PlatformTransactionManager কেন দরকার?

Spring Boot–এ আপনি সাধারণত @Transactional ব্যবহার করেন।
এই @Transactional কাজ করার জন্য ভিতরে যে মেকানিজম ব্যবহৃত হয় —
PlatformTransactionManager সেটাই।

✔ বিভিন্ন ধরনের PlatformTransactionManager

Spring আপনার ডাটাবেজ বা প্ল্যাটফর্ম অনুযায়ী আলাদা Manager ব্যবহার করে:

1️⃣ DataSourceTransactionManager

JDBC বা Simple JDBC Template ব্যবহার করলে

সিঙ্গেল ডাটাবেজ কানেকশন

2️⃣ JpaTransactionManager

Hibernate/JPA ব্যবহার করলে

EntityManager ভিত্তিক

3️⃣ HibernateTransactionManager

Hibernate Session ভিত্তিক

4️⃣ JtaTransactionManager

Distributed transaction (Microservices / XA)

🔥 PlatformTransactionManager কিভাবে কাজ করে?

যখন আপনি কোনো মেথডে @Transactional দেন:

@Transactional
public void saveOrder() {
    // 1. ট্রান্সঅ্যাকশন শুরু
    // 2. ডাটাবেজ অপারেশন
    // 3. সব সফল হলে commit
    // 4. Exception হলে rollback
}


এখন Spring ভিতরে যা করে —

Step-by-step:

PlatformTransactionManager.begin() → ট্রান্সঅ্যাকশন শুরু

আপনার business logic execute হয়

কোনো Exception না হলে → commit()

Exception পেলে → rollback()

🧠 একটি উদাহরণ

Spring Boot–এ সাধারণত আপনি এটা কনফিগার করেন না, Spring নিজেই করে দেয়।

কিন্তু চাইলে ম্যানুয়ালি এভাবে define করা যায়:

@Bean
public PlatformTransactionManager txManager(EntityManagerFactory emf) {
    return new JpaTransactionManager(emf);
}
```
## @Transactional কোন TransactionManager ব্যবহার করে?
```

Spring Boot-এ @Transactional কোন TransactionManager ব্যবহার করবে, তা নির্ভর করে আপনি কোন Persistence Technology ব্যবহার করছেন।

Spring Boot স্বয়ংক্রিয়ভাবে (Auto Configuration) TransactionManager নির্বাচন করে।

🔥 1️⃣ DataSourceTransactionManager

কবে ব্যবহৃত হয়?
✔ আপনি যখন JDBC, JdbcTemplate, NamedParameterJdbcTemplate, MyBatis ইত্যাদি সরাসরি JDBC কানেকশন ব্যবহার করেন।

ব্যাকএন্ড ভিত্তিক:

সিঙ্গেল ডাটাসোর্স

ডাটাবেজ লেভেলে Connection ভিত্তিক ট্রান্সঅ্যাকশন

👉 @Transactional ব্যবহার করলে কী হবে?
@Transactional
public void saveOrder() {
    // SQL execute via JDBC
    // সব OK → commit()
    // Exception → rollback()
}


এখানে Spring ভিতরে ব্যবহার করবে:
➡ DataSourceTransactionManager

🔥 2️⃣ JpaTransactionManager

কবে ব্যবহৃত হয়?
✔ যখন আপনি JPA / Hibernate with EntityManager ব্যবহার করেন
✔ Spring Data JPA / Hibernate ORM ব্যবহৃত হলে

ব্যাকএন্ড ভিত্তিক:

EntityManager

Persistence Context

👉 @Transactional ব্যবহার করলে:
@Transactional
public void saveOrder() {
    entityManager.persist(order);
}


এক্ষেত্রে Spring ব্যবহার করবে:
➡ JpaTransactionManager

🔥 3️⃣ HibernateTransactionManager

কবে ব্যবহৃত হয়?
✔ আপনি Hibernate SessionFactory সরাসরি ব্যবহার করলে
✔ কোনো JPA নয় — pure Hibernate API

যেমন:

@Autowired
private SessionFactory sessionFactory;


এক্ষেত্রে @Transactional ব্যবহার করলে:
➡ HibernateTransactionManager

🔥 4️⃣ JtaTransactionManager

কবে ব্যবহৃত হয়?
✔ Distributed Transaction বা XA Transaction
✔ Microservices / Multiple database / JMS + DB
✔ Atomikos, Bitronix, Narayana ইত্যাদি ব্যবহার করলে

যেমন:

Order Microservice → DB1

Payment Microservice → DB2
একই ট্রান্সঅ্যাকশনে অংশ নিচ্ছে → JTA দরকার।

@Transaction ব্যবহার করলে Spring বেছে নেবে:
➡ JtaTransactionManager
```
