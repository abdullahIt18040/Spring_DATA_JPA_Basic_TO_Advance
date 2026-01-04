## AbstractPlatformTransactionManager – 

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
