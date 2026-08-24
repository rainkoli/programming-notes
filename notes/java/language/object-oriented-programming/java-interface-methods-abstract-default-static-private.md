# Java Interface Methods: abstract, default, static, and private Methods

## Grammar Correction

Original: \> Can you give me a Markdown file that includes all the
content you mentioned above? Please make it more detailed.Includes
offical links. Also, please correct my grammar.

Corrected: \> Can you give me a Markdown file that includes all the
content you mentioned above? Please make it more detailed and include
official links. Also, please correct my grammar.

Corrections: - `offical` → `official` (spelling) - `detailed.Includes` →
`detailed and include` (sentence connection and verb form) - After
"please", use the base verb form: "include"

------------------------------------------------------------------------

# 1. Overview

A Java interface was originally designed as a contract.

An interface describes:

> What a class must do, but not how it does it.

Before Java 8, interfaces mainly contained abstract methods.

Modern Java interfaces can contain:

1.  Abstract methods
2.  Default methods
3.  Static methods
4.  Private methods

Evolution:

Java 1.0: - abstract methods

Java 8: - default methods - static methods

Java 9: - private methods

------------------------------------------------------------------------

# 2. Abstract Methods

An abstract method has no implementation.

Example:

``` java
interface Animal {
    void eat();
}
```

Equivalent:

``` java
interface Animal {
    public abstract void eat();
}
```

Interface methods without `private`, `default`, or `static` modifiers
are implicitly abstract.

Abstract methods define requirements that implementing classes must
satisfy.

Example:

``` java
interface Animal {
    void eat();
}

class Dog implements Animal {

    @Override
    public void eat() {
        System.out.println("Dog eats");
    }
}
```

------------------------------------------------------------------------

# 3. Default Methods (Java 8)

## Why Were Default Methods Added?

Before Java 8, adding a new method to an existing interface could break
every implementation class.

Example:

``` java
interface Animal {
    void eat();
}
```

If we add:

``` java
void sleep();
```

all classes implementing Animal must change.

Java 8 introduced default methods to solve this interface evolution
problem.

Example:

``` java
interface Animal {

    void eat();

    default void sleep() {
        System.out.println("Sleeping");
    }
}
```

Classes can use the default implementation or override it.

Example:

``` java
class Cat implements Animal {

    @Override
    public void eat() {

    }

    @Override
    public void sleep() {
        System.out.println("Cat sleeping");
    }
}
```

------------------------------------------------------------------------

# 4. Static Methods (Java 8)

Static methods belong to the interface itself.

Example:

``` java
interface MathUtil {

    static int add(int a, int b) {
        return a + b;
    }
}
```

Calling:

``` java
int result = MathUtil.add(1, 2);
```

They cannot be called through implementation objects.

Wrong:

``` java
Dog.add();
```

Correct:

``` java
Animal.someStaticMethod();
```

------------------------------------------------------------------------

# 5. Private Methods (Java 9)

Private methods allow code reuse inside an interface.

Example:

``` java
interface Animal {

    default void eatFood() {
        prepareFood();
    }

    default void eatSnack() {
        prepareFood();
    }

    private void prepareFood() {
        System.out.println("Preparing food");
    }
}
```

Private methods:

-   cannot be accessed by implementing classes
-   exist only inside the interface

------------------------------------------------------------------------

# 6. Comparison

  Method Type   Java Version   Has Body   Caller
  ------------- -------------- ---------- ----------------------
  abstract      Java 1.0       No         Implementing class
  default       Java 8         Yes        Object instance
  static        Java 8         Yes        Interface name
  private       Java 9         Yes        Interface internally

------------------------------------------------------------------------

# 7. Access Modifier Rules

Allowed:

``` java
public
private
```

Not allowed:

``` java
protected
package-private
```

A normal interface method:

``` java
void test();
```

means:

``` java
public abstract void test();
```

Unlike a class:

``` java
class A {
    void test() {}
}
```

which means package-private.

------------------------------------------------------------------------

# 8. Official Documentation

Java Language Specification:

https://docs.oracle.com/javase/specs/jls/se17/html/

Interface declarations:

https://docs.oracle.com/javase/specs/jls/se17/html/jls-9.html

Interface members:

https://docs.oracle.com/javase/specs/jls/se17/html/jls-9.html#jls-9.2

Oracle Java Tutorial - Default Methods:

https://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html

------------------------------------------------------------------------

# 9. Core Idea

The evolution of interfaces:

    Java 1.0

    interface = contract

    abstract methods:
    "What must be implemented?"


    Java 8

    default methods:
    "What is the default behavior?"

    static methods:
    "What utility belongs to this interface?"


    Java 9

    private methods:
    "How can the interface reuse internal code?"

Summary:

-   Abstract methods define requirements.
-   Default methods provide optional behavior.
-   Static methods provide interface-level utilities.
-   Private methods support internal implementation reuse.
