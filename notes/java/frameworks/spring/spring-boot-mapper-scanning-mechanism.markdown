# Spring Boot、ComponentScan 与 MyBatis Mapper 扫描机制总结

## 1. 问题背景

测试类：

```java
package com.hs.homework.August.day19mybatis.homework;

import com.hs.homework.August.day19mybatis.mapper.Day19ArticleMapper;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
public class Day19DemoTest {

    @Autowired
    Day19ArticleMapper mapper;

    @Test
    public void testDeleteBatch() {
        System.out.println(123);
    }
}
```

Mapper：

```java
@Mapper
public interface Day19ArticleMapper {
}
```

一开始 IDEA / Spring 测试提示：

```text
Could not autowire.
No beans of 'Day19ArticleMapper' type found.
```

运行测试时真正的关键异常是：

```text
Unable to find a @SpringBootConfiguration by searching packages upwards from the test.
```

后来添加：

```java
package com.hs.homework.August.day19mybatis;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Day19StartApplication {

    public static void main(String[] args) {
        SpringApplication.run(Day19StartApplication.class, args);
    }
}
```

之后 `Day19ArticleMapper mapper` 就可以正常被 Spring 注入。

---

## 2. 真正的根因

问题并不是 `@Mapper` 没有作用。

真正的问题是：

```text
@SpringBootTest
    ↓
找不到 @SpringBootApplication / @SpringBootConfiguration
    ↓
Spring ApplicationContext 无法创建
    ↓
MyBatis 自动配置无法启动
    ↓
@Mapper 无法被 MyBatis 扫描和处理
    ↓
Day19ArticleMapper 的代理 Bean 不存在
    ↓
@Autowired 注入失败
```

因此：

> `@Mapper` 生效的前提，是 Spring Boot 测试上下文能够先正常启动。

---

## 3. `@SpringBootTest` 如何寻找启动配置类

测试类位于：

```text
com.hs.homework.August.day19mybatis.homework.Day19DemoTest
```

当只写：

```java
@SpringBootTest
```

而没有显式指定：

```java
@SpringBootTest(classes = Day19StartApplication.class)
```

Spring Boot 会从测试类所在包开始向父包查找 Spring Boot 主配置类：

```text
com.hs.homework.August.day19mybatis.homework
                         ↑
com.hs.homework.August.day19mybatis
                         ↑
com.hs.homework.August
                         ↑
com.hs.homework
                         ↑
...
```

添加：

```java
package com.hs.homework.August.day19mybatis;

@SpringBootApplication
public class Day19StartApplication {
}
```

之后，测试类向上搜索时就能找到它。

因此：

```text
Day19DemoTest
    ↓
@SpringBootTest
    ↓
向父包查找
    ↓
找到 Day19StartApplication
    ↓
发现 @SpringBootApplication
    ↓
将其作为 Spring Boot 主配置类
    ↓
创建 ApplicationContext
```

---

## 4. 为什么没有写 `@ComponentScan` 也能扫描 Bean

因为：

```java
@SpringBootApplication
```

本身就是一个组合注解。

可以从概念上理解为组合了：

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
```

所以：

```java
@SpringBootApplication
public class Day19StartApplication {
}
```

虽然没有显式写：

```java
@ComponentScan(...)
```

但实际上已经启用了 Spring 的组件扫描机制。

---

## 5. 默认扫描范围是什么

启动类位于：

```text
com.hs.homework.August.day19mybatis
```

因此默认组件扫描的基础包就是：

```text
com.hs.homework.August.day19mybatis
```

并递归扫描它的子包。

例如：

```text
com.hs.homework.August.day19mybatis
│
├── Day19StartApplication
│
├── controller
│   └── ArticleController
│
├── service
│   └── ArticleService
│
├── component
│   └── XxxComponent
│
├── mapper
│   └── Day19ArticleMapper
│
└── homework
    └── Day19DemoTest
```

普通 Spring 组件，例如：

```java
@Service
public class ArticleService {
}
```

会因为 `@ComponentScan` 被扫描并注册成 Bean。

因此：

```text
@SpringBootApplication
    ↓
@ComponentScan
    ↓
com.hs.homework.August.day19mybatis
    ↓
递归扫描子包
    ↓
发现 @Service / @Controller / @Component / @Repository
    ↓
创建普通 Spring Bean
```

---

## 6. `@ComponentScan` 和 `@MapperScan` 的区别

虽然它们名字里都有 `Scan`，但负责的事情不同。

| 注解 | 所属体系 | 主要扫描对象 | Bean 的创建方式 |
|---|---|---|---|
| `@ComponentScan` | Spring | `@Component`、`@Service`、`@Controller`、`@Repository` 等 | Spring 创建普通对象 |
| `@MapperScan` | MyBatis-Spring | Mapper 接口 | MyBatis 创建动态代理对象 |
| `@Mapper` | MyBatis | 标记单个 Mapper 接口 | 由 MyBatis 扫描后创建代理 Bean |

普通 Spring Bean：

```java
@Service
public class ArticleService {
}
```

Spring 可以正常创建：

```text
ArticleService
    ↓
调用构造器
    ↓
创建 ArticleService 实例
    ↓
注册到 IoC 容器
```

但是 Mapper 是接口：

```java
public interface Day19ArticleMapper {
}
```

接口不能直接：

```java
new Day19ArticleMapper();
```

因此 Mapper 的 Bean 不是普通的实例化过程，而是：

```text
Mapper 接口
    ↓
MyBatis 扫描
    ↓
MapperFactoryBean
    ↓
创建 Mapper 动态代理
    ↓
注册到 Spring IoC 容器
```

---

## 7. `@Mapper` 为什么最终能够被自动扫描

项目中使用了：

```text
mybatis-spring-boot-starter
```

而：

```java
@SpringBootApplication
```

包含：

```java
@EnableAutoConfiguration
```

因此 Spring Boot 会根据 classpath 中存在的依赖加载对应的自动配置。

当 MyBatis Starter 存在时：

```text
@SpringBootApplication
    ↓
@EnableAutoConfiguration
    ↓
发现 MyBatis Starter
    ↓
启用 MyBatisAutoConfiguration
    ↓
启动 Mapper Scanner
    ↓
扫描 @Mapper
```

因此：

```java
@Mapper
public interface Day19ArticleMapper {
}
```

最终会经历：

```text
Day19ArticleMapper
    ↓
发现 @Mapper
    ↓
MyBatis Mapper Scanner
    ↓
MapperFactoryBean
    ↓
MapperProxy
    ↓
注册 Spring Bean
    ↓
@Autowired 成功
```

---

## 8. `@Mapper` 和 `@MapperScan` 的关系

### 方案一：每个 Mapper 使用 `@Mapper`

```java
@Mapper
public interface UserMapper {
}

@Mapper
public interface ArticleMapper {
}
```

这种方式下，通常不需要额外写：

```java
@MapperScan(...)
```

因为 MyBatis Spring Boot Starter 会自动寻找带有 `@Mapper` 的接口。

---

### 方案二：统一使用 `@MapperScan`

启动类：

```java
@SpringBootApplication
@MapperScan("com.hs.homework.August.day19mybatis.mapper")
public class Day19StartApplication {
}
```

Mapper：

```java
public interface UserMapper {
}

public interface ArticleMapper {
}
```

此时可以不在每个 Mapper 上重复添加 `@Mapper`。

可以记忆为：

```text
@Mapper
    = 标记单个 Mapper

@MapperScan
    = 指定整个 Mapper 包进行扫描
```

---

## 9. `@ComponentScan` 并不是直接创建 Mapper Bean

容易产生一个错误理解：

```text
@SpringBootApplication
    ↓
@ComponentScan
    ↓
扫描到 @Mapper
    ↓
创建 Mapper Bean
```

这个理解不够准确。

更准确的是：

```text
@SpringBootApplication
        │
        ├───────────────┐
        │               │
        ↓               ↓
@ComponentScan    @EnableAutoConfiguration
        │               │
        ↓               ↓
普通 Spring Bean   MyBatisAutoConfiguration
                        ↓
                   Mapper Scanner
                        ↓
                     @Mapper
                        ↓
                  Mapper Proxy Bean
```

因此：

- 普通 Spring Bean 主要由 `@ComponentScan` 负责发现；
- Mapper 接口由 MyBatis 的 Mapper Scanner 负责处理；
- 最终它们都会被注册进同一个 Spring IoC 容器。

---

## 10. 为什么 `Day19StartApplication` 放在 `day19mybatis` 包下很合适

当前结构：

```text
com.hs.homework.August.day19mybatis
│
├── Day19StartApplication
│
├── mapper
│   └── Day19ArticleMapper
│
└── homework
    └── Day19DemoTest
```

它同时满足两个方向。

### 测试配置类查找：向上

```text
Day19DemoTest
    ↓
所在包：
day19mybatis.homework
    ↑
向父包寻找
    ↑
day19mybatis
    ↑
找到 Day19StartApplication
```

### Bean 扫描：向下

```text
Day19StartApplication
    ↓
基础包：
day19mybatis
    ↓
扫描子包
    ↓
mapper
service
controller
...
```

可以把启动类理解为整个模块的“根节点”：

```text
              测试向上寻找
                    ↑
                    │
        Day19StartApplication
          @SpringBootApplication
                    │
        ┌───────────┴───────────┐
        ↓                       ↓
@ComponentScan          AutoConfiguration
        ↓                       ↓
普通 Spring Bean         MyBatis 自动配置
                                ↓
                           Mapper Scanner
                                ↓
                             @Mapper
```

---

## 11. `main()` 方法不是 `@SpringBootTest` 成功的直接原因

正常启动 Spring Boot 应用时：

```java
public static void main(String[] args) {
    SpringApplication.run(Day19StartApplication.class, args);
}
```

执行链路是：

```text
main()
    ↓
SpringApplication.run()
    ↓
启动 Spring Boot
```

但 `@SpringBootTest` 默认并不是先调用这个 `main()` 方法。

测试更主要的是：

```text
@SpringBootTest
    ↓
寻找 Spring Boot 配置类
    ↓
找到 @SpringBootApplication
    ↓
使用该配置创建测试 ApplicationContext
```

因此，针对之前的 Bean 问题，真正关键的是：

```java
@SpringBootApplication
public class Day19StartApplication {
}
```

而不是 `main()` 方法本身。

`main()` 主要用于正常运行应用程序。

---

## 12. 完整启动与 Bean 创建流程

整个过程可以最终整理为：

```text
Day19DemoTest
    │
    │ @SpringBootTest
    ▼
从测试包向父包查找配置类
    │
    ▼
Day19StartApplication
    │
    │ @SpringBootApplication
    ▼
┌────────────────────────────────────┐
│ @SpringBootConfiguration           │
│ @EnableAutoConfiguration           │
│ @ComponentScan                     │
└────────────────────────────────────┘
          │                  │
          │                  │
          ▼                  ▼
@ComponentScan       MyBatisAutoConfiguration
          │                  │
          ▼                  ▼
扫描普通组件           Mapper Scanner
          │                  │
          ▼                  ▼
@Service 等          搜索 @Mapper
                             │
                             ▼
                   Day19ArticleMapper
                             │
                             ▼
                     MapperFactoryBean
                             │
                             ▼
                       Mapper Proxy
                             │
                             ▼
                   注册到 Spring IoC 容器
                             │
                             ▼
@Autowired Day19ArticleMapper mapper
                             │
                             ▼
                           成功
```

---

## 13. 添加启动类前后的区别

### 添加启动类之前

```text
@SpringBootTest
    ↓
找不到 @SpringBootConfiguration
    ↓
ApplicationContext 启动失败
    ↓
MyBatis 自动配置没有执行
    ↓
@Mapper 没有被处理
    ↓
Day19ArticleMapper Bean 不存在
    ↓
@Autowired 失败
```

### 添加启动类之后

```text
@SpringBootTest
    ↓
找到 Day19StartApplication
    ↓
找到 @SpringBootApplication
    ↓
ApplicationContext 正常创建
    ↓
Spring Boot 自动配置生效
    ↓
MyBatis 自动配置生效
    ↓
扫描 @Mapper
    ↓
创建 Day19ArticleMapper 代理对象
    ↓
注册进 Spring IoC 容器
    ↓
@Autowired 成功
```

---

## 14. 最终结论

这次问题不能简单理解为：

> 加了 `@SpringBootApplication` 后，`@ComponentScan` 自动把 `Day19ArticleMapper` 创建成了 Bean。

更准确的理解是：

> `@SpringBootApplication` 让 `@SpringBootTest` 找到了 Spring Boot 的主配置类并成功创建 ApplicationContext。  
> `@SpringBootApplication` 自身包含组件扫描和自动配置能力。  
> `@ComponentScan` 负责普通 Spring 组件扫描，而 `@EnableAutoConfiguration` 使 MyBatis Spring Boot Starter 的自动配置生效。  
> MyBatis 的 Mapper Scanner 再扫描 `@Mapper` 接口，为 `Day19ArticleMapper` 创建动态代理对象，并将其注册进 Spring IoC 容器。  
> 因此 `@Autowired Day19ArticleMapper mapper` 最终才能成功注入。

可以压缩成一句话：

```text
@SpringBootTest
→ 找到 @SpringBootApplication
→ Spring Context 创建成功
→ Spring Boot 自动配置启动
→ MyBatis 自动配置启动
→ 扫描 @Mapper
→ 创建 Mapper Proxy
→ 注册 Bean
→ @Autowired 成功
```
