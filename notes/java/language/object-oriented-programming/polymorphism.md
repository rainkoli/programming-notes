# 1

## 1. 什么是多态

Java 中的多态可以理解为：

> 同一个父类类型的引用，可以指向不同的子类对象；调用同一个被重写的方法时，会根据对象的实际类型执行不同的实现。

你这个案例中：

```java
public void feed(Animal animal)
```

`feed()` 方法声明接收的是 `Animal`，但调用时既可以传入 `Cat`，也可以传入 `Dog`：

```java
person.feed(cat);
person.feed(dog);
```

因为：

```java
Cat extends Animal
Dog extends Animal
```

所以 `Cat` 和 `Dog` 都属于 `Animal`。

多态通常需要满足三个条件：

1. 存在继承关系。
2. 子类重写父类方法。
3. 父类类型的引用指向子类对象。

你的代码完全满足这些条件。

------

## 2. 类之间的关系

代码的继承关系是：

```text
              Animal
              eat()
             /     \
            /       \
         Cat         Dog
         eat()       eat()
         catchMouse() watch()
```

父类 `Animal` 定义了所有动物都可以进行的行为：

```java
public class Animal {
    public void eat() {

    }
}
```

`Cat` 重写了 `eat()`：

```java
public class Cat extends Animal {
    @Override
    public void eat() {
        System.out.println("Cat eat fish");
    }

    public void catchMouse() {
        System.out.println("Cat catch mouse");
    }
}
```

`Dog` 也重写了 `eat()`：

```java
public class Dog extends Animal {
    @Override
    public void eat() {
        System.out.println("Dog eat bone");
    }

    public void watch() {
        System.out.println("Dog watch door");
    }
}
```

建议在重写的方法上添加 `@Override`。这样如果方法名或参数写错，编译器可以及时提醒。

------

## 3. `person.feed(cat)` 的执行过程

首先创建对象：

```java
Person person = new Person();
Cat cat = new Cat();
```

内存关系可以简单表示为：

```text
cat ──────────→ Cat对象
```

调用：

```java
person.feed(cat);
```

但 `feed()` 的参数类型是：

```java
public void feed(Animal animal)
```

于是传参时发生了自动向上转型：

```java
Animal animal = cat;
```

也可以理解为：

```java
Animal animal = new Cat();
```

此时：

```text
animal的声明类型：Animal
animal的实际类型：Cat
```

引用关系是：

```text
cat ─────┐
         ├──→ 同一个Cat对象
animal ──┘
```

传参不会创建新的 `Cat` 对象，只是把 `cat` 中保存的对象引用复制给形参 `animal`。

------

## 4. 为什么 `animal.eat()` 执行的是 `Cat.eat()`

进入 `feed()`：

```java
public void feed(Animal animal) {
    animal.eat();
}
```

虽然 `animal` 的声明类型是 `Animal`，但是它实际指向的是 `Cat` 对象。

因此：

```java
animal.eat();
```

最终执行的是：

```java
Cat.eat()
```

输出：

```text
Cat eat fish
```

这叫作：

- 方法重写
- 动态绑定
- 动态方法分派
- 运行时多态

可以暂时记成：

> 对于被重写的实例方法，编译时检查引用的声明类型，运行时根据对象的实际类型决定调用哪个方法。

也就是常说的：

```text
编译看左边，运行看右边
```

这里“左边”是：

```java
Animal animal
```

“右边”是它实际指向的：

```java
new Cat()
```

这句话主要适用于被重写的实例方法，并不是所有成员都遵循这个规则。

------

## 5. 为什么不能直接调用 `catchMouse()`

虽然 `animal` 实际指向 `Cat`，但它的声明类型依然是 `Animal`：

```java
Animal animal = new Cat();
```

因此下面的代码不能通过编译：

```java
animal.catchMouse();
```

因为编译器只会根据声明类型 `Animal` 检查可用的方法，而 `Animal` 中没有：

```java
catchMouse()
```

所以必须先把 `Animal` 类型的引用向下转型为 `Cat`：

```java
Cat cat = (Cat) animal;
cat.catchMouse();
```

这个过程叫作**向下转型**。

需要注意，强制类型转换没有把对象变成另一个对象：

```java
Cat cat = (Cat) animal;
```

它只是告诉编译器：

> 我确认 `animal` 指向的对象是一个 `Cat`，请把这个引用当成 `Cat` 类型使用。

------

## 6. `instanceof` 的作用

如果直接进行错误的向下转型：

```java
Animal animal = new Dog();
Cat cat = (Cat) animal;
```

代码可以通过编译，但运行时会出现：

```text
ClassCastException
```

因为真正的对象是 `Dog`，不能把它当成 `Cat`。

所以你的代码先进行了判断：

```java
if (animal instanceof Cat) {
    Cat cat = (Cat) animal;
    cat.catchMouse();
}
```

含义是：

> 判断 `animal` 指向的对象是否是 `Cat`，只有确定是 `Cat` 后才进行类型转换。

同理：

```java
if (animal instanceof Dog) {
    Dog dog = (Dog) animal;
    dog.watch();
}
```

------

## 7. 两次调用的完整执行过程

### 第一次：`person.feed(cat)`

```java
Cat cat = new Cat();
person.feed(cat);
```

进入方法后：

```text
animal实际指向Cat对象
```

执行过程：

```text
animal.eat()
    ↓
实际对象是Cat
    ↓
执行Cat.eat()
    ↓
输出：Cat eat fish
```

接下来：

```java
animal instanceof Cat
```

结果为：

```text
true
```

因此执行：

```java
Cat cat = (Cat) animal;
cat.catchMouse();
```

输出：

```text
Cat catch mouse
```

然后：

```java
animal instanceof Dog
```

结果是：

```text
false
```

所以不会执行 `Dog` 部分。

------

### 第二次：`person.feed(dog)`

```java
Dog dog = new Dog();
person.feed(dog);
```

执行过程：

```text
animal实际指向Dog对象
    ↓
animal.eat()
    ↓
执行Dog.eat()
    ↓
输出：Dog eat bone
```

判断：

```java
animal instanceof Cat
```

结果为 `false`。

判断：

```java
animal instanceof Dog
```

结果为 `true`，于是执行：

```java
Dog dog = (Dog) animal;
dog.watch();
```

输出：

```text
Dog watch door
```

因此整个程序的最终输出为：

```text
Cat eat fish
Cat catch mouse
Dog eat bone
Dog watch door
```

------

## 8. 使用新版 `instanceof` 写法

在较新的 Java 版本中，可以使用模式匹配，判断和转换一起完成：

```java
public void feed(Animal animal) {
    animal.eat();

    if (animal instanceof Cat cat) {
        cat.catchMouse();
    }

    if (animal instanceof Dog dog) {
        dog.watch();
    }
}
```

这里：

```java
animal instanceof Cat cat
```

同时完成两件事：

```text
1. 判断animal是否指向Cat对象
2. 如果是，自动创建Cat类型的变量cat
```

因此不再需要手动写：

```java
Cat cat = (Cat) animal;
```

------

## 9. 这个案例中哪里体现了多态

真正体现多态的是：

```java
public void feed(Animal animal) {
    animal.eat();
}
```

同一条代码：

```java
animal.eat();
```

当传入 `Cat` 时执行：

```java
Cat.eat()
```

当传入 `Dog` 时执行：

```java
Dog.eat()
```

也就是说：

```text
相同的方法调用形式
        ↓
根据实际对象不同
        ↓
表现出不同的执行结果
```

而下面这部分主要展示的是**向下转型**，不是多态本身：

```java
if (animal instanceof Cat) {
    Cat cat = (Cat) animal;
    cat.catchMouse();
}
```

------

## 10. 更符合多态思想的设计

如果希望减少 `instanceof` 和强制类型转换，可以把“吃完之后执行的行为”也抽象到父类中：

```java
public abstract class Animal {
    public abstract void eat();

    public abstract void doSpecialAction();
}
```

`Cat`：

```java
public class Cat extends Animal {
    @Override
    public void eat() {
        System.out.println("Cat eat fish");
    }

    @Override
    public void doSpecialAction() {
        System.out.println("Cat catch mouse");
    }
}
```

`Dog`：

```java
public class Dog extends Animal {
    @Override
    public void eat() {
        System.out.println("Dog eat bone");
    }

    @Override
    public void doSpecialAction() {
        System.out.println("Dog watch door");
    }
}
```

那么 `Person` 就不需要判断动物的具体类型：

```java
public void feed(Animal animal) {
    animal.eat();
    animal.doSpecialAction();
}
```

无论传入猫还是狗，都只需要：

```java
person.feed(new Cat());
person.feed(new Dog());
```

这更能体现多态的核心思想：

> 使用父类提供统一标准，把具体行为交给不同子类实现。

***

# 2

在 **Java Language Specification, Java SE 25 Edition** 中，没有一个单独标题为“Polymorphism（多态）”的章节。你这个案例中的“多态”由**继承、向上转型、方法重写、编译期方法查找和运行期动态分派**等规则共同构成。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/index.html))

## 对应位置总览

| 图中知识点                 | JLS 位置                                      | 对应代码                              |
| -------------------------- | --------------------------------------------- | ------------------------------------- |
| 父类与子类、`extends`      | **§8.1.4 Superclasses and Subclasses**        | `Cat extends Animal`                  |
| 父类变量引用子类对象       | **§4.12.2 Variables of Reference Type**       | `Animal animal = new Cat()`           |
| 子类实参传给父类形参       | **§5.3 Invocation Contexts**                  | `feed(cat)`                           |
| 向上转型                   | **§5.1.5 Widening Reference Conversion**      | `Cat → Animal`                        |
| 方法重写                   | **§8.4.8.1 Overriding (by Instance Methods)** | `Cat.eat()` 重写 `Animal.eat()`       |
| `@Override` 注解           | **§9.6.4.4 @Override**                        | `@Override public void eat()`         |
| 编译期根据声明类型查找方法 | **§15.12.1 Compile-Time Step 1**              | 为什么 `animal.catchMouse()` 编译失败 |
| 运行期动态方法分派         | **§15.12.4.4 Locate Method to Invoke**        | `animal.eat()` 最终执行 `Cat.eat()`   |
| `instanceof` 判断          | **§15.20.2 The instanceof Operator**          | `animal instanceof Cat`               |
| 强制类型转换语法           | **§15.16 Cast Expressions**                   | `(Cat) animal`                        |
| 向下转型及异常             | **§5.1.6 Narrowing Reference Conversion**     | 错误转换可能抛出 `ClassCastException` |

------

## 1. `Cat extends Animal`：继承关系

位置：

> **JLS §8.1.4 — Superclasses and Subclasses**

这一节说明，普通类声明中的 `extends` 子句指定该类的直接父类，并定义什么是直接父类、直接子类、父类和子类。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html))

对应你的代码：

```java
public class Cat extends Animal {
}

public class Dog extends Animal {
}
```

因此：

```text
Animal
├── Cat
└── Dog
```

`Animal` 是 `Cat` 和 `Dog` 的直接父类；`Cat` 和 `Dog` 是 `Animal` 的直接子类。

------

## 2. 父类变量为什么可以指向子类对象

位置：

> **JLS §4.12.2 — Variables of Reference Type**

JLS 明确规定：

> 类类型 `T` 的变量，可以保存 `null`，也可以保存对 `T` 的实例或者 `T` 的任意子类实例的引用。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html))

因此下面的代码合法：

```java
Animal animal1 = new Cat();
Animal animal2 = new Dog();
```

对应关系是：

```text
变量的声明类型       对象的实际运行时类型
Animal animal1  →   Cat
Animal animal2  →   Dog
```

这正是多态引用的类型基础。

需要注意的是，这里存储的是**对象引用**。它不是把 `Cat` 对象复制成一个 `Animal` 对象，也不是把对象本身变成了 `Animal`。

------

## 3. `person.feed(cat)`：方法传参中的向上转型

你的方法是：

```java
public void feed(Animal animal)
```

调用时传入：

```java
Cat cat = new Cat();
person.feed(cat);
```

这里涉及两节。

### §5.3 Invocation Contexts

JLS §5.3 规定，方法调用时，实参值会被赋给对应的形参变量，并允许发生特定的类型转换，其中包括**扩大引用转换**。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-5.html))

因此可以把这个传参过程理解为：

```java
Animal animal = cat;
```

### §5.1.5 Widening Reference Conversion

JLS §5.1.5 规定：当引用类型 `S` 是引用类型 `T` 的子类型时，可以从 `S` 转换到 `T`。这种转换不需要特殊的运行时操作，也不会因为该转换抛出运行时异常。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-5.html))

你的代码中：

```text
Cat 是 Animal 的子类型
Dog 是 Animal 的子类型
```

所以都可以自动转换：

```java
Cat → Animal
Dog → Animal
```

这就是通常所说的**向上转型**。

------

## 4. `Cat.eat()` 和 `Dog.eat()`：方法重写

位置：

> **JLS §8.4.8.1 — Overriding (by Instance Methods)**

这一节定义了实例方法在什么条件下构成重写。核心条件包括：

- 子类 `C` 是父类 `A` 的子类；
- 子类方法与父类方法具有满足要求的方法签名；
- 父类方法具有允许重写的访问权限。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html))

在你的代码中：

```java
public class Animal {
    public void eat() {
    }
}
```

`Cat` 声明：

```java
public void eat() {
    System.out.println("Cat eat fish");
}
```

`Dog` 声明：

```java
public void eat() {
    System.out.println("Dog eat bone");
}
```

因此：

```text
Cat.eat() 重写 Animal.eat()
Dog.eat() 重写 Animal.eat()
```

------

## 5. `@Override` 在哪里定义

位置：

> **JLS §9.6.4.4 — @Override**

JLS 说明，`@Override` 用来帮助程序员尽早发现“本来想重写，实际上却写成了重载或其他方法”的错误。如果标注的方法不符合重写等规定，就会发生编译错误。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-9.html))

因此建议写成：

```java
public class Cat extends Animal {
    @Override
    public void eat() {
        System.out.println("Cat eat fish");
    }
}
```

以及：

```java
public class Dog extends Animal {
    @Override
    public void eat() {
        System.out.println("Dog eat bone");
    }
}
```

`@Override` 并不产生重写关系；它只是让编译器检查这个方法是否真的构成重写。

------

## 6. 为什么 `animal.catchMouse()` 不能直接调用

位置：

> **JLS §15.12.1 — Compile-Time Step 1: Determine Type to Search**

这一节规定，在编译一个方法调用时，编译器首先确定要在哪个类型中查找方法。

对于：

```java
animal.catchMouse();
```

如果 `animal` 是一个变量名，那么 JLS 规定，编译器使用该变量的**声明类型**来搜索方法。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html))

假设：

```java
Animal animal = new Cat();
```

那么：

```text
animal 的声明类型：Animal
animal 指向对象的实际类型：Cat
```

编译器检查：

```java
animal.catchMouse();
```

时会先去 `Animal` 类型中寻找 `catchMouse()`。但 `Animal` 没有该方法：

```java
public class Animal {
    public void eat() {
    }
}
```

所以不能通过编译。

这就是“编译看左边”比较准确的 JLS 依据：

```java
Animal animal = new Cat();
// ↑ 编译期使用这个声明类型检查可调用的方法
```

------

## 7. 为什么 `animal.eat()` 执行子类方法

最核心的位置：

> **JLS §15.12.4.4 — Locate Method to Invoke**

JLS 规定，对于允许重写的实例方法，需要执行**动态方法查找**。查找从目标对象的**实际运行时类**开始。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html))

例如：

```java
Animal animal = new Cat();
animal.eat();
```

分为两个阶段。

### 编译阶段

根据 `animal` 的声明类型 `Animal`，检查：

```text
Animal 中是否存在 eat()
```

存在，所以编译通过。

### 运行阶段

运行时发现：

```text
animal 实际指向 Cat 对象
```

动态查找从 `Cat` 开始。如果 `Cat` 中存在重写后的 `eat()`，就调用该方法。JLS 明确说明，动态查找从目标对象的实际运行时类开始，并优先选择重写的方法。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html))

所以执行：

```java
Cat.eat()
```

同理：

```java
Animal animal = new Dog();
animal.eat();
```

执行：

```java
Dog.eat()
```

JLS 自己也提供了一个与此几乎相同的示例：

```java
Point p = new ColoredPoint();
p.move(...);
```

其中被重写的方法根据对象的运行时类型选择。规范将其描述为方法在运行时“较晚”决定，而不是仅根据编译期类型提前决定。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html))

因此，“编译看左边，运行看右边”更完整地应写为：

> 对于被重写的实例方法，编译期根据引用表达式的声明类型检查方法是否可调用；运行期根据对象的实际类型动态选择最终实现。

------

## 8. `instanceof` 在哪里

位置：

> **JLS §15.20.2 — The instanceof Operator**

这一节说明，`instanceof` 可以执行：

- 类型比较；
- 模式匹配。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html))

你的代码：

```java
if (animal instanceof Cat) {
    Cat cat = (Cat) animal;
    cat.catchMouse();
}
```

运行时规则是：

- 如果 `animal` 为 `null`，结果为 `false`；
- 如果它不是 `null`，并且可以安全转换成 `Cat` 而不抛出 `ClassCastException`，结果为 `true`；
- 否则结果为 `false`。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html))

所以：

```java
Animal animal = new Cat();

animal instanceof Cat    // true
animal instanceof Dog    // false
```

------

## 9. 新版 `instanceof` 模式匹配

你的代码可以写成：

```java
if (animal instanceof Cat cat) {
    cat.catchMouse();
}
```

这仍然主要对应：

> **JLS §15.20.2 — The instanceof Operator**

JLS 将右侧为 `Pattern` 的形式称为模式匹配。当模式匹配成功时，模式变量会被初始化。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html))

因此：

```java
animal instanceof Cat cat
```

同时完成：

```text
1. 判断 animal 指向的对象是否符合 Cat 类型
2. 匹配成功后创建并初始化变量 cat
```

------

## 10. `(Cat) animal`：向下转型

这里要看三个位置。

### §15.16 Cast Expressions

这一节定义了强制类型转换表达式：

```java
(Cat) animal
```

对于引用类型，转换会在运行时检查引用指向的对象是否与目标类型兼容。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html))

### §5.1.6 Narrowing Reference Conversion

从较一般的引用类型转换到更具体的引用类型，属于**缩小引用转换**，也就是通常说的向下转型。某些转换需要在运行时检查；检查失败时会抛出 `ClassCastException`。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-5.html))

正确情况：

```java
Animal animal = new Cat();
Cat cat = (Cat) animal;
```

错误情况：

```java
Animal animal = new Dog();
Cat cat = (Cat) animal;
```

第二段代码可能通过编译，但运行时会因为实际对象是 `Dog` 而抛出：

```text
ClassCastException
```

### 强制转换不会改变对象的实际类型

JLS 在动态方法调用示例中特别说明：强制转换不会改变对象所属的类，只会检查对象的类是否与指定类型兼容。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html))

所以：

```java
Cat cat = (Cat) animal;
```

并不是把一个 `Animal` 对象“变成”新的 `Cat` 对象，而是得到一个声明类型为 `Cat`、仍然指向原对象的引用。

------

## 建议重点阅读顺序

对于你当前这个案例，最重要的是依次阅读：

1. **§4.12.2 Variables of Reference Type**：父类变量为什么能指向子类对象。
2. **§5.1.5 Widening Reference Conversion**：向上转型。
3. **§8.4.8.1 Overriding**：方法重写。
4. **§15.12.1 Compile-Time Step 1**：编译时根据什么类型检查方法。
5. **§15.12.4.4 Locate Method to Invoke**：运行时如何选择子类方法。
6. **§15.20.2 instanceof**：类型判断。
7. **§15.16 和 §5.1.6**：强制转换和向下转型。

其中，真正解释 Java 运行时多态核心机制的是：

> **§15.12.4.4 Locate Method to Invoke**

你可以把整个案例浓缩成：

```text
§8.1.4：Cat、Dog 是 Animal 的子类
        ↓
§4.12.2、§5.1.5：Animal 引用可以指向 Cat、Dog
        ↓
§8.4.8.1：Cat、Dog 重写 eat()
        ↓
§15.12.1：编译时从 Animal 检查是否有 eat()
        ↓
§15.12.4.4：运行时根据实际对象调用 Cat.eat() 或 Dog.eat()
```