# Nowcoder Weekly Round 151

## A. 运动会

题目链接：https://ac.nowcoder.com/acm/contest/137267/A

## Java 代码

```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);

        int n = in.nextInt();
        String s = in.next();

        System.out.println(judge(n, s.toCharArray()));
    }

    public static int judge(int n, char[] strChar) {
        int ans = 0;

        for (int i = 0; i < n; i++) {
            if (strChar[i] == '|') {
                int j = i + 1;

                while (j < n && strChar[j] == '=') {
                    j++;
                }

                if (j < n && j > i + 1 && strChar[j] == '|') {
                    ans++;
                    i = j;
                }
            }
        }

        return ans;
    }
}
```

## 复杂度分析

时间复杂度：`O(n)`。

每个字符最多被扫描常数次。

空间复杂度：`O(n)`。

代码中把字符串转成了字符数组。如果直接使用 `s.charAt(i)`，空间复杂度可以看作 `O(1)`。
