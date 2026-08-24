# Java Encapsulation vs. MyBatis Object Mapping

## 1. Core Question

A common misunderstanding is to treat `private` as equivalent to **encapsulation**.

That is not accurate.

> `private` is an access-control mechanism that can be used to implement encapsulation, but `private` itself is not encapsulation.

Encapsulation is a broader object-oriented design idea:

- hide internal state and implementation details;
- expose only necessary behavior or access points;
- control how object state is read or modified;
- protect object invariants;
- reduce coupling between callers and implementation details.

So the relationship is:

```text
private ≠ encapsulation

private ⊂ mechanisms used to implement encapsulation
```

---

## 2. What `private` Actually Does

Consider:

```java
public class Person {
    private int age;
}
```

The field `age` cannot be accessed directly from outside the class:

```java
Person person = new Person();
person.age = 18; // compile-time error
```

This demonstrates **access control** and contributes to encapsulation because the internal state is hidden.

However, simply adding a setter does not automatically produce good encapsulation:

```java
public class Person {

    private int age;

    public void setAge(int age) {
        this.age = age;
    }
}
```

Now external code can still do:

```java
person.setAge(-100);
```

The field is `private`, but the object does not protect its own state.

A better design is:

```java
public class Person {

    private int age;

    public void setAge(int age) {
        if (age < 0 || age > 150) {
            throw new IllegalArgumentException("Invalid age");
        }

        this.age = age;
    }
}
```

The important point is:

> Encapsulation is not only about hiding data. It is also about controlling how the data can be changed.

---

## 3. Encapsulation and Access Control

Java provides several access modifiers:

```text
private
default
protected
public
```

They control **who is allowed to access a class member**.

For example:

```java
private List<Employee> employees;
```

means that the `employees` field itself cannot be accessed directly from arbitrary external code.

If the class provides:

```java
public List<Employee> getEmployees() {
    return employees;
}
```

or:

```java
public void addEmployee(Employee employee) {
    employees.add(employee);
}
```

then the class exposes controlled access to that internal state.

Therefore:

```text
Access modifier
    ↓
language-level access control

Encapsulation
    ↓
object-oriented design principle
```

Access modifiers are tools. Encapsulation is the design goal.

---

# 4. Case Study: MyBatis `Department` and `Employee`

## 4.1 Original `Department` Class

The original `Department` entity was:

```java
package com.hs.homework.August.day19mybatis.entity;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.List;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Department {
    private long did;
    private String dname;
}
```

Notice that although `java.util.List` is imported, the class does **not** contain:

```java
private List<Employee> employees;
```

---

## 4.2 `Employee` Class

```java
package com.hs.homework.August.day19mybatis.entity;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Employee {

    private long eid;
    private String ename;
    private String esex;
    private long eage;
    private long did;

    private Department department;
}
```

This class models an employee and also contains:

```java
private Department department;
```

which represents the department associated with an employee.

---

# 5. MyBatis Mapping

The MyBatis mapper contains:

```xml
<mapper namespace="com.hs.homework.August.day19mybatis.mapper.Day19DepartmentMapper">

    <resultMap
        id="department-employee"
        type="com.hs.homework.August.day19mybatis.entity.Department">

        <id column="did" property="did"/>

        <result
            column="dname"
            property="dname"/>

        <collection
            property="employees"
            ofType="com.hs.homework.August.day19mybatis.entity.Employee">

            <id
                column="eid"
                property="eid"/>

            <result
                column="ename"
                property="ename"/>

            <result
                column="esex"
                property="esex"/>

            <result
                column="eage"
                property="eage"/>

            <result
                column="did"
                property="did"/>

        </collection>

    </resultMap>

    <select
        id="selectById"
        resultMap="department-employee">

        SELECT
            d.dname,
            d.did,
            e.eid,
            e.ename,
            e.eage,
            e.esex
        FROM demo_20260818_mybatis_employee e
        RIGHT JOIN demo_20260818_mybatis_department d
            ON e.did = d.did
        WHERE d.did = #{did}

    </select>

</mapper>
```

The critical part is:

```xml
<collection
    property="employees"
    ofType="com.hs.homework.August.day19mybatis.entity.Employee">
```

The meaning of:

```xml
property="employees"
```

is approximately:

> Put the employee records produced by the SQL result set into the `employees` property of the `Department` object.

Conceptually, MyBatis expects the Java object model to support something similar to:

```java
Department department = new Department();

department.setDid(...);
department.setDname(...);
department.setEmployees(...);
```

---

# 6. Why the Mapping Fails

The actual `Department` class contains only:

```text
Department
├── did
└── dname
```

but the MyBatis mapping expects:

```text
Department
├── did
├── dname
└── employees
```

Therefore the mapping configuration and Java object model do not match.

The relationship is:

```text
MyBatis resultMap
        │
        │ property="employees"
        ↓
Department
        │
        ├── did          ✅ exists
        ├── dname        ✅ exists
        └── employees    ❌ missing
```

So when calling:

```java
System.out.println(mapper.selectById(1L));
```

MyBatis cannot correctly map the `<collection>` result into the `Department` object because the required Java property does not exist.

---

# 7. Correct `Department` Definition

The class should contain the collection property:

```java
package com.hs.homework.August.day19mybatis.entity;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.List;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Department {

    private long did;

    private String dname;

    private List<Employee> employees;
}
```

Now the Java object model matches the MyBatis mapping:

```text
Department
├── did
├── dname
└── List<Employee> employees
```

and:

```xml
<collection property="employees">
```

has a corresponding Java property.

---

# 8. The Role of Lombok `@Data`

Because the class uses:

```java
@Data
```

Lombok generates methods conceptually similar to:

```java
public List<Employee> getEmployees() {
    return employees;
}

public void setEmployees(List<Employee> employees) {
    this.employees = employees;
}
```

So after adding:

```java
private List<Employee> employees;
```

MyBatis can work with the `employees` property through the bean-style property accessors generated by Lombok.

The important distinction is:

```text
private List<Employee> employees;
        │
        └── defines the Java object state

@Data
        │
        └── generates getter/setter and other methods

<collection property="employees">
        │
        └── tells MyBatis which Java property receives the collection
```

These are three different responsibilities.

---

# 9. Is This an Encapsulation Problem?

The statement:

> "The `Department` class was not encapsulated well because the access control for `private List<Employee> employees` is missing."

is not the best description of this bug.

The main problem is **not that access control is missing**.

The main problem is:

> The `employees` property itself is missing from the Java object model.

This distinction is important.

---

## 9.1 Missing Property

Example:

```java
public class Department {

    private long did;

    private String dname;
}
```

but the XML contains:

```xml
<collection property="employees">
```

This is primarily:

> an incomplete object model / object-relational mapping mismatch.

It is not primarily an encapsulation failure.

---

## 9.2 Property Exists but Is Public

For example:

```java
public class Department {

    public List<Employee> employees;
}
```

Now the property exists.

The MyBatis mapping problem may no longer be the same issue, but from an object-oriented design perspective, this exposes the mutable collection directly.

That is where it becomes reasonable to discuss weak encapsulation.

---

## 9.3 Property Exists and Is Private

A more conventional design is:

```java
public class Department {

    private List<Employee> employees;
}
```

with controlled access through methods such as:

```java
public List<Employee> getEmployees() {
    return employees;
}
```

or behavior-oriented methods such as:

```java
public void addEmployee(Employee employee) {
    employees.add(employee);
}

public void removeEmployee(Employee employee) {
    employees.remove(employee);
}
```

Here `private` contributes to encapsulation by hiding the internal field.

---

# 10. Better Ways to Describe This Bug

Instead of saying:

> The `Department` class was not encapsulated correctly.

a more accurate description is:

> The `Department` entity does not define the `employees` property required by `<collection property="employees">`, so the Java object model does not match the MyBatis result mapping.

Another good description is:

> The one-to-many relationship between `Department` and `Employee` is not completely represented in the Java object model.

For a shorter debugging note:

> MyBatis expects `Department.employees`, but the property is missing from `Department`, causing the collection result mapping to fail.

---

# 11. Relationship Between the Java Objects

The database relationship can be understood as:

```text
Department
    1
    │
    │
    N
Employee
```

In Java, a department-to-employees relationship can be represented as:

```text
Department
│
└── List<Employee> employees
```

while an employee-to-department relationship can be represented as:

```text
Employee
│
└── Department department
```

If both sides are modeled:

```text
Department
    │
    │ employees
    ↓
List<Employee>


Employee
    │
    │ department
    ↓
Department
```

this is a bidirectional object relationship.

However, whether both directions are necessary depends on application requirements. A bidirectional relationship should not be added merely because the database has a foreign key.

---

# 12. Three Concepts That Should Be Kept Separate

This example involves three different concepts.

## 12.1 Encapsulation

```java
private List<Employee> employees;
```

The keyword `private` controls direct field access.

This belongs to Java access control and contributes to encapsulation.

---

## 12.2 Object Modeling

```java
private List<Employee> employees;
```

The existence of this field also expresses:

> A `Department` contains or is associated with multiple `Employee` objects.

This is part of the Java domain/object model.

---

## 12.3 MyBatis Mapping

```xml
<collection property="employees">
```

This tells MyBatis:

> Map the child rows in the SQL result set into the `employees` property of `Department`.

This belongs to persistence/result mapping.

---

The three can be summarized as:

```text
private
    ↓
access control / encapsulation mechanism

employees field
    ↓
Java object model

property="employees"
    ↓
MyBatis result mapping
```

The reported bug belongs mainly to the last two categories.

---

# 13. Final Conclusion

`private` does not equal encapsulation.

It is one mechanism that helps implement encapsulation.

In this MyBatis case, the missing:

```java
private List<Employee> employees;
```

should not mainly be described as:

> a failure of encapsulation.

It should be described as:

> a missing Java property that makes the object model inconsistent with the MyBatis `<collection>` mapping.

A precise final statement is:

> `Department` is missing the `List<Employee> employees` property required by `<collection property="employees">`. Therefore MyBatis cannot map the one-to-many query result into the `Department` object correctly. Adding the field completes the Java object model; declaring it `private` is an access-control decision that contributes to encapsulation.

