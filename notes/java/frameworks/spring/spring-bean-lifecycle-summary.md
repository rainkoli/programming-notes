# Spring Bean 生命周期总结：`@PostConstruct/@PreDestroy` 与 `@Bean(initMethod/destroyMethod)`

## 1. 两种生命周期配置方式

Spring Bean 的初始化与销毁回调常见有两种写法。

### 方式一：`@PostConstruct` / `@PreDestroy`

生命周期方法直接声明在 Bean 类内部：

```java
@Component("Man")
@Scope("singleton")
public class Man {

    public Man() {
        System.out.println("Constructor of Man");
    }

    @PostConstruct
    public void init() {
        System.out.println("Init");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("Destroy");
    }
}
```

含义：

```text
实例化 Bean
    ↓
依赖注入
    ↓
@PostConstruct
    ↓
Bean 正常使用
    ↓
Bean 被 Spring 销毁
    ↓
@PreDestroy
```

Spring Framework 通过 `CommonAnnotationBeanPostProcessor` 识别：

- `jakarta.annotation.PostConstruct`
- `jakarta.annotation.PreDestroy`

官方文档：

- [Using `@PostConstruct` and `@PreDestroy`](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/postconstruct-and-predestroy-annotations.html)

---

### 方式二：`@Bean(initMethod / destroyMethod)`

在配置类中指定生命周期方法：

```java
@Configuration
public class MyConfig {

    @Bean(
        value = "Man1",
        initMethod = "init",
        destroyMethod = "destroy"
    )
    public Man generateMan() {
        return new Man();
    }
}
```

对应的类本身可以完全不使用生命周期注解：

```java
public class Man {

    public void init() {
        System.out.println("Init");
    }

    public void destroy() {
        System.out.println("Destroy");
    }
}
```

`@Bean(initMethod = "...", destroyMethod = "...")` 的意义是：

> 在 BeanDefinition 中告诉 Spring：这个 Bean 初始化时调用哪个方法，销毁时调用哪个方法。

它与 XML 配置中的：

```xml
<bean init-method="init" destroy-method="destroy" />
```

属于同类机制。

官方文档：

- [Using the `@Bean` Annotation — Receiving Lifecycle Callbacks](https://docs.spring.io/spring-framework/reference/core/beans/java/bean-annotation.html#beans-java-lifecycle-callbacks)

---

## 2. 两种方式的区别

| 对比项 | `@PostConstruct / @PreDestroy` | `@Bean(initMethod / destroyMethod)` |
|---|---|---|
| 配置位置 | Bean 类内部 | Spring 配置类 |
| 是否需要修改 Bean 源码 | 需要 | 不需要 |
| 初始化方式 | `@PostConstruct` | `initMethod` |
| 销毁方式 | `@PreDestroy` | `destroyMethod` |
| 是否属于 Spring Bean 生命周期 | 是 | 是 |
| 适合自己编写的类 | 很适合 | 可以 |
| 适合第三方类 | 不方便 | 很适合 |
| 与 Spring 专用接口耦合 | 否，使用 Jakarta 注解 | 配置代码使用 Spring `@Bean` |

Spring 官方在生命周期章节中也推荐优先考虑 `@PostConstruct` / `@PreDestroy`，或者使用自定义 `init-method` / `destroy-method`，以避免业务类实现 `InitializingBean`、`DisposableBean` 而直接耦合 Spring API。

官方文档：

- [Customizing the Nature of a Bean — Lifecycle Callbacks](https://docs.spring.io/spring-framework/reference/core/beans/factory-nature.html#beans-factory-lifecycle)

---

# 3. Spring 支持组合多种生命周期机制

这是最重要的部分。

Spring 官方章节：

- [Combining Lifecycle Mechanisms](https://docs.spring.io/spring-framework/reference/core/beans/factory-nature.html#beans-factory-lifecycle-combined-effects)

Spring 官方列出三种主要生命周期机制：

1. `@PostConstruct` / `@PreDestroy`
2. `InitializingBean` / `DisposableBean`
3. 自定义 `init()` / `destroy()` 方法，例如 `@Bean(initMethod = "...", destroyMethod = "...")`

这些机制可以同时配置在同一个 Bean 上。

---

# 4. 特殊情况：同一个 `init()` 同时被两种机制指定

例如：

```java
public class Man {

    @PostConstruct
    public void init() {
        System.out.println("init");
    }
}
```

同时配置：

```java
@Bean(initMethod = "init")
public Man man() {
    return new Man();
}
```

此时：

```text
@PostConstruct
      \
       \
        ---> init()
       /
@Bean(initMethod = "init")
```

看起来 `init()` 被注册了两次，但 **Spring 不会因此调用两遍**。

## Spring 官方定义

Spring 官方在 **Combining Lifecycle Mechanisms** 中明确说明：

> 如果为同一个 Bean 配置了多个生命周期机制，并且它们使用不同的方法名，则每个方法都会按照规定顺序执行；  
> 但是，如果多个生命周期机制配置的是同一个方法名，例如初始化方法 `init()`，那么这个方法只执行一次。

官方原文所在位置：

- [Spring Framework — Combining Lifecycle Mechanisms](https://docs.spring.io/spring-framework/reference/core/beans/factory-nature.html#beans-factory-lifecycle-combined-effects)

因此：

```java
@PostConstruct
public void init() {
    System.out.println("init");
}
```

加上：

```java
@Bean(initMethod = "init")
```

结果是：

```text
init
```

而不是：

```text
init
init
```

### 核心记忆

```text
多个生命周期机制
        │
        ├── 指向同一个方法名
        │       ↓
        │    只执行一次
        │
        └── 指向不同方法名
                ↓
             全部执行
                ↓
          按 Spring 规定顺序
```

---

# 5. 如果配置的是不同方法，则全部执行

例如：

```java
public class Man {

    @PostConstruct
    public void init1() {
        System.out.println("init1");
    }

    public void init2() {
        System.out.println("init2");
    }
}
```

配置：

```java
@Bean(initMethod = "init2")
public Man man() {
    return new Man();
}
```

因为：

```text
@PostConstruct → init1()
initMethod     → init2()
```

是两个不同的方法，所以都会执行：

```text
init1
init2
```

---

# 6. Spring 官方规定的初始化执行顺序

如果同一个 Bean 同时使用三种机制，并且对应的是不同方法，则初始化顺序为：

```text
① @PostConstruct
        ↓
② InitializingBean.afterPropertiesSet()
        ↓
③ 自定义 init 方法
   例如 @Bean(initMethod = "init3")
```

例如：

```java
public class Man implements InitializingBean {

    @PostConstruct
    public void init1() {
        System.out.println("1. PostConstruct");
    }

    @Override
    public void afterPropertiesSet() {
        System.out.println("2. afterPropertiesSet");
    }

    public void init3() {
        System.out.println("3. custom init");
    }
}
```

配置：

```java
@Bean(initMethod = "init3")
public Man man() {
    return new Man();
}
```

执行顺序：

```text
init1()
↓
afterPropertiesSet()
↓
init3()
```

官方文档：

- [Combining Lifecycle Mechanisms](https://docs.spring.io/spring-framework/reference/core/beans/factory-nature.html#beans-factory-lifecycle-combined-effects)

---

# 7. Spring 官方规定的销毁执行顺序

如果销毁阶段使用的是不同方法，则执行顺序为：

```text
① @PreDestroy
        ↓
② DisposableBean.destroy()
        ↓
③ 自定义 destroy 方法
   例如 @Bean(destroyMethod = "cleanup")
```

也就是：

```text
@PreDestroy
      ↓
DisposableBean.destroy()
      ↓
custom destroyMethod
```

官方文档：

- [Combining Lifecycle Mechanisms](https://docs.spring.io/spring-framework/reference/core/beans/factory-nature.html#beans-factory-lifecycle-combined-effects)

同样，如果 `@PreDestroy` 标注的方法与 `destroyMethod` 指向的是同一个方法名，则不会因为重复配置而调用两次。

---

# 8. 结合当前代码分析

当前代码实际上定义了两个 Spring Bean。

## Bean 1：`Man`

```java
@Component("Man")
@Scope("singleton")
public class Man {
    ...
}
```

Bean 名：

```text
Man
```

作用域：

```text
singleton
```

生命周期大致为：

```text
创建 singleton
      ↓
Constructor
      ↓
@PostConstruct init()
      ↓
正常使用
      ↓
ApplicationContext 正常关闭
      ↓
@PreDestroy destroy()
```

---

## Bean 2：`Man1`

配置：

```java
@Bean(
    value = "Man1",
    initMethod = "init",
    destroyMethod = "destroy"
)
@Scope("prototype")
public Man generateMan() {
    return new Man();
}
```

Bean 名：

```text
Man1
```

作用域：

```text
prototype
```

虽然 `Man` 和 `Man1` 的 Java 类型都是：

```java
Man
```

但在 Spring 容器中，它们属于两个独立的 BeanDefinition。

---

# 9. `Man1` 的 `init()` 为什么只执行一次？

假设 `Man` 类本身有：

```java
@PostConstruct
public void init() {
    System.out.println("Init");
}
```

同时 `Man1` 又配置：

```java
@Bean(initMethod = "init")
```

那么对某一个 `Man1` 实例而言：

```text
@PostConstruct
      \
       \
        ---> init()
       /
initMethod = "init"
```

因为两个机制最终指向的是 **同一个方法名 `init`**，根据 Spring 官方的组合生命周期规则：

```text
这个实例的 init() 只执行一次
```

这正是：

**Combining Lifecycle Mechanisms**

章节中特别定义的情况。

官方链接：

- https://docs.spring.io/spring-framework/reference/core/beans/factory-nature.html#beans-factory-lifecycle-combined-effects

---

# 10. `prototype` Bean 的特殊生命周期

`Man1` 还有：

```java
@Scope("prototype")
```

Spring 官方对 prototype scope 的定义是：

> 每次请求该 Bean 时，都会创建一个新的 Bean 实例。

例如：

```java
Man m1 = context.getBean("Man1", Man.class);
Man m2 = context.getBean("Man1", Man.class);

System.out.println(m1 == m2);
```

结果：

```text
false
```

因此：

```text
第一次 getBean("Man1")
        ↓
new Man()
        ↓
初始化
        ↓
init() 一次


第二次 getBean("Man1")
        ↓
又 new Man()
        ↓
初始化
        ↓
init() 一次
```

最终控制台可能出现两次 `Init`，但原因不是：

```text
同一个 Bean 的 init() 被调用两次
```

而是：

```text
创建了两个不同的 prototype Bean
每个 Bean 都各自初始化一次
```

官方文档：

- [Bean Scopes — The Prototype Scope](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html#beans-factory-scopes-prototype)

---

# 11. Prototype Bean 的销毁回调不会自动执行

这是另一个非常重要的考点。

Spring 官方明确说明：

> Spring 不管理 prototype Bean 的完整生命周期。容器负责实例化、配置和组装 prototype Bean，然后将其交给客户端；初始化生命周期回调会执行，但配置的销毁生命周期回调不会由 Spring 自动调用。

因此，对：

```java
@Bean(
    value = "Man1",
    initMethod = "init",
    destroyMethod = "destroy"
)
@Scope("prototype")
```

而言：

### 会执行

```java
@PostConstruct
```

以及：

```java
initMethod = "init"
```

### Spring 不会自动执行

```java
@PreDestroy
```

以及：

```java
destroyMethod = "destroy"
```

其生命周期可概括为：

```text
getBean("Man1")
      ↓
new Man()
      ↓
依赖注入
      ↓
初始化生命周期回调
      ↓
若 @PostConstruct 与 initMethod 指向同一个 init()
      ↓
init() 只执行一次
      ↓
Bean 交给客户端
      ↓
Spring 不再管理该 prototype 实例的完整生命周期
      ↓
ApplicationContext 关闭
      ↓
不会自动调用 prototype Bean 的销毁回调
```

官方文档：

- [Bean Scopes — The Prototype Scope](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html#beans-factory-scopes-prototype)

---

# 12. Singleton 与 Prototype 对比

| 特性 | Singleton | Prototype |
|---|---|---|
| 创建次数 | 默认一个实例 | 每次请求创建新实例 |
| 初始化回调 | Spring 调用 | Spring 调用 |
| `@PostConstruct` | 调用 | 调用 |
| `initMethod` | 调用 | 调用 |
| Spring 是否完整管理生命周期 | 是 | 否 |
| 销毁回调 | 正常关闭时调用 | **不自动调用** |
| `@PreDestroy` | 调用 | **不自动调用** |
| `destroyMethod` | 调用 | **不自动调用** |

---

# 13. 最终记忆版

## 两种配置方式

```text
@PostConstruct / @PreDestroy
        ↓
生命周期定义在 Bean 类内部
```

```text
@Bean(
    initMethod = "...",
    destroyMethod = "..."
)
        ↓
生命周期定义在外部配置中
```

---

## 多机制组合规则

```text
不同生命周期方法
        ↓
全部执行
        ↓
按照 Spring 规定的顺序执行
```

初始化顺序：

```text
@PostConstruct
      ↓
InitializingBean.afterPropertiesSet()
      ↓
自定义 initMethod
```

销毁顺序：

```text
@PreDestroy
      ↓
DisposableBean.destroy()
      ↓
自定义 destroyMethod
```

---

## 最重要的特殊规则

```java
@PostConstruct
public void init() {}
```

同时：

```java
@Bean(initMethod = "init")
```

两个生命周期机制都指向：

```text
init()
```

因此：

```text
init() 只执行一次
```

不是：

```text
init()
init()
```

Spring 官方对此有明确规定：

- [Combining Lifecycle Mechanisms](https://docs.spring.io/spring-framework/reference/core/beans/factory-nature.html#beans-factory-lifecycle-combined-effects)

---

# 14. 一句话考试版

> Spring Bean 可以同时使用 `@PostConstruct/@PreDestroy`、`InitializingBean/DisposableBean` 和自定义 `initMethod/destroyMethod`。如果不同生命周期机制对应不同的方法，则按照 **注解 → Spring 生命周期接口 → 自定义方法** 的顺序分别执行；如果多个生命周期机制配置的是 **同一个方法名**，该方法只执行一次。对于 `prototype` Bean，初始化回调正常执行，但 Spring 不自动执行其销毁回调。

---

# 15. Spring 官方文档链接汇总

1. **Bean 生命周期总览 / Lifecycle Callbacks**  
   https://docs.spring.io/spring-framework/reference/core/beans/factory-nature.html#beans-factory-lifecycle

2. **组合多个生命周期机制 / Combining Lifecycle Mechanisms**  
   https://docs.spring.io/spring-framework/reference/core/beans/factory-nature.html#beans-factory-lifecycle-combined-effects

3. **`@PostConstruct` 与 `@PreDestroy`**  
   https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/postconstruct-and-predestroy-annotations.html

4. **`@Bean(initMethod, destroyMethod)` 生命周期回调**  
   https://docs.spring.io/spring-framework/reference/core/beans/java/bean-annotation.html#beans-java-lifecycle-callbacks

5. **Bean Scopes / Prototype Scope**  
   https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html#beans-factory-scopes-prototype

---

## 备注：方法名拼写

代码中如果写的是：

```java
public void destory() {}
```

以及：

```java
destroyMethod = "destory"
```

只要字符串与实际方法名一致，Spring 仍然可以找到该方法。

但正确的英文拼写应为：

```java
destroy
```

建议统一改成：

```java
@PreDestroy
public void destroy() {
    System.out.println("Destroy");
}
```

以及：

```java
@Bean(destroyMethod = "destroy")
```

这样能避免后续配置错误。
