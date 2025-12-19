## What Is Programmatic Transaction Handling?
```

@Transactional
public void saveOrder() { ... }


This is declarative transaction management.

But sometimes you need more control, such as:

Begin and commit transaction manually

Run some code inside a transaction

Manage rollback conditions manually

Handle multiple small transactions inside a large workflow

Combine business logic + conditional transaction

👉 For these cases, Spring gives you:
TransactionTemplate
✅ How TransactionTemplate Works

TransactionTemplate allows you to run:

transactionTemplate.execute(status -> {
    // transactional code here
});


Inside the execute() block:

✔ A transaction starts
✔ If no exception → commit
✔ If exception → rollback

You control everything programmatically.

⚙️ STEP–BY–STEP PRACTICAL EXAMPLE

We will create:

Repository

Service (using TransactionTemplate)

Controller (to call the service)

This simulates a real workflow: Save order → Save payment inside one transaction.

✅ Step 1 — Inject TransactionTemplate

Spring Boot auto-configures it if a PlatformTransactionManager exists.

📌 OrderService.java (Programmatic Transaction Example)
@Service
@RequiredArgsConstructor
public class OrderService {

    private final OrderRepository orderRepository;
    private final PaymentRepository paymentRepository;
    private final TransactionTemplate transactionTemplate;

    public String placeOrderProgrammatically(Order order, Payment payment) {

        return transactionTemplate.execute(status -> {

            try {
                // 1. Save order
                orderRepository.save(order);

                // 2. Save payment
                paymentRepository.save(payment);

                // 3. Everything OK → commit automatically
                return "Order + Payment Saved Successfully";

            } catch (Exception e) {

                // 4. Rollback manually if needed
                status.setRollbackOnly();
                return "Transaction Failed: " + e.getMessage();
            }
        });
    }
}

📝 What Happens Above?
✔ transactionTemplate.execute()

Starts a new transaction.

✔ If no error

Spring commits automatically.

✔ If error occurs

We do:

status.setRollbackOnly();


Now Spring will not commit → properly rolls back.

✔ You return a custom response from inside the transaction

(Unlike @Transactional where return value does not control commit/rollback)

🔥 More Advanced: Conditional Rollback Example
transactionTemplate.execute(status -> {
    Customer c = customerRepo.findById(10).orElseThrow();

    if (!c.isActive()) {
        status.setRollbackOnly();
        return "Customer is not active. Rolling back!";
    }

    customerRepo.updateBalance(10, 500);

    return "Balance added successfully.";
});


✔ Rollback based on business logic, not exception.

🔁 Another Style: Using TransactionCallbackWithoutResult
transactionTemplate.execute(new TransactionCallbackWithoutResult() {
    @Override
    protected void doInTransactionWithoutResult(TransactionStatus status) {
        try {
            orderRepository.save(order);
            paymentRepository.save(payment);
        } catch (Exception e) {
            status.setRollbackOnly();
        }
    }
});

```
## Begin and control a transaction programmatically using PlatformTransactionManager or TransactionTemplate
```
DefaultTransactionDefinition def= new DefaultTransactionDefinition();
     def.setIsolationLevel(TransactionDefinition.ISOLATION_READ_COMMITTED);
     def.setPropagationBehavior(Propagation.REQUIRES_NEW.value());
     def.setTimeout(10);
     
        TransactionStatus status2 =  transactionManager.getTransaction(def2);

        @Autowired
private PlatformTransactionManager transactionManager;

public void processTransaction() {
    TransactionDefinition definition =
            new DefaultTransactionDefinition(TransactionDefinition.PROPAGATION_REQUIRED);

    TransactionStatus status = transactionManager.getTransaction(definition);

    try {
        // 🔹 Business logic
        saveData();
        updateData();

        // ✅ Commit transaction
        transactionManager.commit(status);
    } catch (Exception ex) {
        // ❌ Rollback transaction
        transactionManager.rollback(status);
        throw ex;
    }
}
✔ When to use
Using TransactionTemplate (Recommended & Cleaner)

This is safer and cleaner than manual begin/commit.

Example
@Autowired
private TransactionTemplate transactionTemplate;

public void processTransaction() {
    transactionTemplate.execute(status -> {
        // 🔹 Business logic
        saveData();
        updateData();
        return null;
    });
}

✔ Advantages

Automatically begins & commits transaction

Rolls back on RuntimeException

Less boilerplate code
```
## transaction ordering, priority, and execution speed when multiple transactions are opened in the same method.
```
     DefaultTransactionDefinition def= new DefaultTransactionDefinition();
     def.setIsolationLevel(TransactionDefinition.ISOLATION_READ_COMMITTED);
     def.setPropagationBehavior(Propagation.REQUIRES_NEW.value());
     def.setTimeout(10);


     DefaultTransactionDefinition def2 = new DefaultTransactionDefinition();
     def2.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRES_NEW);
        def2.setTimeout(12);






         TransactionStatus status =transactionManager.getTransaction(def);
        TransactionStatus status2 =  transactionManager.getTransaction(def2);
         try {
             var log = new  TransactionLog();
             log.setStatus("success");
             log.setAmount(1000.2);
             log.setFromAccountId(2);
             log.setToAccountId(1);

             transactionlogrepo.save(log);
             transactionManager.commit(status2);
//             Account account = accntrepo.findbyid(2);
//             account.setBlance(account.getBlance-1000);
             transactionManager.commit(status);

             The latest opened transaction becomes the “current” (active) transaction, and all database work runs inside it until it is committed or rolled back.

```
work flow:
<img width="553" height="424" alt="image" src="https://github.com/user-attachments/assets/3061aeba-7a8a-43d5-8488-41a789f84fb5" />


