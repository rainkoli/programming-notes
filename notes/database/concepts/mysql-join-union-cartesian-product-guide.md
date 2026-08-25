# MySQL：INNER JOIN、LEFT JOIN、RIGHT JOIN、UNION、UNION ALL 与笛卡尔积

## 一、整体分类

这些 SQL 概念可以分成三类：

### 1. JOIN：横向连接表

JOIN 的作用可以理解为：

> **根据关联条件，把两张表的列横向组合起来。**

常见类型：

- `INNER JOIN`
- `LEFT JOIN`
- `RIGHT JOIN`

可以简单记忆：

```text
JOIN → 横着拼 → 列变多
```

---

### 2. UNION：纵向合并结果集

UNION 的作用是：

> **把两个 SELECT 查询结果上下拼接起来。**

常见类型：

- `UNION`
- `UNION ALL`

可以简单记忆：

```text
UNION → 竖着拼 → 行变多
```

---

### 3. 笛卡尔积

笛卡尔积表示：

> **一张表中的每一行，都和另一张表中的每一行组合一次。**

如果：

```text
表 A 有 m 行
表 B 有 n 行
```

那么笛卡尔积结果：

```text
m × n 行
```

---

# 二、示例数据

下面使用两张表进行说明。

## student 表

| id | name |
|---:|---|
| 1 | 张三 |
| 2 | 李四 |
| 3 | 王五 |

## score 表

| student_id | score |
|---:|---:|
| 1 | 90 |
| 2 | 80 |
| 4 | 70 |

对应关系：

```text
student.id = 1   ←→   score.student_id = 1
student.id = 2   ←→   score.student_id = 2

student.id = 3   没有对应成绩
score.student_id = 4   没有对应学生
```

---

# 三、INNER JOIN

## 1. 基本含义

`INNER JOIN`：

> **只保留两张表中能够成功匹配的数据。**

SQL：

```sql
SELECT *
FROM student s
INNER JOIN score sc
ON s.id = sc.student_id;
```

结果：

| id | name | student_id | score |
|---:|---|---:|---:|
| 1 | 张三 | 1 | 90 |
| 2 | 李四 | 2 | 80 |

匹配过程：

```text
张三 id=1  ←→ student_id=1    匹配 ✔

李四 id=2  ←→ student_id=2    匹配 ✔

王五 id=3  ←→ 没有             不匹配 ✘

没有 id=4  ←→ student_id=4    不匹配 ✘
```

因此可以记：

> **INNER JOIN = 两边都有的数据才要。**

---

## 2. 常见使用场景

例如：

> 查询已经有考试成绩的学生。

```sql
SELECT
    s.name,
    sc.score
FROM student s
INNER JOIN score sc
ON s.id = sc.student_id;
```

结果：

```text
张三  90
李四  80
```

因为王五没有成绩，所以不会出现在结果中。

---

# 四、LEFT JOIN

## 1. 基本含义

`LEFT JOIN`：

> **左表的数据全部保留，右表能匹配就连接，不能匹配就用 NULL 补齐。**

SQL：

```sql
SELECT *
FROM student s
LEFT JOIN score sc
ON s.id = sc.student_id;
```

这里：

```text
student = 左表
score   = 右表
```

结果：

| id | name | student_id | score |
|---:|---|---:|---:|
| 1 | 张三 | 1 | 90 |
| 2 | 李四 | 2 | 80 |
| 3 | 王五 | NULL | NULL |

过程：

```text
张三 id=1
    ↓
找到 score.student_id=1
    ↓
张三 | 90
```

```text
李四 id=2
    ↓
找到 score.student_id=2
    ↓
李四 | 80
```

```text
王五 id=3
    ↓
score 中找不到
    ↓
王五 | NULL
```

因此可以记：

> **LEFT JOIN = LEFT 左表一个都不能少。**

---

## 2. 常见使用场景

例如：

> 查询所有学生及其成绩，即使某些学生暂时没有成绩。

```sql
SELECT
    s.id,
    s.name,
    sc.score
FROM student s
LEFT JOIN score sc
ON s.id = sc.student_id;
```

结果：

```text
1 张三 90
2 李四 80
3 王五 NULL
```

这里学生是主体，因此学生不能因为没有成绩就消失。

---

# 五、RIGHT JOIN

## 1. 基本含义

`RIGHT JOIN`：

> **右表的数据全部保留，左表能匹配就连接，不能匹配就用 NULL 补齐。**

SQL：

```sql
SELECT *
FROM student s
RIGHT JOIN score sc
ON s.id = sc.student_id;
```

这里：

```text
student = 左表
score   = 右表
```

结果：

| id | name | student_id | score |
|---:|---|---:|---:|
| 1 | 张三 | 1 | 90 |
| 2 | 李四 | 2 | 80 |
| NULL | NULL | 4 | 70 |

因为：

```text
score.student_id = 4
```

虽然找不到：

```text
student.id = 4
```

但右表的这条成绩记录仍然必须保留，因此结果为：

```text
NULL | NULL | 4 | 70
```

可以记：

> **RIGHT JOIN = RIGHT 右表一个都不能少。**

---

# 六、INNER JOIN、LEFT JOIN、RIGHT JOIN 对比

原始数据：

```text
student

1 张三
2 李四
3 王五
```

```text
score

1 90
2 80
4 70
```

---

## INNER JOIN

```text
1 张三  1 90
2 李四  2 80
```

只保留匹配成功的数据。

---

## LEFT JOIN

```text
1 张三  1    90
2 李四  2    80
3 王五  NULL NULL
```

左表全部保留。

---

## RIGHT JOIN

```text
1    张三  1 90
2    李四  2 80
NULL NULL  4 70
```

右表全部保留。

---

## 对比表

| JOIN 类型 | 匹配数据 | 左表未匹配数据 | 右表未匹配数据 |
|---|---:|---:|---:|
| `INNER JOIN` | 保留 | 不保留 | 不保留 |
| `LEFT JOIN` | 保留 | 保留 | 不保留 |
| `RIGHT JOIN` | 保留 | 不保留 | 保留 |

---

# 七、JOIN 的本质

JOIN 可以理解为：

> **根据某个关联条件，把两张表中有关系的记录横向组合起来。**

例如：

```text
student

id | name
```

和：

```text
score

student_id | score
```

JOIN 后：

```text
id | name | student_id | score
```

因此：

```text
JOIN
=
横向连接
=
增加列
```

---

# 八、UNION

## 1. 基本含义

`UNION`：

> **把两个 SELECT 查询结果纵向合并，并自动去重。**

假设有：

### table_a

| name |
|---|
| 张三 |
| 李四 |
| 王五 |

### table_b

| name |
|---|
| 王五 |
| 赵六 |

执行：

```sql
SELECT name
FROM table_a

UNION

SELECT name
FROM table_b;
```

结果：

| name |
|---|
| 张三 |
| 李四 |
| 王五 |
| 赵六 |

注意：

```text
王五
```

只出现了一次。

原因：

> **UNION 会去除重复行。**

可以理解为：

```text
张三
李四
王五

    UNION

王五
赵六

      ↓

张三
李四
王五
赵六
```

---

# 九、UNION ALL

## 1. 基本含义

`UNION ALL`：

> **把两个 SELECT 查询结果纵向合并，但不去重。**

SQL：

```sql
SELECT name
FROM table_a

UNION ALL

SELECT name
FROM table_b;
```

结果：

| name |
|---|
| 张三 |
| 李四 |
| 王五 |
| 王五 |
| 赵六 |

这里：

```text
王五
王五
```

都会保留。

因此：

```text
UNION
=
合并
+
去重
```

而：

```text
UNION ALL
=
合并
+
不去重
```

---

# 十、UNION 与 UNION ALL 的性能区别

`UNION` 的逻辑过程：

```text
查询 A
   ↓
查询 B
   ↓
合并结果
   ↓
检查重复数据
   ↓
去重
   ↓
返回结果
```

`UNION ALL`：

```text
查询 A
   ↓
查询 B
   ↓
直接合并
   ↓
返回结果
```

因此一般情况下：

> **UNION ALL 比 UNION 更快。**

如果业务允许重复数据，通常优先考虑：

```sql
UNION ALL
```

---

# 十一、UNION 的使用要求

UNION 两边的 SELECT：

> **返回列数必须相同，对应位置的数据类型应当兼容。**

例如：

```sql
SELECT id, name
FROM student

UNION

SELECT student_id, score
FROM score;
```

两个 SELECT 都返回两列。

但下面这种写法不行：

```sql
SELECT id, name
FROM student

UNION

SELECT id, name, age
FROM teacher;
```

因为：

```text
第一个 SELECT：2 列
第二个 SELECT：3 列
```

无法纵向拼接。

---

# 十二、JOIN 与 UNION 的本质区别

这是非常重要的区别。

## JOIN

JOIN 是横向连接：

```text
表 A                     表 B

id | name                id | score
----------               ----------
1  | 张三                 1  | 90
2  | 李四                 2  | 80

             JOIN
               ↓

id | name | id | score
-----------------------
1  | 张三 | 1  | 90
2  | 李四 | 2  | 80
```

所以：

> **JOIN = 横向拼接 = 增加列。**

---

## UNION

UNION 是纵向合并：

```text
表 A

name
----
张三
李四

      UNION

表 B

name
----
王五
赵六

        ↓

name
----
张三
李四
王五
赵六
```

所以：

> **UNION = 纵向拼接 = 增加行。**

---

## 记忆方式

```text
JOIN   → 横着拼 → 列变多

UNION  ↓ 竖着拼 → 行变多
```

---

# 十三、笛卡尔积

## 1. 基本概念

假设：

```text
表 A

a1
a2
a3
```

共 3 行。

表 B：

```text
b1
b2
```

共 2 行。

笛卡尔积表示：

> 表 A 的每一行，都和表 B 的每一行组合一次。

结果：

```text
a1 + b1
a1 + b2

a2 + b1
a2 + b2

a3 + b1
a3 + b2
```

一共：

```text
3 × 2 = 6 行
```

因此：

```text
A 有 m 行
B 有 n 行

A × B = m × n 行
```

---

# 十四、SQL 中如何产生笛卡尔积

最明确的写法是：

```sql
SELECT *
FROM student
CROSS JOIN score;
```

如果：

```text
student = 3 行
score   = 3 行
```

结果就是：

```text
3 × 3 = 9 行
```

例如：

```text
张三 + 90
张三 + 80
张三 + 70

李四 + 90
李四 + 80
李四 + 70

王五 + 90
王五 + 80
王五 + 70
```

---

# 十五、忘记连接条件导致笛卡尔积

例如：

```sql
SELECT *
FROM student, score;
```

如果没有 WHERE 或 JOIN 条件进行限制，本质上就是：

```text
student × score
```

假设：

```text
student = 10,000 行
score   = 10,000 行
```

那么结果：

```text
10,000 × 10,000
=
100,000,000 行
```

也就是：

```text
1 亿条组合
```

因此实际开发中，如果意外产生笛卡尔积，可能导致严重的性能问题。

---

# 十六、JOIN 与笛卡尔积的关系

考虑：

```sql
SELECT *
FROM student s
INNER JOIN score sc
ON s.id = sc.student_id;
```

从关系代数和逻辑理解的角度，可以粗略理解为：

```text
JOIN
≈
笛卡尔积
+
连接条件筛选
```

例如所有可能组合：

```text
1 张三 + 1 90   ✔
1 张三 + 2 80   ✘
1 张三 + 4 70   ✘

2 李四 + 1 90   ✘
2 李四 + 2 80   ✔
2 李四 + 4 70   ✘

3 王五 + 1 90   ✘
3 王五 + 2 80   ✘
3 王五 + 4 70   ✘
```

然后根据：

```sql
ON s.id = sc.student_id
```

筛选出：

```text
1 张三 + 1 90
2 李四 + 2 80
```

但是要注意：

> 这只是用于理解 JOIN 的逻辑模型。

MySQL 实际执行 JOIN 时，并不一定真的先生成完整笛卡尔积再进行过滤。

数据库优化器会根据：

- 索引
- 表大小
- 统计信息
- JOIN 顺序
- 可用执行算法

选择效率更高的执行方案。

---

# 十七、ON 条件的意义

例如：

```sql
ON s.id = sc.student_id
```

它告诉数据库：

> 只把 `student.id` 与 `score.student_id` 相同的记录关联起来。

没有连接条件：

```text
student
   ×
score
   ↓
大量无意义组合
```

有连接条件：

```text
student × score
      ↓
id = student_id
      ↓
真正有关联的数据
```

所以：

> **ON 决定两张表中的哪些行应该建立关联。**

---

# 十八、如何判断该使用哪种 JOIN

不要先死记：

```text
到底应该 INNER、LEFT 还是 RIGHT？
```

应该先问：

> **哪张表的数据必须完整保留？**

---

## 情况 1：只需要两边匹配的数据

使用：

```sql
INNER JOIN
```

记忆：

```text
两边都有才要
```

---

## 情况 2：左表数据必须全部保留

使用：

```sql
LEFT JOIN
```

记忆：

```text
左边一个都不能少
```

---

## 情况 3：右表数据必须全部保留

使用：

```sql
RIGHT JOIN
```

记忆：

```text
右边一个都不能少
```

---

# 十九、六个概念总表

| 操作 | 核心含义 | 拼接方向 | 是否保留未匹配数据 |
|---|---|---|---|
| `INNER JOIN` | 两边匹配才保留 | 横向 | 不保留 |
| `LEFT JOIN` | 左表全部保留 | 横向 | 保留左表未匹配数据 |
| `RIGHT JOIN` | 右表全部保留 | 横向 | 保留右表未匹配数据 |
| `UNION` | 合并两个结果集并去重 | 纵向 | 不适用 |
| `UNION ALL` | 合并两个结果集但不去重 | 纵向 | 不适用 |
| `CROSS JOIN` | 每行与每行组合 | 横向 | 所有组合全部产生 |

---

# 二十、最终记忆口诀

可以把整个知识点压缩为：

```text
INNER JOIN
→ 两边匹配的才要

LEFT JOIN
→ 左边全部都要

RIGHT JOIN
→ 右边全部都要

UNION
→ 上下合并 + 去重

UNION ALL
→ 上下合并 + 不去重

CROSS JOIN
→ 所有行两两组合
```

再记住两个最核心的方向：

```text
JOIN
→ 横向拼接
→ 增加列

UNION
→ 纵向拼接
→ 增加行
```

最后理解 JOIN 的本质：

> **JOIN 是根据 ON 条件，从两张表可能的行组合中找到真正应该关联的数据；笛卡尔积则是不加匹配限制，让两张表的所有行两两组合。**
