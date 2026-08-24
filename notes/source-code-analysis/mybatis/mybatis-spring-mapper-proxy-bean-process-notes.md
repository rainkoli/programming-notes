# Spring Boot + MyBatis-Spring Mapper 注册与代理 Bean 创建流程笔记

## 1. 核心问题

在 Spring Boot 项目中：

``` java
@Mapper
public interface Day20UserMapper {
    List<User> select();
    User selectById(Integer id);
}
```

如果去掉 `@Mapper`，同时没有配置：

``` java
@MapperScan
```

那么：

-   Spring 容器无法找到 `Day20UserMapper` Bean
-   MyBatis-Spring 不会创建 Mapper 代理对象
-   依赖该 Mapper 的 Service 注入失败

错误：

    No qualifying bean of type 'Day20UserMapper' available

------------------------------------------------------------------------

# 2. @SpringBootApplication 与组件扫描

`@SpringBootApplication` 等价于：

``` java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
```

其中：

``` java
@ComponentScan
```

负责扫描 Spring 组件：

-   @Component
-   @Service
-   @Controller
-   @Repository

例如：

    day20ssm

    ├── controller
    │       Day20UserController

    ├── service
    │       Day20UserService

    └── mapper
            Day20UserMapper

扫描结果：

    Controller  -> Spring Bean
    Service     -> Spring Bean
    Mapper接口  -> 不处理

原因：

Mapper 是接口：

``` java
public interface Day20UserMapper
```

不是普通 class，Spring 不知道如何实例化。

------------------------------------------------------------------------

# 3. @Mapper 和 @MapperScan

## @Mapper

作用：

标记单个 Mapper 接口：

``` java
@Mapper
public interface UserMapper {

}
```

告诉 MyBatis-Spring：

> 这个接口需要生成代理对象并注册到 Spring。

------------------------------------------------------------------------

## @MapperScan

作用：

批量扫描 Mapper：

``` java
@MapperScan(
    "com.hs.mapper"
)
```

扫描 package 下的 Mapper 接口。

它不是根据文件名：

    xxxMapper.java

判断。

不是：

    名字包含 Mapper

而是：

    扫描 class 元数据
    判断接口
    注册 MapperFactoryBean

------------------------------------------------------------------------

# 4. @MapperScan 内部流程

## 第一阶段：解析 @MapperScan

入口：

    org.mybatis.spring.annotation.MapperScannerRegistrar

实现：

``` java
ImportBeanDefinitionRegistrar
```

关键方法：

``` java
registerBeanDefinitions()
```

作用：

注册：

    MapperScannerConfigurer

------------------------------------------------------------------------

流程：

    @MapperScan
          |
          v
    MapperScannerRegistrar
          |
          v
    注册 MapperScannerConfigurer

------------------------------------------------------------------------

# 5. MapperScannerConfigurer

类：

    org.mybatis.spring.mapper.MapperScannerConfigurer

实现：

    BeanDefinitionRegistryPostProcessor

作用：

在 Spring Bean 创建之前，扫描 Mapper。

流程：

    Spring启动

          |
          v

    执行 postProcessBeanDefinitionRegistry()

          |
          v

    ClassPathMapperScanner 扫描 Mapper

------------------------------------------------------------------------

# 6. ClassPathMapperScanner

核心：

    org.mybatis.spring.mapper.ClassPathMapperScanner

作用：

扫描指定 package。

例如：

    com.hs.homework.day20ssm.mapper

找到：

    Day20UserMapper.class

然后修改 BeanDefinition。

普通 Spring：

    Class
     |
     v
    BeanDefinition
     |
     v
    Object

MyBatis：

    Mapper Interface
     |
     v
    MapperFactoryBean
     |
     v
    Proxy Object

------------------------------------------------------------------------

# 7. MapperFactoryBean

注册之后：

Spring 容器中不是：

    Day20UserMapper 实例

因为接口不能实例化。

而是：

    MapperFactoryBean

MyBatis-Spring 使用 FactoryBean 创建真正对象。

核心：

``` java
getObject()
```

调用：

    SqlSession.getMapper()

------------------------------------------------------------------------

# 8. Mapper Proxy 创建流程

完整流程：

    MapperFactoryBean.getObject()

            |

            v

    SqlSession.getMapper()

            |

            v

    Configuration.getMapper()

            |

            v

    MapperRegistry.getMapper()

            |

            v

    MapperProxyFactory.newInstance(SqlSession)

            |

            v

    new MapperProxy()

            |

            v

    MapperProxyFactory.newInstance(MapperProxy)

            |

            v

    Proxy.newProxyInstance()

            |

            v

    JDK Dynamic Proxy

            |

            v

    Day20UserMapper代理对象

------------------------------------------------------------------------

# 9. MapperRegistry

MyBatis 中：

    org.apache.ibatis.binding.MapperRegistry

保存：

``` java
Map<Class<?>, MapperProxyFactory<?>>
```

结构：

    Day20UserMapper.class

            |

            v

    MapperProxyFactory

获取 Mapper：

``` java
getMapper()
```

调用：

``` java
mapperProxyFactory.newInstance(sqlSession)
```

------------------------------------------------------------------------

# 10. MapperProxyFactory

MyBatis 3.5.19：

关键方法：

## newInstance(SqlSession)

创建：

``` java
MapperProxy
```

代码逻辑：

``` java
MapperProxy mapperProxy =
        new MapperProxy<>(
            sqlSession,
            mapperInterface,
            methodCache
        );
```

------------------------------------------------------------------------

## newInstance(MapperProxy)

真正创建代理：

``` java
Proxy.newProxyInstance(
    mapperInterface.getClassLoader(),
    new Class[]{mapperInterface},
    mapperProxy
);
```

结果：

    Day20UserMapper接口

            +

    MapperProxy InvocationHandler

            ↓

    JDK代理对象

------------------------------------------------------------------------

# 11. 为什么去掉 @Mapper 会失败？

启动过程：

    Spring启动

     |
     v

    @ComponentScan

     |
     +-- Controller Bean
     |
     +-- Service Bean
     |
     X-- Day20UserMapper
            (没有@Mapper)

     |
     v

    创建 Service

     |
     v

    需要 Day20UserMapper

     |
     v

    BeanFactory 查询

     |
     v

    不存在

     |
     v

    UnsatisfiedDependencyException

------------------------------------------------------------------------

# 12. Debug 断点位置

推荐顺序：

## 1. MapperScannerRegistrar

方法：

    registerBeanDefinitions()

观察：

    @MapperScan

如何注册扫描器。

------------------------------------------------------------------------

## 2. MapperScannerConfigurer

方法：

    postProcessBeanDefinitionRegistry()

观察：

扫描开始。

------------------------------------------------------------------------

## 3. ClassPathMapperScanner

搜索：

    MapperFactoryBean.class

观察：

接口如何转换为：

    MapperFactoryBean BeanDefinition

------------------------------------------------------------------------

## 4. MapperFactoryBean

方法：

    getObject()

观察：

Spring Bean 创建阶段进入 MyBatis。

------------------------------------------------------------------------

## 5. MapperRegistry

方法：

    getMapper()

观察：

获取 MapperProxyFactory。

------------------------------------------------------------------------

## 6. MapperProxyFactory

方法：

    newInstance(SqlSession)

创建：

    MapperProxy

------------------------------------------------------------------------

## 7. MapperProxyFactory

方法：

    newInstance(MapperProxy)

观察：

``` java
Proxy.newProxyInstance()
```

生成最终代理对象。

------------------------------------------------------------------------

# 13. 总结

一句话：

> Spring 的 ComponentScan 负责发现普通 Bean，而 MyBatis-Spring 通过
> @Mapper 或 @MapperScan 发现 Mapper 接口，将其注册为
> MapperFactoryBean，最终利用 MyBatis 的 MapperProxyFactory 创建 JDK
> 动态代理对象，并把该代理对象作为 Spring Bean 注入业务层。

官方参考：

-   MyBatis-Spring Mapper 注册机制
-   @MapperScan
-   MapperFactoryBean
