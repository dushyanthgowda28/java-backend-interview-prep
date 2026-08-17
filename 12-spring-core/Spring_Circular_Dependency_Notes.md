# Spring Circular Dependency --- Detailed Interview Notes

## 1. Definition

A **circular dependency** occurs when Spring Bean A depends on Bean B,
while Bean B depends directly or indirectly on Bean A.

``` text
A → B
↑   ↓
└───┘
```

A cycle can contain two beans or many beans.

``` text
A → B → C → A
```

### Key point

> Circular dependency is a dependency-graph problem. Spring encounters
> it while trying to create and initialize the beans.

------------------------------------------------------------------------

# 2. Why Circular Dependency Happens

Common causes:

1.  Two services directly depend on each other.
2.  Multiple services form an indirect cycle.
3.  Responsibilities are not properly separated.
4.  A service performs too many business responsibilities.
5.  Bidirectional service-to-service communication is used
    unnecessarily.
6.  A lower-level component depends on a higher-level component.
7.  Event/listener or callback design creates a reverse dependency.
8.  AOP/proxy-based dependencies make an existing cycle harder to
    resolve.

------------------------------------------------------------------------

# 3. Direct Circular Dependency

## Example

``` java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

``` java
@Service
public class PaymentService {

    private final OrderService orderService;

    public PaymentService(OrderService orderService) {
        this.orderService = orderService;
    }
}
```

Dependency graph:

``` text
OrderService
     ↓
PaymentService
     ↓
OrderService
```

This is a direct cycle.

------------------------------------------------------------------------

# 4. Indirect Circular Dependency

The cycle can contain several beans.

``` java
@Service
public class OrderService {
    public OrderService(PaymentService paymentService) {}
}
```

``` java
@Service
public class PaymentService {
    public PaymentService(NotificationService notificationService) {}
}
```

``` java
@Service
public class NotificationService {
    public NotificationService(OrderService orderService) {}
}
```

Graph:

``` text
OrderService
     ↓
PaymentService
     ↓
NotificationService
     ↓
OrderService
```

This is an **indirect circular dependency**.

------------------------------------------------------------------------

# 5. Constructor Circular Dependency

Constructor circular dependency occurs when dependencies are required
through constructors.

``` java
@Service
public class A {

    private final B b;

    public A(B b) {
        this.b = b;
    }
}
```

``` java
@Service
public class B {

    private final A a;

    public B(A a) {
        this.a = a;
    }
}
```

Graph:

``` text
A → B → A
```

## Creation sequence

Spring starts with `A`.

``` text
Create A
  ↓
A requires B
  ↓
Create B
  ↓
B requires A
  ↓
A is already being created
  ↓
Cycle
```

Neither constructor can finish.

------------------------------------------------------------------------

# 6. Why Constructor Circular Dependency Fails

Consider:

``` java
A(B b)
```

Spring cannot instantiate `A` until `B` exists.

Now:

``` java
B(A a)
```

Spring cannot instantiate `B` until `A` exists.

Therefore:

``` text
A needs B before A can exist
B needs A before B can exist
```

There is no safe starting point.

### Important interview statement

> Constructor injection makes circular dependencies fail early because
> the required dependency must exist before the constructor can execute.

------------------------------------------------------------------------

# 7. Constructor Circular Dependency --- Three Beans

``` java
@Service
class A {
    A(B b) {}
}
```

``` java
@Service
class B {
    B(C c) {}
}
```

``` java
@Service
class C {
    C(A a) {}
}
```

Graph:

``` text
A
↓
B
↓
C
↓
A
```

Creation:

``` text
Create A
  ↓
Need B
  ↓
Create B
  ↓
Need C
  ↓
Create C
  ↓
Need A
  ↓
A is already in creation
  ↓
FAIL
```

------------------------------------------------------------------------

# 8. Setter Circular Dependency

Setter injection separates:

1.  Object instantiation
2.  Dependency injection

Example:

``` java
@Service
public class A {

    private B b;

    @Autowired
    public void setB(B b) {
        this.b = b;
    }
}
```

``` java
@Service
public class B {

    private A a;

    @Autowired
    public void setA(A a) {
        this.a = a;
    }
}
```

Conceptually:

``` text
Create A
   ↓
A exists
   ↓
Create B
   ↓
B exists
   ↓
Inject A into B
   ↓
Inject B into A
```

Spring can resolve **certain singleton setter-based circular
dependencies**.

------------------------------------------------------------------------

# 9. Why Setter Injection Can Be Different

Constructor:

``` java
A(B b)
```

Object creation requires `B`.

Setter:

``` java
A()
setB(B)
```

Object creation does not require `B`.

Spring can first create:

``` java
new A()
```

and later perform:

``` java
a.setB(b);
```

This difference is the reason certain setter cycles can be handled.

------------------------------------------------------------------------

# 10. Field Injection Circular Dependency

Example:

``` java
@Service
public class A {

    @Autowired
    private B b;
}
```

``` java
@Service
public class B {

    @Autowired
    private A a;
}
```

Conceptually:

``` text
Instantiate A
    ↓
Instantiate B
    ↓
Inject A into B
    ↓
Inject B into A
```

Certain singleton cycles can be resolved.

### But do not use field injection just to solve cycles.

Constructor injection is generally preferred.

------------------------------------------------------------------------

# 11. Early Singleton Exposure

## Definition

**Early singleton exposure** means Spring can make an early reference to
a singleton bean available while that bean is still being created.

It is mainly relevant to circular dependencies involving singleton
beans.

Example:

``` text
A → B
↑   ↓
└───┘
```

Conceptual flow:

``` text
Start creating A
       ↓
Instantiate A
       ↓
Register possibility of an early A reference
       ↓
Create B
       ↓
B requires A
       ↓
Obtain early A reference
       ↓
Inject A into B
       ↓
Complete B
       ↓
Inject B into A
       ↓
Complete A
```

------------------------------------------------------------------------

# 12. Early Reference Is Not a Fully Initialized Bean

This distinction is important.

An early reference means:

``` text
Object exists
but
complete Spring initialization has not necessarily finished
```

A fully initialized bean means the relevant creation and initialization
stages have completed.

Therefore:

``` text
Early reference != Fully initialized bean
```

------------------------------------------------------------------------

# 13. Spring Singleton Caches

Spring's singleton registry has three important levels commonly
discussed when explaining circular dependency handling.

``` text
1. singletonObjects
2. earlySingletonObjects
3. singletonFactories
```

------------------------------------------------------------------------

# 14. `singletonObjects`

This is the main singleton cache.

It contains fully initialized singleton instances.

Conceptually:

``` text
singletonObjects

A → fully initialized A
B → fully initialized B
```

Often called the **first-level cache**.

------------------------------------------------------------------------

# 15. `earlySingletonObjects`

Contains early singleton references.

Conceptually:

``` text
earlySingletonObjects

A → early reference to A
```

Often called the **second-level cache**.

It is used when a bean is currently being created and another bean needs
an early reference to it.

------------------------------------------------------------------------

# 16. `singletonFactories`

Contains factories that can create/provide an early reference.

Conceptually:

``` text
singletonFactories

A → ObjectFactory
```

Often called the **third-level cache**.

The factory is especially important when Spring may need to expose an
appropriate reference such as a proxy.

------------------------------------------------------------------------

# 17. Why Three Levels?

The simplified idea is:

``` text
Fully initialized bean
        ↓
singletonObjects
```

If a bean is being created:

``` text
Early reference
        ↓
earlySingletonObjects
```

If Spring needs to generate an early reference:

``` text
ObjectFactory
        ↓
singletonFactories
```

The third level gives Spring an opportunity to obtain the appropriate
early reference instead of blindly exposing the raw object.

------------------------------------------------------------------------

# 18. AOP and Early References

Suppose:

``` java
@Service
@Transactional
public class PaymentService {

    public void pay() {
    }
}
```

Spring may create:

``` text
PaymentService target
        ↓
Spring proxy
        ↓
Other beans interact with proxy
```

During circular dependency resolution, Spring may need to expose the
appropriate proxy/reference.

This is one reason `singletonFactories` matters.

### Important

Do not oversimplify the mechanism as:

> "Spring simply puts the raw object into a cache."

Spring's actual bean creation process also considers post-processors and
proxies.

------------------------------------------------------------------------

# 19. Simplified Internal Flow

For:

``` text
A → B → A
```

a simplified conceptual sequence is:

``` text
1. Start creating A
2. Instantiate A
3. Register an early-reference factory for A
4. Populate A's dependencies
5. A requires B
6. Start creating B
7. B requires A
8. Find A's early reference
9. Inject A into B
10. Complete B
11. Inject B into A
12. Complete initialization of A
13. Store final A in singletonObjects
```

This is a conceptual explanation, not a literal line-by-line
implementation of every Spring version.

------------------------------------------------------------------------

# 20. Why Constructor Injection Cannot Use Early Singleton Exposure in the Same Way

Constructor injection:

``` java
A(B b)
```

requires `B` before `A` can be instantiated.

So:

``` text
A object does not yet exist
       ↓
Cannot provide an already-created A object
       ↓
Constructor cycle fails
```

Setter injection:

``` java
A()
setB(B)
```

allows:

``` text
A object exists
       ↓
Early reference may be exposed
       ↓
B can obtain A
```

This is the core difference.

------------------------------------------------------------------------

# 21. `@Lazy`

`@Lazy` can sometimes break the immediate creation cycle.

Example:

``` java
@Service
public class A {

    private final B b;

    public A(@Lazy B b) {
        this.b = b;
    }
}
```

Conceptually:

``` text
A
 ↓
B proxy/reference
 ↓
Actual B created later
```

Spring can defer creation of `B`.

### When to use

Use `@Lazy` when lazy creation is genuinely appropriate.

Do not use it automatically to hide a design problem.

------------------------------------------------------------------------

# 22. `spring.main.allow-circular-references`

Spring Boot provides:

``` properties
spring.main.allow-circular-references=true
```

This setting controls whether circular references are allowed.

However:

> Enabling circular references should not be the default architectural
> solution.

Better:

``` text
Find cycle
   ↓
Understand why it exists
   ↓
Redesign
   ↓
Remove cycle
```

------------------------------------------------------------------------

# 23. Prototype Beans

Early singleton exposure is specifically about **singleton beans**.

Prototype beans have different lifecycle semantics.

Example:

``` java
@Component
@Scope("prototype")
class A {
    A(B b) {}
}
```

``` java
@Component
@Scope("prototype")
class B {
    B(A a) {}
}
```

The singleton early-reference mechanism cannot simply be applied to
prototype instances.

Therefore prototype circular dependencies generally fail.

------------------------------------------------------------------------

# 24. Real-World Example --- Order and Payment

Bad design:

``` text
OrderService
     ↕
PaymentService
```

Example:

``` java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

``` java
@Service
public class PaymentService {

    private final OrderService orderService;

    public PaymentService(OrderService orderService) {
        this.orderService = orderService;
    }
}
```

Ask:

> Why does PaymentService need OrderService?

Maybe PaymentService only needs the order ID or order details.

Instead of depending on the entire service:

``` text
PaymentService
      ↓
OrderRepository
```

or pass the required data:

``` java
paymentService.pay(orderId);
```

------------------------------------------------------------------------

# 25. Real-World Example --- Event-Based Design

Instead of:

``` text
OrderService → PaymentService
PaymentService → OrderService
```

use:

``` text
OrderService
     ↓
OrderCreatedEvent
     ↓
PaymentService
```

Or:

``` text
PaymentService
     ↓
PaymentCompletedEvent
     ↓
OrderEventHandler
```

This can remove the direct reverse dependency.

------------------------------------------------------------------------

# 26. Real-World Example --- Extract Common Service

Bad:

``` text
A ↔ B
```

Suppose both require the same validation logic.

Extract it:

``` text
        ValidationService
          ↙          ↘
         A            B
```

Now:

``` text
A → ValidationService
B → ValidationService
```

No cycle.

------------------------------------------------------------------------

# 27. Real-World Example --- Too Many Responsibilities

Bad:

``` text
OrderService
- order creation
- payment handling
- notification
- inventory
```

Then:

``` text
PaymentService → OrderService
NotificationService → OrderService
InventoryService → OrderService
```

The service has become a central dependency.

Better:

``` text
OrderService
PaymentService
NotificationService
InventoryService
```

with clearly defined responsibilities and one-way dependencies/events.

------------------------------------------------------------------------

# 28. How to Find the Cycle

Given:

``` text
A → B
B → C
C → D
D → A
```

Trace from any bean:

``` text
A
↓
B
↓
C
↓
D
↓
A
```

When you return to a previously visited bean, you have found a cycle.

In a large application, inspect:

-   Constructor parameters
-   `@Autowired` fields
-   `@Autowired` setters
-   `@Bean` method parameters
-   Service-to-service dependencies
-   Factory/configuration dependencies

------------------------------------------------------------------------

# 29. Common Circular Dependency Patterns

## Pattern 1 --- Service ↔ Service

``` text
AService ↔ BService
```

Very common.

------------------------------------------------------------------------

## Pattern 2 --- Service → Handler → Service

``` text
Service
   ↓
Handler
   ↓
Service
```

------------------------------------------------------------------------

## Pattern 3 --- Service → Event Publisher → Service

Can occur when event infrastructure or listeners create an unintended
reverse dependency.

------------------------------------------------------------------------

## Pattern 4 --- Configuration ↔ Bean

For example, configuration code directly depends on a bean while that
bean indirectly requires the same configuration path.

------------------------------------------------------------------------

## Pattern 5 --- Three or More Services

``` text
A → B → C → A
```

These can be harder to notice because no individual pair appears
circular.

------------------------------------------------------------------------

# 30. Circular Dependency vs Normal Dependency

### Normal

``` text
Controller
    ↓
Service
    ↓
Repository
```

This is a one-way dependency chain.

### Circular

``` text
Service A
    ↓
Service B
    ↓
Service C
    ↓
Service A
```

This is a cycle.

------------------------------------------------------------------------

# 31. Circular Dependency vs Bidirectional Data Relationship

These are not necessarily the same.

For example:

``` text
Order → Customer
Customer → Orders
```

in a domain model does not automatically mean Spring beans have a
circular dependency.

A circular dependency specifically concerns **component/bean
dependencies during object creation or wiring**.

For example:

``` text
OrderService ↔ CustomerService
```

is a Spring bean dependency cycle.

------------------------------------------------------------------------

# 32. Circular Dependency vs Database Relationship

A database can legitimately have relationships between tables without
causing Spring bean cycles.

For example:

``` text
Customer
   ↓
Orders
```

does not mean:

``` text
CustomerService ↔ OrderService
```

The two concepts should not be confused.

------------------------------------------------------------------------

# 33. Why Circular Dependencies Are Usually a Design Smell

A circular dependency means:

``` text
A needs B
B needs A
```

This makes the components tightly coupled.

Consequences can include:

-   Harder testing
-   Harder maintenance
-   Difficult initialization
-   More complicated bean lifecycle
-   Increased coupling
-   Harder refactoring
-   Potential proxy/AOP issues
-   More complicated architecture

Therefore:

> A circular dependency should normally trigger a design review.

------------------------------------------------------------------------

# 34. Best Practices

### Prefer constructor injection

``` java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Constructor injection makes dependencies explicit.

### Avoid using setter injection only to hide a cycle.

### Avoid field injection.

### Keep services focused on one responsibility.

### Prefer one-way dependencies.

### Use events when asynchronous or decoupled communication makes sense.

### Extract shared responsibilities.

### Use `@Lazy` only when it has a legitimate reason.

------------------------------------------------------------------------

# 35. Constructor vs Setter vs Field

Injection     Circular dependency behavior
  ------------- ------------------------------------------
Constructor   Generally fails
Setter        Certain singleton cycles can be resolved
Field         Certain singleton cycles can be resolved
`@Lazy`       Can sometimes defer dependency creation
Redesign      Preferred solution

------------------------------------------------------------------------

# 36. Interview Questions

## Q1. What is circular dependency?

> When two or more beans directly or indirectly depend on each other.

## Q2. Give a simple example.

``` text
A → B
B → A
```

## Q3. Why does constructor injection fail?

> Because A cannot be instantiated without B and B cannot be
> instantiated without A.

## Q4. Why can setter injection sometimes work?

> Because the bean can be instantiated before its setter dependency is
> injected, allowing Spring to use an early reference for certain
> singleton cycles.

## Q5. What is early singleton exposure?

> Making an early reference to a singleton bean available while the bean
> is still being created.

## Q6. What are the three singleton caches?

``` text
singletonObjects
earlySingletonObjects
singletonFactories
```

## Q7. What is `singletonObjects`?

> The primary cache containing fully initialized singleton instances.

## Q8. What is `earlySingletonObjects`?

> A cache containing early singleton references.

## Q9. What is `singletonFactories`?

> A cache of factories that can provide early references, including an
> appropriate reference when proxying is involved.

## Q10. Why is `singletonFactories` important with AOP?

> Because Spring may need to expose a proxy rather than the raw target
> object.

## Q11. Can `@Lazy` solve circular dependency?

> It can sometimes break the immediate creation cycle by injecting a
> lazy reference/proxy, but redesign is preferred.

## Q12. Does Spring always allow circular references?

> No. Circular-reference behavior depends on the bean creation scenario
> and Spring Boot configuration/version.

## Q13. Can prototype beans use early singleton exposure?

> No. The mechanism is specifically for singleton beans.

## Q14. What is the best solution?

> Remove the cycle by redesigning responsibilities and dependencies.

------------------------------------------------------------------------

# 37. Common Interview Trap

### Question

> "If setter injection can resolve circular dependency, why don't we
> always use setter injection?"

### Answer

Because circular dependency is generally a **design smell**.

Setter injection should not be selected simply to bypass the problem.

Constructor injection is generally preferred because:

``` text
Dependencies are explicit
        +
Can be final
        +
Object can be created in a valid state
        +
Easier testing
        +
Circular dependencies are detected early
```

------------------------------------------------------------------------

# 38. Most Important Internal Flow to Remember

For a singleton setter cycle:

``` text
A → B
↑   ↓
└───┘
```

Remember:

``` text
Instantiate A
     ↓
A is being created
     ↓
Register early-reference capability
     ↓
A needs B
     ↓
Instantiate B
     ↓
B needs A
     ↓
Obtain early A reference
     ↓
Inject A into B
     ↓
Finish B
     ↓
Inject B into A
     ↓
Finish A
     ↓
Store fully initialized A
```

------------------------------------------------------------------------

# 39. One-Minute Revision

``` text
Circular Dependency
        ↓
A → B → A
        ↓
Direct or indirect
        ↓
Constructor injection
        ↓
Generally cannot resolve
        ↓
Setter/field injection
        ↓
Some singleton cycles can resolve
        ↓
Why?
        ↓
Early singleton exposure
        ↓
Three levels
        ↓
singletonObjects
earlySingletonObjects
singletonFactories
        ↓
AOP/proxies make early references more important
        ↓
@Lazy can sometimes break creation cycle
        ↓
Prototype ≠ singleton early exposure
        ↓
Best solution
        ↓
Redesign dependency graph
```

------------------------------------------------------------------------

# 40. Final Interview Summary

> **Circular dependency occurs when Spring beans directly or indirectly
> depend on each other. Constructor-based circular dependencies
> generally fail because each bean requires the other before either
> constructor can complete. Certain singleton setter/field circular
> dependencies can be resolved because Spring can instantiate the
> objects first and expose an early singleton reference. Spring's
> singleton registry uses `singletonObjects`, `earlySingletonObjects`,
> and `singletonFactories` as part of this mechanism. `@Lazy` can
> sometimes defer dependency creation, and AOP proxies make
> early-reference handling more important. However, circular
> dependencies usually indicate tight coupling, so the preferred
> solution is to redesign the components and remove the cycle.**
