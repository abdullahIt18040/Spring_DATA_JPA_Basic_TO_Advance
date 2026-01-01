## Transaction Synchronization কী?
transaction principle is 1. single responsible 
                          2.  open close principle(open for extension but close for modification)

Transaction Synchronization মানে হলো
👉 Transaction চলাকালীন বা Transaction শেষ হওয়ার আগে/পরে কিছু নির্দিষ্ট কাজ (callback) execute করা।
```
Spring আমাদের কিছু hook point দেয়, যেমনঃ

Transaction শুরু হলে

Commit হওয়ার আগে

Commit সফল হলে

Rollback হলে

Transaction শেষ হলে

এই কাজগুলো আমরা TransactionSynchronizationManager ব্যবহার করে করতে পারি।

🔹 কেন Transaction Synchronization দরকার?

কিছু কাজ আছে যেগুলো আমরা চাই—

✔️ শুধু তখনই চলুক, যখন Transaction সফলভাবে commit হবে
✔️ Rollback হলে চলবে না
✔️ DB commit হওয়ার পর notification পাঠানো
✔️ Cache update করা
✔️ Kafka / RabbitMQ message পাঠানো

❌ যদি Transaction fail করে, তাহলে এই কাজগুলো execute হওয়া উচিত না

এই সমস্যার সমাধানই হলো Transaction Synchronization

🔹 Real Life Example

ধরি আপনি একটি Order Service বানাচ্ছেন

1️⃣ Order database এ save হলো
2️⃣ Transaction commit হলো
3️⃣ তারপর Customer কে Notification পাঠানো হবে

❗ যদি Order save করার সময় error হয় → transaction rollback হবে → notification পাঠানো যাবে না
```
## code
```
@Transactional
public MyOrder createMyOrder(MyOrder myOrder) throws InterruptedException {
    System.out.println("order place by "+Thread.currentThread().getName());
  MyOrder order= myOrderRepos.save(myOrder);
    EmailRequest emailRequest = new EmailRequest("abdullah@gamil.com","demy email",
            "this is email body");
    //notify
//      EmailRequest emailRequest1 = new EmailRequest("","","");
   try {
     emailService.sendEmail(emailRequest);
   }catch (Exception e)
   {
     e.printStackTrace();
   }

    System.out.println("order created ...........................");
//    if (1==1)
//        throw new RuntimeException("ERROR OCCURED ............................");
    return order;

}
  @Async
    public void sendEmail(EmailRequest request) throws InterruptedException {
        System.out.println(" EMAIL send by  "+Thread.currentThread().getName());


        Thread.sleep(3000);
        System.out.println("email sending successfully .....................");

        throw new RuntimeException("error occured email not sending............");
//        try {
//            MimeMessage message = mailSender.createMimeMessage();
//            MimeMessageHelper helper =
//                    new MimeMessageHelper(message, false, "UTF-8");
//
//            helper.setTo(request.getTo());
//            helper.setSubject(request.getSubject());
//            helper.setText(request.getBody(), true); // HTML enabled
//            helper.setFrom("no-reply@yourcompany.com");
//
//            mailSender.send(message);
//
//            log.info("Email sent successfully to {}", request.getTo());
//
//        } catch (Exception ex) {
//            log.error("Failed to send email to {}", request.getTo(), ex);
//            throw new EmailSendException("Email sending failed");
//        }

    }
}
```
## if we execute all task into a single method.it will break solid principle like
```
@Transactional
public MyOrder createMyOrder(MyOrder myOrder) throws InterruptedException {
    System.out.println("order place by "+Thread.currentThread().getName());
  MyOrder order= myOrderRepos.save(myOrder);
    EmailRequest emailRequest = new EmailRequest("abdullah@gamil.com","demy email",
            "this is email body");

     emailService.sendEmail(emailRequest);
     wareHouseService.notifyToWareHouse(order.getId());


    return order;

}
```
to avoid this problem we are used transaction synchronization .

## ApplicationEventPublisher কী?

ApplicationEventPublisher হলো Spring এর event system।

👉 এটা দিয়ে আপনি কোনো event publish করেন
👉 অন্য class গুলো সেই event listen করে কাজ করে

মানে:
```
Order service → শুধু order তৈরি করবে

Email / Warehouse / Notification → event শুনে কাজ করবে

👉 Loose coupling তৈরি হয় (best design)

🔹 Step-by-Step Execution (Transaction সহ)
1️⃣ Method call
createMyOrder()


Spring transaction শুরু করে।

2️⃣ Order save
MyOrder order = myOrderRepos.save(myOrder);


Order DB তে save হয়

⚠️ এখনো commit হয়নি

Transaction active

3️⃣ Event publish
eventPublisher.publishEvent(new OrderCreatedEvent(order));


👉 এখানে খুব গুরুত্বপূর্ণ বিষয় আছে ❗

❓ Event কখন handle হবে?

By default:

Event সাথে সাথেই publish হয়

Listener transaction এর ভিতরেই execute হয়

🔴 Problem (Default Behavior)

ধরি:

Listener এ email পাঠানো হচ্ছে

Email পাঠাতে error হলো

Listener exception throw করলো

👉 তাহলে:

পুরো transaction ROLLBACK হবে

Order save হবে না ❌

🔹 Correct Way: AFTER_COMMIT Event 🔥

👉 Order commit হওয়ার পরেই event handle হওয়া উচিত

Listener example:
@Component
public class OrderEventListener {

    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void handleOrderCreated(OrderCreatedEvent event) {

        System.out.println("Handling event AFTER COMMIT");

        // email / notification / kafka
    }
}

🔹 Final Flow (Best Practice)
Transaction START
   |
   |-- Order Save
   |-- Event Publish (queued)
   |
Transaction COMMIT
   |
   |-- Event Listener Execute


👉 যদি transaction rollback হয়
👉 Event execute হবে না
```
## MY CODE  IS:
```

@Transactional
public MyOrder createMyOrder(MyOrder myOrder) throws InterruptedException {
    System.out.println("order place by "+Thread.currentThread().getName());
  MyOrder order= myOrderRepos.save(myOrder);
   eventPublisher.publishEvent(new OrderCreatedEvent(order));


    return order;

}


LISTENSER IS
@Component
@RequiredArgsConstructor
public class EmailNotificationListener{

    private final EmailService emailService;
  @TransactionalEventListener(value = OrderCreatedEvent.class,phase = TransactionPhase.AFTER_COMMIT)
    public void handleCreateEventInformNotify(OrderCreatedEvent event) throws InterruptedException {

       var order= event.order();
        EmailRequest emailRequest = new EmailRequest(order.getEmail(),"",
                "order created successfully it is inform from eventlistener");
     emailService.sendEmail(emailRequest);
     Thread.sleep(3000);
      System.out.println("event listener is called");

    }

}
@Component
@RequiredArgsConstructor
public class WareHouseNotificationListener {
    private final WareHouseService wareHouseService;

    @TransactionalEventListener(value = OrderCreatedEvent.class,phase = TransactionPhase.AFTER_COMMIT)
    public void handleWareHouseNotification(OrderCreatedEvent orderCreatedEvent)
    {
        var order=orderCreatedEvent.order();
        wareHouseService.notifyToWareHouse(order.getId());
        System.out.println("wareHousenotification is working now man ;;;;;;;;");

    }
}
```
## above task maintain  transactionsynchronizer 
## TransactionSynchronizationManager কী?

TransactionSynchronizationManager হলো Spring এর low-level core class
যা—

👉 বর্তমান thread-এর সাথে transaction সম্পর্কিত তথ্য bind করে রাখে
👉 Transaction lifecycle অনুযায়ী callback execute করতে দেয়
```
সহজ ভাষায়:
Transaction এর ভেতরে “আগে / পরে” কিছু কাজ করার controller

🔹 Spring ভিতরে কীভাবে ব্যবহার হয়?

Spring internally এই class ব্যবহার করে—

@Transactional

@TransactionalEventListener

@EventListener

Resource binding (EntityManager, Connection)

👉 আপনি যেগুলো ব্যবহার করছেন (event listener),
সবকিছুর নিচে কাজ করছে TransactionSynchronizationManager

🔹 TransactionSynchronizationManager কী কী manage করে?
1️⃣ Transaction active কিনা
TransactionSynchronizationManager.isActualTransactionActive();

2️⃣ Thread-bound resource

JDBC Connection

Hibernate Session / EntityManager

3️⃣ Transaction synchronization register করা

Commit এর পরে

Rollback এর পরে

🔹 Real Simple Example (Manual)
@Transactional
public void createOrder() {

    TransactionSynchronizationManager.registerSynchronization(
        new TransactionSynchronization() {

            @Override
            public void afterCommit() {
                System.out.println("Transaction committed");
            }

            @Override
            public void afterCompletion(int status) {
                if (status == STATUS_ROLLED_BACK) {
                    System.out.println("Transaction rolled back");
                }
            }
        }
    );

    // DB operation
}


👉 এইটাই low-level version
👉 @TransactionalEventListener internally এটা use করে

🔹 আপনার Case এর সাথে Connection

আপনি লিখেছেন:

eventPublisher.publishEvent(new OrderCreatedEvent(order));


এবং listener এ:

@TransactionalEventListener(phase = AFTER_COMMIT)

Internally কী হয়?

1️⃣ Event publish
2️⃣ Spring দেখে → Transaction active
3️⃣ Event কে TransactionSynchronizationManager এ register করে
4️⃣ Transaction commit হলে → listener execute
5️⃣ Rollback হলে → skip / rollback listener execute

🔹 Thread-Bound Concept 🧵
System.out.println(Thread.currentThread().getName());


👉 Transaction + Synchronization সব একই thread এ bound
👉 Thread change হলে (new thread) transaction আর থাকবে না

🔹 Important Methods (Interview)
Method	কাজ
isSynchronizationActive()	Synchronization active কিনা
isActualTransactionActive()	Transaction চলছে কিনা
bindResource()	Resource bind
unbindResource()	Resource unbind
registerSynchronization()	Callback register
🔹 Open Session in View এর সাথে Relation

OpenEntityManagerInViewFilter ব্যবহার করে:

EntityManager thread এ bind থাকে

TransactionSynchronizationManager এর মাধ্যমে

🔹 কখন Directly ব্যবহার করবেন?

✔️ Custom framework
✔️ Low-level infrastructure code
✔️ Advanced transaction control

❌ Normal business service এ direct ব্যবহার না করাই ভালো

🔹 Interview One-Line Answer 🎯

👉 TransactionSynchronizationManager Spring এর core class যা transaction lifecycle অনুযায়ী callback এবং resource manage করে।
```

