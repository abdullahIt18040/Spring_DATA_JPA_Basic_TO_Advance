## Native Query (নেটিভ কুয়েরি)
ব্যাখ্যা:
Native Query হলো একটি সরাসরি ডাটাবেস কুয়েরি (যেমন: SQL)। আপনি যখন Entity Manager বা Spring Data JPA ব্যবহার করে নেটিভ কুয়েরি লেখেন, তখন আপনি সরাসরি আপনার ডাটাবেসের SQL সিনট্যাক্স ব্যবহার করতে পারেন। এটি বিশেষভাবে উপকারী যখন আপনাকে জটিল জয়েন, এগ্রিগেট ফাংশন, বা ডাটাবেস-স্পেসিফিক ফিচার ব্যবহার করতে হয়।
```
বৈশিষ্ট্য:
সরাসরি SQL সিনট্যাক্স

ডাটাবেস-স্পেসিফিক (PostgreSQL, MySQL ইত্যাদি)

JPQL এর চেয়ে বেশি ফ্লেক্সিবল

কিন্তু কম পোর্টেবল (ডাটাবেস পরিবর্তন হলে কুয়েরি পরিবর্তন করতে হতে পারে)

উদাহরণ (PostgreSQL):
ধরি আপনার একটি employees টেবিল আছে:

sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    salary DECIMAL(10,2),
    department VARCHAR(50)
);
Spring Data JPA তে Native Query:

java
@Repository
public interface EmployeeRepository extends JpaRepository<Employee, Long> {
    
    // নেটিভ কুয়েরি - সরাসরি SQL
    @Query(value = "SELECT * FROM employees WHERE salary > :minSalary", nativeQuery = true)
    List<Employee> findEmployeesWithSalaryGreaterThan(@Param("minSalary") BigDecimal minSalary);
    
    // জটিল নেটিভ কুয়েরি
    @Query(value = "SELECT department, AVG(salary) as avg_salary FROM employees GROUP BY department", nativeQuery = true)
    List<Object[]> findAverageSalaryByDepartment();
}
Entity Manager দিয়ে Native Query:

java
@PersistenceContext
private EntityManager entityManager;

public List<Employee> findHighPaidEmployees() {
    String nativeQuery = "SELECT * FROM employees WHERE salary > 50000";
    Query query = entityManager.createNativeQuery(nativeQuery, Employee.class);
    return query.getResultList();
}
```
### JPQL (Java Persistence Query Language)
ব্যাখ্যা:
JPQL হলো একটি অবজেক্ট-ওরিয়েন্টেড কুয়েরি ল্যাঙ্গুয়েজ যা ডাটাবেস টেবিলের বদলে Entity ক্লাস এবং তাদের ফিল্ডের উপর কাজ করে। JPQL কুয়েরি শেষে SQL এ রূপান্তরিত হয়। এটি ডাটাবেস-ইনডিপেন্ডেন্ট।
```
বৈশিষ্ট্য:
Entity এবং ফিল্ড নাম ব্যবহার করে

ডাটাবেস-ইনডিপেন্ডেন্ট

টাইপ-সেফ

OOP কনসেপ্ট সাপোর্ট করে

উদাহরণ:
ধরি আপনার একটি Employee Entity ক্লাস আছে:

java
@Entity
@Table(name = "employees")
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    private BigDecimal salary;
    private String department;
    
    // getters and setters
}
Spring Data JPA তে JPQL:

java
@Repository
public interface EmployeeRepository extends JpaRepository<Employee, Long> {
    
    // সাধারণ JPQL কুয়েরি
    @Query("SELECT e FROM Employee e WHERE e.salary > :minSalary")
    List<Employee> findBySalaryGreaterThan(@Param("minSalary") BigDecimal minSalary);
    
    // জয়েন সহ JPQL
    @Query("SELECT e.department, AVG(e.salary) FROM Employee e GROUP BY e.department")
    List<Object[]> findDepartmentWiseAverageSalary();
    
    // UPDATE অপারেশন
    @Modifying
    @Query("UPDATE Employee e SET e.salary = e.salary * 1.1 WHERE e.department = :dept")
    int giveRaiseToDepartment(@Param("dept") String department);
}
Entity Manager দিয়ে JPQL:

java
@PersistenceContext
private EntityManager entityManager;

public List<Employee> findEmployeesByDepartment(String department) {
    String jpql = "SELECT e FROM Employee e WHERE e.department = :dept";
    TypedQuery<Employee> query = entityManager.createQuery(jpql, Employee.class);
    query.setParameter("dept", department);
    return query.getResultList();
}
```
