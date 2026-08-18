# Dependency Injection and Constructor Dependency Injection in Spring

## 1. Dependency Injection (DI)

### Definition

**Dependency Injection (DI)** is a design technique where an object receives the objects it depends on from an external source instead of creating those dependencies itself.

In Spring, the **IoC container** is responsible for:

* Creating beans
* Resolving dependencies
* Injecting dependencies
* Managing bean lifecycle

### Without Dependency Injection

```java
public class OrderService {

    private PaymentService paymentService = new PaymentService();

    public void placeOrder() {
        paymentService.pay();
    }
}
```

`OrderService` is responsible for creating `PaymentService`.

This creates **tight coupling**.

```text
OrderService
     |
     | creates
     v
PaymentService
```

### With Dependency Injection

```java
public class OrderService {

    private PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    public void placeOrder() {
        paymentService.pay();
    }
}
```

Now:

```text
Spring IoC Container
        |
        | creates
        v
PaymentService
        |
        | injects
        v
OrderService
```

`OrderService` no longer controls the creation of its dependency.

---

# 2. Dependency vs Dependency Injection

These are different concepts.

### Dependency

A dependency is an object that another object requires to perform its work.

```java
class OrderService {

    private PaymentService paymentService;
}
```

Here:

```text
PaymentService = dependency
OrderService    = dependent object
```

### Dependency Injection

Providing that dependency from outside:

```java
OrderService(PaymentService paymentService)
```

is dependency injection.

---

# 3. IoC vs DI

### Inversion of Control

IoC is the broader principle.

Instead of application code controlling object creation:

```java
PaymentService service = new PaymentService();
```

the Spring container controls it.

### Dependency Injection

DI is one of the primary techniques used to implement IoC.

```text
IoC
 |
 +-- Spring controls object creation/lifecycle
 |
 +-- DI
      |
      +-- Spring supplies dependencies
```

A good interview statement:

> **IoC is the principle of transferring control of object creation and management to the container. Dependency Injection is a technique through which the container supplies required dependencies to objects.**

---

# 4. Types of Dependency Injection

Spring commonly supports:

1. Constructor Injection
2. Setter Injection
3. Field Injection

### Constructor Injection

```java
public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

### Setter Injection

```java
@Autowired
public void setPaymentService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

### Field Injection

```java
@Autowired
private PaymentService paymentService;
```

For mandatory dependencies, **constructor injection is the preferred approach**.

---

# 5. Constructor Injection

## Definition

Constructor Injection is a form of dependency injection where dependencies are provided through the class constructor when the object is created.

Example:

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

The important point is:

> The dependency is available during object construction.

---

# 6. How Constructor Injection Actually Works

Consider:

```java
@Component
public class Engine {

    public void start() {
        System.out.println("Engine started");
    }
}
```

```java
@Component
public class Car {

    private final Engine engine;

    public Car(Engine engine) {
        this.engine = engine;
    }

    public void drive() {
        engine.start();
    }
}
```

Spring sees:

```java
Car(Engine engine)
```

and understands:

```text
Car requires Engine
```

Spring resolves the `Engine` bean and conceptually performs:

```java
Engine engine = ...;

Car car = new Car(engine);
```

The important distinction is:

```text
NOT:

Create Car
   ↓
Inject Engine

BUT:

Resolve Engine
   ↓
Call Car constructor with Engine
   ↓
Car object created
```

---

# 7. Does the Object Exist Before Constructor Injection?

This is a common interview question.

Consider:

```java
Car car = new Car(engine);
```

Java performs object creation and constructor execution as part of the `new` operation.

Conceptually:

```text
Allocate memory
      ↓
Initialize object state
      ↓
Invoke constructor
      ↓
Constructor completes
      ↓
Reference returned
```

Therefore, with constructor injection:

```java
new Car(engine)
```

the dependency is supplied **as part of object creation**.

There is not an already-created, fully usable `Car` object waiting for the dependency.

---

# 8. Constructor Injection vs Setter Injection

### Constructor Injection

```java
Car car = new Car(engine);
```

The dependency is provided during construction.

### Setter Injection

```java
Car car = new Car();

car.setEngine(engine);
```

The object exists before the dependency is supplied.

Therefore:

```text
Constructor Injection

Dependency
    ↓
Constructor
    ↓
Object created
```

while:

```text
Setter Injection

Object created
    ↓
Dependency injected
```

---

# 9. Why Constructor Injection Is Preferred

## 9.1 Mandatory Dependencies

If `OrderService` cannot work without `PaymentService`:

```java
public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

The dependency is required at construction time.

---

## 9.2 Supports Immutability

```java
private final PaymentService paymentService;
```

The dependency can be assigned once:

```java
public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

There is no setter to replace it later.

---

## 9.3 Dependencies Are Explicit

The constructor clearly communicates what the class needs:

```java
public OrderService(
        PaymentService paymentService,
        OrderRepository orderRepository,
        NotificationService notificationService) {
}
```

A developer can immediately identify the class's dependencies.

---

## 9.4 Easier Unit Testing

You don't need the Spring container:

```java
PaymentService paymentService = mock(PaymentService.class);

OrderService service = new OrderService(paymentService);
```

This makes unit testing straightforward.

---

## 9.5 Prevents Partially Initialized Objects

With constructor injection:

```java
OrderService service = new OrderService(paymentService);
```

The object receives its required dependency during construction.

With setter injection:

```java
OrderService service = new OrderService();
```

the object can temporarily exist without its dependency.

---

# 10. Constructor Injection and `@Autowired`

In modern Spring, if a class has **one constructor**, `@Autowired` is not required.

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Spring automatically recognizes the single constructor.

You can still write:

```java
@Autowired
public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

but it is redundant for a single constructor.

---

# 11. Multiple Constructors

Suppose:

```java
@Service
public class OrderService {

    public OrderService() {
    }

    public OrderService(PaymentService paymentService) {
    }
}
```

Spring has multiple constructors.

You can explicitly specify the injection constructor:

```java
@Autowired
public OrderService(PaymentService paymentService) {
}
```

This removes ambiguity.

---

# 12. Constructor Injection with Interfaces

A common real-world approach is to depend on an interface.

```java
public interface PaymentService {

    void pay();
}
```

Implementation:

```java
@Service
public class StripePaymentService implements PaymentService {

    @Override
    public void pay() {
        System.out.println("Payment processed");
    }
}
```

Consumer:

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
OrderService
     |
     | requires
     v
PaymentService
     ^
     |
StripePaymentService
```

Spring injects the appropriate implementation.

---

# 13. Multiple Implementations

Suppose:

```java
@Service
public class StripePaymentService implements PaymentService {
}
```

and:

```java
@Service
public class RazorpayPaymentService implements PaymentService {
}
```

Now Spring has two candidates.

```java
PaymentService
      ^
      |
  +---+---+
  |       |
Stripe  Razorpay
```

Spring cannot determine which one to inject automatically.

This can produce:

```text
NoUniqueBeanDefinitionException
```

## Solution 1: `@Primary`

```java
@Service
@Primary
public class StripePaymentService implements PaymentService {
}
```

Spring chooses the primary bean.

## Solution 2: `@Qualifier`

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
            @Qualifier("razorpayPaymentService")
            PaymentService paymentService) {

        this.paymentService = paymentService;
    }
}
```

---

# 14. Constructor Injection Resolution

A simplified dependency-resolution flow:

```text
Spring needs OrderService
        |
        v
Inspect constructor
        |
        v
OrderService(PaymentService)
        |
        v
Resolve PaymentService
        |
        v
Find candidate bean
        |
        v
Create/obtain dependency
        |
        v
Invoke constructor
        |
        v
OrderService created
```

---

# 15. Dependency Creation Order

Suppose:

```java
@Component
class Battery {
}
```

```java
@Component
class Engine {

    private final Battery battery;

    Engine(Battery battery) {
        this.battery = battery;
    }
}
```

```java
@Component
class Car {

    private final Engine engine;

    Car(Engine engine) {
        this.engine = engine;
    }
}
```

Dependency graph:

```text
Car
 |
 v
Engine
 |
 v
Battery
```

Spring must resolve the dependencies needed to construct `Car`.

Conceptually:

```text
Create/resolve Battery
        ↓
Create Engine(Battery)
        ↓
Create Car(Engine)
```

This is dependency resolution, not simply "Spring creates every bean strictly in source-code order."

---

# 16. Is Constructor Injection Eager Loading?

No.

These are different concepts.

### Constructor Injection

Defines:

> How is the dependency provided?

Answer:

```text
Through constructor
```

### Eager Initialization

Defines:

> When is the bean created?

Answer:

```text
During application context initialization
```

For normal singleton beans, Spring eagerly initializes them by default.

Therefore:

```text
Constructor Injection ≠ Eager Initialization
```

---

# 17. Constructor Injection with `@Lazy`

Consider:

```java
@Component
@Lazy
public class ReportService {

    public ReportService() {
        System.out.println("ReportService created");
    }
}
```

Spring doesn't normally instantiate the lazy bean during startup.

It waits until it is needed.

Conceptually:

```text
Application starts
       ↓
ReportService not created
       ↓
Bean requested
       ↓
Create dependencies
       ↓
Call constructor
       ↓
Bean created
```

Constructor injection still works exactly the same way.

Only the **timing of bean creation** changes.

---

# 18. Constructor Injection and Proxies

This is important for experienced developers.

Suppose:

```java
@Service
@Transactional
public class PaymentService {

    public void pay() {
    }
}
```

Spring may create a proxy around the target object.

Conceptually:

```text
Real PaymentService
        ↑
        |
     Proxy
```

Another bean receiving `PaymentService` may receive the proxy:

```text
OrderService
     |
     v
PaymentService Proxy
     |
     v
Real PaymentService
```

A simplified lifecycle is:

```text
Resolve dependencies
        ↓
Create target object
        ↓
Initialize target
        ↓
Apply BeanPostProcessors
        ↓
Create/wrap with proxy when applicable
        ↓
Expose resulting bean
```

The exact proxy timing depends on the Spring lifecycle and post-processors involved, so avoid describing proxy creation as a universal single step for every bean.

---

# 19. Constructor Injection and Circular Dependency

Consider:

```java
@Service
class A {

    private final B b;

    A(B b) {
        this.b = b;
    }
}
```

and:

```java
@Service
class B {

    private final A a;

    B(A a) {
        this.a = a;
    }
}
```

Dependency graph:

```text
A
↓
B
↓
A
```

Spring needs `B` to create `A`.

But `B` needs `A` to be created.

Therefore neither can be constructed normally.

```text
Create A
   ↓
Need B
   ↓
Create B
   ↓
Need A
   ↓
A is not yet fully created
   ↓
Circular dependency
```

Constructor-based circular dependencies generally result in a startup failure rather than being resolved through the traditional early-singleton-exposure mechanism used for some setter/field-injection cases.

### Best solution

Redesign the dependency graph.

Common approaches include:

* Extracting shared responsibilities
* Introducing a third service
* Using an event-based design
* Applying `ObjectProvider` where deferred lookup is genuinely required
* Using `@Lazy` only when appropriate

Do not use `@Lazy` simply to hide a poor dependency design.

---

# 20. XML-Based Constructor Injection

Before annotation-based configuration became common, Spring applications often used XML.

### Java classes

```java
public class Engine {

    public void start() {
        System.out.println("Engine started");
    }
}
```

```java
public class Car {

    private final Engine engine;

    public Car(Engine engine) {
        this.engine = engine;
    }
}
```

### XML

```xml
<bean id="engine"
      class="com.example.Engine"/>

<bean id="car"
      class="com.example.Car">

    <constructor-arg ref="engine"/>

</bean>
```

The important element is:

```xml
<constructor-arg ref="engine"/>
```

It tells Spring:

> Pass the `engine` bean to the `Car` constructor.

Conceptually:

```java
Engine engine = new Engine();

Car car = new Car(engine);
```

---

# 21. XML Constructor Injection with Primitive Values

Java:

```java
public class Employee {

    private final int id;
    private final String name;

    public Employee(int id, String name) {
        this.id = id;
        this.name = name;
    }
}
```

XML:

```xml
<bean id="employee"
      class="com.example.Employee">

    <constructor-arg index="0" value="101"/>
    <constructor-arg index="1" value="John"/>

</bean>
```

Conceptually:

```java
new Employee(101, "John");
```

---

# 22. `ref` vs `value`

This distinction is important.

### `value`

Used to provide a literal value:

```xml
<constructor-arg value="101"/>
```

### `ref`

Used to reference another Spring bean:

```xml
<constructor-arg ref="engine"/>
```

Example:

```xml
<bean id="engine"
      class="com.example.Engine"/>

<bean id="car"
      class="com.example.Car">

    <constructor-arg ref="engine"/>

</bean>
```

---

# 23. XML vs Annotation-Based Configuration

### XML

```xml
<bean id="car"
      class="com.example.Car">

    <constructor-arg ref="engine"/>

</bean>
```

### Annotation

```java
@Component
public class Car {

    private final Engine engine;

    public Car(Engine engine) {
        this.engine = engine;
    }
}
```

Both ultimately achieve the same goal:

```text
Spring creates dependency
        ↓
Spring supplies dependency
        ↓
Target constructor executes
        ↓
Target object created
```

The primary difference is where configuration is expressed.

---

# 24. Constructor Injection and Spring Bean Lifecycle

Constructor injection occurs during the bean creation phase.

A simplified lifecycle is:

```text
BeanDefinition available
        ↓
Resolve constructor
        ↓
Resolve constructor arguments
        ↓
Instantiate bean
        ↓
Populate properties
        ↓
Aware callbacks
        ↓
BeanPostProcessor
        ↓
Initialization callbacks
        ↓
BeanPostProcessor
        ↓
Bean ready
```

Constructor injection belongs to the **instantiation stage**.

This distinction is important:

```text
Constructor Injection
        ↓
Instantiation

Setter / Field Injection
        ↓
Property population
```

---

# 25. Constructor Injection vs Property Injection

### Constructor Injection

```java
public Car(Engine engine) {
    this.engine = engine;
}
```

Dependency is supplied during instantiation.

### Setter Injection

```java
public void setEngine(Engine engine) {
    this.engine = engine;
}
```

Dependency is supplied after instantiation.

### Field Injection

```java
@Autowired
private Engine engine;
```

Dependency is also populated after the object has been instantiated.

---

# 26. Constructor Injection and Bean Scope

Constructor injection works with all Spring bean scopes.

Common scopes include:

* Singleton
* Prototype
* Request
* Session
* Application
* WebSocket

The scope determines the bean lifecycle/instance semantics; constructor injection determines how the dependency is supplied during construction.

---

# 27. Important: Singleton Does Not Mean "Created Once Globally"

A Spring singleton means:

> One bean instance per Spring `ApplicationContext`.

It does not mean one object for the entire JVM in all circumstances.

For example:

```text
ApplicationContext A
    ↓
Service instance A

ApplicationContext B
    ↓
Service instance B
```

Each context can have its own singleton instance.

---

# 28. Constructor Injection with Prototype Dependencies

Suppose:

```java
@Component
@Scope("prototype")
class Report {

}
```

and:

```java
@Component
class ReportService {

    private final Report report;

    ReportService(Report report) {
        this.report = report;
    }
}
```

If `ReportService` is a singleton, its constructor runs once.

Therefore, the prototype `Report` injected into it is also resolved during that singleton's creation.

This means:

```text
Singleton ReportService
        |
        +---- Report prototype instance
```

does **not** mean a new `Report` is created every time `ReportService` uses it.

If you need a new prototype instance for each operation, consider mechanisms such as:

```java
ObjectProvider<Report>
```

or appropriate scoped-proxy/factory designs.

---

# 29. Constructor Injection with `ObjectProvider`

For deferred or repeated lookup:

```java
@Component
class ReportService {

    private final ObjectProvider<Report> reportProvider;

    ReportService(ObjectProvider<Report> reportProvider) {
        this.reportProvider = reportProvider;
    }

    public void generate() {
        Report report = reportProvider.getObject();
    }
}
```

Here the dependency is not the actual `Report` instance.

The dependency is:

```text
ObjectProvider<Report>
```

which allows controlled retrieval of `Report` instances later.

---

# 30. Constructor Injection and Optional Dependencies

Constructor injection is best suited for mandatory dependencies.

For an optional dependency, Spring provides several mechanisms.

For example:

```java
public OrderService(
        PaymentService paymentService,
        @Autowired(required = false)
        AuditService auditService) {
}
```

However, modern designs often prefer explicit optional types or providers where appropriate.

Example:

```java
public OrderService(
        PaymentService paymentService,
        Optional<AuditService> auditService) {

    this.paymentService = paymentService;
    this.auditService = auditService;
}
```

Or:

```java
public OrderService(
        PaymentService paymentService,
        ObjectProvider<AuditService> auditServiceProvider) {
}
```

The appropriate choice depends on whether the dependency is optional, deferred, scoped, or expected to have multiple candidates.

---

# 31. Common Misconceptions

## Misconception 1

> Spring creates the object first and then calls the constructor.

Incorrect.

The constructor is part of object creation.

```text
new Car(engine)
```

creates the object and executes the constructor.

---

## Misconception 2

> Constructor injection means eager loading.

Incorrect.

Constructor injection describes **how dependencies are supplied**.

Eager/lazy initialization describes **when beans are instantiated**.

---

## Misconception 3

> `@Autowired` is always required.

Incorrect.

For a single constructor:

```java
public Car(Engine engine) {
}
```

`@Autowired` is not required in modern Spring.

---

## Misconception 4

> Constructor injection prevents all circular dependencies.

It does not prevent the design problem; it exposes constructor cycles immediately because the objects cannot be constructed.

---

## Misconception 5

> The constructor always receives the actual target object.

Not necessarily.

If Spring has proxied the dependency, the constructor can receive the **proxy/reference exposed by the container**.

---

# 32. Why Constructor Injection Is Considered Best Practice

For required dependencies:

```java
@Service
public class OrderService {

    private final OrderRepository repository;
    private final PaymentService paymentService;

    public OrderService(
            OrderRepository repository,
            PaymentService paymentService) {

        this.repository = repository;
        this.paymentService = paymentService;
    }
}
```

This gives:

* Explicit dependencies
* Immutable fields
* Fully initialized required state
* Easier unit testing
* Better separation of concerns
* Early detection of missing dependencies
* Better support for circular-dependency detection

---

# 33. Constructor Injection Can Reveal Design Problems

Suppose:

```java
public OrderService(
        A a,
        B b,
        C c,
        D d,
        E e,
        F f,
        G g,
        H h) {
}
```

Technically this can work.

But a constructor with many dependencies may indicate that the class has **too many responsibilities**.

Constructor injection therefore provides useful architectural feedback.

Instead of hiding dependencies through field injection, the constructor makes the class's dependency graph visible.

A possible redesign is:

```text
Large Service
     |
     +-- Payment responsibilities
     +-- Notification responsibilities
     +-- Order responsibilities
     +-- Reporting responsibilities
```

Split these responsibilities into focused services.

---

# 34. Constructor Injection and SOLID

Constructor injection strongly supports several SOLID principles.

### Single Responsibility Principle

Explicit dependencies make it easier to identify classes that have too many responsibilities.

### Dependency Inversion Principle

Depend on abstractions:

```java
public OrderService(PaymentService paymentService)
```

rather than:

```java
public OrderService(StripePaymentService paymentService)
```

when the abstraction is appropriate.

### Open/Closed Principle

Implementations can often be replaced without modifying the consuming class.

---

# 35. Testing Example

Production:

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    public void placeOrder() {
        paymentService.pay();
    }
}
```

Test:

```java
PaymentService paymentService =
        mock(PaymentService.class);

OrderService service =
        new OrderService(paymentService);
```

No Spring container is required.

This is one of the strongest practical benefits of constructor injection.

---

# 36. Constructor Injection with Lombok

Lombok can generate the constructor.

```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final PaymentService paymentService;
    private final OrderRepository repository;
}
```

Lombok effectively generates:

```java
public OrderService(
        PaymentService paymentService,
        OrderRepository repository) {

    this.paymentService = paymentService;
    this.repository = repository;
}
```

Spring then uses the generated single constructor.

---

# 37. What Happens Internally at a High Level?

For a bean:

```java
@Service
class OrderService {

    private final PaymentService paymentService;

    OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

A simplified conceptual flow is:

```text
BeanDefinition for OrderService
              ↓
Spring determines constructor
              ↓
Determine constructor arguments
              ↓
Resolve PaymentService bean
              ↓
Create/obtain PaymentService
              ↓
Invoke OrderService constructor
              ↓
OrderService instance created
              ↓
Apply remaining bean lifecycle processing
              ↓
Expose final bean reference
```

The actual Spring internals involve classes such as:

```text
AbstractAutowireCapableBeanFactory
ConstructorResolver
BeanPostProcessor
InstantiationAwareBeanPostProcessor
```

The exact path varies depending on bean definition, autowiring, factory methods, proxies, scopes, and other configuration.

---

# 38. Important Internal Concept: `ConstructorResolver`

When Spring needs to determine which constructor to use and resolve its arguments, Spring's bean factory uses constructor-resolution infrastructure, notably `ConstructorResolver`.

Conceptually:

```text
BeanFactory
    ↓
Determine constructors
    ↓
Resolve constructor arguments
    ↓
Select constructor
    ↓
Instantiate object
```

Do not think of this as simply:

```java
new Object()
```

Spring performs dependency resolution and bean creation around the constructor invocation.

---

# 39. XML Constructor Injection Internally

XML:

```xml
<bean id="car"
      class="com.example.Car">

    <constructor-arg ref="engine"/>
</bean>
```

Conceptually:

```text
Read BeanDefinition
        ↓
Read constructor-arg
        ↓
Resolve "engine"
        ↓
Obtain Engine bean
        ↓
Resolve Car constructor
        ↓
Invoke Car(Engine)
```

The XML is configuration metadata; it is not itself performing the object creation.

Spring's container does that.

---

# 40. Recommended Interview Explanation

For a 7–8 year experienced developer, a strong answer would be:

> **Constructor injection is a dependency injection mechanism where Spring supplies a bean's required dependencies as constructor arguments during bean instantiation. Spring first resolves the constructor and its required arguments, obtains or creates the required dependency beans, and then invokes the target constructor with those dependencies. Therefore, unlike setter or field injection, the dependency is available as part of object construction rather than being populated after the object has been instantiated.**
>
> **Constructor injection is preferred for mandatory dependencies because it makes dependencies explicit, supports immutability through `final` fields, improves unit testability, prevents partially initialized objects, and exposes circular dependencies early. Since Spring 4.3, `@Autowired` is not required when the class has a single constructor. Constructor injection itself should not be confused with eager initialization: eager/lazy behavior determines when the bean is created, while constructor injection determines how its dependencies are supplied.**

---

# 41. Quick Revision

```text
Dependency
    ↓
Object required by another object

Dependency Injection
    ↓
External source provides that dependency

Spring IoC Container
    ↓
Creates + manages beans + resolves dependencies

Constructor Injection
    ↓
Dependency supplied through constructor

Single constructor
    ↓
@Autowired usually unnecessary

Mandatory dependency
    ↓
Constructor injection preferred

Setter/Field injection
    ↓
Dependency populated after instantiation

Eager initialization
    ↓
When bean is created

Constructor injection
    ↓
How dependency is supplied

@Lazy
    ↓
Delays bean creation

Multiple implementations
    ↓
@Primary / @Qualifier

Circular constructor dependency
    ↓
Cannot normally be resolved through early singleton exposure

Prototype dependency in singleton
    ↓
Use ObjectProvider/factory/scoped approach when repeated fresh instances are required
```

# 42. Key Points to Remember

1. **DI means dependencies are supplied externally.**
2. **Spring IoC Container manages the beans.**
3. **Constructor injection supplies dependencies during object construction.**
4. The dependency must be resolved before Spring can invoke the target constructor.
5. The target object does not exist as a fully constructed object before its constructor executes.
6. Constructor injection is **not the same thing as eager initialization**.
7. Singleton beans are eagerly initialized by default unless configured otherwise.
8. `@Lazy` changes when a bean is created, not how constructor injection works.
9. A single constructor does not require `@Autowired` in modern Spring.
10. Multiple constructors may require explicit selection.
11. Constructor injection supports `final` fields and immutability.
12. Constructor injection makes dependencies explicit.
13. Constructor injection is highly testable without Spring.
14. Constructor cycles generally fail because neither object can be constructed first.
15. `@Qualifier` and `@Primary` resolve multiple candidates.
16. Constructor injection occurs during the **instantiation phase** of the Spring bean lifecycle.
17. Proxies can mean that another bean receives a proxy/reference rather than the raw target instance.
18. Large constructors can reveal excessive class responsibilities and potential design problems.
19. XML uses `<constructor-arg>` for constructor-based wiring.
20. For modern Spring applications, **constructor injection should generally be the default choice for required dependencies**.


# Setter-Based Dependency Injection in Spring

## 1. What is Setter-Based Dependency Injection?

Setter-based Dependency Injection is a form of Dependency Injection where Spring provides a bean's dependency by calling a **setter method** after the object has been instantiated.

In simple terms:

```text
1. Spring creates the object
2. Spring finds the setter method
3. Spring obtains the required dependency
4. Spring calls the setter
5. Bean becomes ready for use
```

Example:

```java
@Component
public class Car {

    private Engine engine;

    @Autowired
    public void setEngine(Engine engine) {
        this.engine = engine;
    }
}
```

Spring conceptually performs:

```java
Car car = new Car();

Engine engine = ...; // obtain from Spring container

car.setEngine(engine);
```

---

# 2. Basic Example

## Dependency

```java
@Component
public class Engine {

    public void start() {
        System.out.println("Engine started");
    }
}
```

## Dependent Bean

```java
@Component
public class Car {

    private Engine engine;

    @Autowired
    public void setEngine(Engine engine) {
        this.engine = engine;
    }

    public void drive() {
        engine.start();
        System.out.println("Car is moving");
    }
}
```

## Using the Bean

```java
ApplicationContext context =
        new AnnotationConfigApplicationContext(AppConfig.class);

Car car = context.getBean(Car.class);

car.drive();
```

Spring creates the `Car` object and then injects the `Engine` through `setEngine()`.

---

# 3. How Setter Injection Works Internally

Consider:

```java
@Component
public class Car {

    private Engine engine;

    @Autowired
    public void setEngine(Engine engine) {
        this.engine = engine;
    }
}
```

Conceptually, Spring performs:

```java
Car car = new Car();

Engine engine = applicationContext.getBean(Engine.class);

car.setEngine(engine);
```

The actual Spring implementation involves bean creation, dependency resolution, post-processors, lifecycle callbacks, and other container infrastructure. The above code is only a simplified conceptual representation.

---

# 4. Lifecycle Position

For a normal singleton bean, the simplified sequence is:

```text
Create bean instance
       ↓
Populate properties / inject dependencies
       ↓
Aware callbacks
       ↓
BeanPostProcessor - before initialization
       ↓
@PostConstruct / InitializingBean / custom init
       ↓
BeanPostProcessor - after initialization
       ↓
Bean ready
```

Setter injection happens during the **dependency population phase**, before initialization callbacks such as `@PostConstruct`.

Example:

```java
@Component
public class Car {

    private Engine engine;

    @Autowired
    public void setEngine(Engine engine) {
        System.out.println("Setter called");
        this.engine = engine;
    }

    @PostConstruct
    public void init() {
        System.out.println(engine != null);
    }
}
```

The setter is called before `init()`.

---

# 5. Setter Injection with @Autowired

The most common annotation-based form is:

```java
@Component
public class Car {

    private Engine engine;

    @Autowired
    public void setEngine(Engine engine) {
        this.engine = engine;
    }
}
```

Spring looks for an `Engine` bean and injects it into the setter.

---

# 6. Setter Method Does Not Have to Be Named setEngine()

Spring can inject dependencies into an arbitrary method when it is annotated with `@Autowired`.

```java
@Component
public class Car {

    private Engine engine;

    @Autowired
    public void configureEngine(Engine engine) {
        this.engine = engine;
    }
}
```

Although this is technically method injection through `@Autowired`, it is commonly discussed together with setter-style injection.

A conventional setter is clearer:

```java
public void setEngine(Engine engine) {
    this.engine = engine;
}
```

---

# 7. Multiple Dependencies

A class can have multiple setter-injected dependencies.

```java
@Component
public class Car {

    private Engine engine;
    private Transmission transmission;
    private GPS gps;

    @Autowired
    public void setEngine(Engine engine) {
        this.engine = engine;
    }

    @Autowired
    public void setTransmission(Transmission transmission) {
        this.transmission = transmission;
    }

    @Autowired
    public void setGps(GPS gps) {
        this.gps = gps;
    }
}
```

Spring resolves and injects each dependency.

---

# 8. XML-Based Setter Injection

Setter injection existed before annotation-based configuration.

## Java Classes

```java
public class Engine {

    public void start() {
        System.out.println("Engine started");
    }
}
```

```java
public class Car {

    private Engine engine;

    public void setEngine(Engine engine) {
        this.engine = engine;
    }
}
```

## XML Configuration

```xml
<beans>

    <bean id="engine"
          class="com.example.Engine"/>

    <bean id="car"
          class="com.example.Car">

        <property name="engine" ref="engine"/>

    </bean>

</beans>
```

The important part is:

```xml
<property name="engine" ref="engine"/>
```

It tells Spring to inject the `engine` bean through:

```java
car.setEngine(engine);
```

---

# 9. Setter Injection with Java Configuration

You can also explicitly configure the bean using `@Bean`.

```java
@Configuration
public class AppConfig {

    @Bean
    public Engine engine() {
        return new Engine();
    }

    @Bean
    public Car car() {
        Car car = new Car();
        car.setEngine(engine());
        return car;
    }
}
```

Here Spring uses the setter because the configuration explicitly calls:

```java
car.setEngine(engine());
```

Note that this is Java-based bean configuration using a setter; it does not require `@Autowired` on the setter.

---

# 10. Optional Dependency

One important use case for setter injection is an optional dependency.

```java
@Component
public class Car {

    private GPS gps;

    @Autowired(required = false)
    public void setGps(GPS gps) {
        this.gps = gps;
    }
}
```

If a `GPS` bean exists:

```text
GPS bean
   ↓
setGps(gps)
```

If no `GPS` bean exists, Spring can continue without injecting it.

For modern Spring code, other approaches such as `Optional<T>`, `ObjectProvider<T>`, or `@Nullable` can also be appropriate depending on the requirement.

---

# 11. Setter Injection with Optional<T>

```java
@Component
public class Car {

    private GPS gps;

    @Autowired
    public void setGps(Optional<GPS> gps) {
        this.gps = gps.orElse(null);
    }
}
```

If a `GPS` bean exists, the `Optional` contains it.

If it does not exist, the `Optional` is empty.

A cleaner design may be:

```java
@Component
public class Car {

    private Optional<GPS> gps = Optional.empty();

    @Autowired
    public void setGps(Optional<GPS> gps) {
        this.gps = gps;
    }
}
```

---

# 12. Setter Injection with @Nullable

```java
@Component
public class Car {

    private GPS gps;

    @Autowired
    public void setGps(@Nullable GPS gps) {
        this.gps = gps;
    }
}
```

Spring can pass `null` when no matching bean is available.

---

# 13. Setter Injection with ObjectProvider

`ObjectProvider` is useful when the dependency should be resolved dynamically or may not exist.

```java
@Component
public class Car {

    private ObjectProvider<GPS> gpsProvider;

    @Autowired
    public void setGpsProvider(ObjectProvider<GPS> gpsProvider) {
        this.gpsProvider = gpsProvider;
    }

    public void useGps() {
        GPS gps = gpsProvider.getIfAvailable();

        if (gps != null) {
            gps.locate();
        }
    }
}
```

This can be useful when you do not want to require a `GPS` bean during normal bean creation.

---

# 14. Setter Injection with Multiple Implementations

Suppose:

```java
public interface PaymentService {
    void pay();
}
```

Two implementations:

```java
@Component
public class CardPaymentService implements PaymentService {

    public void pay() {
        System.out.println("Card payment");
    }
}
```

```java
@Component
public class UpiPaymentService implements PaymentService {

    public void pay() {
        System.out.println("UPI payment");
    }
}
```

If you write:

```java
@Autowired
public void setPaymentService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

Spring finds multiple candidates and cannot decide which one to inject.

This can result in:

```text
NoUniqueBeanDefinitionException
```

---

# 15. Solving Multiple Bean Problem with @Qualifier

```java
@Component
public class OrderService {

    private PaymentService paymentService;

    @Autowired
    public void setPaymentService(
            @Qualifier("upiPaymentService")
            PaymentService paymentService) {

        this.paymentService = paymentService;
    }
}
```

Now Spring knows which implementation to use.

```text
PaymentService
      │
      ├── CardPaymentService
      │
      └── UpiPaymentService
              ↑
          selected
```

---

# 16. Solving Multiple Bean Problem with @Primary

```java
@Component
@Primary
public class UpiPaymentService implements PaymentService {

    public void pay() {
        System.out.println("UPI payment");
    }
}
```

Then:

```java
@Autowired
public void setPaymentService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

Spring selects the `@Primary` bean when multiple candidates exist.

---

# 17. Setter Injection and Immutability

Setter injection normally requires a mutable field:

```java
private Engine engine;

public void setEngine(Engine engine) {
    this.engine = engine;
}
```

You cannot normally do:

```java
private final Engine engine;

public void setEngine(Engine engine) {
    this.engine = engine; // invalid if final was already initialized
}
```

This is one reason constructor injection is preferred for mandatory dependencies.

Constructor injection allows:

```java
@Component
public class Car {

    private final Engine engine;

    public Car(Engine engine) {
        this.engine = engine;
    }
}
```

The dependency cannot be changed after construction.

---

# 18. Setter Injection Allows Dependency Replacement

Because the setter can be called again:

```java
car.setEngine(engine1);

car.setEngine(engine2);
```

the dependency can be replaced.

This can be useful in certain configuration or reconfiguration scenarios, but it also means the object is mutable.

For most mandatory application dependencies, replacing dependencies after construction is usually not desirable.

---

# 19. Setter Injection and Bean State

One disadvantage is that the object can temporarily exist without its dependency.

```java
Car car = new Car();
```

At this point:

```java
car.engine == null
```

After Spring performs injection:

```java
car.setEngine(engine);
```

Now:

```java
car.engine != null
```

Therefore:

```text
Constructor Injection

Create object
     ↓
Fully initialized object
```

Whereas:

```text
Setter Injection

Create object
     ↓
Partially initialized object
     ↓
Inject dependency
     ↓
Ready object
```

Spring completes this injection before the bean is normally exposed for use, but the distinction matters for object design.

---

# 20. Setter Injection and Circular Dependency

Setter injection can allow certain circular dependencies to be resolved because Spring can instantiate the objects first and inject references afterward.

Example:

```java
@Component
public class A {

    private B b;

    @Autowired
    public void setB(B b) {
        this.b = b;
    }
}
```

```java
@Component
public class B {

    private A a;

    @Autowired
    public void setA(A a) {
        this.a = a;
    }
}
```

Conceptually:

```text
Create A
   ↓
Create B
   ↓
Inject B into A
   ↓
Inject A into B
```

Spring's singleton creation mechanism and early references can make some setter-based circular dependencies resolvable.

However, **using setter injection to solve a circular dependency is not the recommended design**.

The preferred solution is usually to redesign the dependency relationship.

---

# 21. Setter Injection vs Constructor Injection

| Feature | Setter Injection | Constructor Injection |
|---|---|---|
| Object creation | First | Dependencies supplied during creation |
| Dependency injection | After construction | During construction |
| Mandatory dependency | Less suitable | Best choice |
| Optional dependency | Good choice | Possible but less convenient |
| Immutability | No | Yes |
| `final` dependency | Not suitable | Excellent |
| Circular dependency | Some setter cycles can be resolved | Constructor cycle fails |
| Object can be incomplete | Temporarily | No |
| Dependency replacement | Possible | Not normally possible |
| Recommended use | Optional/configurable dependencies | Required dependencies |

---

# 22. Common Mistake: Calling Business Methods Before Injection

Avoid designing code that assumes the dependency exists inside the constructor:

```java
@Component
public class Car {

    private Engine engine;

    public Car() {
        engine.start(); // WRONG
    }

    @Autowired
    public void setEngine(Engine engine) {
        this.engine = engine;
    }
}
```

The setter has not been called when the constructor executes.

Therefore `engine` is still `null`.

Correct:

```java
@Component
public class Car {

    private Engine engine;

    @Autowired
    public void setEngine(Engine engine) {
        this.engine = engine;
    }

    @PostConstruct
    public void init() {
        engine.start();
    }
}
```

The injection has happened before `@PostConstruct`.

---

# 23. Is @Autowired Required on a Setter?

For annotation-based autowiring, yes, you normally use `@Autowired` to tell Spring that the setter should be used for dependency injection.

```java
@Autowired
public void setEngine(Engine engine) {
    this.engine = engine;
}
```

However, with XML configuration:

```xml
<property name="engine" ref="engine"/>
```

Spring knows to call the setter from the XML configuration, so `@Autowired` is not required.

Similarly, with Java configuration:

```java
@Bean
public Car car(Engine engine) {
    Car car = new Car();
    car.setEngine(engine);
    return car;
}
```

the setter is explicitly called by the configuration code.

---

# 24. Setter Injection with Inheritance

Setter injection can also be used with inherited setters.

```java
public class Vehicle {

    protected Engine engine;

    public void setEngine(Engine engine) {
        this.engine = engine;
    }
}
```

```java
@Component
public class Car extends Vehicle {
}
```

Spring can discover and use the setter as part of bean configuration.

---

# 25. Best Practice

Use setter injection primarily when the dependency is:

- Optional
- Configurable
- Reasonably safe to change after construction

Example:

```java
@Component
public class ReportService {

    private NotificationService notificationService;

    @Autowired
    public void setNotificationService(
            NotificationService notificationService) {

        this.notificationService = notificationService;
    }
}
```

For mandatory dependencies, prefer constructor injection:

```java
@Component
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

---

# 26. Important Interview Points

### What is setter-based dependency injection?

Spring injects a dependency by calling a setter method after the bean has been instantiated.

### When does setter injection happen?

During the bean's dependency population phase, after instantiation and before initialization callbacks such as `@PostConstruct`.

### Why use setter injection?

Primarily for optional or configurable dependencies.

### Can setter injection support optional dependencies?

Yes.

Common approaches include:

```java
@Autowired(required = false)
```

or:

```java
Optional<T>
```

or:

```java
@Nullable
```

or:

```java
ObjectProvider<T>
```

### Can setter injection create circular dependencies?

Some setter-based circular dependencies between singleton beans can be resolved by Spring's singleton creation and early-reference mechanisms, but circular dependencies should generally be redesigned rather than intentionally introduced.

### Main disadvantage?

The object can be created before all dependencies have been injected, and the dependency is mutable.

### Is setter injection preferred over constructor injection?

No. For mandatory dependencies, constructor injection is generally preferred.

---

# 27. Complete Example

## Engine

```java
@Component
public class Engine {

    public void start() {
        System.out.println("Engine started");
    }
}
```

## GPS

```java
@Component
public class GPS {

    public void locate() {
        System.out.println("Location found");
    }
}
```

## Car

```java
@Component
public class Car {

    private Engine engine;
    private GPS gps;

    @Autowired
    public void setEngine(Engine engine) {
        this.engine = engine;
    }

    @Autowired(required = false)
    public void setGps(GPS gps) {
        this.gps = gps;
    }

    @PostConstruct
    public void init() {
        System.out.println("Car initialized");
    }

    public void drive() {

        engine.start();

        if (gps != null) {
            gps.locate();
        }

        System.out.println("Car is driving");
    }
}
```

## Configuration

```java
@Configuration
@ComponentScan("com.example")
public class AppConfig {
}
```

## Main Class

```java
public class Main {

    public static void main(String[] args) {

        ApplicationContext context =
                new AnnotationConfigApplicationContext(AppConfig.class);

        Car car = context.getBean(Car.class);

        car.drive();
    }
}
```

Flow:

```text
Spring starts
     ↓
Finds Engine
     ↓
Finds GPS
     ↓
Creates Car
     ↓
setEngine(engine)
     ↓
setGps(gps)
     ↓
@PostConstruct
     ↓
Car ready
     ↓
car.drive()
```

---

# 28. Final Summary

```text
Setter-Based DI
       │
       ├── Object created first
       │
       ├── Dependency resolved
       │
       ├── Setter called
       │
       └── Bean becomes ready
```

Example:

```java
@Component
public class Car {

    private Engine engine;

    @Autowired
    public void setEngine(Engine engine) {
        this.engine = engine;
    }
}
```

Remember the key point:

> **Setter injection = Spring creates the object first and then injects the dependency by calling a setter method.**

For a 6–7+ year experienced Spring developer, know especially:

1. Setter injection lifecycle position
2. `@Autowired` setter
3. Optional dependencies
4. `Optional<T>`, `@Nullable`, `ObjectProvider<T>`
5. Multiple beans with `@Qualifier` / `@Primary`
6. Setter vs constructor injection
7. Circular dependency and early singleton exposure
8. Why constructor injection is preferred for mandatory dependencies
9. XML `<property>` setter injection
10. Java configuration using setters

# Field Injection in Spring

## 1. What is Field Injection?

Field Injection is a Dependency Injection technique where Spring injects a dependency directly into a class field using `@Autowired`.

The dependency does not need to be passed through a constructor or setter method.

### Basic Example

```java
@Component
public class PetrolEngine {

    public void start() {
        System.out.println("Petrol Engine Started");
    }
}
```

```java
@Component
public class Car {

    @Autowired
    private PetrolEngine engine;

    public void drive() {
        engine.start();
        System.out.println("Car is moving");
    }
}
```

Here:

- `PetrolEngine` is a Spring bean.
- `Car` is a Spring bean.
- `engine` is the dependency.
- `@Autowired` tells Spring to inject the `PetrolEngine` bean into the `engine` field.

---

## 2. How Field Injection Works

The basic flow is:

```text
Application Starts
       |
       v
Component Scanning
       |
       v
Spring finds Car and PetrolEngine
       |
       v
Creates PetrolEngine bean
       |
       v
Creates Car object
       |
       v
engine is initially null
       |
       v
Spring performs dependency injection
       |
       v
engine points to PetrolEngine bean
       |
       v
Car bean is ready
```

### Object creation

Conceptually, Spring first creates the object:

```java
Car car = new Car();
```

At this point:

```java
car.engine == null
```

Spring then injects the dependency.

Conceptually, Spring performs an operation similar to:

```java
Field field = Car.class.getDeclaredField("engine");

field.setAccessible(true);

field.set(car, petrolEngine);
```

After injection:

```text
Car object
    |
    +---- engine ----> PetrolEngine object
```

Spring internally uses reflection to access and set the field.

---

## 3. Why Reflection is Used

Consider:

```java
@Component
public class Car {

    @Autowired
    private PetrolEngine engine;
}
```

The field is:

```java
private PetrolEngine engine;
```

The field is private, so external code cannot directly access it.

Spring can use reflection to access the field and assign the dependency.

Conceptually:

```java
Field field = Car.class.getDeclaredField("engine");
field.setAccessible(true);
field.set(car, petrolEngine);
```

This is one reason field injection is different from constructor injection.

---

## 4. Complete Example

### PetrolEngine

```java
@Component
public class PetrolEngine {

    public void start() {
        System.out.println("Petrol engine started");
    }
}
```

### Car

```java
@Component
public class Car {

    @Autowired
    private PetrolEngine engine;

    public void drive() {
        engine.start();
        System.out.println("Car is moving");
    }
}
```

### Main Application

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {

        ApplicationContext context =
                SpringApplication.run(Application.class, args);

        Car car = context.getBean(Car.class);

        car.drive();
    }
}
```

Output:

```text
Petrol engine started
Car is moving
```

---

## 5. Field Injection with Interface

A common real-world scenario is injecting an implementation through an interface.

### Engine Interface

```java
public interface Engine {

    void start();
}
```

### PetrolEngine

```java
@Component
public class PetrolEngine implements Engine {

    @Override
    public void start() {
        System.out.println("Petrol engine started");
    }
}
```

### Car

```java
@Component
public class Car {

    @Autowired
    private Engine engine;

    public void drive() {
        engine.start();
    }
}
```

Spring searches for a bean implementing:

```java
Engine
```

If there is exactly one implementation, Spring injects it.

---

# 6. Multiple Implementations

Suppose there are two implementations.

```java
@Component
public class PetrolEngine implements Engine {

    @Override
    public void start() {
        System.out.println("Petrol engine started");
    }
}
```

```java
@Component
public class DieselEngine implements Engine {

    @Override
    public void start() {
        System.out.println("Diesel engine started");
    }
}
```

And:

```java
@Component
public class Car {

    @Autowired
    private Engine engine;
}
```

Now Spring finds:

```text
Engine
  |
  +-- PetrolEngine
  |
  +-- DieselEngine
```

Spring cannot determine which implementation should be injected.

This results in:

```text
NoUniqueBeanDefinitionException
```

---

# 7. Solving Multiple Beans with @Qualifier

Use `@Qualifier` when you want to explicitly select a bean.

```java
@Component
public class Car {

    @Autowired
    @Qualifier("petrolEngine")
    private Engine engine;
}
```

Spring injects:

```text
Car
 |
 +-- engine --> PetrolEngine
```

The default bean name for:

```java
@Component
public class PetrolEngine
```

is normally:

```text
petrolEngine
```

You can also explicitly specify the bean name:

```java
@Component("petrol")
public class PetrolEngine implements Engine {
}
```

Then:

```java
@Autowired
@Qualifier("petrol")
private Engine engine;
```

---

# 8. Solving Multiple Beans with @Primary

Another option is `@Primary`.

```java
@Component
@Primary
public class PetrolEngine implements Engine {

    @Override
    public void start() {
        System.out.println("Petrol engine started");
    }
}
```

Now:

```java
@Component
public class Car {

    @Autowired
    private Engine engine;
}
```

Spring chooses the primary bean:

```text
Car
 |
 +-- engine --> PetrolEngine
```

---

# 9. Optional Field Injection

By default, if Spring cannot find the required dependency, bean creation fails.

You can make field injection optional:

```java
@Autowired(required = false)
private PetrolEngine engine;
```

If no `PetrolEngine` bean exists, Spring does not fail because of this injection point.

The field can remain:

```java
engine == null
```

Therefore, code using an optional dependency must handle the possibility of `null`.

Example:

```java
public void drive() {

    if (engine != null) {
        engine.start();
    }
}
```

---

# 10. Field Injection with @Resource

Field injection is not limited to `@Autowired`.

`@Resource` can also be used.

```java
@Component
public class Car {

    @Resource
    private PetrolEngine engine;
}
```

`@Resource` is from:

```java
import jakarta.annotation.Resource;
```

The exact resolution rules differ from `@Autowired`, so do not treat `@Resource` and `@Autowired` as completely identical.

For Spring-specific dependency injection, `@Autowired` is commonly used.

---

# 11. Field Injection with @Inject

JSR-330/Jakarta `@Inject` can also be used.

```java
@Component
public class Car {

    @Inject
    private PetrolEngine engine;
}
```

Import depends on the API used by the application.

Conceptually:

```text
@Autowired
@Inject
@Resource
    |
    v
Can be used for field-based dependency injection
```

Their bean-resolution semantics are not identical.

---

# 12. Field Injection and Bean Lifecycle

Field injection happens after the bean instance has been created.

A simplified lifecycle is:

```text
1. Instantiate bean
       |
       v
2. Populate dependencies
       |
       v
3. Aware callbacks
       |
       v
4. BeanPostProcessor before initialization
       |
       v
5. Initialization callbacks
       |
       v
6. BeanPostProcessor after initialization
       |
       v
7. Bean ready for use
```

For field injection, dependency population occurs during the bean creation process before initialization callbacks complete.

Therefore, when initialization callbacks such as `@PostConstruct` run, injected fields are normally already populated.

Example:

```java
@Component
public class Car {

    @Autowired
    private PetrolEngine engine;

    @PostConstruct
    public void init() {
        engine.start();
    }
}
```

The `engine` field has been injected before `@PostConstruct` executes.

---

# 13. Field Injection and @PostConstruct

Example:

```java
@Component
public class OrderService {

    @Autowired
    private PaymentService paymentService;

    @PostConstruct
    public void init() {

        System.out.println("PaymentService = " + paymentService);
    }
}
```

The dependency is available inside `@PostConstruct`.

Conceptually:

```text
Create OrderService
       |
       v
Inject PaymentService
       |
       v
@PostConstruct
       |
       v
Bean ready
```

---

# 14. Field Injection and Constructor

With field injection:

```java
@Component
public class Car {

    @Autowired
    private PetrolEngine engine;
}
```

Spring creates the object using the no-argument constructor:

```java
Car car = new Car();
```

Then it injects the field.

This is different from constructor injection:

```java
@Component
public class Car {

    private final PetrolEngine engine;

    public Car(PetrolEngine engine) {
        this.engine = engine;
    }
}
```

Constructor injection supplies the dependency during object construction.

---

# 15. Field Injection Cannot Normally Use final Fields

This is not the normal pattern:

```java
@Autowired
private final PetrolEngine engine;
```

The problem is that a `final` field must be initialized during construction.

Field injection happens after construction.

Constructor injection solves this:

```java
@Component
public class Car {

    private final PetrolEngine engine;

    public Car(PetrolEngine engine) {
        this.engine = engine;
    }
}
```

Now:

```java
engine
```

can be `final`.

This is one of the important reasons constructor injection is preferred.

---

# 16. Field Injection and Unit Testing

Field injection makes plain unit testing less convenient.

Example:

```java
@Component
public class Car {

    @Autowired
    private PetrolEngine engine;

    public void drive() {
        engine.start();
    }
}
```

If you create the object yourself:

```java
Car car = new Car();
```

Spring is not involved.

Therefore:

```java
car.drive();
```

can result in:

```text
NullPointerException
```

because:

```java
engine == null
```

---

# 17. Field Injection Test with Reflection

A test can manually inject the dependency using reflection.

For example:

```java
@Test
void shouldDrive() {

    Car car = new Car();

    PetrolEngine engine = new PetrolEngine();

    ReflectionTestUtils.setField(car, "engine", engine);

    car.drive();
}
```

`ReflectionTestUtils` is provided by Spring's testing support.

However, needing reflection for a simple unit test is one of the disadvantages of field injection.

---

# 18. Constructor Injection is Easier to Test

Field injection:

```java
Car car = new Car();

ReflectionTestUtils.setField(car, "engine", engine);
```

Constructor injection:

```java
Car car = new Car(engine);
```

This is much simpler.

For example:

```java
Engine mockEngine = mock(Engine.class);

Car car = new Car(mockEngine);

car.drive();
```

No Spring context is required.

---

# 19. Field Injection Hides Dependencies

Consider:

```java
@Component
public class OrderService {

    @Autowired
    private PaymentService paymentService;

    @Autowired
    private InventoryService inventoryService;

    @Autowired
    private NotificationService notificationService;
}
```

The class has three dependencies, but someone looking only at the constructor sees:

```java
public OrderService() {
}
```

The dependencies are hidden inside the fields.

With constructor injection:

```java
@Component
public class OrderService {

    private final PaymentService paymentService;
    private final InventoryService inventoryService;
    private final NotificationService notificationService;

    public OrderService(
            PaymentService paymentService,
            InventoryService inventoryService,
            NotificationService notificationService) {

        this.paymentService = paymentService;
        this.inventoryService = inventoryService;
        this.notificationService = notificationService;
    }
}
```

The dependencies are immediately visible.

---

# 20. Field Injection and Immutability

Field injection:

```java
@Autowired
private PaymentService paymentService;
```

The field is not `final`.

Constructor injection:

```java
private final PaymentService paymentService;

public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

The dependency cannot normally be reassigned after construction.

This makes constructor injection better suited to immutable design.

---

# 21. Field Injection in Spring Boot

A typical Spring Boot example:

```java
@Service
public class OrderService {

    @Autowired
    private OrderRepository orderRepository;

    public Order findOrder(Long id) {
        return orderRepository.findById(id)
                .orElseThrow();
    }
}
```

Here:

```text
OrderService
      |
      +---- orderRepository
                    |
                    v
             Spring Data Repository
```

Spring injects the repository implementation into the field.

---

# 22. Field Injection with Repository

Example:

```java
@Repository
public interface UserRepository
        extends JpaRepository<User, Long> {
}
```

Service:

```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    public User getUser(Long id) {

        return userRepository.findById(id)
                .orElseThrow();
    }
}
```

The repository is injected directly into the field.

---

# 23. Field Injection with Service

Example:

```java
@Service
public class PaymentService {

    public void processPayment() {
        System.out.println("Payment processed");
    }
}
```

```java
@Service
public class OrderService {

    @Autowired
    private PaymentService paymentService;

    public void placeOrder() {
        paymentService.processPayment();
    }
}
```

---

# 24. Field Injection with Multiple Dependencies

```java
@Service
public class OrderService {

    @Autowired
    private PaymentService paymentService;

    @Autowired
    private InventoryService inventoryService;

    @Autowired
    private NotificationService notificationService;

    public void placeOrder() {

        inventoryService.reserve();

        paymentService.processPayment();

        notificationService.sendConfirmation();
    }
}
```

Spring injects all three dependencies.

Conceptually:

```text
                  Spring Container
                         |
          +--------------+--------------+
          |              |              |
          v              v              v
   PaymentService  InventoryService  NotificationService
          |              |              |
          +--------------+--------------+
                         |
                         v
                    OrderService
```

---

# 25. Field Injection and Circular Dependency

Field injection can sometimes allow Spring to resolve certain circular dependencies that constructor injection cannot.

Example:

```java
@Component
public class A {

    @Autowired
    private B b;
}
```

```java
@Component
public class B {

    @Autowired
    private A a;
}
```

Spring can create the instances first and then populate their fields.

Conceptually:

```text
Create A
Create B
  |
  v
Inject B into A
Inject A into B
```

This is related to Spring's singleton early-reference/three-level-cache mechanism.

However, using field injection to make circular dependencies work is **not a good design solution**.

The preferred solution is usually to redesign the dependency relationship.

---

# 26. Field Injection with @Lazy

You can combine field injection with `@Lazy`.

```java
@Component
public class Car {

    @Autowired
    @Lazy
    private PetrolEngine engine;
}
```

`@Lazy` can cause Spring to inject/use a lazy proxy rather than eagerly creating the target dependency.

This can be useful for:

- Breaking certain initialization cycles
- Delaying expensive bean creation
- Controlling initialization timing

However, `@Lazy` should not be used as a substitute for fixing poor dependency design.

---

# 27. Field Injection Outside Spring

This is important.

Field injection only happens when Spring manages the object.

This does NOT perform dependency injection:

```java
Car car = new Car();
```

because Spring did not create/manage the object.

For example:

```java
public class SomeClass {

    public void test() {

        Car car = new Car();

        car.drive();
    }
}
```

The `engine` field remains `null`.

Spring injection works when:

```java
Car car = applicationContext.getBean(Car.class);
```

or when Spring creates the bean itself.

---

# 28. Common Mistake

Do not expect `@Autowired` to work on arbitrary objects.

This:

```java
public class Car {

    @Autowired
    private PetrolEngine engine;
}
```

does not mean every `new Car()` automatically receives an engine.

This:

```java
Car car = new Car();
```

is a normal Java object.

Spring does not automatically know about it.

Better:

```java
@Component
public class Car {
}
```

and let Spring create it.

---

# 29. Field Injection vs Setter Injection

### Field Injection

```java
@Component
public class Car {

    @Autowired
    private PetrolEngine engine;
}
```

### Setter Injection

```java
@Component
public class Car {

    private PetrolEngine engine;

    @Autowired
    public void setEngine(PetrolEngine engine) {
        this.engine = engine;
    }
}
```

Main difference:

```text
Field Injection
      |
      +-- Spring directly modifies field

Setter Injection
      |
      +-- Spring calls setter method
```

Setter injection can be useful when a dependency is optional or intentionally changeable.

---

# 30. Field Injection vs Constructor Injection

### Field Injection

```java
@Component
public class Car {

    @Autowired
    private PetrolEngine engine;
}
```

### Constructor Injection

```java
@Component
public class Car {

    private final PetrolEngine engine;

    public Car(PetrolEngine engine) {
        this.engine = engine;
    }
}
```

Comparison:

| Feature | Field Injection | Constructor Injection |
|---|---|---|
| Dependency supplied through constructor | No | Yes |
| Uses reflection for injection | Yes | No |
| Dependencies visible in constructor | No | Yes |
| Supports `final` dependency | No | Yes |
| Easy unit testing | Less convenient | Easy |
| Object can be created with all required dependencies | No | Yes |
| Generally recommended | No | Yes |

---

# 31. Why Constructor Injection is Preferred

For required dependencies, constructor injection is generally preferred because:

1. Dependencies are explicit.
2. Dependencies can be `final`.
3. Unit testing is easier.
4. The object cannot be created without required dependencies.
5. It encourages better class design.
6. Circular dependencies become easier to detect.
7. The class has a clear construction contract.

Example:

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

This communicates:

> `OrderService` cannot properly operate without `PaymentService`.

---

# 32. Is Field Injection Deprecated?

No.

Spring still supports field injection.

However, field injection is generally **not the preferred style for required dependencies** in modern Spring applications.

Preferred:

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Instead of:

```java
@Service
public class OrderService {

    @Autowired
    private PaymentService paymentService;
}
```

---

# 33. When Field Injection Can Be Seen

You may encounter field injection in:

- Older Spring applications
- Legacy codebases
- Existing enterprise applications
- Small examples
- Prototypes
- Tests or framework-managed components where direct field injection is convenient

When maintaining an existing project, you should understand it even if you choose constructor injection for new code.

---

# 34. Important Interview Questions

## Q1. What is field injection?

Field injection is a dependency injection technique where Spring injects a dependency directly into a class field, commonly using `@Autowired`.

---

## Q2. How does Spring perform field injection?

Spring creates the bean and then uses its dependency-injection infrastructure, including reflection-based field access, to populate the field.

---

## Q3. When does field injection happen?

It happens during bean population, after the bean instance has been instantiated and before initialization callbacks such as `@PostConstruct` complete.

---

## Q4. Why is field injection not preferred?

Main reasons:

- Dependencies are hidden.
- Unit testing is less convenient.
- Required fields cannot naturally be `final`.
- The class can be instantiated without its required dependencies.
- It can encourage poor dependency design.

---

## Q5. Can field injection work with private fields?

Yes.

```java
@Autowired
private PaymentService paymentService;
```

Spring can access the field through reflection.

---

## Q6. Can field injection work with interfaces?

Yes.

```java
@Autowired
private PaymentService paymentService;
```

Spring resolves an appropriate bean implementing/providing that type.

---

## Q7. What happens when multiple beans match?

Spring may throw:

```text
NoUniqueBeanDefinitionException
```

Use:

```java
@Qualifier
```

or:

```java
@Primary
```

to resolve the ambiguity.

---

## Q8. Does `new` trigger field injection?

No.

```java
Car car = new Car();
```

does not cause Spring to inject dependencies.

The object needs to be managed by Spring for normal container-based injection.

---

## Q9. Can field injection be optional?

Yes:

```java
@Autowired(required = false)
private PaymentService paymentService;
```

But the application code must handle the possibility that the field remains `null`.

---

## Q10. Which injection type is preferred?

For required dependencies:

```text
Constructor Injection
        ^
        |
    Preferred
```

Field injection is supported but generally not recommended for new production code.

---

# 35. Final Example

### Dependency

```java
@Component
public class PaymentService {

    public void pay() {
        System.out.println("Payment successful");
    }
}
```

### Consumer

```java
@Service
public class OrderService {

    @Autowired
    private PaymentService paymentService;

    public void placeOrder() {

        System.out.println("Placing order...");

        paymentService.pay();

        System.out.println("Order placed");
    }
}
```

### Main

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {

        ApplicationContext context =
                SpringApplication.run(Application.class, args);

        OrderService orderService =
                context.getBean(OrderService.class);

        orderService.placeOrder();
    }
}
```

Output:

```text
Placing order...
Payment successful
Order placed
```

---

# 36. Key Points to Remember

```text
Field Injection
      |
      +-- Dependency injected directly into a field
      |
      +-- Usually uses @Autowired
      |
      +-- Happens after object instantiation
      |
      +-- Spring uses reflection-based field access
      |
      +-- Private fields can be injected
      |
      +-- Requires Spring-managed object
      |
      +-- Cannot naturally enforce final dependencies
      |
      +-- Makes dependencies less visible
      |
      +-- Unit testing is less convenient
      |
      +-- Can participate in certain circular dependencies
      |
      +-- Still supported by Spring
      |
      +-- Constructor injection is generally preferred
```

## Interview Summary

> **Field injection is a Spring dependency injection technique where Spring injects a dependency directly into a class field, typically using `@Autowired`. The bean is first instantiated, then Spring populates the field during dependency population. Although simple and supported, field injection is generally discouraged for required dependencies because it hides dependencies, makes unit testing less convenient, prevents natural use of `final` fields, and allows objects to be created without their required dependencies. Constructor injection is generally preferred.**
