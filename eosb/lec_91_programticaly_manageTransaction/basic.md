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
