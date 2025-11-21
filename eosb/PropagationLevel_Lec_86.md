### PlatformTransactionManager কী?

PlatformTransactionManager হলো Spring Framework-এর একটি ট্রানজ্যাকশন কন্ট্রোল মেকানিজম, যা ডাটাবেস অপারেশনগুলোকে
start → commit → rollback—এই ফ্লো অনুযায়ী নিরাপদভাবে পরিচালনা করে।

Spring JDBC, JPA/Hibernate, JMS ইত্যাদি বিভিন্ন টেকনোলজির জন্য আলাদা ট্রানজ্যাকশন ম্যানেজার থাকতে পারে, কিন্তু সবগুলোর common interface হলো —

org.springframework.transaction.PlatformTransactionManager
```
✅ কেন ব্যবহার করা হয়?
✔ 1. ডাটাবেস অপারেশনকে নিরাপদ রাখা

কোনো exception ঘটলে database-এর পরিবর্তন (INSERT/UPDATE) যেন corrupt না হয়।
→ তখন rollback হবে
→ আর সব ঠিক থাকলে commit হবে।

✔ 2. Multiple operations একটিই ট্রানজ্যাকশনে চালাতে

যেমন:

Employee table-এ INSERT

Salary table-এ INSERT
দুটোর একটি fail হলে দুটোই rollback হবে।

✔ 3. Declarative transaction (@Transactional) কে support করা

Spring-এ @Transactional annotation কাজ করে কারণ এর পেছনে PlatformTransactionManager থাকে।

🔥 সহজ ভাষায় উদাহরণ (Bangla Explanation)

ধরুন আপনি ব্যাংকে টাকা ট্রান্সফার করছেন:
A → B

দুটি কাজ হবে একসাথে:

A থেকে টাকা কমবে

B এর Account-এ টাকা বাড়বে

যদি ২ নম্বর ধাপে সমস্যা হয়, আর ১ নম্বর আপডেট already হয়েছে —
→ তাহলে সমস্যা! ডাটা mismatch হয়ে যাবে।

এই সমস্যা প্রতিরোধ করার জন্য Spring PlatformTransactionManager ব্যবহার করে।

✅ Code Example (Java Spring Boot)
👉 Without Annotation — Manual Transaction Handling
@Service
public class BankService {

    @Autowired
    private PlatformTransactionManager txManager;

    @Autowired
    private AccountRepository accountRepository;

    public void transferMoney(Long fromId, Long toId, double amount) {

        TransactionStatus status = txManager.getTransaction(new DefaultTransactionDefinition());

        try {
            Account fromAcc = accountRepository.findById(fromId).get();
            Account toAcc = accountRepository.findById(toId).get();

            fromAcc.setBalance(fromAcc.getBalance() - amount);
            toAcc.setBalance(toAcc.getBalance() + amount);

            accountRepository.save(fromAcc);
            accountRepository.save(toAcc);

            txManager.commit(status);  // সব ঠিক হলে commit

        } catch (Exception ex) {
            txManager.rollback(status); // ভুল হলে rollback
            throw ex;
        }
    }
}

👉 Bangla Explanation:

উপরের কোডে আমরা —

Transaction শুরু করেছি:
txManager.getTransaction()

ডাটাবেসে দুটো Account update করেছি।

সব ঠিক থাকলে:
txManager.commit()

কোনো ভুল/exception হলে:
txManager.rollback()

এভাবেই PlatformTransactionManager পুরো ট্রানজ্যাকশন হ্যান্ডেল করে।

⭐ Best Practice — @Transactional ব্যবহার

Spring Boot-এ আমরা direct PlatformTransactionManager না ব্যবহার করে সাধারণত:

@Transactional
public void transferMoney(...) { }


এটাই ব্যবহার করি।

কারণ Spring internally PlatformTransactionManager ব্যবহার করে ট্রানজ্যাকশন manage করে।

```
নীচে আমি আপনার দেওয়া manual transaction কোডটিকে Spring Boot annotation-based (@Transactional) স্টাইলে convert করে দেখাচ্ছি।
এভাবে করলে PlatformTransactionManager আপনাকে আলাদা করে ব্যবহার করতে হয় না—Spring নিজেই manage করে।
```
✅ Annotation-based Implementation (@Transactional)

(Recommended Spring Boot Best Practice)

@Service
public class BankService {

    @Autowired
    private AccountRepository accountRepository;

    @Transactional
    public void transferMoney(Long fromId, Long toId, double amount) {
        Account fromAcc = accountRepository.findById(fromId)
                .orElseThrow(() -> new RuntimeException("From Account not found"));

        Account toAcc = accountRepository.findById(toId)
                .orElseThrow(() -> new RuntimeException("To Account not found"));

        fromAcc.setBalance(fromAcc.getBalance() - amount);
        toAcc.setBalance(toAcc.getBalance() + amount);

        // Save changes
        accountRepository.save(fromAcc);
        accountRepository.save(toAcc);

        // No need commit(), rollback()
        // Spring automatically handles it
    }
}

📌 এখানে কী ঘটছে?
✔ @Transactional

এই method এর ভিতরের সব কাজ এক ট্রানজ্যাকশনের মধ্যে চলবে

কোনো exception হলে Spring automatic rollback করবে

সব ঠিক থাকলে automatic commit হবে

আপনাকে manual:

txManager.getTransaction()
txManager.commit()
txManager.rollback()


এসব করতে হবে না।

⭐ Internal Flow (@Transactional এর ভিতরে কী হয়?)

Spring internally নিচের কাজগুলো করে:

1️⃣ Method start হলে → PlatformTransactionManager দিয়ে নতুন transaction create
2️⃣ Method success ending → commit
3️⃣ কোনো runtime exception → rollback

🧩 PlatformTransactionManager Config (Optional)

যদি custom transaction manager প্রয়োজন হয়:

@Configuration
public class TxConfig {

    @Bean
    public PlatformTransactionManager transactionManager(EntityManagerFactory emf) {
        return new JpaTransactionManager(emf);
    }
}


কিন্তু Spring Boot সাধারণত default auto-configure করে দেয়, তাই আলাদা configurationও লাগেনা।
```
Spring এ CMT (Container-Managed Transactions) হলো এমন এক ধরনের ট্রানজ্যাকশন ম্যনেজমেন্ট যেখানে ট্রানজ্যাকশন কন্ট্রোল (begin / commit / rollback) ডেভেলপার করে না—Spring Container নিজে স্বয়ংক্রিয়ভাবে ম্যানেজ করে।

এটি সাধারণত Spring এর Declarative Transaction Management (যেমন @Transactional) ব্যবহার করে করা হয়।

##  CMT (Container-Managed Transaction) Spring এ কী?
```
Spring Framework এ যখন আপনি কোনো মেথডে @Transactional দেন, তখন Spring Container:

✔ ট্রানজ্যাকশন শুরু করে
✔ সফল হলে commit করে
✔ exception হলে rollback করে
✔ propagation নিয়ন্ত্রণ করে
✔ isolation level বজায় রাখে

অর্থাৎ — ডেভেলপারকে PlatformTransactionManager ব্যবহার করে ম্যানুয়ালি কিছু করতে হয় না।

এটাই CMT = Container Managed Transaction।

🔍 CMT কেন ব্যবহার করা হয়?
সুবিধা	ব্যাখা
কম কোড, বেশি কাজ	begin/commit/rollback লেখার দরকার নেই
কোড ক্লিন	ব্যবসায়িক লজিক-এ transaction কোড থাকে না
consistent behavior	সব মেথড একইভাবে ট্রানজ্যাকশন অনুসরণ করে
Declarative	কনফিগারেশন annotation দিয়েই হয়ে যায়
🧩 Spring এ CMT কিভাবে কাজ করে?

যখন আপনি @Transactional ব্যবহার করেন:

@Service
public class BankService {

    @Autowired
    private AccountRepository repo;

    @Transactional
    public void transfer(Long fromId, Long toId, double amount) {
        Account from = repo.findById(fromId).get();
        Account to   = repo.findById(toId).get();

        from.setBalance(from.getBalance() - amount);
        to.setBalance(to.getBalance() + amount);

        repo.save(from);
        repo.save(to);
    }
}


এখানে:

🔹 Step 1: Spring Proxy তৈরি করে

BankService ক্লাসের জন্য Spring একটি Proxy তৈরি করে।

🔹 Step 2: Proxy মেথড কল ইন্টারসেপ্ট করে

pxroxy → transfer() মেথড কল হলে:

Transaction শুরু করে

মেথড execute করে

Exception হলে rollback

Success হলে commit

🔹 Step 3: ট্রানজ্যাকশন boundary Spring finalize করে।
🎯 Spring CMT এ @Transactional কী কী নিয়ন্ত্রণ করে?
➤ Propagation

মেথড কোন ট্রানজ্যাকশন ফ্লো ব্যবহার করবে

যেমন:

REQUIRED

REQUIRES_NEW

MANDATORY

SUPPORTS

➤ Isolation Level

READ_COMMITTED

REPEATABLE_READ

SERIALIZABLE

➤ Read Only Transaction

@Transactional(readOnly = true)

📌 Spring CMT বনাম BMT (Bean Managed Transaction)
Feature	CMT	BMT
কে ট্রানজ্যাকশন ম্যানেজ করে?	Spring Container	আপনি নিজে
কোড লিখতে হয়?	না	হ্যাঁ
জটিলতা	কম	বেশি
Annotation	@Transactional	PlatformTransactionManager
🔥 উদাহরণ: BMT (ম্যানুয়াল) - Container করে না

(আপনি আগের মেসেজে যে উদাহরণ পাঠিয়েছিলেন)

TransactionStatus status = txManager.getTransaction(new DefaultTransactionDefinition());
try {
    // code
    txManager.commit(status);
} catch (Exception e) {
    txManager.rollback(status);
}


এটি BMT (Bean Managed Transaction)।
এখানে Developer নিজে ট্রানজ্যাকশন তৈরি করছে।

🎉 FINAL SUMMARY (সহজভাবে)

✔ CMT মানে Spring Container নিজে ট্রানজ্যাকশন শুরু/কমিট/রোলব্যাক পরিচালনা করে।
✔ ডেভেলপার শুধু @Transactional ব্যবহার করলেই হয়।
✔ সবচেয়ে ব্যবহৃত এবং সবচেয়ে clean approach।
✔ Proxy mechanism ব্যবহার করে transaction boundary ম্যানেজ করে।
```
Spring এ CMT (Container Managed Transaction) আসলে Proxy-based AOP ব্যবহার করে কাজ করে।
আপনি @Transactional দিলে Spring পর্দার আড়ালে কিছু ধাপে কাজ করে।
এখানে Step-by-Step সহজভাবে ব্যাখ্যা করছি।

## ✅ Step 1: Spring Proxy তৈরি করে
```
যখন Spring container BankService ক্লাসকে স্ক্যান করে এবং দেখে যে কোনো মেথডে @Transactional আছে, তখন Spring পুরো ক্লাসটির একটি Proxy Object তৈরি করে।

🔥 Proxy কী?

Proxy হলো এমন একটি Wrapper Object
যে আপনার মূল ক্লাসের আগে বসে
এবং method call intercept করে।

উদাহরণ (ধারণা):
Client → Proxy → Original Service


আপনি ভাবছেন আপনি BankService কল করছেন,
কিন্তু আসলে Spring আপনার জন্য একটি ProxyBankService বানিয়ে দেয়।

কেন Proxy তৈরি করে?

কারণ Spring কে প্রতিটি মেথডের কল আটকাতে হবে
→ transaction শুরু করা
→ commit/rollback করতে
প্রক্সি ছাড়া এটা সম্ভব নয়।

✅ Step 2: Proxy method call intercept করে

আপনি কল করলেন:

bankService.transfer(1L, 2L, 500);


যা ঘটে:

1️⃣ Client আসলে Proxy কে কল করে

Proxy দেখবে—এই মেথডে @Transactional আছে কি?

2️⃣ থাকলে সে বলে:

👉 “ঠিক আছে, আগে ট্রানজ্যাকশন শুরু করি”

3️⃣ Proxy → TransactionManager ব্যবহার করে

নতুন transaction শুরু করে

isolation + propagation rule সেট করে

4️⃣ এরপর Proxy → মূল মেথড execute করে
originalBankService.transfer(...)

5️⃣ মেথড চলার সময় দু’টি ঘটনা হতে পারে:
✔ (A) Success — কোনো Exception নেই

Proxy:

transaction commit করবে

❌ (B) Exception হলে

Proxy:

transaction rollback করবে

exception আবার বাইরে থ্রো করবে

6️⃣ সব সময় শেষে Proxy transaction boundary মেইনটেইন করে

অথবা closed করে।

✅ Step 3: Transaction boundary finalize করা

মেথড শেষ হলে Proxy:

🔹 transaction শেষ করে

success → commit

failure → rollback

🔹 connection release করে
🔹 ThreadLocal cleanup করে
🔹 TransactionContext clear করে

এভাবে পুরো transaction lifecycle Proxy “finalize” করে ফেলে।

🔥 সহজ ভাষায় সারসংক্ষেপ:
আপনার কোড           →  BankService.transfer()

Spring আসলে চালায়  →  Proxy.transfer()

Proxy কী করে?

1. Transaction শুরু করে
2. আসল মেথড চালায়
3. সফল হলে commit
4. ভুল হলে rollback

🔥 আরও সহজ analogy (উদাহরণ):

ধরুন আপনি ব্যাংকে টাকা তুলতে গিয়েছেন:

আপনি = Client
ব্যাংকের Cashier = Proxy
ব্যাংকের Vault = Original Service

আপনি Cashier এর কাছে যান, Vault এর কাছে সরাসরি নয়।

Cashier decide করে:

আপনার লেনদেন শুরু হবে

টাকা দেবে বা বাতিল করবে

শেষে রেকর্ড আপডেট করবে

Vault কিছুই জানে না — Proxy সবকিছু হ্যান্ডেল করে।
```
Spring CMT (Container-Managed Transaction) ব্যবহার করলে আপনি ম্যানুয়ালি PlatformTransactionManager দিয়ে ট্রানজ্যাকশন তৈরি করতে পারবেন না — এর কারণ খুবই গুরুত্বপূর্ণ।

এটি বুঝতে হলে প্রথমে জানতে হবে CMT কী করতে চায়।

✅ মূল কারণ: CMT = Spring নিজে ট্রানজ্যাকশন ম্যানেজ করবে

CMT-এর পুরো philosophy হলো:

👉 “Developer ট্রানজ্যাকশন ম্যানেজ করবে না, Spring Container নিজে সব করবে।”
👉 তাই ম্যানুয়ালি ট্রানজ্যাকশন তৈরি করা CMT-এর concept এর বিরুদ্ধে যায়।

🔥 1. Proxy ট্রানজ্যাকশন boundary ঠিক করে — Manual করলে conflict হয়

@Transactional দিলে Spring AOP Proxy:

Before method → Start Transaction

After method → Commit/Rollback

যদি আপনি একই মেথডের ভেতরে ম্যানুয়ালি করেন:

TransactionStatus st = txManager.getTransaction(...);


তাহলে Spring এর Proxy এবং আপনার কোড দু’টি আলাদা transaction তৈরি করবে।

যা বড় ধরনের সমস্যা তৈরি করবে:

✔ Nested transaction conflict
✔ Unexpected rollback
✔ Transaction propagation mismatch
✔ Double commit / double rollback

🔥 2. CMT-এর ক্ষেত্রে ট্রানজ্যাকশন boundary Proxy-এর হাতে

CMT এ:

Client → Proxy → Original Method


Proxy মেথড কল শুরুর আগেই transaction শুরু করে।

আপনি যদি মেথডের ভিতরে transaction শুরু করেন —
আপনার ট্রানজ্যাকশন Proxy-এর ট্রানজ্যাকশন এর ভিতরে চলে যাবে →
এতে নতুন nested ট্রানজ্যাকশন তৈরির মতো আচরণ হবে।

Spring এর default propagation: REQUIRED

তখন:

Proxy → Transaction A তৈরি করবে

Method → Transaction B তৈরি করবে

➡ ফলে ট্রানজ্যাকশন অস্থিতিশীল হয়ে যাবে।

🔥 3. CMT এর key principle: “Declarative, not Programmatic”

CMT design করা হয়েছে declarative programming এর জন্য:

@Transactional
public void doSomething() {}


Spring বলে:

✔ “Developer শুধু annotate করবে”
✔ “Container ট্রানজ্যাকশন তৈরি করবে”

যদি developer আবার manual টানজ্যাকশন করে:

✔ CMT-এর মূল উদ্দেশ্য ভেঙে যায়
✔ কোড complex এবং unpredictable হয়ে যায়
✔ Declarative behavior আর কাজ করে না

🔥 4. Transaction propagation কাজ করবে না

Spring এর propagation rules:

REQUIRED

REQUIRES_NEW

MANDATORY

SUPPORTS

NESTED

এই সব behavior Proxy level-এ control হয়।

কিন্তু আপনি যদি manual করেন:

txManager.getTransaction();


Propagation আর ফলো হবে না
কারণ propagation control করে Proxy, আপনি না।

🔥 5. Transaction Synchronization ThreadLocal দিয়ে কাজ করে

CMT এ transaction context ThreadLocal-এ bind করা হয়।

Proxy → ThreadLocal bind
Manual code → নতুন ThreadLocal bind

➡ দুইটি ভিন্ন context
➡ resource leakage
➡ multiple DB connection
➡ rollback mismatch

🎯 সংক্ষেপে কেন CMT + Manual Transaction করা যায় না?
কারণ	ব্যাখ্যা
Proxy already manages transactions	আপনি করলে double transaction তৈরি হবে
Propagation break হয়ে যাবে	REQUIRED, REQUIRES_NEW কাজ করবে না
Nested transaction conflict	Spring-controlled vs manual-controlled
Declarative control হারিয়ে যাবে	CMT design কে ভেঙে দেবে
Commit/rollback unpredictable	ভুল সময়ে rollback হবে
🔥 Final Summary (সহজ ভাষায়)

CMT এ:

🟢 Spring → ট্রানজ্যাকশন শুরু/শেষ করবে
🔴 Developer → ম্যানুয়ালি করা যাবে না

কারণ:

Proxy যখন transaction boundary control করছে,
আপনি ভেতরে গিয়ে boundary বানালে পুরো system conflict করবে
