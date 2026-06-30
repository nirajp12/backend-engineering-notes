# Spring and Spring Boot

Spring is a Java framework that helps build enterprise applications by handling common infrastructure tasks like dependency injection, transaction management, and web request handling. It provides a flexible programming model and many reusable components.

Spring Boot is built on top of Spring and makes it easier to start new projects. It adds conventions, auto-configuration, and embedded servers so developers can run applications with minimal setup.

- Spring focuses on providing core features and wiring components together.
- Spring Boot focuses on simplifying project setup and running applications quickly.

Together, they let developers build Java applications faster with less configuration.

## Key Annotations

| Annotation | Purpose |
|---|---|
| `@SpringBootApplication` | Entry point of application |
| `@RestController` | Marks class as REST API controller |
| `@GetMapping` / `@PostMapping` | Maps HTTP methods to functions |
| `@Service` | Business logic layer |
| `@Repository` | Database layer |
| `@Autowired` | Inject dependencies automatically |


Without Spring (normal Java):

YOU control object creation — you write new whenever you need an object.

javaTenantRepository repo = new MySQLTenantRepository(); // YOU create it
TenantService service = new TenantService(repo);     // YOU wire it

With Spring (IoC):

Spring controls object creation. You just declare what you need — Spring creates and gives it to you.

java@Service
public class TenantService {
    @Autowired
    private TenantRepository repository; // Spring creates and injects this
}
Control has been INVERTED — from you to the framework. That's literally what "Inversion of Control" means.

DI Is HOW IoC Is Implemented

IoC = the principle (give up control to framework)

DI = the technique Spring uses to implement IoC (inject dependencies into your class)

IoC = the concept/goal
DI = the mechanism that achieves it


3 Types of Dependency Injection — Senior Level
1. Field Injection (what you've likely used)
java@Autowired
private TenantRepository repository;
Easy but hard to test, hides dependencies.
2. Constructor Injection (Senior engineers prefer this)
javaprivate final TenantRepository repository;

public TenantService(TenantRepository repository) {
    this.repository = repository;
}
Dependencies are explicit, easy to test, can be final.
3. Setter Injection
javaprivate TenantRepository repository;

@Autowired
public void setRepository(TenantRepository repository) {
    this.repository = repository;
}
Used when dependency is optional.

Senior Interview Question

"Why is constructor injection preferred over field injection?"

Answer:

Constructor injection makes dependencies immutable (final) and mandatory — object can't exist without them. It also makes unit testing easier because you can pass mock dependencies directly through the constructor without needing Spring context. Field injection requires reflection to inject, making it harder to test and hides the true dependencies of a class.


Spring Bean Lifecycle — Where Objects Come From
1. Spring scans @Component, @Service, @Repository, @Controller
2. Creates objects (Beans) from these classes
3. Stores them in ApplicationContext (container)
4. Injects them wherever @Autowired is used
5. Manages their entire lifecycle (creation to destruction)

Bean = any object managed by Spring container.

ApplicationContext = the container that holds all beans.


Connect To Your Project
java@Repository
public class MySQLTenantRepository implements TenantRepository { }

@Service
public class TenantService {
    private final TenantRepository repository;
    
    @Autowired
    public TenantService(TenantRepository repository) {
        this.repository = repository;
    }
}

Spring scans MySQLTenantRepository, creates it as a Bean, stores it in ApplicationContext. When it creates TenantService, it sees the constructor needs TenantRepository — looks in ApplicationContext, finds MySQLTenantRepository, injects it automatically.

You don't write new anywhere. That's IoC + DI working together.


