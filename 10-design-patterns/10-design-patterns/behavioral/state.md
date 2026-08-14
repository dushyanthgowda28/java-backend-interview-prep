# State Design Pattern

## 1. Overview

The **State Design Pattern** is a **behavioral design pattern** that allows an object to **change its behavior when its internal state changes**.

Instead of having a large number of `if-else` or `switch` statements based on the current state, we encapsulate the behavior of each state into a separate class.

### Simple Definition

> **State Pattern allows an object to alter its behavior when its internal state changes. The object appears to change its class.**

---

## 2. Problem

Suppose we have a **Media Player** with three states:

* Playing
* Paused
* Stopped

The behavior of operations depends on the current state.

| State   | Play            | Pause          | Stop            |
| ------- | --------------- | -------------- | --------------- |
| Playing | Already playing | Pause          | Stop            |
| Paused  | Resume          | Already paused | Stop            |
| Stopped | Start playing   | Cannot pause   | Already stopped |

Without the State Pattern, we might write:

```java
if (state.equals("PLAYING")) {
    // behavior
} else if (state.equals("PAUSED")) {
    // behavior
} else if (state.equals("STOPPED")) {
    // behavior
}
```

As the number of states and operations increases, this becomes difficult to maintain.

---

## 3. Solution

Move state-specific behavior into separate classes.

```text
                  Context
               (MediaPlayer)
                    |
              Current State
                    |
       +------------+------------+
       |            |            |
   PlayingState  PausedState  StoppedState
```

The `MediaPlayer` delegates the operation to its current state.

The current state decides:

1. What behavior should be executed.
2. Whether the state should change.
3. What the next state should be.

---

## 4. Components

The State Pattern mainly contains four components.

### 4.1 Context

The main object whose behavior changes.

Example:

```text
MediaPlayer
```

Responsibilities:

* Maintains the current state.
* Delegates operations to the current state.
* Allows the state to be changed.

---

### 4.2 State

An interface or abstract class that defines operations common to all states.

Example:

```java
public interface State {

    void play(MediaPlayer player);

    void pause(MediaPlayer player);

    void stop(MediaPlayer player);
}
```

---

### 4.3 Concrete States

Implement the behavior for a particular state.

Examples:

```text
PlayingState
PausedState
StoppedState
```

Each state implements the operations differently.

---

### 4.4 Client

Creates and uses the Context.

```java
MediaPlayer player = new MediaPlayer();

player.play();
player.pause();
player.play();
player.stop();
```

The client doesn't need to know which state is currently active.

---

# 5. Java Example

## State Interface

```java
public interface State {

    void play(MediaPlayer player);

    void pause(MediaPlayer player);

    void stop(MediaPlayer player);
}
```

---

## Context

```java
public class MediaPlayer {

    private State state;

    public MediaPlayer() {
        state = new StoppedState();
    }

    public void setState(State state) {
        this.state = state;
    }

    public void play() {
        state.play(this);
    }

    public void pause() {
        state.pause(this);
    }

    public void stop() {
        state.stop(this);
    }
}
```

The important point is:

```java
state.play(this);
```

The `MediaPlayer` doesn't ask:

```java
if (state == PLAYING)
```

Instead, it delegates to the current state.

---

## StoppedState

```java
public class StoppedState implements State {

    @Override
    public void play(MediaPlayer player) {

        System.out.println("Starting music...");

        player.setState(new PlayingState());
    }

    @Override
    public void pause(MediaPlayer player) {

        System.out.println("Cannot pause.");
    }

    @Override
    public void stop(MediaPlayer player) {

        System.out.println("Already stopped.");
    }
}
```

---

## PlayingState

```java
public class PlayingState implements State {

    @Override
    public void play(MediaPlayer player) {

        System.out.println("Already playing.");
    }

    @Override
    public void pause(MediaPlayer player) {

        System.out.println("Pausing...");

        player.setState(new PausedState());
    }

    @Override
    public void stop(MediaPlayer player) {

        System.out.println("Stopping...");

        player.setState(new StoppedState());
    }
}
```

---

## PausedState

```java
public class PausedState implements State {

    @Override
    public void play(MediaPlayer player) {

        System.out.println("Resuming...");

        player.setState(new PlayingState());
    }

    @Override
    public void pause(MediaPlayer player) {

        System.out.println("Already paused.");
    }

    @Override
    public void stop(MediaPlayer player) {

        System.out.println("Stopping...");

        player.setState(new StoppedState());
    }
}
```

---

# 6. Execution Flow

Initially:

```text
MediaPlayer
     |
     v
StoppedState
```

### Step 1

```java
player.play();
```

Flow:

```text
MediaPlayer
     |
     v
StoppedState.play()
     |
     v
new PlayingState()
```

Now:

```text
MediaPlayer
     |
     v
PlayingState
```

---

### Step 2

```java
player.pause();
```

Flow:

```text
MediaPlayer
     |
     v
PlayingState.pause()
     |
     v
new PausedState()
```

Now:

```text
MediaPlayer
     |
     v
PausedState
```

---

### Step 3

```java
player.play();
```

Flow:

```text
MediaPlayer
     |
     v
PausedState.play()
     |
     v
new PlayingState()
```

---

### Step 4

```java
player.stop();
```

Flow:

```text
MediaPlayer
     |
     v
PlayingState.stop()
     |
     v
new StoppedState()
```

---

# 7. Complete Flow

```text
                play()
                  |
                  v
          +---------------+
          |   STOPPED     |
          +---------------+
                  |
                  | play()
                  v
          +---------------+
          |   PLAYING     |
          +---------------+
             |          |
          pause()      stop()
             |          |
             v          v
       +-----------+  STOPPED
       |  PAUSED   |
       +-----------+
             |
           play()
             |
             v
          PLAYING
```

The important concept is that **state transitions are handled by the state objects**.

---

# 8. How It Removes If-Else

### Without State Pattern

```java
public void play() {

    if (state == PLAYING) {
        // ...
    } else if (state == PAUSED) {
        // ...
    } else if (state == STOPPED) {
        // ...
    }
}
```

Every operation potentially contains similar conditions.

### With State Pattern

```java
public void play() {
    state.play(this);
}
```

The current state handles the behavior.

This is the main purpose of the State Pattern.

---

# 9. Real-World Examples

## Vending Machine

Possible states:

```text
Idle
    ↓
MoneyInserted
    ↓
ProductSelected
    ↓
Dispensing
    ↓
Idle
```

The behavior of `insertMoney()`, `selectProduct()`, and `dispense()` depends on the current state.

---

## Order Processing

Possible states:

```text
NEW
 ↓
PAID
 ↓
SHIPPED
 ↓
DELIVERED
```

An order behaves differently depending on its current state.

For example:

```text
NEW       → Payment allowed
PAID      → Shipping allowed
SHIPPED   → Delivery tracking
DELIVERED → No further shipment
```

---

## ATM

Possible states:

```text
NoCard
   ↓
CardInserted
   ↓
PinVerified
   ↓
Transaction
```

The available operations depend on the current state.

---

## Traffic Signal

```text
RED
 ↓
GREEN
 ↓
YELLOW
 ↓
RED
```

Each state defines different behavior.

---

# 10. When to Use State Pattern

Use it when:

* An object's behavior changes based on its current state.
* There are many `if-else`/`switch` statements checking state.
* The object has well-defined state transitions.
* New states are likely to be added.
* State-specific behavior is becoming complex.

---

# 11. When NOT to Use It

Don't use State Pattern just because an object has a state variable.

For example:

```java
enum Status {
    ACTIVE,
    INACTIVE
}
```

If you only have one or two simple conditions, an enum and a simple `if` may be much cleaner.

State Pattern becomes valuable when **state-dependent behavior becomes substantial or complex**.

---

# 12. Advantages

### 1. Removes complex conditional logic

Replaces large:

```java
if-else
switch
```

structures with polymorphism.

### 2. Single Responsibility

Each state class handles one particular state.

### 3. Open/Closed Principle

New states can generally be added without modifying all existing state logic.

### 4. Easier maintenance

State-specific behavior is isolated.

### 5. Explicit state transitions

Transitions such as:

```text
Playing → Paused
Paused → Playing
Playing → Stopped
```

are easy to identify.

---

# 13. Disadvantages

### 1. More classes

Instead of one class:

```text
MediaPlayer
```

you may have:

```text
MediaPlayer
State
PlayingState
PausedState
StoppedState
```

### 2. Can be over-engineering

For very simple state logic, State Pattern may make the code unnecessarily complicated.

### 3. State transitions can become complex

With many states and transitions, the number of interactions can grow significantly.

---

# 14. State vs Strategy

These patterns have a similar structure but different purposes.

| State                                               | Strategy                                         |
| --------------------------------------------------- | ------------------------------------------------ |
| Behavior changes because the object's state changes | Algorithm changes because a strategy is selected |
| Focuses on object lifecycle/state                   | Focuses on interchangeable algorithms            |
| State objects commonly cause state transitions      | Strategy usually doesn't change itself           |
| Context moves between states                        | Client/application chooses the strategy          |
| Example: Order lifecycle                            | Example: Payment algorithm                       |

### State

```text
Order

NEW → PAID → SHIPPED → DELIVERED
```

### Strategy

```text
PaymentService
      |
      +── CreditCardStrategy
      +── UPIStrategy
      +── PayPalStrategy
```

---

# 15. State Pattern vs Enum

A simple state can be represented using an enum:

```java
enum State {
    PLAYING,
    PAUSED,
    STOPPED
}
```

This is fine when behavior is simple.

Use the State Pattern when each state has significant behavior:

```text
PlayingState
    ↓
complex playing behavior

PausedState
    ↓
complex pause behavior

StoppedState
    ↓
complex stopped behavior
```

A useful rule:

> **If the state variable is becoming the center of many conditional statements, consider the State Pattern.**

---

# 16. Key Concept to Remember

The most important idea is:

```text
                    Context
                       |
                Current State
                       |
          +------------+------------+
          |            |            |
       State A       State B      State C
          |            |            |
       Behavior      Behavior     Behavior
```

The Context does **not** need to know how every state behaves.

It simply says:

```java
state.operation(this);
```

The current state decides what happens.

---

# 17. Interview Answer

### What is State Design Pattern?

> State is a behavioral design pattern where an object's behavior changes based on its internal state. Instead of using large conditional statements to handle different states, state-specific behavior is encapsulated into separate classes, and the context delegates operations to its current state.

### Key components

```text
Context
State
Concrete State
Client
```

### Main benefit

> **It replaces complex state-based conditional logic with polymorphism.**

### Typical examples

```text
Media Player
Vending Machine
ATM
Order Processing
Traffic Signal
Document Workflow
```
