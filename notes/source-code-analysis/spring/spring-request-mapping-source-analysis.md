# Spring @RequestMapping 源码分析

## 概述

`@RequestMapping` 是 Spring MVC 中用于声明 HTTP 请求映射规则的核心注解。

它通过 Java 注解、运行时反射以及 Spring MVC 的 HandlerMapping 机制，将
HTTP 请求映射到 Controller 方法。

## 核心源码

``` java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Mapping
@Reflective(ControllerMappingReflectiveProcessor.class)
public @interface RequestMapping {
}
```

## @Target

``` java
@Target({ElementType.TYPE, ElementType.METHOD})
```

表示该注解可以用于：

-   类级别（TYPE）
-   方法级别（METHOD）

类级别用于定义公共路径：

``` java
@RequestMapping("/users")
class UserController {}
```

方法级别用于定义具体接口：

``` java
@RequestMapping("/list")
public List<User> list(){}
```

最终路径会组合：

    /users/list

## @Retention

``` java
@Retention(RetentionPolicy.RUNTIME)
```

表示注解保留到运行时。

Spring 启动时通过反射读取：

``` java
method.getAnnotation(RequestMapping.class)
```

然后建立请求映射。

## @Documented

用于让 Javadoc 文档包含该注解。

## @Mapping

标识该注解属于 Spring 映射体系。

例如：

-   @GetMapping
-   @PostMapping
-   @PutMapping
-   @DeleteMapping

都是基于 @RequestMapping 扩展的组合注解。

## @Reflective

``` java
@Reflective(ControllerMappingReflectiveProcessor.class)
```

用于支持 Spring AOT 和 GraalVM Native Image。

它告诉 Spring：

该注解需要生成对应的反射处理信息。

## ControllerMappingReflectiveProcessor

负责处理 Controller 映射相关注解。

流程：

    Spring Boot启动
            |
    扫描Controller
            |
    发现@RequestMapping
            |
    解析路径和HTTP方法
            |
    注册HandlerMethod

## Spring MVC 请求流程

    HTTP请求
        |
    DispatcherServlet
        |
    HandlerMapping
        |
    HandlerMethod
        |
    Controller方法

## 设计思想

  注解          作用
  ------------- --------------------
  @Target       限制使用位置
  @Retention    保证运行时可读取
  @Documented   支持文档
  @Mapping      Spring映射体系标识
  @Reflective   支持AOT/native

## 总结

`@RequestMapping` 本质是一个运行时保留的声明式路由配置。

Spring MVC 通过扫描和反射读取它，将 URL 请求映射到具体 Controller 方法。
