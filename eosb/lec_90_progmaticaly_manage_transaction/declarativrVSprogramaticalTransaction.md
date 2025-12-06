### TransactionTemplate

```

TransactionTemplate Spring Boot/Spring Framework-এ programmatic transaction handling করার জন্য ব্যবহার করা হয়।

Spring এ সাধারণত আমরা @Transactional ব্যবহার করি (Declarative Transaction)।
কিন্তু কিছু ক্ষেত্রে আমাদের manual / programmatic control দরকার হয়। সেই জায়গায় TransactionTemplate খুব উপকারী।

✅ TransactionTemplate কেন ব্যবহার করা হয়? (Main Reasons)
1️⃣ More fine-grained control (manual transaction control)

@Transactional এ শুরু–শেষ Spring manage করে।
কিন্তু কখনো কখনো আপনাকে নিজে ঠিক করতে হয়—

কোথায় commit হবে

কোথায় rollback হবে

কোন অংশ try হবে

nested logic

তখন TransactionTemplate ব্যবহার করা হয়।

2️⃣ Programmatic rollback control

Example:

transactionTemplate.execute(status -> {
    try {
        service.saveOrder();
        service.updateStock();
    } catch (Exception e) {
        status.setRollbackOnly();  // manual rollback
    }
    return null;
});


আপনি manually বলছেন → rollbackOnly।

3️⃣ When using code without Spring beans

যেমন:

multi-threaded code

async task

schedulers

batch job

custom executor

এগুলিতে @Transactional কাজ নাও করতে পারে (proxy-based limitations)।

4️⃣ Complex workflow / branching logic

Business logic বেশি advance হলে:

if(conditionA) do Tx1
else if(conditionB) do Tx2
else rollback


এগুলো annotation দিয়ে করা সম্ভব না।

5️⃣ Handling Transactions inside another Transaction safely

Sometimes, আপনি চান:

outer transaction থাকলেও inner transaction আলাদা ভাবে commit/rollback হোক।

Example: logging always should commit even if main business fails.

TransactionTemplate দিয়ে possible।

🔥 Transactional vs TransactionTemplate
Feature	@Transactional	TransactionTemplate
Type	Declarative	Programmatic
Control	Limited	Full control
Rollback	automatic rules	manual status.setRollbackOnly()
Nested logic	hard	easy
Works in async?	❌ No	✔️ Yes
Coding effort	কম	বেশি
✔️ Simple Example
@Service
public class OrderService {

    @Autowired
    private TransactionTemplate tx;

    @Autowired
    private OrderRepository repo;

    public void placeOrder() {

        tx.execute(status -> {
            try {
                repo.save(new Order());
                updateStock();
            } catch (Exception e) {
                status.setRollbackOnly();
            }
            return null;
        });
    }
}
```
