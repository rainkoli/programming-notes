# MySQL 8.0 核心系统数据库表速查（实用版）

> 适用范围：MySQL 8.0  
> 整理目标：面向日常开发、排障、权限检查、慢 SQL 优化、事务/锁分析和面试复习。  
> 核心思路：不机械背所有系统表，只掌握最常用的系统库、核心表/视图、关键字段和排查链路。

---

## 一、先建立整体认知：MySQL 8.0 四大核心系统库

| 系统库 | 核心作用 | 最常见使用场景 | 学习优先级 |
|---|---|---|---|
| `mysql` | 账号、权限、角色、认证等服务器级配置 | 用户登录失败、权限过大、授权排查 | ★★★★★ |
| `information_schema` | 数据库对象元数据 | 查库、表、字段、索引、约束、事务 | ★★★★★ |
| `performance_schema` | MySQL 运行时性能监控底层数据 | 会话、慢 SQL、锁、I/O、等待事件 | ★★★★★ |
| `sys` | 对 `performance_schema` 等数据做友好封装 | 日常性能排障、热点表、索引分析 | ★★★★★ |

可以把它们理解成：

```text
mysql
└── 谁能访问、拥有什么权限

information_schema
└── MySQL 里面“有什么对象、对象长什么样”

performance_schema
└── MySQL “现在正在干什么、历史上干得怎么样”

sys
└── 把 performance_schema 的底层数据整理成人更容易看的结果
```

---

# 二、`mysql`：账号、权限、角色

## 1. `mysql.user` —— 全局账号核心表 ★★★★★

### 核心筛选/查看字段

1. `Host`：账号允许从哪些主机来源连接；与 `User` 一起确定 MySQL 账号。
2. `User`：数据库用户名。
3. `plugin`：该账号使用的认证插件。
4. `authentication_string`：认证凭据相关信息，通常为密码认证产生的哈希/认证数据，不保存明文密码。
5. `Select_priv / Insert_priv / Update_priv / Delete_priv`：全局 DML 权限。
6. `Create_priv / Drop_priv / Alter_priv / Index_priv`：全局 DDL / 索引相关权限。
7. `Grant_priv`：是否可将自己拥有的相应权限继续授予其他账号。
8. `Super_priv`：传统 `SUPER` 静态权限标识；MySQL 8.0 中大量职责已拆分为动态权限。
9. `password_expired`：密码是否已过期。
10. `password_last_changed`：密码最近修改时间。
11. `password_lifetime`：账号密码有效期（天）。
12. `account_locked`：账号是否被锁定。
13. `max_connections`：**每小时最大连接次数**，不是并发连接数。
14. `max_user_connections`：该账号允许的**最大并发连接数**。
15. `max_questions`：每小时最大查询次数。
16. `max_updates`：每小时最大更新次数。

> **用途：** 排查账号登录失败、全局权限过大、账号锁定、密码过期、连接资源限制。

### 高频查询

```sql
SELECT
    Host,
    User,
    plugin,
    account_locked,
    password_expired,
    password_last_changed,
    max_connections,
    max_user_connections
FROM mysql.user;
```

### 重要注意

- `mysql.user` 中的权限属于**静态全局权限**，对整个 MySQL Server 生效。
- MySQL 8.0 的动态权限还需要查看 `mysql.global_grants`。
- 实际授权应优先使用 `CREATE USER`、`ALTER USER`、`GRANT`、`REVOKE`。
- 不建议直接 `UPDATE / INSERT / DELETE mysql.user`。

---

## 2. `mysql.global_grants` —— 动态全局权限 ★★★★★

MySQL 8.0 的重点表。

### 核心字段

1. `USER`：用户名。
2. `HOST`：主机范围。
3. `PRIV`：动态权限名称。
4. `WITH_GRANT_OPTION`：该动态权限是否可以继续授权给其他账号。

常见动态权限包括：

- `SYSTEM_VARIABLES_ADMIN`
- `CONNECTION_ADMIN`
- `BACKUP_ADMIN`
- `BINLOG_ADMIN`
- `REPLICATION_APPLIER`
- `REPLICATION_SLAVE_ADMIN`

> **用途：** 检查 MySQL 8.0 管理类高权限。仅查看 `mysql.user` 不足以完整判断一个账号的全局权限。

### 高频查询

```sql
SELECT *
FROM mysql.global_grants
ORDER BY USER, HOST, PRIV;
```

---

## 3. `mysql.db` —— 数据库级权限表 ★★★★★

### 核心字段

1. `Host`：来源主机。
2. `Db`：权限生效的数据库。
3. `User`：用户名。
4. `Select_priv`
5. `Insert_priv`
6. `Update_priv`
7. `Delete_priv`
8. `Create_priv`
9. `Drop_priv`
10. `Alter_priv`
11. `Index_priv`
12. `Grant_priv`
13. `Create_view_priv`
14. `Show_view_priv`
15. `Execute_priv`
16. `Trigger_priv`
17. `Event_priv`

> **用途：** 查看某账号在**某个数据库范围**内拥有的权限。

### 关键理解

不要简单理解成：

> `mysql.db` 权限“优先级低于” `mysql.user`

更准确的理解是：

```text
mysql.user
└── 全局静态权限：*.*

mysql.db
└── 数据库级权限：database.*

mysql.tables_priv
└── 表级权限：database.table

mysql.columns_priv
└── 列级权限：database.table.column
```

MySQL 会按照权限作用域组合账号权限。

---

## 4. `mysql.tables_priv` —— 表级权限 ★★★★

### 核心字段

1. `Host`
2. `Db`
3. `User`
4. `Table_name`
5. `Grantor`
6. `Timestamp`
7. `Table_priv`
8. `Column_priv`

> **用途：** 某个账号只能访问某库中的特定表时，用于查看底层表级授权信息。

---

## 5. `mysql.columns_priv` —— 列级权限 ★★★

### 核心字段

1. `Host`
2. `Db`
3. `User`
4. `Table_name`
5. `Column_name`
6. `Column_priv`

> **用途：** 极细粒度权限控制，例如只允许读取某张表的指定列。

---

## 6. `mysql.role_edges` —— 角色授权关系 ★★★★

### 核心字段

1. `FROM_HOST`
2. `FROM_USER`
3. `TO_HOST`
4. `TO_USER`
5. `WITH_ADMIN_OPTION`

可以理解为：

```text
角色
  ↓
授予
  ↓
用户 / 另一个角色
```

> **用途：** 排查用户为什么拥有某项间接权限、分析 Role 权限继承关系。

---

## 7. `mysql.default_roles` —— 用户默认角色 ★★★★

### 核心字段

1. `HOST`
2. `USER`
3. `DEFAULT_ROLE_HOST`
4. `DEFAULT_ROLE_USER`

> **用途：** 判断用户登录后默认启用哪些角色。

---

## 8. `mysql.password_history` —— 密码历史 ★★

用于 MySQL 密码复用限制。

> **用途：** DBA 密码策略审计。普通业务开发了解即可。

---

# 三、`information_schema`：数据库对象元数据

## 1. `information_schema.SCHEMATA` —— 数据库列表 ★★★★★

### 核心字段

1. `SCHEMA_NAME`：数据库名。
2. `DEFAULT_CHARACTER_SET_NAME`：默认字符集。
3. `DEFAULT_COLLATION_NAME`：默认排序规则。
4. `DEFAULT_ENCRYPTION`：Schema 默认加密设置。

> **用途：** 查看有哪些数据库以及字符集/排序规则配置。

### 高频查询

```sql
SELECT
    SCHEMA_NAME,
    DEFAULT_CHARACTER_SET_NAME,
    DEFAULT_COLLATION_NAME
FROM information_schema.SCHEMATA;
```

---

## 2. `information_schema.TABLES` —— 表级元数据 ★★★★★

这是日常非常重要的一张元数据表。

### 核心字段

1. `TABLE_SCHEMA`：数据库名。
2. `TABLE_NAME`：表名。
3. `TABLE_TYPE`：`BASE TABLE`、`VIEW` 等。
4. `ENGINE`：存储引擎，如 `InnoDB`。
5. `TABLE_ROWS`：估算的行数（InnoDB 中通常不是精确值）。
6. `AVG_ROW_LENGTH`：平均行长度。
7. `DATA_LENGTH`：数据部分大小。
8. `INDEX_LENGTH`：索引大小。
9. `DATA_FREE`：已分配但未使用空间等相关值。
10. `AUTO_INCREMENT`：下一个自增值。
11. `CREATE_TIME`：创建时间。
12. `UPDATE_TIME`：更新时间（含义受引擎等因素影响）。
13. `TABLE_COLLATION`：表排序规则。
14. `TABLE_COMMENT`：表注释。

> **用途：** 查大表、存储引擎、表大小、字符集、表注释。

### 高频查询：按表空间大小排序

```sql
SELECT
    TABLE_SCHEMA,
    TABLE_NAME,
    TABLE_ROWS,
    DATA_LENGTH,
    INDEX_LENGTH,
    DATA_LENGTH + INDEX_LENGTH AS total_bytes
FROM information_schema.TABLES
WHERE TABLE_SCHEMA NOT IN (
    'mysql',
    'information_schema',
    'performance_schema',
    'sys'
)
ORDER BY total_bytes DESC;
```

---

## 3. `information_schema.COLUMNS` —— 字段元数据 ★★★★★

### 核心字段

1. `TABLE_SCHEMA`
2. `TABLE_NAME`
3. `COLUMN_NAME`
4. `ORDINAL_POSITION`：字段在表中的顺序。
5. `COLUMN_DEFAULT`
6. `IS_NULLABLE`
7. `DATA_TYPE`：基础类型，如 `varchar`、`bigint`。
8. `CHARACTER_MAXIMUM_LENGTH`
9. `NUMERIC_PRECISION`
10. `NUMERIC_SCALE`
11. `COLUMN_TYPE`：完整列类型，例如 `varchar(100)`、`bigint unsigned`。
12. `COLUMN_KEY`：`PRI / UNI / MUL` 等。
13. `EXTRA`：例如 `auto_increment`、生成列等额外信息。
14. `COLUMN_COMMENT`

> **用途：** 自动生成数据字典、检查字段类型、NULL、默认值、主键等。

---

## 4. `information_schema.STATISTICS` —— 索引元数据 ★★★★★

### 核心字段

1. `TABLE_SCHEMA`
2. `TABLE_NAME`
3. `NON_UNIQUE`：`0` 表示唯一索引。
4. `INDEX_SCHEMA`
5. `INDEX_NAME`
6. `SEQ_IN_INDEX`：字段在联合索引中的顺序。
7. `COLUMN_NAME`
8. `COLLATION`
9. `CARDINALITY`：基数估计。
10. `SUB_PART`：前缀索引长度。
11. `NULLABLE`
12. `INDEX_TYPE`：如 `BTREE`、`FULLTEXT`。
13. `IS_VISIBLE`：索引是否可见。
14. `EXPRESSION`：函数/表达式索引相关表达式。

> **用途：** 分析索引结构、联合索引顺序、唯一性、索引基数和可见性。

### 高频查询

```sql
SELECT
    TABLE_SCHEMA,
    TABLE_NAME,
    INDEX_NAME,
    NON_UNIQUE,
    SEQ_IN_INDEX,
    COLUMN_NAME,
    CARDINALITY,
    INDEX_TYPE,
    IS_VISIBLE
FROM information_schema.STATISTICS
WHERE TABLE_SCHEMA = 'your_database'
ORDER BY TABLE_NAME, INDEX_NAME, SEQ_IN_INDEX;
```

---

## 5. `information_schema.TABLE_CONSTRAINTS` —— 约束总览 ★★★★

### 核心字段

1. `CONSTRAINT_SCHEMA`
2. `CONSTRAINT_NAME`
3. `TABLE_SCHEMA`
4. `TABLE_NAME`
5. `CONSTRAINT_TYPE`

常见 `CONSTRAINT_TYPE`：

- `PRIMARY KEY`
- `UNIQUE`
- `FOREIGN KEY`
- `CHECK`

> **用途：** 快速查看某表有哪些约束。

---

## 6. `information_schema.KEY_COLUMN_USAGE` —— 键/约束字段关系 ★★★★★

### 核心字段

1. `CONSTRAINT_SCHEMA`
2. `CONSTRAINT_NAME`
3. `TABLE_SCHEMA`
4. `TABLE_NAME`
5. `COLUMN_NAME`
6. `ORDINAL_POSITION`
7. `POSITION_IN_UNIQUE_CONSTRAINT`
8. `REFERENCED_TABLE_SCHEMA`
9. `REFERENCED_TABLE_NAME`
10. `REFERENCED_COLUMN_NAME`

> **用途：** 查看主键、唯一键、外键到底作用在哪些字段上。

特别适合排查：

```text
当前表.column
      ↓ 外键
目标表.column
```

---

## 7. `information_schema.REFERENTIAL_CONSTRAINTS` —— 外键规则 ★★★★

### 高频字段

1. `CONSTRAINT_SCHEMA`
2. `CONSTRAINT_NAME`
3. `TABLE_NAME`
4. `REFERENCED_TABLE_NAME`
5. `UPDATE_RULE`
6. `DELETE_RULE`

例如：

- `CASCADE`
- `SET NULL`
- `RESTRICT`
- `NO ACTION`

> **用途：** 排查删除/更新数据时为什么触发外键限制或级联操作。

---

## 8. `information_schema.INNODB_TRX` —— 当前 InnoDB 事务 ★★★★★

事务和锁排查的核心表之一。

### 核心字段

1. `TRX_ID`：InnoDB 事务 ID。
2. `TRX_STATE`：事务状态。
3. `TRX_STARTED`：事务开始时间。
4. `TRX_REQUESTED_LOCK_ID`：正在等待的锁 ID。
5. `TRX_WAIT_STARTED`：开始等待锁的时间。
6. `TRX_WEIGHT`：事务权重，死锁处理时有参考意义。
7. `TRX_MYSQL_THREAD_ID`：对应 MySQL 连接/线程 ID。
8. `TRX_QUERY`：事务当前执行 SQL。
9. `TRX_OPERATION_STATE`：事务当前操作状态。
10. `TRX_TABLES_IN_USE`
11. `TRX_TABLES_LOCKED`
12. `TRX_LOCK_STRUCTS`
13. `TRX_LOCK_MEMORY_BYTES`
14. `TRX_ROWS_LOCKED`：近似锁定行数。
15. `TRX_ROWS_MODIFIED`：已修改/插入行数。
16. `TRX_ISOLATION_LEVEL`：事务隔离级别。

> **用途：** 长事务、锁等待、事务未提交、事务影响范围分析。

### 高频查询：查长事务

```sql
SELECT
    TRX_ID,
    TRX_STATE,
    TRX_STARTED,
    TIMESTAMPDIFF(SECOND, TRX_STARTED, NOW()) AS trx_seconds,
    TRX_MYSQL_THREAD_ID,
    TRX_ROWS_LOCKED,
    TRX_ROWS_MODIFIED,
    TRX_QUERY
FROM information_schema.INNODB_TRX
ORDER BY TRX_STARTED;
```

---

# 四、`performance_schema`：运行时性能监控底层

## 1. `performance_schema.processlist` —— 实时在线会话 ★★★★★

线上卡顿、慢 SQL、异常连接时首先要想到。

### 核心字段

1. `ID`：连接 ID，可与 `KILL ID` 对应。
2. `USER`：MySQL 用户。
3. `HOST`：客户端主机，TCP/IP 连接通常显示主机 + 客户端端口。
4. `DB`：当前默认数据库。
5. `COMMAND`：当前命令；常见 `Sleep`、`Query`。
6. `TIME`：线程保持**当前状态**已经持续的秒数。
7. `STATE`：线程正在进行的动作/状态。
8. `INFO`：线程当前执行的 SQL；没有 SQL 时可能为 `NULL`。
9. `EXECUTION_ENGINE`：查询执行引擎；MySQL 8.0.29 起存在该字段。

> **注意：**
>
> `TIME = 300` 不能一律理解成“SQL 执行了 300 秒”。
>
> 如果 `COMMAND = Sleep`，表示该连接已经空闲约 300 秒。

### 高频查询

```sql
SELECT
    ID,
    USER,
    HOST,
    DB,
    COMMAND,
    TIME,
    STATE,
    INFO
FROM performance_schema.processlist
ORDER BY TIME DESC;
```

---

## 2. `events_statements_current` —— 当前正在执行的语句 ★★★★

相比 `processlist` 提供更丰富的 Performance Schema Statement Event 信息。

### 高频字段

1. `THREAD_ID`
2. `EVENT_ID`
3. `EVENT_NAME`
4. `TIMER_START`
5. `TIMER_END`
6. `TIMER_WAIT`
7. `SQL_TEXT`
8. `DIGEST`
9. `DIGEST_TEXT`
10. `CURRENT_SCHEMA`
11. `ROWS_AFFECTED`
12. `ROWS_SENT`
13. `ROWS_EXAMINED`
14. `CREATED_TMP_DISK_TABLES`
15. `CREATED_TMP_TABLES`
16. `SELECT_FULL_JOIN`
17. `SELECT_SCAN`
18. `NO_INDEX_USED`
19. `NO_GOOD_INDEX_USED`

> **用途：** 对当前执行语句做更细粒度性能分析。

---

## 3. `events_statements_history` / `events_statements_history_long` ★★★★

- `events_statements_history`：每线程较短历史。
- `events_statements_history_long`：服务器范围内更长的语句历史缓冲。

> **用途：** 某条慢 SQL 已经执行结束，`processlist` 看不到时，可以尝试从历史事件中定位。

---

## 4. `events_statements_summary_by_digest` —— SQL 模板聚合 ★★★★★

长期 SQL 性能分析最重要的表之一。

MySQL 会对结构相同、参数值不同的 SQL 做 Digest 归一化聚合。

例如：

```sql
SELECT * FROM user WHERE id = 1;
SELECT * FROM user WHERE id = 2;
```

可能归为类似模板：

```text
SELECT * FROM user WHERE id = ?
```

### 核心字段

1. `SCHEMA_NAME`
2. `DIGEST`
3. `DIGEST_TEXT`
4. `COUNT_STAR`：累计执行次数。
5. `SUM_TIMER_WAIT`：累计等待/执行计时。
6. `MIN_TIMER_WAIT`
7. `AVG_TIMER_WAIT`
8. `MAX_TIMER_WAIT`
9. `SUM_ROWS_AFFECTED`
10. `SUM_ROWS_SENT`
11. `SUM_ROWS_EXAMINED`
12. `SUM_CREATED_TMP_TABLES`
13. `SUM_CREATED_TMP_DISK_TABLES`
14. `SUM_SELECT_SCAN`
15. `SUM_SELECT_FULL_JOIN`
16. `SUM_NO_INDEX_USED`
17. `SUM_NO_GOOD_INDEX_USED`
18. `FIRST_SEEN`
19. `LAST_SEEN`

> **用途：** 找高频 SQL、总耗时最高 SQL、平均耗时高 SQL、扫描量异常 SQL、未使用索引 SQL。

### 重要判断

不要直接认为：

```text
SUM_ROWS_EXAMINED >> SUM_ROWS_SENT
= 索引失效
```

它只能说明：

> 为返回较少结果扫描了较多数据，是一个值得进一步 `EXPLAIN / EXPLAIN ANALYZE` 的优化信号。

### 高频查询：按总耗时找热点 SQL

```sql
SELECT
    SCHEMA_NAME,
    DIGEST_TEXT,
    COUNT_STAR,
    SUM_TIMER_WAIT,
    AVG_TIMER_WAIT,
    SUM_ROWS_EXAMINED,
    SUM_ROWS_SENT,
    FIRST_SEEN,
    LAST_SEEN
FROM performance_schema.events_statements_summary_by_digest
ORDER BY SUM_TIMER_WAIT DESC
LIMIT 20;
```

---

## 5. `table_io_waits_summary_by_table` —— 表 I/O 热点 ★★★★★

### 核心字段

1. `OBJECT_TYPE`
2. `OBJECT_SCHEMA`
3. `OBJECT_NAME`
4. `COUNT_STAR`
5. `SUM_TIMER_WAIT`
6. `COUNT_READ`
7. `SUM_TIMER_READ`
8. `COUNT_WRITE`
9. `SUM_TIMER_WRITE`
10. `COUNT_FETCH`
11. `COUNT_INSERT`
12. `COUNT_UPDATE`
13. `COUNT_DELETE`

> **用途：** 找出读写最频繁、I/O 等待最明显的热点表。

---

## 6. `table_io_waits_summary_by_index_usage` —— 索引 I/O 使用情况 ★★★★

### 高频字段

1. `OBJECT_SCHEMA`
2. `OBJECT_NAME`
3. `INDEX_NAME`
4. `COUNT_STAR`
5. `COUNT_READ`
6. `COUNT_WRITE`
7. `COUNT_FETCH`
8. `COUNT_INSERT`
9. `COUNT_UPDATE`
10. `COUNT_DELETE`

> **用途：** 从 Performance Schema 底层观察索引访问情况，也是 `sys.schema_unused_indexes` 等视图的重要数据基础。

---

## 7. `table_lock_waits_summary_by_table` —— 表锁等待统计 ★★★★

### 高频字段

1. `OBJECT_SCHEMA`
2. `OBJECT_NAME`
3. `COUNT_STAR`
4. `SUM_TIMER_WAIT`
5. 各种读锁/写锁等待统计字段

> **用途：** 表级锁竞争分析。

---

# 五、事务锁排查核心三件套

## 1. `performance_schema.data_locks` —— 当前持有/请求的数据锁 ★★★★★

### 核心字段

1. `ENGINE`
2. `ENGINE_LOCK_ID`
3. `ENGINE_TRANSACTION_ID`
4. `THREAD_ID`
5. `EVENT_ID`
6. `OBJECT_SCHEMA`
7. `OBJECT_NAME`
8. `INDEX_NAME`
9. `OBJECT_INSTANCE_BEGIN`
10. `LOCK_TYPE`
11. `LOCK_MODE`
12. `LOCK_STATUS`
13. `LOCK_DATA`

> **用途：** 看当前有哪些事务持有什么锁、请求什么锁、锁的是哪个表/索引/记录。

---

## 2. `performance_schema.data_lock_waits` —— 谁在等谁 ★★★★★

这是排查行锁阻塞最核心的关联表之一。

### 核心字段

1. `REQUESTING_ENGINE_LOCK_ID`
2. `REQUESTING_ENGINE_TRANSACTION_ID`
3. `REQUESTING_THREAD_ID`
4. `REQUESTING_EVENT_ID`
5. `BLOCKING_ENGINE_LOCK_ID`
6. `BLOCKING_ENGINE_TRANSACTION_ID`
7. `BLOCKING_THREAD_ID`
8. `BLOCKING_EVENT_ID`

核心关系：

```text
REQUESTING
等待锁的一方
      ↓
data_lock_waits
      ↓
BLOCKING
当前阻塞它的一方
```

> **用途：** 直接建立“被阻塞事务 → 阻塞事务”的依赖关系。

---

## 3. `performance_schema.metadata_locks` —— MDL 元数据锁 ★★★★★

DDL 卡住时必须想到这张表。

### 高频字段

1. `OBJECT_TYPE`
2. `OBJECT_SCHEMA`
3. `OBJECT_NAME`
4. `COLUMN_NAME`
5. `OBJECT_INSTANCE_BEGIN`
6. `LOCK_TYPE`
7. `LOCK_DURATION`
8. `LOCK_STATUS`
9. `SOURCE`
10. `OWNER_THREAD_ID`
11. `OWNER_EVENT_ID`

典型场景：

```text
事务 A
SELECT / UPDATE 某表
但长时间未提交
        ↓
持有 MDL
        ↓
事务 B
ALTER TABLE ...
        ↓
等待 MDL
```

> **用途：** 排查 `ALTER TABLE`、`DROP TABLE` 等 DDL 为什么长时间卡住。

---

# 六、`performance_schema` 配置/监控核心表

## 1. `threads` —— MySQL 内部线程信息 ★★★★

### 高频字段

1. `THREAD_ID`
2. `NAME`
3. `TYPE`
4. `PROCESSLIST_ID`
5. `PROCESSLIST_USER`
6. `PROCESSLIST_HOST`
7. `PROCESSLIST_DB`
8. `PROCESSLIST_COMMAND`
9. `PROCESSLIST_TIME`
10. `PROCESSLIST_STATE`
11. `PROCESSLIST_INFO`

> **用途：** 把 Performance Schema 的 `THREAD_ID` 与客户端连接 ID 联系起来。

---

## 2. `setup_instruments` —— 监控项开关 ★★★

### 高频字段

1. `NAME`
2. `ENABLED`
3. `TIMED`
4. `PROPERTIES`
5. `VOLATILITY`
6. `DOCUMENTATION`

> **用途：** 查看某类事件是否被 Performance Schema 采集以及是否计时。

---

## 3. `setup_consumers` —— 数据消费者开关 ★★★

### 核心字段

1. `NAME`
2. `ENABLED`

例如决定是否维护：

- current
- history
- history_long
- statements
- waits

等类型的 Performance Schema 消费数据。

> **用途：** 如果某些 `events_*` 表一直为空，需要检查对应 consumer 是否启用。

---

# 七、`sys`：日常排障优先使用的友好视图

## 1. `sys.session` —— 用户会话视图 ★★★★★

与 `sys.processlist` 类似，但主要关注用户会话，过滤掉后台线程。

### 高频字段

1. `thd_id`
2. `conn_id`
3. `user`
4. `db`
5. `command`
6. `state`
7. `time`
8. `current_statement`
9. `statement_latency`
10. `lock_latency`
11. `rows_examined`
12. `rows_sent`
13. `rows_affected`
14. `tmp_tables`
15. `tmp_disk_tables`
16. `full_scan`
17. `last_statement`
18. `last_statement_latency`
19. `trx_latency`
20. `trx_state`
21. `program_name`

> **用途：** 日常看在线用户连接、当前 SQL、执行耗时、事务时长、扫描行数。

---

## 2. `sys.processlist` —— 更完整的友好会话视图 ★★★★

与 `sys.session` 的主要区别：

- `sys.processlist`：包含后台线程。
- `sys.session`：主要显示用户 Session。

> **用途：** 想观察 MySQL Server 内部线程时使用 `sys.processlist`。

---

## 3. `sys.statement_analysis` —— SQL 聚合分析 ★★★★★

可以看作 `events_statements_summary_by_digest` 更适合人读的封装之一。

### 高频字段

1. `query`
2. `db`
3. `full_scan`
4. `exec_count`
5. `err_count`
6. `warn_count`
7. `total_latency`
8. `max_latency`
9. `avg_latency`
10. `lock_latency`
11. `rows_sent`
12. `rows_sent_avg`
13. `rows_examined`
14. `rows_examined_avg`
15. `rows_affected`
16. `tmp_tables`
17. `tmp_disk_tables`
18. `rows_sorted`
19. `sort_merge_passes`
20. `first_seen`
21. `last_seen`

> **用途：** 找平均耗时高、总耗时高、全表扫描、临时表多、扫描行数大的 SQL。

---

## 4. `sys.schema_unused_indexes` —— 未观察到使用的索引 ★★★★★

### 字段

1. `object_schema`
2. `object_name`
3. `index_name`

> **用途：** 找“候选未使用索引”。

### 重要注意

不能理解成：

> 查出来就能安全删除。

正确理解：

> 在 Performance Schema 当前统计周期/已观测工作负载中没有观察到索引使用。

例如：

```text
MySQL 刚重启
↓
统计数据重新开始
↓
某个每周只执行一次的报表索引
↓
此刻可能显示 unused
```

因此删除前必须结合：

- 业务完整周期
- 慢查询/查询日志
- `EXPLAIN`
- 代码搜索
- 历史监控

进一步判断。

---

## 5. `sys.schema_redundant_indexes` —— 冗余索引 ★★★★★

### 核心字段

1. `table_schema`
2. `table_name`
3. `redundant_index_name`
4. `redundant_index_columns`
5. `redundant_index_non_unique`
6. `dominant_index_name`
7. `dominant_index_columns`
8. `dominant_index_non_unique`
9. `subpart_exists`
10. `sql_drop_index`

例如：

```text
idx_a(a)

idx_ab(a, b)
```

从最左前缀角度看，`idx_a` 可能被 `idx_ab` 覆盖，因此可能成为冗余候选。

> **用途：** 找重复/被其他索引覆盖的索引候选。

> **注意：** `sql_drop_index` 只是生成可执行的删除语句，不等于应该不经评估直接执行。

---

## 6. `sys.schema_table_statistics` —— 表负载统计 ★★★★★

### 核心字段

1. `table_schema`
2. `table_name`
3. `total_latency`
4. `rows_fetched`
5. `fetch_latency`
6. `rows_inserted`
7. `insert_latency`
8. `rows_updated`
9. `update_latency`
10. `rows_deleted`
11. `delete_latency`
12. `io_read_requests`
13. `io_read`
14. `io_read_latency`
15. `io_write_requests`
16. `io_write`
17. `io_write_latency`
18. `io_misc_requests`
19. `io_misc_latency`

> **用途：** 快速比较每张表的读写量和 I/O 延迟，定位热点表。

---

## 7. `sys.schema_index_statistics` —— 索引使用统计 ★★★★

### 高频字段

1. `table_schema`
2. `table_name`
3. `index_name`
4. `rows_selected`
5. `select_latency`
6. `rows_inserted`
7. `insert_latency`
8. `rows_updated`
9. `update_latency`
10. `rows_deleted`
11. `delete_latency`

> **用途：** 判断哪些索引承担了大量查询、写入以及相关 I/O 等待。

---

## 8. `sys.innodb_lock_waits` —— InnoDB 锁等待友好视图 ★★★★★

相比自己连接 `data_locks`、`data_lock_waits`、线程和事务信息，它更适合日常快速查看。

常见关注内容：

- waiting transaction
- waiting SQL
- blocking transaction
- blocking SQL
- waiting time
- locked table/index
- 锁类型/锁模式

> **用途：** 线上发生锁等待时，通常可以优先查询该视图，再深入到底层 Performance Schema。

---

## 9. `sys.host_summary` —— Host 维度统计 ★★★★

### 核心字段

1. `host`
2. `statements`
3. `statement_latency`
4. `statement_avg_latency`
5. `table_scans`
6. `file_ios`
7. `file_io_latency`
8. `current_connections`
9. `total_connections`
10. `unique_users`
11. `current_memory`
12. `total_memory_allocated`

> **注意：**
>
> - `statement_latency` = 该 Host 的语句**总等待时间**
> - `statement_avg_latency` = 单条语句**平均等待时间**
> - `current_connections` = 当前连接数
> - `total_connections` = 累计连接数
>
> 这里没有应当被记成“峰值连接数”的 `max_connections` 字段。

> **用途：** 从客户端 Host 维度发现异常流量、连接过多、全表扫描和 SQL 负载异常。

---

## 10. `sys.user_summary` —— 用户维度负载统计 ★★★★

与 `host_summary` 类似，但统计维度变为 MySQL 用户。

> **用途：** 判断哪个数据库账号产生了最多 SQL、I/O、连接或内存负载。

---

# 八、最重要的排障链路

## 场景 1：账号无法登录

建议顺序：

```text
mysql.user
│
├── Host / User 是否匹配
├── plugin 是否正确
├── account_locked
├── password_expired
│
↓
mysql.global_grants
│
└── 是否涉及所需动态管理权限
│
↓
SHOW GRANTS
│
↓
mysql.db / tables_priv / columns_priv
```

---

## 场景 2：线上突然卡顿

```text
performance_schema.processlist
        ↓
先看谁正在执行、TIME、STATE、INFO
        ↓
sys.session
        ↓
更友好地看 statement_latency / trx_latency
        ↓
information_schema.INNODB_TRX
        ↓
是否存在长事务
        ↓
sys.innodb_lock_waits
        ↓
是否发生锁阻塞
```

---

## 场景 3：SQL 长期很慢

```text
events_statements_summary_by_digest
        ↓
找高频 / 高总耗时 / 高平均耗时 SQL
        ↓
sys.statement_analysis
        ↓
更直观查看 rows_examined、full_scan、tmp_tables
        ↓
EXPLAIN / EXPLAIN ANALYZE
        ↓
information_schema.STATISTICS
        ↓
检查已有索引
```

---

## 场景 4：发现锁等待

```text
information_schema.INNODB_TRX
        ↓
有哪些事务
        ↓
performance_schema.data_locks
        ↓
持有什么锁
        ↓
performance_schema.data_lock_waits
        ↓
谁在等谁
        ↓
performance_schema.threads
        ↓
THREAD_ID 对应哪个连接
        ↓
processlist
        ↓
最终定位客户端和 SQL
```

更快捷：

```text
sys.innodb_lock_waits
        ↓
快速定位
        ↓
需要深入时再查 performance_schema
```

---

## 场景 5：`ALTER TABLE` 卡住

```text
processlist
        ↓
发现 ALTER TABLE 长时间等待
        ↓
performance_schema.metadata_locks
        ↓
查 PENDING 的 MDL
        ↓
OWNER_THREAD_ID
        ↓
performance_schema.threads
        ↓
PROCESSLIST_ID
        ↓
定位阻塞连接
```

---

## 场景 6：找热点表

```text
performance_schema.table_io_waits_summary_by_table
        ↓
底层表 I/O 数据
        ↓
sys.schema_table_statistics
        ↓
更直观看 rows_fetched / rows_updated / latency
```

---

## 场景 7：索引治理

```text
information_schema.STATISTICS
        ↓
先看索引结构
        ↓
performance_schema.table_io_waits_summary_by_index_usage
        ↓
看索引访问
        ↓
sys.schema_index_statistics
        ↓
看索引负载
        ↓
sys.schema_unused_indexes
        ↓
找未观察到使用的索引
        ↓
sys.schema_redundant_indexes
        ↓
找疑似冗余索引
        ↓
EXPLAIN + 业务完整周期验证
        ↓
再决定是否删除
```

---

# 九、学习优先级：真正建议背下来的表

## 第一梯队：必须会

| 表 / 视图 | 一句话用途 |
|---|---|
| `mysql.user` | 账号、认证、静态全局权限 |
| `mysql.global_grants` | MySQL 8.0 动态全局权限 |
| `mysql.db` | 数据库级权限 |
| `information_schema.TABLES` | 表信息、表大小、引擎 |
| `information_schema.COLUMNS` | 字段定义 |
| `information_schema.STATISTICS` | 索引结构 |
| `information_schema.INNODB_TRX` | 当前 InnoDB 事务 |
| `performance_schema.processlist` | 实时连接与 SQL |
| `events_statements_summary_by_digest` | SQL 模板性能聚合 |
| `performance_schema.data_locks` | 当前数据锁 |
| `performance_schema.data_lock_waits` | 锁等待关系 |
| `performance_schema.metadata_locks` | MDL 元数据锁 |
| `table_io_waits_summary_by_table` | 表 I/O 热点 |
| `sys.session` | 用户会话友好视图 |
| `sys.statement_analysis` | SQL 性能友好分析 |
| `sys.schema_table_statistics` | 表读写与延迟 |
| `sys.schema_unused_indexes` | 候选未使用索引 |
| `sys.schema_redundant_indexes` | 候选冗余索引 |
| `sys.innodb_lock_waits` | 锁等待快速分析 |

---

## 第二梯队：工作中常用

| 表 / 视图 | 用途 |
|---|---|
| `mysql.tables_priv` | 表级权限 |
| `mysql.role_edges` | Role 授权关系 |
| `information_schema.KEY_COLUMN_USAGE` | 主键/外键字段 |
| `information_schema.TABLE_CONSTRAINTS` | 约束 |
| `performance_schema.threads` | 线程映射 |
| `events_statements_current` | 当前 Statement Event |
| `events_statements_history_long` | 历史 SQL |
| `table_io_waits_summary_by_index_usage` | 索引 I/O |
| `sys.schema_index_statistics` | 索引统计 |
| `sys.host_summary` | Host 维度统计 |
| `sys.user_summary` | User 维度统计 |

---

## 第三梯队：知道存在即可

- `mysql.columns_priv`
- `mysql.password_history`
- `setup_instruments`
- `setup_consumers`
- 各类 `events_waits_*`
- 各类 `memory_summary_*`
- 各类 `file_summary_*`
- `information_schema.ROUTINES`
- `information_schema.TRIGGERS`
- `information_schema.EVENTS`
- `information_schema.PARTITIONS`

根据业务需要再深入。

---

# 十、四大系统库的记忆口诀

```text
mysql
= 权限

information_schema
= 结构

performance_schema
= 底层运行数据

sys
= 友好分析视图
```

进一步：

```text
账号问题
→ mysql

表/字段/索引结构问题
→ information_schema

正在执行什么、锁了什么、慢在哪里
→ performance_schema

想快速看懂性能问题
→ sys
```

---

# 十一、几个特别容易记错的点

## 1. `mysql.user.max_connections`

错误：

```text
最大并发连接数
```

正确：

```text
账号每小时最大连接次数
```

最大并发连接数对应：

```text
max_user_connections
```

---

## 2. `processlist.TIME`

错误：

```text
SQL 已执行时间
```

更准确：

```text
线程处于当前状态已经持续的秒数
```

因此：

```text
COMMAND = Query
TIME = 100
```

通常值得怀疑当前查询持续较久。

但：

```text
COMMAND = Sleep
TIME = 100
```

表示连接空闲约 100 秒。

---

## 3. `schema_unused_indexes`

错误：

```text
查出来的索引可以直接删
```

正确：

```text
当前 Performance Schema 已观察工作负载中未发现使用
```

删除前必须进一步确认。

---

## 4. `SUM_ROWS_EXAMINED >> SUM_ROWS_SENT`

错误：

```text
一定是索引失效
```

正确：

```text
是潜在低效扫描信号，需要结合 EXPLAIN 判断
```

---

## 5. `sys.host_summary.statement_latency`

错误：

```text
平均 SQL 耗时
```

正确：

```text
statement_latency
= 总语句等待时间

statement_avg_latency
= 平均语句等待时间
```

---

## 6. `mysql.user` 是否能看到 MySQL 8.0 所有全局权限？

不能。

```text
mysql.user
→ 静态全局权限

mysql.global_grants
→ 动态全局权限
```

两者都要考虑。

---

# 十二、最终速查图

```text
MySQL Server
│
├── mysql
│   ├── user ------------------ 账号 / 静态全局权限
│   ├── global_grants --------- 动态全局权限
│   ├── db -------------------- 数据库级权限
│   ├── tables_priv ----------- 表级权限
│   ├── columns_priv ---------- 列级权限
│   ├── role_edges ------------ Role 授权关系
│   └── default_roles --------- 默认 Role
│
├── information_schema
│   ├── SCHEMATA -------------- 数据库
│   ├── TABLES ---------------- 表
│   ├── COLUMNS --------------- 字段
│   ├── STATISTICS ------------ 索引
│   ├── TABLE_CONSTRAINTS ----- 约束
│   ├── KEY_COLUMN_USAGE ------ 键 / 外键字段
│   ├── REFERENTIAL_CONSTRAINTS 外键规则
│   └── INNODB_TRX ------------ 当前事务
│
├── performance_schema
│   ├── processlist ----------- 当前会话
│   ├── threads --------------- 线程
│   ├── events_statements_* --- SQL 事件
│   ├── events_statements_summary_by_digest
│   │                           SQL 模板性能聚合
│   ├── table_io_waits_* ------ 表 / 索引 I/O
│   ├── data_locks ------------ 数据锁
│   ├── data_lock_waits ------- 锁等待关系
│   ├── metadata_locks -------- MDL
│   ├── setup_instruments ----- 监控项
│   └── setup_consumers ------- 数据消费者
│
└── sys
    ├── session --------------- 用户会话
    ├── processlist ----------- 线程友好视图
    ├── statement_analysis ---- SQL 分析
    ├── schema_table_statistics 表统计
    ├── schema_index_statistics 索引统计
    ├── schema_unused_indexes - 未使用索引候选
    ├── schema_redundant_indexes 冗余索引候选
    ├── innodb_lock_waits ----- 锁等待
    ├── host_summary ---------- Host 统计
    └── user_summary ---------- 用户统计
```

---

# 十三、推荐掌握顺序

如果目的是**开发 + 面试 + 实际排障**，建议：

```text
第一阶段
mysql.user
mysql.global_grants
mysql.db

第二阶段
information_schema.TABLES
information_schema.COLUMNS
information_schema.STATISTICS

第三阶段
performance_schema.processlist
events_statements_summary_by_digest

第四阶段
INNODB_TRX
data_locks
data_lock_waits
metadata_locks

第五阶段
sys.session
sys.statement_analysis
sys.schema_table_statistics
sys.schema_unused_indexes
sys.schema_redundant_indexes
sys.innodb_lock_waits
```

这套顺序比直接背 MySQL 数百张系统表更实用。

---

# 十四、官方文档参考

本文字段和定位以 MySQL 8.0 Reference Manual 为主要依据。

- Grant Tables  
  https://dev.mysql.com/doc/refman/8.0/en/grant-tables.html

- `performance_schema.processlist`  
  https://dev.mysql.com/doc/refman/8.0/en/performance-schema-processlist-table.html

- Performance Schema Statement Summary Tables  
  https://dev.mysql.com/doc/refman/8.0/en/performance-schema-statement-summary-tables.html

- Performance Schema Table I/O and Lock Wait Summary Tables  
  https://dev.mysql.com/doc/refman/8.0/en/performance-schema-table-wait-summary-tables.html

- `performance_schema.data_locks`  
  https://dev.mysql.com/doc/refman/8.0/en/performance-schema-data-locks-table.html

- `performance_schema.data_lock_waits`  
  https://dev.mysql.com/doc/refman/8.0/en/performance-schema-data-lock-waits-table.html

- `performance_schema.metadata_locks`  
  https://dev.mysql.com/doc/refman/8.0/en/performance-schema-metadata-locks-table.html

- INFORMATION_SCHEMA Table Reference  
  https://dev.mysql.com/doc/refman/8.0/en/information-schema-table-reference.html

- `INFORMATION_SCHEMA.INNODB_TRX`  
  https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-trx-table.html

- `sys.session`  
  https://dev.mysql.com/doc/refman/8.0/en/sys-session.html

- `sys.schema_unused_indexes`  
  https://dev.mysql.com/doc/refman/8.0/en/sys-schema-unused-indexes.html

- `sys.schema_redundant_indexes`  
  https://dev.mysql.com/doc/refman/8.0/en/sys-schema-redundant-indexes.html

- `sys.schema_table_statistics`  
  https://dev.mysql.com/doc/refman/8.0/en/sys-schema-table-statistics.html

- `sys.host_summary`  
  https://dev.mysql.com/doc/refman/8.0/en/sys-host-summary.html

---

> **一句话总结：**  
> `mysql` 看权限，`information_schema` 看结构，`performance_schema` 看底层运行状态，`sys` 做日常快速分析。
