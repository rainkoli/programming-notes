# Chapter 8. Classes

## 8.3. Field Declarations

JLS 25 的 **§8.3 Field Declarations** 规定：如果子类声明了一个字段，它会隐藏父类中所有可访问的同名字段；被隐藏的父类字段仍可通过 `super` 或转换为父类类型来访问。

### 8.3.1. Field Modifiers

#### 8.3.1.2. `final` Fields

# 8.6. Instance Initializers

```java
// Instance initializer
{
    System.out.println("I'm code block");
}
```

## 8.8. Constructor Declarations

### 8.8.7. Constructor Body

A constructor body is executed as part of creating a new instance. A constructor may invoke another constructor of the same class with `this(...)`, or a constructor of its direct superclass with `super(...)`.

#### Constructor invocation and the old “first statement” rule

In Java 21 and earlier, an explicit constructor invocation had to be the first statement in a constructor body. Therefore, code such as the following is illegal in Java 17:

```java
class Student extends Person {
    private String className;

    Student(String className, String name, byte age) {
        this.className = className;  // Error in Java 17
        super(name, age);            // Must come first
    }
}
```

It must instead be written as:

```java
Student(String className, String name, byte age) {
    super(name, age);
    this.className = className;
}
```

If a constructor does not explicitly invoke `this(...)` or `super(...)`, Java implicitly begins the constructor with:

```java
super();
```

This invokes the no-argument constructor of the direct superclass. Therefore, the direct superclass must provide an accessible no-argument constructor unless another superclass constructor is invoked explicitly.

### 8.8.7.1. Constructor Invocations

The most common constructor invocations are:

```java
this(...);   // invokes another constructor in the same class
super(...);  // invokes a constructor in the direct superclass
```

For example:

```java
class Person {
    private String name;
    private byte age;

    Person(String name, byte age) {
        this.name = name;
        this.age = age;
    }
}

class Student extends Person {
    private String className;

    Student(String className, String name, byte age) {
        super(name, age);
        this.className = className;
    }
}
```

Here, `super(name, age)` initializes the `Person` part of the `Student` object before the remaining statements in the `Student` constructor are executed.

### Version Change: Flexible Constructor Bodies

The rule that `super(...)` or `this(...)` must always be the first statement was a long-standing Java language restriction, but it has been relaxed in recent Java versions.

- **Java 21 and earlier:** an explicit `super(...)` or `this(...)` must be the first statement in the constructor body.
- **Java 22:** *Statements before `super(...)`* was introduced as a preview feature (JEP 447). Statements could appear before the constructor invocation as long as they did not access the instance being constructed.
- **Java 23:** the feature was revised as *Flexible Constructor Bodies* (JEP 482, second preview). The model was expanded so that certain assignments to fields declared in the current class could occur before `super(...)`.
- **Java 24:** the feature was previewed for a third time (JEP 492) without significant changes.
- **Java 25:** *Flexible Constructor Bodies* became a permanent language feature (JEP 513).

In JLS 25, a constructor body may have the following conceptual structure:

```text
constructor {
    // prologue
    ...

    super(...);   // or this(...)

    // epilogue
    ...
}
```

The statements before the constructor invocation are called the **prologue**. The statements after it are called the **epilogue**.

For example, this is legal in Java 25:

```java
class Student extends Person {
    private String className;

    Student(String className, String name, byte age) {
        if (className == null) {
            throw new IllegalArgumentException("className cannot be null");
        }

        this.className = className;
        super(name, age);
    }
}
```

The constructor can validate arguments and initialize certain fields before calling the superclass constructor.

However, the code before `super(...)` or `this(...)` executes in an **early construction context**. The object is not yet fully constructed, so access to the current instance is restricted. In particular, code in this context must not freely read instance fields, invoke instance methods, pass `this` elsewhere, or access superclass members through `super`.

For example:

```java
class Student extends Person {
    private String className;

    Student(String className, String name, byte age) {
        System.out.println(this);          // Error
        System.out.println(this.className); // Error: reads the current instance
        claim();                           // Error: invokes an instance method
        // System.out.println(super.name); // Also not allowed in early construction

        super(name, age);
        this.className = className;
    }

    void claim() {
        System.out.println("I'm studying.");
    }
}
```

The important distinction is therefore:

```text
Java 17:
    super(...) / this(...) must be first.

Java 25:
    safe prologue statements may appear before super(...) / this(...),
    but the current object is still subject to early-construction restrictions.
```

This change relaxes the old syntactic rule while preserving safe object initialization. Superclass construction still occurs exactly once; Java 25 simply permits controlled work to be performed before the constructor invocation.

