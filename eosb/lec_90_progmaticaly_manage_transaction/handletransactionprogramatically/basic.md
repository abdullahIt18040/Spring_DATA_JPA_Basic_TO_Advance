## programatically handle transaction


```
 @PersistenceContext
 private EntityManager entityManager;
 @Autowired
 private PlatformTransactionManager transactionManager;
 private TransactionTemplate transactionTemplate;
 
     DefaultTransactionDefinition def=   new DefaultTransactionDefinition();
     def.setIsolationLevel(TransactionDefinition.ISOLATION_READ_COMMITTED);
     def.setPropagationBehavior(Propagation.REQUIRES_NEW.value());
     def.setTimeout(10);

##    approach one
         TransactionStatus status =transactionManager.getTransaction(def);
         try {
//             Account account = accntrepo.findbyid(2);
//             account.setBlance(account.getBlance-1000);

             transactionManager.commit(status);
         }catch (Exception e)
         {
             transactionManager.rollback(status);

         }
## Approach two

        transactionTemplate.executeWithoutResult((status1)->{

            //             Account account = accntrepo.findbyid(2);
//             account.setBlance(account.getBlance-1000);
            if (1==1)
            {
                status1.setRollbackOnly();
            }
        });

    }
    
```
## What is Programmatic Transaction?
```

Spring এ transaction দুইভাবে manage করা যায়:

1️⃣ Declarative → @Transactional
2️⃣ Programmatic → TransactionTemplate বা PlatformTransactionManager ব্যবহার করে manually manage করেন

Programmatic transaction মানে:

✔️ আপনি নিজে transaction শুরু করবেন
✔️ commit করবেন
✔️ exception হলে rollback করবেন
✔️ কোন অংশ transaction এর ভিতরে থাকবে নিজে নির্ধারণ করবেন

🌿 WHY Programmatic Transaction?

যেখানে @Transactional কাজ না করে:

Scheduler / batch job

Multi-threaded code

Complex conditional logic

Nested partial commit নেয়ার প্রয়োজন

Try–catch এর ভিতরে custom rollback logic

🏆 METHOD–1: Programmatic Transaction using TransactionTemplate (Most Common)
➤ Spring automatically provides TransactionTemplate.
✔️ Example
@Service
public class OrderService {

    @Autowired
    private TransactionTemplate transactionTemplate;

    @Autowired
    private OrderRepository orderRepo;

    @Autowired
    private ProductRepository productRepo;

    public void createOrder() {

        transactionTemplate.execute(status -> {

            try {
                Order order = new Order("Laptop");
                orderRepo.save(order);

                productRepo.updateStock(1);

            } catch (Exception e) {
                status.setRollbackOnly();  // manual rollback
            }

            return null;
        });
    }
}

▶️ What happens here?

Spring start new transaction

আপনার code execute হয়

আপনি decide করেন rollback হবে কিনা

শেষে commit বা rollback

🏆 METHOD–2: Programmatic Transaction using PlatformTransactionManager

এটা আরো low-level, কিন্তু powerful।

✔️ Example
@Service
public class UserService {

    @Autowired
    private PlatformTransactionManager txManager;

    @Autowired
    private UserRepository userRepo;

    public void saveUserProgrammatically() {

        DefaultTransactionDefinition def = new DefaultTransactionDefinition();
        def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);

        TransactionStatus status = txManager.getTransaction(def);

        try {
            User u = new User("Mamun");
            userRepo.save(u);

            txManager.commit(status);  // manual commit

        } catch (Exception e) {
            txManager.rollback(status);  // rollback manually
        }
    }
}

🔥 Here you control everything:

Start transaction → getTransaction()

Commit → commit()

Rollback → rollback()

This is full-control programmatic transaction.
```
