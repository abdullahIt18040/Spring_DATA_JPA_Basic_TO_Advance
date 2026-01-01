## Transaction Synchronization কী?

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
