## Open Session in View / Open Entity in View কী?

Hibernate / JPA ব্যবহার করলে, ডাটাবেস কানেকশন সাধারণত Service Layer পর্যন্ত open থাকে

Controller বা View render করার সময় Lazy-loaded relationships access করলে LazyInitializationException আসে

সমাধান হিসেবে Spring একটি pattern ব্যবহার করে → Open Session in View

এর মানে:
Hibernate Session / JPA EntityManager পুরো HTTP request lifetime পর্যন্ত open থাকবে
তাই Controller/Thymeleaf/JSON Serializer যেকোনো সময় Lazy-loaded association access করতে পারবে।

```

2️⃣ Lazy Loading সমস্যা
Example Entities
@Entity
public class Post {

    @Id
    @GeneratedValue
    private Long id;

    private String title;

    @OneToMany(mappedBy = "post", fetch = FetchType.LAZY)
    private List<Comment> comments;
}

@Entity
public class Comment {

    @Id
    @GeneratedValue
    private Long id;

    private String content;

    @ManyToOne
    private Post post;
}

❌ Without OSIV
@GetMapping("/posts/{id}")
public Post getPost(@PathVariable Long id) {
    Post post = postRepository.findById(id).orElseThrow();
    // Lazy-loaded comments access outside service
    System.out.println(post.getComments().size()); // ❌ LazyInitializationException
    return post;
}


Reason:

post.getComments() fetch হয়নি

Session/EntityManager already closed at controller layer

🔹 3️⃣ OSIV Enable করলে
Spring Boot default (application.properties)
spring.jpa.open-in-view=true


Spring Boot default true (JPA)

Hibernate Session / EntityManager HTTP request পুরো সময় open থাকে

✅ Controller Now Works
@GetMapping("/posts/{id}")
public Post getPost(@PathVariable Long id) {
    Post post = postRepository.findById(id).orElseThrow();
    System.out.println(post.getComments().size()); // ✅ Works fine
    return post;
}


Lazy-loaded comments access করা যায়

Session open থাকায় exception আসে না

🔹 4️⃣ Pros and Cons
Pros	Cons
সহজে lazy relationships access করা যায়	Performance issue: unnecessary queries during view rendering
Controller/JSON serialization সহজ	N+1 select problem হতে পারে
Default Spring Boot behavior	Session অনেক সময় বেশি open থাকে (long request)
	Transaction boundary blur হয়
🔹 5️⃣ Alternative Approach (Recommended for Production)

OSIV Disable করে explicit fetch করা better

spring.jpa.open-in-view=false

Service Layer
@Transactional
public Post getPostWithComments(Long id) {
    Post post = postRepository.findById(id).orElseThrow();
    post.getComments().size(); // Fetch inside transaction
    return post;
}

```
Lazy loading ঠিকভাবে transaction scope এ হয়

N+1 query problem এর জন্য @EntityGraph বা join fetch ব্যবহার করা যায়
