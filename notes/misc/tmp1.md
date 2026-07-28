int a = 2

int b = a ++ + -- a + a --

\&\&





**`jeandle/jeandle-jdk`**

## 它是做什么的？

Jeandle 是一个面向 Java 的**即时编译器（JIT Compiler）**。它建立在 **OpenJDK** 基础上，并利用 **LLVM** 编译器基础设施生成本地机器码，目标是获得更强的编译优化和更高的程序执行性能。目前项目仍处于开发阶段。([GitHub](https://github.com/jeandle/jeandle-jdk))

可以简单理解为：

```text
Java 源代码
    ↓ javac 编译
.class 字节码
    ↓ JVM 运行并发现热点方法
Jeandle JIT 编译器
    ↓ 使用 LLVM 优化、生成代码
本地机器码
    ↓
CPU 执行
```

这里需要区分：

- **javac**：在程序运行前，把 `.java` 编译成 `.class` 字节码。
- **Jeandle**：在程序运行过程中，把需要频繁执行的 Java 方法编译为机器码。
- **LLVM**：负责中间表示、编译优化以及机器码生成。

所以 Jeandle 研究的不是普通的 Java 语法编译，而是 **JVM 运行期间的 JIT 编译**。

## `jeandle-jdk` 仓库是什么？

Jeandle 实际上主要由两个仓库组成：

- **`jeandle-jdk`**：基于 OpenJDK 修改的 JDK/JVM 部分，用于把 Jeandle 接入 JVM。
- **`jeandle-llvm`**：基于 LLVM 的编译器实现部分。

官方构建文档说明，这两个仓库都需要分别下载和编译；构建 `jeandle-jdk` 时，还需要指定已经构建好的 `jeandle-llvm` 路径。([GitHub](https://github.com/jeandle/jeandle-jdk/blob/main/jeandle-docs/getting-started.md?utm_source=chatgpt.com))

因此，你截图中的仓库并不是普通 Java 项目，而是一个包含大量 **OpenJDK、HotSpot JVM、C++、C 和 Java 底层代码**的完整 JDK 工程。

## 它与 HotSpot 的 C1、C2 有什么关系？

普通 HotSpot JVM 通常使用：

```text
解释器
  ↓
C1 编译器：快速编译，优化程度较低
  ↓
C2 编译器：编译较慢，优化程度较高
```

Jeandle 的研究方向可以理解为：

```text
让 JVM 使用基于 LLVM 的 JIT 编译能力
```

也就是尝试利用 LLVM 的优化和机器码生成体系，为 Java 方法生成高性能代码。它属于 **JVM、编译器和 OpenJDK 底层开发项目**，而不是 Spring Boot、MyBatis 这类应用开发框架。

## 适合什么人研究？

这个项目更适合想学习以下内容的人：

- JVM 的 JIT 编译原理
- HotSpot 源码
- OpenJDK 的构建过程
- LLVM IR 和编译优化
- Java 字节码如何变成机器码
- 编译器后端和 CPU 指令生成

对于目前学习 Java 基础而言，可以先知道它的定位，不必立刻阅读全部源码。它的难度明显高于一般 Java 后端项目，通常需要具备 **Java、C++、JVM、操作系统、计算机组成原理和编译原理**等基础。

***

这些问题其实围绕同一个核心：

> **变量有声明类型，变量中保存的是对象引用；对象有自己的实际运行时类型。**

------

# 2

# 1. “父类类型引用指向子类对象”是什么意思

最典型的代码是：

```java
Animal animal = new Cat();
```

把它拆开：

```text
Animal        animal       =       new Cat();
  ↑              ↑                    ↑
声明类型        变量名              创建 Cat 对象
```

更准确地说：

- `Animal`：变量 `animal` 的**声明类型**，也叫编译时类型；
- `animal`：一个引用变量；
- `new Cat()`：创建一个实际类型为 `Cat` 的对象；
- `animal` 中保存了指向这个 `Cat` 对象的引用值。

简单表示：

```text
Animal animal ─────────→ Cat 对象
```

这里不是把 `Cat` 对象变成了 `Animal` 对象。对象仍然是 `Cat`，只是通过一个 `Animal` 类型的变量引用它。

因为：

```java
class Cat extends Animal
```

所以 `Cat` 是 `Animal` 的子类型，可以自动进行向上转型：

```text
Cat → Animal
```

------

# 2. 你的案例中发生了什么

你的代码是：

```java
public void feed(Animal animal) {
    animal.eat();
}
```

调用：

```java
Cat cat = new Cat();
person.feed(cat);
```

进入 `feed()` 时，可以近似理解为发生了：

```java
Animal animal = cat;
```

此时：

```text
cat 的声明类型：Cat
animal 的声明类型：Animal

cat 指向的实际对象：Cat
animal 指向的实际对象：还是同一个 Cat
```

关系如下：

```text
Cat cat ────────┐
                ├────→ 同一个 Cat 对象
Animal animal ──┘
```

Java 参数传递是值传递，这里复制的是 `cat` 中保存的引用值，不会创建新的 `Cat` 对象。

------

# 3. 什么是“声明类型”和“实际类型”

这句话中的“声音类型”应该是笔误，正确说法是：

> **声明类型**

例如：

```java
Animal animal = new Cat();
```

这里有两种类型：

```text
声明类型 / 编译时类型：Animal
实际类型 / 运行时类型：Cat
```

再比如：

```java
Cat cat = new Cat();
```

这里：

```text
声明类型：Cat
实际类型：Cat
```

所以在你的 `feed()` 方法中，参数变量：

```java
Animal animal
```

它的声明类型是 **`Animal`**，不是 `Cat`。

虽然调用时传入了 `Cat` 对象，但不会改变参数变量在源代码中声明的类型。

------

# 4. 如何理解“编译时检查声明类型，运行时根据实际类型调用方法”

对于：

```java
Animal animal = new Cat();
animal.eat();
```

需要分两个阶段。

## 编译阶段

编译器看到：

```java
animal.eat();
```

首先查看 `animal` 的声明类型：

```text
Animal
```

然后检查：

> `Animal` 类型中是否存在可访问的 `eat()` 方法？

存在：

```java
class Animal {
    public void eat() {
    }
}
```

所以编译通过。

## 运行阶段

运行时，JVM 检查 `animal` 实际指向什么对象：

```text
Cat 对象
```

如果 `Cat` 重写了 `eat()`：

```java
class Cat extends Animal {
    @Override
    public void eat() {
        System.out.println("Cat eat fish");
    }
}
```

最终执行的是：

```java
Cat.eat()
```

因此：

```java
Animal animal = new Cat();
animal.eat();
```

输出：

```text
Cat eat fish
```

可以总结为：

```text
编译时：
根据 Animal 检查有没有 eat()

运行时：
根据实际的 Cat 对象选择 Cat.eat()
```

但“编译看左边，运行看右边”只适合帮助理解**被重写的实例方法**，不能套用到所有成员。

------

# 5. 什么叫实例方法

没有使用 `static` 修饰的方法称为**实例方法**：

```java
public void eat() {
}
```

实例方法属于对象，需要通过对象或对象引用调用：

```java
animal.eat();
cat.catchMouse();
person.feed(cat);
```

实例方法内部存在一个隐含的 `this`：

```java
public void eat() {
    // this 表示当前调用该方法的对象
}
```

相对地：

```java
public static void main(String[] args) {
}
```

是静态方法，不属于某个具体实例，而属于类。

可以这样区分：

| 方法               | 是否为实例方法 |
| ------------------ | -------------- |
| `animal.eat()`     | 是             |
| `cat.catchMouse()` | 是             |
| `person.feed(cat)` | 是             |
| `Person.main()`    | 否，是静态方法 |

运行时多态主要发生在**可以被重写的实例方法**上。

------

# 6. 为什么 `animal.catchMouse()` 编译失败

假设：

```java
Animal animal = new Cat();
animal.catchMouse();
```

虽然实际对象是 `Cat`，但编译器检查的是变量的声明类型：

```text
Animal
```

而 `Animal` 中只有：

```java
public void eat() {
}
```

没有：

```java
catchMouse()
```

因此编译器会认为：

> `Animal` 类型的变量不能调用 `catchMouse()`。

所以产生 compile-time error。

必须先确认并转换为 `Cat`：

```java
if (animal instanceof Cat) {
    Cat cat = (Cat) animal;
    cat.catchMouse();
}
```

这里：

```java
Cat cat = (Cat) animal;
```

使新变量 `cat` 的声明类型成为 `Cat`，编译器便能在 `Cat` 中找到 `catchMouse()`。

Java 17 可以写成模式匹配：

```java
if (animal instanceof Cat cat) {
    cat.catchMouse();
}
```

------

# 7. `instanceof` 是什么

`instanceof` 是 Java 的保留关键字，同时构成类型检查运算符。

```java
animal instanceof Cat
```

意思是：

> `animal` 当前引用的非空对象，能不能安全地转换为 `Cat` 类型？

例如：

```java
Animal animal = new Cat();

System.out.println(animal instanceof Cat);    // true
System.out.println(animal instanceof Animal); // true
System.out.println(animal instanceof Dog);    // false
```

如果引用为 `null`：

```java
Animal animal = null;

System.out.println(animal instanceof Cat); // false
```

使用它可以避免错误的强制转换：

```java
Animal animal = new Dog();
Cat cat = (Cat) animal;
```

这段代码虽然可能通过编译，但运行时会抛出：

```text
ClassCastException
```

安全写法：

```java
if (animal instanceof Cat cat) {
    cat.catchMouse();
}
```

------

# 8. `char[]` 和 `CharSequence` 有关系吗

它们之间**没有继承或实现关系**。

```java
char[] chars = {'a', 'b', 'c'};
```

`char[]` 是数组类型。

而：

```java
CharSequence sequence = "abc";
```

`CharSequence` 是一个接口。

常见实现包括：

```text
CharSequence
├── String
├── StringBuilder
├── StringBuffer
└── CharBuffer
```

但是 `char[]` 没有实现 `CharSequence`，所以下面错误：

```java
char[] chars = {'a', 'b', 'c'};
CharSequence sequence = chars; // compile-time error
```

可以转换成 `String`：

```java
CharSequence sequence = new String(chars);
```

或者使用 `CharBuffer`：

```java
CharSequence sequence = CharBuffer.wrap(chars);
```

数组类型在 Java 中直接属于引用类型，并且数组对象可以赋给：

```java
Object
Cloneable
Serializable
```

但不能赋给 `CharSequence`。

------

# 9. 如何理解类、对象、变量和实例变量

来看：

```java
A aa = new A();
```

分解如下：

```text
A        aa       =       new A();
↑         ↑                   ↑
类/类型   变量名             创建 A 类的实例对象
```

## 类

```java
class A {
}
```

`A` 是类，可以理解为创建对象的模板或类型定义。

## 对象

```java
new A()
```

会创建一个 `A` 类的实例对象。

“对象”和“实例”在这里通常可以近似理解成同一个概念：

```text
A 对象
A 类的实例
```

## 变量

```java
aa
```

是变量，它保存一个引用值，该引用指向 `new A()` 创建的对象。

但 `aa` 是否是实例变量，取决于它声明在哪里。

## 声明在方法中：局部变量

```java
class Demo {
    void test() {
        A aa = new A();
    }
}
```

这里的 `aa` 是：

```text
局部变量
+
引用类型变量
```

它不是实例变量。

## 声明在类中，但没有 `static`：实例变量

```java
class Demo {
    A aa = new A();
}
```

这里的 `aa` 是 `Demo` 对象的实例变量。

每创建一个 `Demo` 对象，就有自己的一份 `aa`：

```java
Demo d1 = new Demo();
Demo d2 = new Demo();
d1 对象中有一份 aa
d2 对象中有一份 aa
```

## 使用 `static`：类变量

```java
class Demo {
    static A aa = new A();
}
```

这里 `aa` 是类变量，所有 `Demo` 实例共享一份。

所以，“引用变量”和“实例变量”不是互相排斥的分类：

```text
引用变量：
描述变量保存的是对象引用

局部变量、实例变量、类变量：
描述变量声明在哪里、属于谁
```

一个变量可以同时是：

```text
实例变量 + 引用类型变量
```

例如：

```java
class Demo {
    A aa;
}
```

------

# 10. 接口中可以有 `static` 方法吗

可以。

```java
interface USB {
    static void description() {
        System.out.println("USB interface");
    }
}
```

调用时使用接口名：

```java
USB.description();
```

不能通过实现类对象调用：

```java
USB usb = new USBImplement();
usb.description(); // compile-time error
```

## 接口静态方法能被实现类重写吗

不能。

原因是接口中的静态方法：

- 属于接口本身；
- 不属于实现类对象；
- 不会被实现类继承；
- 因此不存在重写。

下面不能使用 `@Override`：

```java
class USBImplement implements USB {
    @Override
    public static void description() { // 错误
    }
}
```

接口默认方法则不同：

```java
interface USB {
    default void connect() {
        System.out.println("Default connect");
    }
}
```

实现类可以重写默认实例方法：

```java
class USBImplement implements USB {
    @Override
    public void connect() {
        System.out.println("Custom connect");
    }
}
```

总结：

```text
default 方法：实例方法，可以被重写
static 方法：属于接口，不被继承，不能被重写
```

------

# 11. 栈、堆、方法区和常量池

先说明一点：

> Java 语言规范主要规定程序的行为；具体内存实现由 JVM 决定。

下面以常见 HotSpot JVM 的理解说明。

## 栈

每个线程都有自己的 Java 栈。每次调用方法，会创建一个栈帧。

栈帧中通常包含：

- 局部变量；
- 方法参数；
- 操作数栈；
- 方法调用相关信息。

例如：

```java
public static void main(String[] args) {
    String s = "abc";
}
```

局部变量 `s` 中保存的引用位于 `main()` 的栈帧局部变量区域。

对象本身不在栈里，而在堆里。

## 堆

堆用于存放：

- 对象；
- 数组。

例如：

```java
new Cat()
new String("abc")
new char[]{'a', 'b'}
```

这些对象或数组通常位于堆中。

## 方法区

方法区是 JVM 规范中的共享逻辑区域，保存每个已加载类的信息，例如：

- 类的结构信息；
- 字段和方法信息；
- 方法字节码；
- 运行时常量池。

在 HotSpot JDK 8 之后，许多类元数据存放在本地内存中的 **Metaspace（元空间）**。

所以不要机械地认为：

> 所有静态变量、常量和方法都一定物理存放在某一个固定区域。

JVM 规范描述的是逻辑结构，具体实现可以不同。

## 三种容易混淆的“常量池”

### `.class` 文件常量池

编译后的 `.class` 文件中保存字面量和符号引用，例如：

```text
"abc"
java/lang/String
println
```

### 运行时常量池

类加载以后，JVM 为每个类维护的运行时结构，由 `.class` 常量池转化而来。

### 字符串常量池

它保存被驻留（interned）的字符串对象的规范引用。

例如：

```java
String s = "abc";
```

字符串字面量 `"abc"` 会使用字符串池中的规范对象。

现代 HotSpot 中，字符串对象本身位于堆中。

------

# 12. 分析三个 String 语句

代码：

```java
String s3 = new String("abc");
String s5 = "abc";
String s4 = new String(s5);
```

假设此前字符串池中没有 `"abc"`。

## 第一句

```java
String s3 = new String("abc");
```

这里涉及：

1. 字符串字面量 `"abc"` 对应一个池中的字符串对象；
2. `new String(...)` 又明确创建一个新的 `String` 对象；
3. `s3` 指向新创建的对象，而不是池中对象。

表示为：

```text
字符串池：
"abc" ─────────→ String 对象 H1

栈中的 s3 ─────→ 新 String 对象 H2
```

H1 和 H2 内容相同，但不是同一个对象。

## 第二句

```java
String s5 = "abc";
```

JVM 查找字符串池，发现 `"abc"` 已经存在，所以直接复用 H1：

```text
s5 ─────────→ H1
```

不会因为这一句再创建一个新的普通 `String` 对象。

## 第三句

```java
String s4 = new String(s5);
```

因为有 `new`，会再创建一个新的 `String` 对象 H3：

```text
s4 ─────────→ H3
```

最终关系：

```text
当前方法栈帧：

s3 ───────────────→ H2
s5 ────────┐
           │
           └────────→ H1
s4 ───────────────→ H3


堆内存：

H1：String("abc")，字符串池中的规范对象
H2：new String("abc") 创建的对象
H3：new String(s5) 创建的对象


字符串池：

"abc" ─────────→ H1
```

因此：

```java
System.out.println(s3 == s5); // false
System.out.println(s4 == s5); // false
System.out.println(s3 == s4); // false
```

`==` 比较引用是否指向同一个对象。

但：

```java
System.out.println(s3.equals(s5)); // true
System.out.println(s4.equals(s5)); // true
System.out.println(s3.equals(s4)); // true
```

因为 `String.equals()` 比较字符内容。

------

# 13. 字符串池里的值会发生什么变化

执行这些语句后，字符串池中仍然只有一个规范的 `"abc"` 对象引用：

```text
字符串池：
"abc" → H1
```

`s3` 和 `s4` 指向的新对象不会自动进入字符串池。

只有调用：

```java
String s6 = s3.intern();
```

`intern()` 才会返回字符串池中的规范引用。

因为 `"abc"` 已经存在：

```java
s6 == s5 // true
```

需要注意：

> 字符串池中没有名为 `s3`、`s4`、`s5` 的变量。

这些变量属于当前方法的局部变量。字符串池只维护字符串内容与规范字符串对象之间的关系。

而且 `String` 是不可变类，池中的 `"abc"` 对象内容也不会被改成其他字符串。

------

# 14. 把你的案例统一起来

```java
Cat cat = new Cat();
person.feed(cat);
```

可以理解为：

```text
类：
Animal、Cat、Dog、Person

对象：
new Cat()
new Dog()
new Person()

局部引用变量：
cat、dog、person

方法参数变量：
feed(Animal animal) 中的 animal

声明类型：
animal 是 Animal
cat 是 Cat

实际对象类型：
传入 cat 时，animal 实际指向 Cat 对象

编译期：
检查 Animal 是否有 eat()

运行期：
实际对象是 Cat，所以调用 Cat.eat()

调用 catchMouse：
Animal 中没有 catchMouse，所以 animal.catchMouse() 编译失败

转换后：
Cat cat = (Cat) animal;
cat.catchMouse();
```

最关键的记忆是：

> **变量有声明类型，引用指向对象；对象有实际类型。编译器根据声明类型判断代码是否合法，JVM 对被重写的实例方法根据对象实际类型选择最终实现。**

对应官方规范位置主要是：

- JLS §4.12.2：引用类型变量；
- JLS §5.1.5：扩大引用转换；
- JLS §8.4.8.1：实例方法重写；
- JLS §15.12：方法调用；
- JLS §15.12.4.4：运行时定位要调用的方法；
- JLS §15.20.2：`instanceof`；
- JLS §9.4：接口方法；
- JVMS §2.5：运行时数据区。