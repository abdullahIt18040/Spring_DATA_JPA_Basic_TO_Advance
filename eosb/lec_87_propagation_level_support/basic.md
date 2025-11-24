

## propagation level : SUPPORTS 
```
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
<img width="502" height="477" alt="image" src="https://github.com/user-attachments/assets/cb3e1f61-a879-4fd3-9d27-5ffbb0588fe7" />

```
