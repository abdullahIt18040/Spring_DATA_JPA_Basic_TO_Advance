### Mapped BY 

mappedBy আসলে Hibernate / JPA-কে বলে দেয়,
👉 “এই সম্পর্কটার মালিক আমি না — অন্য entity মালিক।”
```
🧩 কেন mappedBy ব্যবহার করা হয়?

ধরো, তোমার দুইটা entity আছে —
Category আর Course

একটা Category-এর মধ্যে অনেক Course থাকতে পারে
(যেমন "Programming" ক্যাটাগরিতে Java, Python, C++ কোর্স আছে)

🎯 যদি আমরা mappedBy না দিই

তাহলে Hibernate বুঝবে না কে মালিক,
ফলে সে অতিরিক্ত একটা টেবিল (join table) বানাবে।

কিন্তু আমরা চাই না — কারণ Course টেবিলেই category_id রেখে কাজ হয়ে যাবে।

🧠 তাহলে mappedBy দিয়ে কী হয়?

mappedBy দিয়ে Hibernate-কে বলি —
👉 “Foreign key category_id রাখা আছে Course টেবিলে, তাই তুমি আবার তৈরি কোরো না।”

🔹 উদাহরণ
🧱 Category.java
@Entity
public class Category {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    // এখানে mappedBy দেওয়া হয়েছে কারণ Foreign key অন্য পাশে আছে
    @OneToMany(mappedBy = "category")
    private List<Course> courses = new ArrayList<>();
}

🧱 Course.java
@Entity
public class Course {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    // এই দিকেই আসল Foreign key থাকবে
    @ManyToOne
    @JoinColumn(name = "category_id")
    private Category category;
}

🧮 টেবিলে কী হবে
Table	Columns
category	id, name
course	id, title, category_id (FK)

👉 Hibernate category_id কলামটা শুধু course টেবিলে রাখবে।
যদি mappedBy না দিতাম, তাহলে তৃতীয় একটা “category_course” নামের join table বানাত।

🧠 সংক্ষিপ্তভাবে মনে রাখো
দিক	কাজ
Owning Side (মালিক দিক)	যেটা টেবিলে foreign key রাখে (যেমন Course.category_id)
Inverse Side (mappedBy দিক)	শুধু reference রাখে, কিন্তু নিজের টেবিলে FK রাখে না
mappedBy কী বলে	“অন্য entity এই সম্পর্কের মালিক”

চলো সহজভাবে দেখি — যদি তুমি mappedBy না দাও, তাহলে Hibernate কী করে 👇

🧩 উদাহরণ (Category – Course)
✅ Category.java
@Entity
public class Category {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @OneToMany
    private List<Course> courses = new ArrayList<>();
}

✅ Course.java
@Entity
public class Course {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String title;

    @ManyToOne
    private Category category;
}

🧠 Hibernate এখন কী করবে?

যেহেতু তুমি mappedBy দাওনি, Hibernate বুঝতে পারছে না কে owner (মালিক) —
মানে, কার টেবিলে foreign key থাকবে।

তাই Hibernate ভাবে:

“দু’পাশেই relationship আছে, তাই আমি একটা নতুন join table বানিয়ে রাখি!”

🧾 টেবিল গঠন (যদি mappedBy না দাও)
Table Name	Columns
category	id, name
course	id, title, category_id
category_courses	category_id, course_id ✅ (এইটা extra)

👉 দেখো — Hibernate একটা অতিরিক্ত টেবিল বানিয়ে ফেলল category_courses নামে।

এটা বেশিরভাগ সময় অপ্রয়োজনীয়, কারণ আমাদের আসলে course টেবিলেই category_id রাখলেই চলত।

✅ কিন্তু mappedBy দিলে কী হয়?
@OneToMany(mappedBy = "category")
private List<Course> courses;


এখন Hibernate বুঝে গেল —

“ঠিক আছে, Course entity-এর category ফিল্ডেই foreign key আছে, তাই আমি নতুন join table বানাব না।”

🎯 সংক্ষেপে
অবস্থা	কী হয়
❌ mappedBy না দিলে	Hibernate একটা অতিরিক্ত join table বানায়
✅ mappedBy দিলে	Hibernate শুধু এক পাশেই foreign key রাখে (ডুপ্লিকেট হয় না)
```
