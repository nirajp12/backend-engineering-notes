# Spring Boot Dependency Injection — Field vs Constructor Injection

> Revision notes. Explained the way a senior engineer would explain it to a junior on day one.

---

## 1. The Core Idea

A **dependency** is something a class needs from outside itself to do its job. Instead of a class creating its own dependencies (`new Engine()`), Spring creates objects for you (called **beans**) and hands them over. This is **Dependency Injection (DI)**.

There are two ways to receive these dependencies that matter most in practice:

- **Field Injection** — Spring reaches directly into a private field and sets it, using reflection, after the object already exists.
- **Constructor Injection** — Spring passes the dependency in through the constructor, *before* the object can even be created.

That difference — **when and how strictly the dependency is required** — is the entire reason one is considered better than the other. It is not about *what* you declare, it's about what the **compiler enforces** vs what is just a **polite runtime request**.

---

## 2. Class vs Object (the part that confuses people first)

- A **class** is a blueprint (`class Engine { ... }`). It must exist in your code to compile — true regardless of injection style.
- An **object** is the actual built thing (`new Engine()`). This is what might be missing.

```java
class Car {
    private Engine engine; // just an empty labeled box, not an object yet
}
```

Declaring a field does **not** create an object — it reserves space. When `new Car()` runs, that box defaults to `null` until something fills it.

---

## 3. Field Injection

```java
@Component
class Engine {
    void start() { System.out.println("Vroom!"); }
}

@Component
class Car {
    @Autowired
    private Engine engine;     // Spring fills this box AFTER Car is built

    void drive() {
        engine.start();
    }
}
```

`@Autowired` on a field is a **sticky note**, not a Java rule. It only works if **Spring** is the one building the object. Java itself has no idea what `@Autowired` means.

```java
Car myCar = new Car();   // ✅ compiles — engine box is just left null
myCar.drive();           // 💥 NullPointerException at runtime
```

Even inside a running Spring app, if *you* call `new Car()` yourself instead of asking Spring for its managed bean, Spring never touches that object — the box stays empty.

---

## 4. Constructor Injection

```java
@Component
class Car {
    private final Engine engine;     // can be final!

    Car(Engine engine) {
        this.engine = engine;
    }
}
```

```java
Car myCar = new Car();              // ❌ COMPILE ERROR — no such constructor
Car myCar = new Car(new Engine());  // ✅ must supply a real object
```

Java itself refuses to let an incomplete `Car` exist. This is enforced by the **compiler**, not by Spring's goodwill.

---

## 5. Field vs Constructor — Comparison Table

| | Field Injection | Constructor Injection |
|---|---|---|
| How dependency arrives | Reflection sets the field after object creation | Passed in through the constructor before object exists |
| Can object exist in a broken/incomplete state? | Yes — fields default to `null` | No — compiler refuses to build it without dependencies |
| Can field be `final`? | No | Yes |
| Visibility of dependencies | Scattered across the class, must scan all fields | All visible in one place: the constructor signature |
| Easy to unit test without Spring? | Hard — needs reflection tricks (`@InjectMocks`, `ReflectionTestUtils`) | Easy — just call `new Car(fakeEngine)` |
| Circular dependency behavior | Silently works (masks bad design) | Fails fast at startup with a clear error |
| Industry recommendation today | Discouraged | Preferred standard |

---

## 6. Why "discouraged" actually matters (not just style)

1. **Immutability** — constructor injection allows `final` fields; field injection does not.
2. **Fail fast** — missing/ambiguous/circular dependencies are caught at **startup** with constructor injection, vs a crash buried deep in production with field injection.
3. **Testability** — constructor injection lets you write plain Java unit tests with no Spring context needed.
4. **Design smell visibility** — a constructor with 8 parameters is an obvious "this class does too much" signal. A class with 8 scattered `@Autowired` fields hides that smell.
5. **No accidental nulls** — field injection allows an object to silently exist with missing dependencies if it wasn't built by Spring.

---

## 7. Q&A Revision Set (with reasoning)

### Q1. Plain Java, no Spring context
```java
class Car {
    @Autowired
    private Engine engine;
    void drive() { engine.start(); }
}
Car myCar = new Car();
myCar.drive();
```
**Answer: Compiles and runs, then throws `NullPointerException`.**
`@Autowired` is meaningless without Spring's container running. In plain Java, `engine` stays `null`, and calling `.start()` on `null` crashes.

---

### Q2. Spring app running, but object built manually with `new`
```java
@Component class Car { @Autowired private Engine engine; ... }
...
Car myCar = new Car();   // bypassing Spring
myCar.drive();
```
**Answer: Still `NullPointerException`.**
Spring does not intercept every `new` call across your whole codebase. It only wires objects **it itself constructs** and stores in the Application Context. A manually created object is a completely separate instance Spring never touches.

---

### Q3. Circular dependency with constructor injection
```java
@Component class Engine { Engine(Car car) { ... } }
@Component class Car { Car(Engine engine) { ... } }
```
**Answer: Application fails to start — circular dependency error (`BeanCurrentlyInCreationException`).**
Constructor injection requires the dependency to exist *before* the object is built. `Engine` needs `Car` needs `Engine` — there's no valid build order, so Spring fails fast at startup instead of hanging or crashing later.

**Side note:** the same circular setup using *field* injection actually starts successfully, because field injection allows both half-built objects to exist first and get wired into each other afterward. This is not really a win — it just hides a bad design instead of forcing a fix.

---

### Q4. Ambiguous bean — multiple implementations of an interface
```java
interface PaymentMethod { void pay(); }
@Component class CreditCardPayment implements PaymentMethod { ... }
@Component class UpiPayment implements PaymentMethod { ... }

@Component class CheckoutService {
    @Autowired
    private PaymentMethod paymentMethod;
}
```
**Answer: Application fails to start — `NoUniqueBeanDefinitionException`.**
Spring searches by type, finds two valid candidates, and has no way to choose between them. It does not guess or pick randomly — it fails loudly instead.

**Fix options:**
- `@Qualifier("beanName")` to pick a specific bean by name.
- `@Primary` on one implementation to mark it as the default.

---

### Q5. Constructor injection + `@Qualifier` on a parameter
```java
@Component("emailNotifier") class EmailNotifier implements Notifier { ... }
@Component("smsNotifier") class SmsNotifier implements Notifier { ... }

@Component
class AlertService {
    AlertService(@Qualifier("smsNotifier") Notifier notifier) { ... }
}
```
**Answer: Works fine — prints "SMS: Server down!"**
`@Qualifier` works on constructor parameters just as well as on fields. It narrows the search from "any bean of this type" to "the bean registered under this exact name," removing the ambiguity from Q4.

**Note:** bean names default to the class name with a lowercase first letter, so explicit names here (`"smsNotifier"`) happened to match what Spring would have auto-generated anyway. It would only matter more with a custom name like `@Component("backupSender")`.

---

### Q6. Injecting `List<Interface>` — collecting ALL matching beans
```java
@Component class EmailNotifier implements Notifier { ... }
@Component class SmsNotifier implements Notifier { ... }
@Component class PushNotifier implements Notifier { ... }

@Component
class BroadcastService {
    BroadcastService(List<Notifier> notifiers) { ... }
}
```
**Answer: Works fine — all three beans get collected into the list and all three fire.**
Asking for a single `Notifier` triggers the ambiguity error from Q4. Asking for `List<Notifier>` changes the request from "give me exactly one" to "give me everything matching this type" — Spring happily collects all of them. This pattern is commonly used for plugin-style architectures (e.g., running every `Validator` implementation on a form without hardcoding each one).

---

### Q7. Optional dependency with `required = false`
```java
class FileLogger implements Logger { ... }   // NOTE: no @Component — not a bean at all

@Component
class ReportService {
    @Autowired(required = false)
    private Logger logger;

    void generateReport() {
        if (logger != null) logger.log("Report generated");
        else System.out.println("No logger available, skipping log step");
    }
}
```
**Answer: Application starts fine, `logger` stays `null`, and the `else` branch runs.**
Plain `@Autowired` is mandatory by default — a missing bean would fail startup with `NoSuchBeanDefinitionException`. `required = false` tells Spring: "try to wire this, but it's fine if you can't — just leave it `null`." The class is then responsible for null-checking before use, same risk profile as field injection in general.

---

## 8. One-Paragraph Summary

Spring builds and manages objects (beans) so you don't have to wire dependencies by hand. **Field injection** sets dependencies directly on private fields via reflection *after* an object is built, which means broken/incomplete objects can silently exist until something crashes at runtime. **Constructor injection** demands dependencies *before* an object can be built at all, which the Java compiler itself enforces — making it impossible to end up with a half-wired object, easier to test without Spring, and faster to catch design problems (circular dependencies, missing beans, ambiguous beans) at startup instead of in production. For real Spring Boot projects, constructor injection is the standard; field injection is mostly seen in legacy code or quick test classes.
