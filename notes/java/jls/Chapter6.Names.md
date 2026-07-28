# Chapter 6. Names

## 6.6. Access Control

### Misc

不是。

Java 中所谓的**缺省访问权限**，并不是写上 `default`，而是**不写任何访问修饰符**。这种权限通常称为：

- **package-private**
- **package access**
- 中文常称“缺省访问权限”或“包访问权限”

例如：

```java
class Student {
    int age;

    void study() {
    }
}
```

这里的 `Student`、`age` 和 `study()` 前面都没有写 `public`、`protected` 或 `private`，因此它们具有**缺省访问权限**：只能在**同一个包**中访问。

不能写成：

```java
default class Student {   // 编译错误
}
```

`default` 虽然是 Java 关键字，但它有其他用途，例如：

```java
switch (value) {
    default:
        System.out.println("其他情况");
}
```

以及接口中的默认方法：

```java
interface Animal {
    default void eat() {
        System.out.println("eating");
    }
}
```

所以更准确的说法是：

**Java 的缺省访问权限没有对应的关键字；它通过不写访问修饰符来表示。**