原因是：**你的程序没有创建 `Chinese` 对象。**

图中的非静态代码块：

```java
{
    System.out.println("I'm code block");
}
```

正式名称是 **instance initializer（实例初始化器或实例代码块）**。它只有在创建类的实例时才会执行。Oracle JLS §8.6 明确规定：实例初始化器在创建该类的实例时执行。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html))

## 你当前的 `main()` 做了什么

你的有效代码只有：

```java
public static void main(String[] args) {
    Chinese.say();

    /*
    Chinese.country = "Chinese";
    Chinese chinese = new Chinese("San Zhang", (byte) 20);
    chinese.show();
    */
}
```

创建对象的代码：

```java
new Chinese("San Zhang", (byte) 20);
```

被注释掉了，所以没有 `Chinese` 实例被创建，实例代码块自然不会执行。

`Chinese.say()` 是静态方法调用：

```java
public static void say()
```

调用静态方法不需要创建对象，因此不会触发实例代码块。

## 为什么静态代码块执行了

静态代码块：

```java
static {
    System.out.println("I'm static code block");
}
```

在类初始化时执行。你的程序直接运行 `Chinese.main()`，JVM 必须先初始化 `Chinese` 类，然后才能调用其 `main()` 方法。因此静态代码块会先输出。JLS 说明，执行初始类的 `main()` 方法前必须先初始化该类；类初始化会执行静态字段初始化器和静态初始化器。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-12.html))

因此当前执行过程是：

```text
初始化 Chinese 类
    ↓
执行 static 代码块
    ↓
进入 main()
    ↓
调用 Chinese.say()
    ↓
程序结束
```

所以输出是：

```text
I'm static code block
I'm from nulland I'm static method
```

其中 `country` 是 `null`，因为你给它赋值的代码也被注释掉了：

```java
Chinese.country = "Chinese";
```

## 取消注释后

将代码改成：

```java
public static void main(String[] args) {
    Chinese.say();

    Chinese.country = "Chinese";
    Chinese chinese = new Chinese("San Zhang", (byte) 20);
    chinese.show();
}
```

大致会输出：

```text
I'm static code block
I'm from nulland I'm static method
I'm code block
Name: San Zhang, Age: 20, Country: Chinese
```

这里不会输出：

```text
No args constructor
```

因为你调用的是 Lombok `@AllArgsConstructor` 生成的全参数构造器：

```java
new Chinese("San Zhang", (byte) 20);
```

而不是你自己写的无参数构造器：

```java
new Chinese();
```

若改为：

```java
Chinese chinese = new Chinese();
```

则会看到：

```text
I'm code block
No args constructor
```

## 实例化时的执行顺序

简化后的顺序是：

```text
父类构造过程
    ↓
当前类的实例变量初始化和实例代码块
    ↓
当前类构造器的方法体
```

JLS §12.5 规定，实例变量初始化器和实例初始化器按照它们在源代码中出现的顺序执行，然后才继续执行当前构造器的剩余部分。([Oracle Docs](https://docs.oracle.com/javase/specs/jls/se25/html/jls-12.html))

因此你写的注释：

```java
// code block execute before constructor
```

不够准确，更适合写成：

```java
// The instance initializer executes before the body
// of the current class's constructor.
```

也就是：

> **实例代码块在当前类的构造器主体之前执行，但在父类构造过程之后执行。**