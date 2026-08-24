# 报错根本原因

```java
public class User implements Comparable
```

你**原始写法没有泛型**，`Comparable` 裸类型，`compareTo` 参数是 `Object o`，`Object` 类**不存在** `getAge()` **方法**，编译器不知道 `o` 是User对象。

## 问题拆解

1. **裸接口 Comparable（原始类型）**

```java
// 错误写法
public class User implements Comparable{
    @Override
    public int compareTo(Object o) {
        return this.age - o.getAge(); // o 的静态类型是Object，Object没有getAge()，编译报错！
    }
}
```

变量 `o` 编译期类型是 `java.lang.Object`，Object只有 `equals()/hashCode()/toString()`，没有 `getAge()`。Lombok生成getter不改变这个问题。

## 两种标准修复方案

### 方案1：使用泛型 Comparable（推荐，规范写法）

```java
@Getter
@Setter
public class User implements Comparable<User> {
    private String name;
    private byte age;

    @Override
    public int compareTo(User o) {
        // o 直接就是User类型，可以调用getAge()
        return this.age - o.getAge();
    }
}
```

实现 `Comparable<T>` 泛型接口后，重写方法参数直接变成 `User o`，无需强转。

### 方案2：不修改接口，手动强制类型转换（不推荐）

如果你暂时不想加泛型，必须把Object强转为User：

```java
@Override
public int compareTo(Object o) {
    User other = (User) o; // 强制向下转型
    return this.age - other.getAge();
}
```

⚠️ 缺点：类型不安全，传入非User对象会抛出 `ClassCastException`。

## 额外补充知识点

1. Lombok 的 `@Getter` 只是**编译期生成getAge()字节码**，只能在 `User` 实例调用；
2. **编译期看静态类型，不是运行时类型**：编译器只知道 `o` 是Object，不知道运行时传入的是User；
3. 集合TreeSet、TreeMap依赖Comparable，业务代码一律建议带上泛型 `Comparable<实体类>`。

## 拓展优化建议

age是byte，直接相减没问题；如果是Integer包装类，建议使用：

```java
return Integer.compare(this.age, o.getAge());
```

避免数值溢出风险（你这里byte范围小，溢出概率低，但养成习惯更好）。

## 最终正确完整代码

```java
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class User implements Comparable<User> {
    private String name;
    private byte age;

    @Override
    public int compareTo(User other) {
        return this.age - other.getAge();
    }
}
```

如果你想，我顺带给你区分 `Comparable`（内部比较器）和 `Comparator`（外部比较器）的区别。