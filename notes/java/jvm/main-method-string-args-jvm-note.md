# JVM 启动入口与 String\[\] args 缺失问题

## 1. 问题背景

在编写 Java 程序时，经常会看到：

``` java
public static void main(String[] args) {

}
```

如果误写成：

``` java
public static void main() {

}
```

IDE 或命令行启动程序时会出现：

    Error: Main method not found
    Program exited with code: 1

这个问题不是算法错误，而是 Java 程序入口方法不符合 JVM 规范。

------------------------------------------------------------------------

# 2. 相关知识体系

这个问题涉及：

-   Java 方法声明（Method Declaration）
-   方法签名（Method Signature）
-   JVM 程序启动流程
-   main 方法规范
-   static 方法
-   命令行参数

对应 Java 知识体系：

    Java SE
     |
     ├── Language
     |     └── Method
     |
     └── JVM
           └── Program Startup

------------------------------------------------------------------------

# 3. JVM 如何启动 Java 程序

执行：

``` bash
java Main
```

JVM 的执行过程：

    操作系统
        |
        v
    启动 JVM
        |
        v
    ClassLoader 加载 Main.class
        |
        v
    寻找程序入口方法 main(String[])
        |
        v
    调用 main 方法
        |
        v
    执行 Java 代码

关键点：

JVM 不会寻找任意名字叫 main 的方法，而是寻找固定形式：

``` java
public static void main(String[] args)
```

------------------------------------------------------------------------

# 4. main 方法为什么必须包含 String\[\] args

JVM 调用 main 方法时，本质类似：

``` java
Main.main(args);
```

其中：

``` java
String[] args
```

由 JVM 创建。

例如：

执行：

``` bash
java Main hello world
```

JVM 会创建：

``` java
String[] args = {
    "hello",
    "world"
};
```

然后：

``` java
Main.main(args);
```

所以 main 方法必须接收：

``` java
String[]
```

类型参数。

------------------------------------------------------------------------

# 5. 方法签名（Method Signature）

Java 判断方法是否相同，不仅看方法名。

方法签名：

    方法名 + 参数类型

例如：

``` java
main()
```

和：

``` java
main(String[])
```

是两个完全不同的方法。

------------------------------------------------------------------------

## 示例

合法：

``` java
public static void main(String[] args) {

}
```

签名：

    main(String[])

错误：

``` java
public static void main() {

}
```

签名：

    main()

虽然 Java 编译通过，但是 JVM 不认为它是程序入口。

------------------------------------------------------------------------

# 6. 为什么 main 必须 static

普通方法：

``` java
public void test(){

}
```

调用需要对象：

``` java
Main obj = new Main();

obj.test();
```

但是 JVM 启动时：

-   对象还没有创建
-   JVM 不知道如何实例化 Main

因此 main 必须：

``` java
static
```

调用方式：

``` java
Main.main(args);
```

无需创建对象。

------------------------------------------------------------------------

# 7. 为什么 main 必须 public

JVM 属于外部调用者。

如果：

``` java
private static void main(String[] args)
```

JVM 无法访问。

因此要求：

``` java
public
```

------------------------------------------------------------------------

# 8. 为什么返回类型必须 void

JVM 调用：

``` java
Main.main(args);
```

不会接收返回值。

因此规定：

``` java
void
```

如果：

``` java
public static int main(String[] args)
```

不是合法入口。

------------------------------------------------------------------------

# 9. 当前案例分析

错误代码：

``` java
public class Main {

    public static void main() {

        Scanner in = new Scanner(System.in);

    }
}
```

执行：

    java Main

JVM：

    加载 Main.class

            |
            v

    寻找:

    public static void main(String[])

            |
            v

    没有找到

            |
            v

    启动失败

------------------------------------------------------------------------

修改：

``` java
public class Main {

    public static void main(String[] args) {

        Scanner in = new Scanner(System.in);

    }
}
```

执行流程：

    JVM 创建 String[] args

            |
            v

    调用 Main.main(args)

            |
            v

    进入代码

            |
            v

    Scanner读取输入

            |
            v

    算法执行

------------------------------------------------------------------------

# 10. 与 LeetCode / OJ 平台的关系

在线判题平台通常要求：

``` java
public class Main {

    public static void main(String[] args){

    }

}
```

原因：

平台启动 JVM 时，需要固定入口。

如果缺少：

``` java
String[] args
```

平台无法启动程序。

------------------------------------------------------------------------

# 11. 笔记归档建议

推荐目录：

    java
     |
     ├── java-se
     |      |
     |      └── language
     |             |
     |             └── method
     |                    └── main-method.md
     |
     └── jvm
            |
            └── program-startup.md

------------------------------------------------------------------------

# 总结

`String[] args` 缺失导致的错误，本质不是语法错误，而是：

> JVM 根据 Java 规范寻找程序入口时，没有找到符合要求的 main(String\[\])
> 方法。

核心记忆：

``` java
public static void main(String[] args)
```

四个关键词：

-   public：JVM 可以访问
-   static：无需创建对象
-   void：没有返回值
-   String\[\]：接收 JVM 传入的命令行参数
