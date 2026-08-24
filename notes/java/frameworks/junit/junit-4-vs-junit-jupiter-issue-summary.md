# JUnit 4 与 JUnit Jupiter 注解冲突问题总结

## 1. 问题现象

项目中显式添加了 JUnit 4.13.2：

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
</dependency>
```

在测试类中，IDEA 同时给出了两个 `@Test`：

```java
org.junit.Test
org.junit.jupiter.api.Test
```

使用：

```java
import org.junit.Test;
```

测试可以正常运行。

而使用：

```java
import org.junit.jupiter.api.Test;
```

时，运行测试出现类似错误：

```text
java.lang.NoSuchMethodError:
'java.lang.String
org.junit.platform.engine.discovery.MethodSelector
.getMethodParameterTypes()'
```

---

## 2. 两个 `@Test` 并不是同一个注解

它们虽然都叫 `Test`，但来自不同的 JUnit 体系。

### JUnit 4

Maven 依赖：

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
</dependency>
```

对应的测试注解：

```java
import org.junit.Test;
```

关系：

```text
junit:junit:4.13.2
        ↓
     JUnit 4
        ↓
 org.junit.Test
        ↓
      @Test
```

---

### JUnit Jupiter

JUnit Jupiter 是新的 JUnit 测试模型，常见于 JUnit 5 及之后的 JUnit Platform 体系。

对应的测试注解：

```java
import org.junit.jupiter.api.Test;
```

关系：

```text
JUnit Jupiter API
        ↓
org.junit.jupiter.api.Test
        ↓
       @Test
```

因此：

```java
org.junit.Test
```

和：

```java
org.junit.jupiter.api.Test
```

只是名字一样，实际上是两个完全不同的类。

---

## 3. 为什么 IDEA 能同时找到两个 `@Test`

虽然我们手动添加的是：

```text
junit:junit:4.13.2
```

但是当前项目本身还使用了 Spring Boot 相关测试依赖，因此项目的测试 classpath 中还可能存在 JUnit Jupiter / JUnit Platform。

所以项目中实际上可能同时存在：

```text
项目测试依赖
│
├── JUnit 4
│   └── junit:junit:4.13.2
│       └── org.junit.Test
│
└── JUnit Jupiter
    └── org.junit.jupiter...
        └── org.junit.jupiter.api.Test
```

因此 IDEA 自动补全时会同时显示：

```text
Test  org.junit
Test  org.junit.jupiter.api
```

这并不代表两个注解来自同一个依赖。

---

## 4. 为什么 `org.junit.Test` 可以正常运行

当前我们明确添加的是：

```xml
<groupId>junit</groupId>
<artifactId>junit</artifactId>
<version>4.13.2</version>
```

所以：

```java
import org.junit.Test;
```

与当前 JUnit 4 依赖完全匹配。

例如：

```java
package com.hs.August.day10junit;

import org.junit.Test;

public class JunitTest {

    @Test
    public void testHello() {
        System.out.println("Hello World!");
    }
}
```

运行过程可以理解为：

```text
@Test
  ↓
org.junit.Test
  ↓
JUnit 4
  ↓
识别 testHello()
  ↓
执行测试
  ↓
测试成功
```

运行结果：

```text
Tests passed: 1 of 1 test
Hello World!
Process finished with exit code 0
```

---

## 5. 为什么 `org.junit.jupiter.api.Test` 会报错

当写成：

```java
import org.junit.jupiter.api.Test;
```

时，IDEA 会把这个测试当作 JUnit Jupiter 测试来运行。

JUnit Jupiter 的运行依赖于 JUnit Platform。

大致流程：

```text
org.junit.jupiter.api.Test
        ↓
JUnit Jupiter
        ↓
JUnit Platform
        ↓
测试发现与执行
```

之前出现的错误：

```text
java.lang.NoSuchMethodError:
org.junit.platform.engine.discovery.MethodSelector
.getMethodParameterTypes()
```

说明：

```text
某个测试运行组件
        ↓
尝试调用
MethodSelector.getMethodParameterTypes()
        ↓
JVM 确实找到了 MethodSelector 类
        ↓
但当前实际加载到的版本中
不存在这个方法
        ↓
NoSuchMethodError
```

所以这个错误不是简单的：

```text
找不到 JUnit
```

而更接近：

```text
JUnit Jupiter / JUnit Platform / 测试运行器
之间的版本不兼容
```

---

## 6. 为什么这是运行时错误，而不是编译错误

如果：

```java
import org.junit.jupiter.api.Test;
```

完全不存在，那么 Java 编译阶段就会直接报：

```text
Cannot resolve symbol 'jupiter'
```

但是之前代码能够成功编译，说明：

```text
org.junit.jupiter.api.Test
```

确实存在于项目 classpath 中。

真正出问题的是运行阶段。

因此：

```text
编译成功
    ↓
说明 Jupiter API 存在

运行失败
    ↓
说明运行测试时的 JUnit Platform
或测试运行器版本之间存在不兼容
```

---

## 7. `NoSuchMethodError` 的本质

`NoSuchMethodError` 和 `ClassNotFoundException` 不一样。

### ClassNotFoundException

表示：

```text
连这个类都找不到
```

### NoSuchMethodError

表示：

```text
类找到了
    ↓
但是准备调用的方法不存在
```

例如某个组件认为：

```java
selector.getMethodParameterTypes();
```

存在。

但是 JVM 实际加载到的 `MethodSelector` 版本里面没有这个方法。

于是产生：

```text
NoSuchMethodError
```

这种错误经常说明：

```text
编译时依赖版本
        ≠
运行时实际加载版本
```

或者：

```text
框架中的多个组件版本没有对齐
```

---

## 8. 当前项目中应该怎么选择

当前学习目标是使用手动导入的：

```text
JUnit 4.13.2
```

因此应该选择：

```java
import org.junit.Test;
```

而不是：

```java
import org.junit.jupiter.api.Test;
```

最清晰的对应关系：

```text
JUnit 4
================================

Maven：
junit:junit:4.13.2

@Test：
org.junit.Test

导入：
import org.junit.Test;


JUnit Jupiter
================================

Maven：
org.junit.jupiter:...

@Test：
org.junit.jupiter.api.Test

导入：
import org.junit.jupiter.api.Test;
```

---

## 9. 最容易犯的错误

错误理解：

```text
org.junit.Test

和

org.junit.jupiter.api.Test

是同一个 @Test 的两种导入方式
```

这是错误的。

正确理解：

```text
org.junit.Test
        ↓
JUnit 4 的 @Test

org.junit.jupiter.api.Test
        ↓
JUnit Jupiter 的 @Test
```

它们：

- 名称相同；
- 功能目的相似；
- 包名不同；
- 所属依赖不同；
- 测试运行机制也不同。

---

## 10. 当前推荐写法

既然 `pom.xml` 中明确使用：

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
</dependency>
```

那么当前测试代码统一使用：

```java
import org.junit.Test;
```

示例：

```java
package com.hs.August.day10junit;

import org.junit.Test;

public class JunitTest {

    @Test
    public void testHello() {
        System.out.println("Hello World!");
    }
}
```

这样依赖与注解是一一对应的：

```text
pom.xml
   ↓
junit:junit:4.13.2
   ↓
JUnit 4
   ↓
org.junit.Test
   ↓
@Test
   ↓
测试正常执行
```

---

## 11. 一句话总结

> `org.junit.Test` 和 `org.junit.jupiter.api.Test` 虽然都叫 `@Test`，但它们来自不同的 JUnit 依赖和测试体系。当前项目显式导入的是 JUnit 4.13.2，所以使用 `org.junit.Test` 可以正常运行；使用 `org.junit.jupiter.api.Test` 会进入 JUnit Jupiter / JUnit Platform 的运行链路，而当前项目中的相关组件存在版本兼容问题，因此产生 `NoSuchMethodError`。
