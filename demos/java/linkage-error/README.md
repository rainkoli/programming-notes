# Linkage error 与类名大小写示例

该示例用于观察 Java 源文件类名、编译后类名大小写与二进制链接行为。

| 文件 | 作用 |
| --- | --- |
| [`HelloWorld.java`](./HelloWorld.java) | 示例入口源码，引用小写类名 `patch` |
| [`Patch.java`](./Patch.java) | 声明大写类名 `Patch` 的源码 |
| `helloworld.class` | 为复现实验保留的预编译字节码 |
| `patch.class` | 为复现实验保留的预编译字节码 |

这里的两个 `.class` 是固定测试材料，不应按普通构建产物删除。

返回 [Java 示例](../README.md) 或 [Demos](../../README.md)。
