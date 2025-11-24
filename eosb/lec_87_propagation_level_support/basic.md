

## propagation level : SUPPORTS 
```
মূল ধারণা

SUPPORTS মানে:

"যদি parent transaction থাকে, method সেই transaction-এ যোগ হবে।
যদি parent transaction না থাকে, method non-transactionally চলবে।"

অর্থাৎ, optional transactional behavior।

এটি transaction create করবে না, শুধুমাত্র parent transaction থাকলে join করবে।

2️⃣ উদাহরণ
@Service
public class MyService {

    @Transactional(propagation = Propagation.SUPPORTS)
    public void optionalOperation() {
        // এখানে database operation
        // যদি parent transaction থাকে → join করবে
        // যদি parent transaction না থাকে → non-transactionally চলবে
    }
}

Use Case:
@Transactional
public void parentOperation() {
    // parent transaction শুরু
    myService.optionalOperation(); // এটি parent transaction join করবে
}


যদি parentOperation call হয় → optionalOperation transaction-এ join করবে

যদি অন্য non-transactional call হয় → optionalOperation transaction ছাড়া চলবে

3️⃣ 특징

Non-transactional operation হিসেবে execute করতে পারে

কোন নতুন transaction তৈরি করবে না

সাধারণত read-only operations বা optional DB operations-এর জন্য ব্যবহার করা হয়

4️⃣ সাধারণ চিত্র
Parent Transaction Exists → SUPPORTS method joins it
Parent Transaction Does NOT Exist → SUPPORTS method executes non-transactionally

@tX(Requerd)
m1()
{           tx-1
m2()
}
@tx(support)
m2(){
 join with tx-1
}

2nd case
m1()
{          non-transaction
m2()
}
@tx(support)
m2(){
 execute with out tansaction
}


1️⃣ Parent transaction আছে → SUPPORTS join করবে → rollback হবে
2️⃣ Parent transaction নেই → SUPPORTS non-transactional → rollback হবে না

✅ Example Code (Very Clear Example)
Support Service Method
@Service
public class SupportService {

    @Transactional(propagation = Propagation.SUPPORTS)
    public void supportsMethod() {
        System.out.println("Inside SUPPORTS method");
        // simulate error
        if (true) {
            throw new RuntimeException("Error from SUPPORTS method");
        }
    }
}

CASE 1 — Parent transaction exists
Parent transactional method
@Service
public class ParentService {

    @Autowired
    private SupportService supportService;

    @Autowired
    private UserRepository repo;

    @Transactional
    public void parentMethod() {
        User user = new User();
        user.setFirstName("Test");
        repo.save(user); // saved inside transaction

        supportService.supportsMethod(); // SUPPORTS joins parent

        repo.save(new User("Another")); 
    }
}

▶ What happens?

parentMethod() starts a transaction

supportsMethod() joins that transaction

Inside supportsMethod() → exception thrown

Result:

Entire parent transaction rollback

No data saved in DB

Output:
Inside SUPPORTS method
RuntimeException...
ROLLBACK COMPLETE

CASE 2 — Parent transaction does NOT exist
Non-transactional method calling SUPPORTS
@Service
public class NormalService {

    @Autowired
    private SupportService supportService;

    @Autowired
    private UserRepository repo;

    public void noTransactionMethod() {
        User user = new User();
        user.setFirstName("Saved without transaction");
        repo.save(user); // committed immediately

        supportService.supportsMethod(); // runs without TX
    }
}

▶ What happens?

noTransactionMethod() has no @Transactional

DB save is auto committed

SUPPORTS sees no parent transaction → runs without transaction

SUPPORTS throws exception

Exception only stops execution

Already committed data will NOT rollback

Result:

✔ First user data saved in DB
❌ Exception thrown
❌ No rollback
```
