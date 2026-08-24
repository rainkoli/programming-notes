# Chapter 15. Expressions

## 15.11. Field Access Expressions

A **field access expression** is an expression used to access a field of an object or array. JLS §15.11 defines three forms:

```text
FieldAccess:
    Primary . Identifier
    super . Identifier
    TypeName . super . Identifier
```

### Key terms

- **FieldAccess**: an expression that directly accesses a field, such as `person.name`, `this.age`, or `super.value`.
- **Primary**: the expression on the left side of `.` that produces the object whose field is being accessed. Examples include `this`, `new Person()`, and a method invocation such as `getPerson()`.
- **Identifier**: the name of the field being accessed, such as `name`, `age`, or `className`.
- **TypeName**: the name of a type used to qualify `super` in some nested-class situations.
- **super**: a special keyword used to access members of the superclass part of the current object.
- **simple name**: a field may also be referred to by its name alone, such as `age`, when the field is accessible in the current context.

### 1. `Primary . Identifier`

This is the most common form:

```java
class Person {
    public String name = "Tom";
}

Person person = new Person();

System.out.println(person.name);
System.out.println(new Person().name);
```

In `person.name`, `person` is the expression used to obtain the object and `name` is the field identifier. `this.name` is another example of this form because `this` refers to the current object.

### 2. `super . Identifier`

This form accesses a field declared in a superclass:

```java
class Parent {
    int value = 10;
}

class Child extends Parent {
    int value = 20;

    void print() {
        System.out.println(this.value);   // 20
        System.out.println(super.value);  // 10
    }
}
```

If a subclass declares a field with the same name as an accessible field in its superclass, the subclass field **hides** the superclass field. Fields are not overridden in the same way as instance methods. `this.value` refers to the field in `Child`, while `super.value` explicitly refers to the field inherited from `Parent`.

### 3. `TypeName . super . Identifier`

This form is mainly used in nested-class situations to specify the superclass of a particular enclosing instance:

```java
class Parent {
    int value = 10;
}

class Outer extends Parent {
    class Inner {
        void print() {
            System.out.println(Outer.super.value);
        }
    }
}
```

Here, `Outer` is the `TypeName`, and `Outer.super.value` accesses `value` through the superclass of the enclosing `Outer` instance.

### Simple-name access

A field of the current instance or current class can also be referenced using only its name:

```java
class Person {
    int age = 20;

    void print() {
        System.out.println(age);
        System.out.println(this.age);
    }
}
```

In this example, `age` is a **simple name**, while `this.age` is a **field access expression**. Both refer to the same field when no other declaration hides it.

### Field access and compile-time type

Field access is resolved according to the **compile-time type** of the expression, not by dynamic dispatch:

```java
class Parent {
    int value = 10;
}

class Child extends Parent {
    int value = 20;
}

Child child = new Child();
Parent parent = child;

System.out.println(child.value);   // 20
System.out.println(parent.value);  // 10
```

Although `child` and `parent` refer to the same `Child` object, `child.value` selects `Child.value`, while `parent.value` selects `Parent.value`. This is an important difference between **field hiding** and **method overriding**.

#### 15.12.4.4. Locate Method to Invoke

JLS §15.12.4.4 规定：调用实例方法时，如果目标引用为 `null`，就在运行时抛出 `NullPointerException`。



### 15.22.1. Integer Bitwise Operators `&`, `^`, and `|`

### 15.22.2. Boolean Logical Operators `&`, `^`, and `|`

## 15.23. Conditional-And Operator `&&`

## 15.28. `switch` Expressions

```java
switch() {
    case :
        ...;
        break;
    case :
        ...;
        break;
    case :
        ...;
        break;
    default :
        ...;
        break;
}
```

