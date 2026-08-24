# Java 笔记

Java 内容按稳定知识领域划分，不再使用会随学习阶段失真的 `basic` 分类。

## 结构

| 目录 | 内容 |
| --- | --- |
| [language/fundamentals](./language/fundamentals/) | 变量、初始化、`static`、入口方法和类字面量 |
| [language/object-oriented-programming](./language/object-oriented-programming/) | 封装、继承、接口和多态 |
| [language/type-system](./language/type-system/) | 泛型、协变、逆变与不变 |
| [language/annotations](./language/annotations/) | Java 注解语义 |
| [language/jls](./language/jls/) | Java Language Specification 章节笔记 |
| [jvm](./jvm/) | JVM、字节码、运行时和 JIT |
| [frameworks](./frameworks/) | Spring、MyBatis、JUnit 与持久化框架 |

当出现 Java 标准库 API 的使用或行为笔记时，建立 `standard-library/`；如果分析的是 JDK 内部实现，则放入 [JDK 源码分析](../source-code-analysis/jdk/)。构建和发布工具放入 [tools/java-build](../tools/java-build/)。

## 快速入口

- [变量](./language/fundamentals/variable.md)
- [多态](./language/object-oriented-programming/polymorphism.md)
- [接口方法](./language/object-oriented-programming/java-interface-methods-abstract-default-static-private.md)
- [泛型](./language/type-system/genericity.md)
- [型变总结](./language/type-system/variance/variance-summary.md)
- [RetentionPolicy](./language/annotations/retention-policy-source-class-runtime.md)
- [JLS Chapter 5](./language/jls/chapter-05-conversions-and-contexts.md)
- [Jeandle JDK 与 JIT](./jvm/jeandle-jdk.md)
- [Spring 依赖注入](./frameworks/spring/dependency-injection-detailed-notes.md)
- [MyBatis 配置](./frameworks/mybatis/mybatis-spring-boot-configuration-explained.md)

## 边界

- 框架的使用、配置和概念放在 `frameworks/`。
- 框架的具体源码调用链放在 [source-code-analysis](../source-code-analysis/README.md)。
- 可运行的 Java 实验放在 [demos/java](../../demos/java/)。

返回 [笔记总索引](../README.md)。
