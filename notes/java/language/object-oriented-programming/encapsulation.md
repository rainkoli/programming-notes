# Understanding Encapsulation in Java

## 1. The Core Meaning of Encapsulation

In Java, **encapsulation** means:

> Combining an object's data and the methods that operate on that data inside a class, while restricting direct access to the object's internal state.

Encapsulation mainly includes two ideas:

1. Putting data and related behavior together in one class.
2. Hiding internal implementation details and exposing only necessary operations.

For example, a bank account contains a balance and also provides operations such as depositing and withdrawing money.

```java
public class BankAccount {
    private double balance;

    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("The deposit amount must be greater than 0.");
        }

        balance += amount;
    }

    public void withdraw(double amount) {
        if (amount <= 0 || amount > balance) {
            throw new IllegalArgumentException("The withdrawal amount is invalid.");
        }

        balance -= amount;
    }

    public double getBalance() {
        return balance;
    }
}
```

External code cannot directly modify the balance:

```java
BankAccount account = new BankAccount();

// Not allowed because balance is private
account.balance = -1000;
```

Instead, it must use the operations provided by the class:

```java
account.deposit(1000);
account.withdraw(200);
```

This is encapsulation.

---

## 2. Why Fields Should Not Usually Be `public`

Suppose the balance is declared as a public field:

```java
public class BankAccount {
    public double balance;
}
```

External code could then assign any value to it:

```java
account.balance = -10000;
```

This is dangerous because the object loses control over its own state.

After changing the field to `private`:

```java
private double balance;
```

all modifications must go through methods defined by the class.

These methods can:

- Validate input.
- Reject illegal operations.
- Maintain a consistent object state.
- Record logs.
- Trigger related business operations.

Therefore, encapsulation is not only about hiding variables. Its deeper purpose is:

> Allowing an object to control and protect its own state.

---

## 3. Encapsulation Is More Than `private` Fields and Getters/Setters

Many beginners think encapsulation simply means:

```java
private String name;

public String getName() {
    return name;
}

public void setName(String name) {
    this.name = name;
}
```

This uses access control, but it may still be poor encapsulation.

For example:

```java
private int age;

public void setAge(int age) {
    this.age = age;
}
```

External code can still pass an invalid value:

```java
person.setAge(-100);
```

A better implementation validates the input:

```java
public void setAge(int age) {
    if (age < 0 || age > 150) {
        throw new IllegalArgumentException("The age is invalid.");
    }

    this.age = age;
}
```

Some fields should not have setters at all.

For example, a bank account should normally not provide:

```java
public void setBalance(double balance) {
    this.balance = balance;
}
```

Such a method would allow external code to bypass deposit and withdrawal rules.

Instead, the class should provide methods with clear business meaning:

```java
deposit(amount);
withdraw(amount);
```

Therefore:

> `private`, getters, and setters are tools used to implement encapsulation, but they are not encapsulation itself.

---

## 4. What Does Encapsulation Hide?

Encapsulation usually hides two kinds of information.

### 4.1 Internal Data

```java
private double balance;
```

External code cannot directly read or modify the field unless the class explicitly allows it.

### 4.2 Internal Implementation

Consider a method that finds a user:

```java
public User findUser(int id) {
    // The user may be found in an array, database, or cache.
}
```

The caller only needs to write:

```java
User user = repository.findUser(10);
```

The caller does not need to know:

- Whether the implementation uses an array or a linked list.
- Whether the data comes from MySQL or PostgreSQL.
- Whether a cache is used.
- Which SQL statement is executed.

The internal implementation can change without affecting the caller, as long as the public method remains compatible.

This leads to another important meaning of encapsulation:

> Providing a stable public interface while allowing the internal implementation to change freely.

---

## 5. Encapsulation and Access Modifiers

Java uses access modifiers to control visibility.

| Modifier | Visibility |
|---|---|
| `private` | Accessible only inside the current class |
| package-private | Accessible inside the same package |
| `protected` | Accessible in the same package and by eligible subclasses |
| `public` | Accessible from anywhere |

Encapsulation commonly uses `private`, but it does not mean that everything must be private.

A typical design is:

- Use `private` for internal state and implementation details.
- Use `protected` carefully when subclasses need access.
- Use `public` for operations that form the external interface.

Example:

```java
public class Student {
    private String name;
    private int score;

    public Student(String name) {
        this.name = name;
    }

    public void updateScore(int score) {
        if (score < 0 || score > 100) {
            throw new IllegalArgumentException("The score must be between 0 and 100.");
        }

        this.score = score;
    }

    public int getScore() {
        return score;
    }
}
```

The class creates a boundary:

```text
External code
    |
    | Calls only public methods
    v
Public methods
    |
    | Validate and control access
    v
Private internal state
```

---

## 6. Encapsulation Protects Object Invariants

An **invariant** is a rule that must always remain true for an object in a valid state.

For example:

```java
public class Rectangle {
    private double width;
    private double height;

    public Rectangle(double width, double height) {
        if (width <= 0 || height <= 0) {
            throw new IllegalArgumentException("Width and height must be greater than 0.");
        }

        this.width = width;
        this.height = height;
    }
}
```

The invariants of this class are:

```text
width > 0
height > 0
```

Because the fields are private, external code cannot directly break these rules:

```java
rectangle.width = -10;
```

A deeper definition of encapsulation is therefore:

> Encapsulation controls access to an object's state so that the object can remain valid and consistent.

---

## 7. A Real-World Analogy

A car is a good example of encapsulation.

A driver interacts with:

- The steering wheel.
- The accelerator.
- The brake pedal.
- The gear selector.

These are the car's public interface.

The driver does not directly control:

- Engine cylinders.
- Fuel injection.
- Internal gearbox components.
- Brake fluid pressure.

These are implementation details.

The relationship can be represented as follows:

```text
Steering wheel, accelerator, brakes
                |
                v
          Public methods

Engine, gears, braking system
                |
                v
      Private fields and methods
```

The car allows the driver to press the accelerator, but it does not allow the driver to directly modify the engine's internal state.

That is the idea of encapsulation.

---

## 8. Encapsulation vs. Abstraction

Encapsulation and abstraction are related, but they are not the same.

### Abstraction

Abstraction focuses on:

> What ability should an object provide?

Example:

```java
public interface Payment {
    void pay(double amount);
}
```

The interface describes a payment capability.

### Encapsulation

Encapsulation focuses on:

> How is the ability implemented, and which internal details should remain hidden?

Example:

```java
public class CreditCardPayment implements Payment {
    private String cardNumber;

    @Override
    public void pay(double amount) {
        // Internal payment process
    }
}
```

A simple way to remember the difference is:

```text
Abstraction: What does the object provide?
Encapsulation: What does the object hide and protect?
```

---

## 9. Main Benefits of Encapsulation

### 9.1 Data Validation

Invalid values can be rejected:

```java
person.setAge(-100);
```

### 9.2 Lower Coupling

External code depends on public methods rather than internal fields and implementation details.

### 9.3 Easier Maintenance

The internal implementation can change without forcing all callers to change.

### 9.4 Better Security

Sensitive data cannot be read or modified freely.

### 9.5 Clearer Responsibility

An object manages its own state instead of allowing unrelated external code to modify it directly.

---

## 10. Final Understanding

Do not understand encapsulation only as:

> Declaring fields as `private` and providing getters and setters.

A more accurate definition is:

> Encapsulation creates a boundary around an object. External code can interact with the object only through the operations that the object intentionally exposes, while the object itself is responsible for protecting its data and maintaining its rules.

The overall process can be summarized as follows:

```text
Data + methods that operate on the data
                |
                v
          Placed inside a class
                |
                v
   Unnecessary implementation details hidden
                |
                v
  Only safe and meaningful operations exposed
                |
                v
      Object state remains valid and consistent
```

The central question answered by encapsulation is:

> Who is responsible for maintaining an object's state?

The answer is:

> The object itself is responsible, while external code interacts with it only through its public interface.
