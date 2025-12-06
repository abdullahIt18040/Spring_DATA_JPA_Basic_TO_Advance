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
