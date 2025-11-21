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
