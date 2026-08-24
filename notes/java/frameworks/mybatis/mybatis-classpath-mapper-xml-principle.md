# MyBatis、Maven 目录结构与 Classpath 原理

## 1. Maven 项目的标准目录结构

典型 Spring Boot + MyBatis 项目：

    project
    │
    ├── src
    │   ├── main
    │   │   ├── java
    │   │   │   └── com/hs/mapper/UserMapper.java
    │   │   └── resources
    │   │       └── com/hs/mapper/UserMapper.xml
    │   │
    │   └── test
    │       └── java
    │           └── com/hs/UserMapperTest.java
    │
    └── pom.xml

这些目录是 Maven 的源码目录：

-   `src/main/java`：Java 源代码
-   `src/main/resources`：资源文件，例如 XML、YAML、properties
-   `src/test/java`：测试代码

它们不是 JVM 运行时直接访问的位置。

------------------------------------------------------------------------

# 2. Maven 编译后的结构

Maven 会把源码转换到 target 目录。

例如：

    src/main/java/com/hs/UserMapper.java

编译：

    target/classes/com/hs/UserMapper.class

资源：

    src/main/resources/application.yml

复制：

    target/classes/application.yml

最终：

    target/classes

    ├── application.yml
    └── com/hs/mapper
        ├── UserMapper.class
        └── UserMapper.xml

------------------------------------------------------------------------

# 3. 什么是 Classpath？

Classpath：

> JVM 运行时寻找 `.class` 文件和资源文件的路径集合。

它不是一个固定目录。

例如：

    classpath

    =
    target/classes
    +
    依赖 jar

如果：

    target/classes/com/hs/User.class

那么 JVM 查找：

    classpath/com/hs/User.class

------------------------------------------------------------------------

# 4. Classpath 与普通路径区别

文件路径：

``` java
new File("src/main/resources/application.yml");
```

依赖真实磁盘路径。

Classpath：

``` java
ClassLoader.getResource("application.yml");
```

寻找：

    classpath/application.yml

也就是：

    target/classes/application.yml

------------------------------------------------------------------------

# 5. 为什么 Spring Boot 使用 Classpath？

项目打包后：

    demo.jar

    └── BOOT-INF
        └── classes
            └── application.yml

原来的：

    src/main/resources

已经不存在。

但是：

    classpath

仍然存在。

因此：

``` yaml
mybatis:
  mapper-locations: classpath:mapper/*.xml
```

仍然有效。

------------------------------------------------------------------------

# 6. Mapper.xml 为什么通常和 Mapper 接口路径一致？

例如：

Java：

    src/main/java

    com/hs/mapper/UserMapper.java

XML：

    src/main/resources

    com/hs/mapper/UserMapper.xml

编译后：

    target/classes

    com/hs/mapper

    ├── UserMapper.class
    └── UserMapper.xml

二者位于同一个 classpath 命名空间。

因此 MyBatis 可以按照约定找到 XML。

------------------------------------------------------------------------

# 7. MyBatis 真正绑定机制

路径一致不是核心。

真正绑定依靠：

## namespace

XML：

``` xml
<mapper namespace="com.hs.mapper.UserMapper">
</mapper>
```

对应：

``` java
package com.hs.mapper;

public interface UserMapper {
}
```

------------------------------------------------------------------------

## id

Java：

``` java
User selectById(Long id);
```

XML：

``` xml
<select id="selectById">
</select>
```

最终：

    namespace + id

确定 SQL：

    com.hs.mapper.UserMapper.selectById

------------------------------------------------------------------------

# 8. XML 可以不和接口同路径吗？

可以。

例如：

    src/main/java

    com/hs/mapper/UserMapper.java


    src/main/resources

    mapper/UserMapper.xml

需要：

``` yaml
mybatis:
  mapper-locations: classpath:mapper/*.xml
```

告诉 MyBatis：

从 classpath 下寻找 XML。

------------------------------------------------------------------------

# 9. src/main/java、resources、test/java 是否是根？

不是。

它们是：

    Maven source directory

真正运行时：

    target/classes
    target/test-classes

才是 JVM 使用的位置。

------------------------------------------------------------------------

# 10. SpringBootTest 为什么可以找到启动类？

测试：

    src/test/java

编译：

    target/test-classes

运行时 classpath：

    target/test-classes
    +
    target/classes
    +
    dependencies

所以测试代码可以访问：

    target/classes

里的 Spring Boot 启动类。

------------------------------------------------------------------------

# 11. 整体流程

    src/main/java
            |
            ↓
        Maven compile
            |
            ↓
    target/classes


    src/main/resources
            |
            ↓
        copy resource
            |
            ↓
    target/classes


            |
            ↓

        Classpath


            |
            ↓

    UserMapper.class
    +
    UserMapper.xml


            |
            ↓

         MyBatis


    namespace:
    com.hs.mapper.UserMapper


    id:
    selectById


            |
            ↓

    执行 SQL

------------------------------------------------------------------------

# 12. 核心总结

## Maven 层

    src/main/java
    src/main/resources
    src/test/java

表示：

开发阶段源码位置。

------------------------------------------------------------------------

## JVM 层

    target/classes
    target/test-classes

表示：

运行阶段文件位置。

------------------------------------------------------------------------

## Classpath 层

    classpath

表示：

JVM 查找 class 和资源的根。

------------------------------------------------------------------------

## MyBatis 层

路径：

> 帮助找到 XML

namespace：

> 绑定 Mapper 接口

id：

> 绑定接口方法

------------------------------------------------------------------------

# 官方文档

## MyBatis 官方主页

https://mybatis.org/mybatis-3/

------------------------------------------------------------------------

## Mapper XML 文件

https://mybatis.org/mybatis-3/sqlmap-xml.html

重点：

-   mapper
-   namespace
-   mapped statements

------------------------------------------------------------------------

## MyBatis Java API

https://mybatis.org/mybatis-3/java-api.html

重点：

-   Mapper 使用
-   statement 查找

------------------------------------------------------------------------

## MyBatis-Spring Mapper

https://mybatis.org/spring/mappers.html

重点：

-   Mapper 注册
-   Spring 集成

------------------------------------------------------------------------

## MyBatis Spring Boot Starter

https://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/

重点：

-   @Mapper
-   @MapperScan
-   mapper-locations

------------------------------------------------------------------------

# 最终一句话

> Maven 将 src 下的代码和资源整理到 target；JVM 通过 classpath
> 查找运行文件；MyBatis 通过 classpath 找到 Mapper XML，再通过 namespace
> 和 id 将 XML SQL 与 Java Mapper 接口绑定。
