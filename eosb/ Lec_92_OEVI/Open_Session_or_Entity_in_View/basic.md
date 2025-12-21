## Open Session in View / Open Entity in View কী?

Hibernate / JPA ব্যবহার করলে, ডাটাবেস কানেকশন সাধারণত Service Layer পর্যন্ত open থাকে

Controller বা View render করার সময় Lazy-loaded relationships access করলে LazyInitializationException আসে

সমাধান হিসেবে Spring একটি pattern ব্যবহার করে → Open Session in View

এর মানে:
Hibernate Session / JPA EntityManager পুরো HTTP request lifetime পর্যন্ত open থাকবে
তাই Controller/Thymeleaf/JSON Serializer যেকোনো সময় Lazy-loaded association access করতে পারবে।
