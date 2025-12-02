## EntityManager কি?
```
EntityManager হলো JPA (Java Persistence API) এর মূল interface।

DB operations (insert, update, delete, select) perform করার জন্য ব্যবহার হয়।

Spring Boot এ @PersistenceContext অথবা @Autowired EntityManager দিয়ে access করা যায়।

Example:

@PersistenceContext
private EntityManager em;

public void saveUser(User user) {
    em.persist(user); // insert
}

2️⃣ Transaction vs EntityManager
কীভাবে coordinate হয়

Transaction শুরু হয় যখন @Transactional method call হয়।

Spring Hibernate / JPA provider কে বলে → এই transaction context-এ সব DB operation করা হবে।

EntityManager তখন এই transaction এর “persistence context” (first-level cache) ব্যবহার করে data track করে।

Persisted entities transaction commit না হওয়া পর্যন্ত DB-তে permanently write হয় না।

Commit হলে সব change DB-এ save হয়।

Rollback হলে সব change discard হয়।

Flow Example
@Service
public class UserService {

    @PersistenceContext
    private EntityManager em;

    @Transactional
    public void updateUser(Long id, String newName) {
        User user = em.find(User.class, id); // managed entity
        user.setName(newName);               // change tracked in persistence context

        // Optional: em.persist(user); // not needed, already managed
        // DB update happens only at commit
    }
}


Step by Step:

Method call → Spring starts a transaction

em.find() → entity loaded and managed in persistence context

user.setName(newName) → EntityManager tracks change (dirty checking)

Method ends → transaction commit → EntityManager flushes changes to DB

If exception occurs → rollback → no changes in DB

3️⃣ Persistence Context & Flush

EntityManager সব managed entity track করে → called Persistence Context

Flush:

Persistence context এ pending changes DB-এ write করা।

Commit হলে automatically flush হয়।

Manual flush: em.flush()

em.persist(user); // insert
em.flush();       // force DB insert before commit

4️⃣ Rollback Handling
@Transactional
public void updateUser(Long id, String newName) {
    User user = em.find(User.class, id);
    user.setName(newName);
    if(newName.equals("error")) {
        throw new RuntimeException("Force rollback");
    }
}


Exception → Spring transaction manager rolls back

EntityManager discards all changes tracked in persistence context

```
### Flow Diagram: EntityManager & Transaction Coordination
```
Client / Service Call
        │
        ▼
 @Transactional Method Starts
        │
        ▼
Spring Transaction Manager
   - Begins DB Transaction
   - Binds Persistence Context (EntityManager) to transaction
        │
        ▼
EntityManager Operations
  - em.find()/em.persist()/em.merge()/em.remove()
  - Tracks entities in Persistence Context
  - Dirty Checking enabled
        │
        ▼
Method Execution Completes
        │
        ├─ If Success:
        │     - EntityManager flushes changes to DB
        │     - Transaction commits
        │     - Persistence Context cleared
        │
        └─ If Exception:
              - Transaction rolls back
              - EntityManager discards all pending changes
              - DB remains unchanged

Step by Step Example
@Service
public class UserService {

    @PersistenceContext
    private EntityManager em;

    @Transactional
    public void updateUser(Long id, String newName) {
        // Step 1: Load entity (managed by EntityManager)
        User user = em.find(User.class, id);

        // Step 2: Modify entity (EntityManager tracks changes)
        user.setName(newName);

        // Step 3: Optional flush
        // em.flush();  // Forces DB update before commit

        // Step 4: If exception occurs, all changes rollback
        if(newName.equals("error")) {
            throw new RuntimeException("Force rollback");
        }

        // Step 5: Method ends → Transaction commits → DB updated
    }
}

Explanation in Bangla

Transaction শুরু হয় @Transactional দিয়ে → Spring Transaction Manager handle করে।

EntityManager persistence context তৈরি হয় → DB operations track করে।

em.find() বা em.persist() call করলে entity persistence context এ managed হয়।

Dirty Checking: EntityManager automatically detects any change in managed entity।

Method execution শেষ হলে:

যদি success → flush() → commit → DB update

যদি exception → rollback → DB অক্ষত থাকে

Persistence Context commit/rollback শেষে clean হয়ে যায় → memory free।
```
