# Circular Dependency in Spring

## 1. Definition

A **circular dependency** occurs when two or more Spring beans depend on
each other directly or indirectly.

### Direct circular dependency

``` text
A → B
↑   ↓
└───┘
```

Example:

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

Here:

``` text
A depends on B
B depends on A
```

------------------------------------------------------------------------

# 2. What Causes Circular Dependency?

Circular dependency is usually caused by **tightly coupled components or
poor separation of responsibilities**.

Common causes:

1.  Direct mutual dependency
2.  Indirect dependency between multiple beans
3.  Poor separation of responsibilities
4.  Bidirectional service dependencies
5.  Services calling each other in both directions
6.  One service doing too many responsibilities

The important point is:

> Circular dependency is usually a design problem. Spring encounters the
> cycle while creating the beans.

------------------------------------------------------------------------

# 3. Direct Circular Dependency

The simplest case is:

``` text
A → B
B → A
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

Dependency graph:

``` text
OrderService
      ↓
PaymentService
      ↓
OrderService
      ↑
     CYCLE
```

------------------------------------------------------------------------

# 4. Indirect Circular Dependency

A circular dependency can involve more than two beans.

Example:

``` text
A → B
B → C
C → A
```

``` java
@Service
public class A {
    public A(B b) {}
}
```

``` java
@Service
public class B {
    public B(C c) {}
}
```

``` java
@Service
public class C {
    public C(A a) {}
}
```

Dependency graph:

``` text
A
↓
B
↓
C
↓
A
```

This is an **indirect circular dependency**.

------------------------------------------------------------------------

# 5. Constructor Circular Dependency

A constructor circular dependency occurs when dependencies are required
through constructors.

Example:

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

Dependency:

``` text
A → B → A
```

## How Spring processes it

Suppose Spring starts creating `A`.

### Step 1: Create A

Spring sees:

``` java
A(B b)
```

Therefore, `B` is required before `A` can be instantiated.

``` text
Creating A
    ↓
Need B
```

### Step 2: Create B

Spring starts creating `B`.

``` java
B(A a)
```

Therefore, `A` is required.

``` text
Creating A
    ↓
Creating B
    ↓
Need A
```

### Step 3: A is already being created

Spring needs `A`, but `A` cannot finish because it is waiting for `B`.

``` text
A
↓
B
↓
A
↓
B
...
```

The cycle cannot be resolved.

Typically, Spring fails with an error involving:

``` text
BeanCurrentlyInCreationException
```

or Spring Boot reports that the dependencies form a cycle.

------------------------------------------------------------------------

# 6. Why Constructor Circular Dependency Cannot Normally Be Resolved

Constructor injection requires the dependency **before the object can be
instantiated**.

For:

``` java
A(B b)
```

Spring effectively needs:

``` text
B must exist
    ↓
before A can be created
```

For:

``` java
B(A a)
```

Spring needs:

``` text
A must exist
    ↓
before B can be created
```

Therefore:

``` text
A requires B to construct
B requires A to construct
```

There is no completed object that Spring can provide to either
constructor.

### Key point

> Constructor injection exposes circular dependencies because the object
> cannot be instantiated until its constructor dependencies are
> available.

------------------------------------------------------------------------

# 7. Setter Circular Dependency

A setter circular dependency occurs when two beans depend on each other
through setter injection.

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

Dependency:

``` text
A → B
↑   ↓
└───┘
```

Unlike constructor injection, Spring can instantiate the objects without
immediately having their dependencies.

Conceptually:

``` text
Create A
   ↓
Create B
   ↓
Inject A into B
   ↓
Inject B into A
   ↓
Complete initialization
```

Therefore, Spring can resolve **certain circular dependencies involving
singleton beans and setter/field injection**.

------------------------------------------------------------------------

# 8. Field Injection Circular Dependency

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

Dependency:

``` text
A ↔ B
```

Because the objects can be instantiated before the fields are injected,
Spring can resolve some singleton cycles using its early-reference
mechanism.

However:

> Field injection is not recommended just to solve circular
> dependencies.

Constructor injection is generally preferred because dependencies are
explicit and objects can be created in a valid state.

------------------------------------------------------------------------

# 9. Early Singleton Exposure

**Early singleton exposure** means Spring makes an early reference to a
singleton bean available while that bean is still being created.

It is important for understanding how Spring can resolve certain
setter/field circular dependencies.

Example:

``` text
A → B
↑   ↓
└───┘
```

Conceptually:

``` text
Create A
   ↓
A object exists
   ↓
Expose an early reference to A
   ↓
Create B
   ↓
B needs A
   ↓
Use early reference to A
   ↓
Finish B
   ↓
Inject B into A
   ↓
Finish A
```

Important:

> Early exposure does NOT mean that A is fully initialized.

It means that an appropriate reference to the bean can be made available
before complete initialization.

------------------------------------------------------------------------

# 10. Why Early Exposure Is Possible for Setter/Field Injection

With setter injection:

``` java
A(B b)
```

is NOT used.

Instead:

``` java
A()
setB(B)
```

The object can therefore be created first:

``` java
new A()
```

After the object exists, Spring can inject `B`.

That gives Spring an opportunity to expose an early reference to `A`.

With constructor injection:

``` java
new A(B)
```

Spring cannot create `A` without `B`.

Therefore there is no instantiated `A` to expose early.

------------------------------------------------------------------------

# 11. Spring's Three-Level Singleton Cache

Spring's `DefaultSingletonBeanRegistry` uses three important levels of
singleton storage when handling singleton creation and circular
dependencies.

## 11.1 `singletonObjects`

Contains fully initialized singleton beans.

Conceptually:

``` text
singletonObjects

A → fully initialized A
B → fully initialized B
```

This is commonly referred to as the **first-level cache**.

------------------------------------------------------------------------

## 11.2 `earlySingletonObjects`

Contains early references to singleton beans that are currently being
created.

Conceptually:

``` text
earlySingletonObjects

A → early A reference
```

This is commonly referred to as the **second-level cache**.

------------------------------------------------------------------------

## 11.3 `singletonFactories`

Contains `ObjectFactory` instances that can provide an early reference.

Conceptually:

``` text
singletonFactories

A → ObjectFactory<A>
```

This is commonly referred to as the **third-level cache**.

It is particularly important when Spring needs to obtain an appropriate
early reference, including cases involving proxies.

------------------------------------------------------------------------

# 12. Why Does Spring Use `singletonFactories`?

Suppose a bean is subject to AOP.

Example:

``` java
@Service
@Transactional
public class PaymentService {
}
```

The object used by other beans may be a proxy:

``` text
PaymentService target
       ↓
   Spring proxy
       ↓
Other bean
```

Spring may therefore need to expose the **appropriate early reference**,
potentially the proxy, rather than simply exposing the raw object.

The third-level cache provides a factory that can create the early
reference when required.

------------------------------------------------------------------------

# 13. Simplified Early-Exposure Flow

For:

``` text
A → B
B → A
```

the simplified flow is:

``` text
1. Start creating A
        ↓
2. Instantiate A
        ↓
3. Register an ObjectFactory for an early A reference
        ↓
4. Populate A's dependencies
        ↓
5. A needs B
        ↓
6. Start creating B
        ↓
7. B needs A
        ↓
8. Obtain early A reference
        ↓
9. Inject A into B
        ↓
10. Finish B
        ↓
11. Inject B into A
        ↓
12. Initialize A
        ↓
13. Store fully initialized A in singletonObjects
```

This is a **simplified conceptual flow** of Spring's internal bean
creation process.

------------------------------------------------------------------------

# 14. Important: Early Reference vs Fully Initialized Bean

These are different.

### Fully initialized bean

``` text
Instantiate
   ↓
Populate properties
   ↓
BeanPostProcessor processing
   ↓
Initialization callbacks
   ↓
Fully initialized bean
```

### Early reference

``` text
Instantiate
   ↓
Early reference available
   ↓
Bean is still being initialized
```

Therefore:

> An early reference can point to an object that has not completed its
> complete Spring lifecycle yet.

------------------------------------------------------------------------

# 15. AOP / Proxy and Circular Dependencies

Circular dependencies become more complicated when Spring AOP is
involved.

For example:

``` java
@Service
@Transactional
public class A {
}
```

Spring may manage:

``` text
A target
   ↑
   |
Proxy
```

Another bean generally interacts with the Spring-managed proxy rather
than directly with the target object.

Therefore, when exposing an early reference, Spring may need to expose
the appropriate proxy/reference.

This is one reason the third-level singleton factory is important.

------------------------------------------------------------------------

# 16. `@Lazy` and Circular Dependency

`@Lazy` can sometimes break an immediate circular dependency.

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
B proxy
 ↓
Actual B created later
```

Spring can inject a lazy reference/proxy instead of immediately
requiring the actual `B` instance.

### Important

`@Lazy` can be useful, but it should not normally be the first solution.

Prefer:

``` text
Identify cycle
     ↓
Understand why it exists
     ↓
Redesign dependency
     ↓
Remove cycle
```

------------------------------------------------------------------------

# 17. `spring.main.allow-circular-references`

Spring Boot provides:

``` properties
spring.main.allow-circular-references=true
```

This controls whether circular references are allowed by the application
context.

However, enabling this should not be treated as the preferred
architectural solution.

Do not use:

``` properties
spring.main.allow-circular-references=true
```

simply to hide a poorly designed dependency graph.

Prefer removing the cycle.

------------------------------------------------------------------------

# 18. Prototype Scope and Circular Dependency

Circular dependency handling is mainly associated with **singleton
beans**.

Prototype beans have different lifecycle semantics.

Example:

``` java
@Component
@Scope("prototype")
public class A {
    public A(B b) {}
}
```

``` java
@Component
@Scope("prototype")
public class B {
    public B(A a) {}
}
```

Spring cannot rely on the same singleton early-reference mechanism for
prototype instances.

Therefore, prototype circular dependencies generally fail.

### Key point

> Early singleton exposure is specifically a singleton-bean mechanism.
> Do not assume it applies to prototype beans.

------------------------------------------------------------------------

# 19. Circular Dependency in Real Applications

A common example:

``` text
OrderService
      ↓
PaymentService
      ↓
OrderService
```

This may happen because both services contain responsibilities that
should have been separated.

A better design could be:

``` text
OrderService
      ↓
PaymentService
```

Or use an event:

``` text
OrderService
      ↓
OrderCreatedEvent
      ↓
PaymentService
```

Another example:

``` text
PaymentService
      ↓
PaymentCompletedEvent
      ↓
OrderEventHandler
```

This creates a more one-directional dependency structure.

------------------------------------------------------------------------

# 20. How to Fix Circular Dependency

## Solution 1: Redesign the dependency

Best solution:

``` text
A ↔ B
```

becomes:

``` text
A → B
```

or:

``` text
A → CommonService ← B
```

------------------------------------------------------------------------

## Solution 2: Extract common responsibility

Instead of:

``` text
A ↔ B
```

extract shared functionality:

``` text
       CommonService
        ↙         ↘
       A           B
```

This reduces coupling.

------------------------------------------------------------------------

## Solution 3: Use an event

Instead of:

``` text
A → B
B → A
```

use:

``` text
A
↓
Event
↓
B
```

For example:

``` text
OrderService
      ↓
OrderCreatedEvent
      ↓
PaymentService
```

------------------------------------------------------------------------

## Solution 4: Use `@Lazy` when appropriate

``` java
public A(@Lazy B b) {
    this.b = b;
}
```

This can defer creation of the dependency.

Use it carefully.

------------------------------------------------------------------------

## Solution 5: Setter injection

Setter injection can allow some singleton cycles to be resolved:

``` java
@Autowired
public void setB(B b) {
    this.b = b;
}
```

But changing constructor injection to setter injection **only to make
the cycle work** is usually not the best design.

------------------------------------------------------------------------

# 21. Constructor vs Setter vs Field

  Injection type   Circular dependency
  ---------------- -----------------------------------------
  Constructor      Generally cannot be resolved
  Setter           Some singleton cycles can be resolved
  Field            Some singleton cycles can be resolved
  `@Lazy`          Can sometimes break the immediate cycle
  Redesign         Preferred solution

------------------------------------------------------------------------

# 22. Why Constructor Injection Is Still Preferred

Constructor injection provides several advantages:

-   Dependencies are explicit
-   Dependencies can be `final`
-   Object can be created in a valid state
-   Easier to test
-   Prevents partially initialized objects
-   Exposes circular dependencies early

Therefore:

> The fact that setter injection can sometimes resolve circular
> dependencies is not a reason to prefer setter injection.

A circular dependency is often a signal that the design should be
reconsidered.

------------------------------------------------------------------------

# 23. How to Identify a Circular Dependency

Look at the dependency graph.

Example:

``` text
A → B
B → C
C → D
D → A
```

Start from `A`:

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

Because you eventually return to `A`, there is a cycle.

In a real Spring application, the cycle may involve:

``` text
Controller
   ↓
Service A
   ↓
Service B
   ↓
Repository
```

which is fine.

But:

``` text
Service A
   ↓
Service B
   ↓
Service C
   ↓
Service A
```

is circular.

------------------------------------------------------------------------

# 24. Common Misconceptions

## Misconception 1

> Spring always creates all beans first and then injects dependencies.

Not exactly.

Spring creates beans while resolving their dependencies through its bean
creation machinery.

------------------------------------------------------------------------

## Misconception 2

> Setter injection always solves circular dependencies.

No.

Only certain circular dependencies, particularly eligible singleton
cases, can be resolved.

------------------------------------------------------------------------

## Misconception 3

> `@Lazy` fixes the design.

No.

`@Lazy` can break the immediate creation cycle, but the underlying
coupling may still exist.

------------------------------------------------------------------------

## Misconception 4

> Circular dependency is a Spring bug.

No.

The dependency cycle usually exists in the application's design.

Spring detects the cycle while creating the beans.

------------------------------------------------------------------------

## Misconception 5

> Early singleton exposure means the bean is fully initialized.

No.

It means an early reference can be made available before complete
initialization.

------------------------------------------------------------------------

# 25. Complete Mental Model

### Constructor circular dependency

``` text
A requires B to construct
        ↓
B requires A to construct
        ↓
Neither can be instantiated
        ↓
Circular dependency
        ↓
FAIL
```

### Setter/field circular dependency

``` text
Instantiate A
      ↓
A object exists
      ↓
Expose early reference if needed
      ↓
Instantiate B
      ↓
B needs A
      ↓
Use early A reference
      ↓
Complete B
      ↓
Inject B into A
      ↓
Complete A
```

### Best architectural solution

``` text
A ↔ B
 ↓
Find why both need each other
 ↓
Separate responsibilities
 ↓
Extract common functionality / use event / redesign
 ↓
A → B
```

------------------------------------------------------------------------

# 26. Interview Questions to Know

### Q1. What is circular dependency?

> A situation where two or more beans directly or indirectly depend on
> each other.

### Q2. Why does constructor circular dependency fail?

> Because each bean must have the other bean available before its
> constructor can execute, so neither bean can be instantiated first.

### Q3. Can Spring resolve setter circular dependencies?

> Spring can resolve certain circular dependencies involving singleton
> beans and setter/field injection by exposing an early reference.

### Q4. What is early singleton exposure?

> Making an early reference to a singleton bean available while the bean
> is still being created, primarily to support certain circular
> dependency scenarios.

### Q5. What are Spring's three singleton caches?

> `singletonObjects`, `earlySingletonObjects`, and `singletonFactories`.

### Q6. What is the purpose of `singletonFactories`?

> It provides a way to obtain an early reference, which is especially
> important when the appropriate reference may involve an AOP proxy.

### Q7. Does early exposure mean the bean is fully initialized?

> No. The bean is still in the creation/initialization process.

### Q8. Can `@Lazy` solve circular dependency?

> It can sometimes break the immediate creation cycle by injecting a
> lazy/proxy reference, but redesigning the dependency is preferred.

### Q9. Can prototype beans use the same circular dependency mechanism?

> No. Early singleton exposure is specifically related to singleton
> beans, and prototype circular dependencies generally cannot be
> resolved using that mechanism.

### Q10. What is the best way to fix circular dependency?

> Redesign the dependency graph and remove the cycle rather than relying
> on Spring's circular-reference handling.

------------------------------------------------------------------------

# 27. Quick Revision

``` text
Circular Dependency
        ↓
A → B → A
        ↓
Can be direct or indirect
        ↓
Constructor injection
        ↓
Cannot normally resolve
        ↓
Setter/field injection
        ↓
Some singleton cycles can be resolved
        ↓
Early singleton exposure
        ↓
Spring can provide an early reference
        ↓
Three singleton caches
        ↓
singletonObjects
earlySingletonObjects
singletonFactories
        ↓
@Lazy can sometimes break the cycle
        ↓
Best solution
        ↓
Redesign and remove the cycle
```

## Key Takeaway

> **Circular dependency is a dependency cycle between Spring beans.
> Constructor injection normally fails because neither bean can be
> instantiated without the other. Certain singleton setter/field cycles
> can be resolved through Spring's early singleton exposure mechanism,
> which uses the singleton caches to provide an early reference.
> However, circular dependencies generally indicate tight coupling, so
> the preferred solution is to redesign the dependency graph and remove
> the cycle.**
