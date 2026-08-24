# Covariance in Java

## 1. Core idea

Assume the following type hierarchy:

```java
class Animal {
}

class Dog extends Animal {
}

class Cat extends Animal {
}
```

Because `Dog` is a subtype of `Animal`, we can write:

```text
Dog <: Animal
```

A type constructor is **covariant** when it preserves the direction of the subtype relationship:

```text
Dog <: Animal
        ↓
F<Dog> <: F<Animal>
```

Java does not apply covariance uniformly. Arrays, generic types, wildcards, and method return types follow different rules.

---

## 2. Quick comparison

| Java feature | Relationship | Result |
|---|---|---|
| Reference arrays | `Dog[] <: Animal[]` | Covariant |
| Generic classes | `List<Dog>` is not a subtype of `List<Animal>` | Invariant |
| Upper-bounded wildcard | `List<Dog> <: List<? extends Animal>` | Covariant-style read view |
| Lower-bounded wildcard | `List<Animal> <: List<? super Dog>` | Contravariant-style write view |
| Overridden return type | `Animal` may become `Dog` | Covariant return type |
| Overridden parameter type | `Animal` cannot become `Dog` | Different signature; normally overloads |

---

# Part I: Array covariance

## 3. Java reference arrays are covariant

Because `Dog` is a subtype of `Animal`, Java also treats `Dog[]` as a subtype of `Animal[]`:

```java
Dog[] dogs = new Dog[2];
Animal[] animals = dogs;
```

The second assignment is legal:

```text
Dog <: Animal
        ↓
Dog[] <: Animal[]
```

It uses a widening reference conversion.

### Official JLS locations

- [JLS §4.10.3 — Subtyping among Array Types](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html#jls-4.10.3)
- [JLS §5.1.5 — Widening Reference Conversion](https://docs.oracle.com/javase/specs/jls/se25/html/jls-5.html#jls-5.1.5)

---

## 4. Compile-time type and runtime array type

Consider:

```java
Animal[] animals = new Dog[2];
```

There are two types to distinguish:

```text
Compile-time type of the variable: Animal[]
Runtime type of the array object:  Dog[]
```

The assignment changes only the type through which the reference is viewed. It does not transform the actual `Dog[]` object into an `Animal[]` object.

```text
animals: Animal[]
       │
       ▼
actual object: Dog[]
┌────────┬────────┐
│  null  │  null  │
└────────┴────────┘
```

### Official JLS location

- [JLS §10.2 — Array Variables](https://docs.oracle.com/javase/specs/jls/se25/html/jls-10.html#jls-10.2)

---

## 5. The runtime risk: `ArrayStoreException`

The following statement compiles:

```java
Animal[] animals = new Dog[2];
animals[0] = new Animal();
```

At compile time, the compiler sees that `animals` has type `Animal[]`, so storing an `Animal` appears valid.

At runtime, however, the actual object is a `Dog[]`. A plain `Animal` is not necessarily a `Dog`, so Java throws:

```text
java.lang.ArrayStoreException
```

The process is:

```text
Compile time:
animals is declared as Animal[]
new Animal() is compatible with Animal
→ compilation succeeds

Runtime:
the actual array object is Dog[]
new Animal() is not a Dog
→ ArrayStoreException
```

A safe write is:

```java
animals[0] = new Dog();
```

### Runnable example

```java
class Animal {
}

class Dog extends Animal {
}

public class ArrayCovarianceDemo {

    public static void main(String[] args) {
        Animal[] animals = new Dog[2];

        animals[0] = new Dog();    // Valid
        animals[1] = new Animal(); // ArrayStoreException at runtime
    }
}
```

### Official JLS location

- [JLS §10.5 — Array Store Exception](https://docs.oracle.com/javase/specs/jls/se25/html/jls-10.html#jls-10.5)

The official JLS example uses `Point[]` and `ColoredPoint[]` to demonstrate the same behavior.

---

## 6. Interface and abstract-class array example

Suppose:

```java
interface A {
}

abstract class B implements A {
}

class D implements A {
}

class E extends B {
}
```

Both of these array declarations are legal:

```java
A[] first = new A[2];
A[] second = new B[2];
```

### `A[] first = new A[2]`

The actual array type is `A[]`.

It may store references to objects of any class that implements `A`:

```java
first[0] = new D();
first[1] = new E();
```

### `A[] second = new B[2]`

The variable type is `A[]`, but the actual array type is `B[]`:

```text
Variable type:       A[]
Actual array object: B[]
```

It may store an `E`, because `E` extends `B`:

```java
second[0] = new E();
```

The following statement compiles but fails at runtime:

```java
second[1] = new D();
```

Why?

```text
Compile time:
second has type A[]
D implements A
→ accepted

Runtime:
the actual array is B[]
D does not extend B
→ ArrayStoreException
```

### Why is `new B[2]` legal when `B` is abstract?

`new B[2]` creates one array object whose component type is `B`. It does not create two instances of `B`.

Immediately after creation:

```text
B[] array
┌────────┬────────┐
│  null  │  null  │
└────────┴────────┘
```

The following is illegal:

```java
B value = new B(); // Compile-time error: B is abstract
```

But this is legal:

```java
B[] values = new B[2];
```

### Official JLS locations

- [JLS §10.1 — Array Types](https://docs.oracle.com/javase/specs/jls/se25/html/jls-10.html#jls-10.1)
- [JLS §10.2 — Array Variables](https://docs.oracle.com/javase/specs/jls/se25/html/jls-10.html#jls-10.2)
- [JLS §15.10.1 — Array Creation Expressions](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html#jls-15.10.1)
- [JLS §10.5 — Array Store Exception](https://docs.oracle.com/javase/specs/jls/se25/html/jls-10.html#jls-10.5)

---

## 7. Primitive arrays are not covariant with one another

This is illegal:

```java
long[] values = new int[3]; // Compile-time error
```

`int` and `long` have a numeric widening relationship, but that relationship does not make `int[]` a subtype of `long[]`.

Array covariance applies when the component types are reference types.

---

# Part II: Generic invariance

## 8. Java generic types are invariant by default

Even though:

```text
Dog <: Animal
```

Java does not conclude:

```text
List<Dog> <: List<Animal>
```

Therefore, this is illegal:

```java
List<Dog> dogs = new ArrayList<>();
List<Animal> animals = dogs; // Compile-time error
```

This behavior is called **invariance**.

### Why is this restriction necessary?

Imagine that the assignment were allowed:

```java
List<Dog> dogs = new ArrayList<>();
List<Animal> animals = dogs;
```

Then this would also be allowed:

```java
animals.add(new Cat());
```

But `animals` would refer to the same object as `dogs`, so a `Cat` would enter a `List<Dog>`. That would break type safety.

Java prevents this problem at compile time.

### Official JLS locations

The JLS does not use a single sentence titled “generics are invariant.” The result follows from the subtype rules for parameterized types and type-argument containment:

- [JLS §4.10 — Subtyping](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html#jls-4.10)
- [JLS §4.10.2 — Subtyping among Class and Interface Types](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html#jls-4.10.2)
- [JLS §4.5.1 — Type Arguments of Parameterized Types](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html#jls-4.5.1)

---

## 9. Arrays versus generic collections

| Arrays | Generic collections |
|---|---|
| `Dog[]` can be assigned to `Animal[]` | `List<Dog>` cannot be assigned to `List<Animal>` |
| Covariant | Invariant |
| Unsafe writes may fail at runtime | Unsafe relationship is rejected at compile time |
| Arrays retain their runtime component type | Generic type arguments are generally erased at runtime |

Example:

```java
Animal[] array = new Dog[1];
array[0] = new Animal(); // Compiles, then fails at runtime
```

```java
List<Dog> dogs = new ArrayList<>();
List<Animal> animals = dogs; // Fails at compile time
```

---

# Part III: Upper-bounded wildcards

## 10. `? extends T`: a covariant-style view

Although this is illegal:

```java
List<Animal> animals = new ArrayList<Dog>();
```

this is legal:

```java
List<Dog> dogs = new ArrayList<>();
List<? extends Animal> animals = dogs;
```

`List<? extends Animal>` means:

> A `List` whose element type is some unknown type that is `Animal` or a subtype of `Animal`.

The actual list could be:

```text
List<Animal>
List<Dog>
List<Cat>
```

More precisely:

```text
List<Dog> <: List<? extends Animal>
```

but:

```text
List<Dog> is not a subtype of List<Animal>
```

It is therefore more accurate to call `? extends` a **covariant-style wildcard view** rather than saying that `List` itself becomes covariant.

---

## 11. Reading from `? extends`

Reading is safe:

```java
List<? extends Animal> animals = List.of(new Dog());

Animal animal = animals.get(0);
```

The exact captured element type is unknown, but it is guaranteed to be an `Animal` subtype. Therefore, the result can safely be treated as `Animal`.

---

## 12. Writing to `? extends`

These writes are rejected:

```java
animals.add(new Animal()); // Compile-time error
animals.add(new Dog());    // Compile-time error
animals.add(new Cat());    // Compile-time error
```

The compiler does not know what the unknown type really is.

For example, the actual object could be a `List<Cat>`. Adding a `Dog` to it would be unsafe.

The only value that can generally be added is `null`:

```java
animals.add(null);
```

This is legal because `null` is compatible with every reference type, although adding `null` is usually not useful.

### Capture-conversion model

The compiler conceptually captures the wildcard as a fresh unknown type:

```text
CAP extends Animal
```

You know that `CAP` is some subtype of `Animal`, but you do not know whether it is `Dog`, `Cat`, or another subtype.

### Official JLS locations

- [JLS §4.5.1 — Type Arguments of Parameterized Types](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html#jls-4.5.1)
- [JLS §4.10.2 — Subtyping among Class and Interface Types](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html#jls-4.10.2)
- [JLS §5.1.10 — Capture Conversion](https://docs.oracle.com/javase/specs/jls/se25/html/jls-5.html#jls-5.1.10)

---

# Part IV: Lower-bounded wildcards

## 13. `? super T`: a contravariant-style view

Consider:

```java
List<? super Dog> destination;
```

It can refer to:

```text
List<Dog>
List<Animal>
List<Object>
```

For example:

```java
List<Animal> animals = new ArrayList<>();
List<? super Dog> destination = animals;
```

The subtype relationship can be viewed as:

```text
List<Animal> <: List<? super Dog>
```

This does not mean:

```text
List<Animal> <: List<Dog>
```

The wildcard provides a contravariant-style view at the use site.

---

## 14. Writing to `? super`

Writing a `Dog` is safe:

```java
destination.add(new Dog());
```

Writing a subclass of `Dog` is also safe.

This works because every possible underlying list can store a `Dog`:

```text
List<Dog>     can store Dog
List<Animal>  can store Dog
List<Object>  can store Dog
```

A plain `Animal` cannot be added safely:

```java
destination.add(new Animal()); // Compile-time error
```

The actual object might be a `List<Dog>`.

---

## 15. Reading from `? super`

A value read from the list can only be safely treated as `Object`:

```java
Object value = destination.get(0);
```

This is illegal:

```java
Dog dog = destination.get(0); // Compile-time error
```

The actual list might be a `List<Animal>` that already contains a `Cat`.

### Official JLS locations

- [JLS §4.5.1 — Type Arguments of Parameterized Types](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html#jls-4.5.1)
- [JLS §4.10.2 — Subtyping among Class and Interface Types](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html#jls-4.10.2)
- [JLS §5.1.10 — Capture Conversion](https://docs.oracle.com/javase/specs/jls/se25/html/jls-5.html#jls-5.1.10)

---

# Part V: PECS

## 16. Producer Extends, Consumer Super

A common memory rule is:

```text
PECS = Producer Extends, Consumer Super
```

- Use `? extends T` when a structure mainly **produces** values for your code.
- Use `? super T` when a structure mainly **consumes** values from your code.

PECS is a programming guideline, not the title of a JLS rule.

### Example: copying elements

```java
static <T> void copy(
        List<? extends T> source,
        List<? super T> destination) {

    for (T element : source) {
        destination.add(element);
    }
}
```

Explanation:

```text
source produces T
→ ? extends T

destination consumes T
→ ? super T
```

Usage:

```java
List<Dog> dogs = List.of(new Dog());
List<Animal> animals = new ArrayList<>();

copy(dogs, animals);
```

---

# Part VI: Covariant return types

## 17. An overriding method may return a more specific reference type

A superclass method may return `Animal`:

```java
class AnimalFactory {

    public Animal create() {
        return new Animal();
    }
}
```

A subclass may override it and return `Dog`:

```java
class DogFactory extends AnimalFactory {

    @Override
    public Dog create() {
        return new Dog();
    }
}
```

Because:

```text
Dog <: Animal
```

the return type has been specialized in the same direction as the subtype relationship.

This is called a **covariant return type**.

### Why it is useful

Without a covariant return type:

```java
Animal animal = new DogFactory().create();
```

With the more specific return type:

```java
Dog dog = new DogFactory().create();
```

No cast is required.

### Official JLS locations

- [JLS §8.4.5 — Method Result](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.4.5)
- [JLS §8.4.8.3 — Requirements in Overriding and Hiding](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.4.8.3)

JLS §8.4.5 explicitly says that return-type substitutability supports **covariant returns**.

---

## 18. Primitive return types are not covariant

This is illegal:

```java
class Parent {

    public long value() {
        return 1L;
    }
}

class Child extends Parent {

    @Override
    public int value() { // Compile-time error
        return 1;
    }
}
```

Although an `int` value can be widened to `long` in numeric contexts, primitive return types do not participate in covariant return typing. For overriding, the primitive return type must match as required by the method-result rules.

---

# Part VII: Method parameters are not covariant in overriding

## 19. Narrowing a parameter creates a different method signature

Superclass:

```java
class Parent {

    public void handle(Animal animal) {
    }
}
```

Subclass:

```java
class Child extends Parent {

    public void handle(Dog dog) {
    }
}
```

`handle(Dog)` does not override `handle(Animal)`. The parameter types differ, so the methods have different signatures.

The subclass now has an overload:

```text
handle(Animal)  inherited from Parent
handle(Dog)     declared in Child
```

Adding `@Override` reveals the mistake:

```java
class Child extends Parent {

    @Override
    public void handle(Dog dog) { // Compile-time error
    }
}
```

A valid override keeps the compatible signature:

```java
class Child extends Parent {

    @Override
    public void handle(Animal animal) {
    }
}
```

### Why a covariant parameter would be unsafe

A `Child` object may be referenced through a `Parent` variable:

```java
Parent parent = new Child();
parent.handle(new Cat());
```

The contract of `Parent.handle(Animal)` says that every `Animal` is accepted. A subclass cannot replace that contract with a method that accepts only `Dog`.

### Official JLS locations

- [JLS §8.4.2 — Method Signature](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.4.2)
- [JLS §8.4.8.1 — Overriding by Instance Methods](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.4.8.1)
- [JLS §8.4.9 — Overloading](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.4.9)

---

# Part VIII: Complete comparison example

```java
import java.util.ArrayList;
import java.util.List;

class Animal {
}

class Dog extends Animal {
}

class Puppy extends Dog {
}

class Cat extends Animal {
}

class AnimalFactory {

    public Animal create() {
        return new Animal();
    }

    public void handle(Animal animal) {
        System.out.println("Handling an animal");
    }
}

class DogFactory extends AnimalFactory {

    @Override
    public Dog create() {
        return new Dog();
    }

    @Override
    public void handle(Animal animal) {
        System.out.println("Handling an animal in DogFactory");
    }

    public void handle(Dog dog) {
        System.out.println("Handling a dog");
    }
}

public class VarianceDemo {

    public static void main(String[] args) {
        demonstrateArrayCovariance();
        demonstrateGenericInvariance();
        demonstrateExtendsWildcard();
        demonstrateSuperWildcard();
        demonstrateCovariantReturn();
    }

    private static void demonstrateArrayCovariance() {
        Animal[] animals = new Dog[2];

        animals[0] = new Dog();

        try {
            animals[1] = new Cat();
        } catch (ArrayStoreException exception) {
            System.out.println(exception);
        }
    }

    private static void demonstrateGenericInvariance() {
        List<Dog> dogs = new ArrayList<>();

        // List<Animal> animals = dogs;
        // Compile-time error: List<Dog> is not a subtype of List<Animal>.
    }

    private static void demonstrateExtendsWildcard() {
        List<Dog> dogs = List.of(new Dog());
        List<? extends Animal> source = dogs;

        Animal animal = source.get(0);
        System.out.println(animal.getClass().getSimpleName());

        // source.add(new Dog());
        // Compile-time error: the captured element type is unknown.
    }

    private static void demonstrateSuperWildcard() {
        List<Animal> animals = new ArrayList<>();
        List<? super Dog> destination = animals;

        destination.add(new Dog());
        destination.add(new Puppy());

        Object value = destination.get(0);
        System.out.println(value.getClass().getSimpleName());
    }

    private static void demonstrateCovariantReturn() {
        DogFactory factory = new DogFactory();
        Dog dog = factory.create();

        System.out.println(dog.getClass().getSimpleName());
    }
}
```

---

# Part IX: Official JLS reference map

| Topic | Java Language Specification, Java SE 25 |
|---|---|
| General subtype relationships | [§4.10 Subtyping](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html#jls-4.10) |
| Parameterized-type subtype rules | [§4.10.2 Subtyping among Class and Interface Types](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html#jls-4.10.2) |
| Array covariance | [§4.10.3 Subtyping among Array Types](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html#jls-4.10.3) |
| `? extends` and `? super` | [§4.5.1 Type Arguments of Parameterized Types](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html#jls-4.5.1) |
| Widening reference conversion | [§5.1.5 Widening Reference Conversion](https://docs.oracle.com/javase/specs/jls/se25/html/jls-5.html#jls-5.1.5) |
| Wildcard capture | [§5.1.10 Capture Conversion](https://docs.oracle.com/javase/specs/jls/se25/html/jls-5.html#jls-5.1.10) |
| Assignment compatibility | [§5.2 Assignment Contexts](https://docs.oracle.com/javase/specs/jls/se25/html/jls-5.html#jls-5.2) |
| Interface and abstract-class arrays | [§10.1 Array Types](https://docs.oracle.com/javase/specs/jls/se25/html/jls-10.html#jls-10.1) |
| Array variables and actual array objects | [§10.2 Array Variables](https://docs.oracle.com/javase/specs/jls/se25/html/jls-10.html#jls-10.2) |
| Runtime array store checking | [§10.5 Array Store Exception](https://docs.oracle.com/javase/specs/jls/se25/html/jls-10.html#jls-10.5) |
| Array creation expressions | [§15.10.1 Array Creation Expressions](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html#jls-15.10.1) |
| Covariant method returns | [§8.4.5 Method Result](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.4.5) |
| Return-type requirements in overriding | [§8.4.8.3 Requirements in Overriding and Hiding](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.4.8.3) |
| Method signatures | [§8.4.2 Method Signature](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.4.2) |
| Overriding instance methods | [§8.4.8.1 Overriding](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.4.8.1) |
| Method overloading | [§8.4.9 Overloading](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.4.9) |

---

# Part X: Final memory framework

```text
1. Arrays are covariant.
   Dog[] can be assigned to Animal[].
   Unsafe writes may cause ArrayStoreException at runtime.

2. Generic types are invariant.
   List<Dog> cannot be assigned to List<Animal>.

3. ? extends T provides a covariant-style read view.
   Read values as T.
   Usually do not add non-null values.

4. ? super T provides a contravariant-style write view.
   Add T and its subtypes.
   Read values safely only as Object.

5. PECS:
   Producer Extends, Consumer Super.

6. Overridden methods may use covariant reference return types.
   A parent method returning Animal may be overridden with Dog.

7. Method parameters cannot be narrowed to create an override.
   handle(Dog) does not override handle(Animal); it is a different signature.
```
