## CompletableFuture কী?

CompletableFuture হল Java-এর async programming এর জন্য একটি powerful class। এর মাধ্যমে:

ব্যাকগ্রাউন্ডে কাজ চালাতে পারবেন
```
future result আসলে তাকে নিয়ে পরবর্তী কাজ করতে পারবেন

একাধিক async কাজ parallel চালাতে পারবেন

কাজ শেষে combine/multiply করতে পারবেন

🎯 কেন CompletableFuture ব্যবহার করবেন?
Traditional synchronous code:

একটি task শেষ না হলে পরের code execute হয় না → API slow হয়।

CompletableFuture asynchronous:

Task background thread-এ চলবে → API main thread block হবে না → High performance & scalability.

⚙️ Spring Boot-এ CompletableFuture ব্যবহারের উদাহরণ
1️⃣ Step: Async enable করা

@EnableAsync ব্যবহার করতে হবে।

@Configuration
@EnableAsync
public class AsyncConfig {
}

2️⃣ Step: Service method কে async করা

@Async + CompletableFuture<T>

@Service
public class UserService {

    @Async
    public CompletableFuture<String> getUserInfo() throws InterruptedException {
        Thread.sleep(3000); // simulate long task
        return CompletableFuture.completedFuture("User data loaded");
    }
}

3️⃣ Step: Controller থেকে async call করা
@RestController
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @GetMapping("/user")
    public CompletableFuture<String> getUser() {
        return userService.getUserInfo()
                .thenApply(result -> "Response: " + result);
    }
}

✔ API call সঙ্গে সঙ্গে return করবে

ব্যাকএন্ডে ব্যাকগ্রাউন্ড thread কাজ শেষ হলে Future complete হবে।

💡 Multiple Async Task Parallel Example
@Async
public CompletableFuture<String> task1() throws InterruptedException {
    Thread.sleep(2000);
    return CompletableFuture.completedFuture("Task 1 Done");
}

@Async
public CompletableFuture<String> task2() throws InterruptedException {
    Thread.sleep(3000);
    return CompletableFuture.completedFuture("Task 2 Done");
}


Controller:

@GetMapping("/parallel")
public CompletableFuture<String> runParallel() {

    CompletableFuture<String> t1 = service.task1();
    CompletableFuture<String> t2 = service.task2();

    return CompletableFuture.allOf(t1, t2)
            .thenApply(v -> t1.join() + " | " + t2.join());
}

🏃‍♂️ এখানে ২টা কাজ parallel চলবে

মোট সময় লাগবে → max(2s, 3s) = 3s

⭐ Important Methods
Method	Use
supplyAsync()	async result return করে
runAsync()	async void type কাজ
thenApply()	result transform
thenAccept()	শুধু consume
thenCombine()	দুই future combine
allOf()	সব future complete হওয়া পর্যন্ত wait
anyOf()	যেকোনো একটি complete হলেই return
🔥 CompletableFuture + Spring Boot কখন ব্যবহার করবেন?

Heavy database queries
```
Slow external API calls

Email sending

File processing

Parallel execution

Non-blocking REST API
