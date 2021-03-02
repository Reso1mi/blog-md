---
title: 
   DP：背包模型
tags: 
  [算法]
categories:
    [算法]
date: 2021/3/2
---
> 相关文章 [LeetCode背包问题](http://imlgw.top/2019/11/29/leetcode-bei-bao-wen-ti/) 

## [423. 采药](https://www.acwing.com/problem/content/425/)

辰辰是个天资聪颖的孩子，他的梦想是成为世界上最伟大的医师。为此，他想拜附近最有威望的医师为师。

医师为了判断他的资质，给他出了一个难题。

医师把他带到一个到处都是草药的山洞里对他说：“孩子，这个山洞里有一些不同的草药，采每一株都需要一些时间，每一株也有它自身的价值。我会给你一段时间，在这段时间里，你可以采到一些草药。如果你是一个聪明的孩子，你应该可以让采到的草药的总价值最大。”

如果你是辰辰，你能完成这个任务吗？

**输入格式**

输入文件的第一行有两个整数T和M，用一个空格隔开，T代表总共能够用来采药的时间，M代表山洞里的草药的数目。

接下来的M行每行包括两个在1到100之间（包括1和100）的整数，分别表示采摘某株草药的时间和这株草药的价值。

**输出格式**

输出文件包括一行，这一行只包含一个整数，表示在规定的时间内，可以采到的草药的最大总价值。

**数据范围**

1≤T≤1000, 1≤M≤100

**输入样例：**
```c
70 3
71 100
69 1
1 2
```
**输出样例：**
```c
3
```

### 解法一

没啥好说的，裸的01背包，考虑每个物品装或者不装
```java
//裸01背包
public static int solve(int T, int M, int[] v, int[] cost) {
    //考虑前i件药，背包大小为j，能装的最大收益
    int[][] dp = new int[M+1][T+1];
    //dp[i][j] = Math.max(dp[i-1][j], dp[i-1][j-w[i]] + v[i])
    for (int i = 1; i <= M; i++) {
        for (int j = T; j >= 0; j--) {
            dp[i][j] = Math.max(dp[i-1][j], j >= cost[i-1] ? dp[i-1][j-cost[i-1]] + v[i-1] : -1);
        }
    }
    return dp[M][T];
}

//空间优化
public static int solveOpt(int T, int M, int[] v, int[] cost) {
    //考虑前i件药，背包大小为j，能装的最大收益
    //dp[i][j] = Math.max(dp[i-1][j], dp[i-1][j-w[i]] + v[i])
    int[] dp = new int[T+1];
    for (int i = 0; i < M; i++) {
        for (int j = T; j >= cost[i]; j--) {
            //j > cost[i] dp[j] = dp[j](old)
            dp[j] = Math.max(dp[j], dp[j-cost[i]] + v[i]);
        }
    }
    return dp[T];
}
```
[1024. 装箱问题](https://www.acwing.com/problem/content/1026/)和[278. 数字组合](https://www.acwing.com/problem/content/280/)和这个差不多，不多写了

## [8. 二维费用的背包问题](https://www.acwing.com/problem/content/8/)
有 N 件物品和一个容量是 V 的背包，背包能承受的最大重量是 M。

每件物品只能用一次。体积是 vi，重量是 mi，价值是 wi。

求解将哪些物品装入背包，可使物品总体积不超过背包容量，总重量不超过背包可承受的最大重量，且价值总和最大。
输出最大价值。

**输入格式**

第一行两个整数，N，V,M，用空格隔开，分别表示物品件数、背包容积和背包可承受的最大重量。

接下来有 N 行，每行三个整数 vi,mi,wi，用空格隔开，分别表示第 i 件物品的体积、重量和价值。

**输出格式**

输出一个整数，表示最大价值。

**数据范围**：0<N≤1000，0<V,M≤100，0<vi,mi≤100，0<wi≤1000

**输入样例**
```c
4 5 6
1 2 3
2 4 4
3 4 5
4 5 6
```
**输出样例：**
```c
8
```

### 解法一
多维费用背包模板题
```java
import java.io.*;
import java.util.*;

class Main {
    public static void main(String[] args) throws Exception {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int[] in = read(br);
        int N = in[0], V = in[1], M = in[2];
        int[][] dp = new int[V+1][M+1];
        for (int i = 1; i <= N; i++) {
            int[] vmw = read(br);
            int vi = vmw[0], mi = vmw[1], wi = vmw[2];
            for (int j = V; j >= vi; j--) {
                for (int k = M; k >= mi; k--) {
                    dp[j][k] = Math.max(dp[j][k], dp[j-vi][k-mi]+wi);
                }
            }
        }
        System.out.println(dp[V][M]);
    }

    public static int[] read(BufferedReader br) throws Exception {
        return Arrays.stream(br.readLine().split(" ")).mapToInt(Integer::parseInt).toArray();
    }
}
```
## [1020. 潜水员](https://www.acwing.com/problem/content/1022/)
潜水员为了潜水要使用特殊的装备。

他有一个带2种气体的气缸：一个为氧气，一个为氮气。

让潜水员下潜的深度需要各种数量的氧和氮。

潜水员有一定数量的气缸。

每个气缸都有重量和气体容量。

潜水员为了完成他的工作需要特定数量的氧和氮。

他完成工作所需气缸的总重的最低限度的是多少？


**输入格式**

第一行有2个整数 m，n。它们表示氧，氮各自需要的量。

第二行为整数 k 表示气缸的个数。

此后的 k 行，每行包括ai，bi，ci，3个整数。这些各自是：第 i 个气缸里的氧和氮的容量及气缸重量。

**输出格式**

仅一行包含一个整数，为潜水员完成工作所需的气缸的重量总和的最低值。

**数据范围**：1≤m≤21, 1≤n≤79, 1≤k≤1000, 1≤ai≤21, 1≤bi≤79, 1≤ci≤800

**输入样例：**
```c
5 60
5
3 36 120
10 25 129
5 50 250
1 45 130
4 20 119
```
**输出样例：**
```c
249
```

### 解法一
一开始写了个记忆化递归，负数状态不保存，结果T了。

这题也是二维费用的背包问题，但是这里是求能覆盖费用的最小重量，求的是**至少**，上面的求的是 **最多**，方程其实很相似，但是细节还是不一样，首先初始状态不一样，这里求最小值，所以初始状态应该都赋值成+∞，`dp[i][j][k]`代表前i个氧气罐，氧气至少为j，氮气至少为k的时候，气缸最小的重量

核心的递推方程仍然是这个，但是状态的含义变了
```java
//省略一维，y,d,w代表氧气，氮气，质量
dp[j][k] = Math.min(dp[j][k], dp[j-y][k-d]+w);
```
这里二个维度的状态说的都是**至少**，那么意味着`j-y < 0` 或者 `k-d < 0` 也是合法的，而负数状态和`0状态(至少是0)`是等价的，所以我们可以从`0状态`合法的转移过来，使得状态计算完整不遗漏。一开始自己思考的时候想到了这种情况，但是考虑的不够完全

```java
import java.util.*;
import java.io.*;

public class AcWing1020_潜水员 {
    public static void main(String[] args) throws Exception {
        new Main().main();
    }
}

class Main {
    
    //dp[i][j] = Math.min(dp[i][j], dp[i-y][j-d]+w);
    public static void main(String... args) throws Exception {
        // BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream("./input.txt")));
        int INF = 0x3f3f3f3f;
        int[] in = read(br);
        int Y = in[0], D = in[1], N = Integer.parseInt(br.readLine());
        //dp[i][j]: 氧气至少i，氮气至少j，需要的最小重量
        int[][] dp = new int[Y+1][D+1];
        for (int i = 0; i <= Y; i++) {
            Arrays.fill(dp[i], INF);
        }
        dp[0][0] = 0;
        for (int i = 0; i < N; i++) {
            int[] ydw = read(br);
            int yi = ydw[0], di = ydw[1], wi = ydw[2];
            for (int j = Y; j >= 0; j--) {
                for (int k = D; k >= 0; k--) {
                    // if (j < yi || k < di) {
                    //     dp[j][k] = Math.min(dp[j][k], wi);
                    // } else {
                    //     dp[j][k] = Math.min(dp[j][k], dp[j-yi][k-di] + wi);
                    // }
                    // 上面的递推缺乏考虑，如果氧气和氮气只有一种超出了范围，另一维的状态不应该跟着按照0算
                    dp[j][k] = Math.min(dp[j][k], dp[Math.max(j-yi, 0)][Math.max(k-di, 0)] + wi);
                }
            }
        }
        System.out.println(dp[Y][D]);
    }

    public static int[] read(BufferedReader br) throws Exception {
        return Arrays.stream(br.readLine().split(" ")).mapToInt(Integer::parseInt).toArray();
    }
}
```

## [1022. 宠物小精灵之收服](https://www.acwing.com/problem/content/1024/)

宠物小精灵是一部讲述小智和他的搭档皮卡丘一起冒险的故事。

一天，小智和皮卡丘来到了小精灵狩猎场，里面有很多珍贵的野生宠物小精灵。

小智也想收服其中的一些小精灵。

然而，野生的小精灵并不那么容易被收服。

对于每一个野生小精灵而言，小智可能需要使用很多个精灵球才能收服它，而在收服过程中，野生小精灵也会对皮卡丘造成一定的伤害（从而减少皮卡丘的体力）。

当皮卡丘的体力小于等于0时，小智就必须结束狩猎（因为他需要给皮卡丘疗伤），而使得皮卡丘体力小于等于0的野生小精灵也不会被小智收服。

当小智的精灵球用完时，狩猎也宣告结束。

我们假设小智遇到野生小精灵时有两个选择：收服它，或者离开它。

如果小智选择了收服，那么一定会扔出能够收服该小精灵的精灵球，而皮卡丘也一定会受到相应的伤害；如果选择离开它，那么小智不会损失精灵球，皮卡丘也不会损失体力。

小智的目标有两个：主要目标是收服尽可能多的野生小精灵；如果可以收服的小精灵数量一样，小智希望皮卡丘受到的伤害越小（剩余体力越大），因为他们还要继续冒险。

现在已知小智的精灵球数量和皮卡丘的初始体力，已知每一个小精灵需要的用于收服的精灵球数目和它在被收服过程中会对皮卡丘造成的伤害数目。

请问，小智该如何选择收服哪些小精灵以达到他的目标呢？

输入格式
输入数据的第一行包含三个整数：N，M，K，分别代表小智的精灵球数量、皮卡丘初始的体力值、野生小精灵的数量。

之后的K行，每一行代表一个野生小精灵，包括两个整数：收服该小精灵需要的精灵球的数量，以及收服过程中对皮卡丘造成的伤害。

输出格式
输出为一行，包含两个整数：C，R，分别表示最多收服C个小精灵，以及收服C个小精灵时皮卡丘的剩余体力值最多为R。

**数据范围**

0<N≤1000, 0<M≤500, 0<K≤100

**输入样例1：**
```c
10 100 5
7 10
2 40
2 50
1 20
4 20
```
**输出样例1：**
```c
3 30
```
**输入样例2：**
```c
10 100 5
8 110
12 10
20 10
5 200
1 110
```
**输出样例2：**
```c
0 100
```

### 解法一
`dp[i][j][k]`： 考虑前i个宠物，消耗j个精灵球和k点生命值，能捕获的最多的精灵数量，这里直接将一维消掉，就是个多维的01背包，时间复杂度`O(N*M*K)`大约5*10^7，已经在T的边缘了
```java
//多维01背包 O(N*M*K) 5 x 10^7
public static int[] solve(int N, int M, int K, int[] costN, int[] costM) {
    //dp[i][j]: 消耗最多i个精灵球和最多j点生命值，能捕捉的最多精灵数量
    int[][] dp = new int[N+1][M+1];
    for (int i = 0; i < K; i++) {
        for (int j = N; j >= costN[i]; j--) {
            for (int k = M; k >= costM[i]; k--) {
                //注意这里N和M的cost都满足才能计算
                dp[j][k] = Math.max(dp[j][k], dp[j-costN[i]][k-costM[i]] + 1);
            }
        }
    }
    int maxCnt = dp[N][M], minCost = 0;
    //枚举出捕获maxCnt个精灵球，消耗的最小生命值
    for (int i = 0; i <= M; i++) {
        if (dp[N][i] == maxCnt) {
            minCost = i;
            break;
        }
    }
    //最后+1，把之前的加回来
    return new int[]{maxCnt, M+1-minCost};
}
```

### 解法二

背包维度的转换，选择范围较小的作为状态值，设置`dp[i][j][k]`为：考虑前i个宠物，捕获j个精灵消耗k个精灵球，消耗的最少的生命值，这样时间复杂度就变成了`O(K*K*N)` 大概10^7，也是很大的一步优化了，当然也可以进一步优化成`O(K*K*M)`不过稍微难处理点，就不多写了

```java
//交换维度，降低复杂度，O(K^2*N) = 10^7 还可以优化成 k^2*m
public static int[] solveOpt(int N, int M, int K, int[] costN, int[] costM) {
    //dp[i][j]: 捕捉i个精灵，消耗j个精灵球，消耗的最少的体力值
    int[][] dp = new int[K+1][N+1];
    for (int i = 1; i <= K; i++) {
        for (int j = 0; j <= N; j++) {
            dp[i][j] = 0x3f3f3f3f; 
        }
    }
    for (int i = 0; i < K; i++) {
        for (int j = K; j >= 1; j--) {
            for (int k = N; k >= costN[i]; k--) {
                if (dp[j-1][k-costN[i]] + costM[i] <= M) {
                    dp[j][k] = Math.min(dp[j][k], dp[j-1][k-costN[i]]+costM[i]);
                }
            }
        }
    }
    int maxCnt = 0;
    for (int i = K; i >= 0; i--) {
        if (dp[i][N] <= M) {
            maxCnt = i;
            break;
        }
    }
    //最后+1，把之前的加回来
    return new int[]{maxCnt, M+1-dp[maxCnt][N]};
}
```

## [HUD4501.小明系列故事——买年货（HUDOJ）](http://acm.hdu.edu.cn/showproblem.php?pid=4501)

**Problem Description**

春节将至，小明要去超市购置年货，于是小明去了自己经常去的都尚超市。

刚到超市，小明就发现超市门口聚集一堆人。用白云女士的话说就是：“那家伙，那场面，真是人山人海，锣鼓喧天，鞭炮齐呤，红旗招展。那可真是相当的壮观啊！”。好奇的小明走过去，奋力挤过人群，发现超市门口贴了一张通知，内容如下

值此新春佳节来临之际，为了回馈广大顾客的支持和厚爱，特举行春节大酬宾、优惠大放送活动。凡是都尚会员都可用会员积分兑换商品，凡是都尚会员都可**免费拿k件商品**，凡是购物顾客均有好礼相送。满100元送bla bla bla bla，满200元送bla bla bla bla bla...blablabla....

还没看完通知，小明就高兴的要死，因为他就是都尚的会员啊。迫不及待的小明在超市逛了一圈发现超市里有**n件他想要的商品**。小明顺便对这n件商品打了分，表示商品的实际价值。小明发现身上带了**v1的人民币**，会员卡里面有**v2的积分**。他想知道他最多能买多大价值的商品。

由于小明想要的商品实在太多了，他算了半天头都疼了也没算出来，所以请你这位聪明的程序员来帮帮他吧。
 

**Input**
```go
输入包含多组测试用例。
每组数据的第一行是四个整数n，v1，v2，k；
然后是n行，每行三个整数a，b，val，分别表示每个商品的价钱，兑换所需积分，实际价值。
[Technical Specification]
1 <= n <= 100
0 <= v1, v2 <= 100
0 <= k <= 5
0 <= a, b, val <= 100

Ps. 只要钱或者积分满足购买一件商品的要求，那么就可以买下这件商品。
```

**Output**
```go
对于每组数据，输出能买的最大价值。
详细信息见Sample。
```
**Sample Input**

```go
5 1 6 1
4 3 3
0 3 2
2 3 3
3 3 2
1 0 2
4 2 5 0
0 1 0
4 4 1
3 3 4
3 4 4
```

**Sample Output**

```go
12
4
```
Source
2013腾讯编程马拉松初赛第〇场（3月20日）

### 解法一
> 很久之前做过的题，拿出来对比下

三维费用的背包，但是和前面的题有点不一样，三个维度的费用是无关的，上面的小精灵，消耗的精灵球和生命值是相关的，所以两个维度的费用需要同时满足才能做合法的计算，而这里并不需要全部满足，而是分别计算
```java
import java.util.*;
import java.io.*;// petr的输入模板
import java.math.*; // 不是大数题可以不要这个

public class Solve_HDOJ_4501 {

    public static PrintWriter out = new PrintWriter(new OutputStreamWriter(System.out));

    public static void main(String[] args) throws Exception{
        InputReader in = new InputReader(System.in);
        //InputReader in = new InputReader(new FileInputStream("./input.txt"));
        while(!in.EOF()) {
            int n = in.nextInt();
            int v1 = in.nextInt();
            int v2 = in.nextInt();
            int k = in.nextInt();
            int[][] cost = new int[n][3];
            for (int i = 0; i < n; i++) {
                cost[i][0] = in.nextInt();
                cost[i][1] = in.nextInt();
                cost[i][2] = in.nextInt();
            }
            solve(n, v1, v2, k, cost);
        }
        //别忘了flush
        out.flush();
        out.close();
    }

    //因为数据量不大，就直接Scanner了
    public static void solve(int n, int v1, int v2, int k, int[][] cost) {
        int[][][] dp = new int[k+1][v1+1][v2+1];
        for (int i = 0; i < n; i++) {
            for (int j = k; j >= 0; j--) {
                for (int u = v1; u >= 0; u--) {
                    for (int w = v2; w >= 0; w--) {
                        //这里不能直接u>=cost[i][0] w >= cost[i][1]，因为积分和钱和免费拿是分开的，没有关联的
                        //即使我不能免费拿，但是我能用积分拿，即使不能用积分拿，我可以用钱买
                        //dp[j][u][w] = Math.max(dp[j][u][w], dp[j-1][u-cost[i][0]][w-cost[i][1]] + cost[i][2]);
                        int ans = 0;
                        if (j >= 1) { //免费拿
                            ans = Math.max(ans, dp[j-1][u][w] + cost[i][2]);
                        }
                        if (u >= cost[i][0]) { //钱
                            ans = Math.max(ans, dp[j][u-cost[i][0]][w] + cost[i][2]);
                        }
                        if (w >= cost[i][1]) { //积分
                            ans = Math.max(ans, dp[j][u][w-cost[i][1]] + cost[i][2]);
                        }
                        dp[j][u][w] = Math.max(ans, dp[j][u][w]);
                    }
                }
            }
        }
        out.println(dp[k][v1][v2]);
    }
}


class InputReader {

    public BufferedReader reader;
    
    public StringTokenizer tokenizer;

    public InputReader(InputStream stream) {
        //char[32768]
        reader = new BufferedReader(new InputStreamReader(stream), 32768);
        tokenizer = null;
    }

    //默认以" "作为分隔符，读一个
    public String next() {
        while (tokenizer == null || !tokenizer.hasMoreTokens()) {
            try {
                tokenizer = new StringTokenizer(reader.readLine());
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        }
        return tokenizer.nextToken();
    }

    //有的题目不给有多少组测试用例，只能一直读，读到结尾，需要自己判断结束
    //该函数也会读取一行，并初始化tokenizer，后序直接nextInt..等就可以读到该行
    public boolean EOF() {
        String str = null;
        try {
            str = reader.readLine();
            if (str == null) {
                return true;
            }
            //创建tokenizer
            tokenizer = new StringTokenizer(str);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        return false;
    }

    int nextInt(){
        return Integer.parseInt(next());
    }
    
    long nextLong(){
        return Long.parseLong(next());
    }
    
    double nextDouble(){
        return Double.parseDouble(next());
    }
    
    BigInteger nextBigInteger(){
        return new BigInteger(next());
    }

    BigDecimal nextBigDecimal(){
        return new BigDecimal(next());
    }
}
```

## [1023. 买书](https://www.acwing.com/problem/content/1025/)

小明手里有n元钱全部用来买书，书的价格为10元，20元，50元，100元。

问小明有多少种买书方案？（每种书可购买多本） 0 ≤ n ≤ 1000

### 解法一

完全背包，从头自己推了下递推公式，理解更深了一点
![](https://i.loli.net/2020/12/15/ljZW8eI4NkV6bCB.png)
```java
//完全背包
public static int solve(int N) {
    //dp[i][j]: 考虑前i本书，凑齐j价值的种类数
    int[][] dp = new int[5][N+1];
    //dp[i][j] = dp[i-1][j] + dp[i-1][j-c[i]] + dp[i-1][j-2*c[i]] + ... + dp[i-1][k*c[i]]
    //dp[i][j-c[i]] = dp[i-1][j-c[i]] + dp[i-1][j-2*c[i]] + ... + dp[i-1][k*c[i]]
    //dp[i][j] = dp[i-1][j] + dp[i][j-c[i]]
    dp[0][0] = 1;
    for (int i = 1; i <= 4; i++) {
        for (int j = 0; j <= N; j++) {
            dp[i][j] = dp[i-1][j];
            if (j >= price[i-1]) {
                dp[i][j] += dp[i][j-price[i-1]];
            }
        }
    }
    return dp[4][N];
}
```
降维
```java
//空间优化
public static int solveOpt(int N) {
    //dp[i][j]: 考虑前i本书，凑齐j价值的种类数
    int[] dp = new int[N+1];
    dp[0] = 1;
    for (int i = 1; i <= 4; i++) {
        for (int j = price[i-1]; j <= N; j++) {
            dp[j] += dp[j-price[i-1]];
        }
    }
    return dp[N];
}
```
[1021. 货币系统](https://www.acwing.com/problem/content/1023/)和这个一样，不多写

## [532. 货币系统（NOIP2018）](https://www.acwing.com/problem/content/534/)

在网友的国度中共有 n 种不同面额的货币，第 i 种货币的面额为 a[i]，你可以假设每一种货币都有无穷多张。

为了方便，我们把货币种数为 n、面额数组为 a[1..n] 的货币系统记作 (n,a)。 

在一个完善的货币系统中，每一个非负整数的金额 x 都应该可以被表示出，即对每一个非负整数 x，都存在 n 个非负整数 t[i] 满足 a[i]× t[i] 的和为 x。

然而，在网友的国度中，货币系统可能是不完善的，即可能存在金额 x 不能被该货币系统表示出。

例如在货币系统 n=3, a=[2,5,9] 中，金额 1,3 就无法被表示出来。 

两个货币系统 (n,a) 和 (m,b) 是等价的，当且仅当对于任意非负整数 x，它要么均可以被两个货币系统表出，要么不能被其中任何一个表出。 

现在网友们打算简化一下货币系统。

他们希望找到一个货币系统 (m,b)，满足 (m,b) 与原来的货币系统 (n,a) 等价，且 m 尽可能的小。

他们希望你来协助完成这个艰巨的任务：找到最小的 m。

**输入格式**

输入文件的第一行包含一个整数 T，表示数据的组数。

接下来按照如下格式分别给出T组数据。 

每组数据的第一行包含一个正整数 n。

接下来一行包含 n 个由空格隔开的正整数 a[i]。

**输出格式**

输出文件共有T行，对于每组数据，输出一行一个正整数，表示所有与 (n,a) 等价的货币系统 (m,b) 中，最小的 m。

**数据范围** : 1≤n≤100, 1≤a[i]≤25000, 1≤T≤20

**输入样例：**
```c
2 
4 
3 19 10 6 
5 
11 29 13 19 17 
```
**输出样例：**
```c
2
5
```

### 解法一
这里用了一个看起来很显然的结论：简化后的货币系统`(m,b)`，就是在原货币系统`(m,a)`中，剔除所有能被自分解的数得到的。判断集合中一个数能否被其他的数构成，其实问题就变成了完全背包求方案数，最后求得方案数为1的就是不能被分解的数，统计下结果就ok了
```java
public static int solve(int[] w) {
    int n = w.length;
    int max = 0;
    for (int i = 0; i < n; i++) {
        max = Math.max(max, w[i]);
    }
    int[] dp = new int[25001];
    dp[0] = 1;
    //完全背包求构成每个值的方案数
    for (int i = 0; i < n; i++) {
        for (int j = w[i]; j <= max; j++) {
            dp[j] += dp[j-w[i]];
        }
    }
    int res = 0;
    //方案数为1的就说明是不能丢掉的，统计一下就ok
    for (int i = 0; i < n; i++) {
        if (dp[w[i]] == 1) {
            res++;
        }
    }
    return res;
}
```
看起来很显然的结论的[证明](https://www.cnblogs.com/UntitledCpp/p/14083854.html#day1-t2-%E8%B4%A7%E5%B8%81%E7%B3%BB%E7%BB%9F)

## [6. 多重背包问题 III](https://www.acwing.com/problem/content/6/)
有 N 种物品和一个容量是 V 的背包。

第 i 种物品最多有 si 件，每件体积是 vi，价值是 wi。

求解将哪些物品装入背包，可使物品体积总和不超过背包容量，且价值总和最大。
输出最大价值。

**输入格式**

第一行两个整数，N，V (0<N≤1000, 0<V≤20000) ，用空格隔开，分别表示物品种数和背包容积。

接下来有 N 行，每行三个整数 vi,wi,si，用空格隔开，分别表示第 i 种物品的体积、价值和数量。

**输出格式**

输出一个整数，表示最大价值。

**数据范围** : 0<N≤1000, 0<V≤20000, 0<vi,wi,si≤20000

**输入样例**
```c
4 5
1 2 3
2 4 1
3 4 3
4 5 2
```

**输出样例：**
```c
10
```
### 解法一
暴力的做法，时间复杂度`O(NVS)`，4e11了，所以必然是过不了OJ的
```java
import java.util.*;
import java.io.*;
class Main {

    //dp[i][j]=Max(dp[i-1][j], dp[i-1][j-v]+w, dp[i-1][j-2*v]+2*w,... dp[i-1][j-s*v]+s*w)
    public static void main(String[] args) throws Exception {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int[] in = read(br);
        int N = in[0];
        int M = in[1];
        int[][] dp = new int[N+1][M+1];
        LinkedList<Integer> queue = new LinkedList<>();
        for (int i = 1; i <= N; i++) {
            int[] temp = read(br);
            int v = temp[0], w = temp[1], s = temp[2];
            for (int j = 0; j <= M; j++) {
                for (int k = 0; k <= s && k*v <= j; k++) {
                    dp[i][j] = Math.max(dp[i][j], dp[i-1][j-k*v]+k*w);
                }
            }
        }
        System.out.println(dp[N][M]);
    }

    private static int[] read(BufferedReader br) throws IOException {
        return Arrays.stream(br.readLine().split(" ")).mapToInt(Integer::parseInt).toArray();
    }
}
```
### 解法二
二进制优化的方法，明天再看😁
### 解法三

单调队列优化DP，有点难度，看题解都看了好久才理解。。。

首先我们很容易得到整体的递推公式: `dp[i][j]`代表前i个物品，背包体积为j时，能装的最大价值

$$dp[i][j]=\max(dp[i−1][j], dp[i−1][j−v]+w, dp[i−1][j−2∗v]+2∗w,… dp[i−1][j−s∗v]+s∗w)$$

其实和前面完全背包差不多，只是多了一个物品数量s的限制。我们可以发现`dp[i][j]`只和j-v, j-2v, j-3v这些状态有关，而这些状态除以v都是同余的，所以我们可以将这些状态按余数划分为不同的组，余数`r`的范围是`[0, v)`

转移方程变为
```java
// 0 <= r < v
dp[i][r]    =     dp[i-1][r]
dp[i][r+v]  = max(dp[i-1][r] +  w,  dp[i-1][r+v])
dp[i][r+2v] = max(dp[i-1][r] + 2w,  dp[i-1][r+v] +  w, dp[i-1][r+2v])
dp[i][r+3v] = max(dp[i-1][r] + 3w,  dp[i-1][r+v] + 2w, dp[i-1][r+2v] + w, dp[i-1][r+3v])
```
这个时候其实我们已经可以发现一些端倪了，后一项都是前一项加上一个值再加上一些偏移量再取Max的值，我们再将其变形一下
```java
// 0 <= r < v
dp[i][r]    =     dp[i-1][r]
dp[i][r+v]  = max(dp[i-1][r],  dp[i-1][r+v] - w) + w
dp[i][r+2v] = max(dp[i-1][r],  dp[i-1][r+v] - w, dp[i-1][r+2v] - 2w) + 2w
dp[i][r+3v] = max(dp[i-1][r],  dp[i-1][r+v] - w, dp[i-1][r+2v] - 2w, dp[i-1][r+3v] - 3w) + 3w
```
现在其实就很明显了，max内就是一个简单的滑窗，我们可以用单调队列去维护滑窗的最大值（[模板](http://imlgw.top/2019/07/20/leetcode-hua-dong-chuang-kou/#239-%E6%BB%91%E5%8A%A8%E7%AA%97%E5%8F%A3%E6%9C%80%E5%A4%A7%E5%80%BC)）

具体代码如下，时间复杂度`O(NM)`，还可以用滚动数组去优化空间，这里就不写了，不然代码变得更加难懂了
```java
import java.util.*;
import java.io.*;
class Main {

    //dp[i][j]   = Max(dp[i-1][j], dp[i-1][j-v]+w, dp[i-1][j-2*v]+2*w,... dp[i-1][j-s*v]+s*w)
    public static void main(String[] args) throws Exception {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int[] in = read(br);
        int N = in[0];
        int M = in[1];
        int[][] dp = new int[N+1][M+1];
        //单调队列求最大值
        LinkedList<int[]> queue = new LinkedList<>();
        for (int i = 1; i <= N; i++) {
            int[] temp = read(br);
            int v = temp[0], w = temp[1], s = temp[2];
            //枚举余数[0, v)
            for (int r = 0; r < v; r++) {
                queue.clear();
                //枚举同余所有状态dp[j] dp[j+v] dp[j+2v]....
                for (int k = 0; r+k*v <= M; k++) {
                    int val = dp[i-1][r+k*v] - k*w;
                    while (!queue.isEmpty() && queue.getLast()[0] < val) {
                        queue.removeLast();
                    }
                    //同余的数组元素数量可能超过同一物品的使用次数s
                    //区间内使用次数: ((r+k*v)-(r+q.first()[1]*v)) / v = k-q.first()[1]
                    if (!queue.isEmpty() && k - queue.getFirst()[1] > s) {
                        queue.removeFirst();
                    }
                    queue.addLast(new int[]{val, k});
                    dp[i][r+k*v] = queue.getFirst()[0] + k*w;
                }
            }
        }
        System.out.println(dp[N][M]);
    }

    private static int[] read(BufferedReader br) throws IOException {
        return Arrays.stream(br.readLine().split(" ")).mapToInt(Integer::parseInt).toArray();
    }
}
```
还是有点难度喔，这题虽然搞懂了，但是下次遇到类似的题还是不一定会😂
## [1019. 庆功会](https://www.acwing.com/problem/content/description/1021/)
为了庆贺班级在校运动会上取得全校第一名成绩，班主任决定开一场庆功会，为此拨款购买奖品犒劳运动员。

期望拨款金额能购买最大价值的奖品，可以补充他们的精力和体力。

**输入格式**

第一行二个数n，m，其中n代表希望购买的奖品的种数，m表示拨款金额。

接下来n行，每行3个数，v、w、s，分别表示第I种奖品的价格、价值（价格与价值是不同的概念）和能购买的最大数量（买0件到s件均可）。

**输出格式**

一行：一个数，表示此次购买能获得的最大的价值（注意！不是价格）。

**数据范围** ：n≤500, m≤6000, v≤100, w≤1000, s≤10

**输入样例：**
```c
5 1000
80 20 4
40 50 9
30 50 7
40 30 6
20 20 1
```
**输出样例：**
```c
1040
```
### 解法一

暴力解法，这题数据较小3e7可以过
```java
//每个物品有k个数量的限制后，问题就变成了01背包
//dp[i][j] = Max(dp[i-1][j], dp[i-1][j-v[i]]+w[i], dp[i-1][j-2*v[i]]+2*w[i], ... dp[i-1][j-s[i]*v[i]] + s[i]*w[i]) 
public static int solve (int m, int[] v, int[] w, int[] s) {
    int n = v.length;
    int[] dp = new int[m+1];
    for (int i = 0; i < n; i++) {
        //逆序避免覆盖
        for (int j = m; j >= v[i]; j--) {
            for (int k = s[i]; k >= 0; k--) {
                if (j-k*v[i] < 0) continue;
                dp[j] = Math.max(dp[j], dp[j-k*v[i]] + k*w[i]);
            }
        }
    }
    return dp[m];
}
```
单调队列优化和上面一摸一样，不写了

## [7. 混合背包问题](https://www.acwing.com/problem/content/7/)

有 N 种物品和一个容量是 V 的背包。

物品一共有三类：

第一类物品只能用1次（01背包）；
第二类物品可以用无限次（完全背包）；
第三类物品最多只能用 si 次（多重背包）；
每种体积是 vi，价值是 wi。

求解将哪些物品装入背包，可使物品体积总和不超过背包容量，且价值总和最大。
输出最大价值。

**输入格式**

第一行两个整数，N，V，用空格隔开，分别表示物品种数和背包容积。

接下来有 N 行，每行三个整数 vi,wi,si，用空格隔开，分别表示第 i 种物品的体积、价值和数量。

- si=−1 表示第 i 种物品只能用1次；
- si=0 表示第 i 种物品可以用无限次；
- si>0 表示第 i 种物品可以使用 si 次；

**输出格式**

输出一个整数，表示最大价值。

**数据范围**：0<N,V≤1000，0<vi,wi≤1000 ，−1≤si≤1000

**输入样例**
```c
4 5
1 2 -1
2 4 1
3 4 0
4 5 2
```
**输出样例：**
```czuo1
8
```

### 解法一

实际上就是把三种背包混到一起，我们对不同的物品做不同的处理就行了，每个物品的s并不会影响后面的物品存取
```java
import java.util.*;
import java.io.*;

class Main {
    public static void main(String[] args) throws Exception {
        BufferedReader bf = new BufferedReader(new InputStreamReader(System.in));
        int[] in = read(bf);
        int N = in[0];
        int V = in[1];
        int[] dp = new int[V+1];
        for (int i = 1; i <= N; i++) {
            int[] vws = read(bf);
            int v = vws[0], w = vws[1], s = vws[2];
            if (s == -1) s=1;
            if (s == 0) {
                for (int j = v; j <= V; j++) {
                    dp[j] = Math.max(dp[j], dp[j-v]+w);
                }
            } else {
                for (int j = V; j >= 0; j--) {
                    for (int k = 1; k <= s && j-k*v >= 0; k++) {
                        dp[j] = Math.max(dp[j], dp[j-k*v]+k*w);
                    }
                }
            }
        }
        System.out.println(dp[V]);
    }

    public static int[] read(BufferedReader br) throws Exception{
        return Arrays.stream(br.readLine().split(" ")).mapToInt(Integer::parseInt).toArray();
    }
}
```
多重背包的情况暂时还没有做优化，等后面看了二进制优化的方法窄来补一发，单调队列优化的不太想写

## [1013. 机器分配](https://www.acwing.com/problem/content/1015/)

总公司拥有M台 相同 的高效设备，准备分给下属的N个分公司。

各分公司若获得这些设备，可以为国家提供一定的盈利。盈利与分配的设备数量有关。

问：如何分配这M台设备才能使国家得到的盈利最大？

求出最大盈利值。

分配原则：每个公司有权获得任意数目的设备，但总台数不超过设备数M。

**输入格式**

第一行有两个数，第一个数是分公司数N，第二个数是设备台数M；

接下来是一个N*M的矩阵，矩阵中的第 i 行第 j 列的整数表示第 i 个公司分配 j 台机器时的盈利。

**输出格式**

第一行输出最大盈利值；

接下N行，每行有2个数，即分公司编号和该分公司获得设备台数。

答案不唯一，输入任意合法方案即可。

**数据范围**：1≤N≤10, 1≤M≤15

**输入样例：**
```c
3 3
30 40 50
20 30 50
20 25 30
```
**输出样例：**
```c
70
1 1
2 1
3 1
```
### 解法一
其实依然是一个01背包的问题。M件商品分配给N个公司，反过来就是从N个公司中选择几个公司，然后再选择每个公司分配多少个，直接推就行了，不过一开始写二维的方程时写出了bug，内循环中递推写成了$dp[i][j] = \max(dp[i-1][j], dp[i-1][j-k]+w[k-1])$。

这里最内层的循环是枚举的个数，应该和上一轮个数的结果比较，求一个最大的$dp[i][j]$而不是和$dp[i-1][j]$比较，所以应该是$dp[i][j] = \max(dp[i-1][j], \max(dp[i][j], dp[i-1][j-k]+w[k-1]))$，当然如果写一维的背包这里就没问题，$dp[j]$直接就是上一轮个数的结果

```java
//降维 & 简化后的代码
import java.util.*;
import java.io.*;

class Main {

    public static void main(String... args) throws Exception {
        // BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream("./input.txt")));
        int[] in = read(br);
        int N = in[0], M = in[1];
        int[] dp = new int[M+1];
        int[][] back = new int[N+1][M+1];
        for (int i = 1; i <= N; i++) {
            int[] w = read(br);
            for (int j = M; j >= 1; j--) {
                for (int k = 1; k <= j; k++) {
                    int temp = dp[j-k]+w[k-1];
                    if (temp > dp[j]) {
                        //记录当前子公司最优分配多少个，然后从后往前推
                        back[i][j] = k;
                        dp[j] = temp;
                    }
                }
            }
        }
        int[] res = new int[N];
        int idx = 0;
        int x = N, y = M;
        while(idx < N) {
            int k = back[x][y];
            res[idx++] = k;
            x--; y-=k;
        }
        System.out.println(dp[M]);
        for (int i = N-1; i >= 0; i--) {
            System.out.println((N-i)+ " " + res[i]);
        }
    }

    public static int[] read(BufferedReader br) throws Exception {
        return Arrays.stream(br.readLine().split(" ")).mapToInt(Integer::parseInt).toArray();
    }
}

public class AcWing1013_机器分配 {
    public static void main(String[] args) throws Exception {
        new Main().main();
    }
}
```
其实也可以不用back数组，事后我看yxc的做法就是直接根据dp值枚举每个厂分配的个数，倒推看前面是从哪个转移过来的，这样复杂度会高一点点，而且不能做降维操作，其实都一样，核心就是从后往前倒推状态转移的方向

## [426. 开心的金明](https://www.acwing.com/problem/content/428/)

金明今天很开心，家里购置的新房就要领钥匙了，新房里有一间他自己专用的很宽敞的房间。

更让他高兴的是，妈妈昨天对他说：“你的房间需要购买哪些物品，怎么布置，你说了算，只要不超过N元钱就行”。

今天一早金明就开始做预算，但是他想买的东西太多了，肯定会超过妈妈限定的N元。

于是，他把每件物品规定了一个重要度，分为5等：用整数1~5表示，第5等最重要。

他还从因特网上查到了每件物品的价格（都是整数元）。

他希望在不超过N元（可以等于N元）的前提下，使每件物品的价格与重要度的乘积的总和最大。 

设第j件物品的价格为v[j]，重要度为w[j]，共选中了k件物品，编号依次为j1，j2，…，jk，则所求的总和为： 

v[j1]∗w[j1]+v[j2]∗w[j2]+…+v[jk]∗w[jk]
请你帮助金明设计一个满足要求的购物单。

### 解法一

01背包
```java
import java.util.*;
import java.io.*;

class Main {

    public static void main(String... args) throws Exception {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        // BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream("./input.txt")));
        int[] in = read(br);
        int M = in[0], N = in[1];
        int[] dp = new int[M+1];
        for (int i = 1; i <= N; i++) {
            int[] vp = read(br);
            int v = vp[0], p = vp[1];
            for (int j = M; j >= v; j--) {
                dp[j] = Math.max(dp[j], dp[j-v]+v*p);
            }
        }
        System.out.println(dp[M]);
    }

    public static int[] read(BufferedReader br) throws Exception {
        return Arrays.stream(br.readLine().split(" ")).mapToInt(Integer::parseInt).toArray();
    }
}
```

## [11. 背包问题求方案数](https://www.acwing.com/problem/content/11/)

有 $N$ 件物品和一个容量是 $V$ 的背包。每件物品只能使用一次。

第 $i$ 件物品的体积是 $v_i$，价值是 $w_i$。

求解将哪些物品装入背包，可使这些物品的总体积不超过背包容量，且总价值最大。

输出 最优选法的方案数。注意答案可能很大，请输出答案模 $10^9+7$ 的结果。

**输入格式**

第一行两个整数，$N$，$V$，用空格隔开，分别表示物品数量和背包容积。

接下来有 $N$ 行，每行两个整数 $v_i$, $w_i$，用空格隔开，分别表示第 $i$ 件物品的体积和价值。

**输出格式**

输出一个整数，表示 方案数 模 $10^9+7$ 的结果。

**数据范围**

- $0<N,V≤1000$
- $0<v_i,w_i≤1000$

**输入样例**
```c
4 5
1 2
2 4
3 4
4 6
```

**输出样例：**
```c
2
```

### 解法一
背包问题搞起来还是有点麻烦喔，递推方程都是众所周知的，但是递推的细节有点不好想。

下面的解法是设置的状态是: $dp[i][j]$，考虑前$i$个物品体积 **恰好** 是$j$的时候最大价值，同时还需要一个$cnt[i][j]$，考虑前$i$个物品体积恰好是$j$的时候最大价值的方案数量，$cnt$根据$dp$的值进行转移
1. 当$dp[i-1][j-v_i]+w_i > dp[i-1][j]$的时候，说明$cnt[i][j]$当前的价值不是最大的，所以会被$cnt[i-1][j-v_i]$替代
2. 当$dp[i-1][j-v_i]+w_i = dp[i-1][j]$的时候，说明$cnt[i][j]$和$cnt[i-1][j-v_i]$的最大价值相同，可以累加起来

初始状态：$cnt[i][0] = 1$，$dp[i][0] = 0\  other\ \inf$（体积恰好为0只有一种方案，什么都不装）

最后我们需要求出最大的价值`maxV`，注意这里最大价值不一定是$dp[N][V]$，因为我们这里求的是体积 **恰好** $j$的最大价值。然后再根据最大价值的$dp[i][j]$状态求对应$cnt[i][j]$的方案数之和就行了
```java
import java.util.*;
import java.io.*;

class Main {

    public static void main(String... args) throws Exception {
        PrintWriter out = new PrintWriter(new BufferedOutputStream(System.out));
        // BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream("./input.txt")));
        int[] in = read(br);
        int N = in[0], V = in[1];
        int INF = 0x3f3f3f3f;
        int MOD = (int)1e9+7;
        // dp[j]: 体积刚好为j时，最大的价值
        int[] dp = new int[V+1];
        // cnt[j]: 体积刚好为j时，最大价值方案数
        int[] cnt = new int[V+1];
        Arrays.fill(dp, -INF);
        dp[0] = 0; cnt[0] = 1;
        for (int i = 1; i <= N; i++) {
            int[] t = read(br);
            int vi = t[0], wi = t[1];
            for (int j = V; j >= vi; j--) {
                int val = dp[j-vi] + wi;
                if (val > dp[j]) { // 从dp[i-1][j-vi]推过来，val比当前的大，所以cnt[j]被取代
                    cnt[j] = cnt[j-vi];
                    dp[j] = val;
                } else if (val == dp[j]) { // 从dp[i-1][j-vi]推过来，val和当前相同，叠加起来
                    cnt[j] = (cnt[j] + cnt[j-vi]) % MOD;
                }
            }
        }
        int maxW = 0;
        for (int i = 0; i <= V; i++) maxW = Math.max(maxW, dp[i]);
        long res = 0;
        for (int i = 0; i <= V; i++) {
            if (dp[i] == maxW) {
                res = (res + cnt[i]) % MOD;
            }
        }
        out.println(res);
        out.flush();
        out.close();
    }

    public static int[] read(BufferedReader br) throws Exception {
        return Arrays.stream(br.readLine().split(" ")).mapToInt(Integer::parseInt).toArray();
    }
}
```

### 解法二

其实和上面的解法大同小异，但是状态的定义不一样，初始值不一样，但是转移方程是一样的，这里状态定义为：$dp[i][j]$，考虑前$i$个物品体积 **最多** 是$j$的时候最大价值，$cnt[i][j]$同理

初始状态：$cnt[0][j] = 1$，注意和上面的区别，这里实际上没有物品可以装，所以最大价值就是0，那么对于任意体积$j$，我们只能选择不装，方案数量为1
```java
class Main {

    public static void main(String... args) throws Exception {
        PrintWriter out = new PrintWriter(new BufferedOutputStream(System.out));
        // BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream("./input.txt")));
        int[] in = read(br);
        int N = in[0], V = in[1];
        int MOD = (int)1e9+7;
        //dp[j]: 体积小于等于j时，最大的价值
        int[] dp = new int[V+1];
        //cnt[j]: 体积小于等于j时，最大价值方案数
        int[] cnt = new int[V+1];
        //这里其实初始化的是dp[0][i]=1，此时最大价值就是0，什么都不装
        Arrays.fill(cnt, 1);
        for (int i = 0; i < N; i++) {
            int[] t = read(br);
            int vi = t[0], wi = t[1];
            for (int j = V; j >= vi; j--) {
                int val = dp[j-vi] + wi;
                if (val > dp[j]) {
                    dp[j] = val;
                    cnt[j] = cnt[j-vi];
                } else if (val == dp[j]) {
                    cnt[j] = (cnt[j] + cnt[j-vi]) % MOD;
                }
            }
        }
        out.println(cnt[V]);
        out.flush();
        out.close();
    }

    public static int[] read(BufferedReader br) throws Exception {
        return Arrays.stream(br.readLine().split(" ")).mapToInt(Integer::parseInt).toArray();
    }
}
```

> 初始状态的定义初始化还是要好好琢磨下

## [10. 有依赖的背包问题](https://www.acwing.com/problem/content/description/10/)
有 N 个物品和一个容量是 V 的背包。

物品之间具有依赖关系，且依赖关系组成一棵树的形状。如果选择一个物品，则必须选择它的父节点。

如下图所示：

![](https://i.loli.net/2021/02/22/aEhD4KInUcObAMk.png)

如果选择物品5，则必须选择物品1和2。这是因为2是5的父节点，1是2的父节点。

每件物品的编号是 i，体积是 vi，价值是 wi，依赖的父节点编号是 pi。物品的下标范围是 1…N。

求解将哪些物品装入背包，可使物品总体积不超过背包容量，且总价值最大。

输出最大价值。

**输入格式**

第一行有两个整数 N，V，用空格隔开，分别表示物品个数和背包容量。

接下来有 N 行数据，每行数据表示一个物品。
第 i 行有三个整数 vi,wi,pi，用空格隔开，分别表示物品的体积、价值和依赖的物品编号。
如果 pi=−1，表示根节点。 数据保证所有物品构成一棵树。

**输出格式**

输出一个整数，表示最大价值。

**数据范围**：1≤N,V≤100，1≤vi,wi≤100

**父节点编号范围：**
- 内部结点：1≤pi≤N;
- 根节点 pi=−1;

**输入样例**
```c
5 7
2 3 -1
2 2 1
3 5 1
4 7 2
3 6 2
```
**输出样例**
```c
11
```
### 解法一
参考大佬的[解法](https://www.acwing.com/solution/content/5780/)，多叉树转二叉树 + 记忆化递归的做法，感觉相比yxc的分组背包更好理解，也是能想到的范围。除此之外大佬还写了一种dfs序的方法，但是dfs序我还不太了解，先埋个坑，等以后遇到再来填一下
```java
import java.util.*;
import java.io.*;

/*
多叉树的存储（左孩子，右兄弟）+ 记忆化递归
 */
class Main {

    static int N, M;
    static int[][] dp;
    //二叉树节点
    static int[] left, right;
    static int[] son;
    static int[] v, w;

    public static void main(String... args) throws Exception {
        PrintWriter out = new PrintWriter(new BufferedOutputStream(System.out));
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        // BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream("./input.txt")));
        int[] in = read(br);
        N = in[0]; M = in[1];
        left = new int[N+1]; right = new int[N+1];
        v = new int[N+1]; w = new int[N+1];
        son = new int[N+1];
        dp = new int[N+1][M+1];
        for (int i = 0; i <= N; i++) {
            Arrays.fill(dp[i], -1);
        }
        int root = 0;
        for (int i = 1; i <= N; i++) {
            int[] t = read(br);
            v[i] = t[0]; w[i] = t[1];
            if (t[2] == -1) {
                root = i;
            } else {
                if (son[t[2]] == 0) {
                    left[t[2]] = i;
                } else {
                    right[son[t[2]]] = i;
                }
                son[t[2]] = i;
            }
        }
        out.println(dfs(root, M));
        out.flush();
    }

    //在i子树下，体积不超过j的情况下，能得到的最大收益
    //子树的选取不是孤立的，是可以一起选的，所以应该按照体积分配划分子树
    public static int dfs(int i, int j) {
        // i=0子树不存在
        if (i==0 || j < 0) return 0;
        if (dp[i][j] != -1) {
            return dp[i][j];
        }
        //不选当前节点，体积j全部分配给兄弟节点
        dp[i][j] = dfs(right[i], j);
        //当前节点必选，分给子节点k，兄弟节点j-v[i]-k
        for (int k = 0; k <= j-v[i]; k++) {
            dp[i][j] = Math.max(dp[i][j], dfs(left[i], k) + dfs(right[i], j-v[i]-k) + w[i]);
        }
        return dp[i][j];
    }

    public static int[] read(BufferedReader br) throws Exception {
        return Arrays.stream(br.readLine().split(" ")).mapToInt(Integer::parseInt).toArray();
    }
}
```
### 解法二
这个是ycx讲的方法，分组背包的解法，我反正是理解不能...但是这里确实AC了...不是我等凡人能想到的做法
```java
import java.util.*;
import java.io.*;

class Main {

    static int N, V;
    static int[] vs;
    static int[] ws;
    static int idx;
    static int[] h, ne, e;
    static int[][] dp;
    //添加边a->b
    public static void add(int a, int b) {
        e[idx] = b; ne[idx] = h[a]; h[a] = idx++;
    }

    public static void main(String... args) throws Exception {
        PrintWriter out = new PrintWriter(new BufferedOutputStream(System.out));
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        // BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream("./input.txt")));
        int[] in = read(br);
        N = in[0]; V = in[1];
        vs = new int[N+1]; ws = new int[N+1];
        h = new int[N+1]; ne = new int[N+1]; e = new int[N+1];
        Arrays.fill(h, -1);
        dp = new int[N+1][V+1];
        int root = 0;
        for (int i = 1; i <= N; i++) {
            int[] t = read(br);
            vs[i] = t[0]; ws[i] = t[1];
            if (t[2] == -1) {
                root = i;
            } else {
                add(t[2], i);
            }
        }
        dfs(root);
        out.println(dp[root][V]);
        out.flush();
    }

    //dp[i][j]: 在i子树下，体积不操作j的时候最大的价值
    public static void dfs(int u) {
        for (int i = h[u]; i != -1; i = ne[i]) {
            dfs(e[i]);
            //先将根节点的体积空出来，根节点是必选的
            for (int j = V-vs[u]; j >= 0; j--) {
                for (int k = 0; k <= j; k++) {
                    //这里为什么可以直接用根节点+子节点？难道不会重复吗？
                    dp[u][j] = Math.max(dp[u][j], dp[u][j-k]+dp[e[i]][k]);
                }
            }
        }
        for (int j = V; j >= 0; j--) {
            if (j < vs[u]) {
                dp[u][j] = 0;
            } else {
                dp[u][j] = dp[u][j-vs[u]] + ws[u];
            }
        }
    }

    public static int[] read(BufferedReader br) throws Exception {
        return Arrays.stream(br.readLine().split(" ")).mapToInt(Integer::parseInt).toArray();
    }
}
```

## [12. 背包问题求具体方案](https://www.acwing.com/problem/content/12/)

有$N$件物品和一个容量是$V$的背包。每件物品只能使用一次。

第 $i$ 件物品的体积是 $v_i$，价值是 $w_i$。

求解将哪些物品装入背包，可使这些物品的总体积不超过背包容量，且总价值最大。

输出 字典序最小的方案。这里的字典序是指：所选物品的编号所构成的序列。物品的编号范围是 $1…N$。

输入格式
第一行两个整数，$N$，$V$，用空格隔开，分别表示物品数量和背包容积。

接下来有 $N$ 行，每行两个整数 $v_i$, $w_i$，用空格隔开，分别表示第 $i$ 件物品的体积和价值。

**输出格式**

输出一行，包含若干个用空格隔开的整数，表示最优解中所选物品的编号序列，且该编号序列的字典序最小。

物品编号范围是 1…N。

**数据范围**： $0<N,V≤1000$，$0<v_i,w_i≤1000$

**输入样例**
```c
4 5
1 2
2 4
3 4
4 6
```
**输出样例：**
```c
1 4
```
### 解法一
一开始写了使用back数组的方案，但是发现并不好求字典序最小的，back数组是通过**倒推**推出来的，所以得到的一定不是字典序最小的。所以这里我们需要正向的推，改变状态方程定义，设置递推状态为：$dp[i][j]$，从第$i$个物品到最后一个物品（$i$从0开始，包括$i$），体积不超过$j$的最大收益
- 入口：$dp[N][0 \backsim M] = 0$，不选取任何物品
- 转移：$dp[i][j] = \min(dp[i+1][j], dp[i+1][j-v_i]+w_i)$
- 出口：$dp[0][M]$就是最大收益

在状态填充完毕后，我们就可以正向的推出一个字典序最小的方案：顺序遍历物品编号，当$dp[i][M] = dp[i+1][M-v_i] + w_i$，说明选取该物品可以获得最大收益，我们要求的是字典序最小的，所以我们应该**优先选取**靠前的物品，而我们是正向遍历的，所以直接选取该物品输出，然后减去对应的体积，继续循环该过程，最终结果就一定是字典序最小的方案
```java
import java.util.*;
import java.io.*;

class Main {

    public static void main(String... args) throws Exception {
        PrintWriter out = new PrintWriter(new BufferedOutputStream(System.out));
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        // BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream("./input.txt")));
        int[] in = read(br);
        int N = in[0], M = in[1];
        int[] v = new int[N];
        int[] w = new int[N];
        for (int i = 0; i < N; i++) {
            int[] t = read(br);
            v[i] = t[0]; w[i] = t[1];
        }
        //第i个物品到最后（包括i），体积不超过j的情况下最大收益（状态不同于之前的状态）
        //dp[i][j] = min(dp[i+1][j], dp[i+1][j-v]+w)
        int[][] dp = new int[N+1][M+1];
        for (int i = N-1; i >= 0; i--) {
            for (int j = M; j >= 0; j--) {
                dp[i][j] = dp[i+1][j];
                if (j >= v[i]) {
                    dp[i][j] = Math.max(dp[i+1][j-v[i]] + w[i], dp[i+1][j]);;
                }
            }
        }
        for (int i = 0; i <= N-1; i++) {
            if (M-v[i] < 0) continue;
            if (dp[i][M] == dp[i+1][M-v[i]] + w[i]) {
                out.print((i+1)+" ");
                M -= v[i];
            }
        }
        out.flush();
    }

    public static int[] read(BufferedReader br) throws Exception {
        return Arrays.stream(br.readLine().split(" ")).mapToInt(Integer::parseInt).toArray();
    }
}
```

## [734. 能量石](https://www.acwing.com/problem/content/736/)
岩石怪物杜达生活在魔法森林中，他在午餐时收集了N块能量石准备开吃。

由于他的嘴很小，所以一次只能吃一块能量石。

能量石很硬，吃完需要花不少时间。

吃完第 $i$ 块能量石需要花费的时间为$S_i$秒。

杜达靠吃能量石来获取能量。 

不同的能量石包含的能量可能不同。

此外，能量石会随着时间流逝逐渐失去能量。

第 $i$ 块能量石最初包含$E_i$单位的能量，并且每秒将失去$L_i$单位的能量。

当杜达开始吃一块能量石时，他就会立即获得该能量石所含的全部能量（无论实际吃完该石头需要多少时间）。

能量石中包含的能量最多降低至0。

请问杜达通过吃能量石可以获得的最大能量是多少？

**输入格式**

第一行包含整数T，表示共有T组测试数据。

每组数据第一行包含整数N，表示能量石的数量。

接下来$N$行，每行包含三个整数$S_i,E_i,L_i$。

**输出格式**

每组数据输出一个结果，每个结果占一行。

结果表示为“Case #x: y”，其中x是组别编号（从1开始），y是可以获得的最大能量值。

**数据范围**
- $1≤T≤10$
- $1≤N≤100$
- $1≤Si≤100$
- $1≤Ei≤105$
- $0≤Li≤105$
  
**输入样例：**
```c
3
4
20 10 1
5 30 5
100 30 1
5 80 60
3
10 4 1000
10 3 1000
10 8 1000
2
12 300 50
5 200 0
```
**输出样例：**
```c
Case #1: 105
Case #2: 8
Case #3: 500
```
**样例解释**

在样例＃1中，有N = 4个宝石。杜达可以选择的一个吃石头顺序是：

吃第四块石头。这需要5秒，并给他80单位的能量。
吃第二块石头。这需要5秒，并给他5单位的能量（第二块石头开始时具有30单位能量，5秒后失去了25单位的能量）。
吃第三块石头。这需要100秒，并给他20单位的能量（第三块石头开始时具有30单位能量，10秒后失去了10单位的能量）。
吃第一块石头。这需要20秒，并给他0单位的能量（第一块石头以10单位能量开始，110秒后已经失去了所有的能量）。
他一共获得了105单位的能量，这是能获得的最大值，所以答案是105。

在样本案例＃2中，有N = 3个宝石。

无论杜达选择吃哪块石头，剩下的两个石头的能量都会耗光。

所以他应该吃第三块石头，给他提供8单位的能量。

在样本案例＃3中，有N = 2个宝石。杜达可以：

吃第一块石头。这需要12秒，并给他300单位的能量。
吃第二块石头。这需要5秒，并给他200单位的能量（第二块石头随着时间的推移不会失去任何能量！）。
所以答案是500。

### 解法一
贪心+背包的解法。

传统的01背包，我们一般都是直接遍历给定的原始物品顺序，然后进行递推，这里实际上就包含了拿取的顺序，物品靠前的会被优先拿取，只不过传统01背包拿取顺序对结果没有影响，所以直接顺序递推没有问题。

而这题中吃石头的顺序会对结果产生影响，所以需要保证通过我们设定的排列顺序进行拿取最终一定能拿到最优解。

假设最优的吃石头的顺序是$a_0, a_1, a_2,...,a_i,a_j,...,a_k$。$a_i$代表石头在原始序列中的位置，$k$为最终吃的石头个数。那么我们交换任意的**相邻**两个石头$a_i$和$a_j$对其他的石头是没有任何影响的，并且这种交换最终获得的能量一定是**小于等于**当前的最优解获得的能量的，即（$t$为之前消耗的时间）
$$
E_i-(t*l_i)+E_j-(t+S_i)*l_j \geq E_j-(t*l_j)+E_i-(t+S_j)*l_i
$$
化简后为：$S_i*l_j \leq S_j*l_i$，所以我们只要按照这一条件排列石头，然后进行递推就能保证获得最大收益

排列好之后我们以时间$S$作为背包的体积
- 状态：$dp[i][j]$为前$i$个石头，消耗时间**刚好**为$j$时最大的收益
- 转移方程：$dp[i][j] = \max(dp[i-1][j], dp[i-1][j-s_i]+\max(0, E_i-(j-s_i)*l_i))$
- 出口：$\max(dp[i][j])$
  
注意这里必须要设置为时间**恰好为**$j$的最大收益，如果设置成**不超过**，这里$E_i-(j-s_i)*l_i$的计算就有问题，$j-s_i$就不是准确的消耗时间，导致最终结果不对
```java
import java.util.*;
import java.io.*;

class Main {

    public static void main(String... args) throws Exception {
        PrintWriter out = new PrintWriter(new BufferedOutputStream(System.out));
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        // BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream("./input.txt")));
        int T = read(br)[0];
        int idx = 0;
        while (T-- > 0) {
            idx++;
            int N = read(br)[0];
            int[] s = new int[N], e = new int[N], l = new int[N];
            for (int i = 0; i < N; i++) {
                int[] t = read(br);
                s[i] = t[0]; e[i] = t[1]; l[i] = t[2];
            }
            out.printf("Case #%d: %d\n", idx, solve(N, s, e, l));
        }
        out.flush();
    }

    //按照Si/Li排序
    public static int solve(int N, int[] s, int[] e, int[] lo) {
        int MAX = 10005;
        Integer[] id = new Integer[N];
        for (int i = 0; i < N; i++) id[i]=i;
        Arrays.sort(id, (i1, i2)->s[i1]*lo[i2]-s[i2]*lo[i1]);
        int[] dp = new int[MAX];
        int res = 0;
        for (int k = 0; k < N; k++) {
            int i = id[k];
            for (int j = MAX-1; j >= s[i]; j--) {
                dp[j] = Math.max(dp[j], dp[j-s[i]] + Math.max(0, e[i]-(j-s[i])*lo[i]));
                res = Math.max(res, dp[j]);
            }
        }
        return res;
    }

    public static int[] read(BufferedReader br) throws Exception {
        return Arrays.stream(br.readLine().split(" ")).mapToInt(Integer::parseInt).toArray();
    }
}
```

## [487. 金明的预算方案](https://www.acwing.com/problem/content/description/489/)

金明今天很开心，家里购置的新房就要领钥匙了，新房里有一间金明自己专用的很宽敞的房间。

更让他高兴的是，妈妈昨天对他说：“你的房间需要购买哪些物品，怎么布置，你说了算，只要不超过N元钱就行”。

今天一早，金明就开始做预算了，他把想买的物品分为两类：主件与附件，附件是从属于某个主件的，下表就是一些主件与附件的例子：

![](https://i.loli.net/2021/03/02/HGepaVgL4Emdfhr.png)

如果要买归类为附件的物品，必须先买该附件所属的主件。

每个主件可以有0个、1个或2个附件。

附件不再有从属于自己的附件。

金明想买的东西很多，肯定会超过妈妈限定的N元。

于是，他把每件物品规定了一个重要度，分为5等：用整数1~5表示，第5等最重要。

他还从因特网上查到了每件物品的价格（都是10元的整数倍）。

他希望在不超过N元（可以等于N元）的前提下，使每件物品的价格与重要度的乘积的总和最大。

设第j件物品的价格为$v[j]$，重要度为$w[j]$，共选中了$k$件物品，编号依次为$j_1$，$j_2$，…，$j_k$，则所求的总和为：

$v[j_1]∗w[j_1]+v[j_2]∗w[j_2]+…+v[j_k]∗w[j_k]$（其中*为乘号）

请你帮助金明设计一个满足要求的购物单。

**输入格式**

输入文件的第1行，为两个正整数，用一个空格隔开：N m，其中N表示总钱数，m为希望购买物品的个数。

从第2行到第m+1行，第j行给出了编号为j-1的物品的基本数据，每行有3个非负整数v p q，其中v表示该物品的价格，p表示该物品的重要度（1~5），q表示该物品是主件还是附件。

如果q=0，表示该物品为主件，如果q>0，表示该物品为附件，q是所属主件的编号。

**输出格式**

输出文件只有一个正整数，为不超过总钱数的物品的价格与重要度乘积的总和的最大值（<200000）。

**数据范围**： N<32000,m<60,v<10000

**输入样例：**
```c
1000 5
800 2 0
400 5 1
300 5 1
400 3 0
500 2 0
```
**输出样例：**
```c
2200
```

### 解法一
这题开始磨了好久，一开始没看到题目说的附件最多两个，后面看题解看见附件只有最多两个的时候马上滚回去写了下面的解法

因为附件最多两个，所以我们完全可以直接枚举所有的**主件+附件**的情况，然后就是一个简单的分组背包就完事了。下面的解法是二维的，一维的有时候会隐藏一些信息，让人忽略一些细节，这里就不放一维的写法了，反正也就改几行代码的事情
```java
import java.util.*;
import java.io.*;

class Main {

    static int[] v, w, p;

    public static void main(String... args) throws Exception {
        PrintWriter out = new PrintWriter(new BufferedOutputStream(System.out));
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        // BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream("./input.txt")));
        int[] in = read(br);
        int N = in[0], M = in[1];
        v = new int[M+1]; w = new int[M+1]; p = new int[M+1];
        //依赖关系
        List<Integer>[] adj = new ArrayList[M+1];
        for (int i = 1; i <= M; i++) {
            int[] t = read(br);
            v[i] = t[0]; w[i] = t[1]; p[i] = t[2];
            if (adj[p[i]] == null) {
                adj[p[i]] = new ArrayList<>();
            }
            adj[p[i]].add(i);
        }
        int Z = adj[0].size();
        int[][] dp = new int[Z+1][N+1];
        //将依赖关系进行分组枚举所有情况，题目中有一条关键信息就是附件最多只有两件
        //adj[0]就是所有的主件
        for (int i = 1; i <= Z; i++) {
            int pi = adj[0].get(i-1); //pi不是必选，但是要选pi的附件，pi就必选了
            for (int j = N; j >= 0; j--) {
                dp[i][j] = dp[i-1][j]; //不选当前主件
                int s = adj[pi] == null ? 0 : adj[pi].size(); //附件个数,0,1,2
                //枚举当前组内的所有情况
                for (int k = 0; k < (1<<s); k++) {
                    //当前组某项的体积和价值和
                    int t = v[pi]*w[pi];
                    int sv = v[pi];
                    for (int c = 0; c < s; c++) {
                        if (((k>>>c)&1)==1) {
                            int d = adj[pi].get(c);
                            t += v[d]*w[d];
                            sv += v[d];
                        }
                    }
                    if (j >= sv) {
                        //注意这里应该和dp[i][j]比较
                        dp[i][j] = Math.max(dp[i][j], dp[i-1][j-sv] + t);
                    }
                }
            }
        }
        out.println(dp[Z][N]);
        out.flush();
    }

    public static int[] read(BufferedReader br) throws Exception {
        return Arrays.stream(br.readLine().split(" ")).mapToInt(Integer::parseInt).toArray();
    }
}
```
