### Caching ব্যবহার করি?
```
ধরুন, একটি API আছে:

@GetMapping("/user/{id}")
public User getUser(Long id){
    return userRepository.findById(id).get();
}

প্রতি Request এ Database Hit হবে।

Request 1 -> Database
Request 2 -> Database
Request 3 -> Database

যদি একই User Data বারবার Request করা হয়, তাহলে অপ্রয়োজনীয় Database Load তৈরি হবে।

Redis Cache ব্যবহার করলে:

Request 1 -> Database -> Redis Cache
Request 2 -> Redis Cache
Request 3 -> Redis Cache

ফলে Response Time অনেক কমে যায়।

@Cacheable

সবচেয়ে বেশি ব্যবহৃত Annotation।

@Cacheable(value = "users", key = "#id")
public User getUser(Long id){
    return userRepository.findById(id).get();
}
প্রথমবার Call
getUser(100)
Redis এ Data নেই
Database থেকে Data আনবে
Redis এ Store করবে
দ্বিতীয়বার Call
getUser(100)
Redis এ Data আছে
Database এ যাবে না
Cache থেকে Return করবে
@CachePut

Cache Update করার জন্য ব্যবহার করা হয়।

@CachePut(value = "users", key = "#user.id")
public User updateUser(User user){
    return userRepository.save(user);
}
কাজ
Method Execute হবে
Database Update হবে
Cache-ও Update হবে
@CacheEvict

Cache Delete করার জন্য ব্যবহার করা হয়।

@CacheEvict(value = "users", key = "#id")
public void deleteUser(Long id){
    userRepository.deleteById(id);
}
কাজ
Database থেকে Delete
Redis Cache থেকেও Delete
@Caching

একসাথে Multiple Cache Operation করার জন্য।

@Caching(
    put = {
        @CachePut(value = "users", key = "#user.id")
    },
    evict = {
        @CacheEvict(value = "allUsers", allEntries = true)
    }
)
public User updateUser(User user){
    return userRepository.save(user);
}
Redis Cache Flow
Client Request
      |
      v
 @Cacheable
      |
      v
Redis Cache Check
      |
   Found?
   /    \
 Yes     No
  |       |
Return   Database Query
Cache         |
              v
        Save to Redis
              |
              v
          Return Data
Spring Boot Configuration
@EnableCaching
@SpringBootApplication
public class Application {
}

Service:

@Service
public class UserService {

    @Cacheable(value = "users", key = "#id")
    public User getUser(Long id) {
        return userRepository.findById(id).orElse(null);
    }

```
