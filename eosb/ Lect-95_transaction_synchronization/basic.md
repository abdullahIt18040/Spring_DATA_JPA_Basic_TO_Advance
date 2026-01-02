## Transaction কীভাবে চলছে সেটা observe করার জন্য।



## to measure time : 

```
@Slf4j
@Controller
public class AuditListener implements TransactionExecutionListener {
    record TxInfo(String name,long start)
    {}


    private final Map<Integer,TxInfo>map = new ConcurrentHashMap<>();
  public  void beforeBegin(TransactionExecution transaction) {
      var txInfo = new   TxInfo(transaction.getTransactionName(),System.currentTimeMillis());
      map.put(transaction.hashCode(),txInfo);

    }


     public void afterCommit(TransactionExecution transaction, @Nullable Throwable commitFailure) {
      printLog(transaction,"committed");
    }



     public void afterRollback(TransactionExecution transaction, @Nullable Throwable rollbackFailure) {
       printLog(transaction,"rollback");
    }
    private  void printLog(TransactionExecution transaction, String status)
    {

        TxInfo txInfo =map.remove(transaction.hashCode());
        if(txInfo != null)
        {
            log.warn("transaction : {} taken total {} ms",txInfo.name,(System.currentTimeMillis()-txInfo.start()));
        }


    }

}
```
## description 
```
@Slf4j
@Controller
public class AuditListener implements TransactionExecutionListener {

🔹 @Slf4j

Lombok annotation

Automatically log object তৈরি করে
(SLF4J logger)

🔹 @Controller

Spring এই ক্লাসটাকে Bean হিসেবে register করবে

(এখানে @Component দিলেও চলতো)

🔹 implements TransactionExecutionListener

👉 এইটাই core point

এই interface Spring কে বলে:

“Transaction শুরু, commit বা rollback হলে আমাকে জানাও”

TxInfo Record
record TxInfo(String name, long start) {}

🔹 record কেন?

Immutable (data পরিবর্তন করা যাবে না)

Lightweight

শুধু data carry করার জন্য perfect

TxInfo কী রাখে?
Field	Meaning
name	transaction নাম
start	transaction শুরু হওয়ার সময়
Map Declaration
private final Map<Integer, TxInfo> map = new ConcurrentHashMap<>();

🔹 কেন Map?

প্রতিটা transaction এর start time store করার জন্য

🔹 কেন Integer key?

transaction.hashCode() ব্যবহার করা হয়েছে

🔹 কেন ConcurrentHashMap?

Multiple thread এ transaction চলতে পারে

Thread-safe

beforeBegin()
public void beforeBegin(TransactionExecution transaction) {
    var txInfo = new TxInfo(
        transaction.getTransactionName(),
        System.currentTimeMillis()
    );
    map.put(transaction.hashCode(), txInfo);
}

🔹 কখন কল হয়?

👉 Transaction শুরু হওয়ার ঠিক আগে

🔹 কী করছে?

Transaction নাম নিচ্ছে

Current time নিচ্ছে

Map এ store করছে

📌 Example:

TX: createOrder
Start time: 10:00:00

afterCommit()
public void afterCommit(TransactionExecution transaction,
                        @Nullable Throwable commitFailure) {
    printLog(transaction,"committed");
}

🔹 কখন কল হয়?

👉 Transaction successfully commit হলে

🔹 commitFailure কেন nullable?

Normally null

Rare edge case failure info

afterRollback()
public void afterRollback(TransactionExecution transaction,
                          @Nullable Throwable rollbackFailure) {
    printLog(transaction,"rollback");
}

🔹 কখন কল হয়?

👉 Transaction rollback হলে (exception / error)

printLog() – Core Logic
private void printLog(TransactionExecution transaction, String status) {
    TxInfo txInfo = map.remove(transaction.hashCode());
    if (txInfo != null) {
        log.warn(
            "transaction : {} taken total {} ms",
            txInfo.name,
            (System.currentTimeMillis() - txInfo.start())
        );
    }
}

🔹 map.remove() কেন?

Map থেকে entry পুরোপুরি delete

Memory leak prevent

One-time usage

🔹 Execution time calculation
currentTime - startTime


📌 Sample log:

transaction : createOrder taken total 142 ms

Transaction Lifecycle Flow (Step by Step)
1️⃣ Transaction শুরু
   → beforeBegin()
   → start time saved

2️⃣ Business logic execute

3️⃣ Success হলে
   → afterCommit()
   → time calculate + log

   ❌ Failure হলে
   → afterRollback()
   → time calculate + log

4️⃣ Map clean-up

Important Observations ⚠️

status parameter এখন log এ ব্যবহার হচ্ছে না
চাইলে add করা যায়

```

transaction object delete হয় না

শুধু Map entry delete হয়

