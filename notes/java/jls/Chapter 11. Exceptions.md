# Misc

要分清一个东西：
Complie-time error：编译时错误
checked exception：受检异常/检查型异常，不能翻译成 compile-time exception，因为异常并不是在编译时发生的；编译器只会在编译阶段检查它有没有被捕获或者声明

* checked exception：受检异常，e.g. IOException
* unchecked exception：非受检异常
* runtime exception：运行时异常，指 RuntimeException 及其子类
* complite-time error：编译期错误

例如：

IOException is a checked exception.

意思是：

IOException 是一种受检异常

而不是：
IOException is a complie-time exception.

```tex
异常在运行时发生
        ↓
checked exception
        ↓
编译器在编译时检查是否 catch 或 throws
```

> “编译期异常”，这个中文说法在一些教材中常见，但使用 checked exception（受检异常）最准确

checked exception 和 complie-time error 之间的关系：当 checked exception 没有被正确捕获或声明时会导致一个 complite-time error：

```tex
checked exception 没有 catch 或 throws
                    ↓
编译器检查不通过
                    ↓
产生 compile-time error
```

例如：

```java
public static void main(String[] args) {
	Files.readString(Path.of("test.txt"));	
}
```

Files.readString 可能抛出 IOException，它属于 checked exception。如果既不使用 try-catch，也不在方法上写 throws IOException，编译器就会报告 complie-time error

但这里不是说：

> IOException 在编译时被抛出了

而是：

> 编译器在编译时发现你没有处理或声明将来运行时可能发生的 IOException

JLS §11.1.1 把异常类区分为 checked exception classes 和 unchecked exception；

```tex
非受检异常 = 运行时异常 + Error
受检异常 = 全部异常 - 非受检异常 = 全部异常 - 运行时异常 - Error
```

§11.2 的标题则是 Compile-Time Checking of Exceptions，即“对异常进行编译期检查”，这说明“编译期”描述的是检查发生的阶段，不是异常实际发生的阶段。

```tex
Checked exception：受检异常

编译器要求可能抛出受检异常的代码：
1. 使用 try-catch 捕获；或者
2. 使用 throws 声明。

如果两者都没有，则产生 compile-time error。
异常本身仍然只会在程序运行时真正被抛出。
```



Java语法硬性规定：所有继承 Throwable 的类（自定义异常、Error、Exception 等）不能带泛型参数：

```java
public class MyException<T> extends Excpetion{} // 非法代码，编译失败
```

底层原因：JVM 异常处理机制不支持泛型异常。

```java
public class test<T> extends Exception {} // 编译时错误
// Generic class may not extend java.lang.Throwable
```

```tex
泛型异常类继承 Throwable
        ↓
违反 Java 类声明规则
        ↓
compile-time error
```

```tex
TestException extends Exception
        ↓
TestException 是 checked exception
        ↓
抛出时必须 catch 或 throws
```



IDEA红色感叹号不等于“发生异常”，红色标记只表示检测到了一个会导致代码无法正常编译的问题

```tex
int number = "hello";          // 类型不兼容

unknownMethod();               // 找不到方法

privateValue = 10;             // 访问权限不允许

class Test<T> extends Exception { } // 违反类声明规则
```

这些都是编译时错误，但都不是运行时抛出的异常对象。

“报错”和“异常”不是完全相同的概念

日常说话时，人们经常把所有红色提示都叫“异常”或“报错”。但在 Java 的严格术语中需要区分：

| 情况                               | 是否属于 Java 异常 |
| ---------------------------------- | ------------------ |
| IDEA 红色波浪线                    | 不一定             |
| 语法错误                           | 不是异常           |
| 类型不兼容                         | 不是异常           |
| 找不到变量或方法                   | 不是异常           |
| 违反泛型类声明规则                 | 不是异常           |
| 运行时抛出 `NullPointerException`  | 是异常             |
| 主动执行 `throw new IOException()` | 是异常             |

代码还没有成功编译：

```tex
编写源代码
    ↓
IDEA 静态分析
    ↓
发现泛型类不能继承 Exception
    ↓
产生 compile-time error
    ↓
无法正常生成对应的 .class 文件
```

程序没有开始运行，所以也没有真正创建、抛出某个异常对象。

真正的“抛出异常”

```java
String str = null;
str.length();
```

这段代码可以通过编译，但是运行到 `str.length()` 时会抛出：

```java
NullPointerException
```

这是异常：

```tex
成功编译
    ↓
程序开始运行
    ↓
执行非法操作
    ↓
创建并抛出异常对象
```

再例如：

```java
throw new IOException("读取失败");
```

执行到这一行时，才是在运行期间真正抛出异常对象。

改掉 `<T>` 后才是受检异常类

```java
public class TestException extends Exception {
    public TestException(String message) {
        super(message);
    }
}
```

此时：

```tex
TestException 是合法的类
        ↓
它继承 Exception
        ↓
但不是 RuntimeException 的子类
        ↓
它是 checked exception
```

之后这样使用：

```java
throw new TestException("出现问题");
```

编译器才会检查你是否进行了：

```java
try {
    // ...
} catch (TestException e) {
    // ...
}
```

或者声明：

```java
throws TestException
```

结论：

> 红色标记表示代码存在错误，但不等于发生了 Java 异常。

> 图中是违反 Java 类声明规则而产生的编译时错误，不是 checked exception，也没有在运行时抛出异常。



## 1. Complie-time error： 编译时错误

在代码编译阶段违反 Java 语言规则，编译器必须拒绝生成正常的 `.class` 文件

* 访问权限不允许
* 变量未声明
* 方法参数不匹配
* 类型不兼容
* 语法错误
* checked exception 未处理



## 2\. Loading error： 加载错误

程序编译成功后，如果 JVM 找不到二进制文件，或者无法正常加载 `.class` 文件就会产生 loading error

e.g. `ClassNotFoundException`, `NoClassDefFoundException`



## 3\. Linkage error：链接错误

类已经被编译，但运行时不同的 `.class` 文件不兼容

e.g.`NoSuchMethodError`, `NoSuchFileError`, `AbstractMethodError`, `UnsaisfieldLinkError`



## 4\. Initialization error：初始化错误

类加载和链接完成之后，JVM 还要执行：

* `static` 变量初始化器
* `static` 代码块

e.g.

```java

class Test {

&#x09;static int number = 1 / 0;

}

```

第一次主动使用 `Test`  类时，其静态初始化失败：`ExceptionInInitialzerError`



## 5\. Runtime error：

程序通过了编译但是在执行过程中出现了错误：

e.g.

```java

int reslut = 1 / 0; // ArithmeticException



String text = null;

System.out.println(text.lengt); // NullPointerException
```



## 6. Logic errror：逻辑错误

程序可以编译、运行，但是不符合预期

```java
int width = 5;
int width = 3;

int area = width + height; // 程序不会抛异常，但是矩形面积应是：width * height
```
