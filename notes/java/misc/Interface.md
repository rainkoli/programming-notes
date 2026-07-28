# 1

## Main location

The primary section is:

**[JLS Chapter 9 — Interfaces](https://docs.oracle.com/javase/specs/jls/se25/html/jls-9.html)**

This chapter explains interface declarations, inheritance between interfaces, interface members, fields, methods, functional interfaces, and annotation interfaces. It also states that an interface can provide a common supertype for otherwise unrelated classes and cannot be instantiated directly. ([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-9.html))

## Important subsections

| Topic                                | Oracle documentation                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| Declaring an interface               | [§9.1 Interface Declarations](https://docs.oracle.com/javase/specs/jls/se25/html/jls-9.html#jls-9.1) |
| Interface modifiers                  | [§9.1.1 Interface Modifiers](https://docs.oracle.com/javase/specs/jls/se25/html/jls-9.html#jls-9.1.1) |
| Interface inheritance with `extends` | [§9.1.3 Superinterfaces and Subinterfaces](https://docs.oracle.com/javase/specs/jls/se25/html/jls-9.html#jls-9.1.3) |
| Interface members                    | [§9.2 Interface Members](https://docs.oracle.com/javase/specs/jls/se25/html/jls-9.html#jls-9.2) |
| Interface fields and constants       | [§9.3 Field (Constant) Declarations](https://docs.oracle.com/javase/specs/jls/se25/html/jls-9.html#jls-9.3) |
| Interface methods                    | [§9.4 Method Declarations](https://docs.oracle.com/javase/specs/jls/se25/html/jls-9.html#jls-9.4) |
| Functional interfaces                | [§9.8 Functional Interfaces](https://docs.oracle.com/javase/specs/jls/se25/html/jls-9.html#jls-9.8) |

Chapter 9 also explains that a variable whose declared type is an interface can refer to an object of any class that implements that interface. ([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-9.html))

## Where `implements` is defined

For a class implementing an interface, see:

**[JLS §8.1.5 — Superinterfaces](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.1.5)**

This section defines the `implements` clause in a class declaration:

```java
class Cat implements Animal {
}
```

A class can extend only one direct superclass, but it may implement one or more interfaces. ([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html))

## Interface as a type

An interface is also a **reference type**. The relevant locations are:

- **[§4.1 The Kinds of Types and Values](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html#jls-4.1)**
- **[§4.10.2 Subtyping among Class and Interface Types](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html#jls-4.10.2)**

The JLS classifies class types, interface types, and array types as reference types. ([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html))

For studying Java interfaces, the best reading order is:

**§9.1 → §8.1.5 → §9.2 → §9.3 → §9.4 → §9.8**.

# 2

你的理解方向是正确的：**接口中的 `static` 方法和 `default` 方法都是“已经提供实现的方法”**，因此必须写方法体；而普通接口方法通常是抽象方法，只声明规则，不提供实现，所以以分号结束。

Oracle 官方最直接的规定在：

**[JLS §9.4.3 — Interface Method Body](https://docs.oracle.com/javase/specs/jls/se25/html/jls-9.html#jls-9.4.3)**

## 1. 图中四种方法的区别

你的接口大致是：

```java
public interface USB {
    int PI = 12;

    void bb();

    void cc();

    public static void dd() {
    }

    public default void ee() {
    }
}
```

可以分类为：

| 方法                   | 实际性质               | 是否需要方法体 |
| ---------------------- | ---------------------- | -------------- |
| `void bb();`           | `public abstract` 方法 | 否             |
| `void cc();`           | `public abstract` 方法 | 否             |
| `static void dd() {}`  | 静态方法               | 是             |
| `default void ee() {}` | 默认实例方法           | 是             |

------

## 2. 为什么普通接口方法不需要方法体

接口中的方法如果没有 `private`、`default` 或 `static` 修饰符，就会被隐式声明为 `abstract`。

所以：

```java
void bb();
```

实际等价于：

```java
public abstract void bb();
```

抽象方法只规定：

> 实现类必须提供一个名为 `bb`、参数和返回值符合要求的方法。

它不负责说明具体如何执行，因此使用分号结束：

```java
void bb();
```

而不是：

```java
void bb() {
}
```

JLS §9.4 明确规定：缺少 `private`、`default` 和 `static` 修饰符的接口方法隐式为 `abstract`，其方法体使用分号表示，而不是代码块。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-9.html))

例如，实现类负责补充实现：

```java
public class USBImplement implements USB {
    @Override
    public void bb() {
        System.out.println("实现 bb 方法");
    }

    @Override
    public void cc() {
        System.out.println("实现 cc 方法");
    }
}
```

------

## 3. 为什么 `default` 方法必须有方法体

`default` 方法的作用就是：

> 在接口中直接提供一份默认实现。

例如：

```java
default void ee() {
    System.out.println("USB default method");
}
```

实现类即使不重写 `ee()`，也能直接继承并使用这份实现：

```java
USB usb = new USBImplement();
usb.ee();
```

假如允许这样写：

```java
default void ee();
```

那么它没有任何默认实现，也就失去了 `default` 的意义。

JLS §9.4 将默认方法定义为使用 `default` 修饰的实例方法，其代码块为没有重写该方法的实现类提供默认实现。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-9.html))

JLS §9.4.3 进一步明确规定：

- 默认方法具有代码块方法体；
- 如果 `default` 方法使用分号作为方法体，会产生 **compile-time error**。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-9.html))

因此：

```java
default void ee() {
}
```

是正确的。即使 `{}` 里面没有代码，**空代码块仍然是一份完整的方法实现**。

下面则错误：

```java
default void ee(); // compile-time error
```

------

## 4. 为什么 `static` 方法必须有方法体

接口静态方法属于接口本身，通过接口名称调用：

```java
USB.dd();
```

它不是由实现类对象进行动态调用的，也不要求实现类重写。

例如：

```java
static void dd() {
    System.out.println("USB static method");
}
```

调用时 JVM 会直接执行 `USB.dd()` 中定义的代码。

如果写成：

```java
static void dd();
```

既没有方法体，又没有哪个实现类负责实现它，因此该方法没有可执行内容。

JLS §9.4 说明，接口可以声明 `static` 方法，这种方法不依赖某个具体对象调用；接口静态方法和抽象接口方法、默认方法属于不同种类。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-9.html))

JLS §9.4.3 则直接规定：

> `private` 或 `static` 接口方法具有代码块方法体，该代码块提供该方法的实现。

并规定：如果 `static` 接口方法使用分号代替代码块，会产生 **compile-time error**。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-9.html))

所以：

```java
static void dd() {
}
```

正确，而：

```java
static void dd(); // compile-time error
```

错误。

------

## 5. 为什么不能写成 `abstract static`

抽象方法表示：

> 当前方法没有实现，等待子类或实现类通过重写提供实现。

静态方法表示：

> 方法属于声明它的接口本身，不是对象的实例方法。

这两种性质互相冲突。接口的静态方法不会作为普通实例方法交给实现类重写。

因此不能写：

```java
abstract static void dd();
```

JLS §9.4 明确规定，一个接口方法不能同时具有 `abstract`、`default`、`static` 中的两个或多个关键字，否则产生编译时错误。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-9.html))

------

## 6. 三者的调用方式

```java
public interface USB {
    void connect();

    default void check() {
        System.out.println("Default check");
    }

    static void description() {
        System.out.println("USB interface");
    }
}
```

### 抽象方法

由实现类实现，通过对象调用：

```java
USB usb = new USBImplement();
usb.connect();
```

### 默认方法

接口已经提供默认实现，也通过对象调用：

```java
usb.check();
```

实现类可以选择重写：

```java
@Override
public void check() {
    System.out.println("Custom check");
}
```

### 静态方法

通过接口名调用：

```java
USB.description();
```

不能把接口静态方法当成普通实例方法调用：

```java
usb.description(); // compile-time error
```

------

## 7. 你截图中的黄色提示不是编译错误

IntelliJ IDEA 显示：

```text
Modifier 'public' is redundant for interface members
```

意思是：

> 接口中的这些方法已经隐式为 `public`，所以显式写 `public` 是多余的。

因此：

```java
public static void dd() {
}
```

通常简写为：

```java
static void dd() {
}
```

而：

```java
public default void ee() {
}
```

通常简写为：

```java
default void ee() {
}
```

这只是代码风格警告，不是错误。JLS §9.4 规定，接口方法没有显式访问修饰符时隐式为 `public`；显式重复写 `public` 是允许的，但不推荐。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-9.html))

## 最终记忆

```text
abstract 方法：
只声明“必须做什么”
没有实现
使用分号 ;

default 方法：
接口提供默认实现
必须使用代码块 {}

static 方法：
接口本身提供并执行实现
必须使用代码块 {}
```

最关键的官方章节就是：

**[JLS §9.4.3 — Interface Method Body](https://docs.oracle.com/javase/specs/jls/se25/html/jls-9.html#jls-9.4.3)**。