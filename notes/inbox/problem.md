没有使用 this 关键字也能在 constructor 中使用类内的方法？

Mybatis TypeAliasesRegister 构造器内调用本类内的方法（既不没有被static修饰，也没通过实例调用）

[docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.8.7](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.8.7)

[docs.oracle.com/javase/specs/jls/se26/html/jls-15.html?utm_source=chatgpt.com#jls-15.12.4.1](https://docs.oracle.com/javase/specs/jls/se26/html/jls-15.html?utm_source=chatgpt.com#jls-15.12.4.1)








List<> list = new ArrayList<>();

java17 和 java26  List difference https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/util/List.html

15.12.4.4. Locate Method to Invoke


我没有写 `new ConsultLevelController(...)`，Spring 是在哪里调用这个 constructor 的？

```Java
@RestController 
@RequestMapping("/consultLevel") 
public class ConsultLevelController { 
 
    private final ConsultLevelService service; 
 
    public ConsultLevelController(ConsultLevelService service) { 
        this.service = service; 
    } 
} 当启动的时候 spring 是怎么把 ConsultLevelService  Bean 注入到  public ConsultLevelController(ConsultLevelService service) { 
        this.service = service; 
    }
```

Spring MVC   + Spring WebFlux 为什么冗余？



形参中的 slash / 到底要不要加？ springmvc 会自动处理它吗？

@RequestMapping 注解中的形参 name 有什么用？
