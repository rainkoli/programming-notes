# Java 自定义工具类（Utility Library）设计与发布总结

## 1. 什么是工具类？

工具类（Utility Class）：

> 提供一组通用方法，被多个地方复用的类。

常见类型：

-   字符串处理
-   日期处理
-   文件操作
-   线程操作
-   集合操作

类似 Java 生态中的：

-   Apache Commons
-   Spring StringUtils
-   Hutool

------------------------------------------------------------------------

# 2. 工具类设计原则

## 2.1 使用 static 方法

工具类通常不需要创建对象。

``` java
public static boolean isEmpty(String str) {
    return str == null || str.isEmpty();
}
```

使用：

``` java
StringUtils.isEmpty("");
```

而不是：

``` java
new StringUtils().isEmpty("");
```

原因：

工具类主要提供行为，不保存对象状态。

------------------------------------------------------------------------

## 2.2 构造方法私有化

推荐：

``` java
public final class StringUtils {

    private StringUtils() {

    }

}
```

作用：

防止：

``` java
new StringUtils();
```

创建没有意义的对象。

------------------------------------------------------------------------

## 2.3 类声明为 final

例如：

``` java
public final class StringUtils
```

原因：

工具类一般不需要继承。

------------------------------------------------------------------------

# 3. 工具类项目设计

不要把所有工具类放在一个包：

    utils
     |
     ├── StringUtils
     ├── DateUtils
     ├── ThreadUtils
     └── FileUtils

推荐按照功能分类：

    java-utils

    com.rainkoli.commons

    ├── string
    │    └── StringUtils.java
    │
    ├── date
    │    └── DateUtils.java
    │
    ├── thread
    │    └── ThreadUtils.java
    │
    ├── io
    │    └── FileUtils.java
    │
    └── collection
         └── CollectionUtils.java

------------------------------------------------------------------------

# 4. commons 的含义

commons：

表示：

> 公共的、共享的组件。

例如：

    common

    ├── utils
    ├── constants
    ├── exception
    └── model

企业项目中：

    java-homework

    ├── spring-demo
    ├── redis-demo
    └── hs-common
           |
           ├── utils
           ├── constants
           └── exception

多个模块共同依赖 common。

------------------------------------------------------------------------

# 5. 创建 Maven 工具库

IDEA：

    New Project
        ↓
    Java
        ↓
    Maven

不要选择：

-   Spring Boot

原因：

工具类不需要：

-   Spring Container
-   Bean
-   Controller
-   Service

也不需要 Maven Archetype。

普通 Maven Java 项目即可。

------------------------------------------------------------------------

# 6. Maven 坐标

pom.xml：

``` xml
<groupId>com.rainkoli</groupId>

<artifactId>java-utils</artifactId>

<version>1.0.0</version>
```

三个组成：

    groupId
    +
    artifactId
    +
    version

形成 Maven 坐标：

    com.rainkoli:java-utils:1.0.0

------------------------------------------------------------------------

# 7. 示例工具类

## StringUtils

路径：

    com.rainkoli.commons.string.StringUtils

示例：

``` java
public final class StringUtils {

    private StringUtils(){

    }

    public static boolean isEmpty(String str){

        return str == null || str.isEmpty();

    }

}
```

------------------------------------------------------------------------

## ThreadUtils

路径：

    com.rainkoli.commons.thread.ThreadUtils

示例：

``` java
public final class ThreadUtils {

    private ThreadUtils(){

    }

    public static String getCurrentThreadName(){

        return Thread.currentThread().getName();

    }

}
```

------------------------------------------------------------------------

# 8. Maven 打包流程

## compile

    mvn compile

作用：

    .java
     ↓
    .class

输出：

    target/classes

------------------------------------------------------------------------

## package

    mvn package

作用：

    .class
     ↓
    .jar

生成：

    target/java-utils-1.0.0.jar

------------------------------------------------------------------------

## install

    mvn clean install

作用：

除了生成 jar：

还安装到本地 Maven 仓库：

    .m2/repository

    └── com
        └── rainkoli
            └── java-utils
                └── 1.0.0
                    |
                    ├── java-utils-1.0.0.jar
                    └── java-utils-1.0.0.pom

------------------------------------------------------------------------

# 9. 在其他项目中引用

pom.xml：

``` xml
<dependency>

    <groupId>com.rainkoli</groupId>

    <artifactId>java-utils</artifactId>

    <version>1.0.0</version>

</dependency>
```

使用：

``` java
import com.rainkoli.commons.string.StringUtils;

StringUtils.isEmpty("");
```

------------------------------------------------------------------------

# 10. jar 分发方式

## 方式一：Maven install（推荐）

自己开发的 Maven 项目：

    mvn clean install

------------------------------------------------------------------------

## 方式二：install-file

适用于：

别人只给你一个 jar：

    java-utils-1.0.0.jar

没有 pom。

执行：

``` bash
mvn install:install-file \
-Dfile=java-utils-1.0.0.jar \
-DgroupId=com.rainkoli \
-DartifactId=java-utils \
-Dversion=1.0.0 \
-Dpackaging=jar
```

------------------------------------------------------------------------

## 方式三：公司私服

企业通常：

    开发
     ↓
    mvn deploy
     ↓
    Nexus / Artifactory
     ↓
    dependency

------------------------------------------------------------------------

# 11. Git Commit 规范

格式：

    type(scope): description

例如：

第一次创建工具库：

    feat(utils): add basic utility classes

含义：

-   feat：新增功能
-   utils：影响范围
-   description：描述

其他：

新增 JSON：

    feat(json): add json utility support

修复：

    fix(string): fix empty string check logic

重构：

    refactor(utils): reorganize utility package

------------------------------------------------------------------------

# 12. 完整生命周期

    编写工具类

    ↓

    Maven 项目

    ↓

    mvn compile

    ↓

    .class

    ↓

    mvn package

    ↓

    .jar

    ↓

    mvn install

    ↓

    本地 Maven 仓库

    ↓

    dependency 引入

    ↓

    其他项目使用

------------------------------------------------------------------------

# 13. 企业项目结构示例

    project

    ├── common
    │
    │   ├── common-utils
    │   ├── common-exception
    │   └── common-constant
    │
    ├── user-service
    ├── order-service
    └── payment-service

------------------------------------------------------------------------

通过创建 `java-utils`，实际上完成了一个简化版 Java 第三方库开发流程：

    Java 类设计
            ↓
    Maven 项目
            ↓
    Jar 打包
            ↓
    Maven 仓库
            ↓
    Dependency 引用

这也是理解 Spring Boot、MyBatis 等框架依赖管理的基础。
