# 笔记总索引

`notes/` 是仓库中所有知识性文档的唯一主目录。分类时以“未来最可能从哪里寻找这份内容”为准。

## 领域

| 领域 | 目录 | 收录范围 |
| --- | --- | --- |
| Java | [java](./java/README.md) | 语言、JVM、框架与 Java 生态 |
| 数据库 | [database](./database/README.md) | 数据库原理、SQL、MySQL |
| 算法 | [algorithms](./algorithms/README.md) | 算法专题、题单和单题题解 |
| 源码分析 | [source-code-analysis](./source-code-analysis/README.md) | JDK、Spring、MyBatis 等项目的实现细节 |
| 职业 | [career](./career/README.md) | 面试、求职与职业发展 |
| 工具 | [tools](./tools/README.md) | 构建、正则表达式及开发工具 |
| 临时收件箱 | [inbox](./inbox/README.md) | 尚未确定稳定归属的新笔记 |

## 归档判断

```text
这份内容能否运行或复现？
├── 是 -> demos/
└── 否
    ├── 是外部链接、模板或参考材料 -> resources/
    └── 是自己的知识总结 -> notes/
```

进入 `notes/` 后，再按主要检索领域确定唯一位置。跨领域内容保留一个主文件，其他领域通过链接引用。

## 长期维护规则

1. 不建立永久 `misc`；不确定的内容暂存到 `inbox/`。
2. 同类内容积累到约 3–5 篇，或其边界天然稳定时，再建立子目录。
3. 常规目录深度控制在四层以内；稳定标识如平台名、规范章节、项目名可以形成额外层级。
4. 目录名称描述主题，不描述“新手、临时、最近”等会随时间变化的状态。
5. 文件移动后同步更新根 README、领域索引和站点入口。
6. 旧版本依靠 Git 历史保存；确需保留的历史资料放入 `resources/archive/`。

返回 [仓库首页](../README.md)。
