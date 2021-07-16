---
title: DP：计数DP
tags:
  - 算法
  - 动态规划
categories:
  - 算法
date: 2021/4/9
abbrlink: b34c4165
---

## [900. 整数划分](https://www.acwing.com/problem/content/description/902/)

一个正整数$n$可以表示成若干个正整数之和，形如$n=n_1+n_2+…+n_k$，其中$n_1≥n_2≥…≥n_k,k≥1$。我们将这样的一种表示称为正整数$n$的一种划分。

现在给定一个正整数$n$，请你求出$n$共有多少种不同的划分方法。

**输入格式**

共一行，包含一个整数$n$。

**输出格式**

共一行，包含一个整数，表示总划分数量。由于答案可能很大，输出结果请对$10^9+7$取模。

**数据范围**：$1≤n≤1000$

**输入样例:**
```c
5
```
**输出样例：**
```c
7
```
### 解法一
这个题我拿到的第一想法就是直接转换成完全背包：$n+1$个物品，体积就是物品下标$[1,n]$，物品数量不限，求凑成体积为$n$的方案数
```java
import java.util.*;
import java.io.*;

class Main {

    public static void main(String... args) throws Exception {
        PrintWriter out = new PrintWriter(new BufferedOutputStream(System.out));
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        // BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream("./input.txt")));
        int MOD = (int)1e9+7;
        int N = Integer.valueOf(br.readLine());
        //dp[i][j]   = dp[i-1][j] + dp[i-1][j-v] + ... + dp[i-1][j mod v]
        //dp[i][j-v] =              dp[i-1][j-v] + ... + dp[i-1][(j-v) mod v]
        //dp[i][j] = dp[i-1][j] + dp[i][j-v]
        long[] dp = new long[N+1];
        dp[0] = 1;
        for (int i = 1; i <= N; i++) {
            for (int j = i; j <= N; j++) {
                dp[j] = (dp[j] + dp[j-i]) % MOD;
            }
        }
        out.println(dp[N]);
        out.flush();
    }

    public static int[] read(BufferedReader br) throws Exception {
        return Arrays.stream(br.readLine().split(" ")).mapToInt(Integer::parseInt).toArray();
    }
}
```

### 解法二
不使用背包的思路，设置状态$dp[i][j]$为：将数字$i$划分为**恰好**$j$份的方案数，同时我们也可以将问题建模成：将$N$个苹果放到$N$个盘子，有多少种放法，不考虑顺序，每个盘子至少放一个

这里将$i$个苹果放到$j$个盘子，我们可以分两种情况来看
1. 至少有一个盘子只放了一个苹果，这种情况下，我们可以直接将这个只有一个苹果的盘子和苹果去掉再放，并不会影响结果，所以这样就等价于将$i-1$个苹果放到$j-1$个盘子中
2. 所有的盘子至少都有两个苹果，这种情况下，我们可以将所有盘子中的苹果数量减一再放，同样也不会影响结果，所以这种情况下等价于将$i-j$个苹果放到$j$个盘子中

然后将上面两种情况加起来就行了
- 初始状态：$dp[0][0] = 1$，0划分为0份，只有一种分法
- 转移方程：$dp[i][j] =dp[i-1][j-1] + dp[i-j][j],(i \geq j)$
- 出口：$\sum_{i=1}^{N}{dp[N][i]}$

代码实现如下：
```java
import java.util.*;
import java.io.*;

class Main {

    public static void main(String... args) throws Exception {
        PrintWriter out = new PrintWriter(new BufferedOutputStream(System.out));
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        // BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream("./input.txt")));
        int MOD = (int)1e9+7;
        int N = Integer.valueOf(br.readLine());
        // 将i划分为恰好j份的的方案数量
        long[][] dp = new long[N+1][N+1];
        // 1. 最小值是1，等价于dp[i-1][j-1]
        // 2. 最小值不是1，等价于dp[i-j][j]
        // Arrays.fill(dp[0], 1);
        dp[0][0] = 1;
        for (int i = 1; i <= N; i++) {
            for (int j = 1; j <= i; j++) {
                dp[i][j] = (dp[i-1][j-1] + dp[i-j][j]) % MOD;
            }
        }
        long res = 0;
        for (int i = 1; i <= N; i++) {
            res = (res + dp[N][i]) % MOD;
        }
        out.println(res);
        out.flush();
    }

    public static int[] read(BufferedReader br) throws Exception {
        return Arrays.stream(br.readLine().split(" ")).mapToInt(Integer::parseInt).toArray();
    }
}
```
其实也可以将状态方程改成将$i$划分为**不超过**$j$份的方案数量，但是不是特别好理解，还是定义为恰好比较好理解
<details>

```java
public static void main(String... args) throws Exception {
    PrintWriter out = new PrintWriter(new BufferedOutputStream(System.out));
    BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
    // BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream("./input.txt")));
    int MOD = (int)1e9+7;
    int N = Integer.valueOf(br.readLine());
    // 将i划分为不超过j份的的方案数量
    long[][] dp = new long[N+1][N+1];
    // 0划分为任意份都是1种分法
    // dp[i][j] += dp[i][j-1]
    Arrays.fill(dp[0], 1);
    for (int i = 1; i <= N; i++) {
        for (int j = 1; j <= N; j++) {
            //dp[i-1][j-1]被包含在dp[i][j-1]中
            //所以这里我们只需要加上dp[i][j-1]就行了
            dp[i][j] = dp[i][j-1];
            if (i >= j) {
                //dp[i-j][j]没有被包含，需要加上
                dp[i][j] = (dp[i][j] + dp[i-j][j]) % MOD;
            }
        }
    }
    out.println(dp[N][N]);
    out.flush();
}
```
</details>

## [1050. 鸣人的影分身](https://www.acwing.com/problem/content/description/1052/)

在火影忍者的世界里，令敌人捉摸不透是非常关键的。

我们的主角漩涡鸣人所拥有的一个招数——多重影分身之术——就是一个很好的例子。影分身是由鸣人身体的查克拉能量制造的，使用的查克拉越多，制造出的影分身越强。针对不同的作战情况，鸣人可以选择制造出各种强度的影分身，有的用来佯攻，有的用来发起致命一击。

那么问题来了，假设鸣人的查克拉能量为$M$，他影分身的个数最多为$N$，那么制造影分身时有多少种不同的分配方法？

**注意：**

1. 影分身可以分配$0$点能量。
2. 分配方案不考虑顺序，例如：$M=7,N=3$，那么$(2,2,3)$和$(2,3,2)$被视为同一种方案。

**输入格式**

第一行是测试数据的数目$t$。

以下每行均包含二个整数$M$和$N$，以空格分开。

**输出格式**

对输入的每组数据$M$和$N$，用一行输出分配的方法数。

**数据范围**
- $0≤t≤20$
- $1≤M,N≤10$

**输入样例：**
```c
1
7 3
```
**输出样例：**
```c
8
```
### 解法一
这题和上面一题很类似，不过多了一些限制，我们将其按前面的方式建模为：将$M$个苹果放到$N$个盘子，不考虑顺序，有多少种放法，允许有空盘。定义状态$dp[i][j]$为：将$i$个苹果放到**不超过**$j$个盘子中方案数
- 初始状态：$dp[0][0\sim N]=1$
- 转移方程：$dp[i][j]=dp[i][j-1] + dp[i-j][j]$，和上一题类似的转移方式，不多赘述，需要注意这里$j$可能会大于$i$
- 出口：$dp[M][N]$

代码实现如下：
```java
import java.util.*;
import java.io.*;

class Main {

    public static void main(String... args) throws Exception {
        PrintWriter out = new PrintWriter(new BufferedOutputStream(System.out));
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        // BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream("./input.txt")));
        int T = Integer.valueOf(br.readLine());
        while (T-- > 0) {
            int[] t = read(br);
            out.println(solve(t[0], t[1]));
        }
        out.flush();
    }

    public static int solve(int m, int n) {
        // 前i个苹果，放到j个盘子，有多少种放法
        int[][] dp = new int[m+1][n+1];
        Arrays.fill(dp[0], 1);
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                dp[i][j] = dp[i][j-1];
                if (i >= j) {
                    dp[i][j] += dp[i-j][j];
                }
            }
        }
        return dp[m][n];
    }

    public static int[] read(BufferedReader br) throws Exception {
        return Arrays.stream(br.readLine().split(" ")).mapToInt(Integer::parseInt).toArray();
    }
}
```
那么这题能不能将状态设置为将$i$个苹果放到**恰好**$j$个盘子中的方案数呢？答案是不行的，原因是这题是允许空盘子的！如果有空盘子，定义状态为**恰好**，那么$dp[i][j]$其实就已经包含了$dp[i][j-1]，dp[i][j-2]...$（相当于直接将空盘子去掉然后再将苹果放到剩余的盘子中），所以定义出来的状态只能是不超过$j$个盘子，最后返回$dp[M][N]$

### 解法二
这题同样也可以用背包来做，设置状态$dp[i][j][k]$为：前$i$个物品，凑成体积**恰好**为$j$，物品数量不限制，物品体积为物品下标$[0,n]$，使用物品总数**不超过**$k$的方案数
- 初始状态：$dp[0][0][k] = 0$
- 状态转移：枚举所有物品，枚举体积，然后枚举使用的物品总个数$k$，再枚举当前物品$i$使用的个数
  $$
  dp[i][j][k] = dp[i-1][j-i][k-1] + dp[i-1][j-2i][k-2] + ... + dp[i-1][j \mod i][k-j \div i]
  $$
- 出口：$dp[m][m][n]$，和上面一样，这里只能设置成不超过$k$，因为物品体积可以为$0$，所以使用$k$个物品会将使用$k-1$个物品的状态包含进去
- 时间复杂度：$O(m^2n^2)$

代码实现如下：
```java
//完全背包解法，m+1种物品，体积为[0, m]凑齐体积m，且总使用的物品数量不操作n，求方案数量
public static int solve(int m, int n) {
    //前i个数，体积恰好为j，总物品数量不超过k的方案数
    int[][][] dp = new int[m+1][m+1][n+1];
    for (int i = 0; i <= n; i++) {
        dp[0][0][i] = 1;
    }
    for (int i = 1; i <= m; i++) {
        for (int j = 0; j <= m; j++) {
            for (int k = 0; k <= n; k++) {
                for (int p = 0; p <= k && j-p*i >= 0; p++) {
                    dp[i][j][k] += dp[i-1][j-p*i][k-p];
                }
            }
        }
    }
    return dp[m][m][n];
}
```

