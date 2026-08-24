# MyBatis 启动流程与 `Invalid bound statement` 绑定失败案例复盘

> 出错位置是 application.xml 中 mybatis: mapper-locations: classpath: com/hs/homework/August/day21ssm/mapper/*.xml 将"/"写成"."导致
>
> 案例项目：`java-homework`  
> 启动类：`com.hs.homework.August.day21ssm.Day21StartApplication`  
> Mapper：`Day21HomeWorkUserMapper`  
> XML：`Day21HomeWorkUserMapper.xml`  
> 典型异常：  
>
> `org.apache.ibatis.binding.BindingException: Invalid bound statement (not found): com.hs.homework.August.day21ssm.homework.mapper.Day21HomeWorkUserMapper.selectById`

---

## 0. 先给出整个流程的总图

在 Spring Boot + MyBatis 中，要把“启动”和“一次 Mapper 方法调用”分成两个阶段理解。

```text
                    Spring Boot 启动阶段
                           │
                           ▼
                  SpringApplication.run()
                           │
                           ▼
                创建 ApplicationContext
                           │
                           ▼
             读取 application.yaml / properties
                           │
                           ▼
                  MyBatis 自动配置生效
                           │
              ┌────────────┴────────────┐
              ▼                         ▼
       创建 SqlSessionFactory      扫描 Mapper 接口
              │                         │
              ▼                         ▼
       读取 Mapper XML          注册 MapperFactoryBean
              │                         │
              ▼                         ▼
       解析 namespace / SQL        创建 Mapper 代理对象
              │                         │
              ▼                         ▼
    MappedStatement 注册进        注入到 Service
       Configuration
              │
              └────────────┬────────────┘
                           ▼
                    Spring Boot 启动完成


                    HTTP 请求执行阶段
                           │
                           ▼
                    DispatcherServlet
                           │
                           ▼
                       Controller
                           │
                           ▼
                        Service
                           │
                           ▼
                userMapper.selectById(id)
                           │
                           ▼
                    JDK 动态代理对象
                       $Proxy65
                           │
                           ▼
                    MapperProxy.invoke()
                           │
                           ▼
                       MapperMethod
                           │
                           ▼
               根据 namespace + methodName
                  查找 MappedStatement
                           │
                ┌──────────┴──────────┐
                │                     │
              找到                   找不到
                │                     │
                ▼                     ▼
          SqlSession 执行 SQL    BindingException
                │
                ▼
              Executor
                │
                ▼
         StatementHandler / JDBC
                │
                ▼
              MySQL
```

本案例失败的位置正是：

```text
MapperProxy
    ↓
MapperMethod
    ↓
查找 MappedStatement
    ↓
找不到 Day21HomeWorkUserMapper.selectById
    ↓
BindingException
```

也就是说：

> **Mapper 接口已经扫描成功，Mapper 代理对象也已经创建成功；失败的是 XML 中的 SQL Statement 没有成功注册到 MyBatis 的 `Configuration` 中。**

---

# 一、案例背景

项目中的 Mapper 接口大致如下：

```java
package com.hs.homework.August.day21ssm.homework.mapper;

public interface Day21HomeWorkUserMapper {

    int insert(Day21HomeWorkUser user);

    int deleteById(@Param("id") Long id);

    int updateById(Day21HomeWorkUser user);

    Day21HomeWorkUser selectById(@Param("id") Long id);
}
```

启动类通过：

```java
@MapperScan(
    basePackages = {
        "com.hs.homework.August.day21ssm.homework.mapper"
    }
)
```

扫描 Mapper 接口。

Mapper XML 位于资源目录：

```text
src/main/resources/
└── com/
    └── hs/
        └── homework/
            └── August/
                └── day21ssm/
                    └── mapper/
                        └── Day21HomeWorkUserMapper.xml
```

正确的 MyBatis XML 路径配置应该类似：

```yaml
mybatis:
  mapper-locations: classpath*:com/hs/homework/August/day21ssm/mapper/*.xml
```

而案例中曾写成：

```yaml
mybatis:
  mapper-locations: classpath:com.hs.homework.August.day21ssm.mapper/*.xml
```

问题就在于：

```text
Java 包名               com.hs.homework.August.day21ssm.mapper
                         使用 .

classpath 资源路径       com/hs/homework/August/day21ssm/mapper
                         使用 /
```

---

# 二、Spring Boot 启动时 MyBatis 到底做了什么

## 1. 从 `main()` 开始

启动类：

```java
public static void main(String[] args) {
    SpringApplication.run(Day21StartApplication.class, args);
}
```

进入：

```text
SpringApplication.run(...)
```

Spring Boot 开始创建并刷新：

```text
ApplicationContext
```

案例日志可以看到应用使用 Java 17 启动，并激活 `dev` profile：

```text
Starting Day21StartApplication using Java 17.0.11
The following 1 profile is active: "dev"
```

日志来源：`Pasted text(8).txt` 第 13～14 行。

---

## 2. Spring Boot 读取配置文件

在 Environment 准备阶段，Spring Boot 会读取：

```text
application.properties
application.yaml
application-dev.yaml
...
```

因为案例激活了：

```text
dev
```

所以如果存在：

```text
application-dev.yaml
```

也可能参与最终配置。

MyBatis Starter 后续会从 Spring Boot Environment 中获取：

```yaml
mybatis:
  mapper-locations:
  type-aliases-package:
  configuration:
```

这些配置。

其中本案例最重要的是：

```yaml
mapper-locations
```

因为它决定：

> **MyBatis 到哪里寻找 Mapper XML。**

---

# 三、MyBatis 自动配置阶段

项目 classpath 中存在：

```text
mybatis-spring-boot-starter-3.0.5
mybatis-spring-boot-autoconfigure-3.0.5
mybatis-3.5.19
mybatis-spring-3.0.5
```

这些依赖可以从本次启动日志的 classpath 中看到。

Spring Boot 自动配置生效后，核心目标之一是创建：

```text
SqlSessionFactory
```

可以把它理解为：

> **MyBatis 的核心工厂对象。**

后续所有 Mapper SQL 执行最终都要依赖它创建或管理的 SqlSession 体系。

---

# 四、创建 `SqlSessionFactory`

概念上的调用关系可以理解为：

```text
MybatisAutoConfiguration
        ↓
SqlSessionFactoryBean
        ↓
buildSqlSessionFactory()
        ↓
Configuration
        ↓
SqlSessionFactory
```

这里有一个非常重要的对象：

```java
org.apache.ibatis.session.Configuration
```

MyBatis 的大量运行期元数据都会放在这个对象中。

例如：

```text
MappedStatement
ResultMap
ParameterMap
TypeAliasRegistry
TypeHandlerRegistry
MapperRegistry
...
```

因此学习 MyBatis 底层时，可以先建立一个核心认识：

> **`Configuration` 就像 MyBatis 运行时的“总配置仓库”。**

---

# 五、`mapper-locations` 的作用：找到 XML

假设配置是：

```yaml
mybatis:
  mapper-locations: classpath*:com/hs/homework/August/day21ssm/mapper/*.xml
```

Spring Boot/MyBatis 会把这个模式解析成 Resource，然后找到：

```text
Day21HomeWorkUserMapper.xml
```

接下来才有资格进入 XML 解析阶段。

---

## 5.1 本案例为什么会失败

案例中写成了：

```yaml
classpath:com.hs.homework.August.day21ssm.mapper/*.xml
```

这里：

```text
com.hs.homework...
```

并不是实际 classpath 文件路径。

实际资源路径是：

```text
com/hs/homework/August/day21ssm/mapper/Day21HomeWorkUserMapper.xml
```

所以如果匹配不到 XML：

```text
mapper-locations
       ↓
找不到 XML Resource
       ↓
XMLMapperBuilder 没机会解析该 XML
       ↓
<select id="selectById"> 没有注册
       ↓
Configuration 中没有对应 MappedStatement
```

这就是后面请求时 `Invalid bound statement` 的根源之一。

---

# 六、XML 被找到以后：`XMLMapperBuilder` 解析 Mapper XML

假设 XML 成功找到，MyBatis 会解析：

```xml
<mapper namespace="...">

    <resultMap ... />

    <sql ... />

    <select ... />

    <insert ... />

    <update ... />

    <delete ... />

</mapper>
```

可以重点关注：

```text
XMLMapperBuilder
```

概念流程：

```text
Day21HomeWorkUserMapper.xml
        ↓
XMLMapperBuilder
        ↓
parse()
        ↓
configurationElement(...)
        ↓
解析 namespace
解析 resultMap
解析 sql
解析 select / insert / update / delete
        ↓
创建 MappedStatement
        ↓
注册到 Configuration
```

---

# 七、最重要的概念：`MappedStatement`

Mapper XML 中每一条：

```xml
<select id="selectById">
```

并不是运行时再临时读取 XML。

Spring Boot/MyBatis 启动阶段会把它解析成一个 Java 对象：

```text
MappedStatement
```

然后放进：

```text
Configuration
```

可以抽象理解成：

```java
Map<String, MappedStatement> mappedStatements;
```

例如下面的 XML：

```xml
<mapper namespace="com.hs.homework.August.day21ssm.homework.mapper.Day21HomeWorkUserMapper">

    <select id="selectById">
        SELECT ...
    </select>

</mapper>
```

最终 Statement ID 是：

```text
namespace + "." + id
```

也就是：

```text
com.hs.homework.August.day21ssm.homework.mapper.Day21HomeWorkUserMapper
+
.
+
selectById
```

最终完整 ID：

```text
com.hs.homework.August.day21ssm.homework.mapper.Day21HomeWorkUserMapper.selectById
```

这个字符串非常关键。

因为运行时 MyBatis 就是拿这个 ID 去找 `MappedStatement`。

---

# 八、为什么 XML 的 `namespace` 必须和 Mapper 接口对应

Java：

```java
package com.hs.homework.August.day21ssm.homework.mapper;

public interface Day21HomeWorkUserMapper {
    Day21HomeWorkUser selectById(Long id);
}
```

Java Mapper 的完全限定名：

```text
com.hs.homework.August.day21ssm.homework.mapper.Day21HomeWorkUserMapper
```

XML 必须写：

```xml
<mapper namespace="com.hs.homework.August.day21ssm.homework.mapper.Day21HomeWorkUserMapper">
```

里面的方法：

```xml
<select id="selectById">
```

最终才能组成：

```text
com.hs.homework.August.day21ssm.homework.mapper.Day21HomeWorkUserMapper.selectById
```

与 Java 方法对应。

---

# 九、另一条启动链：`@MapperScan` 扫描 Mapper 接口

前面讲的是：

```text
XML → MappedStatement
```

但是 Java Mapper 接口本身也需要被 Spring/MyBatis 找到。

本项目启动类使用：

```java
@MapperScan(
        "com.hs.homework.August.day21ssm.homework.mapper"
)
```

这一部分概念上会进入：

```text
@MapperScan
    ↓
MapperScannerRegistrar
    ↓
MapperScannerConfigurer
    ↓
ClassPathMapperScanner
    ↓
扫描 Day21HomeWorkUserMapper
```

---

# 十、Mapper 接口没有实现类，为什么可以注入？

这是 MyBatis 最核心的问题之一。

你写的是：

```java
public interface Day21HomeWorkUserMapper {
    Day21HomeWorkUser selectById(Long id);
}
```

它只是：

```text
interface
```

没有：

```java
class Day21HomeWorkUserMapperImpl
```

但是 Service 却可以：

```java
private final Day21HomeWorkUserMapper userMapper;
```

原因是 MyBatis 为接口创建了：

```text
JDK 动态代理对象
```

---

# 十一、扫描 Mapper 后注册的并不是普通实现类

`@MapperScan` 扫描到 Mapper 接口以后，会为它注册与：

```text
MapperFactoryBean
```

相关的 BeanDefinition。

概念关系：

```text
Day21HomeWorkUserMapper.class
        ↓
MapperFactoryBean
        ↓
Spring Bean 创建阶段
        ↓
SqlSession.getMapper(...)
        ↓
MyBatis Configuration.getMapper(...)
        ↓
MapperRegistry.getMapper(...)
        ↓
MapperProxyFactory.newInstance(...)
        ↓
Proxy.newProxyInstance(...)
        ↓
$ProxyXX
```

所以 Spring 注入给 Service 的：

```java
Day21HomeWorkUserMapper userMapper
```

真实对象可以理解为：

```text
jdk.proxy2.$Proxy65
```

而不是某个手写的：

```text
Day21HomeWorkUserMapperImpl
```

---

# 十二、启动完成时应该具备的两大条件

应用启动成功以后，MyBatis 至少需要准备好两类东西。

## 12.1 Mapper 代理对象

```text
Day21HomeWorkUserMapper
        ↓
MapperProxyFactory
        ↓
JDK Proxy
        ↓
可以注入到 Service
```

## 12.2 SQL Statement

```text
Day21HomeWorkUserMapper.xml
        ↓
XMLMapperBuilder
        ↓
MappedStatement
        ↓
Configuration
```

正常情况下：

```text
Mapper Proxy
        +
MappedStatement
        =
Mapper 方法可以执行
```

本案例实际状态却是：

```text
Mapper Proxy       √ 已成功
MappedStatement    × 缺失
```

因此：

```text
应用可以启动成功
但调用 Mapper 方法时才报错
```

这也是这个案例最值得理解的地方。

---

# 十三、从日志证明 Spring Boot 和 MyBatis 已经启动完成

日志中出现：

```text
Tomcat started on port 8080
Started Day21StartApplication in 1.999 seconds
```

对应原日志第 30～31 行。

这说明：

```text
Spring ApplicationContext 已经刷新完成
Tomcat 已经启动
Day21StartApplication 已经成功启动
```

所以这个错误不是：

```text
“Spring Boot 启动失败”
```

而是：

```text
“应用启动成功后，在第一次 HTTP 请求执行 Mapper 方法时失败”
```

---

# 十四、请求到来以后：Spring MVC 调用 Controller

Apifox 发请求后，日志中首先出现：

```text
Initializing Spring DispatcherServlet 'dispatcherServlet'
Completed initialization in 0 ms
```

对应原日志第 32～34 行。

接着调用 Controller：

```java
@GetMapping("/{id}")
public Result getById(@PathVariable Long id) {
    return Result.success(userService.getById(id));
}
```

调用关系：

```text
HTTP Request
    ↓
Tomcat
    ↓
DispatcherServlet
    ↓
RequestMappingHandlerAdapter
    ↓
Day21HomeWorkUserController.getById()
```

---

# 十五、Controller 进入 Service

Controller：

```java
userService.getById(id)
```

进入：

```java
public Day21HomeWorkUser getById(Long id) {
    return userMapper.selectById(id);
}
```

此时关键调用发生：

```java
userMapper.selectById(id);
```

---

# 十六、这一行为什么不是直接执行接口方法？

因为：

```java
userMapper
```

实际指向的是 MyBatis 创建的 JDK 动态代理。

所以：

```java
userMapper.selectById(id)
```

运行时更接近于：

```text
$Proxy65.selectById(id)
```

案例日志明确出现：

```text
jdk.proxy2/jdk.proxy2.$Proxy65.selectById(Unknown Source)
```

原日志第 45 行。

这一行是判断：

> **调用已经进入 Mapper 动态代理对象**

的重要证据。

---

# 十七、`$Proxy65.selectById()` 进入 `MapperProxy.invoke()`

JDK 动态代理的核心规则是：

```text
代理对象的方法调用
        ↓
InvocationHandler.invoke(...)
```

MyBatis 的 InvocationHandler 就是：

```text
MapperProxy
```

所以：

```text
$Proxy65.selectById()
        ↓
MapperProxy.invoke()
```

日志中有：

```text
org.apache.ibatis.binding.MapperProxy.invoke(MapperProxy.java:86)
```

因此完整证据是：

```text
$Proxy65.selectById
        +
MapperProxy.invoke
```

两行同时出现时，可以非常明确地判断：

> Java 代码已经进入 MyBatis Mapper 代理执行逻辑。

---

# 十八、`MapperProxy` 接下来创建/获取 `MapperMethod`

`MapperProxy.invoke()` 不会自己写 SQL。

它会把：

```text
Java Method
```

转换成 MyBatis 可以执行的：

```text
MapperMethod
```

日志中出现：

```text
MapperProxy.cachedInvoker
MapperProxy.lambda$cachedInvoker$0
MapperMethod.<init>
MapperMethod$SqlCommand.<init>
```

对应原日志第 38～44 行。

概念流程：

```text
MapperProxy.invoke()
        ↓
cachedInvoker()
        ↓
MapperMethod
        ↓
MapperMethod.SqlCommand
```

---

# 十九、`MapperMethod.SqlCommand` 做了一件非常关键的事

它需要根据当前 Java Mapper 方法找到 SQL Statement。

当前 Java 方法：

```java
Day21HomeWorkUserMapper.selectById(...)
```

MyBatis 会构造/查找：

```text
com.hs.homework.August.day21ssm.homework.mapper.Day21HomeWorkUserMapper.selectById
```

然后去：

```text
Configuration
```

中查对应：

```text
MappedStatement
```

可以理解为：

```java
configuration.getMappedStatement(statementId);
```

这里的 `statementId`：

```text
Mapper 接口完全限定名 + "." + Mapper 方法名
```

---

# 二十、本案例的真正失败位置

日志：

```text
org.apache.ibatis.binding.BindingException:
Invalid bound statement (not found):
com.hs.homework.August.day21ssm.homework.mapper.Day21HomeWorkUserMapper.selectById
```

对应原日志第 35～37 行。

并且异常最深处首先出现：

```text
MapperMethod$SqlCommand.<init>(MapperMethod.java:229)
MapperMethod.<init>(MapperMethod.java:53)
```

因此可以定位：

```text
MapperMethod.SqlCommand
        ↓
准备寻找 SQL Statement
        ↓
Configuration 中找不到
        ↓
BindingException
```

注意：

> **此时还没有执行 JDBC，也没有真正向 MySQL 发送 SQL。**

所以这个异常不是数据库 SQL 语法错误。

---

# 二十一、本案例完整调用链

Java 异常堆栈通常应该从下往上还原调用过程。

案例关键堆栈：

```text
Day21HomeWorkUserController.getById()
        ↑
Day21HomeWorkUserService.getById()
        ↑
$Proxy65.selectById()
        ↑
MapperProxy.invoke()
        ↑
MapperProxy.cachedInvoker()
        ↑
MapperMethod.<init>()
        ↑
MapperMethod$SqlCommand.<init>()
        ↑
BindingException
```

如果按真正“调用方向”重画：

```text
Day21HomeWorkUserController.getById()
        ↓
Day21HomeWorkUserService.getById()
        ↓
userMapper.selectById()
        ↓
$Proxy65.selectById()
        ↓
MapperProxy.invoke()
        ↓
MapperProxy.cachedInvoker()
        ↓
MapperMethod
        ↓
MapperMethod.SqlCommand
        ↓
查找 MappedStatement
        ↓
没有找到
        ↓
BindingException
```

日志依据：原日志第 37～47 行。

---

# 二十二、为什么这能证明 `@MapperScan` 已经成功？

如果：

```java
@MapperScan
```

根本没有扫描到：

```text
Day21HomeWorkUserMapper
```

那么 Spring 连：

```text
Day21HomeWorkUserMapper Bean
```

都创建不了。

Service 在构造时就会失败，例如出现类似：

```text
No qualifying bean of type 'Day21HomeWorkUserMapper'
```

但本案例不是这样。

本案例日志出现：

```text
$Proxy65.selectById
MapperProxy.invoke
```

说明：

```text
@MapperScan
        ↓
Mapper 接口被找到
        ↓
MapperFactoryBean 工作
        ↓
MapperProxyFactory 工作
        ↓
JDK Proxy 创建成功
        ↓
代理对象成功注入 Service
```

所以 Java Mapper 扫描链没有断。

---

# 二十三、为什么 XML 没有加载却不一定导致启动失败？

这是初学 MyBatis 时非常容易困惑的地方。

Java Mapper 接口可以被扫描并创建代理：

```text
Day21HomeWorkUserMapper
        ↓
Proxy
        √
```

但是对应 XML Statement 没被加载：

```text
Day21HomeWorkUserMapper.xml
        ↓
MappedStatement
        ×
```

此时 Spring 仍然可能正常创建：

```text
Controller
Service
Mapper Proxy
```

直到真正调用：

```java
userMapper.selectById(id)
```

MyBatis 才需要查：

```text
MappedStatement
```

这时才发现：

```text
不存在
```

于是运行期抛：

```text
BindingException
```

因此要区分：

```text
Bean 创建成功
≠
Mapper SQL 绑定一定成功
```

---

# 二十四、正确的 XML 绑定条件

以：

```java
Day21HomeWorkUser selectById(@Param("id") Long id);
```

为例，XML 至少要满足以下条件。

## 条件 1：XML 必须被 `mapper-locations` 找到

```yaml
mybatis:
  mapper-locations: classpath*:com/hs/homework/August/day21ssm/mapper/*.xml
```

## 条件 2：namespace 正确

```xml
<mapper namespace="com.hs.homework.August.day21ssm.homework.mapper.Day21HomeWorkUserMapper">
```

## 条件 3：statement id 正确

```xml
<select id="selectById">
```

## 条件 4：XML 被复制进运行时 classpath

编译后应该能看到类似：

```text
target/classes/
└── com/
    └── hs/
        └── homework/
            └── August/
                └── day21ssm/
                    └── mapper/
                        └── Day21HomeWorkUserMapper.xml
```

---

# 二十五、为什么 `mapper-locations` 使用 `/`，而 `@MapperScan` 使用 `.`

这是本案例最直接的错误点。

## `@MapperScan`

扫描的是：

```text
Java package
```

所以：

```java
@MapperScan(
    "com.hs.homework.August.day21ssm.homework.mapper"
)
```

使用：

```text
.
```

## `mapper-locations`

扫描的是：

```text
classpath Resource
```

所以：

```yaml
mapper-locations: classpath*:com/hs/homework/August/day21ssm/mapper/*.xml
```

使用：

```text
/
```

总结：

```text
@MapperScan
        ↓
找 Java class
        ↓
Java 包名规则
        ↓
com.hs.homework.xxx


mapper-locations
        ↓
找资源文件
        ↓
文件路径规则
        ↓
com/hs/homework/xxx
```

---

# 二十六、修复后的配置

## `Day21StartApplication`

```java
@MapperScan(
        "com.hs.homework.August.day21ssm.homework.mapper"
)
@SpringBootApplication
public class Day21StartApplication {

    public static void main(String[] args) {
        SpringApplication.run(Day21StartApplication.class, args);
    }
}
```

## `application.yaml`

```yaml
mybatis:
  mapper-locations: classpath*:com/hs/homework/August/day21ssm/mapper/*.xml

  type-aliases-package: com.hs.homework.August.day21ssm.homework.entity

  configuration:
    map-underscore-to-camel-case: true
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    lazy-loading-enabled: true
```

## `Day21HomeWorkUserMapper.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.hs.homework.August.day21ssm.homework.mapper.Day21HomeWorkUserMapper">

    <select id="selectById" resultType="Day21HomeWorkUser">
        SELECT
            id,
            username,
            password,
            nickname,
            address,
            phone,
            register_time
        FROM demo_20260821_ssm_user
        WHERE id = #{id}
    </select>

</mapper>
```

---

# 二十七、修复后一次正常 `selectById()` 应该怎样执行

如果 XML 成功注册，则运行时：

```text
Controller
    ↓
Service
    ↓
Mapper Proxy
    ↓
MapperProxy.invoke
    ↓
MapperMethod
    ↓
Configuration.getMappedStatement(...)
    ↓
找到 MappedStatement
    ↓
SqlSession
    ↓
Executor
    ↓
StatementHandler
    ↓
PreparedStatement
    ↓
MySQL
    ↓
ResultSet
    ↓
ResultSetHandler
    ↓
Day21HomeWorkUser
    ↓
Service
    ↓
Controller
    ↓
Result.success(...)
    ↓
JSON
```

---

# 二十八、`SqlSession` 在哪里进入流程？

Mapper Proxy 最终并不直接访问 JDBC。

它会通过：

```text
SqlSession
```

执行。

在 Spring 环境下常见的是：

```text
SqlSessionTemplate
```

可以把 Mapper 代理理解成：

```text
Mapper Proxy
      │
      │ 我知道你调用的是 selectById
      ↓
MapperMethod
      │
      │ 我知道 statementId
      ↓
SqlSession
      │
      │ 我负责调用 selectOne / selectList / insert / update / delete
      ↓
Executor
```

例如返回单个实体的方法最终通常走类似：

```text
sqlSession.selectOne(statement, parameter)
```

---

# 二十九、Executor 是真正 SQL 执行链的重要入口

进入 SqlSession 后，会进一步进入 MyBatis Executor。

常见 Executor：

```text
SimpleExecutor
ReuseExecutor
BatchExecutor
CachingExecutor
```

概念执行关系：

```text
SqlSession
    ↓
Executor
    ↓
StatementHandler
    ↓
ParameterHandler
    ↓
JDBC PreparedStatement
    ↓
Database
```

---

# 三十、SQL 参数是怎样进去的

Mapper：

```java
selectById(@Param("id") Long id)
```

XML：

```xml
WHERE id = #{id}
```

MyBatis 根据参数映射得到：

```text
#{id}
```

然后通过：

```text
ParameterHandler
```

将 Java 参数设置到 JDBC：

```text
PreparedStatement
```

概念上类似：

```java
preparedStatement.setLong(1, id);
```

---

# 三十一、查询结果怎样变成 Java 对象

MySQL 返回：

```text
ResultSet
```

MyBatis 使用：

```text
ResultSetHandler
```

根据：

```text
resultType
```

或：

```text
resultMap
```

把数据库列：

```text
id
username
register_time
```

转换到 Java 属性：

```text
id
username
registerTime
```

如果启用了：

```yaml
map-underscore-to-camel-case: true
```

那么：

```text
register_time
        ↓
registerTime
```

可以自动完成驼峰映射。

---

# 三十二、MyBatis 从启动到执行的核心对象关系

可以把最关键对象记成下面这张图：

```text
Spring Boot
    │
    ▼
MybatisAutoConfiguration
    │
    ▼
SqlSessionFactory
    │
    ▼
Configuration
    │
    ├────────────────────────────┐
    │                            │
    ▼                            ▼
MappedStatement             MapperRegistry
    │                            │
    │                            ▼
    │                      MapperProxyFactory
    │                            │
    │                            ▼
    │                         $Proxy65
    │                            │
    └──────────────┬─────────────┘
                   ▼
              MapperMethod
                   │
                   ▼
               SqlSession
                   │
                   ▼
                Executor
                   │
                   ▼
           StatementHandler
                   │
                   ▼
                 JDBC
                   │
                   ▼
                MySQL
```

---

# 三十三、这个案例应该怎么快速排查

以后再次看到：

```text
Invalid bound statement (not found)
```

可以按这个顺序检查。

## 第 1 步：看完整 Statement ID

异常会告诉你：

```text
com.xxx.UserMapper.selectById
```

把它拆开：

```text
namespace = com.xxx.UserMapper

id = selectById
```

---

## 第 2 步：检查 XML namespace

```xml
<mapper namespace="com.xxx.UserMapper">
```

必须和 Mapper 完全限定名一致。

---

## 第 3 步：检查 XML id

Java：

```java
selectById(...)
```

XML：

```xml
<select id="selectById">
```

必须一致。

---

## 第 4 步：检查 `mapper-locations`

不要把：

```text
Java package
```

和：

```text
classpath path
```

混在一起。

正确：

```yaml
classpath*:com/hs/homework/August/day21ssm/mapper/*.xml
```

---

## 第 5 步：检查 `target/classes`

确认：

```text
Day21HomeWorkUserMapper.xml
```

真的已经进入运行时 classpath。

---

## 第 6 步：重新 clean/build

修改 resources 配置后可以：

```text
停止程序
    ↓
Maven clean
    ↓
Build Project
    ↓
重新启动
```

避免旧的 `target/classes` 干扰判断。

---

# 三十四、通过不同异常快速定位 MyBatis 哪一层出错

| 异常/现象 | 更可能出错的位置 |
|---|---|
| `No qualifying bean of type UserMapper` | Mapper 接口扫描 / Spring Bean 注册 |
| `Invalid bound statement (not found)` | Mapper XML / namespace / id / mapper-locations |
| XML parse error | Mapper XML 语法 |
| `Table ... doesn't exist` | 已进入数据库，表名/数据库错误 |
| `SQLSyntaxErrorException` | SQL 已执行到 JDBC/数据库，SQL 语法错误 |
| 参数找不到，例如 `Parameter 'xxx' not found` | Mapper 参数名与 XML `#{}` 不一致 |
| 字段映射后为 null | resultMap / resultType / 驼峰映射问题 |

因此本案例：

```text
Invalid bound statement
```

说明失败点明显比：

```text
JDBC
MySQL
```

更早。

---

# 三十五、本案例的阶段定位

把整个流程画成阶段：

```text
[1] Spring Boot 启动
      √

[2] ApplicationContext 创建
      √

[3] Mapper 接口扫描
      √

[4] Mapper Proxy 创建
      √

[5] Controller / Service 创建
      √

[6] Tomcat 启动
      √

[7] HTTP 请求进入
      √

[8] Controller
      √

[9] Service
      √

[10] $Proxy65.selectById
      √

[11] MapperProxy.invoke
      √

[12] MapperMethod 创建
      √

[13] 查找 MappedStatement
      × 失败

[14] SqlSession 执行
      未进入

[15] Executor
      未进入

[16] JDBC
      未进入

[17] MySQL
      未进入
```

这就是本次异常的精确位置。

---

# 三十六、推荐断点：自己在 IDEA 里重新走一遍

如果想研究 MyBatis 底层，可以依次在下面位置打断点。

## 启动阶段

### 1. Spring Boot

```text
SpringApplication.run(...)
```

### 2. MyBatis Spring Boot 自动配置

关注：

```text
MybatisAutoConfiguration
```

以及创建：

```text
SqlSessionFactory
```

的相关方法。

### 3. SqlSessionFactory 创建

关注：

```text
SqlSessionFactoryBean
```

重点理解：

```text
buildSqlSessionFactory()
```

### 4. XML 解析

关注：

```text
XMLMapperBuilder.parse()
```

如果你的断点没有进入当前：

```text
Day21HomeWorkUserMapper.xml
```

的解析，那么就应该优先怀疑：

```text
mapper-locations
```

---

## Mapper 扫描阶段

关注：

```text
MapperScannerRegistrar
MapperScannerConfigurer
ClassPathMapperScanner
MapperFactoryBean
```

这一段负责解决：

```text
Mapper 接口如何成为 Spring Bean
```

---

## Mapper 代理创建阶段

关注：

```text
Configuration.getMapper()
MapperRegistry.getMapper()
MapperProxyFactory.newInstance()
```

最终会看到 JDK：

```text
Proxy.newProxyInstance(...)
```

这就是为什么最终日志会出现：

```text
$Proxy65
```

---

## 请求执行阶段

重点断点：

```text
MapperProxy.invoke()
MapperProxy.cachedInvoker()
MapperMethod.<init>()
MapperMethod.SqlCommand.<init>()
```

在本案例里，只要进入：

```text
MapperMethod.SqlCommand
```

就可以观察它准备寻找哪个：

```text
statementId
```

---

# 三十七、最值得记住的三个公式

## 公式 1：MappedStatement ID

```text
MappedStatement ID
=
Mapper XML namespace
+
"."
+
SQL 标签 id
```

例如：

```text
com.hs.homework.August.day21ssm.homework.mapper.Day21HomeWorkUserMapper
+
.
+
selectById
```

---

## 公式 2：Mapper 方法默认查找的 Statement ID

```text
Statement ID
=
Mapper 接口完全限定名
+
"."
+
Mapper 方法名
```

所以：

```java
Day21HomeWorkUserMapper.selectById()
```

查找：

```text
com.hs.homework.August.day21ssm.homework.mapper.Day21HomeWorkUserMapper.selectById
```

---

## 公式 3：绑定成功

```text
Mapper 接口完全限定名
=
XML namespace

并且

Mapper 方法名
=
XML statement id
```

于是：

```text
Java Mapper Method
        ⇅
MappedStatement
```

绑定成功。

---

# 三十八、最后用一句话解释本次异常

本案例并不是：

> MyBatis 没有创建 Mapper。

而是：

> MyBatis 已经成功创建 `Day21HomeWorkUserMapper` 的 JDK 动态代理；调用 `selectById()` 后进入 `MapperProxy → MapperMethod`，但由于 `Day21HomeWorkUserMapper.xml` 没有被正确加载/绑定，`Configuration` 中找不到完整 ID 为 `com.hs.homework.August.day21ssm.homework.mapper.Day21HomeWorkUserMapper.selectById` 的 `MappedStatement`，于是抛出了 `BindingException: Invalid bound statement (not found)`。

---

# 三十九、最终记忆版流程

```text
SpringApplication.run
        ↓
Spring Boot 创建容器
        ↓
读取 MyBatis 配置
        ↓
创建 SqlSessionFactory
        ↓
创建 Configuration
        ↓
mapper-locations 找 Mapper XML
        ↓
XMLMapperBuilder 解析 XML
        ↓
MappedStatement 注册进 Configuration
        ↓
@MapperScan 扫描 Mapper interface
        ↓
MapperFactoryBean
        ↓
MapperRegistry
        ↓
MapperProxyFactory
        ↓
JDK Proxy
        ↓
Mapper Proxy 注入 Service
        ↓
应用启动完成
────────────────────────────────────
HTTP Request
        ↓
DispatcherServlet
        ↓
Controller
        ↓
Service
        ↓
Mapper Proxy.selectById()
        ↓
MapperProxy.invoke()
        ↓
MapperMethod
        ↓
根据 Mapper类全限定名 + 方法名找 MappedStatement
        ↓
找到
        ↓
SqlSession
        ↓
Executor
        ↓
StatementHandler
        ↓
JDBC
        ↓
MySQL
        ↓
ResultSetHandler
        ↓
Entity
        ↓
Result
        ↓
JSON
```

本案例：

```text
找到 MappedStatement
        ↓
× 没找到
        ↓
BindingException
```

---

# 四十、案例日志证据索引

本复盘使用的关键日志位置：

| 日志范围 | 说明 |
|---|---|
| 第 13～14 行 | `Day21StartApplication` 启动、Java 17、dev profile |
| 第 15～19 行 | Tomcat 和 Spring WebApplicationContext 初始化 |
| 第 20 行 | MyBatis stdout 日志初始化 |
| 第 30～31 行 | Tomcat 与 Spring Boot 已启动完成 |
| 第 32～34 行 | 第一次请求时 DispatcherServlet 初始化 |
| 第 35～37 行 | `Invalid bound statement` 异常 |
| 第 38～44 行 | `MapperMethod → MapperProxy` 内部调用 |
| 第 45 行 | `$Proxy65.selectById`，证明进入 Mapper 动态代理 |
| 第 46 行 | Service `getById()` |
| 第 47 行 | Controller `getById()` |

---

## 总结

学习这个案例时，不要只记：

```text
mapper-locations 写错了
```

更重要的是通过这个错误把 MyBatis 的两条核心链路连起来：

```text
启动期：
XML → MappedStatement → Configuration

运行期：
Mapper Interface → Mapper Proxy → MapperMethod
```

运行时它们在：

```text
statementId
```

这个位置汇合：

```text
Mapper接口完全限定名 + "." + 方法名
```

只要这个 ID 在 `Configuration` 里找不到，最终就会得到本案例的：

```text
Invalid bound statement (not found)
```
