# Java 中的协变、逆变与不变总结

## 1. 基础类型关系

假设有以下继承关系：

```java
class Person {
}

class Student extends Person {
}

class Teacher extends Person {
}
```

那么：

```text
Student 是 Person 的子类
Teacher 是 Person 的子类
```

因此，普通对象之间可以向上转型：

```java
Student student = new Student();
Person person = student;
```

但是，这种继承关系不能直接推广到泛型类型：

```text
Student <: Person
```

并不能推出：

```text
List<Student> <: List<Person>
```

这是理解 Java 泛型变型的关键。

---

# 2. Covariant：协变

## 2.1 协变的含义

如果 `Student` 是 `Person` 的子类，并且下面的关系也成立：

```text
Container<Student> 是 Container<Person> 的子类型
```

那么称 `Container` 对类型参数是**协变的**。

用符号表示：

```text
Student <: Person
Container<Student> <: Container<Person>
```

协变保持了原来的继承方向。

---

## 2.2 Java 数组是协变的

Java 数组支持协变：

```java
Student[] students = new Student[1];

Person[] people = students;
```

因为 `Student` 是 `Person` 的子类，所以 Java 允许：

```text
Student[] → Person[]
```

但是，这种设计可能导致运行时异常：

```java
people[0] = new Teacher();
```

从变量 `people` 的声明类型看：

```text
people 是 Person[]
Teacher 是 Person 的子类
```

所以代码可以通过编译。

但是，`people` 实际引用的数组仍然是：

```text
Student[]
```

因此 JVM 运行时会检查：

```text
Teacher 能否存入 Student[]？
```

答案是否定的，所以抛出：

```text
java.lang.ArrayStoreException
```

完整示例：

```java
public class Demo {
    public static void main(String[] args) {
        Student[] students = {new Student()};

        Person[] people = students;

        people[0] = new Teacher(); // 运行时抛出 ArrayStoreException
    }
}
```

数组协变的特点：

```text
优点：使用方便
缺点：某些类型错误只能在运行时发现
```

---

## 2.3 Java 泛型中的协变：`? extends T`

Java 泛型默认不协变，但可以通过上界通配符表达“协变式访问”：

```java
List<Student> students = new ArrayList<>();
students.add(new Student());

List<? extends Person> people = students;
```

`List<? extends Person>` 的意思是：

```text
这是一个装着某种 Person 子类型的 List，
但具体是哪一种子类型不知道。
```

它可能是：

```text
List<Person>
List<Student>
List<Teacher>
```

在上面的代码中，它实际是：

```text
List<Student>
```

### 可以安全读取

```java
Person person = people.get(0);
```

无论实际元素是 `Student`、`Teacher`，还是其他 `Person` 子类，都可以向上转型为 `Person`。

因此：

```text
从 List<? extends Person> 中读取 Person 是安全的
```

### 不能安全写入

下面代码不能通过编译：

```java
people.add(new Person());
people.add(new Student());
people.add(new Teacher());
people.set(0, new Student());
```

原因是编译器不知道这个列表实际是哪一种类型。

例如，它可能是：

```text
List<Student>
```

此时不能放入 `Teacher`。

它也可能是：

```text
List<Teacher>
```

此时不能放入 `Student`。

所以编译器禁止添加任何具体的 `Person` 对象。

唯一可以添加的是 `null`，但实际开发中通常不应该依赖这一点。

---

## 2.4 协变示例：生产者

当一个集合主要负责提供数据时，可以使用 `? extends T`：

```java
static void printPeople(List<? extends Person> people) {
    for (Person person : people) {
        System.out.println(person);
    }
}
```

调用：

```java
List<Student> students = List.of(new Student());
List<Teacher> teachers = List.of(new Teacher());

printPeople(students);
printPeople(teachers);
```

这里的方法只读取元素，不向集合中写入元素。

因此：

```text
Producer Extends
```

也就是：

```text
生产者使用 ? extends T
```

---

# 3. Invariant：不变

## 3.1 不变的含义

如果 `Student` 是 `Person` 的子类，但是：

```text
Container<Student>
```

和：

```text
Container<Person>
```

之间没有继承关系，那么称 `Container` 对类型参数是**不变的**。

也就是：

```text
Student <: Person
```

但是：

```text
Container<Student> 不是 Container<Person> 的子类型
Container<Person> 也不是 Container<Student> 的子类型
```

Java 泛型默认是不变的。

---

## 3.2 Java 泛型不变示例

下面代码不能通过编译：

```java
List<Student> students = new ArrayList<>();

List<Person> people = students; // 编译错误
```

IDEA 会提示类似：

```text
Required type:
List<Person>

Provided:
List<Student>
```

原因不是 `Student` 与 `Person` 没有继承关系，而是：

```text
List<Student> 不是 List<Person> 的子类型
```

---

## 3.3 为什么 Java 泛型必须默认不变

假设 Java 允许：

```java
List<Student> students = new ArrayList<>();
List<Person> people = students;
```

此时两个变量将引用同一个集合：

```text
students ──┐
           ├──> 同一个 ArrayList<Student>
people  ───┘
```

由于 `people` 的声明类型是 `List<Person>`，理论上可以添加任何 `Person` 子类：

```java
people.add(new Teacher());
```

但是实际集合是：

```text
List<Student>
```

最终会产生：

```text
List<Student>
├── Student
└── Teacher
```

这样就破坏了类型安全。

因此 Java 在编译阶段直接禁止：

```java
List<Person> people = students;
```

---

## 3.4 不可变集合也不会自动变成协变

下面是一个不可修改的集合：

```java
List<Student> students = List.of(new Student());
```

虽然集合不能修改，但下面的赋值仍然不能通过编译：

```java
List<Person> people = students; // 编译错误
```

原因是：

```text
Java 泛型默认不变
```

集合对象是否可修改，与泛型类型是否协变不是同一个概念。

更准确地说：

```text
不可修改的集合适合通过 ? extends T 进行协变式读取，
但它不会自动成为 List<Person>。
```

正确写法：

```java
List<? extends Person> people = students;
```

---

# 4. Contravariant：逆变

## 4.1 逆变的含义

如果 `Student` 是 `Person` 的子类，但容器类型的继承方向反过来：

```text
Container<Person> 是 Container<Student> 的子类型
```

那么称 `Container` 对类型参数是**逆变的**。

用符号表示：

```text
Student <: Person
Container<Person> <: Container<Student>
```

逆变反转了原来的继承方向。

---

## 4.2 Java 泛型中的逆变：`? super T`

Java 使用下界通配符表达逆变式访问：

```java
List<Person> people = new ArrayList<>();

List<? super Student> students = people;
```

`List<? super Student>` 的意思是：

```text
这是一个装着 Student 或 Student 某个父类型的 List，
但具体是哪一种类型不知道。
```

它可能是：

```text
List<Student>
List<Person>
List<Object>
```

---

## 4.3 可以写入 Student 及其子类

下面代码可以通过编译：

```java
List<? super Student> list = new ArrayList<Person>();

list.add(new Student());
```

因为无论实际列表是：

```text
List<Student>
List<Person>
List<Object>
```

都一定能够接收 `Student`。

如果还有 `GraduateStudent extends Student`：

```java
class GraduateStudent extends Student {
}
```

那么也可以添加：

```java
list.add(new GraduateStudent());
```

因为 `GraduateStudent` 是 `Student` 的子类。

---

## 4.4 读取时只能安全地当作 Object

下面代码不能通过编译：

```java
Student student = list.get(0);
```

原因是 `list` 的实际类型可能是：

```text
List<Person>
```

而其中可能原本存放了其他 `Person` 对象。

例如：

```java
List<Person> people = new ArrayList<>();
people.add(new Teacher());

List<? super Student> list = people;
```

此时从 `list` 中读取的第一个对象可能是 `Teacher`，不能保证是 `Student`。

所以安全的读取类型只能是：

```java
Object value = list.get(0);
```

---

## 4.5 逆变示例：消费者

当一个集合主要负责接收数据时，可以使用 `? super T`：

```java
static void addStudent(List<? super Student> destination) {
    destination.add(new Student());
}
```

它可以接收：

```java
List<Student> students = new ArrayList<>();
List<Person> people = new ArrayList<>();
List<Object> objects = new ArrayList<>();

addStudent(students);
addStudent(people);
addStudent(objects);
```

因为这些集合都可以接收一个 `Student`。

因此：

```text
Consumer Super
```

也就是：

```text
消费者使用 ? super T
```

---

# 5. PECS 原则

Java 泛型中常用的记忆规则是：

```text
PECS
```

它表示：

```text
Producer Extends
Consumer Super
```

中文可以理解为：

```text
生产者使用 extends
消费者使用 super
```

---

## 5.1 Producer Extends

当参数负责提供元素时：

```java
static void copyPeople(
        List<? extends Person> source
) {
    for (Person person : source) {
        System.out.println(person);
    }
}
```

`source` 只负责产生 `Person` 对象，因此使用：

```java
? extends Person
```

可以传入：

```java
List<Person>
List<Student>
List<Teacher>
```

---

## 5.2 Consumer Super

当参数负责接收元素时：

```java
static void addStudents(
        List<? super Student> destination
) {
    destination.add(new Student());
}
```

`destination` 负责接收 `Student`，因此使用：

```java
? super Student
```

可以传入：

```java
List<Student>
List<Person>
List<Object>
```

---

## 5.3 同时读取和写入时通常不用通配符

如果一个方法既要读取 `Student`，又要写入 `Student`，通常使用确定类型：

```java
static void processStudents(List<Student> students) {
    Student student = students.get(0);
    students.add(new Student());
}
```

此时使用：

```java
List<Student>
```

而不是：

```java
List<? extends Student>
```

或：

```java
List<? super Student>
```

---

# 6. 三种变型的对比

| 类型 | Java 表达方式 | 关系 | 主要能力 |
|---|---|---|---|
| 协变 Covariant | `? extends T` | 保持继承方向 | 安全读取 |
| 不变 Invariant | `T` | 泛型类型之间无继承关系 | 可正常读写确定类型 |
| 逆变 Contravariant | `? super T` | 反转继承方向 | 安全写入 |
| 数组协变 | `Sub[] → Super[]` | 保持继承方向 | 可读写，但可能运行时报错 |

---

# 7. 读取和写入规则总结

## `List<Person>`

类型确定为 `Person`：

```java
List<Person> people = new ArrayList<>();

people.add(new Person());
people.add(new Student());
people.add(new Teacher());

Person person = people.get(0);
```

规则：

```text
可以写入 Person 及其子类
读取结果是 Person
```

---

## `List<? extends Person>`

类型是某个未知的 `Person` 子类型：

```java
List<? extends Person> people = List.of(new Student());
```

规则：

```text
可以读取为 Person
不能写入任何具体的 Person 对象
```

示例：

```java
Person person = people.get(0);

// people.add(new Person());   // 编译错误
// people.add(new Student());  // 编译错误
// people.add(new Teacher());  // 编译错误
```

---

## `List<? super Student>`

类型是 `Student` 或它的某个父类型：

```java
List<? super Student> students = new ArrayList<Person>();
```

规则：

```text
可以写入 Student 及其子类
读取时只能安全地当作 Object
```

示例：

```java
students.add(new Student());

Object value = students.get(0);

// Student student = students.get(0); // 编译错误
```

---

# 8. 数组与泛型的区别

## 数组

```java
Student[] students = new Student[1];
Person[] people = students;

people[0] = new Teacher(); // 运行时异常
```

特点：

```text
数组支持协变
错误可能推迟到运行时
运行时保留实际元素类型
```

异常：

```text
ArrayStoreException
```

---

## 泛型

```java
List<Student> students = new ArrayList<>();

List<Person> people = students; // 编译错误
```

特点：

```text
泛型默认不变
错误在编译阶段被阻止
类型更加安全
```

可以通过通配符表达受限的变型：

```java
List<? extends Person> producer = students;
List<? super Student> consumer = new ArrayList<Person>();
```

---

# 9. 一个完整示例

```java
import java.util.ArrayList;
import java.util.List;

class Person {
}

class Student extends Person {
}

class Teacher extends Person {
}

class GraduateStudent extends Student {
}

public class VarianceDemo {

    public static void main(String[] args) {
        invariantExample();
        covariantExample();
        contravariantExample();
    }

    private static void invariantExample() {
        List<Student> students = new ArrayList<>();

        // List<Person> people = students;
        // 编译错误：List<Student> 不是 List<Person> 的子类型
    }

    private static void covariantExample() {
        List<Student> students = new ArrayList<>();
        students.add(new Student());

        List<? extends Person> people = students;

        Person person = people.get(0);

        // people.add(new Student());
        // people.add(new Teacher());
        // 编译错误：实际元素类型未知
    }

    private static void contravariantExample() {
        List<Person> people = new ArrayList<>();

        List<? super Student> students = people;

        students.add(new Student());
        students.add(new GraduateStudent());

        Object value = students.get(0);

        // Student student = students.get(0);
        // 编译错误：读取结果只能保证是 Object
    }
}
```

---

# 10. 最终记忆

```text
Student 是 Person 的子类
```

但是：

```text
List<Student> 不是 List<Person> 的子类
```

这是 Java 泛型的不变性。

协变：

```java
List<? extends Person>
```

记忆：

```text
可以读 Person
不能写具体对象
Producer Extends
```

不变：

```java
List<Person>
```

记忆：

```text
类型确定
可以正常读写 Person 及其子类
```

逆变：

```java
List<? super Student>
```

记忆：

```text
可以写 Student 及其子类
读取只能安全地作为 Object
Consumer Super
```

最终口诀：

```text
PECS：
Producer Extends
Consumer Super
```
