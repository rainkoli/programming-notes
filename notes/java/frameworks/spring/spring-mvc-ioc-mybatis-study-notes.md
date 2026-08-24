# Spring MVC、Spring IoC/DI 与 MyBatis 学习笔记

> 主题：`@Service`、Bean 注入、Mapper 代理、类型别名、`@ResponseBody`、`@RestController`、请求参数绑定、`Integer` 与 `int`、`mybatis.type-aliases-package`、常见 404 与启动异常

---

## 1. 先建立全局视角：一次请求经过了什么

一个典型的 Spring Boot + Spring MVC + MyBatis 请求可以概括为：

```text
HTTP Client
    │
    ▼
DispatcherServlet
    │
    ├─ HandlerMapping：找到 Controller 方法
    ├─ 参数解析器：读取 Path、Query、Header、Body 等数据
    ▼
Controller
    │
    ▼
Service 接口
    │
    ▼
@Service 实现类（Spring Bean）
    │
    ▼
Mapper 接口的 MyBatis 代理（Spring Bean）
    │
    ▼
SqlSession / Mapper XML / JDBC
    │
    ▼
Database
    │
    ▼
Java 对象
    │
    ▼
Controller 返回值处理器
    ├─ @ResponseBody → HttpMessageConverter → 通常输出 JSON
    └─ 普通 @Controller → Model + ViewResolver → 渲染视图
```

前面的数据库查询成功，并不代表最后的 HTTP 响应一定成功。请求可能在参数绑定、Bean 创建、SQL 映射、返回值序列化或视图解析等任一阶段失败。

官方文档：

- [Spring MVC](https://docs.spring.io/spring-framework/reference/web/webmvc.html)
- [DispatcherServlet](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-servlet.html)
- [Controller 方法参数](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/arguments.html)
- [Controller 方法返回值](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/return-types.html)

---

## 2. Spring IoC、Bean 与 DI

### 2.1 IoC 是什么

IoC（Inversion of Control，控制反转）的核心思想是：对象不再亲自创建和查找自己的依赖，而是由 Spring 容器创建对象、管理对象，并把依赖提供给对象。

没有 IoC 时：

```java
public class UserController {
    private final UserService userService = new UserServiceImpl();
}
```

Controller 直接依赖具体实现，也负责创建它。

使用 Spring DI 后：

```java
public class UserController {
    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }
}
```

`UserController` 只声明“我需要一个 `UserService`”；具体对象由 Spring 容器提供。

### 2.2 Bean 是什么

Bean 是由 Spring IoC 容器创建、装配和管理的对象。常见的 Bean 注册方式包括：

- 在具体类上使用 `@Component`、`@Service`、`@Repository`、`@Controller` 等组件注解；
- 在配置类中使用 `@Bean`；
- 由其他框架通过工厂对象注册，例如 MyBatis 的 Mapper 代理。

### 2.3 DI 是什么

DI（Dependency Injection，依赖注入）是 IoC 的具体实现方式之一。依赖可以通过构造器、Setter 或字段注入。

官方文档把构造器注入和 Setter 注入列为两种主要方式，并建议必需依赖优先使用构造器。构造器注入能够：

- 让必需依赖在对象创建时就准备完整；
- 配合 `final` 字段表达不可变依赖；
- 让普通单元测试可以直接 `new` 对象并传入替身；
- 更早暴露依赖过多、循环依赖等设计问题。

官方文档：

- [Spring IoC 容器](https://docs.spring.io/spring-framework/reference/core/beans.html)
- [Dependency Injection](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html)
- [使用 `@Autowired`](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/autowired.html)

---

## 3. `@Service` 应该写在接口还是实现类上

### 3.1 推荐结构

接口只定义业务契约：

```java
public interface Day20UserService {
    List<User> findAll();

    User findById(Integer id);
}
```

实现类承担业务逻辑，并注册为 Spring Bean：

```java
@Service
public class Day20UserServiceImpl implements Day20UserService {
    private final Day20UserMapper userMapper;

    public Day20UserServiceImpl(Day20UserMapper userMapper) {
        this.userMapper = userMapper;
    }

    @Override
    public List<User> findAll() {
        return userMapper.findAll();
    }

    @Override
    public User findById(Integer id) {
        if (id == null) {
            throw new IllegalArgumentException("id must not be null");
        }
        return userMapper.findById(id);
    }
}
```

Controller 依赖接口：

```java
@RestController
@RequestMapping("/users")
public class Day20UserController {
    private final Day20UserService userService;

    public Day20UserController(Day20UserService userService) {
        this.userService = userService;
    }
}
```

### 3.2 为什么不应把普通 `@Service` 只写在接口上

`@Service` 是 `@Component` 的特化，用于让具体实现类通过组件扫描被发现。普通 Java 接口不能直接实例化；Spring 的普通组件扫描也不会像 MyBatis 那样凭空为 Service 接口生成业务实现。

因此：

```java
@Service
public interface Day20UserService {
}
```

是误导性的。它既不能替代实现类，也不会让实现类自动继承这个注解。应删除接口上的 `@Service`，把它放在实现类上。

> 准确性说明：普通接口上的 `@Service` 本身通常不会创建一个可实例化的第二个 Bean，也不会自动生成代理。如果出现“同一接口有多个候选 Bean”，应继续查找多个实现类、`@Bean` 方法、测试替身或框架代理等真实注册来源。

Spring 的 `@Service` Javadoc 明确说明，它是 `@Component` 的特化，可让实现类被 classpath scanning 自动发现：

- [`@Service` 官方 Javadoc](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Service.html)
- [Classpath Scanning and Managed Components](https://docs.spring.io/spring-framework/reference/core/beans/classpath-scanning.html)

### 3.3 为什么注入接口却能得到实现类

这来自 Java 的可赋值关系：

```java
Day20UserService service = new Day20UserServiceImpl(mapper);
```

`Day20UserServiceImpl implements Day20UserService`，所以实现类对象也是一个 `Day20UserService`。Spring 按类型解析依赖时，会寻找能够赋值给目标类型的 Bean。

容器里实际只有实现类对象：

```text
需要的类型：Day20UserService
                     ▲
                     │ implements / is assignable to
实际 Bean：Day20UserServiceImpl
```

接口不是“指向实现类的另一个 Bean”；它只是 Controller 用来声明依赖的类型。

### 3.4 什么时候才会出现多个候选 Bean

例如有两个真实实现：

```java
@Service
public class DatabaseUserService implements Day20UserService {
}

@Service
public class CachedUserService implements Day20UserService {
}
```

此时下面的注入点确实有两个候选者：

```java
private final Day20UserService userService;
```

Spring 无法唯一选择，通常会抛出 `NoUniqueBeanDefinitionException`。可以采用以下方法：

1. 如果本来只应有一个实现，删除或取消注册多余实现；
2. 使用 `@Primary` 指定默认实现；
3. 使用 `@Qualifier` 在注入点明确指定 Bean；
4. 如果业务确实需要全部实现，注入 `List<Day20UserService>`。

示例：

```java
@Service
@Primary
public class DatabaseUserService implements Day20UserService {
}
```

或：

```java
public Day20UserController(
        @Qualifier("cachedUserService") Day20UserService userService) {
    this.userService = userService;
}
```

官方文档：

- [`@Primary` 与多个候选 Bean](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/autowired-primary.html)
- [`@Qualifier`](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/autowired-qualifiers.html)

### 3.5 字段注入与构造器注入

字段注入可以工作：

```java
@Autowired
private Day20UserService userService;
```

但必需依赖通常更适合构造器注入：

```java
private final Day20UserService userService;

public Day20UserController(Day20UserService userService) {
    this.userService = userService;
}
```

如果一个 Spring Bean 只有一个构造器，一般可以省略构造器上的 `@Autowired`。

---

## 4. 普通 Service 接口与 MyBatis Mapper 接口为什么不同

### 4.1 Mapper 接口没有手写实现类也能运行

```java
@Mapper
public interface Day20UserMapper {
    List<User> findAll();

    User findById(@Param("id") Integer id);
}
```

MyBatis-Spring 会通过 `MapperFactoryBean` 为 Mapper 接口创建动态代理，并把代理注册成 Spring Bean：

```text
Day20UserMapper 接口
        │
        ▼
MapperFactoryBean
        │
        ▼
MyBatis 动态代理对象（Spring Bean）
        │
        ▼
根据方法定位 Mapper XML / 注解 SQL
```

因此，Mapper 接口能注入，不是因为“Spring 可以直接实例化接口”，而是因为 MyBatis 明确提供了代理实现。

### 4.2 `@Mapper` 与 `@MapperScan`

逐个标记：

```java
@Mapper
public interface Day20UserMapper {
}
```

统一扫描：

```java
@SpringBootApplication
@MapperScan("com.hs.homework.August.day20ssm.mapper")
public class Day20StartApplication {
}
```

二者选择一种清晰的策略即可。`@MapperScan` 适合 Mapper 较多的项目。

官方文档：

- [MyBatis-Spring：注册与扫描 Mapper](https://mybatis.org/spring/mappers.html)
- [MyBatis Spring Boot Starter](https://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/)

---

## 5. Mapper 接口与 Mapper XML 如何对应

Mapper 接口：

```java
package com.hs.homework.August.day20ssm.mapper;

@Mapper
public interface Day20UserMapper {
    List<User> findAll();

    User findById(@Param("id") Integer id);
}
```

Mapper XML：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.hs.homework.August.day20ssm.mapper.Day20UserMapper">

    <select id="findAll" resultType="user">
        SELECT id, username, password, sex
        FROM demo_20260820_ssm_user
    </select>

    <select id="findById" resultType="user">
        SELECT id, username, password, sex
        FROM demo_20260820_ssm_user
        WHERE id = #{id}
    </select>

</mapper>
```

关键对应关系：

| Mapper 接口 | Mapper XML |
|---|---|
| 接口全限定名 | `<mapper namespace="...">` |
| 方法名 `findById` | `<select id="findById">` |
| 参数名 `id` | `#{id}` |
| 返回类型 `User` | `resultType="user"` 或完整类名 |

注意：

- `namespace` 通常必须是 Mapper 接口的全限定名；
- statement 的 `id` 应与方法名对应；
- `@Param("id")` 明确了 XML 中的参数名，尤其适合多参数方法；
- `parameterType` 通常可以由 MyBatis 推断，并非每个 statement 都必须写；
- `resultType` 可以使用类型别名，也可以使用全限定类名。

参数占位还要区分：

```xml
WHERE id = #{id}
```

`#{id}` 会生成预编译参数，通常应优先使用。`${id}` 是直接文本替换，不能接收不可信输入，否则可能造成 SQL 注入；它只适合经过严格白名单控制、确实不能参数化的 SQL 片段。

官方文档：

- [MyBatis XML 映射文件](https://mybatis.org/mybatis-3/sqlmap-xml.html)
- [MyBatis Java API 与 Mapper](https://mybatis.org/mybatis-3/java-api.html)

---

## 6. MyBatis Type Alias（类型别名）

### 6.1 类型别名解决什么问题

没有别名时：

```xml
<select id="findAll"
        resultType="com.hs.homework.August.day20ssm.entity.User">
    SELECT * FROM demo_20260820_ssm_user
</select>
```

有别名时：

```xml
<select id="findAll" resultType="user">
    SELECT * FROM demo_20260820_ssm_user
</select>
```

类型别名只是 Java 类型的短名称，主要用于减少 MyBatis XML 中重复的全限定类名。它不会：

- 把该类型变成 Spring Bean；
- 扫描 Mapper XML；
- 代替 `@Mapper` 或 `@MapperScan`；
- 改变 Java 类本身的名称。

MyBatis 官方文档将类型别名定义为 Java 类型的较短名称，并说明它主要与 XML 配置有关：

- [MyBatis 类型别名官方文档](https://mybatis.org/mybatis-3/configuration.html#typeAliases)
- [MyBatis 类型别名中文官方文档](https://mybatis.org/mybatis-3/zh_CN/configuration.html#typeAliases)

### 6.2 三种常见注册方式

逐个注册：

```xml
<typeAliases>
    <typeAlias alias="user"
               type="com.hs.homework.August.day20ssm.entity.User"/>
</typeAliases>
```

在 MyBatis 原生配置中扫描包：

```xml
<typeAliases>
    <package name="com.hs.homework.August.day20ssm.entity"/>
</typeAliases>
```

在 Spring Boot 中配置扫描包：

```yaml
mybatis:
  type-aliases-package: com.hs.homework.August.day20ssm.entity
```

### 6.3 默认别名与 `@Alias`

扫描包后，如果类没有 `@Alias`，MyBatis 使用非限定类名作为基础注册别名；官方说明以首字母小写形式展示，例如 `Author` 对应 `author`。类型别名的解析不区分大小写，因此 XML 中最好统一使用一种风格，例如全小写：

```xml
resultType="user"
```

也可以显式指定：

```java
@Alias("day20User")
public class User {
}
```

XML：

```xml
resultType="day20User"
```

### 6.4 一个容易误解的内置别名

MyBatis 官方内置别名中：

| MyBatis XML 别名 | Java 类型 |
|---|---|
| `int` / `integer` | `java.lang.Integer` |
| `_int` / `_integer` | 基本类型 `int` |

因此，XML 中的 `parameterType="int"` 实际指包装类型 `Integer`；带下划线的 `_int` 才代表 Java 基本类型。实际 Mapper 参数通常可以由 MyBatis 推断，所以没有必要只为区分这两者而强制写 `parameterType`。

官方文档：

- [`@Alias` Javadoc](https://mybatis.org/mybatis-3/apidocs/org/apache/ibatis/type/Alias.html)
- [`TypeAliasRegistry` Javadoc](https://mybatis.org/mybatis-3/apidocs/org/apache/ibatis/type/TypeAliasRegistry.html)

---

## 7. `mybatis.type-aliases-package` 的准确作用

### 7.1 官方配置项怎么说

MyBatis Spring Boot Starter 对该属性的简短描述是：

> Packages to search for type aliases.

也就是：指定 MyBatis 到哪些包中搜索要注册为类型别名的 Java 类型。

属性文件中的完整键名使用点号：

```properties
mybatis.type-aliases-package=com.hs.homework.August.day20ssm.entity
```

YAML 则用缩进表达 `mybatis` 这一层：

```yaml
mybatis:
  type-aliases-package: com.hs.homework.August.day20ssm.entity
```

所以 `mybatis:type-aliases-package` 不是通常使用的完整属性写法；冒号是 YAML 中键和值或父子结构的一部分。

官方配置页：

- [MyBatis Spring Boot 配置属性](https://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/)
- [MyBatis Spring Boot 中文配置页](https://mybatis.org/spring-boot-starter/zh/mybatis-spring-boot-autoconfigure/index.html)

### 7.2 它与其他扫描配置的区别

| 配置或注解 | 查找对象 | 主要目的 |
|---|---|---|
| `mybatis.type-aliases-package` | Java 类型，通常是 entity/DTO | 注册 XML 可用的短类型名 |
| `mybatis.mapper-locations` | Mapper XML 资源 | 告诉 `SqlSessionFactory` SQL 映射文件在哪里 |
| `@Mapper` | 单个 Mapper 接口 | 让 MyBatis 为接口创建代理 Bean |
| `@MapperScan` | 一组 Mapper 接口 | 批量创建 Mapper 代理 Bean |
| `@Alias` | 一个 Java 类型 | 自定义该类型的别名 |
| Spring `@ComponentScan` | Spring 组件类 | 注册普通 Spring Bean |

一个较清晰的学习项目配置：

```yaml
mybatis:
  mapper-locations: classpath*:mapper/**/*.xml
  type-aliases-package: com.hs.homework.August.day20ssm.entity
  configuration:
    map-underscore-to-camel-case: true
```

不要把 `type-aliases-package` 理解成“实体类必须配置这个包才能被 MyBatis 使用”。即使不配置别名扫描，也可以在 XML 中使用完整类名。

### 7.3 为什么扫描范围过大会发生 `Car` 冲突

假设学习仓库中存在：

```text
com.hs.homework.July.day23.question4.Car
com.hs.homework.August.day20ssm.entity.Car
```

如果配置：

```yaml
mybatis:
  type-aliases-package: com.hs.homework
```

两个类都会尝试注册为同一个不区分大小写的短别名 `car`。同一个别名不能映射到两个不同的 Java 类型，于是启动时可能出现：

```text
The alias 'Car' is already mapped to the value '...Car'
```

这通常表示“又有一个不同的类试图占用同一个别名”。重复扫描同一个类并不等同于两个不同类型之间的别名冲突。

MyBatis 的官方实现会把别名键规范为小写，并在同一键已映射到另一个类型时抛出 `TypeException`：

- [`TypeAliasRegistry` 官方源码](https://mybatis.org/mybatis-3/xref/org/apache/ibatis/type/TypeAliasRegistry.html)

推荐解决顺序：

1. 首先缩小扫描范围，只扫描当前应用的 entity/DTO 包；
2. 搜索同名类和重复的 `@Alias` 值；
3. 必要时给不同类型设置明确且唯一的 `@Alias`；
4. 大型学习仓库可拆成独立 Maven/Gradle 模块，隔离 classpath；
5. 也可以不使用包扫描，在 XML 中写全限定类名。

进阶配置还可以用 `mybatis.type-aliases-super-type` 按父类型过滤扫描结果；Starter 官方说明，未配置该过滤条件时，会处理 `type-aliases-package` 搜索到的所有候选类型。多数简单项目直接把包范围限制准确会更清楚。多个别名包也可以按官方支持的逗号、分号或空白分隔符书写。

推荐：

```yaml
mybatis:
  type-aliases-package: com.hs.homework.August.day20ssm.entity
```

不推荐在当前结构中扫描整个练习根包：

```yaml
mybatis:
  type-aliases-package: com.hs.homework
```

---

## 8. 如何阅读 `SqlSessionFactory` 启动异常

### 8.1 外层异常不一定是根因

日志可能从 Controller 开始报错：

```text
Error creating bean with name 'day20UserController'
  caused by: day20UserServiceImpl 创建失败
    caused by: day20UserMapper 创建失败
      caused by: sqlSessionTemplate 创建失败
        caused by: sqlSessionFactory 创建失败
          caused by: TypeException: alias 'Car' is already mapped
```

依赖关系使错误逐层向外传播：

```text
Controller
    └─ Service
        └─ Mapper proxy
            └─ SqlSessionTemplate
                └─ SqlSessionFactory
                    └─ TypeAliasRegistry
```

因此最外层的 `UnsatisfiedDependencyException` 只说明上层 Bean 无法完成创建。真正需要修复的通常是异常链底部最具体的 `Caused by`，本例为类型别名冲突。

### 8.2 为什么别名问题会让 Controller 都创建失败

`SqlSessionFactory` 在应用启动阶段读取 MyBatis 配置、注册类型别名并解析 Mapper XML。它创建失败后：

- `SqlSessionTemplate` 无法使用；
- Mapper 代理无法完整创建；
- Service 缺少 Mapper 依赖；
- Controller 又缺少 Service 依赖；
- 最终整个 Spring ApplicationContext 启动失败。

这不代表 Controller、Service、Mapper 每一层都各有一个独立错误，而是一个底层错误沿依赖图传播。

### 8.3 排查顺序

1. 从日志末尾向上查找最具体的 `Caused by`；
2. 判断它属于 Spring Bean、MyBatis 配置、Mapper XML、数据库连接还是 SQL；
3. 如果看到 alias 冲突，检查 `type-aliases-package` 和 `@Alias`；
4. 如果看到 `Invalid bound statement`，检查 namespace、statement id 和 XML 是否被加载；
5. 如果看到参数名不存在，检查 `@Param` 与 `#{...}`；
6. 修复最底层问题后重新启动，避免同时猜测上层所有组件。

---

## 9. `@Controller`、`@ResponseBody` 与 `@RestController`

### 9.1 三种写法

返回 HTML 视图的 MVC Controller：

```java
@Controller
public class PageController {
    @GetMapping("/home")
    public String home() {
        return "home";
    }
}
```

在单个方法上返回响应体：

```java
@Controller
public class UserController {
    @GetMapping("/users/{id}")
    @ResponseBody
    public User findById(@PathVariable("id") Integer id) {
        return userService.findById(id);
    }
}
```

整个类都作为 REST API：

```java
@RestController
@RequestMapping("/users")
public class UserController {
    @GetMapping("/{id}")
    public User findById(@PathVariable("id") Integer id) {
        return userService.findById(id);
    }
}
```

`@RestController` 是组合注解，其效果相当于类级别的：

```java
@Controller
@ResponseBody
```

官方文档：

- [`@ResponseBody`](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/responsebody.html)
- [`@RestController` Javadoc](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/RestController.html)

### 9.2 `@ResponseBody` 的深层处理过程

```text
Controller 返回 User/Result 对象
        │
        ▼
RequestResponseBodyMethodProcessor
        │
        ▼
根据返回类型、produces、Accept 等进行内容协商
        │
        ▼
选择合适的 HttpMessageConverter
        │
        ├─ 常见 JSON：MappingJackson2HttpMessageConverter + Jackson
        ├─ String：StringHttpMessageConverter
        └─ byte[]：ByteArrayHttpMessageConverter
        │
        ▼
写入 HTTP Response Body
```

所以，`@ResponseBody` 的准确含义是“让返回值经过消息转换器写入响应体”。在典型 Spring Boot Web 项目中，类路径上有 Jackson 时，对象通常被序列化为 JSON；但 `@ResponseBody` 本身并不等于“只允许 JSON”。

### 9.3 没有 `@ResponseBody` 时，返回值究竟是什么

要根据返回类型区分：

| Controller 返回值 | 普通 `@Controller` 中的典型含义 |
|---|---|
| `String` | 逻辑视图名，例如 `"home"` |
| `ModelAndView` | 明确的 Model 和 View |
| `void` | 可能使用默认视图名，或表示响应已由方法处理 |
| 普通复杂对象（如 `Result`） | 通常作为 model attribute；随后使用显式或默认视图名 |
| `@ResponseBody` 标记的任意受支持值 | 经 `HttpMessageConverter` 写入响应体 |

一个重要修正是：

```java
@Controller
public class UserController {
    @GetMapping("/users")
    public Result findAll() {
        return Result.ok(userService.findAll());
    }
}
```

这里的 `Result` 对象通常不是简单地用 `toString()` 当作视图名。Spring MVC 往往把这个复杂对象加入 Model，然后在没有显式视图名时，根据请求路径推导默认视图名，再进入 ViewResolver 流程。

官方文档：

- [Controller 返回值类型](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/return-types.html)
- [View Resolution](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-servlet/viewresolver.html)
- [HTTP Message Conversion](https://docs.spring.io/spring-framework/reference/web/webmvc/message-converters.html)

---

## 10. 为什么没加 `@ResponseBody` 可能出现 404

### 10.1 请求前 404 与返回阶段 404 不是同一件事

第一种：没有找到 Controller 路由。

```text
请求到达 DispatcherServlet
    ↓
HandlerMapping 没有匹配方法
    ↓
404
```

这时 Controller 没执行，Service 和 SQL 通常也不会执行。

第二种：Controller 已执行，但错误进入视图解析。

```text
请求匹配成功
    ↓
Controller → Service → Mapper → SQL 全部成功
    ↓
返回值没有按 Response Body 处理
    ↓
Spring 尝试确定逻辑视图名
    ↓
ViewResolver 尝试定位模板、JSP 或内部资源
    ↓
资源不存在，可能表现为 404、模板异常或 500
```

如果日志已显示 SQL 执行成功，就能证明最初的 URL 映射已经匹配；此时应重点检查 Controller 返回值处理和视图解析，而不是再猜“Controller 有没有被扫描”。

### 10.2 为什么说“可能”而不是“一定”是 404

缺少 `@ResponseBody` 不直接规定 HTTP 状态必须是 404。最终现象取决于：

- 返回值类型；
- 是否配置了 Thymeleaf、JSP 或其他视图技术；
- 使用了哪一个 `ViewResolver`；
- 推导出的默认视图名；
- 缺少模板时对应视图技术抛出的异常类型；
- 是否发生内部 forward。

因此，缺少 `@ResponseBody` 可能导致 404，也可能导致 500、循环视图路径错误或模板找不到异常。根因是“返回值进入了错误的处理分支”，不是 `@ResponseBody` 注解直接产生 404。

### 10.3 REST API 的修复

如果整个 Controller 都用于返回 API 数据，使用：

```java
@RestController
@RequestMapping("/users")
public class Day20UserController {
}
```

如果只有某个方法返回响应体，则保留 `@Controller`，只在该方法上加：

```java
@ResponseBody
```

### 10.4 快速判断表

| 现象 | 更可能的阶段 |
|---|---|
| Controller 日志和 SQL 都没有出现，响应 404 | 请求映射没有匹配 |
| SQL 成功执行，之后 404/模板异常 | 返回值或视图解析阶段 |
| 返回对象时出现 JSON 序列化异常 | `HttpMessageConverter` / Jackson 阶段 |
| 响应 406 | 没有客户端可接受的表示形式 |
| 响应 415 | 请求 `Content-Type` 不受支持 |

---

## 11. `{id}` 与 `@PathVariable`

### 11.1 `{id}` 是 URI 模板变量

```java
@RestController
@RequestMapping("/users")
public class UserController {
    @GetMapping("/{id}")
    public User findById(@PathVariable("id") Integer id) {
        return userService.findById(id);
    }
}
```

类级别与方法级别路径组合后为：

```text
/users/{id}
```

请求：

```http
GET /users/5
```

匹配结果：

```text
{id} → 字符串 "5" → 类型转换 → Integer 5
```

`{id}` 不是 Java 变量，也不是数据库字段。它是路由模板中的占位符；`@PathVariable("id")` 才负责把捕获到的值绑定到方法参数。

### 11.2 可以省略注解中的名称吗

可以写：

```java
@GetMapping("/{id}")
public User findById(@PathVariable Integer id) {
}
```

前提是编译结果保留了参数名，并且模板变量名与 Java 参数名相同。为减少构建配置差异，学习阶段显式写出名称更直观：

```java
@PathVariable("id") Integer id
```

### 11.3 路径中没有 `id` 时会怎样

请求 `/users` 或 `/users/` 通常不会给下面的方法传入 `null`：

```java
@GetMapping("/{id}")
```

更常见的结果是路由根本不匹配，从而进入 404 处理。也就是说：

```text
没有路径段
    ≠
成功匹配后把 id 绑定为 null
```

如果确实要同时支持“有 id”和“无 id”，最好定义两个语义明确的方法：

```java
@GetMapping
public List<User> findAll() {
    return userService.findAll();
}

@GetMapping("/{id}")
public User findById(@PathVariable("id") Integer id) {
    return userService.findById(id);
}
```

虽然 `@PathVariable` 有 `required` 属性，但只有请求映射本身也允许无变量路径时，`required = false` 才有实际意义；它不会让 `/{id}` 自动匹配一个缺失的路径段。

官方文档：

- [Request Mapping 与 URI Patterns](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-requestmapping.html)
- [`@PathVariable` Javadoc](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/PathVariable.html)

---

## 12. `@PathVariable`、`@RequestParam`、`@RequestBody` 及相关注解

> 正确名称是 `@RequestParam`，不是 `@Requestparameter` 或 `@RequestParameter`。

### 12.1 从 HTTP 请求的哪个位置取值

假设请求为：

```http
POST /users/42/orders?page=2
Content-Type: application/json
Authorization: Bearer abc
Cookie: theme=dark

{"productId": 7, "quantity": 2}
```

| 数据位置 | 示例 | 常用注解 |
|---|---|---|
| Path | `/users/42` 中的 `42` | `@PathVariable` |
| Query String | `?page=2` | `@RequestParam` |
| Form 参数 | `application/x-www-form-urlencoded` | `@RequestParam` / `@ModelAttribute` |
| HTTP Header | `Authorization: ...` | `@RequestHeader` |
| Cookie | `theme=dark` | `@CookieValue` |
| JSON/XML Body | `{...}` | `@RequestBody` |
| Multipart 中的 part | 文件或某个 JSON part | `@RequestPart` |
| Request Attribute | 过滤器/拦截器放入请求的对象 | `@RequestAttribute` |
| Session Attribute | 已存在的 Session 数据 | `@SessionAttribute` |
| URI path segment 参数 | `/cars;color=red` | `@MatrixVariable` |

### 12.2 `@PathVariable`：定位路径中的资源

```java
@GetMapping("/users/{userId}/orders/{orderId}")
public Order findOrder(
        @PathVariable("userId") Long userId,
        @PathVariable("orderId") Long orderId) {
    // ...
}
```

常见语义是定位资源：

```text
GET /users/42
GET /users/42/orders/9
```

### 12.3 `@RequestParam`：查询、筛选、分页或表单参数

```java
@GetMapping("/users")
public List<User> search(
        @RequestParam(required = false) String keyword,
        @RequestParam(defaultValue = "1") int page,
        @RequestParam(defaultValue = "20") int size) {
    // ...
}
```

请求：

```http
GET /users?keyword=alice&page=2&size=10
```

`@RequestParam` 默认 `required = true`。可选参数常见写法：

```java
@RequestParam(required = false) Integer page
```

或：

```java
@RequestParam Optional<Integer> page
```

或为基本类型提供默认值：

```java
@RequestParam(defaultValue = "1") int page
```

官方文档：

- [`@RequestParam`](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/requestparam.html)

### 12.4 `@RequestBody`：反序列化请求体

```java
@PostMapping(consumes = "application/json")
public User create(@Valid @RequestBody CreateUserRequest request) {
    // ...
}
```

请求：

```http
POST /users
Content-Type: application/json

{
  "username": "alice",
  "password": "secret"
}
```

Spring 使用 `HttpMessageConverter` 把请求体反序列化为 Java 对象。JSON 场景通常由 Jackson 完成。表单数据更适合 `@RequestParam` 或 `@ModelAttribute`，而不是一律使用 `@RequestBody`。

官方文档：

- [`@RequestBody`](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/requestbody.html)

### 12.5 `@RequestHeader`

```java
@GetMapping("/profile")
public Profile profile(
        @RequestHeader("Authorization") String authorization) {
    // ...
}
```

适合读取认证信息、客户端版本、语言、追踪 ID 等 Header。

- [`@RequestHeader` 官方文档](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/requestheader.html)

### 12.6 `@CookieValue`

```java
@GetMapping("/settings")
public Settings settings(
        @CookieValue(name = "theme", defaultValue = "light") String theme) {
    // ...
}
```

- [`@CookieValue` 官方文档](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/cookievalue.html)

### 12.7 `@ModelAttribute`

适合把查询参数或表单字段绑定到一个对象：

```java
public class UserSearchCondition {
    private String keyword;
    private Integer page;
    private Integer size;

    // getters and setters
}

@GetMapping("/users")
public List<User> search(@ModelAttribute UserSearchCondition condition) {
    // ...
}
```

请求：

```http
GET /users?keyword=alice&page=1&size=20
```

它与 `@RequestBody` 的关键差别是：`@ModelAttribute` 走数据绑定，常用于 query/form；`@RequestBody` 通过消息转换器读取整个 body，常用于 JSON/XML。

- [`@ModelAttribute` 方法参数](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/modelattrib-method-args.html)

### 12.8 `@RequestPart`

适合 `multipart/form-data` 中既有文件又有结构化数据的请求：

```java
@PostMapping(path = "/avatars", consumes = "multipart/form-data")
public void upload(
        @RequestPart("metadata") AvatarMetadata metadata,
        @RequestPart("file") MultipartFile file) {
    // ...
}
```

- [Multipart Forms](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/multipart-forms.html)

### 12.9 设计选择：Path 还是 Query

这不是语法强制规则，但常见 REST 设计约定是：

```text
GET /users/42                 定位 ID 为 42 的用户
GET /users?status=active      按条件筛选用户集合
GET /users?page=2&size=20     分页参数
```

简化记忆：

- Path 常表达“哪个资源”；
- Query 常表达“怎样查询、筛选、排序或分页”；
- Body 常表达“要创建或修改的数据内容”。

### 12.10 常见绑定失败与 HTTP 状态

| 情况 | 常见结果 |
|---|---|
| `/users/{id}` 没有匹配到实际路径段 | 路由不匹配，通常 404 |
| 缺少必需的 `@RequestParam` | 400 Bad Request |
| `id=abc` 要转换为 `Integer` | 类型转换失败，通常 400 |
| JSON 语法错误或字段类型不匹配 | 消息读取失败，通常 400 |
| 请求 `Content-Type` 不被方法支持 | 415 Unsupported Media Type |

官方文档：

- [Spring MVC 类型转换](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/typeconversion.html)
- [Controller 方法参数总表](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/arguments.html)

---

## 13. `Integer` 与 `int`：到底应该用哪个

### 13.1 基本差异

| 特性 | `int` | `Integer` |
|---|---|---|
| 类型 | 基本类型 | 包装类型、对象 |
| 能否为 `null` | 不能 | 可以 |
| 对象字段默认值 | `0` | `null` |
| 局部变量默认值 | 没有，使用前必须初始化 | 没有，使用前必须初始化 |
| 泛型中能否直接使用 | 不能写 `List<int>` | 可以写 `List<Integer>` |
| 额外风险 | 无拆箱空指针 | `null` 自动拆箱时可能 `NullPointerException` |

注意：“`int` 默认是 0”只适用于对象字段、静态字段和数组元素；局部变量没有默认值。

### 13.2 Controller 中：`int` 完全可以使用

对于必需的路径变量：

```java
@GetMapping("/{id}")
public User findById(@PathVariable("id") int id) {
    return userService.findById(id);
}
```

这是合法且合理的。因为路由语义本身要求必须有 `id`；没有路径段时通常不会进入这个方法。

下面的 `Integer` 也合法：

```java
@PathVariable("id") Integer id
```

选择依据不应是“MyBatis 只能接收包装类型”，而应是业务上是否需要表达 `null`。

### 13.3 可选的 `@RequestParam` 更适合 `Integer` 或 `Optional<Integer>`

```java
@GetMapping
public List<User> search(
        @RequestParam(required = false) Integer age) {
    // age 可以为 null
}
```

也可以：

```java
@RequestParam Optional<Integer> age
```

如果要使用 `int`，请给默认值：

```java
@RequestParam(defaultValue = "0") int age
```

否则一个缺失的可选值没有办法赋给基本类型。

### 13.4 实体类和 DTO 中如何选择

数据库列允许 `NULL` 时：

```java
private Integer age;
```

数据库列明确 `NOT NULL`，并且 `0` 是合理默认值时，可以使用：

```java
private int age;
```

自增主键在插入前常常处于“尚未生成”状态，因此包装类型更容易表达：

```java
private Integer id; // 插入前 null，回填后有值
```

### 13.5 MyBatis Mapper 参数中，两者都能工作

下面两种签名 MyBatis 都可以处理：

```java
User findById(int id);
```

```java
User findById(Integer id);
```

对于 `findById` 这种必需参数，`int` 并没有技术问题。使用 `Integer` 时，应明确处理 `null`，而不是让它无意中进入 SQL。

危险做法：

```xml
<where>
    <if test="id != null">
        id = #{id}
    </if>
</where>
```

如果一个“按 ID 查询”的方法收到 `null` 后直接删除整个 `WHERE` 条件，就可能意外查询全表。更安全的是在 Controller/Service 边界校验必需参数。

另外：

```sql
WHERE id = NULL
```

在 SQL 三值逻辑中并不会像普通等号那样匹配空值；判断 SQL `NULL` 应使用 `IS NULL`。主键通常本来也不允许为空。

### 13.6 自动装箱与拆箱风险

```java
Integer id = 5; // 自动装箱
int value = id; // 自动拆箱
```

但：

```java
Integer id = null;
int value = id; // NullPointerException
```

因此包装类型提供了 `null` 语义，也要求调用方处理这种语义。

### 13.7 一条实用判断规则

```text
“没有值”是否是合法且有意义的状态？
    ├─ 是 → Integer / Long / Optional 等
    └─ 否 → int / long，或在边界立即校验包装类型非空
```

---

## 14. 一套完整、连贯的示例

### 14.1 Entity

```java
package com.hs.homework.August.day20ssm.entity;

import com.fasterxml.jackson.annotation.JsonIgnore;

public class User {
    private Integer id;
    private String username;

    @JsonIgnore
    private String password;

    private String sex;

    // getters and setters
}
```

### 14.2 Mapper

```java
package com.hs.homework.August.day20ssm.mapper;

@Mapper
public interface Day20UserMapper {
    List<User> findAll();

    User findById(@Param("id") Integer id);
}
```

### 14.3 Service 接口

```java
package com.hs.homework.August.day20ssm.service;

public interface Day20UserService {
    List<User> findAll();

    User findById(Integer id);
}
```

### 14.4 Service 实现

```java
package com.hs.homework.August.day20ssm.service.impl;

@Service
public class Day20UserServiceImpl implements Day20UserService {
    private final Day20UserMapper userMapper;

    public Day20UserServiceImpl(Day20UserMapper userMapper) {
        this.userMapper = userMapper;
    }

    @Override
    public List<User> findAll() {
        return userMapper.findAll();
    }

    @Override
    public User findById(Integer id) {
        if (id == null) {
            throw new IllegalArgumentException("id must not be null");
        }
        return userMapper.findById(id);
    }
}
```

### 14.5 REST Controller

```java
package com.hs.homework.August.day20ssm.controller;

@RestController
@RequestMapping("/users")
public class Day20UserController {
    private final Day20UserService userService;

    public Day20UserController(Day20UserService userService) {
        this.userService = userService;
    }

    @GetMapping
    public List<User> findAll() {
        return userService.findAll();
    }

    @GetMapping("/{id}")
    public User findById(@PathVariable("id") Integer id) {
        return userService.findById(id);
    }
}
```

### 14.6 application.yml

```yaml
mybatis:
  mapper-locations: classpath*:mapper/**/*.xml
  type-aliases-package: com.hs.homework.August.day20ssm.entity
  configuration:
    map-underscore-to-camel-case: true
```

### 14.7 Mapper XML

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.hs.homework.August.day20ssm.mapper.Day20UserMapper">
    <select id="findAll" resultType="user">
        SELECT id, username, password, sex
        FROM demo_20260820_ssm_user
    </select>

    <select id="findById" resultType="user">
        SELECT id, username, password, sex
        FROM demo_20260820_ssm_user
        WHERE id = #{id}
    </select>
</mapper>
```

### 14.8 请求结果

```http
GET /users/1
Accept: application/json
```

流程：

```text
/{id} 捕获 "1"
    ↓
@PathVariable 转换为 Integer 1
    ↓
Controller 调用 Service 接口
    ↓
Spring 注入 Day20UserServiceImpl
    ↓
Service 调用 MyBatis Mapper 代理
    ↓
Mapper XML 执行 SQL
    ↓
resultType="user" 映射为 User
    ↓
@RestController 触发 Response Body 处理
    ↓
Jackson 把 User 序列化为 JSON
```

可能的响应：

```json
{
  "id": 1,
  "username": "zhangsan",
  "password": "***",
  "sex": "male"
}
```

这里用 `@JsonIgnore` 防止示例中的密码进入 JSON。实际项目更推荐使用专门的响应 DTO，只暴露 API 确实需要的字段。

---

## 15. 常见误区纠正

### 误区 1：接口上的 `@Service` 会像 MyBatis 一样自动生成代理

不对。普通 Service 接口没有代理生成器。把 `@Service` 放在具体实现类上；MyBatis Mapper 能生成代理，是因为 MyBatis-Spring 提供了 `MapperFactoryBean`。

### 误区 2：接口和实现类天然是两个 Spring Bean

不对。接口是依赖类型，具体实现对象才是普通 Bean。只有实际注册了两个可赋值给该接口的对象时，才会出现多候选冲突。

### 误区 3：返回 `Result` 对象但没有 `@ResponseBody`，对象的 `toString()` 一定会成为视图名

不准确。普通复杂对象通常会作为 model attribute；若没有显式视图名，Spring MVC 还可能推导默认视图名。`String` 返回值才通常直接解释为逻辑视图名。

### 误区 4：缺少 `@ResponseBody` 一定返回 404

不对。它会使返回值进入 Model/View 分支；最终可能是 404，也可能是 500 或模板解析异常，取决于视图配置。

### 误区 5：请求 `/users/` 时，`/{id}` 会把 `id` 绑定为 `null`

通常不对。缺少该路径段意味着路由不匹配，而不是成功匹配后注入 `null`。

### 误区 6：MyBatis Mapper 参数必须用 `Integer`，不能用 `int`

不对。两者都能使用。是否允许 `null` 才是主要选择依据。

### 误区 7：所有 `int` 变量默认都是 `0`

不对。字段和数组元素有默认值；局部变量必须先初始化。

### 误区 8：`type-aliases-package` 用来查找 Mapper XML

不对。它扫描 Java 类型并注册短别名；Mapper XML 位置由 `mapper-locations` 等配置负责。

### 误区 9：Type Alias 会把实体类注册成 Spring Bean

不对。MyBatis 的类型别名注册表与 Spring IoC 的 Bean 注册表是两个不同机制。

---

## 16. 故障排查速查表

| 错误或现象 | 优先检查 |
|---|---|
| `NoSuchBeanDefinitionException` | 实现类是否注册、组件扫描范围、包结构 |
| `NoUniqueBeanDefinitionException` | 是否有多个实现、`@Bean`、代理或测试替身；再考虑 `@Primary`/`@Qualifier` |
| `Invalid bound statement (not found)` | Mapper XML 是否加载、namespace 与 statement id |
| `Parameter 'xxx' not found` | `@Param` 名称与 `#{xxx}` 是否一致 |
| `The alias 'Car' is already mapped` | 别名扫描范围、同名类、重复 `@Alias` |
| `SqlSessionFactory` 创建失败 | 继续读取最底层 `Caused by`，检查别名、XML、配置和数据源 |
| SQL 已执行，但浏览器 404/模板异常 | `@Controller`、`@ResponseBody`、`@RestController` 与 ViewResolver |
| `/users` 访问 404，但只有 `/{id}` 方法 | 路径本身不匹配，增加集合路由或提供 id |
| `id=abc` 返回 400 | String 到 `Integer`/`int` 的转换失败 |
| JSON 请求返回 415 | `Content-Type`、`consumes` 与消息转换器 |

---

## 17. 官方文档索引

### Spring Framework

- [IoC Container](https://docs.spring.io/spring-framework/reference/core/beans.html)
- [Dependency Injection](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html)
- [Classpath Scanning and Managed Components](https://docs.spring.io/spring-framework/reference/core/beans/classpath-scanning.html)
- [`@Service` Javadoc](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Service.html)
- [`@Autowired`](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/autowired.html)
- [`@Primary`](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/autowired-primary.html)
- [`@Qualifier`](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/autowired-qualifiers.html)
- [Spring MVC](https://docs.spring.io/spring-framework/reference/web/webmvc.html)
- [Request Mapping](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-requestmapping.html)
- [Controller Method Arguments](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/arguments.html)
- [Type Conversion](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/typeconversion.html)
- [`@RequestParam`](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/requestparam.html)
- [`@RequestBody`](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/requestbody.html)
- [`@ResponseBody`](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/responsebody.html)
- [Controller Return Values](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/return-types.html)
- [View Resolution](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-servlet/viewresolver.html)
- [HTTP Message Conversion](https://docs.spring.io/spring-framework/reference/web/webmvc/message-converters.html)

### MyBatis / MyBatis-Spring

- [MyBatis Configuration：Type Aliases](https://mybatis.org/mybatis-3/configuration.html#typeAliases)
- [MyBatis Configuration 中文版：类型别名](https://mybatis.org/mybatis-3/zh_CN/configuration.html#typeAliases)
- [MyBatis Mapper XML](https://mybatis.org/mybatis-3/sqlmap-xml.html)
- [MyBatis-Spring：Injecting Mappers](https://mybatis.org/spring/mappers.html)
- [MyBatis Spring Boot Starter 配置](https://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/)
- [MyBatis Spring Boot Starter 中文配置](https://mybatis.org/spring-boot-starter/zh/mybatis-spring-boot-autoconfigure/index.html)
- [`@Alias` Javadoc](https://mybatis.org/mybatis-3/apidocs/org/apache/ibatis/type/Alias.html)
- [`TypeAliasRegistry` 官方源码](https://mybatis.org/mybatis-3/xref/org/apache/ibatis/type/TypeAliasRegistry.html)

---

## 18. 最终记忆框架

```text
请求数据从哪里来？
    Path        → @PathVariable
    Query/Form  → @RequestParam / @ModelAttribute
    JSON Body   → @RequestBody
    Header      → @RequestHeader
    Cookie      → @CookieValue

业务对象怎样连接？
    Controller 依赖 Service 接口
    @Service 放在具体实现类
    Spring 按可赋值类型注入唯一 Bean

Mapper 为什么没有实现类也能注入？
    MyBatis-Spring 为 Mapper 接口创建动态代理

MyBatis 怎样找到内容？
    mapper-locations       → Mapper XML
    @Mapper / @MapperScan  → Mapper 接口代理
    type-aliases-package   → Java 类型别名

Controller 返回值去哪里？
    @RestController / @ResponseBody → Response Body / 消息转换
    普通 @Controller               → Model / View / ViewResolver

int 还是 Integer？
    是否需要表达“没有值”决定，而不是由 MyBatis 强制决定
```
