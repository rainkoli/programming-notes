这句话**基本正确**，但可以表述得更准确一些：

> **实例变量属于某个具体对象，每创建一个对象，该对象都会拥有自己的一份实例变量；类变量由 `static` 修饰，属于类本身，同一个类的所有实例共享同一份类变量。**

## Oracle 官方规范依据

### 1. JLS §4.12.3：Kinds of Variables

官方链接：

[Java SE 25 JLS §4.12.3 — Kinds of Variables](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html#jls-4.12.3)

该节规定：

- 使用 `static` 声明的字段是 **class variable（类变量）**。
- 没有使用 `static` 声明的字段是 **instance variable（实例变量）**。
- 每创建一个类或其子类的新对象，就会为该对象创建一份相应的实例变量。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html))

### 2. JLS §8.3.1.1：static Fields

官方链接：

[Java SE 25 JLS §8.3.1.1 — static Fields](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.3.1.1)

这一节解释得更加直接：

- 一个 `static` 字段只有**一份**，无论最终创建了多少个该类的实例，甚至一个实例都没有创建。
- 对于非 `static` 字段，每创建一个新对象，都会创建一份与该对象关联的实例变量。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html))

## 代码示例

```java
class Person {
    String name;        // 实例变量
    static int count;   // 类变量
}
```

创建两个对象：

```java
Person p1 = new Person();
Person p2 = new Person();

p1.name = "Alice";
p2.name = "Bob";

Person.count = 2;
```

可以理解为：

```text
Person 类
└── static count = 2        所有 Person 对象共享一份

p1 对象
└── name = "Alice"          p1 自己的一份

p2 对象
└── name = "Bob"            p2 自己的一份
```

因此：

```java
p1.name
p2.name
```

是两个不同对象中的两份变量。

而：

```java
Person.count
```

整个 `Person` 类只有一份。

## 推荐表述

你原来的表述：

> 实例变量隶属于各个实例，类变量属于类整体。

作为入门理解是正确的。更符合 JLS 的表述是：

> **实例变量与每个具体实例相关联；类变量是 `static` 字段，一个类声明中的某个类变量只有一份，与创建多少个实例无关。**

这里不要把“属于类”理解成具体 JVM 内存布局。JLS 规定的是 Java 语言语义，并不要求你将类变量简单理解为“存放在某个类对象里面”。