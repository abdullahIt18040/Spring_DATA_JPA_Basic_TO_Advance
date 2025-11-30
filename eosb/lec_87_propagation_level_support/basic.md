

## propagation level : SUPPORTS 
```
peristance context (Persistence Context হলো Hibernate এর একটি মেমোরি-ভিত্তিক Live Tracking Cache,
যেখানে transactional entity গুলো থাকে এবং Hibernate স্বয়ংক্রিয়ভাবে তাদের পরিবর্তন track করে)

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
## Persistence Context — সহজ বাংলা ব্যাখ্যা

Persistence Context হলো Hibernate/Spring JPA এর একটি “Cash / Memory Area”
যেখানে আপনার entity objects গুলো transaction চলার সময় মেমোরিতে স্টোর থাকে।
```
👉 একে বলা হয়
✔ First Level Cache
✔ Managed State Container

✔ Persistence Context কী করে?

Spring এ যখন @Transactional মেথড শুরু হয়:

Hibernate একটি Persistence Context (Session) তৈরি করে।

মেথডের ভিতরে Entity লোড করলে Hibernate সেটিকে মেমোরিতে রাখে।

একই entity আবার লোড করলে DB hit করে না — Cached value ফেরত দেয়।

Entity তে কোনো পরিবর্তন করলে Hibernate track করে রাখে।

Transaction শেষ হলে Hibernate অটোমেটিক সব পরিবর্তন DB তে flush করে দেয়।

📌 সহজ উদাহরণ
@Transactional
public void updateUser() {

    User u1 = userRepo.findById(1L); // 1st query → hits DB
    User u2 = userRepo.findById(1L); // 2nd query → NO DB (from Persistence Context)

    u1.setFirstName("Mamun Updated");

    // No save() needed → Hibernate detects change
}

👉 কেন দ্বিতীয়বার DB query হয়নি?

কারণ u1 entity already stored in persistence context.

✔ Persistence Context এর States
State	Meaning
Transient	DB-তে নেই, Context-এ নেই
Persistent	Context-এর ভেতরে আছে, Hibernate track করছে
Detached	Context থেকে বের হয়ে গেছে, আর track হয় না
Removed	Context-এ আছে, তবে delete হওয়ার জন্য marked
✔ Context কিভাবে কাজ করে (Step-by-step)

ধরি আপনার transaction এইরকম:

@Transactional
public void process() {
    User u = userRepo.findById(1L);  
    u.setName("Mamun");
}

Step 1️⃣ — DB থেকে data load

Hibernate SELECT * FROM users WHERE id=1 চালায়
এবং সেই object কে Persistence Context এ রাখে।

Step 2️⃣ — Object পরিবর্তন (tracking)

আপনি যখন u.setName("Mamun") করেন
Hibernate সেটাকে note করে রাখে কিন্তু তৎক্ষণাৎ DB-তে আপডেট করে না।

Step 3️⃣ — Transaction শেষ হলে:

Hibernate automatically:

changes detect করে

generate করে:

UPDATE users SET name='Mamun' WHERE id=1

✔ কেন Persistence Context গুরুত্বপূর্ণ?
✔ Performance: কম database query
✔ Auto-dirty-checking: save() না করলেও Hibernate পরিবর্তন ধরে
✔ Transaction consistency
✔ Object identity guarantee
✔ Real Example (Identity Guarantee)
User u1 = em.find(User.class, 1L);
User u2 = em.find(User.class, 1L);


👉 Hibernate কখনো একই ID-এর দুইটা object তৈরি করবে না।
u1 == u2 (same object in memory)

✔ Persistence Context কখন clear হয়?

Transaction শেষ হলে

entityManager.clear() দিলে

entityManager.detach(entity) দিলে

@Transactional boundary শেষ হলে

🔥 Real-world Scenario

ধরুন আপনি E-commerce এ Order place করছেন।

@Transactional
public void placeOrder() {
    Product p = repo.findById(10L);  
    p.setStock(p.getStock() - 1); // update stock

    Order o = new Order(...);
    orderRepo.save(o);
}


Hibernate সবকিছু ধরে রেখে transaction শেষে:

product stock update

order save

একটি single transaction এ সম্পন্ন করে।

🎯 Summary (One Line)
```
Persistence Context হলো Hibernate এর একটি মেমোরি-ভিত্তিক Live Tracking Cache,
যেখানে transactional entity গুলো থাকে এবং Hibernate স্বয়ংক্রিয়ভাবে তাদের পরিবর্তন track করে।

যদি চান, আমি Persistence Context + Hibernate Flush Modes (AUTO, COMMIT, ALWAYS) — এগুলোও deep
```
peristance context (get entity form database)
   |  flush()
   database pool .
   | commit().
   database.

```

## Propagation.NOT_SUPPORTED

```
Propagation.NOT_SUPPORTED — ব্যাখ্যা 

@Transactional(propagation = Propagation.NOT_SUPPORTED)
মানে এই মেথডটি কখনোই ট্রানজ্যাকশনের ভিতর চলবে না।

যদি আগে থেকেই কোনো ট্রানজ্যাকশন চলছে, Spring ওই ট্রানজ্যাকশনকে suspend (অস্থায়ীভাবে থামিয়ে) দেয় এবং এই মেথডকে non-transactional mode-এ চালায়।

🔥 মূল ধারণা
বিষয়	ব্যাখ্যা
“ট্রানজ্যাকশন সাপোর্ট করবে না”	এই মেথড নিজের জন্য ট্রানজ্যাকশন তৈরি করবে না
যদি আগের মেথডে ট্রানজ্যাকশন থাকে	সেই ট্রানজ্যাকশনকে pause করে দিবে
মেথড চলবে	Non-transactional ভাবে
Exception হলে	Rollback হবে না (কারণ ট্রানজ্যাকশন নেই)
🎯 কোথায় ব্যবহার হয়?

যেখানে আপনি স্পষ্টভাবে চান না যে মেথডটি ট্রানজ্যাকশনের ভিতর চলুক।

উদাহরণ:

কিছু কাজ আছে যা slow, ট্রানজ্যাকশনের ভিতরে রাখলে performance খারাপ হবে
যেমন:

logging

audit

file upload

email sending

external API call

এগুলো transactional হওয়ার দরকার নেই।

🧠 রিয়েল লাইফ উদাহরণ (Bangla)

ধরুন আপনি টাকা ট্রান্সফার করছেন:

@Transactional
public void transferMoney() {
    // 1. টাকা কেটে নেওয়া
    // 2. টাকা জমা দেওয়া

    logService.saveLog("Transfer money");
}


এখন saveLog() যদি transaction-এর ভিতর চলে এবং log insert করতে গিয়ে error হয়ে যায় 👉 পুরো transfer rollback হয়ে যাবে!

এটি ঠিক না।

তাই NOT_SUPPORTED ব্যবহার করা হয়:

@Transactional(propagation = Propagation.NOT_SUPPORTED)
public void saveLog(String msg) {
    logRepository.save(new Log(msg));
}


👉 এতে transaction suspend হবে
👉 log insert non-transactional চলবে
👉 main transaction নিরাপদ থাকবে
👉 log ব্যর্থ হলেও money transfer rollback হবে না

🔍 Flow Chart

1️⃣ Calling method has TX → YES
2️⃣ NOT_SUPPORTED method called
3️⃣ Spring TX pause করে
4️⃣ মেথড non-TX mode এ execute
5️⃣ শেষে আবার আগের TX resume

🟩 কখন ব্যবহার করবেন?

কাজটি ট্রানজ্যাকশনের ভেতরে রাখা ঝুঁকিপূর্ণ

কাজটি commit / rollback দ্বারা প্রভাবিত হওয়া উচিত না

কাজটি independent

কাজটি সময়সাপেক্ষ (slow)

🟥 কখন ব্যবহার করবেন না?

যখন মেথডের কাজ database-এর atomic হওয়া জরুরি

যখন rollback-safe behaviour দরকার

যখন মেথড অবশ্যই ট্রানজ্যাকশনের অংশ হওয়া উচিত (e.g. product stock update)

📌 Summary (One Line)

NOT_SUPPORTED মানে — এই মেথড ট্রানজ্যাকশন চায় না; আগের ট্রানজ্যাকশন থাকলে সেটিও থামিয়ে দেয়।
```
