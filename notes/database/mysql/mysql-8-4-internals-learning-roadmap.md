# MySQL 8.4 官方文档与底层学习路线

```text
MySQL 8.0 系统库
│
├── mysql
│   │
│   ├── mysql.user
│   │   ├── 账号
│   │   ├── 认证
│   │   ├── 静态全局权限
│   │   └── 资源限制
│   │
│   ├── mysql.global_grants
│   │   └── 动态全局权限【MySQL 8.0重点】
│   │
│   └── mysql.db
│       └── 数据库级权限
│
├── performance_schema
│   │
│   ├── processlist
│   │   └── 当前线程 / 会话
│   │
│   ├── events_statements_summary_by_digest
│   │   └── SQL模板聚合性能统计
│   │
│   └── table_io_waits_summary_by_table
│       └── 表级I/O统计
│
└── sys
    │
    ├── session
    │   └── 用户会话
    │
    ├── schema_unused_indexes
    │   └── 候选未使用索引
    │
    ├── schema_redundant_indexes
    │   └── 候选冗余索引
    │
    ├── schema_table_statistics
    │   └── 表级读写/延迟
    │
    └── host_summary
        └── Host维度总体统计
```



> 目标：不是把 MySQL 只当作 SQL 工具来学习，而是沿着  
> **MySQL Server → InnoDB → B+Tree / Page → Buffer Pool → MVCC / Lock → Redo / Undo → Disk**  
> 逐步深入到底层实现。

---

## 一、总体原则：不要从 Reference Manual 第一章顺序读

MySQL 8.4 Reference Manual 内容非常庞大，不建议从首页开始逐章顺序阅读。

对于“底层学习”目标，更推荐以 **InnoDB** 为第一主线，再补 **优化器、执行器、源码**。

推荐总路线：

```text
MySQL 8.4 Reference Manual
│
├── 第一阶段：InnoDB 存储结构
│
├── 第二阶段：事务 / MVCC / Lock
│
├── 第三阶段：Buffer Pool / Redo / Disk I/O
│
├── 第四阶段：Optimizer / EXPLAIN
│
└── 第五阶段：源码
```

---

# 二、第一站：The InnoDB Storage Engine

MySQL 8.4 Reference Manual 中，优先进入：

**The InnoDB Storage Engine**

不要机械地从该章第一节读到最后一节，而是按照下面的顺序学习。

---

## 1. InnoDB Architecture

这是正式开始阅读 MySQL 底层时最推荐的第一篇。

目标不是把所有细节都记住，而是先建立整体结构。

你首先应该形成：

```text
InnoDB
│
├── Memory
│   ├── Buffer Pool
│   ├── Change Buffer
│   ├── Adaptive Hash Index
│   └── Log Buffer
│
└── Disk
    ├── Tablespace
    ├── Table / Index Pages
    ├── Undo
    └── Redo
```

### 这一阶段要解决的问题

- InnoDB 是什么？
- MySQL Server 和 InnoDB 是什么关系？
- 哪些数据结构主要存在于内存？
- 哪些数据结构最终存在于磁盘？
- Buffer Pool、Redo、Undo、Tablespace 分别处于什么位置？

第一遍只要求知道“它是什么、为什么存在、和其他模块是什么关系”。

---

# 三、第二站：InnoDB In-Memory Structures

接下来进入：

**InnoDB In-Memory Structures**

这一部分最重要的是：

# Buffer Pool

第一遍学习时，不需要优先深挖 Change Buffer、Adaptive Hash Index。

先彻底弄清 Buffer Pool。

---

## 1. 为什么需要 Buffer Pool？

MySQL 查询数据时，并不是每次都直接从 SSD / Disk 读取。

更常见的流程：

```text
SQL
 ↓
需要访问某个 Page
 ↓
Buffer Pool
 ↓
是否已经缓存？
```

如果命中：

```text
Buffer Pool
 ↓
直接访问内存
```

如果没有命中：

```text
Disk
 ↓
读取 Page
 ↓
放入 Buffer Pool
 ↓
继续访问
```

---

## 2. Buffer Pool 要重点理解什么？

至少要回答：

- 为什么需要 Buffer Pool？
- Buffer Pool 缓存的是 Row 还是 Page？
- Page 从哪里读取？
- Page 没命中怎么办？
- Page 被修改后发生什么？
- 什么是 Clean Page？
- 什么是 Dirty Page？
- Dirty Page 什么时候写回磁盘？

最终应该形成：

```text
Disk
 │
 │ read page
 ▼
Buffer Pool
 │
 │ access / modify
 ▼
Page
```

修改后：

```text
Clean Page
   ↓
Dirty Page
   ↓
Flush
   ↓
Disk
```

Buffer Pool 是后面学习：

- B+Tree
- Page
- Redo
- Checkpoint
- Disk I/O

的重要基础。

---

# 四、第三站：InnoDB On-Disk Structures

然后进入：

**InnoDB On-Disk Structures**

这一部分通常会涉及：

- Tables
- Indexes
- Tablespaces
- Doublewrite Buffer
- Redo Log
- Undo Logs

第一遍不要全部深挖。

优先顺序：

```text
On-Disk Structures
        ↓
Indexes
        ↓
Tablespaces
```

Redo / Undo 放到后面的事务和持久化阶段深入。

---

# 五、第四站：Indexes —— 当前最重要的部分

这是 MySQL 底层学习中非常关键的一块。

推荐先学：

```text
Indexes
   │
   ├── Clustered and Secondary Indexes
   │
   └── Physical Structure of an InnoDB Index
```

---

## 1. Clustered Index

假设有表：

```text
users

id   name   age
1    Tom    20
2    Jack   21
3    Alice  22
```

主键：

```text
PRIMARY KEY(id)
```

你应该逐渐理解为：

```text
             Root Page
                │
        ┌───────┴───────┐
        ▼               ▼
     Page A           Page B
        │               │
        ▼               ▼

id=1 → 完整 Row
id=2 → 完整 Row

                id=3 → 完整 Row
```

核心关系：

```text
Primary Key
      ↓
B+Tree
      ↓
Leaf Page
      ↓
完整 Row
```

InnoDB 的主键索引通常就是聚簇索引。

---

## 2. Secondary Index

假设：

```text
INDEX(name)
```

二级索引可以先理解成：

```text
Secondary Index B+Tree

        name
          ↓

Tom   → id=1
Jack  → id=2
Alice → id=3
```

查询：

```sql
SELECT *
FROM users
WHERE name = 'Tom';
```

过程可以理解成：

```text
Secondary Index
      ↓
找到 Primary Key = 1
      ↓
Clustered Index
      ↓
找到完整 Row
```

这就是：

# 回表

---

## 3. 这一阶段要串起来的知识点

学完 Clustered Index / Secondary Index 后，你应该能够解释：

- 聚簇索引是什么？
- 二级索引是什么？
- 为什么二级索引叶子节点通常包含主键？
- 什么是回表？
- 什么是覆盖索引？
- 为什么主键不宜过长？
- 为什么索引可以提升查询性能？
- 为什么索引过多会降低 INSERT / UPDATE / DELETE 性能？

---

# 六、第五站：Physical Structure of an InnoDB Index

不要停留在：

```text
MySQL 索引使用 B+Tree
```

继续追问：

> B+Tree 中所谓的“节点”，在 InnoDB 中到底是什么？

核心答案逐渐走向：

# Page

建立下面的关系：

```text
B+Tree
  │
  ▼
Page
```

进一步：

```text
Tablespace
│
├── Page 0
├── Page 1
├── Page 2
├── Page 3
└── ...
```

一棵索引 B+Tree 由多个 Page 组成：

```text
          Root Page
              │
      ┌───────┴───────┐
      ▼               ▼
 Internal Page     Internal Page
      │               │
      ▼               ▼
 Leaf Page         Leaf Page
```

叶子 Page 中：

```text
Leaf Page
│
├── Record
├── Record
├── Record
├── Record
└── ...
```

因此你最终要打通：

```text
Tablespace
    ↓
Page
    ↓
B+Tree
    ↓
Record
    ↓
Row
```

---

# 七、第六站：InnoDB Row Formats

在理解 Page 和 Record 以后，再学习 Row Format。

主要了解：

- REDUNDANT
- COMPACT
- DYNAMIC
- COMPRESSED

现代 MySQL 重点理解：

**DYNAMIC**

这一阶段开始理解 Record 大致由什么组成：

```text
Record
│
├── Header
├── NULL 信息
├── Variable-Length 信息
├── Column Data
└── Hidden Fields
```

这里的 Hidden Fields 会直接和后面的：

- MVCC
- Undo
- Transaction

连接起来。

第一遍不必立刻研究每一个 bit 位。

---

# 八、第七站：回头理解 ACID

在理解了：

- Buffer Pool
- Page
- Record
- Redo
- Undo
- Lock
- MVCC

这些概念之后，再理解 ACID 会更有效。

不要只背：

```text
Atomicity
Consistency
Isolation
Durability
```

而是建立机制映射：

```text
Atomicity
   ↓
Undo / Transaction

Isolation
   ↓
MVCC + Lock

Durability
   ↓
Redo

Consistency
   ↓
事务系统整体协同保证
```

这样 ACID 不再只是四个定义，而是可以落实到底层机制。

---

# 九、第八站：Undo Log → MVCC

事务学习建议按照：

```text
Undo Logs
    ↓
InnoDB Multi-Versioning
    ↓
Transaction Isolation
```

---

## 1. Undo 不只是 Rollback

Undo 至少有两个非常重要的作用：

```text
Undo
├── Transaction Rollback
└── MVCC Historical Version
```

例如：

```text
age = 20
```

执行：

```sql
UPDATE users
SET age = 21
WHERE id = 1;
```

Undo 中要保存足够的信息，使得：

```text
21
↓
ROLLBACK
↓
20
```

同时，旧版本还可以被其他事务用于一致性读取。

---

## 2. MVCC 的核心结构

要逐渐建立：

```text
Current Record

DB_TRX_ID
DB_ROLL_PTR
     │
     ▼
Undo Record
     │
     ▼
Older Undo Record
```

这就形成版本链：

```text
Current Version
      ↓
Older Version
      ↓
Older Version
```

然后：

```text
Read View
    +
Version Chain
    ↓
判断当前事务能看到哪个版本
```

---

## 3. 学完后要能回答

- 为什么普通 SELECT 通常不需要加行锁？
- 什么是快照读？
- 什么是当前读？
- 为什么 Undo 不能在事务一提交后立刻全部删除？
- REPEATABLE READ 为什么能实现一致性读取？
- Read View 在 MVCC 中起什么作用？

---

# 十、第九站：Locking and Transaction Model

接下来学习锁。

推荐顺序：

```text
Transaction Isolation Levels
        ↓
Consistent Nonlocking Reads
        ↓
Locking Reads
        ↓
InnoDB Locking
        ↓
Record Lock
        ↓
Gap Lock
        ↓
Next-Key Lock
        ↓
Deadlock
```

---

## 1. 不要把 Lock 和 MVCC 混在一起

可以先这样理解：

```text
MVCC
 ↓
主要解决一致性读

Lock
 ↓
主要解决并发修改冲突
```

---

## 2. 最好的学习方法：两个 Session 做实验

Session A：

```sql
BEGIN;

UPDATE users
SET money = money - 100
WHERE id = 10;
```

不提交。

Session B：

```sql
UPDATE users
SET money = money + 100
WHERE id = 10;
```

观察阻塞。

然后不断改变条件：

- 等值查询
- 范围查询
- 有索引
- 无索引
- READ COMMITTED
- REPEATABLE READ

逐渐理解：

- Record Lock
- Gap Lock
- Next-Key Lock
- Deadlock

锁一定要配合实验学习。

---

# 十一、第十站：Redo Log

当你已经理解：

- Buffer Pool
- Dirty Page
- Page
- Disk

以后，再学 Redo 会非常自然。

一个 UPDATE 可以粗略理解成：

```text
UPDATE
   ↓
修改 Buffer Pool 中的 Page
   ↓
Dirty Page
```

与此同时：

```text
Redo Log
```

负责记录崩溃恢复需要的信息。

---

## 1. 为什么需要 Redo？

如果只修改内存中的 Page：

```text
Buffer Pool
 ↓
Dirty Page
```

还没写回磁盘时突然断电，就会存在数据丢失风险。

Redo 的作用就是支持 Crash Recovery。

---

## 2. 这一部分要重点理解

- WAL（Write-Ahead Logging）
- LSN
- Log Buffer
- Redo Log File
- Dirty Page
- Checkpoint
- Crash Recovery

最终形成：

```text
UPDATE

   ↓

Buffer Pool Page
   ↓
Dirty Page

与此同时：

Redo Log
   ↓
记录恢复信息

COMMIT

之后：

Dirty Page
   ↓
逐步 Flush
   ↓
Tablespace
```

---

# 十二、第十一站：Disk I/O + Checkpoint

继续学习：

**InnoDB Disk I/O and File Space Management**

重点：

**Checkpoint**

理解：

```text
Buffer Pool
    │
 Dirty Page
    │
    ▼
Flush
    │
    ▼
Tablespace
```

与此同时：

```text
Redo Log
 ↓
Checkpoint
 ↓
标记哪些修改已经安全进入数据文件
```

这一部分会把 MySQL 和操作系统的 I/O 知识真正连接起来。

---

# 十三、然后再去 Chapter 10：Optimization

不要一开始就啃完整 Optimization。

等理解：

- B+Tree
- Clustered Index
- Secondary Index
- Page
- Buffer Pool

以后，再进入：

# Chapter 10 — Optimization

第一遍重点：

```text
Optimization
│
├── Optimization Overview
│
├── Optimization and Indexes
│   ├── How MySQL Uses Indexes
│   ├── Primary Key Optimization
│   ├── Column Indexes
│   ├── Multiple-Column Indexes
│   └── B-Tree vs Hash
│
└── Understanding the Query Execution Plan
    └── EXPLAIN
```

后续第二遍再深入：

- Optimizer Statistics
- Cost Model
- Range Optimization
- Join Optimization
- Optimizer Hints
- Optimizer Trace

---

# 十四、EXPLAIN 不要靠背字段学习

推荐自己准备一张大表：

```text
users

1,000,000 rows
```

分别建立：

```sql
INDEX(age)

INDEX(city)

INDEX(age, city)

INDEX(city, age)
```

然后不断执行：

```sql
EXPLAIN ANALYZE
SELECT ...
```

观察：

- 使用了哪个索引？
- 预计扫描多少行？
- 实际扫描多少行？
- Cost 如何变化？
- actual time 如何变化？
- 联合索引顺序为什么影响执行计划？

这样才能真正理解优化器。

---

# 十五、最后进入源码

不要一开始就从 MySQL Server 的 main 函数一路往下读。

应该按照“问题 → 模块”定位源码。

## InnoDB 源码地图

| 知识点 | 主要源码方向 |
|---|---|
| B-Tree | `storage/innobase/btr/` |
| Buffer Pool | `storage/innobase/buf/` |
| Page | `storage/innobase/page/` |
| Record | `storage/innobase/rem/` |
| Row Operations | `storage/innobase/row/` |
| Transaction | `storage/innobase/trx/` |
| Lock | `storage/innobase/lock/` |
| Redo | `storage/innobase/log/` |
| Mini Transaction | `storage/innobase/mtr/` |
| File / Tablespace | `storage/innobase/fil/`、`storage/innobase/fsp/` |

## MySQL Server 源码地图

| 知识点 | 主要源码方向 |
|---|---|
| Join Optimizer | `sql/join_optimizer/` |
| Range Optimizer | `sql/range_optimizer/` |
| Executor / Iterator | `sql/iterators/` |
| Server SQL Layer | `sql/` |

---

# 十六、推荐的完整学习顺序

## 第一阶段：存储结构

```text
InnoDB Architecture
        ↓
In-Memory Structures
        ↓
Buffer Pool
        ↓
On-Disk Structures
        ↓
Indexes
        ↓
Clustered Index
        ↓
Secondary Index
        ↓
B+Tree
        ↓
Page
        ↓
Record
        ↓
Row Format
```

---

## 第二阶段：事务系统

```text
Undo Log
    ↓
MVCC
    ↓
Read View
    ↓
Transaction
    ↓
Isolation Level
    ↓
Lock
    ↓
Record Lock
    ↓
Gap Lock
    ↓
Next-Key Lock
    ↓
Deadlock
```

---

## 第三阶段：持久化

```text
Buffer Pool
    ↓
Dirty Page
    ↓
Redo Log
    ↓
WAL
    ↓
Checkpoint
    ↓
Disk I/O
```

---

## 第四阶段：查询优化

```text
Indexes
   ↓
EXPLAIN
   ↓
EXPLAIN ANALYZE
   ↓
Statistics
   ↓
Cost Model
   ↓
Optimizer
```

---

## 第五阶段：源码

```text
Source Code Documentation
        ↓
mysql-server Source
        ↓
sql/
        +
storage/innobase/
```

---

# 十七、当前应该先点哪一章？

你现在在 MySQL 8.4 Reference Manual 首页。

直接展开：

**The InnoDB Storage Engine**

然后第一轮按照下面顺序：

```text
① InnoDB Architecture

          ↓

② InnoDB In-Memory Structures
   └── Buffer Pool ★★★★★

          ↓

③ InnoDB On-Disk Structures

          ↓

④ Indexes
   ├── Clustered and Secondary Indexes ★★★★★
   └── Physical Structure of an InnoDB Index ★★★★★

          ↓

⑤ Tablespaces

          ↓

⑥ InnoDB Row Formats
```

第一轮暂时不需要优先深入：

- Replication
- Group Replication
- NDB Cluster
- Partitioning
- Stored Objects
- Security
- Backup
- Encryption
- Online DDL
- Performance Schema 全部细节

这些不是当前构建 MySQL 底层主干最重要的内容。

---

# 十八、第一遍阅读官方文档的原则

第一遍不要追求：

> 每句话都看懂、每个参数都记住。

第一遍只解决三个问题：

1. **它是什么？**
2. **为什么需要它？**
3. **它和上下模块怎么连接？**

第二遍再研究：

> **它内部究竟是怎么实现的？**

这样可以避免一开始陷入：

- `innodb_buffer_pool_size`
- 各种系统变量
- 各种边缘配置
- 大量运维参数

而失去底层学习主线。

---

# 十九、用“一条 SQL 生命周期”串起所有模块

最终建议把所有知识放回这张总图：

```text
Java Application
      │
      ▼
   MyBatis
      │
      ▼
   JDBC API
      │
      ▼
MySQL Connector/J
      │
      │ MySQL Protocol / TCP
      ▼
┌──────────────────────────────┐
│         MySQL Server         │
│                              │
│ Connection / Session         │
│          ↓                   │
│       SQL Parser             │
│          ↓                   │
│       Resolver               │
│          ↓                   │
│       Optimizer              │
│          ↓                   │
│ Execution / Iterator         │
│          ↓                   │
│ Storage Engine API / handler│
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│            InnoDB            │
│                              │
│       Buffer Pool            │
│            ↓                 │
│       B+Tree / Page          │
│            ↓                 │
│       Record / Row           │
│                              │
│      MVCC      Lock          │
│       │          │           │
│     Undo Log                 │
│                              │
│       Redo Log               │
└──────────────┬───────────────┘
               │
               ▼
       Tablespace / Log File
               │
               ▼
        File System / OS
               │
               ▼
           SSD / Disk
```

---

# 二十、最终学习目标

## 第一条主线：SELECT

```text
SELECT
  ↓
Optimizer
  ↓
Index
  ↓
B+Tree
  ↓
Page
  ↓
Buffer Pool
  ↓
Record
```

## 第二条主线：UPDATE

```text
UPDATE
  ↓
找到 Record
  ↓
Lock
  ↓
Undo
  ↓
修改 Buffer Pool Page
  ↓
Dirty Page
  ↓
Redo
  ↓
Commit
  ↓
Checkpoint
  ↓
Disk
```

如果这两条链能够完整解释，你对 MySQL 的理解就已经从：

**“会使用 MySQL”**

逐渐进入：

**“理解 MySQL Server / InnoDB 的工作原理”**。

---

# 官方资料建议

- MySQL 8.4 Reference Manual  
  https://dev.mysql.com/doc/refman/8.4/en/

- The InnoDB Storage Engine  
  https://dev.mysql.com/doc/refman/8.4/en/innodb-storage-engine.html

- Optimization  
  https://dev.mysql.com/doc/refman/8.4/en/optimization.html

- MySQL Server Source Code Documentation  
  https://dev.mysql.com/doc/dev/mysql-server/latest/

- MySQL Server Source Code  
  https://github.com/mysql/mysql-server

---

## 一句话总结

当前最合理的起点不是 SQL Statements，也不是 Optimization，而是：

```text
The InnoDB Storage Engine
        ↓
InnoDB Architecture
        ↓
Buffer Pool
        ↓
Indexes
        ↓
B+Tree
        ↓
Page
        ↓
Record
```

先把 **“数据到底怎么存、怎么进内存、索引在物理上是什么”** 搞清楚，再进入 MVCC、Lock、Redo、Optimizer 和源码。
