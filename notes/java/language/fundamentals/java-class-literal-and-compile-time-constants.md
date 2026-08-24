# Java 中的 `Class`、Class Literal 与编译期常量

## 1. 核心结论

在 Java 源码中写：

```java
User.class
```

这里的 `User.class` **不是磁盘中的 `User.class` 字节码文件**，而是一个 Java **表达式（expression）**。

更准确地说，它属于：

> **类字面量（class literal）**

这个表达式求值后，会得到一个代表 `User` 类型的 `Class<User>` 对象引用。

因此：

```text
User.class
│
├─ 是 expression（表达式）                    ✅
├─ 更具体：class literal（类字面量）          ✅
├─ 表达式类型是 Class<User>                   ✅
├─ 是 constant expression（常量表达式）       ❌
└─ 是磁盘上的 User.class 文件                 ❌
```

JLS 中对应位置：

- **JLS §15.8.2 — Class Literals**  
  https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html#jls-15.8.2
- **JLS §15.29 — Constant Expressions**  
  https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html#jls-15.29

---

## 2. `User.class` 到底是什么

例如：

```java
class User {
}

Class<User> clazz = User.class;
```

可以把右侧理解为：

```text
Class<User> clazz = User.class;
                    └────────┘
                     expression
                         │
                         └─ class literal
```

`User.class` 被求值后，结果是：

```text
User.class
    │
    │ evaluation（求值）
    ↓
代表 User 类型的 Class 对象引用
    │
    ↓
类型为 Class<User>
```

所以它和下面这些一样，都是表达式：

```java
1 + 2            // expression，结果类型 int
"hello"          // expression，结果类型 String
new User()       // expression，结果类型 User
user.getClass()  // expression，结果类型 Class<?>
User.class       // expression，结果类型 Class<User>
```

区别只在于 `User.class` 是一种特殊的表达式：**class literal**。

---

## 3. 为什么源码还没有编译，也能写 `User.class`

假设当前正在编辑：

```text
User.java
```

源码：

```java
public class User {

    public static void main(String[] args) {
        Class<User> clazz = User.class;
    }
}
```

此时磁盘中可能还没有：

```text
User.class
```

但源码中的：

```java
User.class
```

依然完全合法。

原因是 Java 编译器 `javac` 在这里处理的是：

> **类型（type）**

而不是：

> **未来生成的某个文件（file）**

当编译器已经看到了：

```java
public class User {
```

它就已经知道当前编译单元（**compilation unit**）中声明了一个名为 `User` 的类型。

因此：

```java
User.class
```

是在表达：

> “取得 `User` 这个类型对应的 `Class` 对象。”

而不是：

> “去磁盘打开 `User.class` 文件。”

---

## 4. 源码中的 `.class` 与文件扩展名 `.class` 是两回事

这是最需要建立的概念边界。

### 源码中的 `User.class`

```java
Class<User> clazz = User.class;
```

这里的 `.class` 是 **Java 语言语法的一部分**。

正式名称：

> **class literal（类字面量）**

### 编译后的 `User.class`

```text
User.java
   │
   │ javac
   ↓
User.class
```

这里的 `.class` 是 **class file（类文件）** 的文件扩展名。

它保存 JVM 能够读取的 class-file format 二进制数据。

所以：

```text
源码：User.class
       │
       └─ class literal expression

文件：User.class
       │
       └─ class file / bytecode file
```

二者只是文本写法碰巧都出现 `.class`，概念完全不同。

---

# 5. Class Literal 与 Constant Expression 的区别

这是最容易混淆的地方。

## 5.1 Class Literal

正式术语：

> **Class Literal（类字面量）**

例如：

```java
String.class
User.class
int.class
void.class
SpringBootTest.class
```

它们表示某个 Java 类型对应的 `Class` 对象。

例如：

```java
Class<String> c1 = String.class;
Class<User> c2 = User.class;
Class<Integer> c3 = int.class;
```

注意：

```java
int.class
```

的类型是：

```java
Class<Integer>
```

JLS §15.8.2 对 class literal 有正式定义。

---

## 5.2 Constant Expression

正式术语：

> **Constant Expression（常量表达式）**

JLS §15.29 对它有严格定义。

例如：

```java
1 + 2
10 * 20
"hello"
"a" + "b"
2 * 3 + 4
```

这些表达式在满足 JLS 规则时，可以在编译期间求值。

例如：

```java
int x = 1 + 2;
```

编译器可以直接得到：

```text
3
```

因此可以发生 **constant folding（常量折叠）**。

---

## 5.3 为什么 `User.class` 不是 Constant Expression

因为 JLS §15.29 规定，constant expression 的结果类型只能是：

- primitive type（基本类型）
- `String`

而：

```java
User.class
```

的类型是：

```java
Class<User>
```

`Class<User>` 是引用类型（**reference type**），并且不是 `String`。

因此：

```text
User.class
    │
    ├─ class literal           ✅
    └─ constant expression     ❌
```

---

# 6. “编译器常量”这个说法需要拆开

中文里经常有人说“编译器常量”或“编译期常量”，但这个说法容易把几个不同概念混在一起。

Java 中至少要区分以下概念。

## 6.1 Constant Expression

**常量表达式（constant expression）**

例如：

```java
1 + 2
"a" + "b"
```

它是“表达式”的分类。

---

## 6.2 Constant Variable

**常量变量（constant variable）**

JLS §4.12.4 中定义了 constant variable。

一个变量要成为 constant variable，通常需要满足：

1. 是 `final`；
2. 类型是 primitive type 或 `String`；
3. 使用 constant expression 初始化。

例如：

```java
final int MAX = 100;
final String NAME = "Java";
```

这里：

```text
100      → constant expression
MAX      → constant variable
```

而：

```java
final Class<User> TYPE = User.class;
```

虽然 `TYPE` 是 `final`，但它**不是 JLS 所说的 constant variable**，因为它的类型是 `Class<User>`。

---

## 6.3 Constant Folding

**常量折叠（constant folding）**

是编译器优化行为。

例如：

```java
int x = 10 * 20;
```

编译器可以直接把：

```text
10 * 20
```

折叠为：

```text
200
```

这是 compiler optimization（编译器优化），不要与 class literal 混淆。

---

## 6.4 Constant Pool

**常量池（constant pool）** 又是另一个概念。

`.class` 文件内部存在：

> **constant_pool table（常量池表）**

其中可以存储：

- 类符号引用
- 方法符号引用
- 字段符号引用
- 字符串
- 数值常量
- 方法句柄等

因此：

```text
constant expression
constant variable
constant folding
constant pool
class literal
```

这五个词不能混为一谈。

---

# 7. Class Literal 与 Constant Expression 对照表

| 对比项 | Class Literal | Constant Expression |
|---|---|---|
| 中文 | 类字面量 | 常量表达式 |
| 示例 | `User.class` | `1 + 2` |
| JLS | §15.8.2 | §15.29 |
| 是否是 expression | 是 | 是 |
| 结果类型 | `Class<T>` | primitive type 或 `String` |
| 是否要求编译期可计算为普通常量值 | 否 | 是 |
| 是否代表类型 | 是 | 否 |
| 是否等于 `.class` 文件 | 否 | 无关 |

最关键的一句话：

> **Class literal 是一种表达式的语法类别；constant expression 是满足特定编译期求值规则的一类表达式。`User.class` 属于前者，不属于后者。**

---

# 8. 编译后发生什么

例如：

```java
Class<User> clazz = User.class;
```

经过 `javac`：

```text
Java source
    │
    │ javac
    ↓
class file
```

class file 中会保存对 `User` 类型的符号引用（**symbolic reference**）。

JVM 执行相应字节码时，会取得代表 `User` 类型的 `Class` 对象。

可以抽象为：

```text
源码
User.class
    │
    │ javac
    ↓
class-file constant pool
    │
    └─ User 的 symbolic reference
              │
              │ JVM resolution
              ↓
          Class<User>
```

这里的关键术语：

- **symbolic reference**：符号引用
- **resolution**：解析
- **Class object**：`Class` 对象
- **class-file constant pool**：类文件常量池

---

# 9. `User.class` 不等于“执行时一定去磁盘读取文件”

常见的初学者描述是：

> “执行到 `User.class` 时 JVM 去磁盘读取 `User.class` 文件。”

这个说法太绝对。

更准确的是：

> JVM 需要得到对应类型的运行时 `Class` 对象；如果相关类型尚未加载或相关符号尚未解析，则可能发生 loading / linking / resolution。

另外，类的字节码来源也不一定是本地磁盘文件。

ClassLoader（类加载器）可以从不同来源获得 class bytes，例如：

- 文件系统
- JAR
- 网络
- 动态生成的 byte array
- 自定义 ClassLoader

所以专业表达最好是：

> **加载类的 binary representation（二进制表示）**

而不是固定说：

> “读取磁盘上的 `.class` 文件。”

---

# 10. `User.class` 与类初始化

还要区分：

```text
Loading         加载
Linking         链接
Initialization  初始化
```

仅仅求值：

```java
User.class
```

不会因为这个 class literal 本身就触发 `User` 的类初始化（**class initialization**）。

例如：

```java
class User {
    static {
        System.out.println("initialized");
    }
}

Class<User> clazz = User.class;
```

`User.class` 本身不是 JLS §12.4.1 中规定的主动初始化触发条件。

JLS：

- **§12.4.1 When Initialization Occurs**  
  https://docs.oracle.com/javase/specs/jls/se25/html/jls-12.html#jls-12.4.1

---

# 11. `User.class`、`getClass()` 与 `Class.forName()`

三者虽然都能接触到 `Class`，语义不同。

## `User.class`

```java
Class<User> c = User.class;
```

- class literal
- 编译期类型可知
- 得到表示 `User` 类型的 `Class<User>`
- 本身不触发类初始化

## `obj.getClass()`

```java
Class<?> c = obj.getClass();
```

- instance method call（实例方法调用）
- 根据一个已经存在的对象，获得其实际运行时类

## `Class.forName()`

```java
Class<?> c = Class.forName("com.example.User");
```

- 通过字符串类名查找并加载类
- 常用于反射
- 默认形式通常会触发类初始化

---

# 12. 最终知识图

```text
Java Source Code
│
├─ User.class
│    │
│    ├─ expression
│    │
│    └─ class literal
│          │
│          └─ expression type: Class<User>
│
├─ 1 + 2
│    │
│    └─ constant expression
│
├─ final int X = 3;
│    │
│    └─ X = constant variable
│
└─ javac
     │
     ↓
Class File
│
├─ constant_pool
│    └─ symbolic references / literals / constants
│
└─ bytecode instructions
     │
     ↓
JVM Runtime
│
├─ loading
├─ linking
│    └─ resolution
└─ Class<User> object
```

---

# 13. 一句话总结

> **源码中的 `User.class` 是 Java 的 class literal expression（类字面量表达式），求值得到 `Class<User>` 对象；它既不是 JLS 意义上的 constant expression（常量表达式），也不是磁盘中的 `User.class` 类文件。**
