## N+1 query problem and how to fix it in PostgreSQL + Java/Spring Boot with examples
```

What is the N+1 Query Problem? (Easy Explanation)

Suppose you have 2 tables:

1. Students
id	name
1	Mamun
2	Rafi
2. Courses
id	student_id	course_name
1	1	Math
2	1	English
3	2	Physics
❗ Problem

You write a query (or JPA repository call) to load all students:

List<Student> students = studentRepository.findAll();


Now for each student, you access student.getCourses()

❗ What happens internally?

✔ First query:

SELECT * FROM student;


This loads 2 students → Mamun & Rafi.

Then for each student, JPA loads courses separately:

✔ Query 2:

SELECT * FROM course WHERE student_id = 1;


✔ Query 3:

SELECT * FROM course WHERE student_id = 2;

➝ Total Queries = N (students) + 1 (main query)

That’s why the problem is called N+1.

❌ Why is this bad?

Because if you have:

✔ 2 students → 3 queries
✔ 100 students → 101 queries
✔ 10,000 students → 10,001 queries (very slow)

This kills performance in PostgreSQL.

🟢 How to Fix N+1 in PostgreSQL + Spring Boot?

There are 3 common solutions:

✔ Fix #1: Use JOIN FETCH (Best Method)

Tell JPA to fetch related data in one query.

Solution in Repository
@Query("SELECT s FROM Student s LEFT JOIN FETCH s.courses")
List<Student> findAllWithCourses();

SQL generated:
SELECT s.*, c.* 
FROM student s
LEFT JOIN course c ON s.id = c.student_id;

Result ➝ Only 1 query

No more N+1 problem.

✔ Fix #2: Use @EntityGraph (Cleaner Code)
@EntityGraph(attributePaths = {"courses"})
@Query("SELECT s FROM Student s")
List<Student> findAllWithCourses();


This also runs in one SQL query.

✔ Fix #3: Change Fetch Type to EAGER (Not Recommended)
@OneToMany(mappedBy = "student", fetch = FetchType.EAGER)
private List<Course> courses;


But this can cause heavy queries and memory use.

❌ Not recommended for production.

🧪 Example Output
Before Fix (N+1 problem)
1) SELECT * FROM student;
2) SELECT * FROM course WHERE student_id=1;
3) SELECT * FROM course WHERE student_id=2;


3 queries.

After Fix (JOIN FETCH)
1) SELECT s.*, c.* FROM student s
   LEFT JOIN course c ON s.id=c.student_id;


Only 1 query.

🧠 Extra Tip to Check N+1 Issue

Enable SQL logging in Spring Boot:

spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true


You will clearly see if too many queries are running.

🎯 Summary
Method	Fixes N+1?	Recommended?
JOIN FETCH	✅	⭐ Best
@EntityGraph	✅	⭐ Best
EAGER Fetch	⚠️ Sometimes	❌ Not recommended
Lazy Fetch + No JOIN	❌	—

```
### very simple beginner-friendly explanation of JOIN FETCH — what it is, why we use it, and how it fixes the N+1 query problem in Java Spring Boot + PostgreSQL.
```
 What is JOIN FETCH? (Easy Explanation)
JOIN FETCH = Load parent + child data in one single query

It tells Hibernate:
“Load the main entity AND also load the related entities together.”

So instead of:

First query (load students)

Then N more queries (load courses for each student)

JOIN FETCH loads:

all students

all their courses
➡️ in ONE SQL query

❗ Why do we need JOIN FETCH?

Because by default:

@OneToMany(fetch = FetchType.LAZY)
private List<Course> courses;


Lazy means:
→ courses are loaded only when getCourses() is called.

That causes N+1 queries.

⭐ JOIN FETCH fixes this
Repository:
@Query("SELECT s FROM Student s LEFT JOIN FETCH s.courses")
List<Student> findAllWithCourses();

🧠 What does JOIN FETCH mean?
JOIN

Normal SQL join (student + course tables combine).

FETCH

Tell Hibernate:

“Immediately load this relationship into memory.”

Not lazy, not later — load now.

🎯 What actually happens behind the scenes?
Without JOIN FETCH:
SELECT * FROM student;            -- 1 main query
SELECT * FROM course WHERE student_id = 1;  -- query 2
SELECT * FROM course WHERE student_id = 2;  -- query 3
... (N queries)

With JOIN FETCH (only 1 query):
SELECT s.*, c.*
FROM student s
LEFT JOIN course c ON s.id = c.student_id;


ONE query returns ALL the data.

📌 Full Example for Beginners
Step 1: Entities
Student Entity
@Entity
public class Student {

    @Id
    private Long id;
    private String name;

    @OneToMany(mappedBy = "student", fetch = FetchType.LAZY)
    private List<Course> courses;
}

Course Entity
@Entity
public class Course {

    @Id
    private Long id;
    private String courseName;

    @ManyToOne
    @JoinColumn(name = "student_id")
    private Student student;
}

Step 2: Normal Query (slow)
students = studentRepository.findAll();

for (Student s : students) {
    s.getCourses();   // causes extra queries
}


Total: N+1 queries.

Step 3: JOIN FETCH Query (fast)
Repository Method:
@Query("SELECT s FROM Student s LEFT JOIN FETCH s.courses")
List<Student> findAllWithCourses();

Service:
List<Student> students = studentRepository.findAllWithCourses();


➡️ All students + their courses are already loaded.

🟢 Benefits of JOIN FETCH
Benefit	Explanation
1. Only 1 SQL query	No N+1 problem
2. High performance	PostgreSQL runs faster
3. Easy to understand	Only a simple JPQL query
4. No EAGER loading problems	Still uses LAZY by default
```
### How @EntityGraph compares with JOIN FETCH
```
Both solve the N+1 problem, but work differently.

⭐ JOIN FETCH (JPQL way)

You manually write a JOIN:

@Query("SELECT s FROM Student s LEFT JOIN FETCH s.courses")
List<Student> findAllWithCourses();

➕ Advantages
Advantage	Explanation
Full control	You customize join type, filters, where conditions
Supports multiple fetches	Fetch multiple relationships
Simple for complex queries	Good for advanced JPQL
➖ Disadvantages
Issue	Why
Cannot be used with Pageable	Hibernate breaks due to duplicate parent rows
Complex for big projects	Query becomes long & hard to maintain
⭐ @EntityGraph (Annotation way)

You let Spring tell Hibernate:

“When loading this entity, also load these relationships.”

@EntityGraph(attributePaths = {"courses"})
@Query("SELECT s FROM Student s")
List<Student> findAllWithCourses();

➕ Advantages
Advantage	Explanation
Cleaner code	No JOIN writing manually
Works with findAll(), findById(), etc.	Hibernate handles fetch internally
Easier to maintain	Good for large projects
Supports pagination	Better than JOIN FETCH for pageable
➖ Disadvantages
Issue	Why
Less control	You cannot customize custom JOIN conditions
Limited to simple fetch needs	Not ideal for complex joins
⭐ Summary: Which one to use?
Feature	JOIN FETCH	@EntityGraph
N+1 Fix	✔	✔
Pagination support	❌	✔
Complex query	✔	△
Easy & clean	△	✔
Multiple joins	✔	✔
✅ 2. When JOIN FETCH Should NOT Be Used

JOIN FETCH is powerful, but sometimes dangerous.

❌ 1. When using Pagination (Pageable)

Example:

Page<Student> page = studentRepo.findAll(PageRequest.of(0, 10));


If you add JOIN FETCH:

@Query("SELECT s FROM Student s LEFT JOIN FETCH s.courses")
Page<Student> findAllStudents(Pageable pageable);


➡ This will fail or produce wrong results.

❗ Why?

Because JOIN FETCH duplicates rows:

Student: Mamun
Courses: Math, English


Query result becomes:

(Mamun, Math)
(Mamun, English)


Pagination counts these as TWO rows.
So page number becomes WRONG.

Hibernate forbids it.

❌ 2. When child table has huge data

Example:

Student → 10,000 courses
JOIN FETCH loads EVERYTHING — bad performance.

Use lazy loading instead.

❌ 3. When you need multiple JOIN FETCH on collection relationships

Example:

Student → courses (list)
Student → addresses (list)


This gives a Cartesian explosion:

Every course × every address


Result size becomes HUGE.

❌ 4. When you need filtering on child table

Example:

You want courses where status = 'ACTIVE':

LEFT JOIN FETCH s.courses c WHERE c.status = 'ACTIVE'


This filters courses but also filters students.
Not desired in many cases.

Better solution → @EntityGraph.

✅ 3. JOIN FETCH with Pagination (Very Important!)

JOIN FETCH and pagination do NOT work directly.

So we use one of these solutions:

⭐ Solution A: Two-query strategy (recommended)
Step 1: Fetch only parent IDs with pagination
@Query("SELECT s.id FROM Student s")
Page<Long> findStudentIds(Pageable pageable);

Step 2: Fetch students with JOIN FETCH by IDs
@Query("SELECT s FROM Student s LEFT JOIN FETCH s.courses WHERE s.id IN :ids")
List<Student> findByIdInWithCourses(@Param("ids") List<Long> ids);

Result

✔ Correct pagination
✔ Only ONE JOIN FETCH per page
✔ No duplication issues

⭐ Solution B: Use @EntityGraph with Pageable (super easy)
@EntityGraph(attributePaths = {"courses"})
Page<Student> findAll(Pageable pageable);


✔ Works perfectly
✔ Automatically removes duplicates
✔ Best for simple pagination

⭐ Solution C: Use DISTINCT with JOIN FETCH (sometimes works)
@Query("SELECT DISTINCT s FROM Student s LEFT JOIN FETCH s.courses")
List<Student> findAllWithCourses();


This fixes duplicates, BUT:

Does NOT work with pageable

Should not be used with large collections

🎯 Final Summary
When to Use What?
Situation	Best Method
Simple loading related data	@EntityGraph
Complex joins / filters	JOIN FETCH
Pagination needed	@EntityGraph + Pageable
Massive child collections	Avoid both (use DTO query instead)
Many tables join	JOIN FETCH (carefully)
```

### understand why projection is faster, how it avoids N+1, and how to use:
```
✔ Interface Projection
✔ Class-based Projection (DTO)
✔ JPQL / Native SQL Projection

✅ What is Projection? (Very Simple Explanation)

👉 Projection means: you select only the fields you need.
You do NOT load entire entity.

Example:

❌ Normal Entity Load
SELECT * FROM student;


Loads ALL columns from student table.

✔ Projection (select only required fields)
SELECT name, email FROM student;


This returns smaller data → ⏩ faster.

❗ Why do we need Projection?
✔ Faster

Because you load fewer columns.

✔ Reduces memory

Hibernate does not create full entity object.

✔ Avoids N+1 problem

Because you load exactly what you need with one query.

✔ Best for APIs

Most API endpoints don't need entire entity.

⭐ 3 Types of Projection in Spring Data JPA

Interface Projection

Class-based Projection (DTO Projection)

Native SQL or JPQL Projection

Let’s explain all three with examples.

🟢 1. Interface Projection (Most common)
Step 1: Create interface
public interface StudentView {
    String getName();
    String getEmail();
}

Step 2: Add repository method
@Query("SELECT s.name AS name, s.email AS email FROM Student s")
List<StudentView> findAllStudentViews();

SQL behind the scenes:
SELECT s.name, s.email FROM student;


➡ Only name & email selected
➡ No full entity loaded
➡ No N+1 problem

🟢 2. DTO Projection (Class-based)

Best for REST APIs because you get a real class.

Step 1: Create DTO class
public class StudentDTO {
    private String name;
    private String email;

    public StudentDTO(String name, String email) {
        this.name = name;
        this.email = email;
    }
}

Step 2: Repository
@Query("SELECT new com.example.StudentDTO(s.name, s.email) FROM Student s")
List<StudentDTO> findAllStudentDTO();

Benefits:

✔ Strong type safety
✔ Works with MapStruct
✔ Good for API response

🟢 3. Native Query Projection

When field names do not match or you need complex SQL.

Step 1: Create interface
public interface StudentNativeView {
    String getStudentName();
    String getStudentEmail();
}

Step 2: Repository
@Query(value = "SELECT name AS studentName, email AS studentEmail FROM student", 
       nativeQuery = true)
List<StudentNativeView> findNativeStudentInfo();

🔥 ⭐ Best Example: Projection to Avoid N+1

Student → Course relationship.

❌ Bad way (causes N+1)
List<Student> students = studentRepo.findAll();
students.forEach(s -> s.getCourses().size());

✔ Good way (Projection)
@Query("SELECT new com.example.StudentCourseDTO(s.name, c.courseName) 
        FROM Student s 
        JOIN s.courses c")
List<StudentCourseDTO> findStudentsWithCourses();


➡ One single query
➡ No JOIN FETCH needed
➡ No N+1
➡ DTO is small and efficient

🧠 Bonus: Nested Projection

Example:

public interface StudentCourseProjection {
    String getName();
    List<CourseView> getCourses();

    interface CourseView {
        String getCourseName();
    }
}


Repository:

List<StudentCourseProjection> findAllProjectedBy();


Hibernate generates optimized SQL.

🎯 Which Projection Should You Use?
Use Case	Best Projection
Simple API response	DTO Projection
Select few fields only	Interface Projection
Complex SQL	Native Projection
Avoid N+1	JOIN + DTO Projection
High performance	DTO Projection
⚡ Why Projection is Faster Than JOIN FETCH?

JOIN FETCH loads:

Whole parent entity

Whole child entity

Builds Hibernate objects (heavy)

Projection:

Selects only needed fields

No Hibernate object creation

Returns lightweight object

public class StudentCourseDTO {

    private String studentName;
    private String courseName;

    // Constructor MUST match the @Query parameter order
    public StudentCourseDTO(String studentName, String courseName) {
        this.studentName = studentName;
        this.courseName = courseName;
    }

    // Getters
    public String getStudentName() {
        return studentName;
    }

    public String getCourseName() {
        return courseName;
    }
}
```

