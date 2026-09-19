Mybatis TypeAliasesRegister 构造器内调用本类内的方法（既不没有被static修饰，也没通过实例调用）

Q：没有使用 this 关键字也能在 constructor 中使用类内的方法

[docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.8.7](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.8.7)

[docs.oracle.com/javase/specs/jls/se26/html/jls-15.html?utm_source=chatgpt.com#jls-15.12.4.1](https://docs.oracle.com/javase/specs/jls/se26/html/jls-15.html?utm_source=chatgpt.com#jls-15.12.4.1)



List<> list = new ArrayList<>();

java17 和 java26  List difference https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/util/List.html

15.12.4.4. Locate Method to Invoke
