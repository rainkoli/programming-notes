# Programming Notes

面向长期积累的个人编程知识库。目录按“内容的职责”组织，而不是按文件格式或临时学习阶段组织。

Personal programming knowledge base designed for long-term maintenance. Content is organized by responsibility and subject.

## Repository model / 仓库模型

```text
programming-note/
├── notes/                         # 知识笔记：按领域分类
│   ├── README.md                  # 笔记总索引与放置规则
│   ├── inbox/                     # 暂时无法归类的收件箱
│   ├── java/
│   │   ├── language/              # 语言、类型系统、注解与 JLS
│   │   ├── jvm/                   # JVM、字节码与 JIT
│   │   └── frameworks/            # Spring、MyBatis、JUnit 等
│   ├── database/
│   ├── algorithms/
│   │   ├── topics/                # 可复用专题
│   │   └── problems/              # 按题目来源归档
│   ├── source-code-analysis/      # JDK、Spring、MyBatis 源码分析
│   ├── career/
│   └── tools/
├── demos/                         # 可运行或可复现的代码实验
├── resources/                     # 链接、外部参考与历史资料
├── assets/                        # GitHub Pages 静态资源
├── _includes/                     # Jekyll 约定目录
├── index.md                       # GitHub Pages 门户
└── README.md
```

## Navigation / 导航

| Area / 分类 | Index / 索引 | Responsibility / 职责 |
| --- | --- | --- |
| Notes / 笔记 | [notes](./notes/README.md) | 全部知识性 Markdown 的唯一主目录 |
| Java | [java](./notes/java/README.md) | Java 语言、JVM 与框架 |
| Databases / 数据库 | [database](./notes/database/README.md) | 数据库原理、SQL 与 MySQL |
| Algorithms / 算法 | [algorithms](./notes/algorithms/README.md) | 算法专题和题解 |
| Source-code analysis / 源码分析 | [source-code-analysis](./notes/source-code-analysis/README.md) | 实现细节、调用链与生命周期 |
| Career / 职业 | [career](./notes/career/README.md) | 面试与职业发展 |
| Tools / 工具 | [tools](./notes/tools/README.md) | 构建、正则表达式和开发工具 |
| Demos / 示例 | [demos](./demos/README.md) | 可运行、可编译或用于复现行为的代码 |
| Resources / 资源 | [resources](./resources/README.md) | 外部链接、参考资料和历史归档 |

## Placement rules / 放置规则

1. 解释概念、记录结论、复盘机制的内容放入 `notes/`。
2. 能运行、编译或复现问题的代码放入 `demos/`。
3. 外部链接、模板和不属于原创知识笔记的材料放入 `resources/`。
4. 一份内容只保留一个主位置；跨领域关系使用链接，不复制文件。
5. 暂时无法判断归属时放入 [notes/inbox](./notes/inbox/README.md)，整理后移出。
6. 只有出现实际内容时才新建分类目录，通常不为远期计划预建空目录。
7. 已公开的路径尽量保持稳定；需要重构时同步更新索引和内部链接。

## Important boundaries / 重要边界

- Java API 的用法和行为属于 `notes/java/`；JDK 类的实现细节属于 `notes/source-code-analysis/jdk/`。
- Spring/MyBatis 的使用、配置和概念属于 `notes/java/frameworks/`；源码调用链属于 `notes/source-code-analysis/`。
- `notes/algorithms/topics/` 存放可复用知识；`notes/algorithms/problems/` 存放具体题目。
- `resources/` 面向人阅读和复用；`assets/` 仅服务于站点或文档引用。

## Naming / 命名

- 普通 Markdown 使用 lowercase kebab-case，例如 `method-resolution.md`。
- 目录导航统一使用 GitHub 约定的 `README.md`，它是普通文章命名规则的明确例外。
- Java 源文件保留与公开类一致的大小写。
- 题目编号、规范章节号等稳定标识可以作为文件名前缀。

## Selected notes / 精选笔记

- [Java polymorphism / Java 多态](./notes/java/language/object-oriented-programming/polymorphism.md)
- [JLS Chapter 5: Conversions and Contexts](./notes/java/language/jls/chapter-05-conversions-and-contexts.md)
- [JDK source analysis: java.lang.Short](./notes/source-code-analysis/jdk/java-lang-short.md)
- [Spring dependency injection](./notes/java/frameworks/spring/dependency-injection-detailed-notes.md)
- [MySQL index performance](./notes/database/mysql/mysql-index-performance-guide.md)
- [ACM-ICPC / OI knowledge system](./notes/algorithms/topics/acm-knowledge-system.md)
- [LeetCode records](./notes/algorithms/problems/leetcode/README.md)
- [Nowcoder Weekly Round 155](./notes/algorithms/problems/nowcoder/round-155.md)

## Notice / 说明

题目描述和平台材料的版权归原作者及对应平台所有。本仓库仅用于个人学习、复习和题解记录。MIT License 仅适用于仓库中的原创笔记、说明与代码。
