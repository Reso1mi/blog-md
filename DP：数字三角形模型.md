---
title: 
   DP：数字三角形模型
tags: 
  [算法]
categories:
    [算法]
---


> 现在打算写一些短点的文章了，LeetCode系列不会再append了，如果写lc题会单独开一篇文章，然后写题解

## 数字三角形模型

[120. 三角形最小路径和](http://imlgw.top/2019/09/01/leetcode-dong-tai-gui-hua/#120-%E4%B8%89%E8%A7%92%E5%BD%A2%E6%9C%80%E5%B0%8F%E8%B7%AF%E5%BE%84%E5%92%8C)

[64. 最小路径和](http://imlgw.top/2019/09/01/leetcode-dong-tai-gui-hua/#64-%E6%9C%80%E5%B0%8F%E8%B7%AF%E5%BE%84%E5%92%8C)

[Path sum: three ways](http://imlgw.top/2019/09/01/leetcode-dong-tai-gui-hua/#Path-sum-three-ways)

之前写过题解的就不重复写了，还有很多类似的题就不一一列举出来了，详见[LeetCode动态规划](http://imlgw.top/2019/09/01/leetcode-dong-tai-gui-hua/)

> 后续如果还有类似的题会append进来
## [1027. 方格取数](https://www.acwing.com/problem/content/1029/)

设有 N×N 的方格图，我们在其中的某些方格中填入正整数，而其它的方格中则放入数字0。如下图所示：

![](https://files.catbox.moe/hhonhx.png)

某人从图中的左上角 A 出发，可以向下行走，也可以向右行走，直到到达右下角的 B 点。

在走过的路上，他可以取走方格中的数（取走后的方格中将变为数字0）。

此人从 A 点到 B 点共走了两次，试找出两条这样的路径，使得取得的数字和为最大。

**输入格式**

第一行为一个整数N，表示 N×N 的方格图。

接下来的每行有三个整数，第一个为行号数，第二个为列号数，第三个为在该行、该列上所放的数。

行和列编号从 1 开始。

一行“0 0 0”表示结束。

**输出格式**

输出一个整数，表示两条路径上取得的最大的和。

**数据范围**

N≤10

**输入样例：**
```c
8
2 3 13
2 6 6
3 5 7
4 4 14
5 2 21
5 6 4
6 3 15
7 2 14
0 0 0
```
**输出样例：**
```c
67
```
**难度：** 简单

**来源：**《信息学奥赛一本通》 , NOIP2000提高组

### 解法一

一开始确实没想出来，主要是看着范围好小，然后不确定复杂度，以为可以搜索的，看了y总讲解了一点，然后想过来了，自己写了一个N^4的解法
```java
//input output省略
//N^4 dp
public static int slove(int[][] w, int N) {
    int[][][][] dp = new int[N+1][N+1][N+1][N+1];
    for (int i1 = 1; i1 <= N; i1++) {
        for (int j1 = 1; j1 <= N; j1++) {
            for (int i2 = 1; i2 <= N; i2++) {
                for (int j2 = 1; j2 <= N; j2++) {
                    int temp = dp[i1][j1][i2][j2];
                    temp = Math.max(temp, dp[i1-1][j1][i2-1][j2]);
                    temp = Math.max(temp, dp[i1-1][j1][i2][j2-1]);
                    temp = Math.max(temp, dp[i1][j1-1][i2-1][j2]);
                    temp = Math.max(temp, dp[i1][j1-1][i2][j2-1]);
                    dp[i1][j1][i2][j2] = temp + ((i1==i2&&j1==j2) ? w[i1-1][j1-1] : (w[i1-1][j1-1] + w[i2-1][j2-1])); 
                }
            }
        }
    }
    return dp[N][N][N][N];
}
```
其实如果是只有一条路径就很简单，线性DP就行了，`dp[i][j] = Max(dp[i-1][j], dp[i][j-1])`，但是这题涉及到了两条路径，所以我们可以将状态定义为`dp[i1][j1][i2][j2]`，含义为从（1，1）到（i1，j1）,(i2，j2)这两个点的最大和，到每个点有两种走法，2*2一共4种子状态，取Max就行了

但是这里会有一个问题，某个点的val被取走之后就不能再被取了。也就是说，这两条路径可能会有交点，这种情况需要额外处理，而相交的情况实际上也就是`i1==i2 && j1 == j2`的时候，这种情况交点的值就只能算一次

### 解法二

y总讲的解法，优化了一维状态，N^3的解法，非常的巧妙，因为只能向下和向右走，所以从（1，1）到（N-1，N-1），走的总步数一定是`2*N`，那么我们就可以通过步数K计算出另一个坐标，从而省去一维状态

```java
//N^3优化，根据走的总步数计算另一个坐标位置
public static int slove2(int[][] w, int N) {
    int[][][] dp = new int[2*N+1][N+1][N+1];
    for (int k = 2; k <= 2*N; k++) {
        for (int i1 = 1; i1 <= N; i1++) {
            for (int i2 = 1; i2 <= N; i2++) {
                int j1 = k-i1, j2 = k-i2;
                if (1 <= j1 && j1 <= N && 1 <= j2 && j2 <= N) {
                    int temp = dp[k][i1][i2]; 
                    temp = Math.max(temp, dp[k-1][i1-1][i2-1]);
                    temp = Math.max(temp, dp[k-1][i1-1][i2]);
                    temp = Math.max(temp, dp[k-1][i1][i2-1]);
                    temp = Math.max(temp, dp[k-1][i1][i2]);
                    dp[k][i1][i2] = temp + ((i1==i2&&j1==j2) ? w[i1-1][j1-1] : (w[i1-1][j1-1] + w[i2-1][j2-1]));     
                }
            }
        }
    }
    return dp[2*N][N][N];
}
```
## [275. 传纸条](https://www.acwing.com/problem/content/277/)

小渊和小轩是好朋友也是同班同学，他们在一起总有谈不完的话题。

一次素质拓展活动中，班上同学安排坐成一个 m 行 n 列的矩阵，而小渊和小轩被安排在矩阵对角线的两端，因此，他们就无法直接交谈了。

幸运的是，他们可以通过传纸条来进行交流。

纸条要经由许多同学传到对方手里，小渊坐在矩阵的左上角，坐标(1,1)，小轩坐在矩阵的右下角，坐标(m,n)。

从小渊传到小轩的纸条只可以向下或者向右传递，从小轩传给小渊的纸条只可以向上或者向左传递。 

在活动进行中，小渊希望给小轩传递一张纸条，同时希望小轩给他回复。

班里每个同学都可以帮他们传递，但只会帮他们一次，也就是说如果此人在小渊递给小轩纸条的时候帮忙，那么在小轩递给小渊的时候就不会再帮忙，反之亦然。 

还有一件事情需要注意，全班每个同学愿意帮忙的好感度有高有低（注意：小渊和小轩的好心程度没有定义，输入时用0表示），可以用一个0-100的自然数来表示，数越大表示越好心。

小渊和小轩希望尽可能找好心程度高的同学来帮忙传纸条，即找到来回两条传递路径，使得这两条路径上同学的好心程度之和最大。

现在，请你帮助小渊和小轩找到这样的两条路径。

**输入格式**

第一行有2个用空格隔开的整数 m 和 n，表示学生矩阵有 m 行 n 列。

接下来的 m 行是一个 m∗n 的矩阵，矩阵中第 i 行 j 列的整数表示坐在第 i 行 j 列的学生的好心程度，每行的 n 个整数之间用空格隔开。

**输出格式**

输出一个整数，表示来回两条路上参与传递纸条的学生的好心程度之和的最大值。

**数据范围:**

1≤n,m≤50

**输入样例：**
```c
3 3
0 3 9
2 8 5
5 7 0
```
**输出样例：**
```c
34
```
### 解法一

和上面一题一样，虽然题目说的是一个从左上向右下，一个是从右下到左上，但是实际上依然是要求两条路径连接左上到右下，使得和最大，所以实际上直接照搬上面的代码就ok了

```java
//N^3
public static int solve(int[][] w, int m, int n){
    int[][][] dp = new int[m+n+1][m+1][m+1];
    for (int k = 2; k <= m+n; k++) {
        for (int i1 = 1; i1 <= m; i1++) {
            for (int i2 = 1; i2 <= m; i2++) {
                int j1 = k - i1, j2 = k - i2;
                if (j1 < 1 || j1 > n || j2 < 1 || j2 > n) continue;
                int temp = dp[k][i1][i2]; 
                temp = Math.max(temp, dp[k-1][i1-1][i2]);
                temp = Math.max(temp, dp[k-1][i1][i2-1]);
                temp = Math.max(temp, dp[k-1][i1][i2]);
                temp = Math.max(temp, dp[k-1][i1-1][i2-1]);
                dp[k][i1][i2] = temp + (i1==i2 ? w[i1-1][j1-1] : (w[i1-1][j1-1] + w[i2-1][j2-1]));
            }
        }
    }
    return dp[m+n][m][m];
}
```

## [1463. 摘樱桃 II](https://leetcode-cn.com/problems/cherry-pickup-ii/)

Difficulty: **困难**

给你一个 `rows x cols` 的矩阵 `grid` 来表示一块樱桃地。 `grid` 中每个格子的数字表示你能获得的樱桃数目。

你有两个机器人帮你收集樱桃，机器人 1 从左上角格子 `(0,0)` 出发，机器人 2 从右上角格子 `(0, cols-1)` 出发。

请你按照如下规则，返回两个机器人能收集的最多樱桃数目：

*   从格子 `(i,j)` 出发，机器人可以移动到格子 `(i+1, j-1)`，`(i+1, j)` 或者 `(i+1, j+1)` 。
*   当一个机器人经过某个格子时，它会把该格子内所有的樱桃都摘走，然后这个位置会变成空格子，即没有樱桃的格子。
*   当两个机器人同时到达同一个格子时，它们中只有一个可以摘到樱桃。
*   两个机器人在任意时刻都不能移动到 `grid` 外面。
*   两个机器人最后都要到达 `grid` 最底下一行。


给你一个 rows x cols 的矩阵 grid 来表示一块樱桃地。 grid 中每个格子的数字表示你能获得的樱桃数目。

你有两个机器人帮你收集樱桃，机器人 1 从左上角格子 (0,0) 出发，机器人 2 从右上角格子 (0, cols-1) 出发。

请你按照如下规则，返回两个机器人能收集的最多樱桃数目：

**示例 1：**

![](https://files.catbox.moe/da5rxh.png)
```java
输入：grid = [[3,1,1],[2,5,1],[1,5,5],[2,1,1]]
输出：24
解释：机器人 1 和机器人 2 的路径在上图中分别用绿色和蓝色表示。
机器人 1 摘的樱桃数目为 (3 + 2 + 5 + 2) = 12 。
机器人 2 摘的樱桃数目为 (1 + 5 + 5 + 1) = 12 。
樱桃总数为： 12 + 12 = 24 。
```

**示例 2：**

![](https://i.loli.net/2020/11/10/aSykAceXPH1snjM.png)
```java
输入：grid = [[1,0,0,0,0,0,1],[2,0,0,0,0,3,0],[2,0,9,0,0,0,0],[0,3,0,5,4,0,0],[1,0,2,3,0,0,6]]
输出：28
解释：机器人 1 和机器人 2 的路径在上图中分别用绿色和蓝色表示。
机器人 1 摘的樱桃数目为 (1 + 9 + 5 + 2) = 17 。
机器人 2 摘的樱桃数目为 (1 + 3 + 4 + 3) = 11 。
樱桃总数为： 17 + 11 = 28 。
```
**示例 3：**

```java
输入：grid = [[1,0,0,3],[0,0,0,3],[0,0,3,3],[9,0,3,3]]
输出：22
```
**示例 4：**

```java
输入：grid = [[1,1],[1,1]]
输出：4
```

**提示：**

- rows == grid.length
- cols == grid[i].length
- 2 <= rows, cols <= 70
- 0 <= grid[i][j] <= 100 


### 解法一

本来只写了前两题就打算push的，y总也只讲了这几道题，但是突然想起来很久之前的一道周赛题（27th双周赛T4），大概半年前了，没想到居然能想起来😄，看了下，和上面两题是一样的，随手写了下，一开始写的有bug，调试了好一会儿，最后把状态打出来手推了下，发现有些状态没有初始化好，导致递推的时候使用了非法的状态

先上暴力N^4的解法，这题范围不大，N^4勉强能扛过去
```java
//暴力N^4解法
public int cherryPickup2(int[][] grid) {
    int m = grid.length, n = grid[0].length;
    int[][][][] dp = new int[m+1][n+2][m+1][n+2];
    int INF = -0x3f3f3f3f;
    for (int i1 = 0; i1 <= m; i1++) {
        for (int j1 = 0; j1 <= n+1; j1++) {
            for (int i2 = 0; i2 <= m; i2++) {
                Arrays.fill(dp[i1][j1][i2], INF);
            }
        }
    }
    dp[1][1][1][n] = grid[0][0]+grid[0][n-1];
    for (int i1 = 2; i1 <= m; i1++) {
        for (int j1 = 1; j1 <= n; j1++) {
            for (int i2 = 2; i2 <= m; i2++) {
                for (int j2 = 1; j2 <= n; j2++) {
                    int temp = INF;
                    temp = Math.max(temp, dp[i1-1][j1][i2-1][j2]);
                    temp = Math.max(temp, dp[i1-1][j1][i2-1][j2+1]);
                    temp = Math.max(temp, dp[i1-1][j1][i2-1][j2-1]);
                    
                    temp = Math.max(temp, dp[i1-1][j1+1][i2-1][j2]);
                    temp = Math.max(temp, dp[i1-1][j1+1][i2-1][j2+1]);
                    temp = Math.max(temp, dp[i1-1][j1+1][i2-1][j2-1]);   
                    
                    temp = Math.max(temp, dp[i1-1][j1-1][i2-1][j2]);
                    temp = Math.max(temp, dp[i1-1][j1-1][i2-1][j2+1]);   
                    temp = Math.max(temp, dp[i1-1][j1-1][i2-1][j2-1]);
                    dp[i1][j1][i2][j2] = temp + ((i1==i2&&j1==j2) ? grid[i1-1][j1-1] : (grid[i1-1][j1-1] + grid[i2-1][j2-1]));
                }
            }
        }
    }
    int res = 0;
    for (int j1 = 0; j1 <= n; j1++) {
        for (int j2 = 0; j2 <= n; j2++) {
            res = Math.max(res, dp[m][j1][m][j2]);   
        }
    }
    return res;
}
```

### 解法二

和上面两题一样，两个机器人走的层数是固定的，都是m层，所以可以优化掉一维的状态，同时上面的写法看起来也很ugly，可以用方向变量简化写法

```java
//N^3解法，优化代码写法，简化代码
public int cherryPickup(int[][] grid) {
    int[] dir = {1, -1, 0};
    int m = grid.length, n = grid[0].length;
    //这题求最大值，在dp数组上边和左右两边各加一行作为边界
    //这里也可以用滚动数组优化下空间
    int[][][] dp = new int[m+1][n+2][n+2];
    int INF = -0x3f3f3f3f;
    //全部初始化为INF（实际上初始化两边和最上面的就ok了）
    for (int r = 0; r <= m; r++) {
        for (int j1 = 0; j1 <= n+1; j1++) {
            Arrays.fill(dp[r][j1], INF);
        }
    }
    dp[1][1][n] = grid[0][0] + grid[0][n-1];
    for (int r = 2; r <= m; r++) {
        for (int j1 = 1; j1 <= n; j1++) {
            for (int j2 = 1; j2 <= n; j2++) {
                //方向向量，简化写法
                for (int d1 = 0; d1 < 3; d1++) {
                    for (int d2 = 0; d2 < 3; d2++) {
                        dp[r][j1][j2] = Math.max(dp[r][j1][j2], dp[r-1][j1+dir[d1]][j2+dir[d2]]);
                    }
                }
                dp[r][j1][j2] += (j1==j2 ? grid[r-1][j1-1] : (grid[r-1][j1-1] + grid[r-1][j2-1]));
            }
        }
    }
    int res = 0;
    for (int j1 = 0; j1 <= n; j1++) {
        for (int j2 = 0; j2 <= n; j2++) {
            res = Math.max(res, dp[m][j1][j2]);   
        }
    }
    return res;
}
```