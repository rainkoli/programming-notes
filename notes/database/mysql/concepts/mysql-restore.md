# MySQL Binlog 误删数据恢复流程（Markdown完整版，新增参数详解章节）

## 一、前置准备

> 📌 新增：binlog状态与日志格式查询，确认是否具备恢复条件
> 登录MySQL客户端执行下面SQL，确认binlog开启状态、当前binlog文件、binlog日志格式：
```sql
-- 查看当前所有binlog文件列表
SHOW BINARY LOGS;
-- 查看binlog_format日志格式变量（注意原语句笔误：BOLG_FORMAT → BINLOG_FORMAT）
SHOW VARIABLES LIKE 'binlog_format';
```

**查询结果解读：**
1. `SHOW BINARY LOGS;`
- 返回所有二进制日志文件清单、每个文件大小，`Log_name`列就是binlog文件名，用来确认目标`binlog.001224`、`binlog.001225`真实存在；
- `File_size`代表文件字节大小；
- 当前正在写入的binlog为列表最后一条。

2. `SHOW VARIABLES LIKE 'binlog_format';`
- `Value`为`ROW`：行模式，**可以解析出行级原始数据，支持本教程的误删恢复**；
- `Value`为`STATEMENT`：语句模式，只能看到执行的SQL语句，看不到删除的行数据，无法通过本方式拿到删除前原始行值；
- `Value`为`MIXED`：混合模式，不一定能拿到完整行数据，生产推荐设置为`ROW`。

1. 确认环境：MySQL开启**binlog行模式（Row）**，已定位目标binlog文件（本次为`binlog.001224`、`binlog.001225`）
2. 获取关键信息：
	- 数据库名：`demo_mysql_practice`
	- 目标表：`temp_students`
	- 数据删除时间：`2026-08-06 10:34:23`
3. 切换CMD工作目录至MySQL数据目录：

```cmd
cd C:\Program Files\MySQL\MySQL Server 8.0\data
```

## 二、解析Binlog日志导出可读文件

### 执行解析命令（单行写法，无续行符坑）

```cmd
mysqlbinlog --base64-output=decode-rows -vv -d demo_mysql_practice binlog.001224 binlog.001225 > raw_log.sql
```

### 🔍 命令逐段详解 + 参数全称说明

> 注意：若误写为`raw_log.sqlb`属于文件名笔误，标准导出后缀为`.sql`，`>`为系统输出重定向语法

1. **`mysqlbinlog`**
	MySQL官方自带二进制日志解析工具，读取binlog二进制日志，将加密二进制操作日志转为可读文本。
2. **`--base64-output=decode-rows`**
	无短参数别名；Row行模式下行变更数据默认base64编码存储，该参数强制解码行数据，才能看到`@1=1 @2='AA'`这类字段明细，不配置仅能看到执行DELETE，看不到具体删除值。
3. **`-vv`**
	全称：`--verbose --verbose`（单个`-v`=`--verbose`）

- 单`-v`：基础解析SQL结构；
- `vv`双层开启：额外输出字段数据类型、是否可空等元信息注释，例如`@1=1 /* INT meta=0 nullable=1 is_null=0 */`，避免构造恢复SQL时字段类型出错。

4. **`-d demo_mysql_practice`**
	短参数`-d`全称`--database`，等价写法`--database=demo_mysql_practice`；
	作用：仅解析输出指定数据库的日志事件，过滤其他库无关日志、精简内容；控制台出现的GTID相关WARNING为官方正常提示，仅标记事务标识，**不影响数据解析结果**。
5. **`binlog.001224 binlog.001225`**
	按时间升序排列的待解析连续binlog文件，顺序错乱会导致事务时间线解析异常。
6. **`> raw_log.sql`**
	CMD/Shell输出重定向符号，把终端刷屏的解析结果持久写入文件，便于检索分析；去掉该符号则日志直接打印在命令行窗口。

### 核心参数对照表

| 短参数                        | 完整长参数            | 核心功能                         |
| ----------------------------- | --------------------- | -------------------------------- |
| `-v`                          | `--verbose`           | 开启行日志基础可读解析           |
| `-vv`                         | `--verbose --verbose` | 双层冗余输出，附带字段元数据注释 |
| `-d`                          | `--database`          | 按数据库名称过滤日志事件         |
| `--base64-output=decode-rows` | 无短参数别名          | 解码Row模式下base64加密的行数据  |

- 注意：控制台`--database`相关WARNING为提示信息，**不影响日志解析，无需处理**

## 三、检索定位误删记录

1. 用编辑器打开生成的`raw_log.sql`
2. 搜索关键词：`DELETE FROM demo_mysql_practice.temp_students`
3. 提取行数据：

```sql
### DELETE FROM `demo_mysql_practice`.`temp_students`
### WHERE
###   @1=1  /* INT类型，第1列 */
###   @2='AA' /* VARCHAR(80)，第2列 */
```

- 规则：`@1`=表第1个字段、`@2`=表第2个字段，依次对应建表字段顺序

## 四、反向构造恢复SQL语句

### 1. 转换逻辑

将`DELETE WHERE @n=值` → 改写为`INSERT (字段名) VALUES(对应值)`

### 2. 本次示例语句

假设表字段依次为`id`、`name`：

```sql
USE demo_mysql_practice;
INSERT INTO temp_students (id, name)
VALUES (1, 'AA');
```

> 若有多条删除记录，按相同逻辑逐条批量生成INSERT语句

## 五、执行数据恢复

1. 登录MySQL客户端，进入目标库

```sql
mysql -u账号 -p密码
USE demo_mysql_practice;
```

2. 逐条/批量执行构造好的`INSERT`语句
3. 查询表校验数据是否成功恢复：

```sql
SELECT * FROM temp_students;
```

## 六、关键避坑要点

1. ❌ 禁止直接修改`raw_log.sql`解析文件，仅用于查看分析
2. ❌ `mysqlbinlog`是系统CMD命令，**不能在mysql>交互窗口执行**
3. ❌ 新手优先单行命令，Windows多行续行才用`^`，单行无需该符号
4. ❌ 命令结尾不要加SQL分号`;`，会触发参数识别异常
5. ✅ 目录必须匹配：要么CMD切到binlog所在目录，要么给binlog写完整绝对路径
6. ✅ binlog文件全程只读解析，解析操作不会改动原始日志文件
7. ✅ 恢复前优先执行`SHOW BINARY LOGS; SHOW VARIABLES LIKE 'binlog_format';`校验环境，不满足ROW模式无法解析原始行数据

## 七、拓展预防建议

1. 生产环境定期备份全量数据+binlog归档
2. 执行`DELETE/UPDATE`前先`SELECT`核对范围，事务操作先`BEGIN`校验再`COMMIT`
3. 高危SQL可开启MySQL审计日志，便于后续追溯操作人
4. 上线环境my.cnf/my.ini配置`binlog_format=ROW`，保证误删时有完整行日志可恢复
