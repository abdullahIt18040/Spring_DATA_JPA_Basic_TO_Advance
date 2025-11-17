## Spring Transaction Propagation: NESTED vs REQUIRES_NEW (Bangla Explanation)

Spring Boot-এ transactional method যখন আরেক transactional method-কে কল করে, তখন Propagation নির্ধারণ করে তাদের transaction flow কেমন হবে।

সবচেয়ে বেশি কনফিউশন হয় NESTED এবং REQUIRES_NEW নিয়ে।

চলুন সহজ করে বুঝি 👇

1️⃣ REQUIRES_NEW
📌 Meaning (সহজ ভাষায়)

আগের transaction pause হবে।

নতুন একটি পুরো independent transaction শুরু হবে।

Inner transaction commit হয়ে যাবে, outer ব্যর্থ হলেও।

Outer rollback → inner rollback হবে না।

🎯 Example (Bangla ব্যাখ্যাসহ)
@Service
public class MyService {

    @Autowired
    private AnotherService anotherService;

    @Transactional
    public void outerMethod() {
        System.out.println("Outer starting...");
        anotherService.innerRequiresNew(); // NEW transaction
        throw new RuntimeException("Outer failed");
    }
}

@Service
public class AnotherService {

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void innerRequiresNew() {
        System.out.println("Inner (REQUIRES_NEW) started");
        // insert...insert...
        // This will COMMIT even if outer fails
    }
}

✔ Result

innerRequiresNew() DATA → COMMIT হবে

outerMethod() exception → outer rollback

Inner unaffected because it is independent.

2️⃣ NESTED
📌 Meaning (সহজ ভাষায়)

Outer transaction এর ভিতরে sub-transaction তৈরি হয় not create new transaction, ।

it used JDBC savepoint ব্যবহার করে means it point the transaction utill it correpted,when transaction correpted only roollback correpted portion . other part of transaction perform well and commited whole task execute one single transaction ।

Inner rollback হলে শুধুমাত্র inner অংশটি rollback হবে।

Outer rollback হলে → Everything rollback (inner + outer)

🎯 Example
@Service
public class MyService {

    @Autowired
    private AnotherService anotherService;

    @Transactional
    public void outerMethod() {
        System.out.println("Outer starting...");
        try {
            anotherService.innerNested();  // Nested transaction
        } catch (Exception e) {
            System.out.println("Nested rolled back");
        }
        System.out.println("Outer continues...");
    }
}

@Service
public class AnotherService {

    @Transactional(propagation = Propagation.NESTED)
    public void innerNested() {
        System.out.println("Inner (NESTED) started");
        // insert...
        throw new RuntimeException("Nested failed");
    }
}

✔ Result

innerNested() error → শুধু nested অংশ rollback

outer এখনো চলতে পারে (continue)

কিন্তু outer যদি পরে fail করে → পুরো transaction rollback হবে।

🔥 Main Difference (সহজ comparison)
বিষয়	REQUIRES_NEW	NESTED
Transaction type	সম্পূর্ণ নতুন	Outer transaction-এর ভিতরে sub transaction
Outer rollback এ কি হয়?	Inner safe থাকে	সবকিছু rollback
Inner rollback এ কি হয়?	Outer unaffected	শুধুমাত্র nested অংশ rollback
Implementation	নতুন connection/transaction	JDBC savepoint
Usage	আলাদা ভাবে commit দরকার	বড় transaction-এর ভিতরে ছোট অংশ আলাদা করে handle করতে
🎉 সবচেয়ে সহজ Summary

REQUIRES_NEW = আলাদা transaction, outer এর সাথে কোনো সম্পর্ক নেই

NESTED = একই transaction, শুধু savepoint দিয়ে inner handle করা হয়

