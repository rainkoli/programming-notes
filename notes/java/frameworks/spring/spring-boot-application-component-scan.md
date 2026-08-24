# `@SpringBootApplication` 自带包扫描功能总结

## 1. 核心结论

Spring Boot 中：

```java
@SpringBootApplication
```

本身就带有**组件扫描（Component Scan）**能力。

当前 Spring Boot 的 `@SpringBootApplication` 是一个组合注解，核心包含：

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
```

因此，在普通 Spring Boot 项目中，通常**不需要再额外添加 `@ComponentScan`**。

---

## 2. 默认从哪里开始扫描

假设启动类如下：

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

那么默认组件扫描的基础范围可以理解为：

```text
com.example.demo
```

以及它下面的所有子包。

例如：

```text
com.example.demo
├── DemoApplication.java
├── controller
│   └── UserController.java
├── service
│   └── UserService.java
├── repository
│   └── UserRepository.java
└── config
    └── WebConfig.java
```

这些子包都位于：

```text
com.example.demo
```

之下，因此正常情况下可以被扫描到。

---

## 3. 默认会扫描哪些 Spring 组件

Spring 的组件扫描通常可以发现带有以下注解的类：

```java
@Component
@Service
@Repository
@Controller
@Configuration
```

以及基于 `@Component` 派生的其他 stereotype 注解。

例如：

```java
@Service
public class UserService {
}
```

只要 `UserService` 位于启动类所在包或它的子包中，就可以通过组件扫描注册为 Spring Bean。

`@RestController` 在 Web 项目中也属于常见的 Spring 组件注解，因此通常同样会被扫描并注册。

---

## 4. 为什么启动类通常放在最外层包

推荐结构：

```text
com.example.project
├── ProjectApplication.java
├── controller
├── service
├── repository
├── entity
└── config
```

启动类：

```java
package com.example.project;

@SpringBootApplication
public class ProjectApplication {
}
```

这样：

```text
controller
service
repository
config
...
```

都位于：

```text
com.example.project
```

下面，默认扫描范围就能自然覆盖整个项目。

Spring Boot 官方也推荐将主启动类放在项目的根包（root package）中。

---

## 5. 启动类位置太深时会发生什么

例如项目结构：

```text
com.example.project
├── controller
│   └── UserController.java
├── service
│   └── UserService.java
└── application
    └── ProjectApplication.java
```

启动类位于：

```java
package com.example.project.application;
```

那么默认组件扫描主要会从：

```text
com.example.project.application
```

开始。

于是同级的：

```text
com.example.project.controller
com.example.project.service
```

不属于 `application` 的子包，就可能无法被默认的组件扫描发现。

因此更推荐把启动类移动到：

```text
com.example.project
```

这一层。

---

## 6. 自定义扫描包

如果确实不能调整启动类的位置，可以直接通过 `@SpringBootApplication` 指定组件扫描范围。

### 写法一：`scanBasePackages`

```java
@SpringBootApplication(
    scanBasePackages = {
        "com.example.project",
        "com.example.common"
    }
)
public class ProjectApplication {
}
```

这相当于为 `@ComponentScan` 指定 `basePackages`。

单个包也可以写成：

```java
@SpringBootApplication(scanBasePackages = "com.example.project")
public class ProjectApplication {
}
```

---

## 7. 更推荐的类型安全写法

除了字符串包名，还可以使用：

```java
scanBasePackageClasses
```

例如：

```java
@SpringBootApplication(
    scanBasePackageClasses = {
        ProjectApplication.class,
        CommonPackageMarker.class
    }
)
public class ProjectApplication {
}
```

Spring 会扫描这些类所在的包。

这种方式的优点是：

- 不依赖手写字符串包名；
- IDE 重构包名时更安全；
- 包名修改后更不容易出现字符串未同步的问题。

实际项目中也可以专门创建一个空的 marker 类或接口，用来表示需要扫描的包。

---

## 8. `scanBasePackages` 只影响组件扫描

这是一个非常重要的区别。

例如：

```java
@SpringBootApplication(
    scanBasePackages = "com.example"
)
```

这里的：

```java
scanBasePackages
```

是 `@ComponentScan` 的别名，它主要控制的是 **Spring Component 扫描**。

Spring Boot 官方 API 明确说明：这个属性**不会直接修改 JPA `@Entity` 的扫描范围，也不会直接修改 Spring Data Repository 的扫描范围**。

如果需要单独指定 JPA Entity，可以使用：

```java
@EntityScan("com.example.entity")
```

如果需要单独指定 Spring Data JPA Repository，可以使用：

```java
@EnableJpaRepositories("com.example.repository")
```

例如：

```java
@SpringBootApplication(scanBasePackages = "com.example")
@EntityScan("com.example.entity")
@EnableJpaRepositories("com.example.repository")
public class ProjectApplication {
}
```

---

## 9. 默认搜索包与 JPA 的关系

虽然：

```java
@SpringBootApplication(scanBasePackages = ...)
```

中的 `scanBasePackages` 本身只是在配置 `@ComponentScan`，但 Spring Boot 会把主应用类所在的包作为一个重要的默认“搜索包”。

在常规 JPA 项目布局下，如果启动类放在：

```text
com.example.project
```

而实体放在：

```text
com.example.project.entity
```

通常无需额外写：

```java
@EntityScan
```

这也是为什么 Spring Boot 推荐把启动类放在整个应用的根包。

---

## 10. 不建议使用 Java 默认包

例如启动类没有：

```java
package com.example.project;
```

而是直接写：

```java
@SpringBootApplication
public class Application {
}
```

这种情况下类位于 Java 的默认包（default package）。

Spring Boot 官方不推荐这种方式，因为使用 `@ComponentScan`、`@EntityScan`、`@SpringBootApplication` 等扫描机制时，可能导致非常大的扫描范围，甚至检查 classpath 中大量类。

因此应该正常声明包名，例如：

```java
package com.example.project;
```

---

## 11. 常见问题

### 问题 1：已经写了 `@SpringBootApplication`，还要不要写 `@ComponentScan`？

通常不需要。

因为：

```java
@SpringBootApplication
```

本身已经包含：

```java
@ComponentScan
```

只有在确实需要定制扫描规则时，才需要额外调整组件扫描配置。

### 问题 2：为什么 `@Service` 明明写了，还是注入失败？

首先检查它是不是位于启动类所在包的子包中。

例如：

```text
启动类：
com.example.app.Application

Service：
com.example.service.UserService
```

这里：

```text
com.example.service
```

不是：

```text
com.example.app
```

的子包，因此默认组件扫描可能扫描不到。

可以：

1. 把启动类移动到共同父包 `com.example`；或
2. 使用 `scanBasePackages` 显式指定扫描范围。

### 问题 3：`scanBasePackages` 配了以后，为什么 JPA Entity 还是找不到？

因为：

```java
scanBasePackages
```

只是 `@ComponentScan` 的配置别名，并不等于：

```java
@EntityScan
```

必要时应单独配置 JPA Entity 和 Repository 的扫描范围。

---

## 12. 推荐项目结构

比较推荐：

```text
com.example.project
├── ProjectApplication.java
├── controller
│   └── UserController.java
├── service
│   └── UserService.java
├── repository
│   └── UserRepository.java
├── entity
│   └── User.java
└── config
    └── AppConfig.java
```

启动类：

```java
package com.example.project;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ProjectApplication {

    public static void main(String[] args) {
        SpringApplication.run(ProjectApplication.class, args);
    }
}
```

这种结构下，大部分情况下都不需要手动指定组件扫描包。

---

## 13. 一句话记忆

可以把：

```java
@SpringBootApplication
```

简单记成：

```text
Spring Boot 配置
+ 自动配置
+ Component 包扫描
```

而默认扫描范围可以记成：

```text
启动类所在包 + 所有子包
```

所以最重要的项目结构原则是：

> **把 Spring Boot 启动类放在项目代码的共同根包中。**

---

## 14. 官方参考资料

- Spring Boot `@SpringBootApplication` API：  
  https://docs.spring.io/spring-boot/api/java/org/springframework/boot/autoconfigure/SpringBootApplication.html

- Spring Boot — Structuring Your Code：  
  https://docs.spring.io/spring-boot/reference/using/structuring-your-code.html

- Spring Framework — Classpath Scanning and Managed Components：  
  https://docs.spring.io/spring-framework/reference/core/beans/classpath-scanning.html
