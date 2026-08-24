# Spring Boot / Maven / Apifox 报错案例总结

本文整理了本次排查过程中出现的几个典型错误，以及对应原因和解决方法。

---

## 1. Maven 编译失败：`缺少返回语句`

### 报错信息

```text
[ERROR] COMPILATION ERROR
FileController.java:[21,5] 缺少返回语句
[INFO] BUILD FAILURE
```

### 原因

Java 方法声明了返回值类型，但并不是所有执行路径都会执行 `return`。

例如：

```java
public Result upload(...) {
    try {
        return Result.ok();
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

如果进入 `catch`，方法执行结束时没有返回 `Result`，因此编译器报错。

### 解决方法

确保所有执行路径都有返回值：

```java
public Result upload(...) {
    try {
        return Result.ok();
    } catch (Exception e) {
        e.printStackTrace();
        return Result.fail();
    }
}
```

### 补充

日志中的：

```text
使用或覆盖了已过时的 API
```

属于 `deprecated` 警告，不是本次 `BUILD FAILURE` 的直接原因。

---

## 2. Spring Boot 启动失败：`Ambiguous mapping`

### 报错信息

```text
Ambiguous mapping. Cannot map 'day14Controller' method
Day14Controller#addHelloWorld(String)

There is already 'testParamController' bean method
TestParamController#receiveUserParam(String, String) mapped.
```

以及：

```text
to {POST [/ || ]}
```

### 原因

两个 Controller 方法被 Spring 注册成了相同的请求方式和请求路径，例如都变成：

```text
POST /
```

Spring 收到请求时无法判断应该调用哪个方法，因此出现 `Ambiguous mapping`。

---

## 3. `@PostMapping(name = "...")` 并不是设置 URL

这是这次 `Ambiguous mapping` 的关键原因。

### 错误写法

```java
@PostMapping(name = "/testParam")
```

以及：

```java
@RequestMapping(name = "/day14")
@PostMapping(name = "/getHelloWorld")
```

这里的 `name` 是 Mapping 的逻辑名称，不是请求路径。

所以 Spring 并不会把它们识别成：

```text
/testParam
/day14/getHelloWorld
```

### 正确写法

```java
@PostMapping("/testParam")
```

或者：

```java
@PostMapping(path = "/testParam")
```

或者：

```java
@PostMapping(value = "/testParam")
```

Day14 可以写成：

```java
@Controller
@ResponseBody
@RequestMapping("/day14")
public class Day14Controller {

    @PostMapping("/getHelloWorld")
    public Result addHelloWorld(
            @RequestParam(
                    name = "messageX",
                    required = false,
                    defaultValue = "Hello!"
            ) String message
    ) {
        return Result.ok(message);
    }
}
```

最终路径为：

```text
POST /day14/getHelloWorld
```

Day13 例如：

```java
@PostMapping("/testParam")
```

最终路径为：

```text
POST /testParam
```

这样就不会冲突。

---

## 4. 为什么 `@RequestParam(name = "...")` 又可以使用 `name`

不同注解中的 `name` 属性含义不同。

例如：

```java
@RequestParam(name = "user_name")
```

这里的 `name` 表示 HTTP 请求参数名称，所以是正确的。

例如请求：

```text
?user_name=Tom
```

但是：

```java
@PostMapping(name = "/hello")
```

这里的 `name` 并不代表 URL。

可以简单记忆为：

```java
// 错误：name 不是路径
@PostMapping(name = "/hello")

// 正确
@PostMapping("/hello")

// 正确
@PostMapping(path = "/hello")

// 正确
@PostMapping(value = "/hello")
```

---

## 5. 为什么启动 Day14，却扫描到了 Day13 的 Controller

日志中出现了：

```text
Day14Controller
```

同时又出现：

```text
TestParamController
```

说明 Spring 的组件扫描范围包含了 Day13 和 Day14。

如果启动类中使用了较大的扫描范围，例如：

```java
@ComponentScan("com.hs.homework")
```

那么该包下面以前写过的：

```text
@Controller
@RestController
@Component
@Service
Repository
```

等组件都可能被注册进当前 Spring 容器。

### 建议

如果希望不同练习互不影响，可以：

1. 给不同 Controller 设置不同的请求前缀；
2. 缩小组件扫描范围；
3. 将每天的练习拆成更独立的 Spring Boot 模块或项目。

---

## 6. IDEA 跳到 `WebMvcConfigurationSupport.java` 不代表 Spring 源码有问题

出现 `Ambiguous mapping` 时，IDEA 可能跳到：

```java
WebMvcConfigurationSupport.java
```

例如：

```java
requestMappingHandlerMapping(...)
```

这是 Spring MVC 框架内部创建和校验 URL 映射的位置。

真正的问题通常仍然在自己写的：

```text
Controller
@RequestMapping
@GetMapping
@PostMapping
```

等代码中。

### 注意

不要直接修改 Spring 框架源码来解决自己的 Controller 映射问题。

---

## 7. 文件上传失败：HTTP `413`

### Apifox 表现

请求返回：

```text
413
```

### Spring Boot 控制台

```text
MaxUploadSizeExceededException: Maximum upload size exceeded
```

### 原因

上传的文件超过了 Spring Boot 当前允许的 multipart 文件大小。

而且这个异常通常发生在 Spring MVC 解析 `multipart/form-data` 时，因此请求可能还没有真正进入：

```java
public Result uploadFile(MultipartFile file)
```

---

## 8. 调整 Spring Boot 文件上传大小

可以在 `application.yml` 中配置：

```yaml
spring:
  servlet:
    multipart:
      max-file-size: 20MB
      max-request-size: 20MB
```

含义：

```text
max-file-size
```

表示单个文件允许的最大大小。

```text
max-request-size
```

表示整个 multipart HTTP 请求允许的最大大小。

如果文件较大，也可以调整为：

```yaml
spring:
  servlet:
    multipart:
      max-file-size: 100MB
      max-request-size: 100MB
```

### 如果使用 Spring Profile

例如当前启用了：

```yaml
spring:
  profiles:
    active: dev
```

那么应确认上传大小配置写在实际生效的 `dev` 配置段中，例如：

```yaml
---
spring:
  config:
    activate:
      on-profile: dev

  servlet:
    multipart:
      max-file-size: 20MB
      max-request-size: 20MB

server:
  port: 8080

filePath: E:/files
```

修改配置后，需要重新启动 Spring Boot。

---

## 9. 上传目录也需要真实存在

Controller 中如果使用：

```java
file.transferTo(new File(filePath, originalFilename));
```

并且配置：

```yaml
filePath: E:/files
```

那么需要确保：

```text
E:/files
```

目录真实存在，并且程序有写入权限。

否则解决文件大小问题之后，还可能继续出现：

```text
目录不存在
路径不存在
AccessDeniedException
FileNotFoundException
```

等文件系统错误。

---

## 10. IDEA 的 `Upload to Apifox` 会不会同步 `application.yml`

一般不会。

`Upload to Apifox` 主要用于同步 API 定义，例如：

```text
Controller
URL
HTTP Method
请求参数
返回类型
接口文档
```

像下面这样的 Spring Boot 运行配置：

```yaml
spring:
  servlet:
    multipart:
      max-file-size: 20MB
      max-request-size: 20MB
```

属于服务端本地配置，不需要同步到 Apifox。

正确流程是：

```text
修改 application.yml
        ↓
保存文件
        ↓
重新启动 Spring Boot
        ↓
Spring Boot 读取新配置
        ↓
在 Apifox 中重新 Send 请求
```

修改上传大小限制后，关键操作是 **重启 Spring Boot**，而不是点击 `Upload to Apifox`。

---

# 快速排错总结

| 现象 | 关键报错 | 主要原因 | 解决方向 |
|---|---|---|---|
| Maven 编译失败 | `缺少返回语句` | 方法并非所有路径都有 `return` | 补全返回值 |
| Spring 启动失败 | `Ambiguous mapping` | 两个接口映射到相同 Method + URL | 修改 Controller 路径 |
| URL 明明不同却冲突 | `{POST [/ || ]}` | 错把 `name` 当成 URL | 使用 `value/path` 或直接写路径 |
| IDEA 跳进 Spring 源码 | `WebMvcConfigurationSupport` | Spring 正在校验接口映射 | 不改框架源码，检查 Controller |
| 文件上传失败 | HTTP `413` | 文件超过上传大小限制 | 配置 multipart 大小 |
| 配置改了仍无效 | 仍然 413 | 没重启或配置没写到当前 Profile | 检查 Profile 并重启 |
| 文件大小问题解决后仍失败 | 文件路径异常 | 上传目录不存在或不可写 | 创建目录并检查权限 |
| 点击 Upload to Apifox | application.yml 未同步 | 该功能主要同步 API 定义 | 本地配置修改后重启应用 |

---

# 推荐排查顺序

遇到 Spring Boot 报错时，可以按照以下顺序检查：

1. 先看控制台最后的 `Application run failed` 或 `BUILD FAILURE`。
2. 从下往上寻找第一个关键 `Caused by:`。
3. 如果是编译错误，先解决 Java 语法和返回值问题。
4. 如果是 `Ambiguous mapping`，检查所有 Controller 的 HTTP Method 和 URL。
5. 如果日志显示 `{POST [/ || ]}`，重点检查是否错误使用了 `name`。
6. 如果是 `413`，检查 `spring.servlet.multipart` 配置。
7. 修改 `application.yml` 后重新启动 Spring Boot。
8. 如果上传仍失败，再检查目标目录、文件权限和 `transferTo()`。

