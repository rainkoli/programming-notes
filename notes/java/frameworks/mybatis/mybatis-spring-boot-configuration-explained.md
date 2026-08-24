# MyBatis Spring Boot Configuration Explained

------------------------------------------------------------------------

# MyBatis Configuration Overview

Example:

``` yaml
mybatis:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: com.hs.homework.August.day20ssm.entity
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    lazy-loading-enabled: true
```

These configurations control different parts of MyBatis:

-   Mapper XML file scanning
-   Java class alias registration
-   SQL logging
-   Lazy loading behavior

Official documentation: - MyBatis Configuration:
https://mybatis.org/mybatis-3/configuration.html

------------------------------------------------------------------------

# 1. mapper-locations (Mapper XML Location)

## Purpose

`mapper-locations` tells Spring Boot where MyBatis Mapper XML files are
located.

Example:

``` yaml
mybatis:
  mapper-locations: classpath:mapper/*.xml
```

Project structure:

    src
     └── main
         ├── java
         └── resources
              └── mapper
                   ├── UserMapper.xml
                   └── ArticleMapper.xml

`classpath:` means:

    src/main/resources

after compilation:

    target/classes

Therefore:

    classpath:mapper/*.xml

means:

    target/classes/mapper/*.xml

------------------------------------------------------------------------

# Runtime Process

Startup:

    Spring Boot
          |
          v
    MyBatis Auto Configuration
          |
          v
    SqlSessionFactory
          |
          v
    Load Mapper XML
          |
          v
    Register SQL statements

Example:

Mapper interface:

``` java
public interface ArticleMapper {
    Article findById(Integer id);
}
```

XML:

``` xml
<select id="findById">
    select * from article where id=#{id}
</select>
```

MyBatis connects:

    ArticleMapper.findById()
            |
            v
    namespace + id
            |
            v
    SQL statement

------------------------------------------------------------------------

# 2. type-aliases-package (Type Alias Package)

## Purpose

Register entity classes as aliases.

Without this configuration:

``` xml
<select resultType="com.hs.homework.August.day20ssm.entity.Article">
</select>
```

With:

``` yaml
type-aliases-package:
  com.hs.homework.August.day20ssm.entity
```

You can write:

``` xml
<select resultType="Article">
</select>
```

------------------------------------------------------------------------

## Internal Process

Spring Boot starts:

    SqlSessionFactoryBean
            |
            v
    TypeAliasRegistry
            |
            v
    Scan package
            |
            v
    Register classes

Example:

    Article.class
    
    becomes:
    
    article -> Article.class

MyBatis automatically converts the first letter to lowercase.

------------------------------------------------------------------------

# 3. log-impl (Logging Implementation)

Configuration:

``` yaml
configuration:
  log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

Purpose:

Enable SQL output.

Without logging:

    No SQL information

With StdOutImpl:

    ==> Preparing:
    select * from article where id=?
    
    ==> Parameters:
    1(Integer)
    
    <== Total:
    1

------------------------------------------------------------------------

## Common Implementations

### StdOutImpl

    org.apache.ibatis.logging.stdout.StdOutImpl

Advantages:

-   Simple
-   Good for learning

Disadvantages:

-   Not recommended for production

------------------------------------------------------------------------

### Slf4jImpl

    org.apache.ibatis.logging.slf4j.Slf4jImpl

Usually used in Spring Boot production projects.

Flow:

    MyBatis
       |
       v
    SLF4J
       |
       v
    Logback

------------------------------------------------------------------------

# 4. lazy-loading-enabled (Lazy Loading)

Configuration:

``` yaml
configuration:
  lazy-loading-enabled: true
```

Purpose:

Enable delayed loading of associated objects.

Official concept:

Lazy loading means related objects are loaded only when they are
accessed.

------------------------------------------------------------------------

# Immediate Loading vs Lazy Loading

## Immediate Loading

Example:

``` java
User user = userMapper.findById(1);
```

SQL:

    select * from user where id=1
    
    select * from order where user_id=1

Even if:

``` java
user.getName();
```

does not need orders.

------------------------------------------------------------------------

## Lazy Loading

First:

    select * from user where id=1

Order is not loaded.

Later:

``` java
user.getOrder();
```

Triggers:

    select * from order where user_id=1

------------------------------------------------------------------------

# Internal Mechanism

MyBatis uses proxy objects.

Process:

    Query User
        |
        v
    Create proxy object
        |
        v
    Return User proxy
        |
        v
    Call getOrder()
        |
        v
    Execute SQL
        |
        v
    Load association

------------------------------------------------------------------------

# Configuration Relationship

    mapper-locations
            |
            v
    Find XML files


    type-aliases-package
            |
            v
    Find Java entity classes


    log-impl
            |
            v
    Show SQL execution


    lazy-loading-enabled
            |
            v
    Control loading timing

------------------------------------------------------------------------

# Important Notes

## mapUnderscoreToCamelCase

This configuration is different from the above.

It solves:

    Database column
            |
            v
    Java property

Example:

    create_time
    
            |
    
    createTime

It does not automatically change SQL written by developers.

Example:

``` xml
create_time = #{createTime}
```

The left side is database column name. The right side is Java property
name.

------------------------------------------------------------------------

# Official Documentation Links

## MyBatis Official Website

https://mybatis.org/mybatis-3/

## Configuration Documentation

https://mybatis.org/mybatis-3/configuration.html

## Mapper XML Documentation

https://mybatis.org/mybatis-3/sqlmap-xml.html
