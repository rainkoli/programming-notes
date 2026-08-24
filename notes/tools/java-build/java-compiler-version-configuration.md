# Java 编译版本配置问题整理

## 1. 问题现象

项目明明已经设置了：

- Project SDK = 17
- Project Language Level = 17
- Module Language Level = 17
- Maven `pom.xml` 里也写了 Java 17

但是编译时仍然出现：

```text
找不到符号：
方法 of(java.lang.String, java.lang.String, java.lang.String)
位置：接口 java.util.List
```

或者：

```text
从发行版 10 开始，'var' 是受限类型名称……
```

例如下面代码在 Java 17 中本来是合法的：

```java
List<String> list =
        new ArrayList<>(List.of("Java", "Spring", "Redis"));
```

以及：

```java
var annotations = DemoClass.class.getDeclaredAnnotations();
```

但仍然报错。

---

## 2. 真正原因

原因不是 JDK 17 没有安装，也不是 `List.of()` 写错了。

真正原因是 IntelliJ IDEA 的：

```text
Settings
→ Build, Execution, Deployment
→ Compiler
→ Java Compiler
```

里面仍然存在：

```text
Per-module bytecode version

java-learning   1.8
javase-demo     1.8
```

同时还勾选了：

```text
Use '--release' option for cross-compilation
```

于是 IDEA 实际编译时可能相当于执行：

```bash
javac --release 8 ...
```

这就会导致：

- Java 9 才有的 `List.of()` 不可用
- Java 10 才有的 `var` 不可用
- 即使实际使用的是 JDK 17，也会按 Java 8 的 API/语言环境进行限制

---

## 3. 为什么 JDK 17 还会按 Java 8 编译？

因为下面这些配置不是同一个东西。

### 3.1 Project SDK

例如：

```text
Project SDK = 17
```

表示：

> IDEA 使用哪一套 JDK。

也就是会使用 JDK 17 中的：

```text
java.exe
javac.exe
java.base
java.util
java.lang
...
```

但这并不代表编译目标一定是 Java 17。

例如 JDK 17 的 `javac` 完全可以执行：

```bash
javac --release 8
```

所以：

```text
编译器本身 = JDK 17
编译目标   = Java 8
```

是完全合法的。

---

### 3.2 Language Level

例如：

```text
Language Level = 17
```

表示：

> IDEA 编辑器允许你使用到 Java 17 为止的语言特性。

例如：

```java
var name = "Java";
```

`var` 是 Java 10 引入的。

如果 Language Level = 8，IDEA 会认为这种语法不合法。

如果 Language Level = 17，IDEA 会认为这种语法合法。

但是：

> 编辑器认为合法，不代表实际编译参数一定也是 Java 17。

---

### 3.3 Target Bytecode Version

IntelliJ IDEA 中：

```text
Settings
→ Compiler
→ Java Compiler
→ Per-module bytecode version
```

会影响实际编译目标。

例如：

```text
javase-demo → 1.8
```

就会让模块尝试按 Java 8 的目标环境编译。

如果同时使用：

```text
--release 8
```

就不仅限制 `.class` 版本，还会限制可以使用的 Java API。

---

## 4. `--release` 为什么比 `source/target` 更重要？

假设实际使用 JDK 17。

### 只限制 target

```bash
javac -target 8
```

主要表示：

> 生成尽量兼容 Java 8 的 `.class` 文件格式。

但这并不天然保证你没有错误使用更高版本 JDK 才存在的 API。

---

### 使用 `--release`

```bash
javac --release 8
```

相当于要求：

> 按 Java 8 的语言/API 环境编译。

因此：

```java
List.of("Java", "Spring");
```

会失败。

原因是：

```text
Java 8 的 java.util.List
没有 of() 方法
```

而不是 JDK 17 本身没有这个方法。

---

## 5. 为什么 `List.of()` 会报错？

`List.of()` 是 Java 9 引入的。

因此：

```text
Java 8
java.util.List
→ 没有 of()

Java 9+
java.util.List
→ 有 of()
```

如果编译器按：

```text
--release 8
```

运行，那么它看到的是 Java 8 的标准库 API 视图。

因此：

```java
List.of(...)
```

就会出现：

```text
找不到符号：方法 of(...)
```

---

## 6. 为什么 `var` 会报错？

`var` 是 Java 10 引入的局部变量类型推断语法。

例如：

```java
var annotations = DemoClass.class.getDeclaredAnnotations();
```

如果编译环境是：

```text
Java 8
```

就会报错。

如果环境是：

```text
Java 17
```

则完全合法。

---

## 7. Maven 和 IDEA 是两套配置来源

Maven 的 Java 版本配置通常写在 `pom.xml` 中。

例如：

```xml
<properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
</properties>
```

或者更推荐：

```xml
<properties>
    <maven.compiler.release>17</maven.compiler.release>
</properties>
```

而 IDEA 自己还有：

```text
Project SDK
Project Language Level
Module Language Level
Java Compiler Target Bytecode
Maven Runner JRE
```

所以可能出现：

```text
Maven 配置 = 17
IDEA Language Level = 17
IDEA Compiler Override = 1.8
```

这就是“看起来已经设置成 17，但 Build 仍然报 Java 8 错误”的原因。

---

## 8. 推荐的统一配置

建议全部统一为 Java 17。

### Project

```text
Project SDK = 17
Language Level = 17
```

### Module

```text
javase-demo
Language Level = 17
Module SDK = Project SDK (17)
```

### IDEA Compiler

```text
Project bytecode version:
Same as language level
```

并删除或者改掉：

```text
Per-module bytecode version

java-learning   1.8
javase-demo     1.8
```

推荐：

```text
不要单独覆盖
```

或者明确设置：

```text
java-learning   17
javase-demo     17
```

---

## 9. Maven 推荐配置

父 `pom.xml` 中推荐：

```xml
<properties>
    <maven.compiler.release>17</maven.compiler.release>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>
```

相比：

```xml
<maven.compiler.source>17</maven.compiler.source>
<maven.compiler.target>17</maven.compiler.target>
```

`release` 的含义更完整。

可以简单理解为：

```text
source
→ 允许什么语法

target
→ 生成什么版本的 class

release
→ 语言版本
 + class 版本
 + 标准库 API 版本
```

---

## 10. 修改后应该做什么？

修改 IDEA / Maven 配置后：

```text
Maven
→ Reload All Maven Projects
```

然后：

```text
Build
→ Rebuild Project
```

如果仍然异常，再检查：

```text
Settings
→ Build Tools
→ Maven
→ Runner
→ JRE
```

确保使用：

```text
Project JDK 17
```

---

## 11. 最终理解模型

可以把 Java 编译版本理解成：

```text
Project SDK
   ↓
决定使用哪套 JDK

Language Level
   ↓
决定 IDE 允许哪些 Java 语法

Compiler --release / target
   ↓
决定实际按哪个 Java 版本编译

Maven pom.xml
   ↓
决定 Maven 构建时的 Java 编译版本
```

如果它们互相冲突：

```text
SDK = 17
Language Level = 17
--release = 8
```

最终仍然可能表现为：

```text
Java 8 编译限制
```

因此，**“JDK 版本”和“编译目标版本”必须分开理解。**
