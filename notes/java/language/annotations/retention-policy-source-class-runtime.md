# Java `@Retention`：SOURCE / CLASS / RUNTIME 完整笔记

## 1. `@Retention` 是什么？

Java 中：

```java
package java.lang.annotation;

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Retention {
    RetentionPolicy value();
}
```

`@Retention` 是一个元注解。

所谓“元注解”，就是：

> 用来修饰其他注解的注解。

例如：

```java
@Retention(RetentionPolicy.RUNTIME)
@interface RuntimeAnnotation {
}
```

这里：

```java
@Retention(...)
```

不是修饰普通类，而是在说明：

> `RuntimeAnnotation` 这个注解要保留到什么时候。

---

## 2. `RetentionPolicy` 有哪三种？

Java 中：

```java
public enum RetentionPolicy {
    SOURCE,
    CLASS,
    RUNTIME
}
```

它们分别表示：

```text
SOURCE
CLASS
RUNTIME
```

核心区别是：

> 注解最多能保留到 Java 生命周期中的哪个阶段。

---

# 3. Java 代码从 `.java` 到运行时

先理解整体流程：

```text
.java 源代码
    ↓
javac 编译
    ↓
.class 字节码文件
    ↓
JVM 加载
    ↓
Class 对象
    ↓
Reflection 反射
```

`RetentionPolicy` 决定注解能走到哪一步。

---

# 4. SOURCE

定义：

```java
@Retention(RetentionPolicy.SOURCE)
@interface SourceAnnotation {
}
```

使用：

```java
@SourceAnnotation
class Demo {
}
```

在源文件中：

```text
Demo.java

@SourceAnnotation
class Demo {
}
```

但是经过：

```text
javac
```

以后：

```text
Demo.class
```

中不会再保存这个注解。

可以理解成：

```text
.java
@SourceAnnotation
      ↓
    javac
      ↓
.class
注解被丢弃
```

所以反射：

```java
Demo.class.getDeclaredAnnotation(SourceAnnotation.class);
```

结果是：

```text
null
```

原因不是反射 API 有问题，而是：

> `.class` 里压根就已经没有这个注解了。

---

## 5. SOURCE 适合什么场景？

SOURCE 一般用于：

> 只在编译期间有意义的注解。

典型思路是：

```text
写代码时存在
    ↓
编译器 / 注解处理器使用
    ↓
编译结束后没有必要继续保存
```

它不需要被 JVM 在运行时读取。

---

# 6. CLASS

定义：

```java
@Retention(RetentionPolicy.CLASS)
@interface ClassAnnotation {
}
```

使用：

```java
@ClassAnnotation
class Demo {
}
```

这次经过 javac：

```text
.java
@ClassAnnotation
      ↓
    javac
      ↓
.class
仍然保存
```

所以：

> CLASS 注解会进入 `.class` 文件。

但是它通常不会作为“运行时可见注解”暴露给标准 Java Reflection API。

因此：

```java
Demo.class.getDeclaredAnnotation(ClassAnnotation.class);
```

结果仍然是：

```text
null
```

---

## 7. CLASS 最容易误解的地方

很多人会想：

> 既然 CLASS 注解在 `.class` 里，那为什么反射拿不到？

因为：

```text
存在于 .class
```

和：

```text
运行时可通过 Reflection 获取
```

不是一回事。

CLASS 的含义是：

> 编译器要把它记录到 `.class` 中，但 JVM 不要求把它作为运行时反射可见信息保留下来。

所以：

```text
CLASS
→ .class 中存在
→ 普通运行时反射拿不到
```

---

# 8. RUNTIME

定义：

```java
@Retention(RetentionPolicy.RUNTIME)
@interface RuntimeAnnotation {
}
```

使用：

```java
@RuntimeAnnotation
class Demo {
}
```

这次：

```text
.java
@RuntimeAnnotation
      ↓
    javac
      ↓
.class
仍然存在
      ↓
JVM 加载
      ↓
Reflection 可以读取
```

因此：

```java
Demo.class.getDeclaredAnnotation(RuntimeAnnotation.class);
```

会返回：

```text
@RuntimeAnnotation()
```

而不是 `null`。

---

# 9. 三种模式最核心对比

```text
                         .java
                           │
             ┌─────────────┼─────────────┐
             ↓             ↓             ↓
          SOURCE         CLASS         RUNTIME
             │             │             │
          javac          javac          javac
             │             │             │
             ↓             ↓             ↓
.class      没有           有             有

Reflection  拿不到         拿不到         拿得到
```

再写成表格：

| RetentionPolicy | `.java` 中存在 | `.class` 中存在 | 运行时 Reflection 可见 |
|---|---:|---:|---:|
| SOURCE | ✅ | ❌ | ❌ |
| CLASS | ✅ | ✅ | ❌ |
| RUNTIME | ✅ | ✅ | ✅ |

---

# 10. 最推荐的测试代码

```java
package com.rainkoli.javase.annotation.retention;

import org.junit.jupiter.api.Test;

import java.lang.annotation.Annotation;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.util.Arrays;

import static org.junit.jupiter.api.Assertions.*;

class RetentionPolicyTest {

    @Retention(RetentionPolicy.SOURCE)
    @interface SourceAnnotation {
    }

    @Retention(RetentionPolicy.CLASS)
    @interface ClassAnnotation {
    }

    @Retention(RetentionPolicy.RUNTIME)
    @interface RuntimeAnnotation {
    }

    @SourceAnnotation
    @ClassAnnotation
    @RuntimeAnnotation
    static class DemoClass {
    }

    @Test
    void testGetDeclaredAnnotation() {
        SourceAnnotation source =
                DemoClass.class.getDeclaredAnnotation(SourceAnnotation.class);

        ClassAnnotation clazz =
                DemoClass.class.getDeclaredAnnotation(ClassAnnotation.class);

        RuntimeAnnotation runtime =
                DemoClass.class.getDeclaredAnnotation(RuntimeAnnotation.class);

        System.out.println("SOURCE  -> " + source);
        System.out.println("CLASS   -> " + clazz);
        System.out.println("RUNTIME -> " + runtime);

        assertNull(source);
        assertNull(clazz);
        assertNotNull(runtime);
    }

    @Test
    void testGetAllRuntimeVisibleAnnotations() {
        Annotation[] annotations =
                DemoClass.class.getDeclaredAnnotations();

        Arrays.stream(annotations)
                .forEach(System.out::println);

        assertEquals(1, annotations.length);
        assertEquals(
                RuntimeAnnotation.class,
                annotations[0].annotationType()
        );
    }
}
```

预期结果：

```text
SOURCE  -> null
CLASS   -> null
RUNTIME -> @...RuntimeAnnotation()
```

---

# 11. 为什么 `getDeclaredAnnotations()` 只能看到 RUNTIME？

代码：

```java
Annotation[] annotations =
        DemoClass.class.getDeclaredAnnotations();
```

它是运行时反射。

因此只能看到：

```text
运行时可见的注解
```

而三种保留策略里：

```text
SOURCE  → 编译后已经消失
CLASS   → 虽在 class 中，但不是运行时可见
RUNTIME → 运行时可见
```

所以只会得到：

```text
RuntimeAnnotation
```

---

# 12. 和 Spring `@Configuration` 的关系

之前 Spring 中会写：

```java
Configuration configuration =
        Day24AppConfigDemo.class
                .getAnnotation(Configuration.class);
```

然后：

```java
System.out.println(configuration);
```

之所以能够拿到：

```text
@Configuration(...)
```

本质原因就是：

> `@Configuration` 在运行时仍然存在，因此 Spring 和 Java Reflection 才有机会读取它。

如果某个注解是：

```java
@Retention(RetentionPolicy.SOURCE)
```

那么经过编译后：

```text
.class 中已经没有它
```

Spring 在运行时自然也拿不到。

---

# 13. `getAnnotation()` 和 `getDeclaredAnnotation()` 的区别

你现在学习 Retention 时，更推荐先用：

```java
getDeclaredAnnotation()
```

因为它只检查：

> 当前类自己直接声明的注解。

例如：

```java
DemoClass.class.getDeclaredAnnotation(RuntimeAnnotation.class);
```

非常适合做这个实验。

而：

```java
getAnnotation()
```

还会涉及：

```java
@Inherited
```

以及父类继承的情况。

因此刚开始研究 Retention 时，不要把 `@Inherited` 的影响混进来。

---

# 14. 更底层：查看 `.class`

如果只看：

```java
getDeclaredAnnotation(...)
```

你只能得到：

```text
SOURCE -> null
CLASS  -> null
RUNTIME -> 非 null
```

但是 SOURCE 和 CLASS 都是 `null`，还不能直接看出它们之间的差别。

真正想看懂 CLASS，应该检查 `.class`。

编译后测试类通常在：

```text
target/test-classes/
```

例如：

```text
target/test-classes/
└── com/rainkoli/javase/annotation/retention/
    └── RetentionPolicyTest$DemoClass.class
```

可以使用：

```bash
javap -v RetentionPolicyTest$DemoClass.class
```

重点观察：

```text
RuntimeVisibleAnnotations
RuntimeInvisibleAnnotations
```

通常可以理解为：

```text
RUNTIME
→ RuntimeVisibleAnnotations

CLASS
→ RuntimeInvisibleAnnotations

SOURCE
→ class 文件里没有记录
```

因此：

```text
SOURCE
.java 有
.class 没有

CLASS
.java 有
.class 有
反射不可见

RUNTIME
.java 有
.class 有
反射可见
```

这才是三者最本质的差别。

---

# 15. 最终记忆方式

可以直接记：

```text
SOURCE
→ 只活到源代码阶段

CLASS
→ 活到 class 文件阶段

RUNTIME
→ 活到 JVM 运行阶段
```

或者：

```text
SOURCE  = 源码可见
CLASS   = 字节码可见
RUNTIME = 反射可见
```

但要注意：

```text
RUNTIME 也一定存在于 .class 中
```

因此它的生命周期最长。

最终关系：

```text
SOURCE < CLASS < RUNTIME
```

表示保留阶段越来越长。

---

# 16. 一句话总结

`@Retention` 不是决定“注解能不能写”，而是决定：

> 这个注解从 `.java` 开始，最多能被保留到编译后的 `.class`，还是继续保留到 JVM 运行时供 Reflection / Framework 使用。
