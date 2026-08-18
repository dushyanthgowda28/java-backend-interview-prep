# `@Autowired` in Spring — Detailed Explanation

`@Autowired` is one of the most important annotations in Spring for **Dependency Injection (DI)**.

It tells Spring:

> **"Find a suitable Spring-managed bean and inject it into this class."**

For a 7–8 year experienced Java/Spring Boot developer, you should understand not only how `@Autowired` works, but also **how Spring resolves dependencies, what happens internally, constructor vs field/setter injection, multiple-bean scenarios, `@Primary`, `@Qualifier`, optional dependencies, circular dependencies, and how proxies interact with injection.**

---

## 1. What is `@Autowired`?

`@Autowired` is a Spring annotation used for **automatic dependency injection**.

Example:

```java @Service public class OrderService {

    @Autowired
    private PaymentService paymentService;

    public void createOrder() {
    paymentService.processPayment();
    }
}
```

Here:

```java @Autowired private PaymentService paymentService; ```

means:

> Spring should find a bean of type `PaymentService` and assign it to `paymentService`.

You don't manually do:

```java PaymentService paymentService = new PaymentService(); ```

Spring manages the object and its dependency.

---

# 2. Why do we need `@Autowired`?

Without dependency injection:

```java public class OrderService {

    private PaymentService paymentService;

    public OrderService() {
    this.paymentService = new PaymentService();
    }
}
```

`OrderService` is tightly coupled to:

```java new PaymentService() ```

With Spring:

```java @Service public class OrderService {

    private final PaymentService paymentService;

    @Autowired
    public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
    }
}
```

Now Spring creates and supplies `PaymentService`.

The dependency is **injected from outside**.

This gives you:

- Loose coupling
- Better testability
- Easier replacement of implementations
- Better separation of responsibilities
- Easier configuration
- Support for proxies/**AOP**
- Centralized object lifecycle management

---

# 3. Where can `@Autowired` be used?

`@Autowired` can commonly be used on:

## Constructor

## Setter method
3. Field
## Method
## Configuration method/parameters

For example:

### Constructor

```java
@Autowired
public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

### Setter

```java
@Autowired
public void setPaymentService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

### Field

```java @Autowired private PaymentService paymentService; ```

---

# 4. How does `@Autowired` actually work?

This is an important interview topic.

Suppose:

```java @Service public class OrderService {

    @Autowired
    private PaymentService paymentService;
}
```

Spring doesn't simply see `@Autowired` and execute:

```java orderService.paymentService = paymentService; ```

There is a Spring mechanism behind it.

The major component involved is:

```text AutowiredAnnotationBeanPostProcessor ```

This is a Spring `BeanPostProcessor` responsible for processing `@Autowired`.

Simplified flow:

```text
Application starts
       ↓
Spring creates ApplicationContext
       ↓
Bean definitions are discovered
       ↓
Spring creates beans
       ↓
@Autowired is detected
       ↓
AutowiredAnnotationBeanPostProcessor
       ↓
Dependency resolution
       ↓
Required bean is found
       ↓
Dependency is injected
       ↓
Bean initialization continues
```

---

# 5. `AutowiredAnnotationBeanPostProcessor`

This class is extremely important for understanding `@Autowired`.

Spring registers:

```text AutowiredAnnotationBeanPostProcessor ```

It scans bean classes for annotations such as:

```java @Autowired ```

and processes them during bean creation.

Conceptually:

```text Bean | |-- @Autowired field |-- @Autowired constructor |-- @Autowired method | ↓ AutowiredAnnotationBeanPostProcessor | ↓ Dependency resolution | ↓ Injection ```

For interviews, remember:

> **`@Autowired` is processed by `AutowiredAnnotationBeanPostProcessor`.**

---

# 6. How does Spring find the dependency?

Suppose:

```java @Service public class OrderService {

    @Autowired
    private PaymentService paymentService;
}
```

Spring needs to find:

```text PaymentService ```

It first considers the dependency's type.

Conceptually:

```text
PaymentService.class
       ↓
Find beans matching PaymentService
       ↓
Candidate beans
       ↓
Resolve candidate
       ↓
Inject selected bean
```

---

# 7. Dependency resolution is primarily type-based

This is a very important point.

Consider:

```java @Service public class PaymentService { } ```

and:

```java @Service public class OrderService {

    @Autowired
    private PaymentService paymentService;
}
```

Spring searches for a bean compatible with:

```java PaymentService ```

So:

```text
@Autowired
     ↓
PaymentService type
     ↓
Find matching bean
```

This is why `@Autowired` is commonly described as **by-type autowiring**.

---

# 8. What happens if only one bean exists?

Suppose:

```java @Service public class PaymentService { } ```

and:

```java @Service public class OrderService {

    @Autowired
    private PaymentService paymentService;
}
```

There is exactly one candidate.

Therefore:

```text
PaymentService
      ↓
1 matching bean
      ↓
Inject it
```

No problem.

---

# 9. What if multiple beans exist?

This is where interviews become interesting.

Suppose:

```java
public interface PaymentService {
    void pay();
}
```

Implementation 1:

```java @Service public class CardPaymentService implements PaymentService {

    @Override
    public void pay() {
    System.out.println(*Card payment*);
    }
}
```

Implementation 2:

```java @Service public class UpiPaymentService implements PaymentService {

    @Override
    public void pay() {
    System.out.println(***UPI** payment*);
    }
}
```

Now:

```java @Service public class OrderService {

    @Autowired
    private PaymentService paymentService;
}
```

Spring finds:

```text
PaymentService
    ├── CardPaymentService
    └── UpiPaymentService
```

There are two candidates.

Spring cannot arbitrarily choose one.

You may get:

```text NoUniqueBeanDefinitionException ```

---

# 10. Solving multiple beans with `@Qualifier`

Use:

```java @Service public class OrderService {

    @Autowired
    @Qualifier(*upiPaymentService*)
    private PaymentService paymentService;
}
```

Now Spring knows:

```text
PaymentService
      ↓
@Qualifier(*upiPaymentService*)
      ↓
UpiPaymentService
```

The default bean name for:

```java @Service public class UpiPaymentService ```

is generally:

```text upiPaymentService ```

---

# 11. `@Primary`

Another solution is:

```java @Service @Primary public class CardPaymentService implements PaymentService { } ```

Then:

```java @Service public class OrderService {

    @Autowired
    private PaymentService paymentService;
}
```

Spring sees:

```text
CardPaymentService
    @Primary
    
UpiPaymentService
```

and chooses:

```text CardPaymentService ```

---

# 12. `@Primary` vs `@Qualifier`

This is a common interview question.

### `@Primary`

Defines the **default preferred candidate**.

```java @Primary @Service public class CardPaymentService implements PaymentService { } ```

### `@Qualifier`

Explicitly identifies the required candidate.

```java @Autowired @Qualifier(*upiPaymentService*) private PaymentService paymentService; ```

Think:

```text
@Primary
    ↓
Default choice

@Qualifier
    ↓
Explicit choice
```

If both are present:

```java @Autowired @Qualifier(*upiPaymentService*) private PaymentService paymentService; ```

the qualifier can identify the specific bean.

---

# 13. Constructor Injection with `@Autowired`

Example:

```java @Service public class OrderService {

    private final PaymentService paymentService;

    @Autowired
    public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
    }
}
```

Spring essentially does:

```text
Find OrderService
      ↓
Inspect constructor
      ↓
Constructor requires PaymentService
      ↓
Find PaymentService bean
      ↓
Pass it to constructor
      ↓
Create OrderService
```

Conceptually:

```java
PaymentService paymentService =
        applicationContext.getBean(PaymentService.class);

OrderService orderService =
        new OrderService(paymentService);
```

This is only a conceptual representation; Spring performs much more internally.

---

# 14. Is `@Autowired` required on a constructor?

This is an important modern Spring behavior.

If a class has **only one constructor**, `@Autowired` is generally not required.

For example:

```java @Service public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
    }
}
```

This works.

You don't need:

```java @Autowired ```

on the constructor.

Spring recognizes the single constructor as the injection constructor.

---

# 15. Why is constructor injection preferred?

For production Spring applications, constructor injection is generally preferred.

Instead of:

```java @Autowired private PaymentService paymentService; ```

prefer:

```java private final PaymentService paymentService;

public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

Advantages:

### 1. Dependencies are explicit

You can immediately see:

```java
OrderService(
    PaymentService paymentService,
    InventoryService inventoryService
)
```

and understand what the class needs.

### 2. Supports immutability

```java private final PaymentService paymentService; ```

### 3. Easier unit testing

```java PaymentService paymentService = mock(PaymentService.class);

OrderService service =
        new OrderService(paymentService);
```

### 4. Prevents partially initialized objects

A constructor cannot finish successfully without the required dependency.

### 5. Helps identify circular dependencies

Constructor circular dependencies are detected during bean creation.

---

# 16. Field Injection

Example:

```java @Service public class OrderService {

    @Autowired
    private PaymentService paymentService;
}
```

The process is conceptually:

```text
Create OrderService object
        ↓
Perform dependency injection
        ↓
Set paymentService
```

Conceptually:

```java OrderService orderService = new OrderService();

orderService.paymentService = paymentService; ```

Actual Spring internals are more sophisticated, but this gives the correct conceptual model.

---

# 17. Important: Field injection happens after object instantiation

This is a very common interview question.

Consider:

```java @Service public class OrderService {

    @Autowired
    private PaymentService paymentService;

    public OrderService() {
    System.out.println(*Constructor*);
    }
}
```

The simplified lifecycle is:

```text
## Instantiate OrderService
        ↓
## Constructor executes
        ↓
3. @Autowired dependency injection
        ↓
## Initialization callbacks
        ↓
## Bean becomes available
```

Therefore:

> **For field injection, the object is instantiated first and dependencies are injected afterward.**

This is one reason constructor injection is generally preferred.

---

# 18. Setter Injection

Example:

```java @Service public class OrderService {

    private PaymentService paymentService;

    @Autowired
    public void setPaymentService(PaymentService paymentService) {
    this.paymentService = paymentService;
    }
}
```

Conceptually:

```text
Create OrderService
       ↓
Call setter
       ↓
setPaymentService(paymentService)
       ↓
Dependency injected
```

Setter injection can be useful when a dependency is optional or can legitimately be changed after construction.

---

# 19. Method Injection with `@Autowired`

`@Autowired` can also be placed on a method.

```java @Service public class OrderService {

    private PaymentService paymentService;
    private InventoryService inventoryService;

    @Autowired
    public void initialize(
    PaymentService paymentService,
    InventoryService inventoryService) {

    this.paymentService = paymentService;
    this.inventoryService = inventoryService;
    }
}
```

Spring resolves both parameters and calls the method.

Conceptually:

```text
@Autowired method
       ↓
Resolve PaymentService
       ↓
Resolve InventoryService
       ↓
Invoke initialize(...)
```

---

# 20. Can `@Autowired` be used with private fields?

Yes.

```java @Autowired private PaymentService paymentService; ```

Spring can inject it even though the field is private.

This is one reason field injection can hide dependencies.

---

# 21. Can `@Autowired` be used on static fields?

Generally, you should **not use `@Autowired` for static fields**.

For example:

```java @Autowired private static PaymentService paymentService; ```

This is not a supported/appropriate dependency-injection pattern.

Spring manages **instances**, while static fields belong to the class rather than an individual bean instance.

Prefer:

```java @Service public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
    }
}
```

---

# 22. Optional dependency

Sometimes a dependency isn't mandatory.

You can use:

```java @Autowired(required = false) private PaymentService paymentService; ```

If no matching bean exists, Spring won't fail because of this injection point.

However, this approach is generally less explicit.

Better modern approaches include:

```java Optional<PaymentService> ```

or:

```java ObjectProvider<PaymentService> ```

or constructor design where the dependency is explicitly optional.

---

# 23. `Optional<T>` with dependency injection

Example:

```java @Service public class OrderService {

    private final Optional<PaymentService> paymentService;

    public OrderService(Optional<PaymentService> paymentService) {
    this.paymentService = paymentService;
    }
}
```

If the bean exists:

```text Optional[PaymentService] ```

If it doesn't:

```text Optional.empty() ```

This expresses optionality more clearly than field injection.

---

# 24. `ObjectProvider`

For lazy or optional dependency access:

```java @Service public class OrderService {

    private final ObjectProvider<PaymentService> paymentServiceProvider;

    public OrderService(
    ObjectProvider<PaymentService> paymentServiceProvider) {

    this.paymentServiceProvider = paymentServiceProvider;
    }

    public void process() {

    PaymentService service =
    paymentServiceProvider.getIfAvailable();

    if (service != null) {
    service.pay();
    }
    }
}
```

`ObjectProvider` is useful when you need:

- Optional dependency
- Lazy lookup
- Multiple beans
- On-demand retrieval
- Prototype bean access

---

# 25. `@Autowired` and `@Lazy`

Consider:

```java @Service public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
    @Lazy PaymentService paymentService) {

    this.paymentService = paymentService;
    }
}
```

`@Lazy` can cause Spring to inject a lazy-resolution proxy rather than immediately creating the actual target dependency.

Conceptually:

```text
OrderService
     ↓
PaymentService proxy
     ↓
Actual PaymentService
```

When the dependency is actually used, Spring can resolve/create the target.

This can also be relevant when dealing with certain circular dependencies.

---

# 26. `@Autowired` and circular dependency

Consider:

```java @Service public class A {

    @Autowired
    private B b;
}
```

and:

```java @Service public class B {

    @Autowired
    private A a;
}
```

This creates:

```text A → B ↑   ↓ └───┘ ```

With setter/field-style injection, Spring has mechanisms around early singleton exposure that can sometimes resolve singleton circular dependencies.

But constructor injection:

```java @Service public class A {

    private final B b;

    public A(B b) {
    this.b = b;
    }
}
```

and:

```java @Service public class B {

    private final A a;

    public B(A a) {
    this.a = a;
    }
}
```

creates an unresolvable construction cycle.

Therefore:

> Constructor injection generally makes circular dependencies fail fast rather than allowing them to be hidden.

The preferred solution is usually **redesigning the dependency relationship**, rather than using `@Lazy` merely to hide the cycle.

---

# 27. `@Autowired` with interfaces

Very common in real projects.

```java
public interface NotificationService {
    void send();
}
```

Implementation:

```java
@Service
public class EmailNotificationService
        implements NotificationService {

    @Override
    public void send() {
    System.out.println(*Email*);
    }
}
```

Consumer:

```java @Service public class OrderService {

    private final NotificationService notificationService;

    public OrderService(NotificationService notificationService) {
    this.notificationService = notificationService;
    }
}
```

Spring sees:

```text
NotificationService
       ↑
EmailNotificationService
```

and injects:

```text EmailNotificationService ```

---

# 28. Multiple implementations with `@Qualifier`

```java
@Service(*emailService*)
public class EmailNotificationService
        implements NotificationService {
}
```

```java
@Service(*smsService*)
public class SmsNotificationService
        implements NotificationService {
}
```

Then:

```java @Service public class OrderService {

    private final NotificationService notificationService;

    public OrderService(
    @Qualifier(*emailService*)
    NotificationService notificationService) {

    this.notificationService = notificationService;
    }
}
```

This is one of the cleanest ways to explicitly select an implementation.

---

# 29. `@Autowired` with collections

Spring can inject **all matching beans**.

Suppose:

```java
@Service
public class EmailNotificationService
        implements NotificationService {
}
```

```java
@Service
public class SmsNotificationService
        implements NotificationService {
}
```

Then:

```java @Service public class NotificationManager {

    private final List<NotificationService> services;

    public NotificationManager(
    List<NotificationService> services) {

    this.services = services;
    }
}
```

Spring injects:

```text List ├── EmailNotificationService └── SmsNotificationService ```

This is extremely useful for strategy-pattern implementations.

---

# 30. Injecting `Map<String, BeanType>`

You can also inject:

```java Map<String, NotificationService> ```

Example:

```java @Service public class NotificationManager {

    private final Map<String, NotificationService> services;

    public NotificationManager(
    Map<String, NotificationService> services) {

    this.services = services;
    }
}
```

Conceptually:

```text
services
    |
    ├── *emailNotificationService*
    │       → EmailNotificationService
    |
    └── *smsNotificationService*
    → SmsNotificationService
```

This is useful when selecting implementations dynamically.

---

# 31. Ordering multiple beans

When injecting a collection, you can control ordering using:

```java @Order ```

Example:

```java
@Service
@Order(1)
public class EmailNotificationService
        implements NotificationService {
}
```

```java
@Service
@Order(2)
public class SmsNotificationService
        implements NotificationService {
}
```

Then:

```java List<NotificationService> ```

can be ordered accordingly.

---

# 32. `@Autowired` vs `@Resource`

Both can perform dependency injection, but their resolution semantics differ.

### `@Autowired`

Primarily:

```text Type → Qualifier/Primary if needed ```

### `@Resource`

Traditionally favors:

```text Name → Type ```

Example:

```java @Resource(name = *emailNotificationService*) private NotificationService notificationService; ```

For Spring-specific applications, `@Autowired` is commonly used, though constructor injection often removes the need for either annotation.

---

# 33. `@Autowired` vs `@Inject`

`@Inject` comes from:

```text ### Jakarta Dependency Injection ```

while:

```java @Autowired ```

is Spring-specific.

Conceptually:

```text
@Autowired
    ↓
Spring-specific

@Inject
    ↓
Jakarta standard
```

Spring supports both.

---

# 34. What happens when the dependency doesn't exist?

Suppose:

```java @Service public class OrderService {

    @Autowired
    private PaymentService paymentService;
}
```

but there is no:

```java PaymentService ```

bean.

For a required dependency, application startup can fail with an error such as:

```text NoSuchBeanDefinitionException ```

or a related dependency-resolution exception.

This happens because Spring cannot satisfy the required dependency.

---

# 35. What happens when multiple candidates exist?

Suppose:

```text
PaymentService
    ├── CardPaymentService
    └── UpiPaymentService
```

and:

```java @Autowired private PaymentService paymentService; ```

Spring can't determine which one to inject.

You can resolve it using:

```java @Primary ```

or:

```java @Qualifier ```

Otherwise, you can encounter:

```text NoUniqueBeanDefinitionException ```

---

# 36. Bean name vs variable name

Consider:

```java @Autowired private PaymentService paymentService; ```

Don't assume Spring simply searches based on:

```text paymentService ```

The fundamental matching mechanism is type-based.

However, bean names/qualifiers can participate in resolving ambiguity.

For example:

```java
@Service(*upiPayment*)
public class UpiPaymentService
        implements PaymentService {
}
```

Then:

```java @Autowired @Qualifier(*upiPayment*) private PaymentService paymentService; ```

explicitly selects that bean.

---

# 37. `@Autowired` is not responsible for creating every bean

This distinction is very important.

Consider:

```java @Autowired private PaymentService paymentService; ```

`@Autowired` itself does **not mean *create PaymentService.***

The bean must be known to the Spring container through mechanisms such as:

```java @Service @Component @Repository @Controller @Bean ```

or other bean-registration mechanisms.

So:

```text
@Bean / @Component / @Service
        ↓
Bean registration
        ↓
Spring container knows the bean
        ↓
@Autowired
        ↓
Dependency injection
```

---

# 38. `@Component` + `@Autowired`

Example:

```java @Component public class PaymentService { } ```

```java @Component public class OrderService {

    @Autowired
    private PaymentService paymentService;
}
```

Flow:

```text
@Component
    ↓
PaymentService registered
    ↓
@Component
    ↓
OrderService registered
    ↓
@Autowired
    ↓
PaymentService injected
```

---

# 39. `@Bean` + `@Autowired`

You can also define:

```java @Configuration public class AppConfig {

    @Bean
    public PaymentService paymentService() {
    return new PaymentService();
    }
}
```

Then:

```java @Service public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
    }
}
```

Spring finds the `PaymentService` bean created by the `@Bean` method and injects it.

---

# 40. `@Autowired` and proxies

This is an advanced but important topic.

Suppose:

```java @Service @Transactional public class PaymentService {

    public void pay() {
    }
}
```

Spring may create a proxy around the target object.

Conceptually:

```text
OrderService
    |
    | injected dependency
    ↓
PaymentService Proxy
    |
    ↓
Actual PaymentService
```

Therefore, what gets injected may be a **proxy**, depending on Spring features being applied.

For example:

```text @Transactional @Async @Cacheable ### Spring Security **AOP** ```

can result in proxy-based behavior.

This is one reason you shouldn't think of dependency injection as simply:

```java new PaymentService() ```

---

# 41. Does Spring create the object before injecting the dependency?

It depends on the injection style.

### Constructor injection

Dependency is resolved before/during construction:

```text
Resolve dependencies
       ↓
Call constructor
       ↓
Object instantiated
```

Conceptually:

```java new OrderService(paymentService); ```

### Field injection

Object is instantiated first:

```text
Instantiate OrderService
       ↓
Inject PaymentService
```

### Setter injection

Object is instantiated first:

```text
Instantiate OrderService
       ↓
Call setter
       ↓
Inject PaymentService
```

This distinction is important when discussing bean lifecycle.

---

# 42. `@Autowired` and bean lifecycle

A simplified lifecycle:

```text
BeanDefinition
      ↓
Instantiate bean
      ↓
Populate properties / dependencies
      ↓
Aware callbacks
      ↓
BeanPostProcessor before initialization
      ↓
@PostConstruct
      ↓
InitializingBean
      ↓
Custom init method
      ↓
BeanPostProcessor after initialization
      ↓
Bean ready
```

For field/setter injection, dependency injection occurs during the **property population** stage.

`AutowiredAnnotationBeanPostProcessor` participates in this processing.

---

# 43. `@Autowired` and `@PostConstruct`

Consider:

```java @Service public class OrderService {

    @Autowired
    private PaymentService paymentService;

    @PostConstruct
    public void init() {
    System.out.println(paymentService);
    }
}
```

By the time `@PostConstruct` executes, the required field dependency has been injected.

So:

```text Constructor ↓ @Autowired injection ↓ @PostConstruct ```

Conceptually.

---

# 44. Constructor injection and `@PostConstruct`

```java @Service public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
    }

    @PostConstruct
    public void init() {
    paymentService.pay();
    }
}
```

Here the dependency is already available when the constructor finishes.

This is another advantage of constructor injection.

---

# 45. `@Autowired` is not the same as `getBean()`

These two approaches are different.

### Dependency Injection

```java @Service public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
    }
}
```

The dependency is supplied by Spring.

### Programmatic lookup

```java
PaymentService paymentService =
        applicationContext.getBean(PaymentService.class);
```

Here your application explicitly asks the container for a bean.

So:

```text
@Autowired
    ↓
### Dependency Injection

getBean()
    ↓
### Dependency Lookup
```

Generally, dependency injection is preferred over manually calling `getBean()`.

---

# 46. `@Autowired` vs manual object creation

### Manual

```java PaymentService service = new PaymentService(); ```

Spring doesn't manage this object.

Therefore you may miss:

- Dependency injection
- **AOP**
- `@Transactional`
- `@Async`
- Lifecycle callbacks
- Other Spring-managed behavior

### Spring-managed

```java @Autowired private PaymentService service; ```

The injected bean is managed by Spring.

---

# 47. Can `@Autowired` inject prototype beans?

Yes.

Suppose:

```java @Component @Scope(*prototype*) public class PaymentService { } ```

and:

```java @Service public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
    }
}
```

The important point is:

> Injecting a prototype bean into a singleton does **not automatically create a new prototype instance every time the singleton uses the field**.

The dependency resolution happens according to the injection mechanism.

If you need a fresh prototype instance on demand, mechanisms such as:

```java ObjectProvider<PaymentService> ```

can be used.

---

# 48. `ObjectProvider` with prototype

```java @Service public class OrderService {

    private final ObjectProvider<PaymentService> provider;

    public OrderService(ObjectProvider<PaymentService> provider) {
    this.provider = provider;
    }

    public void process() {

    PaymentService paymentService =
    provider.getObject();

    paymentService.pay();
    }
}
```

Each lookup can obtain a new prototype instance.

---

# 49. Why field injection is generally discouraged

This:

```java @Autowired private PaymentService paymentService; ```

works, but has several disadvantages.

### Hidden dependency

The constructor doesn't reveal that `PaymentService` is required.

### Harder testing

You cannot simply:

```java new OrderService(paymentService); ```

if there is no constructor.

### Mutability

The field isn't naturally immutable:

```java private PaymentService paymentService; ```

### Reflection-based injection

Spring has to inject the field after object construction.

### Can hide excessive dependencies

A class with 10 `@Autowired` fields can hide the fact that it has too many responsibilities.

---

# 50. Recommended approach

Instead of:

```java @Service public class OrderService {

    @Autowired
    private PaymentService paymentService;

    @Autowired
    private InventoryService inventoryService;
}
```

prefer:

```java @Service public class OrderService {

    private final PaymentService paymentService;
    private final InventoryService inventoryService;

    public OrderService(
    PaymentService paymentService,
    InventoryService inventoryService) {

    this.paymentService = paymentService;
    this.inventoryService = inventoryService;
    }
}
```

Even better, with a single constructor, no `@Autowired` is necessary.

---

# 51. Lombok and constructor injection

In Spring Boot projects, you may see:

```java @Service @RequiredArgsConstructor public class OrderService {

    private final PaymentService paymentService;
    private final InventoryService inventoryService;
}
```

Lombok generates the constructor.

Conceptually it becomes:

```java
public OrderService(
    PaymentService paymentService,
    InventoryService inventoryService) {

    this.paymentService = paymentService;
    this.inventoryService = inventoryService;
}
```

Spring then performs constructor injection.

This is extremely common in modern Spring Boot projects.

---

# 52. Important interview question: Is `@Autowired` mandatory?

### Field injection

Yes, you need an injection mechanism such as:

```java @Autowired ```

for Spring to know that the field should be autowired.

### Single constructor

No.

```java
public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

is enough.

### Multiple constructors

If there are multiple constructors, you generally need to indicate which constructor Spring should use, commonly with:

```java @Autowired ```

---

# 53. Important interview question: What happens internally?

A good interview answer:

> `@Autowired` is processed by Spring's `AutowiredAnnotationBeanPostProcessor`. During bean creation, Spring identifies the injection point, resolves the dependency from the BeanFactory/ApplicationContext, applies dependency-resolution rules such as type matching, qualifiers and primary candidates, and injects the resolved bean into the constructor, field, setter, or method.

That is a strong 7–8 year experience answer.

---

# 54. Complete mental model

Think about `@Autowired` like this:

```text
    Spring Container
    |
    |
    ┌───────────┴───────────┐
    ↓                       ↓
    OrderService            PaymentService
    |                       |
    |                       |
    └────── dependency ─────┘
    ↑
    |
    @Autowired
```

Spring manages:

```text
PaymentService bean
       ↓
OrderService bean
```

and establishes:

```text OrderService → PaymentService ```

---

# 55. Complete example

```java public interface PaymentService {

    void pay();
}
```

```java
@Service(*upiPayment*)
public class UpiPaymentService
        implements PaymentService {

    @Override
    public void pay() {
    System.out.println(***UPI** payment*);
    }
}
```

```java
@Service(*cardPayment*)
public class CardPaymentService
        implements PaymentService {

    @Override
    public void pay() {
    System.out.println(*Card payment*);
    }
}
```

Consumer:

```java @Service public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
    @Qualifier(*upiPayment*)
    PaymentService paymentService) {

    this.paymentService = paymentService;
    }

    public void createOrder() {
    paymentService.pay();
    }
}
```

Flow:

```text
ApplicationContext starts
        ↓
Find @Service classes
        ↓
Register bean definitions
        ↓
Create PaymentService implementations
        ↓
Create OrderService
        ↓
Constructor requires PaymentService
        ↓
Two candidates found
        ↓
@Qualifier(*upiPayment*)
        ↓
Select UpiPaymentService
        ↓
Inject into OrderService constructor
        ↓
OrderService becomes ready
```

---

# 56. `@Autowired` interview questions you should know

For a 7–8 year Spring Boot developer, be prepared for:

### Basic

## What is `@Autowired`?

## How does `@Autowired` work? ## Where can `@Autowired` be used? ## Is `@Autowired` mandatory? ## What is dependency injection? ## What is the difference between DI and dependency lookup?

### Resolution

## How does Spring resolve an `@Autowired` dependency?

## What happens if there is only one candidate?
## What happens if there are multiple candidates?
## What is `@Primary`?
## What is `@Qualifier`?
12. `@Primary` vs `@Qualifier`?
## What happens if no bean is found?
## What is `NoUniqueBeanDefinitionException`?

### Injection styles

## Constructor vs setter vs field injection?

## Why is constructor injection preferred? ## When does field injection happen? ## When does setter injection happen? ## Can private fields be autowired? ## Can static fields be autowired?

### Internals

## Which Spring component processes `@Autowired`?

## What is `AutowiredAnnotationBeanPostProcessor`? ## At what stage of the bean lifecycle does autowiring happen? ## How does dependency resolution work? ## How do Spring proxies affect injected objects?

### Advanced

## How does `@Autowired` work with multiple implementations?

## How do you inject all implementations?
## How do you inject `Map<String, Interface>`?
## How do you make a dependency optional?
30. `Optional<T>` vs `ObjectProvider<T>`?
## How does `@Lazy` affect injection?
## How does `@Autowired` relate to circular dependencies?
## Constructor circular dependency vs setter circular dependency?
## How does prototype scope interact with injection?
35. `@Autowired` vs `@Resource` vs `@Inject`?

---

## The most important points to remember

```text @Autowired ↓ Spring dependency injection annotation ```

```text @Autowired ↓ Primarily type-based resolution ```

```text Multiple beans ↓ @Primary / @Qualifier ```

```text @Autowired ↓ AutowiredAnnotationBeanPostProcessor ```

```text Field injection ↓ Object created first ↓ Dependency injected afterward ```

```text Constructor injection ↓ Dependencies resolved for constructor ↓ Object constructed with dependencies ```

```text Single constructor ↓ @Autowired usually not required ```

```text Preferred approach ↓ Constructor injection ↓ final fields ```

And the most important architectural distinction:

```text
@Bean / @Component / @Service
        ↓
Bean registration / creation
        ↓
### Spring Container
        ↓
@Autowired
        ↓
Dependency injection
```

So **`@Autowired` doesn't mean *create this object.*** It means ***inject a suitable Spring-managed dependency here.***


`@Qualifier` is a **Spring annotation used to tell the Spring IoC container exactly which bean should be injected when multiple beans of the same type exist.**

It is mainly used with **Dependency Injection**, especially constructor, setter, and field injection.

---

# 1. Why do we need `@Qualifier`?

Suppose you have an interface:

```java
public interface PaymentService {
    void pay();
}
```

And two implementations:

```java @Service public class CreditCardPaymentService implements PaymentService {

    @Override
    public void pay() {
    System.out.println(*Credit Card Payment*);
    }
}
```

```java @Service public class UpiPaymentService implements PaymentService {

    @Override
    public void pay() {
    System.out.println(***UPI** Payment*);
    }
}
```

Now you have **two Spring beans** of type `PaymentService`.

If you write:

```java @Service public class OrderService {

    @Autowired
    private PaymentService paymentService;
}
```

Spring sees:

```text
PaymentService
    |
    +---- CreditCardPaymentService
    |
    +---- UpiPaymentService
```

Spring knows the required type is:

```text PaymentService ```

but it doesn't know **which implementation** you want.

This results in:

```text NoUniqueBeanDefinitionException ```

Typically:

```text No qualifying bean of type 'PaymentService' available: expected single matching bean but found 2 ```

This is where `@Qualifier` comes in.

---

# 2. Basic `@Qualifier` example

Give each implementation a qualifier:

```java @Service @Qualifier(*creditCard*) public class CreditCardPaymentService implements PaymentService {

    @Override
    public void pay() {
    System.out.println(*Credit Card Payment*);
    }
}
```

```java @Service @Qualifier(*upi*) public class UpiPaymentService implements PaymentService {

    @Override
    public void pay() {
    System.out.println(***UPI** Payment*);
    }
}
```

Then:

```java @Service public class OrderService {

    private final PaymentService paymentService;

    @Autowired
    public OrderService(
    @Qualifier(*upi*) PaymentService paymentService) {

    this.paymentService = paymentService;
    }
}
```

Now Spring understands:

```text Required type: PaymentService

Qualifier: upi

Find bean: UpiPaymentService ```

So:

```text
OrderService
    |
    | @Qualifier(*upi*)
    ↓
UpiPaymentService
```

---

# 3. `@Qualifier` with bean names

There is an important point here.

You don't necessarily have to put `@Qualifier` on the implementation.

For example:

```java
@Service(*creditCardPayment*)
public class CreditCardPaymentService
        implements PaymentService {
}
```

```java
@Service(*upiPayment*)
public class UpiPaymentService
        implements PaymentService {
}
```

Then:

```java @Service public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
    @Qualifier(*upiPayment*)
    PaymentService paymentService) {

    this.paymentService = paymentService;
    }
}
```

Here:

```java @Qualifier(*upiPayment*) ```

matches the bean name:

```java @Service(*upiPayment*) ```

So Spring injects:

```text UpiPaymentService ```

---

# 4. `@Qualifier` with field injection

You can use it with field injection:

```java @Service public class OrderService {

    @Autowired
    @Qualifier(*upiPayment*)
    private PaymentService paymentService;
}
```

Spring effectively says:

```text
Find beans of type PaymentService
        ↓
CreditCardPaymentService
UpiPaymentService
        ↓
Apply qualifier *upiPayment*
        ↓
UpiPaymentService
        ↓
Inject it
```

However, for modern Spring applications, **constructor injection is generally preferred** over field injection.

---

# 5. `@Qualifier` with setter injection

You can also use it with setter injection:

```java @Service public class OrderService {

    private PaymentService paymentService;

    @Autowired
    public void setPaymentService(
    @Qualifier(*upiPayment*)
    PaymentService paymentService) {

    this.paymentService = paymentService;
    }
}
```

---

# 6. `@Qualifier` with constructor injection

This is the recommended style:

```java @Service public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
    @Qualifier(*upiPayment*)
    PaymentService paymentService) {

    this.paymentService = paymentService;
    }
}
```

If there is only **one constructor**, `@Autowired` is not required.

So this is enough:

```java @Service public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
    @Qualifier(*upiPayment*)
    PaymentService paymentService) {

    this.paymentService = paymentService;
    }
}
```

---

# 7. How Spring resolves `@Qualifier`

This is important for interviews.

Suppose:

```java public interface PaymentService { } ```

There are three beans:

```text CreditCardPaymentService UpiPaymentService PaypalPaymentService ```

All implement:

```java PaymentService ```

And you have:

```java
public OrderService(
    @Qualifier(*upiPayment*)
    PaymentService paymentService) {
}
```

Spring's dependency resolution is roughly:

```text Step 1 Find dependency type

PaymentService

        ↓

Step 2 Find candidate beans

CreditCardPaymentService UpiPaymentService PaypalPaymentService

        ↓

Step 3 Apply @Qualifier

*upiPayment*

        ↓

Step 4 Find matching candidate

UpiPaymentService

        ↓

Step 5 Inject bean ```

So `@Qualifier` acts as a **candidate narrowing mechanism**.

---

# 8. `@Primary` vs `@Qualifier`

This is a very common interview question.

Suppose:

```java
@Service
@Primary
public class CreditCardPaymentService
        implements PaymentService {
}
```

```java
@Service
public class UpiPaymentService
        implements PaymentService {
}
```

Now:

```java @Autowired private PaymentService paymentService; ```

Spring chooses:

```text CreditCardPaymentService ```

because it is marked:

```java @Primary ```

But:

```java @Autowired @Qualifier(*upiPayment*) private PaymentService paymentService; ```

will choose:

```text UpiPaymentService ```

### Priority concept

Think of it like:

```text
@Autowired
    ↓
Find beans matching type
    ↓
@Qualifier?
    ↓
Use matching qualifier
```

`@Primary` is useful when you want a **default bean**.

`@Qualifier` is useful when you want a **specific bean**.

---

# 9. `@Qualifier` vs `@Primary`

| Feature                  | `@Qualifier`         | `@Primary`          |
| ------------------------ | -------------------- | ------------------- |
| Purpose                  | Select specific bean | Define default bean |
| Used at injection point  | Yes                  | No                  |
| Used on bean             | Yes                  | Yes                 |
| Multiple implementations | Excellent            | Good                |
| Explicit selection       | Yes                  | No                  |
| Default choice           | No                   | Yes                 |

Example:

```java
@Service
@Primary
public class CreditCardPaymentService
        implements PaymentService {
}
```

```java
@Service
@Qualifier(*upi*)
public class UpiPaymentService
        implements PaymentService {
}
```

Without qualifier:

```java PaymentService service; ```

→ `CreditCardPaymentService`

With:

```java @Qualifier(*upi*) PaymentService service; ```

→ `UpiPaymentService`

---

# 10. Is `@Qualifier` a bean name?

This is an important distinction.

People often say:

> `@Qualifier` is the bean name.

That's **not completely accurate**.

Consider:

```java
@Service(*upiPayment*)
public class UpiPaymentService
        implements PaymentService {
}
```

Here:

```text Bean name = upiPayment ```

But:

```java @Qualifier(*upi*) ```

is a **qualifier value**.

They can happen to match, but conceptually they are different mechanisms.

For example:

```java
@Service(*paymentBean*)
@Qualifier(*upi*)
public class UpiPaymentService
        implements PaymentService {
}
```

Now:

```text Bean name: paymentBean

Qualifier: upi ```

And:

```java @Qualifier(*upi*) PaymentService service ```

can identify it using the qualifier.

---

# 11. `@Qualifier` can be used with `@Bean`

You can also define qualifiers on `@Bean` methods.

```java @Configuration public class PaymentConfig {

    @Bean
    @Qualifier(*upi*)
    public PaymentService upiPaymentService() {
    return new UpiPaymentService();
    }

    @Bean
    @Qualifier(*creditCard*)
    public PaymentService creditCardPaymentService() {
    return new CreditCardPaymentService();
    }
}
```

Then:

```java @Service public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
    @Qualifier(*upi*)
    PaymentService paymentService) {

    this.paymentService = paymentService;
    }
}
```

Spring selects the bean associated with:

```text @Qualifier(*upi*) ```

---

# 12. Multiple qualifiers

Spring also supports multiple qualifier annotations in more advanced scenarios, especially when using custom qualifier annotations.

For example, instead of relying on strings:

```java @Qualifier(*upi*) ```

you can create your own qualifier.

```java
@Target({
    ElementType.**FIELD**,
    ElementType.**PARAMETER**,
    ElementType.**TYPE**,
    ElementType.**METHOD**
})
@Retention(RetentionPolicy.**RUNTIME**)
@Qualifier
public @interface PaymentType {

    String value();
}
```

Then:

```java
@Service
@PaymentType(*upi*)
public class UpiPaymentService
        implements PaymentService {
}
```

And:

```java
public OrderService(
    @PaymentType(*upi*)
    PaymentService paymentService) {

    this.paymentService = paymentService;
}
```

This is particularly useful in **large enterprise applications**, where string-based qualifiers can become difficult to maintain.

---

# 13. `@Qualifier` with generic types

Modern Spring applications can also use generics as part of dependency resolution.

For example:

```java public interface PaymentProcessor<T> {

    void process(T payment);
}
```

Implementations:

```java
@Component
public class UpiPaymentProcessor
        implements PaymentProcessor<UpiPayment> {
}
```

```java
@Component
public class CardPaymentProcessor
        implements PaymentProcessor<CardPayment> {
}
```

Spring can use generic type information to narrow candidates in supported injection scenarios.

However, `@Qualifier` is still useful when the distinction is based on an explicit business role rather than simply the Java generic type.

---

# 14. `@Qualifier` with collections

You can inject multiple beans:

```java @Autowired private List<PaymentService> paymentServices; ```

Then Spring gives you:

```text
[
    CreditCardPaymentService,
    UpiPaymentService,
    PaypalPaymentService
]
```

But you can also use qualifiers in collection injection scenarios to narrow what you want, depending on how the qualifiers are defined.

For example:

```java @Autowired @Qualifier(*online*) private List<PaymentService> paymentServices; ```

This can be useful when grouping beans by a common qualifier.

---

# 15. `@Qualifier` is not the same as `@Autowired`

This distinction is important.

### `@Autowired`

Answers:

> **What dependency do I need?**

Example:

```java @Autowired PaymentService paymentService; ```

Meaning:

```text I need a PaymentService. ```

### `@Qualifier`

Answers:

> **Which candidate should I use?**

Example:

```java @Autowired @Qualifier(*upi*) PaymentService paymentService; ```

Meaning:

```text I need a PaymentService, specifically the **UPI** one. ```

So:

```text
@Autowired
     ↓
Dependency injection

@Qualifier
     ↓
Candidate selection
```

---

# 16. Internal working — interview level

For a 7–8 year experienced Spring developer, you should understand the basic internal flow.

When Spring creates:

```java OrderService ```

it needs to resolve:

```java PaymentService paymentService ```

Spring's `BeanFactory` / `ApplicationContext` eventually delegates dependency resolution to mechanisms involving:

```text
AutowiredAnnotationBeanPostProcessor
            ↓
Dependency resolution
            ↓
DefaultListableBeanFactory
            ↓
resolveDependency()
            ↓
Find candidate beans
            ↓
Apply qualifiers
            ↓
Select candidate
            ↓
Return bean
            ↓
Inject into OrderService
```

Conceptually:

```text
OrderService
    |
    | requires PaymentService
    ↓
resolveDependency()
    |
    ↓
Find candidates
    |
    +---- CreditCardPaymentService
    |
    +---- UpiPaymentService
    |
    +---- PaypalPaymentService
    |
    ↓
@Qualifier(*upi*)
    |
    ↓
UpiPaymentService
    |
    ↓
Inject
```

The key class you should remember for interviews is:

```text DefaultListableBeanFactory ```

and, for processing `@Autowired`/related injection annotations:

```text AutowiredAnnotationBeanPostProcessor ```

---

# 17. What happens if there is no `@Qualifier`?

Suppose:

```java
@Service
class CreditCardPaymentService
        implements PaymentService {
}
```

```java
@Service
class UpiPaymentService
        implements PaymentService {
}
```

And:

```java
public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

Spring finds:

```text
PaymentService
    ├── CreditCardPaymentService
    └── UpiPaymentService
```

Two candidates exist.

If there is no way to choose between them, Spring throws:

```text NoUniqueBeanDefinitionException ```

Unless another mechanism resolves the ambiguity, such as:

```text @Primary ```

or

```text @Qualifier ```

---

# 18. What if both `@Primary` and `@Qualifier` exist?

Example:

```java
@Service
@Primary
public class CreditCardPaymentService
        implements PaymentService {
}
```

```java
@Service
@Qualifier(*upi*)
public class UpiPaymentService
        implements PaymentService {
}
```

Injection:

```java
public OrderService(
    @Qualifier(*upi*)
    PaymentService paymentService) {
}
```

The qualifier specifically identifies:

```text UpiPaymentService ```

So the qualified bean is selected rather than simply using the primary candidate.

---

# 19. `@Qualifier` with `@Resource`

Don't confuse:

```java @Autowired @Qualifier(*upi*) ```

with:

```java @Resource(name = *upiPayment*) ```

They have different semantics.

### `@Autowired`

Primarily **type-based** resolution.

```java @Autowired PaymentService service; ```

### `@Qualifier`

Narrows the candidates by qualifier.

```java @Autowired @Qualifier(*upi*) PaymentService service; ```

### `@Resource`

Primarily **name-based** resolution.

```java @Resource(name = *upiPayment*) PaymentService service; ```

A useful interview summary:

```text
@Autowired
    → primarily type

@Autowired + @Qualifier
    → type + qualifier

@Resource
    → primarily name
```

---

# 20. Real-world example

Imagine an insurance application with multiple notification channels:

```java public interface NotificationService {

    void send(String message);
}
```

Implementations:

```java
@Service
@Qualifier(*email*)
public class EmailNotificationService
        implements NotificationService {

    @Override
    public void send(String message) {
    System.out.println(*Sending Email*);
    }
}
```

```java
@Service
@Qualifier(*sms*)
public class SmsNotificationService
        implements NotificationService {

    @Override
    public void send(String message) {
    System.out.println(*Sending **SMS***);
    }
}
```

```java
@Service
@Qualifier(*push*)
public class PushNotificationService
        implements NotificationService {

    @Override
    public void send(String message) {
    System.out.println(*Sending Push Notification*);
    }
}
```

Now:

```java @Service public class PolicyService {

    private final NotificationService notificationService;

    public PolicyService(
    @Qualifier(*email*)
    NotificationService notificationService) {

    this.notificationService = notificationService;
    }

    public void issuePolicy() {
    notificationService.send(*Policy issued successfully*);
    }
}
```

Spring injects:

```text
PolicyService
    |
    | @Qualifier(*email*)
    ↓
EmailNotificationService
```

This is a very common enterprise use case.

---

# 21. `@Qualifier` vs bean name vs `@Primary`

Keep these three concepts separate:

```text
@Bean / @Component
        ↓
creates/registers bean

Bean name
        ↓
identifies bean in container

@Primary
        ↓
marks preferred candidate

@Qualifier
        ↓
narrows/selects candidate
```

For example:

```java
@Service(*upiPayment*)
@Qualifier(*upi*)
public class UpiPaymentService
        implements PaymentService {
}
```

You can conceptually have:

```text Bean name     → upiPayment Qualifier     → upi Type          → PaymentService ```

Then:

```java @Qualifier(*upi*) PaymentService service; ```

selects the bean associated with that qualifier.

---

# 22. Interview questions you should know

For a senior Java/Spring interview, be ready for:

1. **Why do we need `@Qualifier`?**
## What happens when multiple beans have the same type?
## Difference between `@Qualifier` and `@Primary`.
## Difference between `@Qualifier` and `@Resource`.
## Can `@Qualifier` be used with constructor injection?
## Can it be used with setter injection?
## Can it be used with field injection?
## Can it be used with `@Bean` methods?
## Is `@Qualifier` the same as bean name?
## How does Spring resolve a dependency when multiple candidates exist?
## What is `NoUniqueBeanDefinitionException`?
## What happens when both `@Primary` and `@Qualifier` are present?
## Can we create custom qualifier annotations?
## What role does `DefaultListableBeanFactory` play?
## What role does `AutowiredAnnotationBeanPostProcessor` play?

### The key thing to remember

```text
@Autowired
    ↓
*I need a PaymentService*

@Qualifier(*upi*)
    ↓
*Of all PaymentService beans,
 give me the **UPI** one*
```

So the simplest definition is:

> **`@Qualifier` is used to resolve ambiguity when multiple Spring beans match the required dependency type by narrowing the candidate to a specific qualified bean.**
> 
> # `@Primary` in Spring

`@Primary` is a Spring annotation used to **tell the Spring IoC container which bean should be preferred when multiple beans of the same type are available for dependency injection**.

In simple terms:

> **`@Primary` marks one bean as the default/preferred candidate when Spring finds multiple beans matching the required type.**

---

## 1. Why do we need `@Primary`?

Suppose you have an interface:

```java
public interface PaymentService {
    void pay();
}
```

And two implementations:

```java
@Service
public class CreditCardPaymentService implements PaymentService {

    @Override
    public void pay() {
        System.out.println("Credit Card Payment");
    }
}
```

```java
@Service
public class UpiPaymentService implements PaymentService {

    @Override
    public void pay() {
        System.out.println("UPI Payment");
    }
}
```

Now suppose:

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Spring needs to inject a `PaymentService`.

But there are two candidates:

```text
PaymentService
     |
     +---- CreditCardPaymentService
     |
     +---- UpiPaymentService
```

Spring doesn't know which one to choose.

You will typically get:

```text
NoUniqueBeanDefinitionException
```

---

# 2. Using `@Primary`

You can tell Spring which implementation should be the default:

```java
@Service
@Primary
public class CreditCardPaymentService implements PaymentService {

    @Override
    public void pay() {
        System.out.println("Credit Card Payment");
    }
}
```

The other implementation remains normal:

```java
@Service
public class UpiPaymentService implements PaymentService {

    @Override
    public void pay() {
        System.out.println("UPI Payment");
    }
}
```

Now:

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Spring sees:

```text
PaymentService
     |
     +---- CreditCardPaymentService (@Primary)
     |
     +---- UpiPaymentService
```

Therefore:

```text
OrderService
     |
     ↓
CreditCardPaymentService
```

gets injected.

---

# 3. What exactly does `@Primary` mean?

It does **not** mean:

> "Create this bean first."

It does **not** mean:

> "This bean has higher priority during bean creation."

It means:

> **"If multiple beans are candidates for this dependency, prefer this bean."**

For example:

```java
@Service
@Primary
public class CreditCardPaymentService implements PaymentService {
}
```

means:

```text
If Spring needs PaymentService
and multiple PaymentService beans exist
        ↓
Prefer CreditCardPaymentService
```

---

# 4. `@Primary` does NOT prevent other beans from being created

This is very important.

Suppose:

```java
@Service
@Primary
public class CreditCardPaymentService implements PaymentService {
}
```

and:

```java
@Service
public class UpiPaymentService implements PaymentService {
}
```

Both are still Spring beans.

```text
Spring Container

CreditCardPaymentService
        |
        | @Primary
        ↓
Preferred candidate

UpiPaymentService
        |
        ↓
Still a valid Spring bean
```

`@Primary` doesn't remove or disable `UpiPaymentService`.

It only affects **autowire candidate selection**.

---

# 5. `@Primary` with constructor injection

This is the most common modern usage.

```java
@Service
@Primary
public class CreditCardPaymentService
        implements PaymentService {
}
```

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Because `CreditCardPaymentService` is primary:

```text
PaymentService
      ↓
multiple candidates
      ↓
find @Primary
      ↓
CreditCardPaymentService
      ↓
inject
```

---

# 6. `@Primary` with `@Autowired`

It also works with field injection:

```java
@Service
public class OrderService {

    @Autowired
    private PaymentService paymentService;
}
```

If:

```java
@Service
@Primary
public class CreditCardPaymentService
        implements PaymentService {
}
```

then:

```text
paymentService
      ↓
CreditCardPaymentService
```

is injected.

---

# 7. `@Primary` with `@Bean`

You can also use `@Primary` with Java configuration.

```java
@Configuration
public class PaymentConfig {

    @Bean
    @Primary
    public PaymentService creditCardPaymentService() {
        return new CreditCardPaymentService();
    }

    @Bean
    public PaymentService upiPaymentService() {
        return new UpiPaymentService();
    }
}
```

Now:

```java
public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

gets:

```text
creditCardPaymentService
```

because it is marked `@Primary`.

---

# 8. `@Primary` vs `@Qualifier`

This is one of the **most important Spring interview questions**.

Suppose:

```java
@Service
@Primary
public class CreditCardPaymentService
        implements PaymentService {
}
```

```java
@Service
public class UpiPaymentService
        implements PaymentService {
}
```

Without a qualifier:

```java
public OrderService(PaymentService paymentService) {
}
```

Spring chooses:

```text
CreditCardPaymentService
```

because it is `@Primary`.

But if you explicitly specify:

```java
public OrderService(
        @Qualifier("upiPaymentService")
        PaymentService paymentService) {
}
```

then Spring selects the UPI bean.

So:

```text
@Primary
    ↓
Default/preferred candidate

@Qualifier
    ↓
Specific candidate
```

---

# 9. Think of it like a default

Suppose you have:

```text
PaymentService
 ├── CreditCardPaymentService  ← @Primary
 ├── UpiPaymentService
 └── PaypalPaymentService
```

If you say:

```java
PaymentService paymentService;
```

Spring thinks:

```text
Which one?

CreditCard?
UPI?
PayPal?
```

`@Primary` says:

```text
Use CreditCardPaymentService by default.
```

But if you say:

```java
@Qualifier("upiPaymentService")
PaymentService paymentService;
```

you're saying:

```text
I don't want the default.
I specifically want UPI.
```

---

# 10. Can multiple beans have `@Primary`?

You **should not have multiple primary candidates for the same dependency type** when Spring needs to select a single bean.

For example:

```java
@Service
@Primary
public class CreditCardPaymentService
        implements PaymentService {
}
```

```java
@Service
@Primary
public class UpiPaymentService
        implements PaymentService {
}
```

Now:

```java
public OrderService(PaymentService paymentService) {
}
```

Spring still has an ambiguity:

```text
PaymentService
      |
      +---- CreditCardPaymentService @Primary
      |
      +---- UpiPaymentService @Primary
```

There isn't a unique primary candidate.

Therefore Spring can throw:

```text
NoUniqueBeanDefinitionException
```

So normally you should have **one preferred bean** for a given dependency.

---

# 11. What if there is only one bean?

Suppose:

```java
@Service
public class UpiPaymentService
        implements PaymentService {
}
```

There is only one candidate.

You don't need:

```java
@Primary
```

Spring can simply inject it.

```text
PaymentService
      ↓
One candidate
      ↓
UpiPaymentService
```

`@Primary` becomes useful when there are **multiple candidates**.

---

# 12. What happens internally?

This is the important part for a senior-level Spring interview.

Suppose:

```java
public OrderService(PaymentService paymentService) {
}
```

Spring has to resolve the dependency.

Conceptually:

```text
OrderService creation
        ↓
Dependency required:
PaymentService
        ↓
Spring searches for candidates
        ↓
Find all PaymentService beans
        ↓
CreditCardPaymentService
UpiPaymentService
PaypalPaymentService
        ↓
Multiple candidates
        ↓
Check @Primary
        ↓
CreditCardPaymentService
        ↓
Inject it
```

The relevant Spring infrastructure includes:

```text
AutowiredAnnotationBeanPostProcessor
              ↓
Dependency resolution
              ↓
DefaultListableBeanFactory
              ↓
resolveDependency()
```

`DefaultListableBeanFactory` is particularly important because it performs much of Spring's bean lookup and dependency-resolution work.

---

# 13. `@Primary` is part of candidate selection

This is the key concept.

Imagine:

```text
@Autowired
PaymentService paymentService;
```

Spring first needs to determine:

> Which beans are eligible candidates?

Then it needs to determine:

> If there are multiple candidates, which one should be selected?

`@Primary` participates in that **candidate selection process**.

Conceptually:

```text
Dependency:
PaymentService
        ↓
Find candidates
        ↓
Candidate 1
Candidate 2
Candidate 3
        ↓
Is there a @Primary candidate?
        ↓
Yes
        ↓
Select primary candidate
```

---

# 14. `@Primary` doesn't mean "higher bean priority"

This is a common misunderstanding.

Don't say in an interview:

> "`@Primary` gives the bean higher priority in the Spring lifecycle."

That's incorrect.

It is about **autowiring ambiguity resolution**, not:

* bean creation order
* bean initialization order
* bean lifecycle priority
* execution priority
* startup priority

Correct statement:

> "`@Primary` marks a bean as the preferred candidate for autowiring when multiple beans of the same type are available."

---

# 15. `@Primary` and bean creation order

For example:

```java
@Service
@Primary
public class CreditCardPaymentService {
    
    public CreditCardPaymentService() {
        System.out.println("Credit Card created");
    }
}
```

```java
@Service
public class UpiPaymentService {
    
    public UpiPaymentService() {
        System.out.println("UPI created");
    }
}
```

`@Primary` does **not** mean:

```text
CreditCardPaymentService
        ↓
must be created first
```

Both are independent beans.

`@Primary` only tells Spring which one to select **when resolving an ambiguous dependency**.

---

# 16. `@Primary` and `@Qualifier` together

Suppose:

```java
@Service
@Primary
public class CreditCardPaymentService
        implements PaymentService {
}
```

```java
@Service
@Qualifier("upi")
public class UpiPaymentService
        implements PaymentService {
}
```

Then:

```java
public OrderService(PaymentService paymentService) {
}
```

gets:

```text
CreditCardPaymentService
```

because it is primary.

But:

```java
public OrderService(
        @Qualifier("upi")
        PaymentService paymentService) {
}
```

gets:

```text
UpiPaymentService
```

So a useful mental model is:

```text
No qualifier
    ↓
Prefer @Primary

Explicit @Qualifier
    ↓
Select the qualified candidate
```

---

# 17. Real-world example

Imagine an insurance application with multiple notification services:

```java
public interface NotificationService {
    void send(String message);
}
```

You have:

```java
@Service
@Primary
public class EmailNotificationService
        implements NotificationService {

    @Override
    public void send(String message) {
        System.out.println("Sending Email");
    }
}
```

```java
@Service
public class SmsNotificationService
        implements NotificationService {

    @Override
    public void send(String message) {
        System.out.println("Sending SMS");
    }
}
```

```java
@Service
public class PushNotificationService
        implements NotificationService {

    @Override
    public void send(String message) {
        System.out.println("Sending Push Notification");
    }
}
```

Now:

```java
@Service
public class PolicyService {

    private final NotificationService notificationService;

    public PolicyService(NotificationService notificationService) {
        this.notificationService = notificationService;
    }

    public void issuePolicy() {
        notificationService.send("Policy issued");
    }
}
```

Spring chooses:

```text
PolicyService
      |
      ↓
NotificationService
      |
      +---- EmailNotificationService @Primary ← SELECTED
      |
      +---- SmsNotificationService
      |
      +---- PushNotificationService
```

So email becomes the **default notification mechanism**.

---

# 18. `@Primary` vs `@Qualifier` — interview answer

If the interviewer asks:

> **What is the difference between `@Primary` and `@Qualifier`?**

A good answer is:

> "`@Primary` is used to designate a default or preferred bean when multiple beans of the same type are available. `@Qualifier` is used at the injection point to explicitly select a particular bean. If there are multiple candidates, `@Primary` provides a default choice, whereas `@Qualifier` provides explicit candidate selection."

---

# 19. `@Primary` vs `@Qualifier` example

```java
@Service
@Primary
public class EmailNotificationService
        implements NotificationService {
}
```

```java
@Service
public class SmsNotificationService
        implements NotificationService {
}
```

Default:

```java
public NotificationService service;
```

→ Email

Specific:

```java
public MyService(
        @Qualifier("smsNotificationService")
        NotificationService service) {
}
```

→ SMS

Think:

```text
@Primary
    =
"Use this one by default"

@Qualifier
    =
"Use this exact one"
```

---

# 20. Important interview points

For a senior Spring developer, remember these:

### `@Primary`

* Resolves ambiguity between multiple beans.
* Marks a bean as the preferred candidate.
* Works with `@Component`, `@Service`, `@Repository`, etc.
* Works with `@Bean`.
* Does not prevent other beans from being created.
* Does not control bean creation order.
* Does not control initialization order.
* Does not mean higher lifecycle priority.
* Multiple primary candidates can still cause ambiguity.
* An explicit qualifier can select another candidate.

### Most important mental model

```text
Multiple beans
      ↓
Same dependency type
      ↓
Spring has multiple candidates
      ↓
@Primary
      ↓
Choose this bean as the default
```

And the one-line definition to remember for interviews:

> **`@Primary` marks a Spring bean as the preferred candidate for autowiring when multiple beans match the required dependency type.**

## `@Resource` in Spring

`@Resource` is a **dependency injection annotation** defined by **Jakarta/**JDK** standard annotations**, commonly used by Spring to inject a bean.

```java @Service public class OrderService {

    @Resource
    private PaymentService paymentService;
}
```

Spring finds a `PaymentService` bean and injects it into `OrderService`.

### 1. How `@Resource` resolves the dependency

The important interview point:

> **`@Resource` primarily resolves by bean name, then falls back to type.**

Example:

```java @Resource private PaymentService paymentService; ```

Spring first looks for a bean named:

```text paymentService ```

If it cannot find one, it can resolve by type.

---

### 2. `@Resource(name = *...*)`

You can explicitly specify the bean name:

```java @Resource(name = *stripePaymentService*) private PaymentService paymentService; ```

This tells Spring exactly which bean to inject.

With:

```java @Service(*stripePaymentService*) public class StripePaymentService implements PaymentService { } ```

the injection works by name.

---

### 3. Multiple implementations

Suppose:

```java public interface PaymentService { } ```

```java @Service(*stripePaymentService*) public class StripePaymentService implements PaymentService { } ```

```java @Service(*paypalPaymentService*) public class PaypalPaymentService implements PaymentService { } ```

Then:

```java @Resource(name = *stripePaymentService*) private PaymentService paymentService; ```

selects `StripePaymentService`.

Without specifying the name:

```java @Resource private PaymentService paymentService; ```

Spring's name-based resolution becomes important.

---

### 4. `@Resource` vs `@Autowired`

|                       | `@Resource`                     | `@Autowired`              |
| --------------------- | ------------------------------- | ------------------------- |
| Standard              | Jakarta annotation              | Spring annotation         |
| Primary resolution    | **Name → Type**                 | **Type → Qualifier/Name** |
| `name` attribute      | Yes                             | No                        |
| `@Qualifier`          | Usually unnecessary with `name` | Commonly used             |
| Constructor injection | Not the typical choice          | **Preferred**             |

Example with `@Autowired`:

```java @Autowired @Qualifier(*stripePaymentService*) private PaymentService paymentService; ```

Equivalent intent using `@Resource`:

```java @Resource(name = *stripePaymentService*) private PaymentService paymentService; ```

---

### 5. Field vs setter injection

`@Resource` is commonly used on fields:

```java @Resource private PaymentService paymentService; ```

It can also be used on a setter:

```java
@Resource
public void setPaymentService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

For modern Spring applications, **constructor injection is generally preferred** because dependencies are explicit and the object can be immutable.

---

### 6. Important 7–8 YOE interview points

Know these:

- `@Resource` is primarily **name-based dependency injection**.
- Resolution is generally **by name first, then by type**.
- `@Resource(name = *...*)` explicitly selects a bean.
- It is useful when you have **multiple implementations** and want name-based selection.
- `@Autowired` is Spring-specific and primarily **type-based**.
- `@Qualifier` is normally used with `@Autowired` when selecting between multiple beans.
- `@Resource` can be applied to **fields and setter methods**.
- Prefer **constructor injection** for new production code.

### Simple mental model

```text
@Resource
    ↓
Bean name?
    ↓
Find matching bean
    ↓
If not resolved → type-based resolution
```

The **most important distinction to remember for interviews** is:

> **`@Resource` → name first** > **`@Autowired` → type first**
> 
>
>## `@Inject` in Spring

`@Inject` is a **Jakarta Dependency Injection (DI) standard annotation**. Spring supports it as an alternative to `@Autowired`.

```java @Service public class OrderService {

    private final PaymentService paymentService;

    @Inject
    public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
    }
}
```

Spring detects `@Inject` and injects the required `PaymentService` bean.

### 1. How `@Inject` resolves dependencies

The key point:

> **`@Inject` is primarily type-based.**

If there is only one `PaymentService` implementation:

```java @Service public class StripePaymentService implements PaymentService { } ```

then:

```java @Inject private PaymentService paymentService; ```

works because Spring finds the bean by type.

---

### 2. Multiple implementations

If you have:

```java @Service public class StripePaymentService implements PaymentService { } ```

```java @Service public class PaypalPaymentService implements PaymentService { } ```

then:

```java @Inject private PaymentService paymentService; ```

is ambiguous because there are two candidates.

Use `@Named`:

```java @Inject @Named(*stripePaymentService*) private PaymentService paymentService; ```

This is conceptually similar to:

```java @Autowired @Qualifier(*stripePaymentService*) private PaymentService paymentService; ```

---

### 3. `@Inject` vs `@Autowired` vs `@Resource`

| Feature               | `@Inject`  | `@Autowired` | `@Resource`               |
| --------------------- | ---------- | ------------ | ------------------------- |
| Standard              | Jakarta DI | Spring       | Jakarta                   |
| Primary resolution    | **Type**   | **Type**     | **Name**                  |
| Multiple beans        | `@Named`   | `@Qualifier` | `name`                    |
| Constructor injection | ✅          | ✅            | Possible, but less common |
| Spring-specific       | ❌          | ✅            | ❌                         |

### 4. Constructor injection

For modern Spring applications:

```java @Service public class OrderService {

    private final PaymentService paymentService;

    @Inject
    public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
    }
}
```

With Spring, `@Autowired` can be omitted when there is only **one constructor**:

```java @Service public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
    }
}
```

This is generally the preferred production approach.

### 5. What you should remember for interviews

> **`@Inject` = standard Jakarta DI annotation, primarily type-based.**

Know these three distinctions:

```text @Inject     → Type @Autowired  → Type + Spring features @Resource   → Name first, then type ```

And when multiple implementations exist:

```text @Inject + @Named @Autowired + @Qualifier @Resource(name = *...*) ```