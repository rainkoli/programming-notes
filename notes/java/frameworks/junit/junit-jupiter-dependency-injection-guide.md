# JUnit Jupiter 构造器和方法的依赖注入 / Dependency Injection for Constructors and Methods

> 本文整理并扩展原资料中的 JUnit Jupiter **2.13 Dependency Injection for Constructors and Methods**，包含中文翻译、概念解释、版本差异、可运行的 Java demo、常见错误，以及官方文档链接。

## 版本说明 / Version notes

- “2.13” 这一章节编号出现在 JUnit 5.9.x 和 5.10.x 文档中；后续 JUnit 5 版本调整了章节编号，JUnit 6 则改为按主题组织页面。因此应按标题查找，而不要只依赖章节号。
- 原资料可对应 [JUnit 5.10.0 User Guide — 2.13](https://docs.junit.org/5.10.0/user-guide/index.html#writing-tests-dependency-injection)。
- 本文的可运行 demo 使用 **Java 17 + JUnit 5.14.4**，以便使用较新的 JUnit 5 行为与文档；全部核心 demo 已通过 `mvn test` 验证。
- [Current documentation](https://docs.junit.org/current/writing-tests/dependency-injection-for-constructors-and-methods.html) 会随当前稳定版本更新，可能已经指向 JUnit 6。迁移时请同时阅读所用版本的固定链接。

一个值得特别注意的差异：JUnit 5.10 的官方示例中，构造器收到的 `TestInfo` 描述测试类或测试容器；在 JUnit 5.14 默认的 `PER_METHOD` 生命周期下，它可以描述当前测试调用。不要编写依赖这一差异的脆弱断言，除非项目已经锁定 JUnit 版本。

## Corrected request / 语法修正版

> Could you create a detailed Markdown file containing everything you mentioned above, including links to the official documentation? Please use a kebab-case filename. The content may be a mix of Chinese and English, but technical terms should be provided in both languages. Also, please correct my grammar and include the demo examples mentioned above.

主要修改：

- `The content can be ...` 改为更自然的 `The content may be ...`。
- `technical terms should include both ... versions` 改为 `technical terms should be provided in both languages`。
- `examples of demo mentioned` 改为 `the demo examples mentioned above`。

---

## 1. 核心结论 / Core idea

JUnit Jupiter 允许测试类构造器和由 JUnit 调用的方法声明参数。这使构造器依赖注入（constructor dependency injection）和方法参数注入（method parameter injection）成为可能。

但“允许声明参数”不等于“JUnit 能自动提供任意对象”。每个参数都必须由以下某种机制提供：

1. 已注册的参数解析器（registered `ParameterResolver`）；或
2. 参数化测试（parameterized test）的参数源（argument source），例如 `@ValueSource`、`@CsvSource`。

因此，下面的测试仍然会失败：

```java
@Test
void test(String value) {
    // No resolver or argument source supplies value.
}
```

JUnit 不知道这个 `String` 应该从哪里取得，通常会抛出参数解析异常（`ParameterResolutionException`）。

### JUnit 4 与 JUnit Jupiter 的区别

JUnit 4 的标准 Runner 要求普通 `@Test` 方法没有参数：

```java
public class JUnit4StyleTest {

    @org.junit.Test
    public void testSomething() {
        // Standard JUnit 4 test method: no parameters.
    }
}
```

标准 JUnit 4 Runner 不接受下面的普通测试方法：

```java
@org.junit.Test
public void testSomething(String value) {
    // Invalid for the standard JUnit 4 runner.
}
```

不过，“JUnit 4 完全不支持任何参数”并不准确。专用 Runner，例如 `Parameterized` 或自定义 Runner，可以提供某些形式的参数化；JUnit 4 缺少的是 JUnit Jupiter 这种通用的参数解析扩展模型（general parameter-resolution extension model）。

JUnit Jupiter 的典型写法如下：

```java
@Test
void testSomething(TestInfo testInfo) {
    System.out.println(testInfo.getDisplayName());
}
```

这里的 `TestInfo` 由 JUnit 内置扩展提供，测试代码不需要执行 `new TestInfo(...)`。

---

## 2. 术语表 / Terminology

| 中文 | English | 含义 |
|---|---|---|
| 依赖注入 | Dependency Injection, DI | 调用方不主动创建或查找依赖，而由框架提供所需值。 |
| 构造器注入 | Constructor Injection | 通过测试类构造器参数提供依赖。 |
| 方法参数注入 | Method Parameter Injection | 通过测试方法或生命周期方法参数提供依赖。 |
| 参数解析器 | Parameter Resolver | 实现 `ParameterResolver`、负责判断并解析参数的扩展。 |
| 扩展模型 | Extension Model | JUnit Jupiter 提供的可插拔扩展机制。 |
| 扩展注册 | Extension Registration | 使用 `@ExtendWith` 等机制启用扩展。 |
| 生命周期方法 | Lifecycle Method | `@BeforeAll`、`@AfterAll`、`@BeforeEach`、`@AfterEach` 方法。 |
| 测试容器 | Test Container | 测试类或其他包含测试的结构。 |
| 测试调用 | Test Invocation | 某个测试方法的一次实际执行。 |
| 显示名称 | Display Name | IDE 或报告中显示的测试名称，可通过 `@DisplayName` 自定义。 |
| 报告条目 | Reporting Entry | 通过 `TestReporter` 发布给测试报告基础设施的结构化信息。 |
| 参数解析冲突 | Parameter Resolution Conflict | 多个解析器同时声称支持同一参数。 |

---

## 3. `ParameterResolver` 如何工作 / How parameter resolution works

[`ParameterResolver`](https://docs.junit.org/5.14.4/api/org.junit.jupiter.api/org/junit/jupiter/api/extension/ParameterResolver.html) 是 JUnit Jupiter 扩展 API。它有两个核心方法：

```java
boolean supportsParameter(
        ParameterContext parameterContext,
        ExtensionContext extensionContext);

Object resolveParameter(
        ParameterContext parameterContext,
        ExtensionContext extensionContext);
```

执行流程（execution flow）如下：

```text
JUnit 准备调用构造器或方法
        ↓
检查每一个 parameter（参数）
        ↓
询问已注册的 ParameterResolver：supportsParameter(...) ?
        ↓ true
调用 resolveParameter(...)
        ↓
得到 argument（实参）
        ↓
JUnit 调用构造器或方法
```

例如：

```java
@Test
void test(TestInfo info) {
}
```

概念上相当于：

```text
发现参数类型 TestInfo
        ↓
TestInfoParameterResolver 支持该参数
        ↓
解析当前测试的 TestInfo
        ↓
调用 test(info)
```

“解析参数”不一定意味着创建新对象。解析器可以返回：

- 当前上下文的元数据（metadata）；
- 缓存的共享资源（cached resource）；
- 固定值或随机值；
- Mockito mock；
- Spring bean；
- 扩展自行管理的其他对象。

### 两个方法的职责

`supportsParameter(...)`：

- 判断当前解析器是否负责这个参数；
- 应尽量同时检查参数类型、限定注解（qualifier annotation）和所处上下文；
- 不应过度宽泛地声称支持所有 `String`、`int` 或 `Object` 参数。

`resolveParameter(...)`：

- 只有在前一个方法返回 `true` 后才会调用；
- 返回真正传给构造器或方法的实参；
- 只有非基本类型（non-primitive type）参数才允许解析为 `null`。

---

## 4. 哪些位置可以接收参数 / Supported locations

以下位置可以参与 JUnit 的参数解析，但某个具体解析器仍可以限制自己的适用范围：

| 位置 | 是否可参与解析 | 备注 |
|---|---:|---|
| 测试类构造器（test-class constructor） | ✅ | JUnit 创建测试实例前解析参数。 |
| `@Test` 方法 | ✅ | 所有参数都必须有提供者。 |
| `@RepeatedTest` 方法 | ✅ | `RepetitionInfo` 只在这一上下文中可用。 |
| `@ParameterizedTest` 方法 | ✅ | 参数源参数和扩展参数必须遵守顺序规则。 |
| `@TestFactory` 方法 | ✅ | factory 方法可接收解析参数；其生成的 `DynamicTest` lambda 本身不会接收注入。 |
| `@TestTemplate` 方法 | ✅ | 具体能力由 invocation context 和扩展决定。 |
| `@BeforeEach` / `@AfterEach` | ✅ | `RepetitionInfo` 仅在重复测试调用周围可用。 |
| `@BeforeAll` / `@AfterAll` | ✅ | 通常必须为 `static`，除非使用 `PER_CLASS` 生命周期。 |
| 普通 helper 方法 | ❌ | JUnit 不负责调用，因而不会自动注入。 |

[`@ExtendWith`](https://docs.junit.org/5.14.4/api/org.junit.jupiter.api/org/junit/jupiter/api/extension/ExtendWith.html) 可以声明式注册扩展（declarative extension registration）。常见写法是类级注册：

```java
@ExtendWith(MyParameterResolver.class)
class MyTest {
}
```

JUnit 还支持方法级或参数级 `@ExtendWith`、`@RegisterExtension`、组合注解（composed annotation）和自动扩展检测等方式。`@ExtendWith` 是最直接、最常用的入门方式，但不是唯一方式。

---

## 5. 本节讨论的三个自动解析能力 / Three automatically available resolvers

| 注入类型 | 提供者 | 典型用途 | 主要限制 |
|---|---|---|---|
| `TestInfo` | `TestInfoParameterResolver` | 读取当前容器或测试的名称、类、方法、标签 | 类和方法以 `Optional` 返回，某些上下文中可能不存在 |
| `RepetitionInfo` | `RepetitionExtension`；较旧文档曾写作 `RepetitionInfoParameterResolver` | 读取当前重复次数、总次数等 | 仅在 `@RepeatedTest` 上下文中有效 |
| `TestReporter` | `TestReporterParameterResolver` | 发布结构化测试报告信息 | 如何显示由 IDE、listener 或构建工具决定 |

这里的“三个”是原章节重点讨论的三种自动解析能力，不代表 JUnit 只有这三种参数提供方式。例如 `@TempDir` 也能注入临时目录，而参数化测试的 argument sources 使用另一条参数供应管线。

### 5.1 `TestInfo`：测试信息 / Test information

[`TestInfo`](https://docs.junit.org/5.14.4/api/org.junit.jupiter.api/org/junit/jupiter/api/TestInfo.html) 可提供：

- `getDisplayName()`：显示名称（display name）；
- `getTags()`：标签集合（tags）；
- `getTestClass()`：当前测试类，返回 `Optional<Class<?>>`；
- `getTestMethod()`：当前测试方法，返回 `Optional<Method>`。

`Optional` 表示该信息在当前上下文中可能不存在，因此优先使用 `map(...)`、`orElse(...)` 或 `orElseThrow(...)`，不要在不确定的上下文中直接 `.get()`。

`TestInfo` 也可视为 JUnit 4 `TestName` rule 的现代替代方式之一。

### 5.2 `RepetitionInfo`：重复执行信息 / Repetition information

[`RepetitionInfo`](https://docs.junit.org/5.14.4/api/org.junit.jupiter.api/org/junit/jupiter/api/RepetitionInfo.html) 用于 `@RepeatedTest`，常见信息包括：

- `getCurrentRepetition()`：当前第几次执行，从 1 开始；
- `getTotalRepetitions()`：总执行次数；
- 新版还提供失败次数（failure count）和失败阈值（failure threshold）信息。

它只会在重复测试方法以及围绕该调用的 `@BeforeEach`、`@AfterEach` 中提供。它不会在普通 `@Test`、构造器、`@BeforeAll` 或 `@AfterAll` 中自动提供。

如果某个类同时包含普通测试和重复测试，不要无条件声明类级的：

```java
@BeforeEach
void beforeEach(RepetitionInfo repetitionInfo) {
}
```

否则该方法在普通测试前执行时，没有解析器能够提供 `RepetitionInfo`。

### 5.3 `TestReporter`：测试报告器 / Test reporter

[`TestReporter`](https://docs.junit.org/5.14.4/api/org.junit.jupiter.api/org/junit/jupiter/api/TestReporter.html) 用于向测试报告基础设施（reporting infrastructure）发布数据：

```java
testReporter.publishEntry("status message");
testReporter.publishEntry("user", "Rainkoli");
testReporter.publishEntry(Map.of("feature", "login", "result", "passed"));
```

这些信息可以由 `TestExecutionListener`、IDE 或测试报告消费。IDE **可能**把它显示为类似 `user = Rainkoli` 的内容，但显示位置和格式不是 `TestReporter` 的保证。

对于希望进入测试报告的诊断数据，它通常比随意打印到 `stdout` 或 `stderr` 更合适。它不是日志框架（logging framework），也不能代替断言（assertion）。

JUnit 5.14 还支持发布文件和目录；这是相对原 5.10 章节更晚的能力，不应假设旧版 JUnit 一定支持。

---

## 6. Runnable demo project / 可运行 demo 项目

### 6.1 项目结构 / Project layout

```text
junit-dependency-injection-demo/
├── pom.xml
└── src/test/java/example/
    ├── TestInfoDemoTest.java
    ├── RepetitionInfoDemoTest.java
    ├── TestReporterDemoTest.java
    ├── RandomInt.java
    ├── RandomIntParameterResolver.java
    ├── RandomIntDemoTest.java
    └── ParameterizedResolverDemoTest.java
```

### 6.2 Maven build configuration / Maven 构建配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>example</groupId>
    <artifactId>junit-dependency-injection-demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.release>17</maven.compiler.release>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <junit.version>5.14.4</junit.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.5.3</version>
            </plugin>
        </plugins>
    </build>
</project>
```

运行：

```shell
mvn test
```

官方构建说明：[JUnit 5.14.4 Build Support](https://docs.junit.org/5.14.4/running-tests/build-support.html)。

### 6.3 Gradle alternative / Gradle 备选配置

```groovy
plugins {
    id 'java'
}

repositories {
    mavenCentral()
}

dependencies {
    testImplementation(platform('org.junit:junit-bom:5.14.4'))
    testImplementation('org.junit.jupiter:junit-jupiter')
    testRuntimeOnly('org.junit.platform:junit-platform-launcher')
}

test {
    useJUnitPlatform()
}
```

运行：

```shell
./gradlew test
```

Windows：

```powershell
gradlew.bat test
```

---

## 7. Demo 1 — `TestInfo` 注入 / `TestInfo` injection

文件：`src/test/java/example/TestInfoDemoTest.java`

```java
package example;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertTrue;

import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Tag;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestInfo;

@DisplayName("TestInfo 注入演示 / TestInfo injection demo")
class TestInfoDemoTest {

    private final String constructorDisplayName;

    @BeforeAll
    static void beforeAll(TestInfo testInfo) {
        assertEquals(
                "TestInfo 注入演示 / TestInfo injection demo",
                testInfo.getDisplayName());
    }

    TestInfoDemoTest(TestInfo testInfo) {
        constructorDisplayName = testInfo.getDisplayName();
    }

    @BeforeEach
    void beforeEach(TestInfo testInfo) {
        assertEquals("登录成功 / login succeeds", testInfo.getDisplayName());
    }

    @Test
    @DisplayName("登录成功 / login succeeds")
    @Tag("fast")
    void exposesMetadata(TestInfo testInfo) {
        // JUnit 5.14.4 + default PER_METHOD lifecycle.
        assertEquals("登录成功 / login succeeds", constructorDisplayName);
        assertEquals("登录成功 / login succeeds", testInfo.getDisplayName());
        assertTrue(testInfo.getTags().contains("fast"));
        assertEquals(
                TestInfoDemoTest.class,
                testInfo.getTestClass().orElseThrow());
    }
}
```

这个 demo 同时展示：

- `@BeforeAll` 中的容器级信息（container-level information）；
- 构造器注入（constructor injection）；
- `@BeforeEach` 生命周期方法注入（lifecycle-method injection）；
- `@Test` 方法注入（test-method injection）；
- `@DisplayName` 与 `@Tag` 元数据。

如果使用 JUnit 5.10.x，构造器中的 `TestInfo` 可能显示类/容器名称，而不是当前测试方法名称。这个差异不影响 `ParameterResolver` 的核心概念。

---

## 8. Demo 2 — `RepetitionInfo` 注入 / Repetition injection

文件：`src/test/java/example/RepetitionInfoDemoTest.java`

```java
package example;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertTrue;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.RepeatedTest;
import org.junit.jupiter.api.RepetitionInfo;
import org.junit.jupiter.api.TestInfo;

class RepetitionInfoDemoTest {

    @BeforeEach
    void beforeEach(RepetitionInfo repetitionInfo) {
        assertTrue(repetitionInfo.getCurrentRepetition() >= 1);
    }

    @RepeatedTest(
            value = 3,
            name = "{displayName} — {currentRepetition}/{totalRepetitions}")
    @DisplayName("重复测试 / repeated test")
    void repeatsThreeTimes(
            RepetitionInfo repetitionInfo,
            TestInfo testInfo) {
        assertEquals(3, repetitionInfo.getTotalRepetitions());
        assertTrue(repetitionInfo.getCurrentRepetition() <= 3);
        assertTrue(testInfo.getDisplayName()
                .startsWith("重复测试 / repeated test"));
    }

    @AfterEach
    void afterEach(RepetitionInfo repetitionInfo) {
        assertTrue(
                repetitionInfo.getCurrentRepetition()
                        <= repetitionInfo.getTotalRepetitions());
    }
}
```

预期执行三次：

```text
重复测试 / repeated test — 1/3
重复测试 / repeated test — 2/3
重复测试 / repeated test — 3/3
```

这个类只包含重复测试，是有意为之。若同一类中还有普通 `@Test`，带 `RepetitionInfo` 的 `@BeforeEach` 会在普通测试前解析失败。

官方说明：[Repeated Tests](https://docs.junit.org/5.14.4/writing-tests/repeated-tests.html)。

---

## 9. Demo 3 — `TestReporter` 注入 / Test reporting

文件：`src/test/java/example/TestReporterDemoTest.java`

```java
package example;

import java.util.Map;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestReporter;

class TestReporterDemoTest {

    @Test
    void publishesReportEntries(TestReporter testReporter) {
        testReporter.publishEntry("status message");
        testReporter.publishEntry("user", "Rainkoli");
        testReporter.publishEntry(Map.of(
                "feature", "login",
                "result", "passed"));
    }
}
```

这些 entry（条目）进入 JUnit reporting infrastructure（报告基础设施）。IDE、构建插件或 listener 可以选择显示或持久化它们；代码不应依赖某个 IDE 的具体展示格式。

---

## 10. Demo 4 — 自定义随机整数解析器 / Custom random integer resolver

这是此前提到的 `@Random int number` 示例的完整、可运行版本。与“只要是 `int` 就处理”相比，它同时检查限定注解和类型，从而减少解析冲突。

### 10.1 参数限定注解 / Parameter qualifier annotation

文件：`src/test/java/example/RandomInt.java`

```java
package example;

import static java.lang.annotation.ElementType.PARAMETER;
import static java.lang.annotation.RetentionPolicy.RUNTIME;

import java.lang.annotation.Retention;
import java.lang.annotation.Target;

@Target(PARAMETER)
@Retention(RUNTIME)
public @interface RandomInt {
    int min() default 0;

    int maxExclusive() default 100;
}
```

### 10.2 `ParameterResolver` 实现 / Resolver implementation

文件：`src/test/java/example/RandomIntParameterResolver.java`

```java
package example;

import java.util.concurrent.ThreadLocalRandom;

import org.junit.jupiter.api.extension.ExtensionContext;
import org.junit.jupiter.api.extension.ParameterContext;
import org.junit.jupiter.api.extension.ParameterResolutionException;
import org.junit.jupiter.api.extension.ParameterResolver;

public class RandomIntParameterResolver implements ParameterResolver {

    @Override
    public boolean supportsParameter(
            ParameterContext parameterContext,
            ExtensionContext extensionContext) {
        return parameterContext.isAnnotated(RandomInt.class)
                && parameterContext.getParameter().getType() == int.class;
    }

    @Override
    public Object resolveParameter(
            ParameterContext parameterContext,
            ExtensionContext extensionContext) {
        RandomInt randomInt = parameterContext
                .findAnnotation(RandomInt.class)
                .orElseThrow();

        if (randomInt.min() >= randomInt.maxExclusive()) {
            throw new ParameterResolutionException(
                    "@RandomInt requires min < maxExclusive");
        }

        return ThreadLocalRandom.current().nextInt(
                randomInt.min(),
                randomInt.maxExclusive());
    }
}
```

### 10.3 注册并使用扩展 / Registering and using the extension

文件：`src/test/java/example/RandomIntDemoTest.java`

```java
package example;

import static org.junit.jupiter.api.Assertions.assertTrue;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;

@ExtendWith(RandomIntParameterResolver.class)
class RandomIntDemoTest {

    private final int constructorValue;

    RandomIntDemoTest(
            @RandomInt(min = 1, maxExclusive = 7) int constructorValue) {
        this.constructorValue = constructorValue;
    }

    @BeforeEach
    void setUp(
            @RandomInt(min = 10, maxExclusive = 20) int setupValue) {
        assertTrue(setupValue >= 10 && setupValue < 20);
    }

    @Test
    void injectsRandomIntegers(
            @RandomInt(min = -5, maxExclusive = 6) int first,
            @RandomInt(min = 100, maxExclusive = 101) int second) {
        assertTrue(constructorValue >= 1 && constructorValue < 7);
        assertTrue(first >= -5 && first < 6);
        assertTrue(second == 100);
    }
}
```

这里没有使用 `assertNotEquals(first, second)` 来证明随机性，因为两个随机值偶然相等是合法情况，会造成概率性失败（flaky test）。范围断言更可靠。

解析流程：

```text
发现 @RandomInt int
        ↓
RandomIntParameterResolver.supportsParameter(...)
        ↓ true
RandomIntParameterResolver.resolveParameter(...)
        ↓
生成范围内的 Integer，并自动拆箱为 int
        ↓
调用构造器、生命周期方法或测试方法
```

官方延伸阅读：[Parameter Resolution](https://docs.junit.org/5.14.4/extensions/parameter-resolution.html) 和 [Registering Extensions](https://docs.junit.org/5.14.4/extensions/registering-extensions.html)。

---

## 11. Demo 5 — 与参数化测试配合 / Parameterized-test interoperability

参数源（argument source）提供的参数必须位于前面，普通 `ParameterResolver` 提供的参数位于后面。

文件：`src/test/java/example/ParameterizedResolverDemoTest.java`

```java
package example;

import static org.junit.jupiter.api.Assertions.assertFalse;
import static org.junit.jupiter.api.Assertions.assertTrue;

import org.junit.jupiter.api.TestInfo;
import org.junit.jupiter.api.TestReporter;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.ValueSource;

class ParameterizedResolverDemoTest {

    @ParameterizedTest(name = "fruit={0}")
    @ValueSource(strings = {"apple", "pear"})
    void sourceArgumentsComeFirst(
            String fruit,
            TestInfo testInfo,
            TestReporter testReporter) {
        assertFalse(fruit.isBlank());
        assertFalse(testInfo.getDisplayName().isBlank());
        testReporter.publishEntry("fruit", fruit);
        assertTrue(fruit.equals("apple") || fruit.equals("pear"));
    }
}
```

参数顺序（parameter ordering）应为：

1. indexed parameters：由 `@ValueSource`、`@CsvSource` 等提供；
2. aggregators：例如 `ArgumentsAccessor` 或 `@AggregateWith`；
3. other resolver parameters：例如 `TestInfo`、`TestReporter`、`@Mock` 参数。

不要把 `@ValueSource` 或 `@CsvSource` 与 `ParameterResolver` 当成同一种机制。两者可以协作，但使用不同的参数供应管线。

官方说明：[Parameterized-test lifecycle and interoperability](https://docs.junit.org/5.10.0/user-guide/index.html#writing-tests-parameterized-tests-lifecycle-interop)。

---

## 12. 常见失败场景 / Common failure modes

### 12.1 没有解析器 / No resolver

```java
@Test
void unresolved(String value) {
}
```

没有任何 resolver 或 argument source 提供 `value`，因此会出现 `ParameterResolutionException`。

### 12.2 多个解析器同时支持 / Competing resolvers

如果两个已注册解析器的 `supportsParameter(...)` 都对同一参数返回 `true`，JUnit 不会简单选择“先注册的那个”，而是报告解析冲突。

解决方法：

- 使用不同的限定注解，例如 `@RandomInt` 与 `@PortNumber`；
- 让解析器同时检查类型和注解；
- 使用不同的 wrapper types（包装类型）；
- 缩小扩展注册范围，例如只注册到某个方法；
- 避免让自定义解析器抢占 `TestInfo` 等内置类型。

官方说明：[Parameter conflicts](https://docs.junit.org/5.14.4/extensions/parameter-resolution.html#parameter-conflicts)。

### 12.3 在普通测试中请求 `RepetitionInfo`

```java
@Test
void ordinaryTest(RepetitionInfo repetitionInfo) {
}
```

`RepetitionInfo` 只在 `@RepeatedTest` 上下文中存在，所以此写法会失败。

### 12.4 参数化测试顺序错误 / Wrong parameter order

概念上的错误示例：

```java
@ParameterizedTest
@ValueSource(strings = "apple")
void wrongOrder(TestInfo info, String value) {
}
```

argument-source parameter `String value` 应位于 resolver parameter `TestInfo info` 之前。

### 12.5 把 `TestReporter` 当作固定控制台输出

`publishEntry(...)` 发布的是报告事件，不保证立即以固定文本打印到 console。需要固定日志时使用日志框架；需要验证结果时使用 assertion。

---

## 13. Mockito 与 Spring 示例 / Mockito and Spring examples

### 13.1 Mockito extension / Mockito 扩展

```java
import java.time.Clock;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    UserRepository repository;

    @InjectMocks
    UserService service;

    @Test
    void usesMethodLocalMock(@Mock Clock clock) {
        // repository and clock are Mockito mocks.
    }
}
```

`MockitoExtension` 会初始化 mocks，并为带有 `@Mock` 等限定注解的参数提供值。它不会解析任意未注解的业务对象，也不是 bean container（Bean 容器）。

官方实现：[MockitoExtension source](https://github.com/mockito/mockito/blob/main/mockito-extensions/mockito-junit-jupiter/src/main/java/org/mockito/junit/jupiter/MockitoExtension.java)。

### 13.2 Spring extension / Spring 扩展

```java
import static org.junit.jupiter.api.Assertions.assertEquals;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.test.context.junit.jupiter.SpringJUnitConfig;

@SpringJUnitConfig(SpringInjectionDemoTest.TestConfig.class)
class SpringInjectionDemoTest {

    @Configuration
    static class TestConfig {
        @Bean
        GreetingService greetingService() {
            return () -> "hello";
        }
    }

    interface GreetingService {
        String greet();
    }

    @Test
    void injectsSpringBean(
            @Autowired GreetingService greetingService) {
        assertEquals("hello", greetingService.greet());
    }
}
```

`@SpringJUnitConfig(TestConfig.class)` 是组合注解（composed annotation），结合了 `@ExtendWith(SpringExtension.class)` 和 Spring context configuration（上下文配置）。只写 `@ExtendWith(SpringExtension.class)` 而不提供应用上下文配置，通常不足以获得想要的业务 bean。

Spring 的 `SpringExtension` 实现 JUnit 的参数解析契约，把 Spring `ApplicationContext` 中的 bean 接入 JUnit 调用过程。它可以根据 `ApplicationContext` 类型或 `@Autowired`、`@Qualifier`、`@Value` 等注解解析参数。

官方说明：[Spring Framework 6.2 — Dependency Injection with `SpringExtension`](https://docs.spring.io/spring-framework/reference/6.2/testing/testcontext-framework/support-classes.html#testcontext-junit-jupiter-di)。

---

## 14. JUnit DI 与 Spring DI 的关系 / JUnit DI vs. Spring DI

两者共享依赖反转（inversion of control）的思想：使用者声明自己需要什么，由框架在调用时提供。

```text
JUnit Jupiter
测试构造器或方法参数
        ↓
ParameterResolver
        ↓
解析并提供 argument
        ↓
JUnit 调用测试代码
```

```text
Spring
构造器、factory method 或属性声明依赖
        ↓
ApplicationContext / BeanFactory
        ↓
查找或创建 bean、管理 object graph
        ↓
注入应用对象
```

| 比较项 | JUnit Jupiter parameter resolution | Spring dependency injection |
|---|---|---|
| 中文名称 | JUnit 参数解析/测试依赖注入 | Spring 依赖注入 |
| 主要目标 | 为测试运行时的构造器和方法提供参数 | 构建并管理应用对象图 |
| 核心扩展点 | `ParameterResolver` | `ApplicationContext` / `BeanFactory` |
| 生命周期 | 测试容器、测试类、测试调用 | Spring bean scope |
| 是否天然管理完整 Bean 图 | 否 | 是 |
| 典型数据 | `TestInfo`、随机值、mock、临时资源 | service、repository、configuration bean |

因此它们“思想相似”，但不能简单认为 JUnit 自身就是一个完整的 IoC container。`SpringExtension` 才是把 Spring 容器桥接到 JUnit 扩展模型中的组件。

还要注意：如果 Spring 判定整个测试构造器可自动装配，它可能负责该构造器的所有参数；此时其他 resolver 无法再解析其中某个参数。组合多个扩展时，应避免它们同时声称支持同一参数。

---

## 15. 最佳实践 / Best practices

1. **让 resolver 只支持明确参数。** 同时检查类型和限定注解，避免 `Object`、`String`、`int` 等过宽匹配。
2. **使用 public API。** 业务测试依赖 `RepetitionInfo`，不要导入内部的 `RepetitionExtension` 或 `RepetitionInfoParameterResolver`。
3. **安全处理 `Optional`。** `TestInfo.getTestClass()` 和 `getTestMethod()` 并非在所有上下文中都有值。
4. **避免概率性断言。** 随机 demo 验证范围，不验证两个随机值“必须不同”。
5. **限制扩展范围。** 仅某个测试需要的 resolver 可以注册到方法级，减少冲突。
6. **区分参数化数据和依赖。** argument-source 参数放在前面，resolver 参数放在后面。
7. **让报告与断言各司其职。** `TestReporter` 提供报告元数据，Assertions 判断测试是否通过。
8. **锁定并记录版本。** 特别是构造器 `TestInfo` 上下文、`RepetitionInfo` 新属性和 `TestReporter` 文件 API 等行为。

---

## 16. 对此前简化说法的修正 / Corrections to the simplified explanation

| 简化说法 | 更准确的表述 |
|---|---|
| “JUnit 4 不允许参数” | 标准 Runner 的普通测试方法要求无参；专用或自定义 Runner 可提供有限参数化。 |
| “JUnit 5 方法可以随便写参数” | 可以声明参数，但每个参数必须有 resolver 或相应 argument source。 |
| “ParameterResolver 创建对象” | 它解析并提供 argument；该值可以是新对象、缓存对象、常量、mock 或 bean。 |
| “JUnit 只有三个解析器” | 原章节重点介绍三种自动能力；`@TempDir`、参数化来源和第三方扩展还会提供其他参数。 |
| “RepetitionExtension 到处都可用” | 它只在 `@RepeatedTest` 调用上下文中提供 `RepetitionInfo`。 |
| “IDE 会显示 `user = Rainkoli`” | IDE 或 listener 可能显示报告条目，具体格式与位置取决于运行环境。 |
| “其他 resolver 必须只用 `@ExtendWith`” | `@ExtendWith` 最常见；另有 `@RegisterExtension`、组合注解和自动检测等方式。 |

---

## 17. Summary / 总结

JUnit Jupiter 的构造器和方法依赖注入，本质上是 extension model（扩展模型）中的 runtime parameter resolution（运行时参数解析）：

```text
执行测试
    ↓
检查构造器或方法参数
    ↓
查找唯一匹配的 ParameterResolver
    ↓
解析 argument
    ↓
调用构造器或方法
```

最重要的规则：

- 参数必须有人提供；JUnit 不会猜测任意对象的来源。
- `TestInfo` 用于测试元数据。
- `RepetitionInfo` 只用于重复测试上下文。
- `TestReporter` 用于结构化报告信息。
- 自定义 resolver 通过 `supportsParameter(...)` 和 `resolveParameter(...)` 工作。
- `@ExtendWith` 是最常见的声明式注册方式。
- Spring 和 Mockito 可以通过各自 extension 接入相同的 JUnit 扩展模型。
- 多个 resolver 同时支持同一参数会造成冲突，而不是按注册顺序自动选择。

---

## 18. Official documentation / 官方文档

### Source-matched JUnit 5 documentation / 与原章节匹配

- [JUnit 5.10.0 — Dependency Injection for Constructors and Methods (§2.13)](https://docs.junit.org/5.10.0/user-guide/index.html#writing-tests-dependency-injection)
- [JUnit 5.10.0 — `ParameterResolver` API](https://docs.junit.org/5.10.0/api/org.junit.jupiter.api/org/junit/jupiter/api/extension/ParameterResolver.html)
- [JUnit 5.10.0 — `TestInfo` API](https://docs.junit.org/5.10.0/api/org.junit.jupiter.api/org/junit/jupiter/api/TestInfo.html)
- [JUnit 5.10.0 — `RepetitionInfo` API](https://docs.junit.org/5.10.0/api/org.junit.jupiter.api/org/junit/jupiter/api/RepetitionInfo.html)
- [JUnit 5.10.0 — `TestReporter` API](https://docs.junit.org/5.10.0/api/org.junit.jupiter.api/org/junit/jupiter/api/TestReporter.html)
- [JUnit 5.10.0 — Extension Registration](https://docs.junit.org/5.10.0/user-guide/index.html#extensions-registration)
- [JUnit 5.10.0 — Repeated Tests](https://docs.junit.org/5.10.0/user-guide/index.html#writing-tests-repeated-tests)
- [JUnit 5.10.0 — Dynamic Tests](https://docs.junit.org/5.10.0/user-guide/index.html#writing-tests-dynamic-tests)

### Newer JUnit 5 documentation / 较新的 JUnit 5 文档

- [JUnit 5.14.4 — Dependency Injection for Constructors and Methods](https://docs.junit.org/5.14.4/writing-tests/dependency-injection-for-constructors-and-methods.html)
- [JUnit 5.14.4 — Parameter Resolution](https://docs.junit.org/5.14.4/extensions/parameter-resolution.html)
- [JUnit 5.14.4 — Registering Extensions](https://docs.junit.org/5.14.4/extensions/registering-extensions.html)
- [JUnit 5.14.4 — Build Support](https://docs.junit.org/5.14.4/running-tests/build-support.html)

### Current JUnit documentation / 当前文档入口

- [Current — Dependency Injection for Constructors and Methods](https://docs.junit.org/current/writing-tests/dependency-injection-for-constructors-and-methods.html)
- [Current — Parameter Resolution](https://docs.junit.org/current/extensions/parameter-resolution.html)
- [Current — `ParameterResolver` API](https://docs.junit.org/current/api/org.junit.jupiter.api/org/junit/jupiter/api/extension/ParameterResolver.html)
- [Current — `@ExtendWith` API](https://docs.junit.org/current/api/org.junit.jupiter.api/org/junit/jupiter/api/extension/ExtendWith.html)

### Third-party integrations / 第三方集成

- [Spring Framework 6.2 — Dependency Injection with `SpringExtension`](https://docs.spring.io/spring-framework/reference/6.2/testing/testcontext-framework/support-classes.html#testcontext-junit-jupiter-di)
- [Mockito — `MockitoExtension` source](https://github.com/mockito/mockito/blob/main/mockito-extensions/mockito-junit-jupiter/src/main/java/org/mockito/junit/jupiter/MockitoExtension.java)

---

## Verification / 验证记录

本文第 7–11 节的核心 demo 已使用以下环境执行并通过：

```text
Java: 17
JUnit Jupiter: 5.14.4
Maven Surefire Plugin: 3.5.3
Command: mvn test
Result: passed
```
