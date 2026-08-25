# MySQL 8.0 权限管理代码汇总

本文整理 MySQL 8.0 中常用的账号权限查询、权限回收、角色检查和账号删除操作。

> 示例账号为 `'user2'@'%'`。MySQL 账号由 `User` 和 `Host` 共同确定，执行时必须确保两者都匹配。

## 一、查询账号和权限

### 1. 查看 `mysql.user` 表的全部列名

```sql
SELECT COLUMN_NAME
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = 'mysql'
  AND TABLE_NAME = 'user';
```

作用：查看 `mysql.user` 的字段结构，例如：

- `User`、`Host`：账号标识；
- `authentication_string`：认证凭据的哈希信息；
- `Select_priv`、`Insert_priv`、`Update_priv`、`Delete_priv`：全局 DML 权限；
- `Create_priv`、`Drop_priv`、`Alter_priv`：全局 DDL 权限；
- `Grant_priv`：转授权限；
- `password_expired`：密码是否过期；
- `account_locked`：账号是否锁定。

常见拼写错误：

```sql
-- 错误：FORM 应为 FROM
SELECT COLUMN_NAME FORM INFORMATION_SCHEMA.COLUMNS;
```

### 2. 查询服务器中的账号

```sql
SELECT User, Host
FROM mysql.user;
```

### 3. 查看账号的授权语句

```sql
SHOW GRANTS FOR 'user2'@'%';
```

如果结果只有：

```sql
GRANT USAGE ON *.* TO `user2`@`%`;
```

表示该账号存在，但没有直接授予的普通权限。`USAGE` 是“无权限”的同义表示，并不代表一项实际的业务权限，也不能单独证明账号一定能够成功登录。

### 4. 查看账号的定义和登录属性

```sql
SHOW CREATE USER 'user2'@'%';
```

该语句可用于检查认证插件、SSL 要求、密码过期策略和账号锁定状态等属性。

## 二、分层查询底层授权表

### 1. 全局静态权限

```sql
SELECT *
FROM mysql.user
WHERE User = 'user2' AND Host = '%';
```

### 2. 全局动态权限

```sql
SELECT *
FROM mysql.global_grants
WHERE User = 'user2' AND Host = '%';
```

### 3. 数据库级权限

```sql
SELECT *
FROM mysql.db
WHERE User = 'user2' AND Host = '%';
```

### 4. 表级权限

```sql
SELECT *
FROM mysql.tables_priv
WHERE User = 'user2' AND Host = '%';
```

### 5. 列级权限

```sql
SELECT *
FROM mysql.columns_priv
WHERE User = 'user2' AND Host = '%';
```

### 6. 存储过程和函数权限

```sql
SELECT *
FROM mysql.procs_priv
WHERE User = 'user2' AND Host = '%';
```

### 7. `PROXY` 权限

```sql
SELECT *
FROM mysql.proxies_priv
WHERE User = 'user2' AND Host = '%';
```

### 8. 角色授权关系

```sql
SELECT *
FROM mysql.role_edges
WHERE TO_USER = 'user2' AND TO_HOST = '%';
```

### 9. 默认角色

```sql
SELECT *
FROM mysql.default_roles
WHERE USER = 'user2' AND HOST = '%';
```

> 底层授权表适合排查权限来源。日常授权和回收应优先使用 `GRANT`、`REVOKE` 等账号管理语句，不建议直接修改 `mysql.*` 授权表。

## 三、按授权层级回收权限

### 1. 回收全局层级权限

```sql
REVOKE ALL PRIVILEGES ON *.*
FROM 'user2'@'%';
```

这里的 `ON *.*` 明确指定全局层级。它不会自动清除原先单独授予的库级、表级、列级或存储例程级权限。

### 2. 回收全局层级的 `GRANT OPTION`

```sql
REVOKE GRANT OPTION ON *.*
FROM 'user2'@'%';
```

### 3. 回收数据库级权限

```sql
REVOKE ALL PRIVILEGES ON account.*
FROM 'user2'@'%';
```

### 4. 回收表级权限

```sql
REVOKE ALL PRIVILEGES ON account.course
FROM 'user2'@'%';

REVOKE ALL PRIVILEGES ON hospital.doctor
FROM 'user2'@'%';
```

### 5. 回收列级权限

列级权限通常需要明确写出权限类型和列名：

```sql
REVOKE SELECT (course_name), UPDATE (course_name)
ON account.course
FROM 'user2'@'%';
```

### 6. 回收存储过程权限

```sql
REVOKE EXECUTE ON PROCEDURE database_name.procedure_name
FROM 'user2'@'%';
```

### 7. 回收存储函数权限

```sql
REVOKE EXECUTE ON FUNCTION database_name.function_name
FROM 'user2'@'%';
```

## 四、一条语句回收全部直接权限

如果目标是回收该账号在各个层级被直接授予的全部权限，推荐使用：

```sql
REVOKE ALL PRIVILEGES, GRANT OPTION
FROM 'user2'@'%';
```

它与带 `ON *.*` 的写法并不相同：

| 语句 | 作用范围 |
| --- | --- |
| `REVOKE ALL PRIVILEGES ON *.* FROM ...` | 回收 `*.*` 全局层级上的权限 |
| `REVOKE ALL PRIVILEGES, GRANT OPTION FROM ...` | 回收该账号各层级的直接权限和转授权限 |

该语句不会删除账号本身。账号通过角色获得的权限还需要单独检查并回收角色。

## 五、回收角色

### 1. 查看账号被授予的角色

```sql
SHOW GRANTS FOR 'user2'@'%';
```

如果结果包含类似内容：

```sql
GRANT `role_name`@`%` TO `user2`@`%`;
```

可以执行：

```sql
REVOKE 'role_name'@'%'
FROM 'user2'@'%';
```

如果有多个角色，可以一起回收：

```sql
REVOKE 'role_a'@'%', 'role_b'@'%'
FROM 'user2'@'%';
```

## 六、彻底清空账号权限的推荐流程

```sql
-- 1.清理前查看直接授权和角色
SHOW GRANTS FOR 'user2'@'%';

-- 2.回收各层级的全部直接权限以及 GRANT OPTION
REVOKE ALL PRIVILEGES, GRANT OPTION
FROM 'user2'@'%';

-- 3.再次检查是否还存在角色授权
SHOW GRANTS FOR 'user2'@'%';

-- 4.如果存在角色，按实际角色名称单独回收
-- REVOKE 'role_name'@'%' FROM 'user2'@'%';

-- 5.最终校验
SHOW GRANTS FOR 'user2'@'%';
```

如果最终只显示：

```sql
GRANT USAGE ON *.* TO `user2`@`%`;
```

通常表示账号仍然存在，但已经没有直接授予的普通业务权限。还应结合角色、强制角色以及账号环境进行最终安全核验。

## 七、删除账号

如果目标不是保留空权限账号，而是彻底删除账号：

```sql
DROP USER 'user2'@'%';
```

删除前可以先确认账号：

```sql
SELECT User, Host
FROM mysql.user
WHERE User = 'user2' AND Host = '%';
```

> `DROP USER` 会删除账号及其权限，但不会自动删除以该账号为 `DEFINER` 的视图、触发器、事件或存储程序。删除前应检查相关对象，避免对象之后因定义者账号不存在而执行失败。

## 八、`FLUSH PRIVILEGES` 的正确使用

使用以下语句管理账号或权限时，修改会由 MySQL 自动加载并生效：

```sql
GRANT ...;
REVOKE ...;
CREATE USER ...;
ALTER USER ...;
DROP USER ...;
```

因此，正常执行上述语句后不需要：

```sql
FLUSH PRIVILEGES;
```

只有直接修改 `mysql.*` 授权表等特殊场景下，才可能需要执行 `FLUSH PRIVILEGES`。直接修改授权表本身并不推荐。

## 九、重要注意事项

1. MySQL 账号是 `'User'@'Host'` 的组合，`'user2'@'%'` 和 `'user2'@'localhost'` 是两个不同账号。
2. `ALL PRIVILEGES` 通常不包含 `GRANT OPTION`；需要时应明确回收 `GRANT OPTION`。
3. 带 `ON *.*` 的 `REVOKE ALL` 只处理全局层级；不带 `ON` 的特殊清空语法可以回收各层级直接权限。
4. 角色授权可能继续提供有效权限，不能只看普通权限表。
5. `USAGE` 表示没有权限，不是一项可以执行数据库业务操作的权限。
6. `Query OK, 0 rows affected` 不能用于证明账号原来没有权限，应通过 `SHOW GRANTS` 校验。
7. 账号能否登录还受密码、Host 匹配、账号锁定、认证插件、SSL 要求和服务器网络配置等因素影响。
8. 正常使用 `GRANT`、`REVOKE` 等语句后不需要执行 `FLUSH PRIVILEGES`。

## 十、最简实用版

```sql
-- 查看账号当前权限
SHOW GRANTS FOR 'user2'@'%';

-- 回收全部直接权限和转授权限
REVOKE ALL PRIVILEGES, GRANT OPTION
FROM 'user2'@'%';

-- 检查是否还有角色，并确认最终状态
SHOW GRANTS FOR 'user2'@'%';

-- 如果不再需要该账号，可将其删除
-- DROP USER 'user2'@'%';
```

## 参考资料

- [MySQL 8.0 Reference Manual：Adding Accounts, Assigning Privileges, and Dropping Accounts](https://dev.mysql.com/doc/refman/8.0/en/creating-accounts.html)
- [MySQL 8.0 Reference Manual：Privileges Provided by MySQL](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html)
- [MySQL 8.0 Reference Manual：REVOKE Statement](https://dev.mysql.com/doc/refman/8.0/en/revoke.html)
- [MySQL 8.0 Reference Manual：Grant Tables](https://dev.mysql.com/doc/refman/8.0/en/grant-tables.html)

## 十一、使用 `SELECT` 查询 `user2` 和 `root` 的权限

下面给出不用 `SHOW GRANTS`、改用 `SELECT` 查询权限的示例。需要注意：`SELECT` 查询系统表可以查看权限数据，但与 `SHOW GRANTS` 的汇总展示方式并不完全相同。

### 1. 查询 `user2` 账号及 Host

```sql
SELECT User, Host
FROM mysql.user
WHERE User = 'user2';
```

如果已知账号是 `'user2'@'%'`，可以精确查询：

```sql
SELECT User, Host
FROM mysql.user
WHERE User = 'user2'
  AND Host = '%';
```

### 2. 查询 `user2` 的常见全局静态权限

```sql
SELECT
    User,
    Host,
    Select_priv,
    Insert_priv,
    Update_priv,
    Delete_priv,
    Create_priv,
    Drop_priv,
    Grant_priv
FROM mysql.user
WHERE User = 'user2'
  AND Host = '%';
```

其中：

- `Y` 表示拥有该项全局权限；
- `N` 表示没有该项全局权限；
- `Grant_priv` 对应是否具有全局层级的 `GRANT OPTION`。

### 3. 使用 `information_schema` 查询 `user2` 的各层级权限

```sql
SELECT
    'GLOBAL' AS privilege_level,
    PRIVILEGE_TYPE,
    '*.*' AS object_name,
    IS_GRANTABLE
FROM information_schema.USER_PRIVILEGES
WHERE GRANTEE = '''user2''@''%'''

UNION ALL

SELECT
    'DATABASE',
    PRIVILEGE_TYPE,
    CONCAT(TABLE_SCHEMA, '.*'),
    IS_GRANTABLE
FROM information_schema.SCHEMA_PRIVILEGES
WHERE GRANTEE = '''user2''@''%'''

UNION ALL

SELECT
    'TABLE',
    PRIVILEGE_TYPE,
    CONCAT(TABLE_SCHEMA, '.', TABLE_NAME),
    IS_GRANTABLE
FROM information_schema.TABLE_PRIVILEGES
WHERE GRANTEE = '''user2''@''%'''

UNION ALL

SELECT
    'COLUMN',
    PRIVILEGE_TYPE,
    CONCAT(TABLE_SCHEMA, '.', TABLE_NAME, '.', COLUMN_NAME),
    IS_GRANTABLE
FROM information_schema.COLUMN_PRIVILEGES
WHERE GRANTEE = '''user2''@''%''';
```

这里 `GRANTEE` 字段中的账号形式类似：

```text
'user2'@'%'
```

因此 SQL 字符串中需要使用两个单引号表示一个实际的单引号。

### 4. 查询 `root` 账号及 Host

`root` 不一定是 `'root'@'%'`，很多 MySQL 安装默认使用 `'root'@'localhost'`，所以应先确认 Host：

```sql
SELECT User, Host
FROM mysql.user
WHERE User = 'root';
```

如果结果为：

```text
root    localhost
```

则该账号的完整形式是：

```text
'root'@'localhost'
```

### 5. 查询 `root` 的常见全局静态权限

如果 `root` 的 Host 是 `localhost`：

```sql
SELECT
    User,
    Host,
    Select_priv,
    Insert_priv,
    Update_priv,
    Delete_priv,
    Create_priv,
    Drop_priv,
    Grant_priv
FROM mysql.user
WHERE User = 'root'
  AND Host = 'localhost';
```

如果暂时不确定 `root` 的 Host，可以查询所有名为 `root` 的账号：

```sql
SELECT *
FROM mysql.user
WHERE User = 'root';
```

### 6. 使用 `information_schema` 查询 `'root'@'localhost'` 的各层级权限

```sql
SELECT
    'GLOBAL' AS privilege_level,
    PRIVILEGE_TYPE,
    '*.*' AS object_name,
    IS_GRANTABLE
FROM information_schema.USER_PRIVILEGES
WHERE GRANTEE = '''root''@''localhost'''

UNION ALL

SELECT
    'DATABASE',
    PRIVILEGE_TYPE,
    CONCAT(TABLE_SCHEMA, '.*'),
    IS_GRANTABLE
FROM information_schema.SCHEMA_PRIVILEGES
WHERE GRANTEE = '''root''@''localhost'''

UNION ALL

SELECT
    'TABLE',
    PRIVILEGE_TYPE,
    CONCAT(TABLE_SCHEMA, '.', TABLE_NAME),
    IS_GRANTABLE
FROM information_schema.TABLE_PRIVILEGES
WHERE GRANTEE = '''root''@''localhost'''

UNION ALL

SELECT
    'COLUMN',
    PRIVILEGE_TYPE,
    CONCAT(TABLE_SCHEMA, '.', TABLE_NAME, '.', COLUMN_NAME),
    IS_GRANTABLE
FROM information_schema.COLUMN_PRIVILEGES
WHERE GRANTEE = '''root''@''localhost''';
```

如果实际 `root` 账号的 Host 不是 `localhost`，应把上面所有 `localhost` 替换成查询到的真实 Host。

### 7. `SHOW GRANTS` 与 `SELECT` 的区别

```sql
-- MySQL 直接汇总并显示授权语句
SHOW GRANTS FOR 'user2'@'%';

-- 使用 SELECT 查看权限系统表中的数据
SELECT *
FROM mysql.user
WHERE User = 'user2' AND Host = '%';
```

`SHOW GRANTS` 更适合快速确认某个账号最终有哪些授权；`SELECT` 更适合学习或排查权限具体存储在哪些系统表中。数据库级、表级、列级、动态权限和角色权限可能分布在不同系统表里，因此只查询 `mysql.user` 不能代表账号的全部权限。

