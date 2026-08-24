# Java `main(String[] args)` Summary

## 1. Meaning of `String[] args`

The standard Java entry method is:

```java
public static void main(String[] args)
```

The parameter consists of:

- `String[]`: an array of strings.
- `args`: the parameter name.

It stores command-line arguments passed when the program starts.

For example:

```text
java Main apple 100
```

Java passes approximately:

```java
new String[]{"apple", "100"}
```

Therefore:

```java
args.length == 2
args[0].equals("apple")
args[1].equals("100")
```

The name `args` is conventional and can be changed.

---

## 2. `args` and `System.in` Are Different

In the switch problem, input is read through:

```java
Scanner in = new Scanner(System.in);
```

The values entered in the online judge, such as:

```text
1 0 0
```

come from standard input, not from `args`.

The difference is:

| Input source | Example | Purpose |
|---|---|---|
| `String[] args` | `java Main 1 0 0` | Command-line arguments |
| `System.in` | Input box containing `1 0 0` | Standard input |

The program may not use `args`, but keeping the parameter is still normal.

---

## 3. Why Removing the Parameter Causes an Error

This method:

```java
public static void main() {
}
```

is usually a valid ordinary Java method, so it may compile.

However, in traditional Java versions, the launcher searches for:

```java
public static void main(String[] args)
```

A parameterless `main()` does not match that entry-point signature.

As a result, the program may compile successfully but fail when started with an error similar to:

```text
Error: Main method not found in class Main
```

The problem is therefore normally a runtime startup error rather than a syntax error.

---

## 4. Why Each Part Is Needed

```java
public static void main(String[] args)
```

- `public`: allows the launcher to access the method.
- `static`: allows Java to call it without first creating a `Main` object.
- `void`: the method does not return a value to the launcher.
- `main`: the recognized entry-method name.
- `String[]`: the traditional command-line argument type.

---

## 5. The Parameter Name Can Change

These declarations are equivalent entry methods:

```java
public static void main(String[] args)
```

```java
public static void main(String[] values)
```

```java
public static void main(String[] commandLineArguments)
```

This form is also equivalent:

```java
public static void main(String... args)
```

The launcher cares about the parameter type, not the parameter name.

However, this is invalid because the parameter has no name:

```java
public static void main(String[])
```

---

## 6. Application to the Switch Program

A compatible solution is:

```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);

        int count = 0;

        for (int i = 0; i < 3; i++) {
            int n = in.nextInt();

            if (n == 1) {
                count++;
            }
        }

        System.out.println(count % 2 == 0 ? "OFF" : "ON");
    }
}
```

In this program:

- `args` is unused.
- `Scanner` reads the three switch states from `System.in`.
- The program prints `ON` when the count of active switches is odd.
- The program prints `OFF` when the count is even.

---

## 7. Execution Process

```text
The online judge starts the JVM
        ↓
The JVM loads class Main
        ↓
The Java launcher looks for main(String[])
        ↓
The launcher calls the method
        ↓
Scanner reads values from System.in
        ↓
The program calculates the result
        ↓
The program prints ON or OFF
```

---

## 8. Java 25 Note

Java SE 25 supports additional forms of launchable `main` methods, including some parameterless forms.

However, many online judges still use earlier Java versions or expect the traditional entry method.

For competitive programming, the safest and most compatible declaration remains:

```java
public static void main(String[] args)
```

---

## Final Conclusion

`String[] args` represents command-line arguments and is part of the traditional Java program entry-point signature.

Even when the program does not use command-line arguments, removing the parameter changes the method signature. In most Java environments, the launcher can no longer recognize the method as the program entry point, so startup fails.
