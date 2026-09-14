# Spring Boot `application.properties`、`application.yml` 与 `application.yaml` 的加载和优先级

> 本文根据 Spring Boot 当前 Config Data 源码整理，重点解释：配置文件为什么由 `spring-boot` 负责、YAML 扩展名如何注册、`addFirst()` 如何影响候选顺序，以及多个配置文件同时存在时应该如何理解优先级。
>
> **重要说明：** 文中的源码链接指向 Spring Boot GitHub 的 `main` 分支。源码会随版本变化；如果项目使用固定版本，应切换到对应的 tag（例如 `v3.x.x`）再核对实现。

## 1. 先给结论

1. `application.properties`、`application.yml` 和 `application.yaml` 的自动发现与加载，属于 **Spring Boot 的外部化配置 / Config Data 机制**，不属于只提供基础容器与环境抽象的 `spring-framework`。
2. YAML loader 在 `YamlPropertySourceLoader#getFileExtensions()` 中声明支持的扩展名：

   ```java
   return new String[] { "yml", "yaml" };
   ```

3. `StandardConfigDataLocationResolver#getReferencesForConfigName()` 遍历所有 `PropertySourceLoader` 和扩展名，并使用 `references.addFirst(reference)` 插入候选项。因此，当前实现中 YAML 扩展名经过反复头插后，候选顺序表现为：

   ```text
   yaml -> yml
   ```

4. Spring Boot 官方稳定保证的是：**同一位置同时存在 `.properties` 与 YAML 文件时，`.properties` 优先。**
5. `.yml` 与 `.yaml` 同时存在时，依据当前源码可以推导出 `.yml` 位于 `.yaml` 之后，因此同名配置通常表现为 `.yml` 覆盖 `.yaml`；但官方文档没有把这个相对顺序作为稳定规则公开承诺。实际项目不应依赖这种行为。

## 2. 为什么逻辑在 `spring-boot`，而不是 `spring-framework`

### 2.1 两个项目的职责不同

`spring-framework` 提供 Spring 的基础能力，例如：

- `Environment`、`PropertySource` 等环境与属性抽象；
- IoC 容器、Bean 生命周期和依赖注入；
- Spring MVC、WebFlux、AOP 等基础框架能力。

`spring-boot` 在这些基础能力之上提供应用级约定和启动流程，例如：

- 默认配置文件名 `application`；
- 自动查找 classpath 和外部目录中的配置文件；
- `spring.config.name`、`spring.config.location`、`spring.config.additional-location`；
- profile-specific 文件，例如 `application-prod.yaml`；
- `spring.config.import`；
- `application.properties` 与 YAML 的 Config Data 处理；
- 将解析后的配置加入 Spring `Environment`。

因此，问题可以分成两层：

```text
Spring Framework
    └── 提供 Environment / PropertySource 等基础抽象

Spring Boot
    └── 决定 application 文件在哪里、何时、以什么规则被查找和加载
```

### 2.2 源码目录对比

Spring Boot 相关源码主要位于：

```text
spring-boot/
└── core/
    └── spring-boot/
        └── src/main/java/org/springframework/boot/
            ├── env/
            └── context/config/
```

而不是：

```text
spring-framework/
└── spring-core/
    └── src/main/java/org/springframework/core/env/
```

后者有 `PropertySource` 等抽象，但不会定义 Spring Boot 的 `application.yml` / `application.yaml` 搜索策略。

## 3. `YamlPropertySourceLoader#getFileExtensions()`：声明 YAML 扩展名

核心类：

```text
org.springframework.boot.env.YamlPropertySourceLoader
```

当前源码位置：

[YamlPropertySourceLoader.java（GitHub）](https://github.com/spring-projects/spring-boot/blob/main/core/spring-boot/src/main/java/org/springframework/boot/env/YamlPropertySourceLoader.java)

核心方法：

```java
@Override
public String[] getFileExtensions() {
    return new String[] { "yml", "yaml" };
}
```

这段代码表达的是：

```text
YamlPropertySourceLoader 支持：

.yml
.yaml
```

它解决的是“哪些后缀属于 YAML”这个问题，但还没有完整决定多个文件最终如何覆盖。真正把这些扩展名转化为配置文件候选引用的地方，是 `StandardConfigDataLocationResolver`。

## 4. `spring.factories`：注册 `PropertySourceLoader`

Spring Boot 需要先找到可用的属性源 loader。当前注册文件是：

```text
core/spring-boot/src/main/resources/META-INF/spring.factories
```

源码链接：

[spring.factories（GitHub）](https://github.com/spring-projects/spring-boot/blob/main/core/spring-boot/src/main/resources/META-INF/spring.factories)

其中相关内容为：

```properties
org.springframework.boot.env.PropertySourceLoader=\\
org.springframework.boot.env.PropertiesPropertySourceLoader,\\
org.springframework.boot.env.YamlPropertySourceLoader
```

这表示 Spring Boot 通过 `PropertySourceLoader` 接口注册两个内置实现：

```text
PropertiesPropertySourceLoader
    └── 负责 .properties

YamlPropertySourceLoader
    ├── 负责 .yml
    └── 负责 .yaml
```

`StandardConfigDataLocationResolver` 的构造过程会通过 `SpringFactoriesLoader.loadFactories(...)` 读取这些实现：

```java
this.propertySourceLoaders = SpringFactoriesLoader.loadFactories(
        PropertySourceLoader.class,
        resourceLoader.getClassLoader());
```

相关源码：

[StandardConfigDataLocationResolver.java（GitHub）](https://github.com/spring-projects/spring-boot/blob/main/core/spring-boot/src/main/java/org/springframework/boot/context/config/StandardConfigDataLocationResolver.java)

这里要区分两种顺序：

- `spring.factories` 中 loader 的注册顺序：当前是 properties loader 在前、YAML loader 在后；
- 每个 loader 返回的扩展名顺序：YAML loader 当前返回 `yml`、`yaml`。

最终候选集合的顺序还要结合 `getReferencesForConfigName()` 中的 `addFirst()` 观察，不能只看 `spring.factories`。

## 5. `getReferencesForConfigName()` 与 `addFirst()`

当前方法位于：

```text
org.springframework.boot.context.config.StandardConfigDataLocationResolver
```

源码链接：

[StandardConfigDataLocationResolver.java（GitHub）](https://github.com/spring-projects/spring-boot/blob/main/core/spring-boot/src/main/java/org/springframework/boot/context/config/StandardConfigDataLocationResolver.java)

核心逻辑可以概括为：

```java
Deque<StandardConfigDataReference> references = new ArrayDeque<>();

for (PropertySourceLoader propertySourceLoader : this.propertySourceLoaders) {
    for (String extension : propertySourceLoader.getFileExtensions()) {
        StandardConfigDataReference reference = ...;
        if (!references.contains(reference)) {
            references.addFirst(reference);
        }
    }
}
```

### 5.1 只观察 YAML loader

YAML loader 返回：

```text
["yml", "yaml"]
```

逐步执行 `addFirst()`：

```text
初始：
[]

处理 yml：
[yml]

处理 yaml：
[yaml, yml]
```

所以，**当前实现生成的 YAML 文件候选顺序是 `yaml` 在前、`yml` 在后。**

### 5.2 把 properties loader 也放进来

按当前 `spring.factories` 的注册顺序，简化后可以表示为：

```text
loader 顺序：
PropertiesPropertySourceLoader -> YamlPropertySourceLoader

扩展名遍历顺序：
properties -> yml -> yaml

每次都 addFirst：
properties
    -> [properties]

yml
    -> [yml, properties]

yaml
    -> [yaml, yml, properties]
```

这说明 `addFirst()` 会反转“遍历过程中加入”的顺序。需要注意：这个 deque 是解析候选引用的一部分；最终 `Environment` 中的覆盖关系还由 Config Data 的贡献者树、文档顺序和属性源添加顺序共同决定。因此不能仅凭一个局部集合就概括所有配置场景。

## 6. 官方文档规定的属性源优先级

Spring Boot 官方文档说明：配置源按照特定顺序加入，**后面的属性源可以覆盖前面的属性源**。

官方文档：

[Externalized Configuration（Spring Boot 官方文档）](https://docs.spring.io/spring-boot/reference/features/external-config.html)

文档列出的主要属性源顺序包括：

```text
默认属性
    ↓
@PropertySource
    ↓
Config Data（application.properties / YAML 等）
    ↓
环境变量
    ↓
Java System Properties
    ↓
SPRING_APPLICATION_JSON
    ↓
命令行参数
    ↓
测试相关属性源
```

简化理解为：

```text
越靠后，通常优先级越高；
同名 key 出现时，后加入的值覆盖先加入的值。
```

### 6.1 Config Data 内部的官方顺序

官方文档还说明，Config Data 通常按以下层次处理：

```text
1. jar 内的 application.properties / YAML
2. jar 内的 application-{profile}.properties / YAML
3. jar 外的 application.properties / YAML
4. jar 外的 application-{profile}.properties / YAML
```

因此，外部配置通常可以覆盖打包在 jar 内的配置，profile-specific 配置可以覆盖非 profile-specific 配置。

默认搜索位置也有顺序，例如：

```text
classpath:/
classpath:/config/
当前目录 ./
当前目录 config/
config/*/
```

具体行为应以使用版本的官方文档为准，因为 Config Data 机制在 Spring Boot 2.4 发生过重要变化。

## 7. `.properties` 与 YAML 同时存在时

假设同一位置存在：

```text
application.properties
application.yml
application.yaml
```

并且三个文件都定义：

```text
app.name=...
```

Spring Boot 官方文档明确说明：**如果同一位置同时存在 `.properties` 和 YAML 格式，`.properties` 优先。**

因此可以稳定记住：

```text
同一位置：

application.properties  >  YAML 配置
```

这里的 `>` 表示覆盖优先级，不表示文件系统排序。

官方文档同时建议：整个应用尽量坚持使用一种配置格式。这样可以避免维护者误以为三个文件会按文件名排序、按编辑时间排序，或者按 IDE 中显示的顺序加载。

## 8. `.yml` 与 `.yaml` 同时存在时：当前源码推导

假设同一位置存在：

```text
application.yml
application.yaml
```

当前源码的推导过程如下：

```text
YamlPropertySourceLoader#getFileExtensions()
    -> ["yml", "yaml"]

StandardConfigDataLocationResolver#getReferencesForConfigName()
    -> 处理 yml，addFirst(yml)
    -> 处理 yaml，addFirst(yaml)

候选 deque
    -> [yaml, yml]
```

结合 Config Data 的属性源覆盖模型，可以得到当前实现下的常见表现：

```text
application.yaml
    ↓
application.yml 覆盖同名 key
```

也就是说，在完全相同的位置、相同 basename、相同 profile 且两个文件都被加载的前提下，当前源码实现通常表现为：

```text
application.yml  >  application.yaml
```

### 8.1 这不是官方稳定承诺

必须把下面两句话分开：

```text
官方规则：
同一位置的 .properties 优先于 YAML。

当前实现推导：
.yml 可能覆盖 .yaml。
```

前者是官方文档明确写出的配置规则；后者是根据当前 `getFileExtensions()` 与 `addFirst()` 推导出的实现细节。Spring Boot 官方文档没有把 `.yml` 相对于 `.yaml` 的覆盖优先级作为稳定公共契约详细说明。

因此不建议这样设计生产配置：

```text
application.yaml 作为默认值
application.yml   作为覆盖值
```

推荐做法是：

- 一个应用统一使用 `.properties` 或一种 YAML 扩展名；
- 需要环境差异时，使用明确的 profile 文件；
- 需要外部覆盖时，使用官方支持的外部配置位置或 `spring.config.additional-location`；
- 不把 `.yml` / `.yaml` 的相对顺序当成业务逻辑的一部分。

## 9. 完整调用链

以 Spring Boot 启动并加载默认 `application` 配置为例，可以按下面的链路追源码：

```text
SpringApplication.run(...)
    ↓
SpringApplication 准备 Environment
    ↓
SpringApplicationRunListeners.environmentPrepared(...)
    ↓
EnvironmentPostProcessorApplicationListener
    ↓
ConfigDataEnvironmentPostProcessor.postProcessEnvironment(...)
    ↓
ConfigDataEnvironment.processAndApply(...)
    ↓
ConfigDataImporter.resolveAndLoad(...)
    ↓
StandardConfigDataLocationResolver.resolve(...)
    ↓
getReferences(...)
    ↓
getReferencesForConfigName("application", ...)
    ↓
SpringFactoriesLoader.loadFactories(PropertySourceLoader.class, ...)
    ↓
PropertiesPropertySourceLoader / YamlPropertySourceLoader
    ↓
YamlPropertySourceLoader#getFileExtensions()
    ↓
["yml", "yaml"]
    ↓
references.addFirst(...)
    ↓
StandardConfigDataLoader.load(...)
    ↓
PropertySourceLoader.load(...)
    ↓
生成 ConfigData / PropertySource
    ↓
加入 ConfigurableEnvironment
    ↓
后续同名属性按优先级覆盖
```

Config Data 相关核心类通常集中在：

```text
org.springframework.boot.context.config
```

属性文件 loader 通常集中在：

```text
org.springframework.boot.env
```

## 10. 推荐的源码调试断点

### 10.1 断点一：确认 YAML 扩展名

文件：

```text
core/spring-boot/src/main/java/org/springframework/boot/env/YamlPropertySourceLoader.java
```

方法：

```java
YamlPropertySourceLoader#getFileExtensions()
```

观察：

```text
["yml", "yaml"]
```

### 10.2 断点二：确认 loader 如何注册

文件：

```text
core/spring-boot/src/main/resources/META-INF/spring.factories
```

观察：

```text
PropertySourceLoader=
    PropertiesPropertySourceLoader,
    YamlPropertySourceLoader
```

也可以在 `StandardConfigDataLocationResolver` 构造方法中观察：

```java
SpringFactoriesLoader.loadFactories(...)
```

### 10.3 断点三：确认 `addFirst()` 的效果

文件：

```text
core/spring-boot/src/main/java/org/springframework/boot/context/config/StandardConfigDataLocationResolver.java
```

方法：

```java
getReferencesForConfigName(...)
```

重点观察：

```java
for (PropertySourceLoader propertySourceLoader : this.propertySourceLoaders)
for (String extension : propertySourceLoader.getFileExtensions())
references.addFirst(reference)
```

调试变量建议查看：

```text
propertySourceLoaders
extension
references
```

### 10.4 断点四：确认实际文件是否被加载

继续进入：

```text
StandardConfigDataLoader#load(...)
```

并查看：

```text
ConfigDataResource
PropertySourceLoader
Resource
```

最终可以在：

```text
ConfigDataEnvironment#applyToEnvironment(...)
```

附近观察 Config Data 生成的 `PropertySource` 如何加入 `Environment`。

## 11. 调试实验建议

创建三个文件，并给同一个 key 设置不同值：

```properties
# application.properties
demo.source=properties
```

```yaml
# application.yaml
demo:
  source: yaml
```

```yaml
# application.yml
demo:
  source: yml
```

在应用中读取：

```java
@RestController
class ConfigController {

    private final Environment environment;

    ConfigController(Environment environment) {
        this.environment = environment;
    }

    @GetMapping("/config-source")
    String configSource() {
        return environment.getProperty("demo.source");
    }
}
```

然后配合断点和 Actuator 的环境信息（注意保护敏感值）观察：

```text
Environment
    ├── PropertySources 顺序
    ├── 具体 PropertySource 名称
    └── demo.source 的 origin
```

实验结果应结合当前 Spring Boot 版本解释，不要只依据文件管理器或 IDE 显示顺序下结论。

## 12. 常见误区

### 误区一：认为这是 Spring Framework 自动扫描的

Spring Framework 提供属性源抽象，但默认 `application` 文件名、YAML 扩展名和 Config Data 搜索目录是 Spring Boot 的启动配置功能。

### 误区二：认为 `yml` 和 `yaml` 是两个不同格式

二者都由 `YamlPropertySourceLoader` 处理，差别主要是文件扩展名。

### 误区三：认为 `addFirst()` 就等于最终覆盖顺序

`addFirst()` 直接说明了候选引用在该局部 deque 中的插入方式，但完整覆盖顺序还要结合 Config Data 导入树、文件位置、profile、文档顺序和 `Environment` 中的属性源顺序。

### 误区四：认为官方保证 `.yml` 一定覆盖 `.yaml`

官方明确保证的是 `.properties` 优先于 YAML。`.yml` 与 `.yaml` 的相对顺序应视为当前版本实现细节。

### 误区五：只看文件名的字典序

Spring Boot 不按操作系统文件管理器显示的字母顺序简单决定所有配置优先级。应追踪 Config Data resolver、loader 和 property source 的实际顺序。

## 13. 官方与源码参考

### Spring Boot 官方文档

- [Externalized Configuration](https://docs.spring.io/spring-boot/reference/features/external-config.html)
- [Spring Boot Reference Documentation](https://docs.spring.io/spring-boot/reference/)

### Spring Boot GitHub 源码

- [YamlPropertySourceLoader.java](https://github.com/spring-projects/spring-boot/blob/main/core/spring-boot/src/main/java/org/springframework/boot/env/YamlPropertySourceLoader.java)
- [PropertiesPropertySourceLoader.java](https://github.com/spring-projects/spring-boot/blob/main/core/spring-boot/src/main/java/org/springframework/boot/env/PropertiesPropertySourceLoader.java)
- [PropertySourceLoader.java](https://github.com/spring-projects/spring-boot/blob/main/core/spring-boot/src/main/java/org/springframework/boot/env/PropertySourceLoader.java)
- [StandardConfigDataLocationResolver.java](https://github.com/spring-projects/spring-boot/blob/main/core/spring-boot/src/main/java/org/springframework/boot/context/config/StandardConfigDataLocationResolver.java)
- [ConfigDataEnvironmentPostProcessor.java](https://github.com/spring-projects/spring-boot/blob/main/core/spring-boot/src/main/java/org/springframework/boot/context/config/ConfigDataEnvironmentPostProcessor.java)
- [ConfigDataEnvironment.java](https://github.com/spring-projects/spring-boot/blob/main/core/spring-boot/src/main/java/org/springframework/boot/context/config/ConfigDataEnvironment.java)
- [ConfigDataImporter.java](https://github.com/spring-projects/spring-boot/blob/main/core/spring-boot/src/main/java/org/springframework/boot/context/config/ConfigDataImporter.java)
- [StandardConfigDataLoader.java](https://github.com/spring-projects/spring-boot/blob/main/core/spring-boot/src/main/java/org/springframework/boot/context/config/StandardConfigDataLoader.java)
- [`META-INF/spring.factories`](https://github.com/spring-projects/spring-boot/blob/main/core/spring-boot/src/main/resources/META-INF/spring.factories)

## 14. 最终记忆版

```text
application.* 的自动加载
    -> Spring Boot，不是 Spring Framework

YAML 扩展名
    -> YamlPropertySourceLoader#getFileExtensions()
    -> ["yml", "yaml"]

loader 的发现
    -> META-INF/spring.factories
    -> SpringFactoriesLoader

候选顺序的关键操作
    -> StandardConfigDataLocationResolver#getReferencesForConfigName()
    -> references.addFirst(...)
    -> 当前 YAML 候选表现为 yaml -> yml

官方优先级
    -> 后加入的属性源覆盖先加入的
    -> 同一位置 properties 优先于 YAML

yml vs yaml
    -> 当前源码可推导为 yml 覆盖 yaml
    -> 但这是实现细节，不是官方稳定承诺
```

