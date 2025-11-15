## API slow হলে এর পিছনে ৩টি প্রধান কারণ থাকে

```
Excellent question 💡 — তুমি একদম প্র্যাক্টিক্যাল জায়গায় হিট করেছো।
Spring Boot + PostgreSQL প্রজেক্টে API slow হলে এর পিছনে ৩টি প্রধান কারণ থাকে —
(১) Query design, (২) Database structure, (৩) I/O (network / serialization) bottleneck.

চলো ধাপে ধাপে দেখি কীভাবে API কে দ্রুত করা যায় ⚡👇

🧩 1️⃣ Query Optimization — (সবচেয়ে গুরুত্বপূর্ণ)
🧠 কারণ

JPA (Hibernate) অনেক সময় lazy loading / joins / N+1 queries করে, যেটা huge data এ খুব slow হয়।

✅ সমাধান
🔹 Use JPQL / Native Query

তুমি আগেই custom JPQL ব্যবহার করছো — এটা ভালো 👍
কিন্তু performance-critical হলে native SQL query ব্যবহার করো:

@Query(
  value = """
    SELECT u.first_name, c.title, e.enrolled_at
    FROM enrollment e
    JOIN users u ON e.user_id = u.id
    JOIN course c ON e.course_id = c.id
    WHERE u.id = :userId
  """,
  nativeQuery = true
)
List<Object[]> getEnrollmentsFast(@Param("userId") Long userId);


➡ Native query অনেক দ্রুত চলে কারণ Hibernate overhead কম।

🧩 2️⃣ Use DTO Projection (Only Select Needed Fields)

Hibernate যখন entity লোড করে তখন সব column ফেচ করে।
যদি শুধু কিছু ফিল্ড দরকার হয়, DTO projection ব্যবহার করো:

@Query("""
  SELECT new com.sil.digitalbankingbackend.dtos.EnrollmentDTO(
    u.id, u.firstName, c.title, e.enrolledAt
  )
  FROM Enrollment e
  JOIN e.user u
  JOIN e.course c
""")
List<EnrollmentDTO> findEnrollmentSummary();


➡ এতে Hibernate শুধু প্রয়োজনীয় কলাম ফেচ করে — গতি বেড়ে যায়।

🧩 3️⃣ Add Proper Indexes in PostgreSQL

তোমার সম্পর্কগুলোতে (foreign key + frequently filtered column) index থাকা খুব দরকার।

CREATE INDEX idx_user_email ON users(email);
CREATE INDEX idx_enrollment_user ON enrollment(user_id);
CREATE INDEX idx_enrollment_course ON enrollment(course_id);
CREATE INDEX idx_course_category ON course(category_id);


➡ Filtering / joining অনেক দ্রুত হবে 🚀

🧩 4️⃣ Pagination ব্যবহার করো (তুমি ইতিমধ্যে করছো ✅)

LIMIT এবং OFFSET ব্যবহার করে pagination করলে একসাথে হাজার data না এনে কেবল কিছু রেকর্ড আসে।
Spring Data JPA-র Pageable system এটাই করে।

🧩 5️⃣ Enable 2nd Level Cache (Hibernate Cache)

Hibernate তোমার query result cache করে রাখতে পারে,
যাতে একই query বারবার গেলে database hit না করে cache থেকে দেয়।

✳️ Step:
spring.jpa.properties.hibernate.cache.use_second_level_cache=true
spring.jpa.properties.hibernate.cache.region.factory_class=org.hibernate.cache.jcache.JCacheRegionFactory
spring.cache.jcache.config=classpath:ehcache.xml


➡ এতে repeated query এর response অনেক দ্রুত আসবে ⚡

🧩 6️⃣ Use FetchType.LAZY for Relationships

তুমি যদি সব relation eager fetch করো, Hibernate সব related entity লোড করে — যেটা অনেক সময় লাগে।

✅ করো এমনভাবে:

@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "user_id")
private User user;


➡ Lazy load মানে কেবল তখনি data load হবে যখন দরকার হবে।

🧩 7️⃣ Use Response Compression (Network Speed)

Spring Boot এ GZIP compression enable করলে payload ছোট হয়।

server.compression.enabled=true
server.compression.mime-types=application/json,application/xml,text/html,text/xml,text/plain
server.compression.min-response-size=1024


➡ Network latency কমবে 📡

🧩 8️⃣ Connection Pooling (HikariCP)

Spring Boot by default HikariCP ব্যবহার করে (খুবই দ্রুত)।
তবে যদি thread বা load বেশি হয়, pool size ঠিক রাখো:

spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=5

🧩 9️⃣ Caching at Service Layer

যদি একই user এর data বারবার আসে, Spring Cache ব্যবহার করো:

@Cacheable("enrollments")
public List<EnrollmentDTO> getUserEnrollments(Long userId) {
    return enrollmentRepository.findByUserId(userId);
}


➡ ২য় বার call করলে সরাসরি cache থেকে result দেবে ⚡

🧩 10️⃣ Profile & Monitor (JProfiler / VisualVM / pg_stat_statements)

Performance বুঝতে Hibernate log enable করো:

spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.generate_statistics=true


PostgreSQL এ slow query monitor করতে পারো:

SELECT * FROM pg_stat_statements ORDER BY total_exec_time DESC LIMIT 5;

🚀 Summary — Fast API Checklist
সমস্যা	সমাধান
Query ধীর	Native query / DTO projection
Relationship heavy	FetchType.LAZY
Filtering slow	Proper indexing
Repeated query	Cache (Hibernate + Spring Cache)
Large payload	GZIP compression
Database bottleneck	Connection pool tuning
Bulk data	Pagination
```
### প্রজেক্ট গঠন (Core Entities)
```
User (1) ───< Enrollment >─── (∞) Course (∞) ───< Category (1)

1️⃣ Entity Classes (Optimized with Lazy Loading & Index)
🧠 User.java
@Entity
@Table(name = "users", indexes = {
        @Index(columnList = "email", name = "idx_user_email")
})
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String firstName;
    private String lastName;

    @Column(unique = true, nullable = false)
    private String email;

    private String password;

    @Builder.Default
    private String role = "USER";

    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Enrollment> enrollments;
}

📚 Course.java
@Entity
@Table(indexes = @Index(columnList = "title", name = "idx_course_title"))
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Course {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "category_id")
    private Category category;
}

🗂 Category.java
@Entity
@Table(name = "categories")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Category {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String description;

    @OneToMany(mappedBy = "category", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Course> courses;
}

🎓 Enrollment.java
@Entity
@Table(indexes = {
        @Index(columnList = "user_id", name = "idx_enrollment_user"),
        @Index(columnList = "course_id", name = "idx_enrollment_course")
})
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Enrollment {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    private User user;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "course_id")
    private Course course;

    private LocalDateTime enrolledAt;
}

2️⃣ DTO for Projection (Only Fetch What You Need)
@Data
@AllArgsConstructor
@NoArgsConstructor
public class EnrollmentDTO {
    private String userName;
    private String courseTitle;
    private String categoryName;
    private LocalDateTime enrolledAt;
}

3️⃣ Repository Layer (Optimized JPQL Query)
@Repository
public interface EnrollmentRepository extends JpaRepository<Enrollment, Long> {

    @Query("""
        SELECT new com.sil.digitalbankingbackend.dtos.EnrollmentDTO(
            CONCAT(u.firstName, ' ', u.lastName),
            c.title,
            cat.name,
            e.enrolledAt
        )
        FROM Enrollment e
        JOIN e.user u
        JOIN e.course c
        JOIN c.category cat
        WHERE 
            (:userId IS NULL OR u.id = :userId)
            AND (:categoryName IS NULL OR LOWER(cat.name) LIKE LOWER(CONCAT('%', :categoryName, '%')))
    """)
    Page<EnrollmentDTO> filterEnrollments(
            @Param("userId") Long userId,
            @Param("categoryName") String categoryName,
            Pageable pageable
    );
}

4️⃣ Service Layer (Cache + Pagination)
@Service
@RequiredArgsConstructor
public class EnrollmentService {

    private final EnrollmentRepository enrollmentRepository;

    @Cacheable(value = "enrollmentCache", key = "#userId + '-' + #categoryName + '-' + #page")
    public Page<EnrollmentDTO> getFilteredEnrollments(
            Long userId, String categoryName, int page, int size
    ) {
        Pageable pageable = PageRequest.of(page, size, Sort.by("enrolledAt").descending());
        return enrollmentRepository.filterEnrollments(userId, categoryName, pageable);
    }
}

5️⃣ Controller Layer (Filter + Cache API)
@RestController
@RequestMapping("/api/enrollments")
@RequiredArgsConstructor
public class EnrollmentController {

    private final EnrollmentService enrollmentService;

    @GetMapping("/filter")
    public ResponseEntity<Page<EnrollmentDTO>> filterEnrollments(
            @RequestParam(required = false) Long userId,
            @RequestParam(required = false) String categoryName,
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "5") int size
    ) {
        Page<EnrollmentDTO> result = enrollmentService.getFilteredEnrollments(userId, categoryName, page, size);
        return ResponseEntity.ok(result);
    }
}

6️⃣ Enable Caching (application.properties)
spring.cache.type=simple
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.show-sql=false
spring.jpa.properties.hibernate.jdbc.fetch_size=50
spring.jpa.properties.hibernate.jdbc.batch_size=30
spring.datasource.hikari.maximum-pool-size=20

⚡ 7️⃣ PostgreSQL Index Boost

Run this once in DB:

CREATE INDEX idx_user_email ON users(email);
CREATE INDEX idx_enrollment_user ON enrollment(user_id);
CREATE INDEX idx_enrollment_course ON enrollment(course_id);
CREATE INDEX idx_course_category ON course(category_id);

✅ 8️⃣ Example API Call
GET /api/enrollments/filter?userId=1&categoryName=Programming&page=0&size=10

🔸 Output:
{
  "content": [
    {
      "userName": "Mamun Abdullah",
      "courseTitle": "Spring Boot Masterclass",
      "categoryName": "Programming",
      "enrolledAt": "2025-10-29T10:30:00"
    }
  ],
  "totalPages": 3,
  "totalElements": 25,
  "size": 10,
  "number": 0
}

🚀 এই setup এর সুবিধা
ফিচার	লাভ
Lazy loading	অপ্রয়োজনীয় relation load হয় না
DTO projection	শুধুমাত্র দরকারি কলাম ফেচ হয়
Indexing	Join / Filter query অনেক দ্রুত
Cache	Same query ২য়বার গেলে DB hit হয় না
Pagination	Huge dataset এ performance ঠিক থাকে
Connection pool tuning	Load handle করে সহজে
```
## তুমি বলছো → একটা API বানিয়েছো যেটা ৪টা টেবিল থেকে relational data join / union করে রিটার্ন দেয়,
কিন্তু সেটা slow চলছে 🐢
```
চলো দেখি ধাপে ধাপে কিভাবে এটাকে “lightning fast ⚡” করা যায় —
Spring Boot + PostgreSQL অনুযায়ী।

🧩 ১️⃣ প্রথমে বুঝি — কেন ৪টা টেবিল join/union করলে ধীর হয়

যখন তুমি এভাবে করো:

SELECT * 
FROM enrollment e
JOIN user u ON e.user_id = u.id
JOIN course c ON e.course_id = c.id
JOIN category cat ON c.category_id = cat.id;


তখন Hibernate প্রতিটি row-এর জন্য join result তৈরি করে।
যদি টেবিলগুলো বড় হয় (যেমন ১ লাখ enrollment, ৫০ হাজার user, ইত্যাদি),
তাহলে DB CPU ও Memory অনেক ব্যবহার করে → ফলাফল ধীর হয়।

✅ সমাধান Step-by-Step
🧠 Step 1: শুধু দরকারি কলাম নাও (Full Entity নয়)

যদি তুমি Entity পুরোটা রিটার্ন করো → Hibernate lazy load / proxy তৈরি করে, যেটা অনেক heavy হয়।

এর পরিবর্তে DTO Projection ব্যবহার করো 👇

@Query("""
SELECT new com.sil.digitalbankingbackend.dtos.EnrollmentViewDTO(
    u.firstName,
    c.title,
    cat.name,
    e.enrolledAt
)
FROM Enrollment e
JOIN e.user u
JOIN e.course c
JOIN c.category cat
""")
List<EnrollmentViewDTO> getJoinedData();


➡ এটা শুধুমাত্র ৪টা দরকারি কলাম ফেচ করবে, সব entity না।
⚡ Performance অনেক বেড়ে যাবে।

🧠 Step 2: Index যোগ করো (Join কলামে)

যতগুলো টেবিল join হচ্ছে, তাদের join কলামে index থাকতে হবে।

CREATE INDEX idx_enrollment_user_id ON enrollment(user_id);
CREATE INDEX idx_enrollment_course_id ON enrollment(course_id);
CREATE INDEX idx_course_category_id ON course(category_id);


➡ PostgreSQL join দ্রুত resolve করবে 🚀

🧠 Step 3: N+1 Query সমস্যা চেক করো

Hibernate অনেক সময় নিচের মতো করে ১টা main query এর পর
related entity আলাদা আলাদা query করে ফেলে 👇

select * from enrollment;
select * from user where id=?;
select * from course where id=?;
select * from category where id=?;


➡ এটা অনেক slow।

✅ সমাধান:

@EntityGraph(attributePaths = {"user", "course", "course.category"})
@Query("SELECT e FROM Enrollment e")
List<Enrollment> getAllEnrollmentsWithRelations();


এতে Hibernate একসাথে join করবে → একবারেই সব ডাটা পাবে।

🧠 Step 4: Pagination ব্যবহার করো

যদি API তে সব ডাটা রিটার্ন করো (যেমন ১০,০০০ রো) → API অনেক slow হবে।
✅ শুধু দরকারি page ফেচ করো:

Pageable pageable = PageRequest.of(page, size, Sort.by("enrolledAt").descending());
return enrollmentRepository.findAll(pageable);


➡ Database কেবল ২০-৫০ রো ফেরত দেবে, পুরোটা না।

🧠 Step 5: Native Query ব্যবহার করো (Complex Join/Union এর জন্য)

Hibernate query translation slow হয়।
তুমি যদি heavy UNION ব্যবহার করো, তখন Native SQL ব্যবহার করো 👇

@Query(value = """
SELECT u.first_name, c.title, cat.name, e.enrolled_at 
FROM enrollment e
JOIN users u ON e.user_id = u.id
JOIN course c ON e.course_id = c.id
JOIN category cat ON c.category_id = cat.id
UNION
SELECT u.first_name, c.title, cat.name, NOW() as enrolled_at 
FROM other_table o
JOIN users u ON o.user_id = u.id
JOIN course c ON o.course_id = c.id
JOIN category cat ON c.category_id = cat.id
""", nativeQuery = true)
List<Map<String, Object>> getUnionData();


➡ Native query সরাসরি PostgreSQL এক্সিকিউট করে, Hibernate কে এড়িয়ে যায়।
এই কারণে এটি অনেক দ্রুত ⚡

🧠 Step 6: Database View ব্যবহার করো (Best Practice 💥)

তুমি যদি প্রতিবার একই join/union করো, তাহলে DB তে একটা VIEW তৈরি করো:

CREATE VIEW enrollment_summary AS
SELECT e.id, u.first_name, c.title, cat.name, e.enrolled_at
FROM enrollment e
JOIN users u ON e.user_id = u.id
JOIN course c ON e.course_id = c.id
JOIN category cat ON c.category_id = cat.id;


তারপর তোমার Spring Boot থেকে শুধু:

@Query(value = "SELECT * FROM enrollment_summary", nativeQuery = true)
List<Map<String, Object>> getFastSummary();


➡ PostgreSQL view data pre-optimized থাকে — অনেক দ্রুত চলে 🚀

🧠 Step 7: Result Cache করো (Spring Cache বা Redis)

যদি একই filter/union API অনেক ইউজার একসাথে হিট করে →
তাহলে cache ব্যবহার করো:

@Cacheable("unionResults")
public List<EnrollmentViewDTO> getUnionDataCached() {
    return enrollmentRepository.getUnionData();
}


➡ পরের বার query run না হয়ে সরাসরি cache থেকে data আসবে (millisecond level) ⚡

🧠 Step 8: Connection Pool Tune করো
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=5
spring.jpa.properties.hibernate.jdbc.fetch_size=50
spring.jpa.properties.hibernate.jdbc.batch_size=30


➡ অনেক concurrent request handle করবে কোনো latency ছাড়া।

🧠 Step 9: Database EXPLAIN ব্যবহার করো

তোমার query slow হলে PostgreSQL-এ দেখো কোন অংশ বেশি সময় নিচ্ছে:

EXPLAIN ANALYZE
SELECT * FROM enrollment e
JOIN users u ON e.user_id = u.id
JOIN course c ON e.course_id = c.id
JOIN category cat ON c.category_id = cat.id;


এখানে “Seq Scan” দেখালে বুঝবে index লাগাতে হবে।

✅ Summary Table
সমস্যা	সমাধান	লাভ
Join heavy query	DTO projection / Native query	Hibernate overhead কম
Filter slow	Proper Index	Join/Filter গতি বৃদ্ধি
N+1 problem	EntityGraph / Fetch join	একবারেই ডাটা লোড
Large data	Pagination	Memory usage কম
Same data repeat	Cache (Spring/Redis)	Response time মিলিসেকেন্ডে
Complex join logic	Database View	Pre-optimized join
DB load বেশি	Connection pool tuning	Load handle করে
```
