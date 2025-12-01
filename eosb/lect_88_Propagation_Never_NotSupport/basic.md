## Propagation.NEVER কী?
```
Propagation.NEVER নির্দেশ করে যে মেথডটি কখনও ট্রানজেকশনের মধ্যে execute হবে না।

অর্থাৎ, যদি caller method ইতিমধ্যেই ট্রানজেকশনের মধ্যে থাকে, তাহলে Exception throw হবে।

সাধারণত use করা হয় যেখানে operation non-transactional হতে হবে, যেমন logging, audit, read-only operations বা legacy calls।

2️⃣ Spring Boot Configuration

প্রথমে pom.xml এ Spring Data JPA এবং H2/MySQL dependency থাকতে হবে।

3️⃣ Entity উদাহরণ
@Entity
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private int quantity;
}

4️⃣ Repository
@Repository
public interface ProductRepository extends JpaRepository<Product, Long> { }

5️⃣ Service Layer Example
Step 1: Non-Transactional Method (NEVER)
@Service
@RequiredArgsConstructor
public class ProductService {

    private final ProductRepository productRepository;

    @Transactional(propagation = Propagation.NEVER)
    public void logProductAccess(Long productId) {
        // এই method কখনও transaction এর মধ্যে run হবে না
        System.out.println("Product accessed: " + productId + " at " + LocalDateTime.now());
    }
}

Step 2: Caller Method
@Service
@RequiredArgsConstructor
public class OrderService {

    private final ProductService productService;
    private final ProductRepository productRepository;

    @Transactional // Transactional caller
    public void placeOrder(Long productId) {
        // Some DB operation
        Product product = productRepository.findById(productId)
                .orElseThrow(() -> new RuntimeException("Product not found"));
        product.setQuantity(product.getQuantity() - 1);
        productRepository.save(product);

        // Call NEVER method
        // **এই call এখন Exception দিবে**, কারণ caller already transactional
        productService.logProductAccess(productId);
    }

    public void nonTransactionalCaller(Long productId) {
        // Non-transactional call → Safe for Propagation.NEVER
        productService.logProductAccess(productId);
    }
}

6️⃣ Controller Layer
@RestController
@RequiredArgsConstructor
@RequestMapping("/api/order")
public class OrderController {

    private final OrderService orderService;

    @PostMapping("/place/{id}")
    public String placeOrder(@PathVariable("id") Long productId) {
        try {
            orderService.placeOrder(productId);
            return "Order Placed Successfully";
        } catch (Exception e) {
            return "Failed: " + e.getMessage();
        }
    }

    @PostMapping("/log/{id}")
    public String logProduct(@PathVariable("id") Long productId) {
        // Non-transactional caller → safe
        orderService.nonTransactionalCaller(productId);
        return "Logged Successfully";
    }
}

7️⃣ Behavior Summary
Caller Transaction	NEVER method behavior
Caller is transactional	Throws IllegalTransactionStateException
Caller is non-transactional	Executes normally
8️⃣ Key Points

Propagation.NEVER মানে “No Transaction Allowed”

ট্রানজেকশন থাকা অবস্থায় call করলে Exception

Non-transactional বা plain method থেকে call করলে safe

সাধারণ use-case: Logging, Audit, Read-only operations
```
