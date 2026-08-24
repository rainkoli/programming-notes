# Java 变量初始化总结

## 1. Java 变量必须先有值才能使用

Java 规定：变量在被读取之前必须已经完成初始化。

不同类型、不同位置的变量，初始化规则不同。

------------------------------------------------------------------------

## 2. 会自动初始化的变量

以下变量创建时会自动获得默认值：

-   类变量（static variable）
-   实例变量（成员变量）
-   数组元素（array component）

### 示例：成员变量

``` java
class Student {
    int age;
    String name;
}
```

创建对象：

``` java
Student s = new Student();
```

结果：

``` text
s.age = 0
s.name = null
```

------------------------------------------------------------------------

## 3. 基本数据类型默认值

  类型      默认值
  --------- -----------
  byte      0
  short     0
  int       0
  long      0L
  float     0.0f
  double    0.0d
  char      '\\u0000'
  boolean   false

------------------------------------------------------------------------

## 4. 引用类型默认值

所有 reference type 默认：

``` java
null
```

例如：

``` java
String s;
```

作为成员变量时：

``` text
s = null
```

包括：

-   String
-   Integer
-   Object
-   自定义类对象

------------------------------------------------------------------------

## 5. 数组初始化规则

### int 数组

``` java
int[] dp = new int[5];
```

数组元素自动初始化：

``` text
[0,0,0,0,0]
```

原因：

数组元素属于 array component，会使用默认值。

------------------------------------------------------------------------

### Integer 数组

``` java
Integer[] dp = new Integer[5];
```

结果：

``` text
[null,null,null,null,null]
```

原因：

Integer 是引用类型。

------------------------------------------------------------------------

## 6. 局部变量不会自动初始化

方法内部：

``` java
public static void main(String[] args){
    int x;
    System.out.println(x);
}
```

编译错误：

``` text
variable x might not have been initialized
```

必须：

``` java
int x = 0;
```

------------------------------------------------------------------------

## 7. DP 中的意义

例如：

``` java
int[] dp = new int[r];
```

初始：

``` text
dp = [0,0,0,...]
```

所以：

``` java
dp[j] = dp[j] + num;
```

不是空值相加。

第一次可能：

``` text
0 + num
```

之后：

``` text
之前保存的状态 + 当前数字
```

------------------------------------------------------------------------

## 8. 官方定义位置

Java Language Specification:

-   JLS 4.12.5 Initial Values of Variables

核心规则：

-   数组组件、成员变量自动初始化
-   基本类型使用默认值
-   引用类型默认 null
-   局部变量必须显式赋值
