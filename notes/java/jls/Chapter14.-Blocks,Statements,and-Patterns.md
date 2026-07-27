

## 14.11. The `switch` Statement

## 14.20.2. Execution of `try`-`finally` and `try`-`catch`-`finally`

一句话，finally优先：finally里有return，不管try里有异常还是返回值直接走finally里的return

```java
package cn.test;

public class Test05 {
    public static void main(String[] args) {
//        int tt = tt1();
//        int tt = tt2();
//        int tt = tt3();
        int tt = tt4();
        System.out.println(tt);
    }


    // 定义方法 - 如果返回值在try中，且有finally则先执行finally，再返回结果
    public static int tt1(){
        try {
            return 11;
        } catch (Exception e) {
            throw new RuntimeException(e);
        } finally {
            System.out.println("finally");
        }
    }
    // 如果finally中有return返回值，则覆盖try和catch中的返回值
    public static int tt2(){
        try {
            return 10;
        } catch (Exception e) {
            return 30;
        } finally {
            return 20;
        }
    }
    // 如果finally中有return返回值，则屏蔽try和catch中抛出的异常
    public static int tt3(){
        try {
            int a = 10 / 0;
            return 10;
        } catch (Exception e) {
            System.out.println("我是异常处理类");
            throw new RuntimeException("我是异常处理类");
        } finally {
            System.out.println("finally");
            return 200;
        }
    }
    // 如果finally中有异常，则覆盖catch中的异常
    public static int tt4(){
        try {
            int a = 10 / 0;
            return 10;
        } catch (Exception e) {
            System.out.println("我是异常处理类");
            throw new RuntimeException("我是异常处理类");
        } finally {
            System.out.println("finally");
            throw new RuntimeException("我是finally抛出的异常");
        }
    }



}
```

