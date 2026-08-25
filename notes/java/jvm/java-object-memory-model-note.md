# Java Object Memory Model 深入理解

## 1. 为什么学习 Class、Object、Instance、Variable？

Java 初学阶段经常会混淆：

- Class（类）
- Object（对象）
- Instance（实例）
- Variable（变量）
- Reference Variable（引用变量）
- Instance Variable（实例变量）
- Heap（堆）
- String Constant Pool（字符串常量池）

这些概念实际上对应 Java 语言规范和 JVM 内存模型中的不同层次。

---

# 2. Class 是什么？

例如：

```java
class Person {

    String name;

    int age;
}
```

Person 是一个 Class。

可以把 Class 理解成：

> 一张设计图（blueprint）

它描述未来创建出来的对象应该拥有：

- 哪些字段
- 哪些方法

例如：

```
Person Class

Fields:

name
age
```

注意：

此时：

```java
String name;
int age;
```

只是定义，并没有产生真实的数据。

---

# 3. Object / Instance 是什么？

代码：

```java
new Person();
```

会触发 JVM 创建一个对象。

例如：

```java
Person person = new Person();
```

其中：

```
new Person()
```

不是对象本身，而是：

> 创建对象的表达式

真正的对象存在于 Heap 中。

例如：

```
Heap

Person Object

+-------------+
| name=null   |
| age=0       |
+-------------+
```

这个对象就是：

- Object
- Instance of Person

Java 中通常：

```
Object ≈ Instance
```

---

# 4. Person person = new Person() 分解

代码：

```java
Person person = new Person();
```

可以拆成三个部分：

```
Person          person          new Person()

类型            变量            创建对象
```

---

## 4.1 Person

第一个 Person 是变量类型。

类似：

```java
int number;
```

中的 int。

它表示：

该变量可以保存 Person 类型对象的引用。

它不是对象。

---

## 4.2 person

person 是变量名。

如果在方法中：

```java
public static void main(String[] args){

    Person person = new Person();

}
```

那么：

person 是 local variable。

同时它保存对象引用：

```
Stack

person
 |
 |
 v

Heap 中的 Person Object
```

---

## 4.3 new Person()

new Person() 创建对象。

JVM：

1. 在 Heap 申请空间
2. 创建对象
3. 初始化字段
4. 返回对象引用

---

# 5. Stack 和 Heap

代码：

```java
Person person = new Person();
```

运行后：

```
Stack

main()

person
0x1000



Heap

0x1000

Person Object

+-------------+
| name=null   |
| age=0       |
+-------------+
```

person 保存的是对象引用。

不是对象本身。

---

# 6. Heap 是什么？

Heap 是 JVM 用来存放对象实例的区域。

例如：

```java
Person p1 = new Person();

Person p2 = new Person();
```

Heap:

```
Person Object #1

name
age



Person Object #2

name
age
```

两个对象拥有独立的数据。

---

# 7. Instance Variable（实例变量）

代码：

```java
class Person {

    String name;

    int age;
}
```

其中：

```
name
age
```

叫：

instance variables。

原因：

它们没有 static。

每创建一个对象：

都会创建一份属于该对象的变量。

例如：

```
Person Object #1

name = Tom
age = 0


Person Object #2

name = Jerry
age = 0
```

---

# 8. String 引用关系

代码：

```java
class Person {

    String name;
}


Person p1 = new Person();

p1.name = "Tom";
```

内存：

```
Stack

p1
 |
 v


Heap

Person Object

+----------------+
| name ----------+
+----------------+

        |
        v

String Object

+-------------+
| "Tom"       |
+-------------+
```

name 并不保存字符 Tom。

它保存 String 对象引用。

---

# 9. String Constant Pool

Java 对字符串字面量进行了优化。

例如：

```java
String a = "Tom";

String b = "Tom";
```

JVM 会复用同一个字符串对象。

```
Heap

String Constant Pool

+-------------+
| "Tom"       |
+-------------+

     ^
     |
   a,b
```

所以：

```java
a == b
```

结果：

```
true
```

---

# 10. String Pool 的位置

需要区分：

## JVM 规范

Runtime Constant Pool 属于 JVM Runtime Data Area。

## HotSpot 实现

Java 7 以后：

String Pool 位于 Heap。

因此现代 Java：

```
Heap

Objects

String Constant Pool
```

---

# 11. 完整对象关系图

```
Method Area

Person Class

name
age



Stack

p1
 |
 |
 v



Heap


Person Object

name = reference
age = 0


        |
        v


String Pool

"Tom"
```

---

# 12. 现实类比

Class：

像建筑设计图。


Object：

按照图纸建出来的房子。


Reference Variable：

房子的门牌号或者钥匙。


Instance Variable：

房子里面的家具。

例如：

```
Person Class

设计图


new Person()

建造房子


person

钥匙


name age

房间里的东西
```

---

# 13. 总结

|概念|含义|
|-|-|
|Class|对象设计模板|
|Object|Heap 中真实存在的数据|
|Instance|某个 Class 创建出的对象|
|Reference Variable|保存对象引用的变量|
|Instance Variable|对象内部字段|
|Heap|保存对象的内存区域|
|String Object|字符串对象|
|String Constant Pool|字符串字面量共享区域|

---

# Official Documentation

## Java Language Specification SE 25

Variables:
https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html#jls-4.12

Instance Variables / Static Fields:
https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.3.1.1

Creation of New Class Instances:
https://docs.oracle.com/javase/specs/jls/se25/html/jls-12.html#jls-12.5

String Literals:
https://docs.oracle.com/javase/specs/jls/se25/html/jls-3.html#jls-3.10.5

## Java Virtual Machine Specification SE 25

Runtime Constant Pool:
https://docs.oracle.com/javase/specs/jvms/se25/html/jvms-5.html#jvms-5.1

## Java String API

String.intern():
https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/String.html#intern() 
