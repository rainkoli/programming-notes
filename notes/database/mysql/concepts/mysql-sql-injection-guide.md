# MySQL SQL 注入（SQL Injection）详解

> 本文整理并扩展了前面对 MySQL、JDBC、MyBatis 与 MyBatis-Plus 中 SQL 注入的说明。示例用于安全学习与代码审查，请只在自己拥有或明确获准测试的系统中使用。

最后更新：2026-08-20

## 目录

- [1. 一句话定义](#1-一句话定义)
- [2. 本质：SQL 代码与数据混在了一起](#2-本质sql-代码与数据混在了一起)
- [3. 一个典型的登录查询示例](#3-一个典型的登录查询示例)
- [4. SQL 注入为什么能够发生](#4-sql-注入为什么能够发生)
- [5. 正确防御：JDBC 参数化查询](#5-正确防御jdbc-参数化查询)
- [6. PreparedStatement 的边界与常见误用](#6-preparedstatement-的边界与常见误用)
- [7. MyBatis 参数语法的关键区别](#7-mybatis-参数语法的关键区别)
- [8. MyBatis-Plus 中的安全做法](#8-mybatis-plus-中的安全做法)
- [9. 完整防御清单](#9-完整防御清单)
- [10. 常见误区](#10-常见误区)
- [11. 双语术语表](#11-双语术语表)
- [12. 官方文档与权威资料](#12-官方文档与权威资料)
- [13. 总结](#13-总结)

## 1. 一句话定义

**SQL 注入（SQL Injection）**是指：应用程序通过**不安全的动态 SQL 构造（unsafe dynamic SQL construction）**，例如字符串拼接或原样文本替换，把**不可信输入（untrusted input）**放进 SQL 文本，导致输入的一部分被数据库解析为 SQL 代码，从而改变程序原本想执行的查询结构或含义。

可以记成下面这个公式：

```text
SQL 注入（SQL Injection）
= 不可信输入（untrusted input）
+ 不安全的动态 SQL 构造（unsafe dynamic SQL construction）
  例如字符串拼接（string concatenation）或文本替换（text substitution）
+ SQL 解析执行（SQL parsing and execution）
```

SQL 注入通常不是 MySQL 数据库本身的漏洞，而是应用程序构造 SQL 的方式存在问题。MySQL 只负责解析并执行应用程序最终发送给它的完整 SQL；它并不知道某段文本最初来自程序源代码，还是来自用户输入。

[MySQL 官方客户端编程安全指南](https://dev.mysql.com/doc/refman/8.4/en/secure-client-programming.html)明确建议 Java JDBC 使用 `PreparedStatement` 和占位符（placeholders）。

## 2. 本质：SQL 代码与数据混在了一起

一条查询中通常同时包含两类内容：

| 类型 | English | 示例 | 谁应该控制 |
| --- | --- | --- | --- |
| SQL 结构 / SQL 代码 | SQL structure / SQL code | `SELECT`、`WHERE`、表名、列名、运算符 | 后端程序 |
| 数据值 | data value | 用户名、邮箱、订单编号、搜索关键词 | 可以来自用户，但必须作为参数绑定 |

安全程序会明确分离二者：

```text
固定 SQL 模板（fixed SQL template）
        +
绑定参数（bound parameters）
        ↓
数据库把 SQL 结构当代码，把参数只当数据
```

不安全程序则把它们先拼成一段文本：

```text
SQL 代码 + 用户输入 + SQL 代码
              ↓
        字符串拼接
              ↓
数据库收到一整段可重新改变含义的 SQL 文本
```

这正是为什么“用户输入只是一个字符串”并不能保证安全：如果这个字符串被放入 SQL 源文本中，它就可能影响引号、运算符、条件表达式或注释。

## 3. 一个典型的登录查询示例

### 3.1 程序原本的意图

假设程序想查询用户名和密码都匹配的用户：

```sql
SELECT *
FROM user
WHERE username = '用户输入的用户名'
  AND password = '用户输入的密码';
```

下面这种 Java 写法使用了**字符串拼接（string concatenation）**，因此是不安全的：

```java
String sql =
    "SELECT * FROM user WHERE username = '" + username
    + "' AND password = '" + password + "'";
```

### 3.2 正常输入时为什么看起来没问题

输入：

```text
username = Tom
password = 123456
```

拼接后的 SQL 是：

```sql
SELECT *
FROM user
WHERE username = 'Tom'
  AND password = '123456';
```

正常数据没有改变 SQL 结构，所以查询看起来能够按预期工作。这也是此类漏洞容易隐藏在代码中的原因：正常功能测试可能全部通过。

### 3.3 特制输入如何改变 SQL 结构

以下示例仅用于解释解析过程。假设用户名输入为：

```text
Tom' OR '1'='1' -- 
```

那么拼接后的 SQL 近似变成：

```sql
SELECT *
FROM user
WHERE username = 'Tom' OR '1'='1' -- ' AND password = 'anything';
```

这里发生了三件事：

1. 输入中的单引号结束了原本的**字符串字面量（string literal）**。
2. `OR '1'='1'` 增加了一个始终为真的**布尔条件（Boolean condition）**。
3. `-- ` 把后面的内容变成了 SQL 注释，使原密码条件不再参与查询。

在 MySQL 中，双连字符注释必须写成 `-- `，即第二个 `-` 后需要有空白或控制字符；参见 [MySQL 8.4 注释语法](https://dev.mysql.com/doc/refman/8.4/en/comments.html)。

如果登录代码只判断“查询是否返回了任意一行”，查询含义就可能被改变。这只是 SQL 注入的一种表现；实际影响还取决于应用逻辑、数据库权限、驱动设置和具体 SQL。

可能的后果包括：

- 绕过认证（authentication bypass）
- 未授权披露（unauthorized disclosure）
- 数据篡改（data tampering）
- 数据删除（data deletion）
- 权限提升（privilege escalation）
- 拒绝服务（denial of service）

SQL 注入并不自动意味着攻击者可以完成以上所有操作。数据库账号的权限和应用架构会限制实际影响，因此**最小权限原则（principle of least privilege）**仍然非常重要。

## 4. SQL 注入为什么能够发生

关键问题不只是“输入中出现了特殊字符”，而是应用把不可信数据放进了 SQL 源代码所在的语法上下文。

```java
"SELECT ... WHERE username = '" + username + "'"
```

应用开发者看到的是：

```text
程序写的 SQL + 用户提供的数据 + 程序写的 SQL
```

MySQL 解析器（MySQL parser）看到的却只是：

```text
一条已经完成拼接的 SQL 语句
```

数据库无法根据文本来源自动建立安全边界。因此，安全边界必须由应用通过**参数化查询（parameterized query）**和**参数绑定（parameter binding）**建立。

正常路径与注入路径可以对比如下：

```text
正常路径
用户输入 → 作为参数值传递 → 数据库按普通数据处理 → SQL 结构不变

危险路径
用户输入 → 拼入 SQL 文本 → 输入参与 SQL 解析 → SQL 结构或含义改变
```

## 5. 正确防御：JDBC 参数化查询

### 5.1 使用 PreparedStatement

在 JDBC（Java Database Connectivity，Java 数据库连接）中，应使用**预编译语句（prepared statement）**与参数占位符：

```java
String sql =
    "SELECT * FROM user WHERE username = ? AND password = ?";

try (PreparedStatement ps = connection.prepareStatement(sql)) {
    ps.setString(1, username);
    ps.setString(2, password);

    try (ResultSet rs = ps.executeQuery()) {
        // 处理查询结果（process the result set）
    }
}
```

这里的核心结构是：

```text
SELECT ... WHERE username = ? AND password = ?
                            ↑                ↑
                      第 1 个参数       第 2 个参数
```

- `?` 是**参数占位符 / 参数标记（parameter placeholder / parameter marker）**。
- JDBC 参数序号从 `1` 开始，而不是从 `0` 开始。
- `setString(1, username)` 把 `username` 绑定到第一个 `?`。
- `setString(2, password)` 把 `password` 绑定到第二个 `?`。
- 对整数、日期等类型，应使用兼容的 setter，例如 `setInt`、`setLong` 或 `setDate`。
- `executeQuery()` 用于返回一个 `ResultSet` 的查询；`INSERT`、`UPDATE`、`DELETE` 通常使用 `executeUpdate()`。

[Oracle JDBC 教程](https://docs.oracle.com/javase/tutorial/jdbc/basics/prepared.html)说明，绑定的客户端数据会被作为参数内容处理，而不是作为 SQL 语句的一部分；当前接口细节可查阅 [Java SE 26 `PreparedStatement` API](https://docs.oracle.com/en/java/javase/26/docs/api/java.sql/java/sql/PreparedStatement.html)。

### 5.2 参数绑定为什么有效

PreparedStatement 的安全价值在于先确定 SQL 结构，再单独提供参数值：

```text
阶段 1：确定 SQL 模板
SELECT * FROM user WHERE username = ? AND password = ?

阶段 2：绑定数据
参数 1 = username
参数 2 = password

阶段 3：执行
数据库/驱动按参数类型处理值，而不是重新把参数当成 SQL 语法解析
```

所以，即使用户名中包含引号、空格或看起来像 SQL 的字符，这些内容仍然只是一个完整的参数值。MySQL 官方说明，带占位符的预处理语句可以防止参数中的引号和分隔字符变成 SQL 注入；参见 [MySQL 8.4 Prepared Statements](https://dev.mysql.com/doc/refman/8.4/en/sql-prepared-statements.html)。

> 准确表述：PreparedStatement 能保护“通过占位符正确绑定的值”，而不是自动保护任意动态拼接的 SQL。

### 5.3 `?` 不需要、也不应该放在引号中

正确：

```java
String sql = "SELECT * FROM user WHERE username = ?";
```

错误：

```java
String sql = "SELECT * FROM user WHERE username = '?'";
```

第二种写法中的 `?` 是字符串字面量的一部分，不是参数标记。MySQL 对参数标记位置的规则可查阅 [`PREPARE` Statement](https://dev.mysql.com/doc/refman/8.4/en/prepare.html)。

### 5.4 生产环境中的密码处理

前面的 `WHERE username = ? AND password = ?` 只是为了对应原始教学示例。真实登录系统不应存储或直接比较明文密码（plaintext password）。更合理的流程是：

1. 用参数化查询按用户名读取用户记录和密码哈希（password hash）。
2. 使用专门的密码哈希库验证用户输入。
3. 存储带盐（salted）的慢速密码哈希，而不是明文密码或普通快速哈希。

```java
String sql =
    "SELECT id, username, password_hash FROM user WHERE username = ?";

try (PreparedStatement ps = connection.prepareStatement(sql)) {
    ps.setString(1, username);

    try (ResultSet rs = ps.executeQuery()) {
        if (rs.next()) {
            String storedHash = rs.getString("password_hash");
            boolean valid = passwordHasher.verify(password, storedHash);
            // passwordHasher 代表项目采用的专业密码哈希组件
        }
    }
}
```

具体密码存储建议参见 [OWASP Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)。

## 6. PreparedStatement 的边界与常见误用

### 6.1 先拼接，再 prepare，仍然不安全

下面的代码虽然创建了 `PreparedStatement`，但用户输入已经进入 SQL 模板：

```java
String sql =
    "SELECT * FROM user WHERE username = '" + username + "'";

PreparedStatement ps = connection.prepareStatement(sql); // 仍然不安全
```

安全性来自“固定 SQL 模板 + 参数绑定”，不是来自类名本身。

### 6.2 占位符通常只能代替数据值，不能代替标识符

**SQL 标识符（SQL identifier）**包括表名和列名。下面这种尝试通常不能把参数当成列名：

```java
String sql = "SELECT * FROM user ORDER BY ?";
PreparedStatement ps = connection.prepareStatement(sql);
ps.setString(1, sortColumn);
```

参数会被当作一个数据值，而不是 SQL 语法中的列标识符。参数标记也不能代替 `ASC`、`DESC` 等关键字。

当业务确实需要动态列名、表名或排序方向时，应使用后端控制的**允许列表（allowlist）**映射：

```java
String orderBy = switch (String.valueOf(requestedSort)) {
    case "name"    -> "username";
    case "created" -> "created_at";
    default        -> "id";
};

String direction = "desc".equalsIgnoreCase(requestedDirection)
    ? "DESC"
    : "ASC";

String sql =
    "SELECT id, username, created_at FROM user ORDER BY "
    + orderBy + " " + direction;
```

这里能够进入 SQL 的每个片段都来自后端固定集合；前端输入只是用来选择集合中的一个选项。不要只检查“看起来不像攻击”，而应明确规定“只允许哪些值”。OWASP 对不能使用绑定变量的表名、列名和排序方向也建议采用允许列表或重新设计查询；参见 [SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html#allow-list-input-validation)。

### 6.3 手工转义不是首选安全边界

**转义（escaping）**处理与字符集、连接配置和具体 SQL 上下文有关，容易出错。不要把以下做法当成主要防御：

- 手工把 `'` 替换成 `''`
- 删除空格或 SQL 关键字
- 用正则表达式猜测“恶意字符串”
- 只依赖 Web 应用防火墙（Web Application Firewall, WAF）

优先使用参数绑定。只有在无法参数化的结构位置才采用严格允许列表，而且拼接内容必须来自后端固定映射。

### 6.4 客户端预编译语句与服务端预编译语句不是核心安全区别

MySQL Connector/J 可以实现**客户端预编译语句（client-side prepared statement）**或**服务端预编译语句（server-side prepared statement）**。这个差异更多涉及驱动实现与性能，不改变应用层的核心要求：SQL 模板保持固定，所有数据值通过 JDBC 参数接口绑定。参见 [MySQL Connector/J JDBC API Implementation Notes](https://dev.mysql.com/doc/connectors/en/connector-j-reference-implementation-notes.html)。

## 7. MyBatis 参数语法的关键区别

MyBatis 是**持久层框架（persistence framework）**，通常也称为 **SQL 映射框架（SQL mapper framework）**。它提供两种外观相似、含义却完全不同的参数语法：`#{}` 与 `${}`。参见 [MyBatis 官方简介](https://mybatis.org/mybatis-3/)。

### 7.1 `#{}`：参数绑定，通常应优先使用

```xml
<select id="findByUsername" resultType="User">
    SELECT id, username
    FROM user
    WHERE username = #{username}
</select>
```

MyBatis 通常会把它处理成类似下面的 JDBC SQL：

```sql
SELECT id, username
FROM user
WHERE username = ?;
```

然后通过 TypeHandler（类型处理器）把值安全地设置到 PreparedStatement 参数中。

```text
#{}
↓
PreparedStatement 参数占位符
↓
参数绑定（parameter binding）
↓
SQL 结构与数据分离
```

因此，值参数通常应使用 `#{}`。不要在它外面手工加单引号：

```xml
<!-- 正确 -->
WHERE username = #{username}

<!-- 错误思路：占位符不应被手工包进字符串字面量 -->
WHERE username = '#{username}'
```

### 7.2 `${}`：原样字符串替换

```xml
SELECT id, username
FROM user
WHERE username = '${username}'
```

`${username}` 是**字符串替换（string substitution）**。MyBatis 会把内容直接放入 SQL 文本，不会替你修改或转义它：

```text
${}
↓
文本替换（textual substitution）
↓
输入直接参与最终 SQL 的形成
↓
内容不可信且缺少严格限制时产生 SQL 注入风险
```

MyBatis 官方文档明确说明，`#{}` 会生成 PreparedStatement 参数，而 `${}` 会插入未经修改的字符串；把用户输入未经限制地提供给 `${}` 会带来 SQL 注入风险。参见：

- [MyBatis Mapper XML Files — String Substitution](https://mybatis.org/mybatis-3/sqlmap-xml.html#String_Substitution)
- [MyBatis XML 映射器 — 字符串替换（中文）](https://mybatis.org/mybatis-3/zh_CN/sqlmap-xml.html#%E5%AD%97%E7%AC%A6%E4%B8%B2%E6%9B%BF%E6%8D%A2)

准确地说，`${}` 并非“出现就必然被攻击”；它是一种直接文本替换机制。当替换内容可被攻击者控制且没有严格约束时，就会形成风险。

### 7.3 为什么 MyBatis 仍然提供 `${}`

参数占位符不能代替表名、列名等 SQL 元数据（SQL metadata），所以某些动态 SQL 结构可能需要文本生成。例如：

```xml
ORDER BY ${columnName}
```

但 `columnName` 绝不能直接使用任意前端输入。更安全的做法是由后端把业务选项映射为固定列名，或者直接使用 MyBatis 的条件结构：

```xml
ORDER BY
<choose>
    <when test="sortBy == 'name'">username</when>
    <when test="sortBy == 'created'">created_at</when>
    <otherwise>id</otherwise>
</choose>
```

这段 XML 让最终列名只能来自三个后端写死的选项，而不是把原始输入直接放入 SQL。

### 7.4 对比表

| 写法 | 机制 | 是否修改 SQL 结构 | 典型用途 | 风险判断 |
| --- | --- | --- | --- | --- |
| JDBC `?` + setter | 参数绑定（parameter binding） | 否 | 数据值 | 正确使用时安全 |
| MyBatis `#{value}` | PreparedStatement 参数 | 否 | 数据值 | 通常首选 |
| MyBatis `${fragment}` | 字符串替换（string substitution） | 可以 | 受控 SQL 元数据或片段 | 任意不可信输入会产生高风险 |
| Java `"..." + input` | 字符串拼接（string concatenation） | 可以 | 不应构造含不可信值的 SQL | 高风险 |

一句话记忆：

```text
#{}：把输入当数据
${}：把输入放进 SQL 文本
```

## 8. MyBatis-Plus 中的安全做法

MyBatis-Plus 的 SQL 生成能力建立在 MyBatis 之上。普通字段值通常会被参数化，但接受列名、SQL 片段或原始字符串的方法仍然需要后端严格控制。

### 8.1 优先使用 Lambda Wrapper

相较于从前端接收字符串列名，优先使用类型安全的列引用：

```java
LambdaQueryWrapper<User> wrapper = Wrappers.lambdaQuery(User.class)
    .eq(User::getUsername, username)
    .orderByDesc(User::getCreatedAt);
```

这里的字段引用来自后端 Java 代码，而数据值 `username` 由框架参数化处理。

### 8.2 不允许前端传递任意 SQL 片段

以下内容都不应直接接收未经验证的前端字符串：

- 表名（table name）
- 列名（column name）
- 排序表达式（sort expression）
- 子查询（subquery）
- `last`、`apply`、`having`、`setSql` 等方法中的 SQL 片段

如果业务允许用户选择排序字段，应使用后端字段映射或枚举，而不是直接把参数放进 SQL。

对于 `apply`，SQL 片段的结构也应由后端固定，并把动态数据放在 `{0}`、`{1}` 等参数位置：

```java
// SQL 结构由后端控制，requestedDate 作为参数值绑定
queryWrapper.apply(
    "DATE(created_at) = {0}",
    requestedDate
);
```

不要这样拼接：

```java
// 不安全：requestedDate 直接进入 SQL 文本
queryWrapper.apply(
    "DATE(created_at) = '" + requestedDate + "'"
);
```

### 8.3 注入检查是补充措施，不是主要边界

MyBatis-Plus 提供自动和手动检查：

```java
Wrappers.query()
    .checkSqlInjection()
    .orderByDesc(frontendField);
```

```java
SqlInjectionUtils.check(frontendField);
```

官方同时建议采用允许列表，因为检查可能无法覆盖所有情况。最安全的做法仍然是不让前端传入 SQL 片段。参见：

- [MyBatis-Plus Data Security Protection](https://baomidou.com/en/guides/security/)
- [MyBatis-Plus 预防安全漏洞（中文）](https://baomidou.com/reference/about-cve/)
- [MyBatis-Plus Wrapper 指南](https://baomidou.com/en/guides/wrapper/)

这些检查属于**纵深防御（defense in depth）**，不能代替参数绑定和后端允许列表。

## 9. 完整防御清单

### 9.1 首要措施

- [ ] 所有数据值都使用参数化查询（parameterized queries）。
- [ ] JDBC 使用 `PreparedStatement`、`?` 和类型匹配的 setter。
- [ ] MyBatis 的数据值使用 `#{}`，不使用 `${}`。
- [ ] 在调用 `prepareStatement()` 之前，不把不可信输入拼入 SQL 模板。

### 9.2 动态 SQL 结构

- [ ] 表名、列名和排序方向由后端固定代码控制。
- [ ] 必须动态选择时使用允许列表（allowlist）或枚举映射。
- [ ] 不允许前端直接提交 SQL 片段。
- [ ] 优先使用 MyBatis `<choose>` 或 MyBatis-Plus Lambda Wrapper 等结构化方式。

### 9.3 纵深防御

- [ ] 数据库应用账号遵循最小权限原则（principle of least privilege）。
- [ ] 不让 Web 应用使用 MySQL 管理员账号。
- [ ] 输入验证（input validation）作为业务约束和补充检测，但不替代参数绑定。
- [ ] 对外返回通用错误信息，避免泄露原始 SQL、表结构或数据库堆栈信息。
- [ ] 日志中避免记录明文密码、令牌或其他秘密。
- [ ] 通过代码审查和自动化测试搜索危险拼接点。
- [ ] 使用受维护的数据库驱动、ORM 和框架版本。

### 9.4 代码审查时重点搜索

```text
createStatement(
executeQuery("..." +
executeUpdate("..." +
prepareStatement("..." +
${
.last(
.apply(
.having(
.setSql(
ORDER BY " +
```

命中这些模式不一定代表存在漏洞，但应进一步确认其中是否包含不可信输入，以及是否经过固定允许列表映射。

## 10. 常见误区

### 误区 1：只要使用 PreparedStatement 就一定安全

不正确。只有通过占位符绑定数据值，并保持 SQL 模板不受不可信输入影响时才安全。先拼接再创建 `PreparedStatement` 仍然可能被注入。

### 误区 2：只过滤单引号就能防御 SQL 注入

不正确。注入上下文不只有字符串，也可能涉及数字、标识符、排序表达式或其他 SQL 片段。黑名单（blocklist）还可能遗漏编码、数据库方言和语法组合。

### 误区 3：输入验证可以代替参数化查询

不正确。输入验证用于保证业务数据格式，例如订单编号只能是正整数；参数化查询用于建立代码与数据的安全边界。两者可以同时使用，但职责不同。

### 误区 4：`${}` 一定不能使用

表述过于绝对。`${}` 是直接字符串替换，因此不能接收任意不可信输入。某些受控 SQL 元数据场景可以使用，但替换值必须来自后端固定允许列表；如果能使用结构化方式，应优先避免 `${}`。

### 误区 5：`#{column}` 可以安全地代替列名

不正确。`#{}` 绑定的是数据值，不是 SQL 标识符。列名、表名或关键字应由后端代码选择。

### 误区 6：存储过程天然不会被注入

不正确。安全构造且使用参数的存储过程可以防御注入；如果存储过程内部继续拼接动态 SQL，仍可能出现同类问题。

### 误区 7：参数化以后就不需要最小权限

不正确。参数化是主要预防措施，最小权限则限制其他缺陷或配置错误造成的损害范围，两者属于不同层次的防御。

## 11. 双语术语表

| 中文 | English | 简要说明 |
| --- | --- | --- |
| SQL 注入 | SQL injection | 不可信输入改变 SQL 结构或含义的漏洞类型 |
| 不可信输入 | untrusted input | 来自用户、请求、消息、文件或外部系统，尚未建立信任边界的数据 |
| 不安全的动态 SQL 构造 | unsafe dynamic SQL construction | 通过拼接或文本替换等方式让不可信输入进入 SQL 文本 |
| 字符串拼接 | string concatenation | 把多个字符串连接为一段 SQL 文本 |
| 参数化查询 | parameterized query | SQL 模板与数据参数分开提供的查询方式 |
| 预编译语句 | prepared statement | 表示预先准备或预编译并可重复执行的 SQL 语句，通常包含可绑定参数；JDBC 对应 `PreparedStatement` |
| 参数绑定 | parameter binding | 把运行时数据设置到参数位置的过程 |
| 参数占位符 / 参数标记 | parameter placeholder / parameter marker | SQL 中等待绑定值的位置，例如 `?` |
| 输入参数 | input parameter / IN parameter | 传给 SQL 语句的数据参数 |
| 字符串替换 | string substitution | 把文本直接替换进 SQL，例如 MyBatis `${}` |
| SQL 结构 | SQL structure | 关键字、表达式、表名、列名、运算符等组成的语法结构 |
| SQL 标识符 | SQL identifier | 表、列、数据库等对象的名称 |
| 字符串字面量 | string literal | SQL 中由引号包围的文本值 |
| 动态 SQL | dynamic SQL | 在运行时根据条件生成部分或全部 SQL 的方式 |
| 对象关系映射 | Object-Relational Mapping, ORM | 在程序对象和关系数据库之间进行映射的技术 |
| 持久层框架 | persistence framework | 帮助应用访问、保存和映射持久化数据的框架 |
| SQL 映射框架 | SQL mapper framework | 将 SQL、参数和查询结果映射到程序对象的框架 |
| 类型处理器 | type handler | 在 Java 类型和 JDBC/SQL 类型之间转换并设置参数的组件 |
| 允许列表 | allowlist | 只接受明确列出的安全选项 |
| 黑名单 / 阻止列表 | blocklist | 尝试拒绝已知危险模式，容易遗漏未知形式 |
| 输入验证 | input validation | 验证数据是否满足业务格式、范围和语义要求 |
| 转义 | escaping | 按特定语法规则处理具有特殊意义的字符 |
| 最小权限原则 | principle of least privilege | 只授予账号完成任务所需的最低权限 |
| 纵深防御 | defense in depth | 使用多层相互补充的安全控制 |
| 认证绕过 | authentication bypass | 绕过身份验证流程获得未授权访问 |
| 密码哈希 | password hash | 使用专用单向密码哈希算法生成的验证值 |
| 盐 | salt | 密码哈希时加入的每用户唯一随机值 |

## 12. 官方文档与权威资料

### MySQL 官方文档

- [MySQL 8.4 — Client Programming Security Guidelines](https://dev.mysql.com/doc/refman/8.4/en/secure-client-programming.html)  
  客户端程序安全、PreparedStatement、占位符、最小权限与错误信息处理建议。
- [MySQL 8.4 — Prepared Statements](https://dev.mysql.com/doc/refman/8.4/en/sql-prepared-statements.html)  
  预处理语句、占位符、防 SQL 注入及重复执行的解析开销。
- [MySQL 8.4 — `PREPARE` Statement](https://dev.mysql.com/doc/refman/8.4/en/prepare.html)  
  参数标记的位置和限制；参数不能代替关键字或标识符。
- [MySQL 8.4 — Comments](https://dev.mysql.com/doc/refman/8.4/en/comments.html)  
  `#`、`-- ` 和 `/* ... */` 注释语法。
- [MySQL Connector/J — JDBC API Implementation Notes](https://dev.mysql.com/doc/connectors/en/connector-j-reference-implementation-notes.html)  
  Connector/J 的客户端和服务端 prepared statement 实现说明。

### Java / JDBC 官方文档

- [Oracle Java Tutorial — Using Prepared Statements](https://docs.oracle.com/javase/tutorial/jdbc/basics/prepared.html)  
  JDBC 参数占位符、setter、复用方式和 SQL 注入防御。该教程面向 JDK 8，但核心 JDBC 用法仍适用。
- [Java SE 26 API — `PreparedStatement`](https://docs.oracle.com/en/java/javase/26/docs/api/java.sql/java/sql/PreparedStatement.html)  
  当前接口定义、setter、参数序号和执行方法。
- [Java SE 26 API — `Connection`](https://docs.oracle.com/en/java/javase/26/docs/api/java.sql/java/sql/Connection.html)  
  `prepareStatement(String sql)` 和 `?` 输入参数占位符。

### MyBatis 官方文档

- [MyBatis 3 — Introduction](https://mybatis.org/mybatis-3/)  
  MyBatis 作为持久层框架的官方定位。
- [MyBatis 3 — Mapper XML Files](https://mybatis.org/mybatis-3/sqlmap-xml.html)  
  `#{}` 参数映射、PreparedStatement 以及 `${}` 字符串替换的官方说明。
- [MyBatis 3 — XML 映射器（中文）](https://mybatis.org/mybatis-3/zh_CN/sqlmap-xml.html)  
  上述内容的官方中文版本。
- [MyBatis 3 — Dynamic SQL](https://mybatis.org/mybatis-3/dynamic-sql.html)  
  `<if>`、`<choose>`、`<where>`、`<foreach>` 等动态 SQL 结构。

### MyBatis-Plus 官方文档

- [MyBatis-Plus — Data Security Protection](https://baomidou.com/en/guides/security/)  
  自动和手动 SQL 注入检查，以及不允许前端传入 SQL 片段的建议。
- [MyBatis-Plus — 预防安全漏洞（中文）](https://baomidou.com/reference/about-cve/)  
  表结构部分、字段值、允许列表及 Wrapper 的安全边界。
- [MyBatis-Plus — Wrapper](https://baomidou.com/en/guides/wrapper/)  
  条件构造器、Lambda Wrapper 与直接 SQL 片段相关方法。

### 权威安全指南

- [OWASP — SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)  
  参数化查询、允许列表、最小权限和纵深防御。
- [OWASP — Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)  
  密码哈希、盐和密码存储算法建议。

## 13. 总结

最核心的一句话是：

> **SQL 注入（SQL Injection）= 不可信输入因为进入 SQL 文本而被当成 SQL 代码解析，从而改变程序原本想执行的 SQL。**

最重要的防御原则是：

1. 数据值使用参数化查询和参数绑定。
2. JDBC 使用 `PreparedStatement`、`?` 和 setter。
3. MyBatis 中值参数优先使用 `#{}`；不要让任意用户输入进入 `${}`。
4. 列名、表名和排序方向使用后端允许列表映射。
5. 结合最小权限、输入验证、安全错误处理和代码审查进行纵深防御。

最后可以用下面这组对比快速记忆：

```text
不安全：SQL 代码 + 用户输入 + SQL 代码 → 拼成一条可被改变的 SQL

安全：  固定 SQL 模板 + 独立参数值       → SQL 代码与数据分离
```
