<img width="1806" height="783" alt="image" src="https://github.com/user-attachments/assets/ee4154b9-b66c-486e-8adb-a88c66e3b21d" />

## HikariCP কী?

HikariCP হলো Java/Spring Boot-এর সবচেয়ে fast ও lightweight JDBC Connection Pool।

👉 Spring Boot 2+ / 3+ এ এটা default connection pool।

🔹 Connection Pool মানে কী? (সহজ উদাহরণ)

ভাবো:

Database = ব্যাংক

Connection = ব্যাংকের কাউন্টার

User request = কাস্টমার

❌ যদি pool না থাকে:

প্রতিটা request এ নতুন connection খুলতে হবে

আবার close করতে হবে

খুব slow হবে

✅ Pool থাকলে:

আগেই কিছু connection বানানো থাকে

দরকার হলে নেয়

কাজ শেষ হলে ফেরত দেয়

Performance অনেক ভালো

🔹 HikariCP কীভাবে কাজ করে?
1️⃣ Application start হলে

HikariCP:

কিছু database connection আগেই তৈরি করে

এগুলো pool-এ রাখে

[Conn1, Conn2, Conn3, Conn4, Conn5]

2️⃣ যখন Service / Repository DB call করে
DataSource.getConnection();


HikariCP:

pool থেকে একটা free connection দেয়

নতুন connection বানায় না

3️⃣ কাজ শেষ হলে
connection.close();


⚠️ এখানে আসলে connection close হয় না ❌
✔️ pool-এ ফেরত চলে যায়

🔹 Spring Boot-এ HikariCP Flow
Controller
   ↓
Service
   ↓
Repository (JPA/JDBC)
   ↓
HikariCP Pool
   ↓
PostgreSQL

🔹 গুরুত্বপূর্ণ Configuration (বাংলায়)
spring.datasource.hikari.maximum-pool-size=10


👉 একসাথে সর্বোচ্চ 10টা connection ব্যবহার করা যাবে

spring.datasource.hikari.minimum-idle=5


👉 সবসময় কমপক্ষে 5টা connection idle থাকব
```
প্রতিটা Physical DB Connection তৈরি করতে অনেক সময় লাগে।
এই সময় কমানোর জন্য HikariCP আগেই কিছু Physical Connection তৈরি করে রাখে (reserve করে)। 
      var con = DriverManager.getConnection("jdbc:postgresql://localhost:5432/sildb", (it take 48 ms for take a connection)
               "postgres",
               "1234");
     var stm= con.createStatement();
        long st = System.currentTimeMillis();
     ResultSet rs = stm.executeQuery("select * from post");
        while (rs.next()) {
            System.out.println(
                    "title is : " + rs.getString("title") +
                            ", content is : " + rs.getString("content")
            );
        }

        stm.close();
     con.close();
     long et = System.currentTimeMillis();
       System.out.println("total time consume is ..................."+(et-st));ে
```
## Repository Hierache :

<img width="1432" height="972" alt="image" src="https://github.com/user-attachments/assets/2240aed3-2172-49e2-87a8-7ef66a478585" />
