# MySQL 数据库、SQL 文件与目录管理规范

## 1. 核心原则

不要只按“是不是项目”分类，还要区分文件的性质。

同一个项目可以同时出现在两个位置：

- `projects`：保存项目开发过程中长期维护的 SQL 源文件
- `backups`：保存该项目数据库的完整备份快照

例如，`blog_gp` 是毕设数据库：

```text
projects/blog-gp/
```

保存建表、查询、索引、初始化数据和升级脚本。

```text
backups/projects/blog-gp/
```

保存 Navicat 或 `mysqldump` 导出的完整数据库备份。

因此不是二选一，而是：

> 项目脚本放 `projects`，完整恢复文件放 `backups`。

---

## 2. 推荐的总目录结构

```text
mysql-workspace/
├── backups/
│   ├── projects/
│   └── demos/
│
├── projects/
├── demos/
└── temp/
```

### 2.1 `backups`

保存完整数据库快照，主要用于恢复。

通常是通过 Navicat、`mysqldump` 等工具导出的完整 SQL 文件。

```text
backups/
├── projects/
│   └── blog-gp/
│       ├── blog_gp_2026-08-05.sql
│       └── blog_gp_before_delete_2026-08-05.sql
│
└── demos/
    ├── mysql-practice/
    │   ├── demo_mysql_practice.sql
    │   └── demo_mybatis_practice.sql
    │
    ├── spring/
    │   ├── demo_spring_6.sql
    │   └── demo_spring_mvc.sql
    │
    └── digital-mall/
        ├── seata.sql
        ├── shuma_user.sql
        ├── shuma_log.sql
        └── shuma_permission.sql
```

这里一般不放手写的单条查询语句或索引实验。

### 2.2 `projects`

保存具体项目长期维护的数据库脚本。

```text
projects/
└── blog-gp/
    ├── README.md
    └── sql/
        ├── schema/
        ├── data/
        ├── indexes/
        ├── queries/
        └── migrations/
```

更完整的示例：

```text
projects/
└── blog-gp/
    ├── README.md
    └── sql/
        ├── schema/
        │   ├── create_database.sql
        │   ├── create_user_table.sql
        │   └── create_article_table.sql
        │
        ├── data/
        │   ├── initial_data.sql
        │   └── test_data.sql
        │
        ├── indexes/
        │   ├── create_user_indexes.sql
        │   └── create_article_indexes.sql
        │
        ├── queries/
        │   ├── query_user_statistics.sql
        │   ├── find_article_by_keyword.sql
        │   └── query_popular_articles.sql
        │
        └── migrations/
            ├── V001__create_user_table.sql
            ├── V002__create_article_table.sql
            └── V003__add_user_email_index.sql
```

目录用途：

| 目录 | 用途 |
|---|---|
| `schema` | 建库、建表、约束、视图等结构脚本 |
| `data` | 初始化数据、测试数据 |
| `indexes` | 创建、修改、删除索引 |
| `queries` | 项目常用查询语句 |
| `migrations` | 数据库结构的版本变更记录 |

### 2.3 `demos`

保存与具体正式项目无关的学习练习。

```text
demos/
└── mysql-practice/
    ├── basic/
    │   ├── create_table_practice.sql
    │   └── crud_practice.sql
    │
    ├── queries/
    │   ├── join_practice.sql
    │   ├── subquery_practice.sql
    │   └── group_by_practice.sql
    │
    ├── indexes/
    │   └── index_practice.sql
    │
    └── transactions/
        └── transaction_practice.sql
```

适合保存：

- 增删改查
- 多表连接
- 子查询
- 分组与聚合
- 事务
- 索引
- 视图
- 存储过程
- 触发器

### 2.4 `temp`

保存一次性测试文件。

```text
temp/
├── test_restore.sql
├── test_index_performance.sql
├── test_transaction.sql
└── test_query.sql
```

这里的文件应当满足：

> 随时删除也不会影响正式项目或重要备份。

重要脚本不要长期放在 `temp` 中。

---

## 3. 数据库命名规范

Navicat 左侧数据库列表建议统一使用前缀。

### 3.1 推荐前缀

```text
demo_       学习和练习数据库
project_    完整项目或课程设计数据库
tmp_        临时测试数据库
```

示例：

```text
demo_mysql_practice
demo_mybatis_practice
demo_spring_mvc

project_blog
project_library_management
project_student_leave

tmp_seata_restore
tmp_index_test
```

同类数据库会按字母自然聚集，Navicat 左侧列表会更清晰。

### 3.2 统一规则

数据库名建议：

- 全部小写
- 以英文字母开头
- 多个单词使用下划线连接
- 只使用字母、数字和下划线
- 名称能够表达实际用途
- 避免空格、连字符、中文和含义不明的数字

推荐：

```text
demo_mysql_practice
project_blog
project_library_management
tmp_restore_test
```

不推荐：

```text
MySQLPractice
mysql-practice
mysql practice
test1
newdb
mblog1
studentleavesystem
```

### 3.3 综合练习数据库

用于综合练习 MySQL 时，推荐：

```text
demo_mysql_practice
```

创建语句：

```sql
CREATE DATABASE IF NOT EXISTS demo_mysql_practice
    DEFAULT CHARACTER SET utf8mb4
    COLLATE utf8mb4_0900_ai_ci;
```

进入数据库：

```sql
USE demo_mysql_practice;
```

---

## 4. 文件夹命名规范

Windows 文件夹建议使用：

```text
小写字母 + 连字符
```

例如：

```text
mysql-workspace
course-projects
mysql-practice
digital-mall
blog-gp
```

建议保持一致：

- 数据库名：使用下划线
- 文件夹名：使用连字符
- SQL 文件名：使用下划线

示例：

```text
数据库：project_blog_gp
文件夹：blog-gp
SQL 文件：create_article_table.sql
```

---

## 5. SQL 文件命名规范

SQL 文件统一使用：

```text
小写字母 + 下划线 + .sql
```

文件名最好能够回答：

> 这个 SQL 文件执行后会做什么？

### 5.1 按操作命名

推荐：

```text
create_user_table.sql
create_article_indexes.sql
insert_initial_data.sql
query_user_statistics.sql
drop_unused_index.sql
update_article_status.sql
```

不推荐：

```text
aa_index.sql
sql1.sql
test2.sql
new.sql
最终版.sql
```

### 5.2 查询文件

推荐：

```text
find_user_by_email.sql
query_article_statistics.sql
query_monthly_sales.sql
query_active_users.sql
```

不推荐：

```text
user_query.sql
query1.sql
select.sql
```

### 5.3 索引文件

推荐：

```text
create_user_indexes.sql
create_article_title_index.sql
drop_unused_user_index.sql
analyze_order_index.sql
```

单个索引可以写得更具体：

```text
create_user_email_index.sql
```

### 5.4 数据库迁移文件

建议使用：

```text
V序号__说明.sql
```

示例：

```text
V001__create_user_table.sql
V002__create_article_table.sql
V003__add_email_column.sql
V004__create_email_index.sql
```

注意序号和说明之间是两个下划线。

这种格式以后可以方便接入 Flyway。

---

## 6. 备份文件命名规范

完整数据库备份建议与数据库名称保持一致。

数据库：

```text
project_blog
```

备份：

```text
project_blog.sql
```

保留时间版本时：

```text
project_blog_2026-08-05.sql
project_blog_2026-08-20.sql
```

删除前专门制作的备份：

```text
project_blog_before_delete_2026-08-05.sql
```

推荐格式：

```text
数据库名_用途或日期.sql
```

例如：

```text
demo_mysql_practice_2026-08-05.sql
project_blog_before_delete_2026-08-05.sql
```

---

## 7. 毕设项目 `blog_gp` 的推荐结构

```text
mysql-workspace/
├── projects/
│   └── blog-gp/
│       ├── README.md
│       └── sql/
│           ├── schema/
│           │   ├── create_database.sql
│           │   ├── create_user_table.sql
│           │   └── create_article_table.sql
│           │
│           ├── data/
│           │   └── initial_data.sql
│           │
│           ├── indexes/
│           │   ├── create_user_indexes.sql
│           │   └── create_article_indexes.sql
│           │
│           ├── queries/
│           │   ├── query_article_list.sql
│           │   └── query_user_statistics.sql
│           │
│           └── migrations/
│               └── V001__initial_schema.sql
│
└── backups/
    └── projects/
        └── blog-gp/
            ├── blog_gp_2026-08-05.sql
            └── blog_gp_before_delete_2026-08-05.sql
```

其中：

- `projects/blog-gp`：数据库相关源文件
- `backups/projects/blog-gp`：完整数据库快照

---

## 8. 删除数据库前的标准流程

```text
导出完整 SQL
    ↓
确认文件不是 0 KB
    ↓
创建 tmp_ 恢复测试库
    ↓
运行 SQL 文件
    ↓
检查表数量和重要数据
    ↓
确认项目不再连接原数据库
    ↓
保留异地备份
    ↓
删除原数据库
```

示例：

```text
原数据库：project_blog
测试库：tmp_project_blog_restore
```

确认恢复成功后再执行：

```sql
DROP DATABASE project_blog;
```

对于毕设、课程设计等重要数据库，建议至少保留：

- 本机完整备份一份
- 其他磁盘或云端备份一份
- 项目中的初始化和迁移脚本一份

---

## 9. 最终规范速查

### 数据库

```text
demo_技术_用途
project_项目名
tmp_测试用途
```

示例：

```text
demo_mysql_practice
project_blog
tmp_blog_restore
```

### 文件夹

```text
小写字母 + 连字符
```

示例：

```text
mysql-workspace
course-projects
mysql-practice
```

### SQL 文件

```text
小写字母 + 下划线 + .sql
```

示例：

```text
create_user_table.sql
query_article_statistics.sql
create_email_index.sql
```

### 内容分类

```text
backups   完整数据库快照
projects  项目长期维护的 SQL
demos     独立技术练习
temp      一次性测试
```

> `projects` 保存数据库相关源文件，`backups` 保存完整恢复快照，`demos` 保存学习练习，`temp` 保存随时可删除的测试文件。
