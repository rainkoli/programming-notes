# How `@SpringBootTest` Finds `Day19StartApplication` from `src/main/java` and Builds the Spring Context

## 1. Background

A common Spring Boot project structure:

    src
    ├── main
    │   └── java
    │       └── com.hs.homework.August.day19mybatis
    │           ├── Day19StartApplication.java
    │           ├── entity
    │           ├── mapper
    │           └── service
    │
    └── test
        └── java
            └── com.hs.homework.August.day19mybatis.homework
                └── Day19DepartmentTest.java

The startup class:

``` java
package com.hs.homework.August.day19mybatis;

@SpringBootApplication
public class Day19StartApplication {

    public static void main(String[] args) {
        SpringApplication.run(Day19StartApplication.class, args);
    }
}
```

The test class:

``` java
package com.hs.homework.August.day19mybatis.homework;

@SpringBootTest
public class Day19DepartmentTest {

    @Autowired
    Day19DepartmentMapper mapper;

}
```

Question:

Why can a test class under `src/test/java` find the Spring Boot startup
class under `src/main/java` and use it for component scanning?

------------------------------------------------------------------------

# 2. The Short Answer

The complete process contains three layers:

    Maven build system
            |
            v
    JVM ClassPath
            |
            v
    Spring Boot Test mechanism
            |
            v
    @SpringBootApplication
            |
            v
    Component Scan + Auto Configuration

The key idea:

> `src/test/java` does not directly scan `src/main/java`. Maven compiles
> both directories and puts them into the same test runtime classpath.

------------------------------------------------------------------------

# 3. Understanding Maven Source Directories

## 3.1 Source folders are only development organization

Maven defines:

    src/main/java

as application source code.

    src/test/java

as test source code.

They are not runtime boundaries.

------------------------------------------------------------------------

## 3.2 Compilation result

Main code:

    src/main/java
            |
            | javac
            v
    target/classes

Example:

    target/classes/

    com/hs/homework/August/day19mybatis/
        Day19StartApplication.class

------------------------------------------------------------------------

Test code:

    src/test/java
            |
            | javac
            v
    target/test-classes

Example:

    target/test-classes/

    com/hs/homework/August/day19mybatis/homework/
        Day19DepartmentTest.class

------------------------------------------------------------------------

# 4. Test Runtime ClassPath

When Maven executes tests, the JVM receives:

    ClassPath =
        target/test-classes
        +
        target/classes
        +
        dependency libraries

Therefore:

    Day19DepartmentTest.class

can load:

    Day19StartApplication.class

because both classes exist in the same runtime environment.

The relationship is:

    test code
        |
        | depends on
        v
    main code

The opposite is not recommended:

    main code
        X
    test code

because production applications should not depend on test code.

------------------------------------------------------------------------

# 5. How Does SpringBootTest Find the Startup Class?

The test only contains:

``` java
@SpringBootTest
```

It does not contain:

``` java
@SpringBootTest(classes = Day19StartApplication.class)
```

So Spring Boot needs to automatically locate the configuration class.

According to Spring Boot documentation:

`@SpringBootTest` searches for a `@SpringBootConfiguration`
automatically if no configuration class is explicitly specified.

Official documentation:

https://docs.spring.io/spring-boot/reference/testing/spring-boot-applications.html

------------------------------------------------------------------------

# 6. The Searching Algorithm

The test class package:

    com.hs.homework.August.day19mybatis.homework

Spring Boot starts searching here.

Step 1:

    com.hs.homework.August.day19mybatis.homework

Does this package contain:

    @SpringBootConfiguration

No.

------------------------------------------------------------------------

Step 2:

Move to parent package:

    com.hs.homework.August.day19mybatis

Find:

    Day19StartApplication

which contains:

``` java
@SpringBootApplication
```

------------------------------------------------------------------------

Step 3:

Spring Boot recognizes:

    @SpringBootApplication

as a valid configuration source.

Therefore:

    Day19StartApplication

becomes the ApplicationContext startup configuration.

------------------------------------------------------------------------

# 7. Relationship Between Packages and Directories

Important:

Java packages and source folders are different concepts.

This:

    src/test/java

does not matter to Spring component scanning.

Spring cares about:

``` java
package com.hs.homework.August.day19mybatis.homework;
```

not:

    src/test/java

The package relationship is:

    com.hs.homework.August.day19mybatis

            |
            +---- homework

                  |
                  +---- Day19DepartmentTest

The startup class is the parent package of the test class.

That is why automatic discovery works.

------------------------------------------------------------------------

# 8. What Does @SpringBootApplication Actually Contain?

`@SpringBootApplication` is a composite annotation.

Conceptually:

``` java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
public @interface SpringBootApplication
```

Official API:

https://docs.spring.io/spring-boot/api/java/org/springframework/boot/autoconfigure/SpringBootApplication.html

------------------------------------------------------------------------

# 9. Three Important Parts

## 9.1 @SpringBootConfiguration

Purpose:

Defines the primary Spring Boot configuration class.

It tells Spring Boot:

"This class can be used to create the ApplicationContext."

------------------------------------------------------------------------

## 9.2 @EnableAutoConfiguration

Purpose:

Enables automatic configuration.

Example:

MyBatis:

    Spring Boot
        |
        v
    MyBatisAutoConfiguration
        |
        v
    SqlSessionFactory
        |
        v
    Mapper registration

------------------------------------------------------------------------

## 9.3 @ComponentScan

Purpose:

Find Spring components.

Default scanning rule:

    package of startup class
    +
    all child packages

Example:

Startup class:

    com.hs.homework.August.day19mybatis

Scan range:

    com.hs.homework.August.day19mybatis.*

Therefore:

    entity
    mapper
    service
    controller

can be discovered.

------------------------------------------------------------------------

# 10. How MyBatis Mapper Becomes a Bean

A common misunderstanding:

    @ComponentScan
            |
            v
    find interface
            |
            v
    create Bean

This is incorrect.

Interfaces cannot become normal Spring Beans automatically.

The real process:

    Spring Boot starts
            |
            v
    MyBatis Auto Configuration
            |
            v
    Mapper Scanner
            |
            v
    Create Mapper Proxy object
            |
            v
    Register proxy as Spring Bean

Then:

``` java
@Autowired
Day19DepartmentMapper mapper;
```

works.

------------------------------------------------------------------------

# 11. Complete Execution Flow

When running:

    Day19DepartmentTest

the process is:

    1. IDEA/Maven creates test runtime environment

            |

    2. JVM loads Day19DepartmentTest.class

            |

    3. Spring finds @SpringBootTest

            |

    4. SpringBootTest starts ApplicationContext

            |

    5. Search for @SpringBootConfiguration

            |

    6. Find Day19StartApplication

            |

    7. Read @SpringBootApplication

            |

    8. Enable:
           - ComponentScan
           - AutoConfiguration

            |

    9. Scan Spring Beans

            |

    10. MyBatis registers Mapper Proxy Beans

            |

    11. Dependency injection happens

            |

    12. Test method executes

------------------------------------------------------------------------

# 12. Two Directions to Remember

## 12.1 Finding startup class

Direction:

    Test package

            ^
            |
            |
    Parent packages

            ^
            |
    @SpringBootConfiguration

Spring searches upward.

------------------------------------------------------------------------

## 12.2 Component scanning

Direction:

    Startup package

            |
            v

    Child packages

            |
            v

    @Component
    @Service
    @Repository
    @Mapper

Spring scans downward.

------------------------------------------------------------------------

# 13. What Happens If Package Structure Is Wrong?

Example:

Startup:

    com.hs.homework.August.day19mybatis

Test:

    com.hs.test

Spring searches:

    com.hs.test
            |
            v
    com.hs
            |
            v
    com

It cannot find:

    com.hs.homework.August.day19mybatis.Day19StartApplication

The result:

    Unable to find a @SpringBootConfiguration

Solution:

Specify explicitly:

``` java
@SpringBootTest(classes = Day19StartApplication.class)
```

------------------------------------------------------------------------

# 14. Final Mental Model

Remember these layers:

    Maven

    Responsible for:
    Where code is compiled


            |

    JVM ClassPath

    Responsible for:
    Which classes can be loaded


            |

    @SpringBootTest

    Responsible for:
    Which Spring Context starts


            |

    @SpringBootApplication

    Responsible for:
    How Spring Context is configured


            |

    @ComponentScan

    Responsible for:
    Which Beans are discovered


            |

    @EnableAutoConfiguration

    Responsible for:
    How frameworks are configured

------------------------------------------------------------------------

# 15. Official Documentation

## Spring Boot Testing

https://docs.spring.io/spring-boot/reference/testing/spring-boot-applications.html

## SpringBootApplication API

https://docs.spring.io/spring-boot/api/java/org/springframework/boot/autoconfigure/SpringBootApplication.html

## Maven Standard Directory Layout

https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html

------------------------------------------------------------------------

# Final Summary

`Day19DepartmentTest` can use `Day19StartApplication` because:

1.  Maven compiles main and test code separately.
2.  During test execution, both compiled outputs are added to the JVM
    classpath.
3.  `@SpringBootTest` searches upward from the test package for
    `@SpringBootConfiguration`.
4.  `@SpringBootApplication` provides the configuration entry point.
5.  Spring creates the ApplicationContext.
6.  Component scanning and MyBatis auto configuration register required
    Beans.

The test class does not "cross folders".

Instead:

    Maven combines classes
            +
    Spring Boot discovers configuration
            +
    Spring creates the container

That is the complete mechanism.
