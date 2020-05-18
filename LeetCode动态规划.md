---
title: 
   LeetCode动态规划
tags: 
  [LeetCode,动态规划]
categories:
	[算法]
cover: http://static.imlgw.top/blog/20190903/6oWNO41xhu1B.jpg?imageslim
date: 2019/9/1
---

## [70. 爬楼梯](https://leetcode-cn.com/problems/climbing-stairs/)

You are climbing a stair case. It takes *n* steps to reach to the top.

Each time you can either climb 1 or 2 steps. In how many distinct ways can you climb to the top?

**Note:** Given *n* will be a positive integer.

**Example 1:**

```java
Input: 2
Output: 2
Explanation: There are two ways to climb to the top.

1. 1 step + 1 step
2. 2 steps
```

**Example 2:**

```java
Input: 3
Output: 3
Explanation: There are three ways to climb to the top.

1. 1 step + 1 step + 1 step
2. 1 step + 2 steps
3. 2 steps + 1 step
```

**解法一**

没啥好说的，可以说是最经典的dp题了

```java
public static int climbStairsDp(int n){
    int []dp=new int[n+1];
    dp[0]=1;
    dp[1]=1;
    for (int i=2;i<=n;i++) {
        dp[i]=dp[i-1]+dp[i-2];
    }
    return dp[n];
}
```

## [198. 打家劫舍](https://leetcode-cn.com/problems/house-robber/)

你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警。

给定一个代表每个房屋存放金额的非负整数数组，计算你在不触动警报装置的情况下，能够偷窃到的最高金额。

**Example 1:**

```java
Input: [1,2,3,1]
Output: 4
Explanation: Rob house 1 (money = 1) and then rob house 3 (money = 3).
             Total amount you can rob = 1 + 3 = 4.
```

**Example 2:**

```java
Input: [2,7,9,3,1]
Output: 12
Explanation: Rob house 1 (money = 2), rob house 3 (money = 9) and rob house 5 (money = 1).
             Total amount you can rob = 2 + 9 + 1 = 12.
```

**解法一**

简单的dp题，dp方程很容易得到，`dp[i]=max(dp[i-2]+nums[i],dp[i-1])` （2020.3.22更新了代码）

```java
//  Max[i]=max(nums[i-2]+nums[i],nums[i-1])
public int rob(int[] nums) {
    if(nums==null || nums.length<=0) return 0;
    int[] dp=new int[nums.length];
    dp[0]=nums[0];
    for(int i=1;i<nums.length;i++){
        dp[i]=Math.max(i>=2?dp[i-2]+nums[i]:nums[i],dp[i-1]);
    }
    return dp[nums.length-1];
}
```

时间空间都是O(N)，需要注意边界条件

**解法二**

空间复杂度的优化

```java
// Max[i]=max(pre+nums[i],cur);  
public int rob(int[] nums) {
    if(nums==null||nums.length<=0){
        return 0;
    }
    int pre= 0; //前一天
    int cur= 0; //当天，其实就是为了保存前后两天的dp[i]
    for (int i=0;i<nums.length;i++) {
        int temp=cur;
        cur=Math.max(pre+nums[i],cur);
        //向后走
        pre=temp;
    }
    return cur;
}
```

## [213. 打家劫舍 II](https://leetcode-cn.com/problems/house-robber-ii/)

你是一个专业的小偷，计划偷窃沿街的房屋，每间房内都藏有一定的现金。这个地方所有的房屋都**围成一圈**，这意味着第一个房屋和最后一个房屋是紧挨着的。同时，相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警。

给定一个代表每个房屋存放金额的非负整数数组，计算你在不触动警报装置的情况下，能够偷窃到的最高金额。

示例 1:

```java
输入: [2,3,2]
输出: 3
解释: 你不能先偷窃 1 号房屋（金额 = 2），然后偷窃 3 号房屋（金额 = 2）, 因为他们是相邻的。
```

示例 2:

```java
输入: [1,2,3,1]
输出: 4
解释: 你可以先偷窃 1 号房屋（金额 = 1），然后偷窃 3 号房屋（金额 = 3）。
     偷窃到的最高金额 = 1 + 3 = 4 。
```

**解法一**

与上一题相比房屋都围成一圈了，其实围成一圈也就说明`nums[0]`和`nums[len-1]`不能同时偷，所以就可以分两种情况分别求最大值，然后求最终的最大值

```java
public int rob(int [] nums){
    if(nums==null||nums.length<=0){
        return 0;
    }
    if(nums.length==1){
        return nums[0];
    }
    if(nums.length==2){
        return Math.max(nums[0],nums[1]);
    }
    int n=nums.length;
    return Math.max(rob(nums,1,n),rob(nums,0,n-1));
}

public int rob(int[] nums,int start,int end) {
    int pre=0; //前一家最大值
    int cur=0; //当前最大值
    for (int i=start;i<end;i++) {
        int temp=cur;
        cur=Math.max(pre+nums[i],cur);
        pre=temp;
    }
    return cur;
}
```

## [64. 最小路径和](https://leetcode-cn.com/problems/minimum-path-sum/)

给定一个包含非负整数的 m x n 网格，请找出一条从左上角到右下角的路径，使得路径上的数字总和为最小。

**说明**：每次只能向下或者向右移动一步。

**示例**:

```java
输入:
[
  [1,3,1],
  [1,5,1],
  [4,2,1]
]
输出: 7
解释: 因为路径 1→3→1→1→1 的总和最小。
```

很经典的DP题。

递推公式：`dp[i][j]  = grid[i][j] + Math.min(dp[i-1][j],dp[i][j-1])`

dp[i] [j] 为每个格子到`左上角`最短路径 ，每个格子到左上角的最小距离等于，左边格子最小值和上边格子最小值的最小值加上当前格子的值。

```java
public int minPathSum(int[][] grid) {
    int[][] dp=new int[grid.length][grid[0].length];
    for (int i=0;i<grid.length;i++) {
        for (int j=0;j<grid[0].length;j++) {
            //第一行
            if(i==0 && j!=0){
                dp[i][j]=grid[i][j]+dp[i][j-1];
            }else if(i!=0 && j==0){
                dp[i][j]=grid[i][j]+dp[i-1][j];
            }else if(i!=0 && j!=0){
                dp[i][j]=grid[i][j]+Math.min(dp[i-1][j],dp[i][j-1]);
            }else{
                dp[i][j]=grid[i][j];
            }
        }
    }
    return dp[grid.length-1][grid[0].length-1];
}
```

## [62. 不同路径](https://leetcode-cn.com/problems/unique-paths/)

一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为“Start” ）。

机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为“Finish”）。

问总共有多少条不同的路径？

![mark](http://static.imlgw.top/blog/20190818/DfwXk3ObWxX8.png?imageslim)

**说明：**m 和 n 的值均不超过 100。

**示例 1:**

```java
输入: m = 3, n = 2
输出: 3
解释:
从左上角开始，总共有 3 条路径可以到达右下角。

1. 向右 -> 向右 -> 向下
2. 向右 -> 向下 -> 向右
3. 向下 -> 向右 -> 向右
```

**示例 2:**

```java
输入: m = 7, n = 3
输出: 28
```

**动态规划解法**

这题算是我第一道自己做出来的DP题了，虽然和上面的一题一样的思路，但是还是挺开心的

```java
public int uniquePaths(int m, int n) {
    if(m<0||n<0){
        return 0;
    }
    int[][] dp=new int[n][m];
    for (int i=0;i<n;i++) {
        for (int j=0;j<m;j++) {
            if(i==0&&j!=0){
                dp[i][j]=1;
            }else if(j==0 && i!=0){
                dp[i][j]=1;
            }else if(j!=0&&i!=0){
                dp[i][j]=dp[i][j-1]+dp[i-1][j];
            }else{
                dp[i][j]=1;
            }
        }
    }
    return dp[n-1][m-1];
}
```
时间复杂度`O(m*n)`，空间复杂度`O(m*n)`可以看出来完全是按照前一题的模板来的，代码写的不好，可以把 if-else合并起来，这题空间复杂度其实还可以优化成 O(m)或O(n)的，按行来向下走

**空间复杂度优化**

![leetCode题解](http://static.imlgw.top/blog/20190824/7uacSvsRMaWq.png?imageslim)

```java
public static int uniquePaths2(int m, int n) {
    if(m<0||n<0){
        return 0;
    }
    int[] dp=new int[n]; 
    Arrays.fill(dp,1); //第一行，第一列均为 1
    for (int i=1;i<m;i++) { //一行一行向下走
        for (int j=1;j<n;j++) {
            //这里的dp[j]和dp[j-1]分别就代表了 上边 和 左边的元素
            dp[j]=dp[j]+dp[j-1]; //求每一行每个元素的dp值
        }
    }
    return dp[n-1]; //最后一行最后一个dp值就是结果
}
```

**组合数公式**

这题其实就是我们初中还是高中学的排列组合的问题，机器人一共走`m+n-2` 步，向下会走`m-1` 步，向下会走`n-1` 步，所以实际上就是求一个组合数的结果 `C(n-1，m+n-2)` ，

```java
public int uniquePaths(int m, int n) {
    if(m<0||n<0){
        return 0;
    }
    int step=m+n-2;
    int down=n-1;
    long res=1;
    for (int i=1;i<=down;i++) {
        res=res*(step-down+i)/i; //递推
    }
    return (int)res;
}
```

## [63. 不同路径 II](https://leetcode-cn.com/problems/unique-paths-ii/)

一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为“Start” ）。

机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为“Finish”）。

现在考虑网格中有障碍物。那么从左上角到右下角将会有多少条不同的路径？

![mark](http://static.imlgw.top/blog/20190818/DfwXk3ObWxX8.png?imageslim)

网格中的障碍物和空位置分别用 1 和 0 来表示。

**说明：**m 和 n 的值均不超过 100。

**示例 1:**

```java
输入:
[
  [0,0,0],
  [0,1,0],
  [0,0,0]
]
输出: 2
解释:
3x3 网格的正中间有一个障碍物。
从左上角到右下角一共有 2 条不同的路径：

1. 向右 -> 向右 -> 向下 -> 向下
2. 向下 -> 向下 -> 向右 -> 向右
```

**解法一**

看到这种路径的题，首先想到的可能是递归回溯

```java
//dfs回溯 , TLE
private int[][] direction={{0,1},{1,0}};

private int count=0;

public int uniquePathsWithObstacles(int[][] obstacleGrid) {
    if (obstacleGrid==null || obstacleGrid.length<=0 || obstacleGrid[0][0]==1) {
        return 0;
    }
    boolean[][] visit=new boolean[obstacleGrid.length][obstacleGrid[0].length];
    dfs(obstacleGrid,visit,0,0);
    return count;
}

public void dfs(int[][] obstacleGrid,boolean[][] visit,int x,int y) {
    //!=1 是为了coner case
    if (x==obstacleGrid.length-1 && y==obstacleGrid[0].length-1 && obstacleGrid[x][y]!=1) {
        count++;
        return;
    }
    for (int i=0;i<direction.length;i++) {
        int nx=x+direction[i][0];
        int ny=y+direction[i][1];
        if (nx<obstacleGrid.length && ny<obstacleGrid[0].length && !visit[nx][ny] && obstacleGrid[nx][ny] !=1) {
            visit[nx][ny]=true;
            dfs(obstacleGrid,visit,nx,ny);
            visit[nx][ny]=false;
        }
    }
}
```
结果很惨，TLE了，只跑了不到一半的case就TLE了

**解法二**

动态规划的解法，其实和上面一题是一样的，只不过需要注意障碍物位置的dp值

```java
public int uniquePathsWithObstacles(int[][] obstacleGrid) {
    if (obstacleGrid==null || obstacleGrid.length<=0 || obstacleGrid[0][0]==1) {
        return 0;
    }
    int[][] dp=new int[obstacleGrid.length][obstacleGrid[0].length];
    for (int i=0;i<obstacleGrid.length;i++) {
        for (int j=0; j<obstacleGrid[0].length;j++) {
            if (obstacleGrid[i][j]==1) {
                dp[i][j]=0;
            }else if (i==0 && j==0) {
                dp[0][0]=1;
            }else if(i==0 && j!=0){
                dp[0][j]=dp[0][j-1];
            }else if (j==0 && i!=0) {
                dp[i][0]=dp[i-1][0];
            }else{
                dp[i][j]=dp[i-1][j]+dp[i][j-1];
            }
        }
    }
    return dp[obstacleGrid.length-1][obstacleGrid[0].length-1];
}
```
和上面一样也可以进行空间上的优化

```java
//一维dp
public int uniquePathsWithObstacles(int[][] obstacleGrid) {
    if (obstacleGrid==null || obstacleGrid.length<=0 || obstacleGrid[0][0]==1) {
        return 0;
    }
    //以行为单位向下走
    int[] dp=new int[obstacleGrid[0].length];
    dp[0]=1;
    for (int i=0;i<obstacleGrid.length;i++) {
        for (int j=0; j<obstacleGrid[0].length;j++) {
            if (obstacleGrid[i][j]==1) {
                dp[j]=0;
            }else if(i==0 && j!=0){ //第一行
                dp[j]=dp[j-1];
            }else if(i!=0 && j!=0){ 
                dp[j]=dp[j]+dp[j-1];
            }
            //每一行第一列的dp[j]=dp[j],和上一行的第一列保持一致就行
        }
    }
    return dp[obstacleGrid[0].length-1];
}
```
## [576. 出界的路径数](https://leetcode-cn.com/problems/out-of-boundary-paths/)

给定一个 **m × n** 的网格和一个球。球的起始坐标为 **(i,j)** ，你可以将球移到**相邻**的单元格内，或者往上、下、左、右四个方向上移动使球穿过网格边界。但是，你**最多**可以移动 **N** 次。找出可以将球移出边界的路径数量。答案可能非常大，返回 结果 mod 109 + 7 的值。

**示例 1：**

```java
输入: m = 2, n = 2, N = 2, i = 0, j = 0
输出: 6
解释:
```

**示例 2：**

```java
输入: m = 1, n = 3, N = 3, i = 0, j = 1
输出: 12
解释:
```

**说明:**

1. 球一旦出界，就不能再被移动回网格内。
2. 网格的长度和高度在 [1,50] 的范围内。
3. N 在 [0,50] 的范围内。

**解法一**

首先写出来的解法，自顶向下，记忆化递归

```java
int[][] dir={{0,1},{1,0},{-1,0},{0,-1}};

int mod=(int)1e9+7;

Long[][][] dp=null;

public int findPaths(int m, int n, int N, int i, int j) {
    dp=new Long[m][n][N+1];
    return (int)(dfs(m,n,i,j,N)%mod);
}

public long dfs(int m,int n,int x,int y,int k){
    if(k==0) return 0;
    if(dp[x][y][k]!=null){
        return dp[x][y][k];
    }
    long count=0;
    for(int i=0;i<dir.length;i++){
        int nx=x+dir[i][0];
        int ny=y+dir[i][1];
        if(!valid(m,n,nx,ny)){
            count++;
            continue;
        }
        count=(count+dfs(m,n,nx,ny,k-1))%mod;
    }
    return dp[x][y][k]=(count)%mod;
}

public boolean valid(int m,int n,int x,int y){
    return x>=0 && x<m && y>=0 && y<n;
}
```

有一点需要注意，这里不需要visit，因为有K的限制，不用担心死循环，感觉加了visit反而会错？

**解法二**

整体上来说还是属于比较简单的动态规划，至少递推方程好想

```java
//自底向上递推
public int findPaths(int m, int n, int N, int i, int j) {
    int mod=(int)1e9+7;
    //群里偷学到的
    int[] dir={0,1,0,-1,0};
    long[][][] dp=new long[m][n][N+1];
    for (int k=1;k<=N;k++) { //想想K为什么在最外层
        for (int r=0;r<m;r++) {
            for (int c=0;c<n;c++) {
                for (int d=0;d<4;d++) {
                    int nx=r+dir[d];
                    int ny=c+dir[d+1];
                    if(nx<0 || ny<0 || nx>=m ||ny>=n){
                        dp[r][c][k]++;
                    }else{
                        dp[r][c][k]=(dp[r][c][k]+dp[nx][ny][k-1])%mod;
                    }
                }
                //提前结束，只需要i,j,N就行了(貌似没有快多少)
                if(k==N && r==i && c==j){
                    return (int)(dp[i][j][N]);
                }
            }
        }
    }
    //return (int)(dp[m-1][n-1][N]); md一开始返回错了，看了半天
    return 0;
}
```
K放在最外层的原因其实很容易想到，我们求某一个点的时候，我们需要周围4个方向的`K-1`的值，这些值必须是已经计算过的，所以K肯定是要放在最外层的

## [688. “马”在棋盘上的概率](https://leetcode-cn.com/problems/knight-probability-in-chessboard/)

已知一个 `N`x`N` 的国际象棋棋盘，棋盘的行号和列号都是从 0 开始。即最左上角的格子记为 `(0, 0)`，最右下角的记为 `(N-1, N-1)`。 

现有一个 “马”（也译作 “骑士”）位于 `(r, c)` ，并打算进行 `K` 次移动。 

如下图所示，国际象棋的 “马” 每一步先沿水平或垂直方向移动 2 个格子，然后向与之相垂直的方向再移动 1 个格子，共有 8 个可选的位置。

![YsAXFS.png](https://s1.ax1x.com/2020/05/15/YsAXFS.png) 

现在 “马” 每一步都从可选的位置（包括棋盘外部的）中独立随机地选择一个进行移动，直到移动了 `K` 次或跳到了棋盘外面。

求移动结束后，“马” 仍留在棋盘上的概率。

**示例：**

```java
输入: 3, 2, 0, 0
输出: 0.0625
解释: 
输入的数据依次为 N, K, r, c
第 1 步时，有且只有 2 种走法令 “马” 可以留在棋盘上（跳到（1,2）或（2,1））。对于以上的两种情况，各自在第2步均有且只有2种走法令 “马” 仍然留在棋盘上。
所以 “马” 在结束后仍在棋盘上的概率为 0.0625。
```

**注意：**

- `N` 的取值范围为 [1, 25]
- `K` 的取值范围为 [0, 100]
- 开始时，“马” 总是位于棋盘上

**错误解法一**

首先采用了和上一题类似做法，明显是不行的

```java
//小数据可以过，大数据过不了
//K[0,100]直接8^100就爆了，所以肯定不能像上面那样求，dp得存概率
public double knightProbability(int N, int K, int r, int c) {
    int[][] dir={{-2,-1},{-2,1},{-1,2},{1,2},{2,1},{2,-1},{1,-2},{-1,-2}};
    int[][][] dp=new int[N][N][K+1];
    for(int i=0;i<N;i++){
        for(int j=0;j<N;j++){
            //这题如果反着求出界路径就错了，得正向求
            dp[i][j][0]=1; 
        }
    }
    for(int k=1;k<=K;k++){
        for(int i=0;i<N;i++){
            for(int j=0;j<N;j++){
                for(int d=0;d<dir.length;d++){
                    int nx=i+dir[d][0];
                    int ny=j+dir[d][1];
                    if(nx>=0 && nx<N && ny>=0 && ny<N){
                        dp[i][j][k]+=(dp[nx][ny][k-1]);
                    }
                }
            }
        }
    }
    return dp[r][c][K]/Math.pow(8,K);
}
```
> 多说一点，这里为什么不和上面一样，算出界的路径数，然后用`8^K`减去出界数量，然后除以`8^K`？其实主要是出界的这里并不好算，这里要求的是概率，出界的路径并不能完全包含出界的方式（可以理解为出界后仍然可以走）

**解法二**

标准的dp解法，注意概率的计算就行了

```java
//DP存概率AC解法
public double knightProbability(int N, int K, int r, int c) {
    int[][] dir={{-2,-1},{-2,1},{-1,2},{1,2},{2,1},{2,-1},{1,-2},{-1,-2}};
    double[][][] dp=new double[N][N][K+1];
    for(int i=0;i<N;i++){
        for(int j=0;j<N;j++){
            dp[i][j][0]=1;
        }
    }
    for(int k=1;k<=K;k++){
        for(int i=0;i<N;i++){
            for(int j=0;j<N;j++){
                for(int d=0;d<dir.length;d++){
                    int nx=i+dir[d][0];
                    int ny=j+dir[d][1];
                    if(nx>=0 && nx<N && ny>=0 && ny<N){
                        //md，一开始不知道为啥想到求独立事件P(A U B)上去了
                        //但是其实两者并不是独立事件，是互斥事件直接P(A)+P(B)就行了
                        dp[i][j][k]+=(dp[nx][ny][k-1])/8;
                    }
                }
                //提前返回
                if(i==r && j==c && k==K){
                    return dp[i][j][k];
                }
            }
        }
    }
    return dp[r][c][K];
}
```

**解法三**

记忆化的方式，其实会比上面递推的会快一点，上面递推的会有很多无用状态

```java
//自顶向上
Double[][][] dp=null;

int[][] dir={{-2,-1},{-2,1},{-1,2},{1,2},{2,1},{2,-1},{1,-2},{-1,-2}};

public double knightProbability(int N, int K, int r, int c) {
    dp=new Double[N][N][K+1];
    return dfs(N,K,r,c);
}

public double dfs(int N,int k,int x,int y){
    if(k==0) return 1.0;
    if(dp[x][y][k]!=null){
        return dp[x][y][k];
    }
    double p=0;
    for (int i=0;i<dir.length;i++) {
        int nx=x+dir[i][0];
        int ny=y+dir[i][1];
        if(nx>=0 && nx<N && ny>=0 && ny<N){
            p+=dfs(N,k-1,nx,ny)/8;
        }
    }
    return dp[x][y][k]=p;
}
```

## [935. 骑士拨号器](https://leetcode-cn.com/problems/knight-dialer/)

国际象棋中的骑士可以按下图所示进行移动：

![YsAXFS.png](https://s1.ax1x.com/2020/05/15/YsAXFS.png) .         ![YcP3o8.png](https://s1.ax1x.com/2020/05/16/YcP3o8.png)


这一次，我们将 “骑士” 放在电话拨号盘的任意数字键（如上图所示）上，接下来，骑士将会跳 N-1 步。每一步必须是从一个数字键跳到另一个数字键。

每当它落在一个键上（包括骑士的初始位置），都会拨出键所对应的数字，总共按下 `N` 位数字。

你能用这种方式拨出多少个不同的号码？

因为答案可能很大，**所以输出答案模 10^9 + 7**。

**示例 1：**

```java
输入：1
输出：10
```

**示例 2：**

```java
输入：2
输出：20
```

**示例 3：**

```java
输入：3
输出：46
```

**提示：**

- `1 <= N <= 5000`

**解法一**

先上一个极其粗糙的解法，也是最开始的解法

```java
int[][] dir={{-2,-1},{-2,1},{-1,2},{1,2},{2,1},{2,-1},{1,-2},{-1,-2}};

int mod=(int)1e9+7;

Integer[][][] dp=null;

public int knightDialer(int N) {
    int[][] grid={{1,2,3},{4,5,6},{7,8,9},{-1,0,-1}};
    int res=0;
    dp=new Integer[4][3][N];
    for(int i=0;i<grid.length;i++){
        for(int j=0;j<grid[0].length;j++){
            if(grid[i][j]!=-1){
                res=(res+dfs(grid,i,j,N-1))%mod;
            }
        }
    }
    return res;
}

public int dfs(int[][] grid,int x,int y,int N){
    if(N==0) return 1;
    if(dp[x][y][N]!=null) return dp[x][y][N];
    int res=0; //一开始赋值成1了。。从当前位置起跳，所以当前位置不应该有初始值
    for(int i=0;i<dir.length;i++){
        int nx=x+dir[i][0];
        int ny=y+dir[i][1];
        if(valid(grid,nx,ny)){
            res=(res+dfs(grid,nx,ny,N-1))%mod;
        }
    }
    return dp[x][y][N]=res;
}

public boolean valid(int[][] grid,int x,int y){
    return x>=0 && y>=0 && x<grid.length && y<grid[0].length && grid[x][y]!=-1;
}
```

**解法二**

小巧精致的解法

```java
int mod=(int)1e9+7;

Integer[][] dp=null;

public int knightDialer(int N) {
    //直接看图建立对应关系
    int[][] next={{4,6},{6,8},{7,9},{4,8},{0,3,9},{},{0,1,7},{2,6},{1,3},{2,4}};
    int res=0;
    dp=new Integer[10][N];
    for(int i=0;i<=9;i++){
        res=(res+dfs(i,N-1,next))%mod;
    }
    return res;
}

public int dfs(int num,int N,int[][] next){
    if(N==0) return 1;
    if(dp[num][N]!=null) return dp[num][N];
    int res=0; //注意别写成1了
    for(int i=0;i<next[num].length;i++){
        res=(res+dfs(next[num][i],N-1,next))%mod;
    }
    return dp[num][N]=res;
}
```

**解法三**

递推的方式

```java
//递推的方式
public int knightDialer(int N) {
    //next[i]: i下一步能跳到的位置
    int[][] next={{4,6},{6,8},{7,9},{4,8},{0,3,9},{},{0,1,7},{2,6},{1,3},{2,4}};
    int[][] dp=new int[N][10];
    Arrays.fill(dp[0],1); //base dp[0]
    int mod=(int)1e9+7;
    int res=0;
    for (int i=1;i<N;i++) {
        for (int num=0;num<=9;num++) {
            for (int j=0;j<next[num].length;j++) {
                dp[i][num]=(dp[i][num]+dp[i-1][next[num][j]])%mod;   
            }
        }
    }
    for(int i=0;i<=9;i++) res=(res+dp[N-1][i])%mod;
    return res;
}
```

## [303. 区域和检索 - 数组不可变](https://leetcode-cn.com/problems/range-sum-query-immutable/)

给定一个整数数组  nums，求出数组从索引 i 到 j  (i ≤ j) 范围内元素的总和，包含 i,  j 两点。

**示例：**

```java
给定 nums = [-2, 0, 3, -5, 2, -1]，求和函数为 sumRange()

sumRange(0, 2) -> 1
sumRange(2, 5) -> -1
sumRange(0, 5) -> -3
```

**说明:**

- 你可以假设数组不可变。
- 会多次调用 sumRange 方法。

**解法一**

这题其实一开始就是想的缓存一下之前调用产生的值，没有理解到这题的要点。

```java
class NumArray {

    private int[] sums=null;

    public NumArray(int[] nums) {
        if (nums!=null && nums.length>0) {
            sums=new int[nums.length];
            sums[0]=nums[0];
            for (int i=1;i<sums.length;i++) {
                sums[i]=sums[i-1]+nums[i];
            }    
        }
        
    }
    
    public int sumRange(int i, int j) {
        if(i==0){
            return sums[j];
        }
        return sums[j]-sums[i-1];
    }
}

/**
 * Your NumArray object will be instantiated and called as such:
 * NumArray obj = new NumArray(nums);
 * int param_1 = obj.sumRange(i,j);
 */
```

这题关键有两个地方，一个是 `sumRange(i , j) = sumRange(0, j)-sumRange(0, i-1)` ，另一个关键就是在构造器里面将所有的 `sumRange(0 , i)` 给预先求出来，这样在后面的调用时时间复杂度就是`O(1)`的了。

这里求`sumRange(0, i)`的过程就是一个很简单的dp.

## [413. 等差数列划分](https://leetcode-cn.com/problems/arithmetic-slices/)

如果一个数列至少有三个元素，并且任意两个相邻元素之差相同，则称该数列为等差数列。

例如，以下数列为等差数列:

```java
1, 3, 5, 7, 9
7, 7, 7, 7
3, -1, -5, -9
```


以下数列不是等差数列。

```java
1, 1, 2, 5, 7
```


数组 A 包含 N 个数，且索引从0开始。数组 A 的一个子数组划分为数组 (P, Q)，P 与 Q 是整数且满足 0<=P<Q<N 。

如果满足以下条件，则称子数组(P, Q)为等差数组：

元素 A[P], A[p + 1], ..., A[Q - 1], A[Q] 是等差的。并且 P + 1 < Q 。

函数要返回数组 A 中所有为等差数组的子数组个数。

 **示例:**

```java
A = [1, 2, 3, 4]

返回: 3, A 中有三个子等差数组: [1, 2, 3], [2, 3, 4] 以及自身 [1, 2, 3, 4]。
```

dp的题啊，答案一看就明白，就是自己想不到。。。

```java
public int numberOfArithmeticSlices(int[] A) {
    if(A.length<3){
        return 0;
    }
    //dp[i]=dp[i-1]+1;
    int []dp=new int[A.length]; //以 A[i] 结尾的等差数列有多少个
    for (int i=2;i<A.length;i++) {
        if(A[i]-A[i-1]==A[i-1]-A[i-2]){
            dp[i]=dp[i-1]+1;
        }
    }
    int res=0;
    for (int i=0;i<dp.length;i++) {
        res+=dp[i];
    }
    return res;
}
```

就是没想到要对以当前元素结尾的动态数组数量做dp，当然上面的做法的空间还可以优化成O(1)的

```java
public int numberOfArithmeticSlices2(int[] A) {
    if(A.length<3){
        return 0;
    }
    //dp[i]=dp[i-1]+1;
    int dp=0;
    int res=0;
    for (int i=2;i<A.length;i++) {
        if(A[i]-A[i-1]==A[i-1]-A[i-2]){
            dp=dp+1;
            res+=dp;
        }else{
            dp=0;
        }
    }
    return res;
}
```

只需要根据上一次的dp值更新就行，如果break等差数列，就置为0

------

## [343. 整数拆分](https://leetcode-cn.com/problems/integer-break/)

给定一个正整数 n，将其拆分为**至少**两个正整数的和，并使这些整数的乘积最大化。 返回你可以获得的最大乘积。

**示例 1:**

```java
输入: 2
输出: 1
解释: 2 = 1 + 1, 1 × 1 = 1。
```

**示例 2:**

```java
输入: 10
输出: 36
解释: 10 = 3 + 3 + 4, 3 × 3 × 4 = 36。
```

**说明:** 你可以假设 n 不小于 2 且不大于 58。

**解法一**

别问，问就是暴力

![leetCode题解97wgl](http://static.imlgw.top/blog/20190822/BKqmfoXMVq3Q.png?imageslim)

**递推式：** `F(n) = max {i * F(n - i)}，i = 1，2，...，n - 1`  

穷举每一种分解情况，求最大值

```java
public static int integerBreak(int n) {
    //递归出口
    if (n==2) {
        return 1;
    }
    int res=-1;
    for (int j=1;j<n;j++) {
        //第一个Max求最大值，第二个Max主要是为了考虑 只分两个数就是最大值的情况（<=7的情况）
        res=Math.max(res, Math.max((n-j)*j,integerBreak(n-j)*j));
    }
    return res;
}
```

需要注意的就是第二个Max 别忘了**只分为两个数就到达最大值**的情况，比如n<=7的时候，`integerBreak(n-j)*j)`  至少是三个，下面的DP也是一样的要注意别忘了两个数的情况

递归的时间复杂度应该是`O(N!)` ，后面的case会超时，其实很明显，递归过程中会有很多的重复计算所以可以采用记忆化递归缓存一下之前的值。

```java
private  Integer[] cache=null;
//只顶向下,记忆化递归
public  int integerBreak(int n) {
    cache=new Integer[n+1];
    return breakInteger(n);
}
public  int breakInteger(int n) {
    if (cache[n]!=null) {
        return cache[n];
    }
    //递归出口
    if (n==2) {
        return 1;
    }
    int res=-1;
    for (int j=1;j<n;j++) {
        res=Math.max(res, Math.max((n-j)*j,breakInteger(n-j)*j));
    }
    cache[n]=res;
    return res;
}
```
这样的解法是可以AC的，缓存之前计算的结果，避免了重复的计算

**解法二**

动态规划，结合上面的递归得到`递推方程`

`dp[i] = max(dp[i]，max(dp[ i-j ] * j ，(i-j) * j));`

```java
public static int integerBreak2(int n) {
    if (n==2) {
        return 1;
    }
    int [] dp=new int[n+1];
    dp[2]=1; //0,1不考虑
    for (int i=3;i<=n;i++) {
        for (int j=1;j<i;j++) {
            dp[i]=Math.max(dp[i],Math.max(dp[i-j]*j,(i-j)*j));
        }
    }
    return dp[n];
}
```
**解法三**

打板找规律，笔试的时候能直接一眼看出来规律的肯定优先找规律了

```java
public static int integerBreak3(int n) {
    int[] base={1,2,4,6,9,12};
    if(n<=7){
        return base[n-2];
    }
    int[] dp=new int[n+1];
    dp[2]=base[0];
    dp[3]=base[1];
    dp[4]=base[2];
    dp[5]=base[3];
    dp[6]=base[4];
    dp[7]=base[5];
    for (int i=8;i<=n;i++) {
        dp[i]=dp[i-3]*3; //N>7之后 dp[N]=dp[N-3]*3
    }
    return dp[n];
}
```

**解法四**

数学分析，其实分析一下会发现，只有2和3是分解后值比**本身还要小**的，4分解后相等，这样一分析，从整体上来看其实就清楚了，只要分到2或者3的时候就不再分就行了，此时的值一定是最大值，如果2，3继续分值就会减小

```java
public int integerBreak(int n) {
    int[] base={1,2,4};
    if(n<=4){
        return base[n-2];
    }
    int res=1;
    while(n>=5){
        res*=3;
        n-=3;
    }
    res*=n;
    return res;
}
```

取模写法，这里其实涉及到取模的一个规则，`(a*b)%mod=(a%mid * b%mid) % mod` 

```java
public int cuttingRope(int n) {
    if(n<=3) return n-1;
    long res=1;
    while(n>=5){
        n-=3;
        res=(res*3)%1000000007;
    }
    return (int)(res*n%1000000007);
}
```

## [279. 完全平方数](https://leetcode-cn.com/problems/perfect-squares/)

给定正整数 n，找到若干个完全平方数（比如 1, 4, 9, 16, ...）使得它们的和等于 n。你需要让组成和的完全平方数的个数最少。

**示例 1:**

```java
输入: n = 12
输出: 3 
解释: 12 = 4 + 4 + 4.
```

**示例 2:**

```java
输入: n = 13
输出: 2
解释: 13 = 4 + 9.
```

**解法一**

动态规划，不容易啊，自己推出了DP方程，没看答案，其实也借鉴了上一题的思路

`dp[i] =min(dp[i]，dp[i-j*j]+1)` 

其实也是将整数拆分，不过拆分的时候按照 **完全平方数** 来拆分，完全平方数的肯定是 1所以最后要加个 1

```java
public static int numSquares(int n) {
    int[] dp=new int[n+1];
    dp[0]=0;
    for (int i=1;i<=n;i++) {
        dp[i]=Integer.MAX_VALUE; //dp[i]初始化
        for (int j=1;i>=j*j;++j){
            dp[i]=Math.min(dp[i],dp[i-j*j]+1);
        }
    }
    return dp[n];
}
```
50%左右，看了下时间复杂度应该是 `O(N^1.5)`  还是挺高的（评论区用Py的都超时了）

**解法二**

BFS，将数字按照完全平方数连接为一个图，然后BFS求一个从n到0的最短的路径

```java
public static int numSquares(int n) {
    Queue<Pair> queue=new LinkedList<>();
    boolean[] visit=new boolean[n+1];
    queue.add(new Pair(n,0));
    visit[n]=true;
    while(!queue.isEmpty()){
        Pair pair=queue.poll();
        int num=pair.num;
        int step=pair.step;
        if (num==0) {
            return step;
        }
        for (int i=1;i*i<=num;i++) {
            int temp=num-i*i;
            if (!visit[temp]) {
                queue.add(new Pair(temp,step+1));
                visit[temp]=true;
            }
        }
    }
    return -1;
}

static class Pair{
    public int step;
    public int num;
    public Pair(int num,int step){
        this.num=num;
        this.step=step;
    }
}
```

这题最快的做法应该是利用 **四平方和定理** 我也不太清楚，所以暂时先不研究这种做法

## [91. 解码方法](https://leetcode-cn.com/problems/decode-ways/)

一条包含字母 A-Z 的消息通过以下方式进行了编码：

```java
'A' -> 1
'B' -> 2
...
'Z' -> 26
```


给定一个只包含数字的非空字符串，请计算解码方法的总数。

**示例 1:**

```java
输入: "12"
输出: 2
解释: 它可以解码为 "AB"（1 2）或者 "L"（12）。
```

**示例 2:**

```java
输入: "226"
输出: 3
解释: 它可以解码为 "BZ" (2 26), "VF" (22 6), 或者 "BBF" (2 2 6) 。
```

**解法一**

哇，这题我吐了，好不容易看出来了递推公式，被边界给整死了。。。面向测试用例编程。。

```java
public static int numDecodings(String s) {
    if (s.startsWith("0")) {
        return 0;
    }
    int[] s_nums=new int[s.length()];
    for (int i=0;i<s.length();i++) {
        s_nums[i]=Integer.valueOf(s.charAt(i)-48);
    }

    int[] dp =new int[s_nums.length];
    Arrays.fill(dp,1);
    dp[0]=1;
    int res=1;
    for (int j=1;j<s_nums.length;j++) {
        if(s_nums[j]==0 && s_nums[j-1]==0 || (s_nums[j]==0 && s_nums[j-1]>2)){
            //直接return
            return 0;
        }
        if( s_nums[j]==0 || s_nums[j-1]==0 || (j<s_nums.length-1 &&s_nums[j+1]==0)){
            //为了处理 12120这种情况
            res*=dp[j-1];
            continue;
        }
        if(s_nums[j-1]*10+s_nums[j]<=26){
            if(j==1){
                dp[j]=2;
            }else{
                dp[j]=dp[j-1]+dp[j-2];    
            }
        }else{
            //划分点。。。
            res*=dp[j-1];
            dp[j-1]=1;
            dp[j]=1;
        }
    }
    //最后一段
    res*=dp[s_nums.length-1];
    return res;
}
```

首先要推出这题的递推公式，我也是在纸上画了半天才看出来，如果这个数组前后两个元素之和都是小于26的那么这一段的编码方式就是一个**斐波拉契数列** ，也就是 `dp[i]=dp[i-1]+dp[i-2]`

80%左右，这题其实一开始并不是这样做的，我觉得我开始的思路还是挺好的，将字符串分段，当前一个和后一个无法合并的时候就作为一个分界点，最后将每一段的编码方式相乘就是结果了。

但是 ！！！ 我忽略了0这个东西了，这题后面给的case里面都是带有0的！！！！处理这个边界处理了一上午终于跑过了。。。代码也没什么好说的，写的太烂了，这种代码没啥意义，都是走一步看一步。

**解法二**

```java
public static int numDecodings2(String s) {
    if (s.startsWith("0")) {
        return 0;
    }
    int[] s_nums=new int[s.length()];
    for (int i=0;i<s.length();i++) {
        s_nums[i]=Integer.valueOf(s.charAt(i)-48);
    }
    int[] dp=new int[s.length()+1];
    dp[0]=1;
    dp[1]=1;
    for (int i=2;i<=s.length();i++) {
        int a=s_nums[i-1];
        if(a!=0){
            //到这里dp[i]==0,没有初始化,默认就是0
            //其实等价于dp[i]=dp[i-1],延续前一个字符的状态
            dp[i]+=dp[i-1];
        }

        if(s_nums[i-2]==0){
            //如果前前一个字符为0那么就不用dp了,保持i-1的状态就OK
            continue;
        }

        //前前一个字符和
        int b=s_nums[i-2]*10+s_nums[i-1];
        if(b<=26){
            dp[i]+=dp[i-2]; //上下这两个dp必须执行一个否则最后就 return 0
        }
    }
    return dp[s_nums.length];
}
```

这种写法就清晰多了，分析各种情况，dp代表截至字符的编码方式

💬  `s_num[i-1]!=0 && s_num[i-2]!=0 && s_num[i-2]+s_num[i-1] <=26`  

类似于 **112**  这样的，这就是正常dp的情况符合斐波拉契数列

💬  `s_num[i-1]==0 && s_num[i-2]!=0 && s_num[i-2]+s_num[i-1] <=26`

类似于 **102** 这样的，这样就直接`继承 dp[i-2]的状态`就行了，相当于`dp[i]=dp[i-2]`

💬  `s_num[i-1]!=0 && s_num[i-2]!=0 && s_num[i-2]+s_num[i-1] >26`

类似于 **132** 这样的，这样就直接`继承 dp[i-2]的状态`就行了，相当于`dp[i]=dp[i-2]`

💬 `s_num[i-1]!=0 && s_num[i-2]==0 && s_num[i-2]+s_num[i-1] <=26`

类似于 **...012**这样的，`dp[i]=dp[i-1]`

💬 `s_num[i-1]==0 && s_num[i-2]==0` 或者`s_num[i-1]==0 && s_num[i-2]!=0 && s_num[i-2]+s_num[i-1]>26`

类似于**00** 和**30** 这样的， 这样就代表这个字符串无法编码了 **dp[i]=0** 最后返回的就是0

## [5375. 恢复数组](https://leetcode-cn.com/problems/restore-the-array/)

某个程序本来应该输出一个整数数组。但是这个程序忘记输出空格了以致输出了一个数字字符串，我们所知道的信息只有：数组中所有整数都在 [1, k] 之间，且数组中的数字都没有前导 0 。

给你字符串 s 和整数 k 。可能会有多种不同的数组恢复结果。

按照上述程序，请你返回所有可能输出字符串 s 的数组方案数。

由于数组方案数可能会很大，请你返回它对 10^9 + 7 取余 后的结果。

**示例 1：**

```java
输入：s = "1000", k = 10000
输出：1
解释：唯一一种可能的数组方案是 [1000]
```

**示例 2：**

```java
输入：s = "1000", k = 10
输出：0
解释：不存在任何数组方案满足所有整数都 >= 1 且 <= 10 同时输出结果为 s 。
```

**示例 3：**

```java
输入：s = "1317", k = 2000
输出：8
解释：可行的数组方案为 [1317]，[131,7]，[13,17]，[1,317]，[13,1,7]，[1,31,7]，[1,3,17]，[1,3,1,7]
```


**示例 4：**

```java
输入：s = "2020", k = 30
输出：1
解释：唯一可能的数组方案是 [20,20] 。 [2020] 不是可行的数组方案，原因是 2020 > 30 。 [2,020] 也不是可行的数组方案，因为 020 含有前导 0 。
```


**示例 5：**

```java
输入：s = "1234567890", k = 90
输出：34
```

**提示：**

- 1 <= s.length <= 10^5.
- s 只包含数字且不包含前导 0 。
- 1 <= k <= 10^9.

**解法一**

24th双周赛的T4，其实dp的状态转换还是像出来了，我想的是从左向右，但是细节没理清楚，瞄了一眼评论区，发现从右往左比较简单

```java
public int numberOfArrays(String s, int k) {
    int mod=1_000_000_000 + 7;
    int[] dp=new int[s.length()+1];        
    dp[0]=1;
    //dp[i]=dp[i-1]+dp[i-2]+...+dp[0];
    for (int i=1;i<=s.length();i++) {
        for (int j=i-1;j>=0 && i-j<=9;j--) {
            if(s.charAt(j)!='0' && valid(s,j,i,k)){
                dp[i]=(dp[i]+dp[j])%mod;
            }
        }
    }
    return dp[s.length()];
}

//10 0000 0000
public boolean valid(String s,int j,int i,int k){
    long value=Long.valueOf(s.substring(j,i));
    return value<=k && value>=1;
}
```
我开始以为时间复杂度是`O(N^2)`会T掉，后来写出来才发现其实时间复杂度其实是`O(10N)` ，并不难，还是题目写少了啊！！！

**解法二**

一点小优化

```java
public int numberOfArrays(String s, int k) {
    int mod=1_000_000_000 + 7;
    int[] dp=new int[s.length()+1];        
    dp[0]=1;
    for (int i=1;i<=s.length();i++) {
        for (int j=i-1;j>=0 && i-j<=9;j--) {
            if(s.charAt(j)=='0') continue;
            long value=Long.valueOf(s.substring(j,i));
            if(value>k) break; //这一步其实是对上面解法的优化
            dp[i]=(dp[i]+dp[j])%mod;
        }
    }
    return dp[s.length()];
}
```
## [300. 最长上升子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/)

给定一个无序的整数数组，找到其中最长上升子序列的长度。

**示例:**

```java
输入: [10,9,2,5,3,7,101,18]
输出: 4 
解释: 最长的上升子序列是 [2,3,7,101]，它的长度是 4。
```

**说明:**

- 可能会有多种最长上升子序列的组合，你只需要输出对应的长度即可。
- 你算法的时间复杂度应该为 O(n2) 。
- 进阶: 你能将算法的时间复杂度降低到 O(n log n) 吗?

**解法一**

LIS，这题好像在很多地方见到过了，还是要好好理解下，**dp[i]表示以当前元素结尾的最长上升子序列的长度**

```java
public static int lengthOfLIS(int[] nums) {
    if (nums==null||nums.length<=0) {
        return 0;
    }
    int[] dp=new int[nums.length];
    //Arrays.fill(dp,1);
    dp[0]=1;
    for (int i=1;i<nums.length;i++) {
        dp[i]=1; //初始化dp[i]=1
        for (int j=0;j<i;j++) {
            if(nums[i]>nums[j]){
                //遍历i之前的元素,找一个最大的dp[j]+1
                dp[i]=Math.max(dp[j]+1,dp[i]);
            }
        }
    }
    int res=-1;
    for (int i=0;i<dp.length;i++) {
        res=res>dp[i]?res:dp[i];
    }
    return res;
}
```

一开始题目就没看清楚就开始做，以为是连续的，心想这不是O(N)的么，为啥要我写O(N^2)的🤣 

![mark](http://static.imlgw.top/blog/20190824/86VLMN8VWcpI.png?imageslim)

灵魂画手，其实也是一道很简单的DP(然而我还是没想出来)，但是时间复杂度比较高O(N^2)，可以优化成O(NlogN)

**解法二**

`贪心+二分`  这个贪心还是有点骚的，核心思想：

**如果前面的数越小，后面接上一个随机数，就会有更大的可能性构成一个更长的“上升子序列”。**

而`tail[i]` 中存储长度为 `i + 1` 的最长递增子序列的最后一个元素，所以最后len就是整个数组的最长上升子序列，更多题解可以 参考[LeetCode题解](https://leetcode-cn.com/problems/longest-increasing-subsequence/solution/dong-tai-gui-hua-er-fen-cha-zhao-tan-xin-suan-fa-p/) 

```java
public static int lengthOfLIS(int[] nums) {
    int[] tail = new int[nums.length];
    int len = 0;
    for (int num : nums) {
        int index=binarySearch(tail,len,num); 
        //最难理解的就是这一步，直接替换了原数组里面的对应的元素
        //按照贪心的规则,将index位置的元素变小，或者添加到队尾
        tail[index] = num;
        if (index == len) {
            len++;
        }
    }
    return len;
}

//返回target可以插入的位置
private static int binarySearch(int[] nums, int len, int target) {
    int l=0,r=len-1;
    while(l<=r){
        int mid=l+(r-l)/2;
        if(target<nums[mid]){
            r=mid-1;
        }else if(target>nums[mid]){
            l=mid+1;
        }else{
            return mid;
        }
    }
    return l;
}
```

**解法二代码优化**

2020.3.20

```java
public static int lengthOfLIS(int[] nums) {
    int[] top = new int[nums.length];
    int len = 0;
    for (int num : nums) {
        //寻找左侧最小的堆顶
        int index=binarySearch(top,len,num);
        if (index == len) {
            len++;
        }
        top[index] = num;
    }
    return len;
}

//可以搜索
private static int binarySearch(int[] nums, int len, int target) {
    int left=0,right=len;
    while(left<right){
        int mid=left+(right-left)/2;
        if(nums[mid]<target){
            left=mid+1;
        }else {
            right=mid;
        }
    }
    return left;
}
```
首先是记住了这个套路，然后优化了二分的写法

![mark](http://static.imlgw.top/blog/20200320/zhcxcxmK4Amt.png?imageslim)


（copy自从liweiwei大佬 [题解](https://leetcode-cn.com/problems/longest-increasing-subsequence/solution/dong-tai-gui-hua-er-fen-cha-zhao-tan-xin-suan-fa-p/)）

## [354. 俄罗斯套娃信封问题](https://leetcode-cn.com/problems/russian-doll-envelopes/)

给定一些标记了宽度和高度的信封，宽度和高度以整数对形式 `(w, h)` 出现。当另一个信封的宽度和高度都比这个信封大的时候，这个信封就可以放进另一个信封里，如同俄罗斯套娃一样。

请计算最多能有多少个信封能组成一组“俄罗斯套娃”信封（即可以把一个信封放到另一个信封里面）。

**说明:**
不允许旋转信封。

**示例:**

```java
输入: envelopes = [[5,4],[6,4],[6,7],[2,3]]
输出: 3 
解释: 最多信封的个数为 3, 组合为: [2,3] => [5,4] => [6,7]。
```

**解法一**

```java
public int maxEnvelopes(int[][] envelopes) {
    int [] dp=new int[envelopes.length];
    //lamdba会比较慢
    Arrays.sort(envelopes,(a,b)->(a[0]-b[0])); //保证后面的不会被前面的装进去就行了
    int res=0;
    for (int i=0;i<envelopes.length;i++) {
        dp[i]=1;
        for (int j=0;j<i;j++) {
            if (envelopes[i][0]>envelopes[j][0] && envelopes[i][1]>envelopes[j][1]){
                dp[i]=Math.max(dp[j]+1,dp[i]);
            }
        }
        res=Math.max(res,dp[i]);
    }
    return res;
}
```
其实和上面的LIS是一样的，所以我放在了一起，差点bugfree，被空输入坑了一发，可惜这个解法并不是最优解，而且也不符合这题的困难tag，这题的最优解应该是利用上面的贪心+二分的做法，这里实在是能力时间都有限，没法去研究那种解法，暂且先用这个解法吧

**解法二**

2020.3.20 阿里笔试考了这一题，dp超时了。。。。

```java
public int maxEnvelopes(int[][] envelopes) {
    Arrays.sort(envelopes,(a,b)->a[0]!=b[0]?a[0]-b[0]:b[1]-a[1]); //这个排序很关键
    int N=envelopes.length;
    int[] top=new int[N];
    int len=0;
    for(int i=0;i<N;i++){
        int cur=envelopes[i][1];
        //二分搜索堆顶，找第一个小于当前牌的
        int index=binarySearch(top,cur,len);
        if(index==len){ //没找到合适的位置
            len++; //新建牌堆，牌堆++
        }
        top[index]=cur;
    }
    return len;
}

public int binarySearch(int[] top,int target,int len){
    int left=0,right=len;
    while(left<right){
        int mid=left+(right-left)/2;
        if(top[mid]<target){
            left=mid+1;
        }else{
            right=mid;
        }
    }
    return left;
}
```

第一步得排序很关键，如果只安装宽度排序，那么在宽度相同的时候，就有可能存在后面的把前面的装进去的情况，这明显是不符合题意得

## [53. 最大子序和](https://leetcode-cn.com/problems/maximum-subarray/)

给定一个整数数组 `nums` ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

**示例:**

```java
输入: [-2,1,-3,4,-1,2,1,-5,4],
输出: 6
解释: 连续子数组 [4,-1,2,1] 的和最大，为 6。
```

**进阶:**

如果你已经实现复杂度为 O(n) 的解法，尝试使用更为精妙的分治法求解。

**解法一**

动态规划递推的解法，没啥好说的，这题其实还有一道姊妹题，要求出最大子序列，不仅仅是最大值，[我放到了滑动窗口专题中]((http://imlgw.top/2019/07/20/leetcode-hua-dong-chuang-kou-tag/#239-%E6%BB%91%E5%8A%A8%E7%AA%97%E5%8F%A3%E6%9C%80%E5%A4%A7%E5%80%BC)) 

```java
//这次是直接在Web上写的
public int maxSubArray(int[] nums) {
    if(nums==null || nums.length==0){
        return 0;
    }
    int[] dp=new int[nums.length];
    //一开始没处理好Wa了一发
    int max=dp[0]=nums[0];
    for(int i=1;i<nums.length;i++){
        if(dp[i-1]<=0){
            dp[i]=nums[i];
        }else{
            dp[i]=dp[i-1]+nums[i];
        }
        max=Math.max(max,dp[i]);
    }
    return max;
}
```
**解法二**

根据二维改一维

```java
public int maxSubArray(int[] nums) {
    if(nums==null || nums.length==0){
        return 0;
    }
    int dp1=nums[0],dp2=0,max=nums[0];
    for (int i=1;i<nums.length;i++) {
        if (dp1<=0) {
            dp2=nums[i];
        }else{
            dp2=dp1+nums[i];
        }
        dp1=dp2;
        max=Math.max(max,dp2);
    }
    return max;
}
```

感觉一维的不用去理解含义，能根据二维改出来就ok了，重点是理解多维的

## [646. 最长数对链](https://leetcode-cn.com/problems/maximum-length-of-pair-chain/)

给出 `n` 个数对。 在每一个数对中，第一个数字总是比第二个数字小。

现在，我们定义一种跟随关系，当且仅当 `b < c` 时，数对`(c, d)` 才可以跟在 `(a, b)` 后面。我们用这种形式来构造一个数对链。

给定一个对数集合，找出能够形成的最长数对链的长度。你不需要用到所有的数对，你可以以任何顺序选择其中的一些数对来构造。

**示例 :**

```java
输入: [[1,2], [2,3], [3,4]]
输出: 2
解释: 最长的数对链是 [1,2] -> [3,4]
```

**注意：**

- 给出数对的个数在 [1, 1000] 范围内。

**解法一**

动态规划

```java
public int findLongestChain(int[][] pairs) {
    Arrays.sort(pairs,(a,b)->a[0]-b[0]);
    int[] dp=new int[pairs.length];
    int res=0;
    for (int i=0;i<pairs.length;i++) {
        dp[i]=1;
        for (int j=0;j<i;j++) {
            if (pairs[i][0]>pairs[j][1]) {
                dp[i]=Math.max(dp[i],dp[j]+1);
            }
        }
        res=res>dp[i]?res:dp[i];
    }
    return res;
}
```
注意这里没有要求顺序，所以先给他排个序，然后再进行DP，如果不排序很难dp，dp[i]表示当前数对能构成的最长链，dp过程和上面的最长上升子序列相同

**解法二**

贪心，也是最优解，同样先排序，不过是按照第二个元素来排序，核心思想

`每次都在列表中找第二个元素最小的数对依次组成链，最后得到的一定是最长的链`

```java
public int findLongestChain(int[][] pairs) {
    Arrays.sort(pairs,(a,b)->a[1]-b[1]);
    int res=1;
    int tail=pairs[0][1];
    for (int i=1;i<pairs.length;i++) {
        if (pairs[i][0]>tail) {
            res++;
            tail=pairs[i][1];
        }
    }
    return res;
}
```
**解法三**

也就是上面说的分治法，分治法在这里并不是最优解，时间复杂度`O(NlogN)`，但是思路还是很巧妙，首先也是讲数组一分为二，然后最大序列和就有三种情况	

1. 全在左边
2. 全在右边
3. 跨中点，两边都有

然后统计三个值得最大值，递归子过程求解，代码就不写了，后面想起来有时间再补

## [376. 摆动序列](https://leetcode-cn.com/problems/wiggle-subsequence/)

如果连续数字之间的差严格地在正数和负数之间交替，则数字序列称为摆动序列。第一个差（如果存在的话）可能是正数或负数。少于两个元素的序列也是摆动序列。

例如， `[1,7,4,9,2,5]` 是一个摆动序列，因为差值 `(6,-3,5,-7,3)` 是正负交替出现的。相反, `[1,4,7,2,5]` 和 `[1,7,4,5,5]` 不是摆动序列，第一个序列是因为它的前两个差值都是正数，第二个序列是因为它的最后一个差值为零。

给定一个整数序列，返回作为摆动序列的最长子序列的长度。 通过从原始序列中删除一些（也可以不删除）元素来获得子序列，剩下的元素保持其原始顺序。

**示例 1:**

```java
输入: [1,7,4,9,2,5]
输出: 6 
解释: 整个序列均为摆动序列。
```

**示例 2:**

```java
输入: [1,17,5,10,13,15,10,5,16,8]
输出: 7
解释: 这个序列包含几个长度为 7 摆动序列，其中一个可为[1,17,10,13,10,16,8]。
```

**示例 3:**

```java
输入: [1,2,3,4,5,6,7,8,9]
输出: 2
```

**进阶:**

- 你能否用 O(n) 时间复杂度完成此题?

**解法一**

O(N^2 )动态规划，dp[i]代表目前为止以 nums[i]结尾的最长的`上升摇摆序列`，down[i]代表目前为止以nums[i]结尾的最长的`下降摇摆序列`，后面过程就和上面的`LIS`的解法类似了

```java
public int wiggleMaxLength(int[] nums) {
    if (nums==null || nums.length<=0) {
        return 0;
    }
    int []up=new int[nums.length]; 
    int []down=new int[nums.length]; 
    for (int i=0;i<nums.length;i++) {
        up[i]=1; down[i]=1; //初始化
        for (int j=0;j<i;j++) {
            if(nums[i]>nums[j]){
                //找一个最大的下降沿
                up[i]=Math.max(up[i],down[j]+1);
            }else if(nums[i]<nums[j]){
                //找一个最大的上升沿
                down[i]=Math.max(down[i],up[j]+1);
            }
        }
    }
    return Math.max(up[nums.length-1],down[nums.length-1]);
}
```
**解法二**

线性动态规划，其实上面的循环找最大操作有点多余，如果`nums[i]>nums[i-1]` 那么以nums[i]为上升沿结尾（摆动上升）的最长摇摆序列 `up[i]=down[i-1]+1`

```java
public int wiggleMaxLength2(int[] nums) {
    if (nums==null || nums.length<=0) {
        return 0;
    }
    int []up=new int[nums.length];
    int []down=new int[nums.length];
    up[0]=down[0]=1;
    for (int i=1;i<nums.length;i++) {
        if(nums[i]>nums[i-1]){
            up[i]=down[i-1]+1;
            down[i]=down[i-1];
        }else if(nums[i]<nums[i-1]){
            down[i]=up[i-1]+1;
            up[i]=up[i-1];
        }else{
            //相等的时候别忘了继承前面的状态
            down[i]=down[i-1];
            up[i]=up[i-1];
        }
    }
    return Math.max(up[nums.length-1],down[nums.length-1]);
}
```

我感觉能写出O(1)的前提是先写出O(N)的，写出O(N)的之后再改成O(1)会比较简单

```java
public int wiggleMaxLength(int[] nums) {
    if (nums==null || nums.length<=0) {
        return 0;
    }
    int up=1;
    int down=1;
    for (int i=1;i<nums.length;i++) {
        if (nums[i]>nums[i-1]) {
            up=down+1;
        }else if (nums[i]>nums[i-1]){
            down=up+1;
        }//else就是不变
    }
    return Math.max(up,down);
}
```

## [978. 最长湍流子数组](https://leetcode-cn.com/problems/longest-turbulent-subarray/)

当 `A` 的子数组 `A[i], A[i+1], ..., A[j]` 满足下列条件时，我们称其为*湍流子数组*：

- 若 `i <= k < j`，当 `k` 为奇数时， `A[k] > A[k+1]`，且当 `k` 为偶数时，`A[k] < A[k+1]`；
- **或** 若 `i <= k < j`，当 `k` 为偶数时，`A[k] > A[k+1]` ，且当 `k` 为奇数时， `A[k] < A[k+1]`。

也就是说，如果比较符号在子数组中的每个相邻元素对之间翻转，则该子数组是湍流子数组。（上面的是废话）

返回 `A` 的最大湍流子数组的**长度**。

**示例 1：**

```java
输入：[9,4,2,10,7,8,8,1,9]
输出：5
解释：(A[1] > A[2] < A[3] > A[4] < A[5])
```

**示例 2：**

```java
输入：[4,8,12,16]
输出：2
```

**示例 3：**

```java
输入：[100]
输出：1
```

**提示：**

1. `1 <= A.length <= 40000`
2. `0 <= A[i] <= 10^9`

**解法一**

在滑窗tag里发现的，一直没往dp上想，看了评论区才发现😂，这题其实是上面[376. 摆动序列](#376-摆动序列)的弱化，这里是要求连续的，但是做法还是一样的

```java
//动态规划，从滑窗tag过来的...没往dp上想，看了评论区才想起来
public int maxTurbulenceSize(int[] A) {
    if(A==null || A.length<=0){
        return 0;
    }
    int up=1,down=1;
    int res=1;
    for (int i=1;i<A.length;i++) {
        if(A[i]>A[i-1]){
            up=down+1;
            down=1;
        }else if (A[i]<A[i-1]){
            down=up+1;
            up=1;
        }else{
            up=1;down=1;
        }
        res=Math.max(res,Math.max(up,down));
    }
    return res;
}
```

**解法二**

滑窗，也是一开始的做法，比较简单

```java
//优化if条件
public int maxTurbulenceSize(int[] A) {
    if(A==null || A.length<=0){
        return 0;
    }
    int left=0;
    int res=1;
    for(int right=left+1;right<A.length;right++){
        if(A[right]==A[right-1]){ //跳过相等的
            left=right;
            continue;
        }
        //优化了if条件，之前看别人题解学到的
        if(right>=2 && (A[right]>A[right-1])==(A[right-1]>A[right-2])){
            left=right-1; //left跳到中间值
        }
        res=Math.max(res,right-left+1);
    }
    return res;
}
```
## [96. 不同的二叉搜索树](https://leetcode-cn.com/problems/unique-binary-search-trees/)

Given *n*, how many structurally unique **BST's** (binary search trees) that store values 1 ... *n*?

**Example:**

```java
Input: 3
Output: 5
Explanation:
Given n = 3, there are a total of 5 unique BST's:

   1         3     3      2      1
    \       /     /      / \      \
     3     2     1      1   3      2
    /     /       \                 \
   2     1         2                 3
```

**解法一**

这个解法实际上来自 [95. 不同的二叉搜索树 II](https://leetcode-cn.com/problems/unique-binary-search-trees-ii/)

```java
public static int numTrees(int n) {
    return generateTree(1,n).size();
}

public static List<TreeNode> generateTree(int start,int end){
    List<TreeNode> res=new ArrayList<>();
    if (start>end) {
        res.add(null);
        return res;
    }
    for (int i=start;i<=end;i++) {
        List<TreeNode> left=generateTree(start,i-1);
        List<TreeNode> right=generateTree(i+1,end);
        for (TreeNode l:left) {
            for (TreeNode r:right) {
                TreeNode currentNode=new TreeNode(i);
                currentNode.left=l;
                currentNode.right=r;
                res.add(currentNode);
            }
        }
    }
    return res;
}
```
这种写法很明显过不了这题，时间复杂度太高了，但是这个方法可以求出所有的BST集合。

**解法二**

动态规划，`dp[i]`代表长度为 `i` 的递增序列构成的不同的BST的数量（序列从`1 到 i` 递增）

递推公式: `dp[i]=dp[i-1]* dp[n-i]`

```java
public int numTrees(int n) {
    if (n<=1) {
        return 1;
    }
    int []dp=new int[n+1];
    dp[0]=1;
    dp[1]=1;
    for (int i=2;i<=n;i++) { //控制序列长度 直到我们求得 n
        for (int j=1;j<=i;j++) { //控制根的选择
            //累加和，j为根节点 dp[j-1]代表左子树数量，dp[i-j]代表右子树数量
            dp[i]+=dp[j-1]*dp[i-j];
        }
    }
    return dp[n];
}
```

以 `i` 位置元素为根节点，可以得出可能的 BST数量为，`[1 ~ i-1]` 可以构成的BST数量 乘以  `[i+1，n]`可以构成的BST数量，也就是_左子树  *  右子树_  ，再推广到整个序列的BST总数，其实也就是 **以每个元素为根节点构成的BST的数量之和** ，需要理解的地方就是`dp[i]` 的含义，明确本题中`dp[i]` 其实**只和长度 `i`有关**而和内容无关，比如1，2，3和4，5，6所能构成的BST数量其实是相同的

**解法三**

卡塔兰数，递推式 `Gn+1 =2(2n+1)/ n+2  *  Gn`

```java
public int numTrees3(int n) {
    long C = 1;
    for(int i = 0; i < n; ++i){
        C = C * 2 * (2 * i + 1) /(i + 2);
    }
    return (int) C;
}
```


## [120. 三角形最小路径和](https://leetcode-cn.com/problems/triangle/)

给定一个三角形，找出自顶向下的最小路径和。每一步只能移动到下一行中相邻的结点上。

例如，给定三角形：

```java
[
     [2],
    [3,4],
   [6,5,7],
  [4,1,8,3]
]
```


自顶向下的最小路径和为 `11`（即，2 + 3 + 5 + 1 = 11）。

**说明：**

如果你可以只使用 O(n) 的额外空间（n 为三角形的总行数）来解决这个问题，那么你的算法会很加分

**解法一**

二维dp，emmm，一开始写成贪心了。。贪心明显是错的

```java
//dp做法 O(N^2)空间
public int minimumTotal(List<List<Integer>> triangle) {
    int rows=triangle.size();
    //dp[i][j]代表的是每i层第j个元素到最底层的最短距离
    int[][] dp=new int[rows+1][rows+1]; //row+1
    /*for (int i=0;i<triangle.get(rows-1).size();i++) {
        //给最后一行赋初始值
        dp[rows-1][i]=triangle.get(rows-1).get(i);
    }
    for (int i=rows-2;i>=0;i--) {
        for (int j=0;j<triangle.get(i).size();j++) {
            //核心递推式 dp[i][j]=min(dp[i+1][j],dp[i+1][j+1])+triangle(i)(j)
            dp[i][j]=Math.min(dp[i+1][j],dp[i+1][j+1])+triangle.get(i).get(j);
        }
    }*/
    //直接从最后一行开始,这样就不用手动给最后一行赋初始值了
    for (int i=rows-1;i>=0;i--) {
        for (int j=0;j<triangle.get(i).size();j++) {
            //核心递推式 dp[i][j]=min(dp[i+1][j],dp[i+1][j+1])+triangle(i)(j)
            dp[i][j]=Math.min(dp[i+1][j],dp[i+1][j+1])+triangle.get(i).get(j);
        }
    }
    return dp[0][0];
}
```
**解法二**

记忆化递归，也很快

```java
//记忆化递归
public int minimumTotal2(List<List<Integer>> triangle) {
    //用Integer比较好,方便判空
    Integer [][] cache=new Integer[triangle.size()][triangle.size()];
    return minimumTotal(triangle,0,0,cache);
}

public int minimumTotal(List<List<Integer>> triangle,int cen,int index,Integer[][]cache) {
    if (cache[cen][index]!=null) {
        return cache[cen][index];
    }
    if (cen==triangle.size()-1) {
        return triangle.get(cen).get(index);
    }
    int left=minimumTotal(triangle,cen+1,index,cache);
    int right=minimumTotal(triangle,cen+1,index+1,cache);
    return cache[cen][index]=triangle.get(cen).get(index)+(left<right?left:right);
}
```
**解法三**

```java
//dp O(N)空间
public int minimumTotal(List<List<Integer>> triangle) {
    int rows=triangle.size();
    //dp[i]代表的是每一层第i个元素到最底层的最短距离
    //上面二维dp实际上也只和下一层的状态有关,所以我们可以重复的使用这个数组保存每一层的状态
    int[] dp=new int[rows+1];
    for (int i=rows-1;i>=0;i--) {
        for (int j=0;j<triangle.get(i).size();j++) {
            //到这里其实Math.min()里面的都是上一次循环的结果
            //也就是下一层的,对应当前j位置左右两个相邻节点的最小距离
            dp[j]=Math.min(dp[j+1],dp[j])+triangle.get(i).get(j);
        }
    }
    return dp[0];
}
```
O(N)空间复杂度，只和下面那一层的每一个元素的dp[i]有关，所以可以直接改成一维dp

## [121. 买卖股票的最佳时机](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/)

给定一个数组，它的第 i 个元素是一支给定股票第 i 天的价格。

如果你**最多只允许完成一笔交易**（即买入和卖出一支股票），设计一个算法来计算你所能获取的最大利润。

注意你不能在买入股票前卖出股票。

**示例 1:**

```java
输入: [7,1,5,3,6,4]
输出: 5
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5  ，注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格。
```

**示例 2:**

```java
输入: [7,6,4,3,1]
输出: 0
解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。
```

**解法一**

非动态规划的思路，还是挺简单的，求个最大的差值就ok，因为只能买一次

```java
//非递归的思路
public int maxProfit(int[] prices) {
    if (prices==null || prices.length<=0) {
        return 0;
    }
    int min=Integer.MAX_VALUE,max=0;
    for (int i=0;i<prices.length;i++) {
        //当天价格减去 *之前* 价格最低的买入时机
        max=Math.max(max,prices[i]-min);
        //统计价格最低的买入时机
        min=Math.min(min,prices[i]);
    }
    return max;
}
```
**解法二**

既然是动态规划的题，自然得写个动态规划的版本

```java
public int maxProfit(int[] prices) {
    if (prices==null || prices.length<=0) {
        return 0;
    }
    int[][] dp=new int[prices.length][2];
    dp[0][0]=0;
    dp[0][1]= -prices[0];
    for (int i=1;i<prices.length;i++) {
        dp[i][0]=Math.max(dp[i-1][0],dp[i-1][1]+[i]);
        dp[i][1]=Math.max(dp[i-1][1],-prices[i]);
    }
    return dp[prices.length-1][0];
}
```
其实改成一维的就和上面的一模一样了

## [122. 买卖股票的最佳时机 II](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii/)

给定一个数组，它的第 i 个元素是一支给定股票第 i 天的价格。

设计一个算法来计算你所能获取的最大利润。**你可以尽可能地完成更多的交易**（多次买卖一支股票）。

**注意：**你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

**示例 1:**

```java
输入: [7,1,5,3,6,4]
输出: 7
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 3 天（股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。
随后，在第 4 天（股票价格 = 3）的时候买入，在第 5 天（股票价格 = 6）的时候卖出, 这笔交易所能获得利润 = 6-3 = 3 。
```

**示例 2:**

```java
输入: [1,2,3,4,5]
输出: 4
解释: 在第 1 天（股票价格 = 1）的时候买入，在第 5 天 （股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。
注意你不能在第 1 天和第 2 天接连购买股票，之后再将它们卖出。为这样属于同时参与了多笔交易，你必须在再次购买前出售掉之前的股票。
```

**示例 3:**

```java
输入: [7,6,4,3,1]
输出: 0
解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。
```

**解法一**

正常的思路（非动态规划），注意审题，上面的只能买一次，这里是不限制次数的

```java
public int maxProfit(int[] prices) {
    if (prices==null || prices.length<=0) {
        return 0;
    }
    int ans=0;
    for (int i=1;i<prices.length;i++) {
        if (prices[i]>prices[i-1]) {
            ans+=prices[i]-prices[i-1];
        }
    }
    return ans;
}
```
这里的代码还是很有迷惑性的，实际上分了三种情况，有两种合并了

>**单独交易日**：明天比今天价格高，今天买，明天卖
>
>**连续上涨交易日**：开始的第一天买 , 涨停的最后一天卖最有利，或者也可以转换成说`除了第一天，每天都卖了又买`，也就是`pn-p1 = (p2-p1)+(p3-p2)+....(pn-pn-1)`  很自然的就转换成了单独交易日的情况
>**连续下降交易日**：不买卖最有利

如果还是不明白可以看看官方的图

![mark](http://static.imlgw.top/blog/20191017/Wpsvs09EmnmF.png?imageslim)

A->D是一段连续上涨日，`D-A的差值` 就是A->D各个相邻节点差值之和

**解法二**

我们的主角，动态规划

```java
//dp的思路
public int maxProfit(int[] prices) {
    if (prices==null || prices.length<=0) {
        return 0;
    }
    int[][] dp=new int[prices.length][2];
    dp[0][0]=0;
    dp[0][1]=-prices[0];
    for (int i=1;i<prices.length;i++) {
        dp[i][0]=Math.max(dp[i-1][0],dp[i-1][1]+prices[i]);
        dp[i][1]=Math.max(dp[i-1][1],dp[i-1][0]-prices[i]);
    }
    return dp[prices.length-1][0];
}
```
第一眼看可能觉得看不懂，熟悉了就知道了，其实都是套路，股票题特有的套路，先考虑题目有几种状态，这题说了不限制次数，那么就不考虑次数的问题，然后剩下的状态就是：那一天，和是否持有股票，所以我们用一个二维的数组来表述这两种状态 比如`dp[i][0]` 则代表第i天不持有股票的最大收益，`dp[i][1]`则代表第i天持有股票的最大收益，然后我们再看看状态转换的过程

![mark](http://static.imlgw.top/blog/20191017/J7tfvKDvqC2b.png?imageslim)

其实无非就是这两个状态之间的转换，然后根据这个状态转换的过程就可以很轻易的写出我们的状态转换方程

```java
dp[i][0]=Math.max(dp[i-1][0],dp[i-1][1]+prices[i]);
dp[i][1]=Math.max(dp[i-1][1],dp[i-1][0]-prices[i]);
```
不过这里其实也很容易就可以改成一唯的dp，空间复杂度变为O(1)，因为当天的最大收益只和前一天的最大收益相关，两个变量就是足够表示

```java
public int maxProfit(int[] prices) {
    if (prices==null || prices.length<=0) {
        return 0;
    }
    int hlod=-prices[0];
    int empty=0;
    for (int i=1;i<prices.length;i++) {
        //保存前一天的状态
        int temp=empty;
        empty=Math.max(empty,hlod+prices[i]);
        hlod=Math.max(hlod,temp-prices[i]);
    }
    return empty;
}
```
这里我感觉leetcode的case有问题，我开始没有缓存前一天的empty值，直接将empty带到hold去了，结果还过了。。。不过我懒得去想case了，反正提交了leecode也不会理我😅

## [123. 买卖股票的最佳时机 III](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iii/)

给定一个数组，它的第 i 个元素是一支给定的股票在第 i 天的价格。

设计一个算法来计算你所能获取的最大利润。你最多可以完成 两笔 交易。

**注意:** 你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

**示例 1:**

```java
输入: [3,3,5,0,0,3,1,4]
输出: 6
解释: 在第 4 天（股票价格 = 0）的时候买入，在第 6 天（股票价格 = 3）的时候卖出，这笔交易所能获得利润 = 3-0 = 3 。
     随后，在第 7 天（股票价格 = 1）的时候买入，在第 8 天 （股票价格 = 4）的时候卖出，这笔交易所能获得利润 = 4-1 = 3 。
```


**示例 2:**

```java
输入: [1,2,3,4,5]
输出: 4
解释: 在第 1 天（股票价格 = 1）的时候买入，在第 5 天 （股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。   
     注意你不能在第 1 天和第 2 天接连购买股票，之后再将它们卖出。   
     因为这样属于同时参与了多笔交易，你必须在再次购买前出售掉之前的股票。
```

**示例 3:**

```java
输入: [7,6,4,3,1] 
输出: 0 
解释: 在这个情况下, 没有交易完成, 所以最大利润为 0
```

**解法一**

二维动态规划，定义出每个状态，然后转换就可以了

```java
public int maxProfit(int[] prices){
    if (prices==null || prices.length<=0) {
        return 0;
    }
    //状态定义：
    //j=0 什么都不做
    //j=1 第一次买入
    //j=2 第一次卖出
    //j=3 第二次买入
    //j=4 第二次卖出
    int[][] dp=new int[prices.length][5];
    //这样会溢出
    //Arrays.fill(dp[0],Integer.MIN_VALUE);
    //这样可以过,但是感觉还是判断一下好
    //Arrays.fill(dp[0],-0x3f3f3f3f);
    int INF=Integer.MIN_VALUE,n=prices.length;
    Arrays.fill(dp[0],INF); //不可达状态
    dp[0][0]=0;
    dp[0][1]=-prices[0];
    for(int i=1;i<n;i++){
        dp[i][0]=0;
        dp[i][1]=Math.max(-prices[i],dp[i-1][1]);
        dp[i][2]=Math.max(dp[i-1][1]+prices[i],dp[i-1][2]);
        dp[i][3]=Math.max(dp[i-1][2]!=INF?dp[i-1][2]-prices[i]:INF,dp[i-1][3]);
        dp[i][4]=Math.max(dp[i-1][3]!=INF?dp[i-1][3]+prices[i]:INF,dp[i-1][4]);
    }
    return Math.max(Math.max(dp[n-1][0],dp[n-1][2]),dp[n-1][4]);
}
```
> 因为题目的数据不是特别大，没有超过`10^9` 直接初始化为 `-0x3f3f3f3f` 就不用判断前面的状态是否可达了，

**解法二**

优化成一维的

```java
public int maxProfit(int[] prices){
    if (prices==null || prices.length<=0) {
        return 0;
    }
    int[] dp=new int[5];
    int n=prices.length;
    Arrays.fill(dp,-0x3f3f3f3f);
    dp[0]=0;
    dp[1]=-prices[0];
    for(int i=1;i<n;i++){
        //逆序递推避免覆盖(其实正着写也是对的,这题相邻的状态不会同时更新,但是为了规范最好还是逆序写)
        dp[4]=Math.max(dp[3]+prices[i],dp[4]);
        dp[3]=Math.max(dp[2]-prices[i],dp[3]);
        dp[2]=Math.max(dp[1]+prices[i],dp[2]);
        dp[1]=Math.max(-prices[i],dp[1]);
        dp[0]=0;
    }
    return Math.max(Math.max(dp[0],dp[2]),dp[4]);
}
```
## [188. 买卖股票的最佳时机 IV](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iv/)

给定一个数组，它的第 i 个元素是一支给定的股票在第 i 天的价格。

设计一个算法来计算你所能获取的最大利润。你最多可以完成 k 笔交易。

注意: 你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

**示例 1:**

```java
输入: [2,4,1], k = 2
输出: 2
解释: 在第 1 天 (股票价格 = 2) 的时候买入，在第 2 天 (股票价格 = 4) 的时候卖出，这笔交易所能获得利润 = 4-2 = 2 。
```


**示例 2:**

```java
输入: [3,2,6,5,0,3], k = 2
输出: 7
解释: 在第 2 天 (股票价格 = 2) 的时候买入，在第 3 天 (股票价格 = 6) 的时候卖出, 这笔交易所能获得利润 = 6-2 = 4 。
     随后，在第 5 天 (股票价格 = 0) 的时候买入，在第 6 天 (股票价格 = 3) 的时候卖出, 这笔交易所能获得利润 = 3-0 = 3 。
```

**解法一**

这题其实就是求上面一题的通解，上面的代码规律已经非常明显了，`j`为奇数的时候买入，偶数的时候卖出，我们统计最后偶数的最大值就可以

```java
public int maxProfit(int k, int[] prices) {
    if (prices==null || prices.length<=0 || k<=0) {
        return 0;
    }
    if(k>prices.length/2){
        return maxProfit(prices);
    }
    //k次交易,2*k+1种状态
    int[] dp=new int[2*k+1];
    int n=prices.length;
    int res=0;
    Arrays.fill(dp,-0x3f3f3f3f);
    dp[0]=0;
    dp[1]=-prices[0];
    for(int i=1;i<n;i++){
        //注意倒推
        for(int j=2*k;j>0;j--){
            if((j&1)==1){
                dp[j]=Math.max(dp[j-1]-prices[i],dp[j]);
            }else{
                dp[j]=Math.max(dp[j-1]+prices[i],dp[j]);
                res=Math.max(dp[j],res);
            }
        }
    }
    return res;
}

public int maxProfit(int[] prices) {
    if (prices==null || prices.length<=0) {
        return 0;
    }
    int ans=0;
    for (int i=1;i<prices.length;i++) {
        if (prices[i]>prices[i-1]) {
            ans+=prices[i]-prices[i-1];
        }
    }
    return ans;
}
```
**解法二**

正常的DP通解

```java
//常规2维解法
public int maxProfit(int k, int[] prices) {
    if (prices==null || prices.length<=0 || k<=0) {
        return 0;
    }
    if(k>prices.length/2){
        return maxProfit(prices);
    }
    //第k次交易,持股/不持股
    int[][] dp=new int[k+1][2];
    int n=prices.length;
    int res=0;
    int INF=-0x3f3f3f3f;
    for(int i=0;i<=k;i++){
        Arrays.fill(dp[i],INF);
    }
    //其实这题难搞的就是对于初始状态的init
    //第一天没有交易和第一天有一次交易的初始值
    dp[0][0]=0;dp[0][1]=INF;
    dp[1][0]=0;dp[1][1]=-prices[0];
    for(int i=1;i<n;i++){
        //注意倒推
        for(int j=k;j>0;j--){
            //这里将买入和卖出作为一个状态,所以这里买入新股票，肯定就是属于下一次交易了
            //所以这里的j需要减一，代表上一次交易卖出时候的收益
            dp[j][0]=Math.max(dp[j][1]+prices[i],dp[j][0]);
            dp[j][1]=Math.max(dp[j-1][0]-prices[i],dp[j][1]);
            res=Math.max(dp[j][0],res);
        }
    }
    return res;
}

public int maxProfit(int[] prices) {
    if (prices==null || prices.length<=0) {
        return 0;
    }
    int ans=0;
    for (int i=1;i<prices.length;i++) {
        if (prices[i]>prices[i-1]) {
            ans+=prices[i]-prices[i-1];
        }
    }
    return ans;
}
```
## [714. 买卖股票的最佳时机含手续费](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)

给定一个整数数组 `prices`，其中第 i 个元素代表了第 i 天的股票价格 ，非负整数 fee 代表了交易股票的手续费用

你可以无限次地完成交易，但是你每次交易都需要付手续费。如果你已经购买了一个股票，在卖出它之前你就不能再继续购买股票了。

返回获得利润的最大值。

**示例 1:**

```java
输入: prices = [1, 3, 2, 8, 4, 9], fee = 2
输出: 8
解释: 能够达到的最大利润:  
在此处买入 prices[0] = 1
在此处卖出 prices[3] = 8
在此处买入 prices[4] = 4
在此处卖出 prices[5] = 9
总利润: ((8 - 1) - 2) + ((9 - 4) - 2) = 8.
```

**注意:**

- `0 < prices.length <= 50000.`
- `0 < prices[i] < 50000.`
- `0 <= fee < 50000.`

**解法一**

我起了，一枪秒了，有什么好说的？

```java
public int maxProfit(int[] prices, int fee) {
    if (prices==null || prices.length<=0) {
        return 0;
    }
    int[][] dp=new int[prices.length][2];
    dp[0][0]=0;
    dp[0][1]=-prices[0];
    for (int i=1;i<prices.length;i++) {
        dp[i][0]=Math.max(dp[i-1][0],dp[i-1][1]+prices[i]-fee);
        dp[i][1]=Math.max(dp[i-1][1],dp[i-1][0]-prices[i]);
    }
    return dp[prices.length-1][0];
}
```
改为O(1)空间

```java
//改为O(1)
public int maxProfit(int[] prices, int fee) {
    if (prices==null || prices.length<=0) {
        return 0;
    }
    int empty=0;
    int hold=-prices[0];
    for (int i=1;i<prices.length;i++) {
        int temp=empty;
        empty=Math.max(empty,hold+prices[i]-fee);
        hold=Math.max(hold,temp-prices[i]);
    }
    return empty;
}
```
## [309. 最佳买卖股票时机含冷冻期](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/)

给定一个整数数组，其中第 i 个元素代表了第 i 天的股票价格 

设计一个算法计算出最大利润。在满足以下约束条件下，你可以尽可能地完成更多的交易（多次买卖一支股票）:

- 你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）
- 卖出股票后，你无法在第二天买入股票 (即冷冻期为 1 天)

**示例:**

```java
输入: [1,2,3,0,2]
输出: 3 
解释: 对应的交易状态为: [买入, 卖出, 冷冻期, 买入, 卖出]
```

**解法一**

依然是上面题的升级版，相比上一题又多了个限制条件，加了cd，买卖一次后有一天的cd，当天不能再买

```java
public int maxProfit(int[] prices) {
    if (prices==null || prices.length<=0) {
        return 0;
    }
    //dp[i][0] 代表第i天不持有股票的最大利润
    //dp[i][1] 代表第i天持有股票的最大利润
    int[][] dp=new int[prices.length][2];
    dp[0][0]=0;
    dp[0][1]=-prices[0];
    for (int i=1;i<prices.length;i++) {
        //这些玩意想起还是蛮打脑阔的
        dp[i][0]=Math.max(dp[i-1][1]+prices[i],dp[i-1][0]);
        //dp[i][1]=Math.max(i<2?-prices[i]:dp[i-2][0]-prices[i],dp[i-1][1]);
        //这里i<2就是第一次循环，i=1,也就是第二天持有股票，所以肯定是首次买股票，直接初始化为 -prices[1]
        //昨天有股票，或者昨天冷冻前天卖出
        dp[i][1]=Math.max(i<2?-prices[1]:dp[i-2][0]-prices[i],dp[i-1][1]);
    }
    return dp[prices.length-1][0];
}
```
评论区很多的解法都是用了三个状态数组，相比这种多了一个 冷冻期的最大值，感觉挺迷惑的。。。虽然结果是对的，但是总时感觉别扭，所以我们还是按照上面的思路，用两个状态表示，状态转换图和上面是一样的

![mark](http://static.imlgw.top/blog/20191017/J7tfvKDvqC2b.png?imageslim)

当天持有股票的最大收益，就是你当天买股票，或者rest啥也不干延续前一天的状态，两者的最大值

这里和上面不同的就是从`不持有股票` 到`持有股票`（也就是买股票）的情况， 因为有冷冻期的存在，**所以你当天买股票的最大值不再是昨天不持有股票的最大值，而是前天不持有股票的最大值**

**解法三**

改为空间复杂度O(1)的

```java
public int maxProfit(int[] prices) {
    if (prices==null || prices.length<=0) {
        return 0;
    }
    int hold=-prices[0];
    int empty=0;
    //前一天买出的最大收益
    int prePre=0;
    for (int i=1;i<prices.length;i++) {
        int temp=empty;
        empty=Math.max(hold+prices[i],empty);
        hold=Math.max(i<2?-prices[1]:prePre-prices[i],hold);
        //到这里pre就变成了前一天
        prePre=temp;
    }
    return empty;
}
```
> 看了下评论区还是蛮多争议的，感觉这里根据dp[i-2]转换有点不好理解，然后翻了翻评论区看见了大神的回答
>
> `dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i])`
>
> `dp[i][1] = max(dp[i-1][1], dp[i-1][0] - prices[i])`
>
> 添加 “冷冻期” 条件后，买入的前一天是不能卖的，而`dp[i-1][0] - prices[i]`并不能确保 `i-1天不卖出`   
>
> 所以进一步拆解`dp[i-1][0]`， 套用上面的转换方程可以得到
>
> `dp[i-1][0] = max(dp[i-2][0], dp[i-2][1] + prices[i-1])` 
>
> 再带入原式子，则可以得到
>
> `dp[i][1] = max(dp[i-1][1], max(dp[i-2][0], dp[i-2][1] + prices[i-1]) - prices[i])`
>
> 其中`dp[i-2][1] + prices[i-1]` 代表着 `“i-1发生了卖出行为”`，有违题意，应予删除，得下式，最终和上面的状态转换方程一致
>
> `dp[i][1] = max(dp[i-1][1], max(dp[i-2][0]) - prices[i])`
>
> 这样一证明就很清楚了



## [1269. 停在原地的方案数](https://leetcode-cn.com/problems/number-of-ways-to-stay-in-the-same-place-after-some-steps/)

有一个长度为 arrLen 的数组，开始有一个指针在索引 0 处。

每一步操作中，你可以将指针向左或向右移动 1 步，或者停在原地（指针不能被移动到数组范围外）。

给你两个整数 steps 和 arrLen ，请你计算并返回：在恰好执行 steps 次操作以后，指针仍然指向索引 0 处的方案数。

由于答案可能会很大，请返回方案数 模 10^9 + 7 后的结果。

**示例 1：**

```java
输入：steps = 3, arrLen = 2
输出：4
解释：3 步后，总共有 4 种不同的方法可以停在索引 0 处。
向右，向左，不动
不动，向右，向左
向右，不动，向左
不动，不动，不动
```

**示例  2：**

```java
输入：steps = 2, arrLen = 4
输出：2
解释：2 步后，总共有 2 种不同的方法可以停在索引 0 处。
向右，向左
不动，不动
```

**示例 3：**

```java
输入：steps = 4, arrLen = 2
输出：8
```

**提示：**

- `1 <= steps <= 500`
- `1 <= arrLen <= 10^6`

**解法一**

11.24 的周赛最后一题，很可惜没做出来。。。要是把这题放在第三题我可能就做出来了。。。。还是菜啊

```java
public int numWays(int steps, int arrLen) {
    int mod=1_000_000_007;
    long[][] dp=new long[steps+1][steps+1];
    dp[0][0]=1; //dp[i][j]的含义是i步之后处于j位置的方案数
    for (int i=1;i<=steps;i++) {
        //i步能达到的最大距离就是i,所以我们这里取一个最小值
        int k=Math.min(i,arrLen-1);
        for (int j=0;j<=k;j++) {
            if (j==0) {
                dp[i][j]=(dp[i-1][j+1]+dp[i-1][j])%mod;
            }else if (j<k) {
                dp[i][j]=(dp[i-1][j-1]+dp[i-1][j+1]+dp[i-1][j])%mod;
            }else{ //大于k就只能是从右边过来的
                dp[i][j]=(dp[i-1][j-1]+dp[i-1][j])%mod;
            }
        }
    }
    return (int)dp[steps][0];
}
```
**解法二**

我比较喜欢的记忆化递归😁，10ms甚至比上面还快一点

```java
int mod=1_000_000_007;

//cache
Long[][] cache;

public int numWays(int steps, int arrLen) {
    int maxIndex=Math.min(steps,arrLen-1);
    cache=new Long[steps+1][maxIndex+1];
    return (int)dfs(steps,0,maxIndex);
}

public long dfs(int steps, int index,int maxIndex) {
    if (steps==0) {
        return index==0?1:0;
    }
    if (index<0 || index > maxIndex )  {
        return 0;
    }

    if (cache[steps][index]!=null) {
        return cache[steps][index];
    }
    //steps一直在递减,steps最多走到steps位置
    //为了节约时间可以在这里优化下
    maxIndex=Math.min(steps,maxIndex);
    //不动
    long res=dfs(steps-1,index,maxIndex);
    //向左
    res+=dfs(steps-1,index-1,maxIndex);
    //向右
    res+=dfs(steps-1,index+1,maxIndex);
    return cache[steps][index]=res%mod;
}
```

## [152. 乘积最大子序列](https://leetcode-cn.com/problems/maximum-product-subarray/)

给定一个整数数组 nums ，找出一个序列中乘积最大的连续子序列（该序列至少包含一个数）。

**示例 1:**

```java
输入: [2,3,-2,4]
输出: 6
解释: 子数组 [2,3] 有最大乘积 6。
```


**示例 2:**

```java
输入: [-2,0,-1]
输出: 0
解释: 结果不能为 2, 因为 [-2,-1] 不是子数组。
```

**解法一**

自己最开始自己写出来的脑瘫dp

```java
public static int maxProduct(int[] nums) {
    int[] dp1=new int[nums.length];
    int[] dp2=new int[nums.length];
    //最大值
    dp1[0]=nums[0]>=0?nums[0]:0;
    //最小值
    dp2[0]=nums[0]>=0?0:nums[0];
    int max=nums[0]>0?dp1[0]:dp2[0];
    for(int i=1;i<nums.length;i++){
        if (nums[i]>0) {
            if (nums[i-1]<0) {
                dp1[i]=dp1[i-1]!=0?nums[i]*dp1[i-1]:nums[i];
                dp2[i]=dp2[i-1]*nums[i];
            }else if(nums[i-1]>0){
                dp1[i]=dp1[i-1]*nums[i];
                dp2[i]=dp2[i-1]!=0?nums[i]*dp2[i-1]:0;
            }else{
                dp1[i]=nums[i];
                dp2[i]=0;
            }
        }else if (nums[i]<0) {
            if (nums[i-1]<0) {
                dp1[i]=dp2[i-1]*nums[i];
                dp2[i]=dp1[i-1]!=0?dp1[i-1]*nums[i]:nums[i];
            }else if(nums[i-1]>0){
                dp1[i]=dp1[i-1]!=0?nums[i]*dp2[i-1]:0;
                dp2[i]=dp1[i-1]*nums[i];
            }else{
                dp1[i]=0;
                dp2[i]=nums[i];
            }
        }else{
            dp1[i]=0;
            dp2[i]=0;
        }
        //System.out.println(dp1[i]+","+dp2[i]);
        max=Math.max(max,dp1[i]);
    }
    return max;
}

/**
    //     -2  2  3   -4        
    //dp1:  0  2  6   48        
    //dp2: -2 -4 -12 -24 
    //
    // 2 -5   -2  -4   3
    // 2  0   20   8   24
    // 0 -10  -2  -80 -240
*/
```
不想解释太多，虽然过了，但是确实有点蠢

**解法二**

`max[i],min[i]` 代表的就是以`nums[i]` 结尾的**最大**乘积和**最小**乘积

```java
public static int maxProduct(int[] nums) {
    int[] min=new int[nums.length];
    int[] max=new int[nums.length];
    int res=max[0]=min[0]=nums[0];
    for (int i=1;i<nums.length;i++) {
        max[i]=Math.max(nums[i]*min[i-1],Math.max(nums[i],nums[i]*max[i-1]));
        min[i]=Math.min(nums[i]*min[i-1],Math.min(nums[i],nums[i]*max[i-1]));
        res=Math.max(res,max[i]);
    }
    return res;
}
```
为什么要记录最小乘积相信不需要我多说了吧，其实只要想清楚一点就ok，每个位置的`max[i]`其实只有三种来源（纸上写写就明白了）

- `nums[i] >= 0` 并且`max[i-1] > 0`，`max[i] = max[i-1] * nums[i]`
-  `nums[i] >= 0` 并且`max[i-1] < 0`，此时如果和前边的数累乘的话，会变成负数，所以`max[i] = nums[i]`
-  `nums[i] < 0`，如果是负数就不应该再和前面`max[i]`相乘，而是考虑`min[i]`
  - `当min[i-1] < 0，max[i] = min[i-1] * nums[i]`
  - `当min[i-1] >= 0，max[i] = nums[i]`

 然后我们直接求这三种情况的最大值就ok了，不用考虑那些分支，我上面的第一种解法其实就是考虑了所有分支，结果才写出了那样的dp， `min[i]` 的过程和上面一样，就不赘述

## [221. 最大正方形](https://leetcode-cn.com/problems/maximal-square/)

在一个由 0 和 1 组成的二维矩阵内，找到只包含 1 的最大正方形，并返回其面积。

**示例:**

```java
输入: 

1 0 1 0 0
1 0 1 1 1
1 1 1 1 1
1 0 0 1 0

输出: 4
```

**解法一**

递推公式：`dp[i][j]=1+min(dp[i-1][j-1],dp[i-1][j],dp[i][j-1])`  有的类似短板理论，取决于最小的哪一个正方形

```java
public int maximalSquare(char[][] matrix) {
    if (matrix==null || matrix.length<=0) {
        return 0;
    }
    int m=matrix.length;
    int n=matrix[0].length;
    //这里+1处理边界
    int[][] dp=new int[m+1][n+1];
    int max=0;
    for (int i=1;i<=m;i++) {
        for (int j=1;j<=n;j++) {
            if (matrix[i-1][j-1]=='1') {
                dp[i][j]=1+Math.min(dp[i-1][j-1],Math.min(dp[i][j-1],dp[i-1][j]));
                max=Math.max(max,dp[i][j]);
            }
        }
    }
    return max*max;
}
```

这题还有个需要注意的地方就是边界的处理，这里实现的代码中`dp[i][j]` 其实表示的是以 `matrix[i-1][j-1]` 作为右下角结尾的正方形的最大边长，这样的话就不用考虑上下两条边的边界case，相当于在dp数组上下边界之外又加了一层 0

> 到这里我也大致明白了一些动态规划的题目的递推写法为都是从1开始了，比如上面的编辑距离

## [1277. 统计全为 1 的正方形子矩阵](https://leetcode-cn.com/problems/count-square-submatrices-with-all-ones/)

给你一个 `m * n` 的矩阵，矩阵中的元素不是 `0` 就是 `1`，请你统计并返回其中完全由 `1` 组成的 **正方形** 子矩阵的个数。

示例 1：

```java
输入：matrix =
[
  [0,1,1,1],
  [1,1,1,1],
  [0,1,1,1]
]
输出：15
解释： 
边长为 1 的正方形有 10 个。
边长为 2 的正方形有 4 个。
边长为 3 的正方形有 1 个。
正方形的总数 = 10 + 4 + 1 = 15.
```

**示例 2：**

```java
输入：matrix = 
[
  [1,0,1],
  [1,1,0],
  [1,1,0]
]
输出：7
解释：
边长为 1 的正方形有 6 个。 
边长为 2 的正方形有 1 个。
正方形的总数 = 6 + 1 = 7.
```

**提示：**

- `1 <= arr.length <= 300`
- `1 <= arr[0].length <= 300`
- `0 <= arr[i][j] <= 1`

**解法一**

12.1的周赛题目，其实先做的上面的最大正方形，然后发现这一题和上面的一样。

```java
public int countSquares(int[][] matrix) {
    if (matrix==null || matrix.length<=0) {
        return 0;
    }
    int m=matrix.length;
    int n=matrix[0].length;
    int [][]dp=new int[m+1][n+1];
    int res=0;
    for (int i=1;i<=m;i++) {
        for (int j=1;j<=n;j++) {
            if (matrix[i-1][j-1]==1) {
                dp[i][j]=Math.min(dp[i-1][j-1],Math.min(dp[i][j-1],dp[i-1][j]))+1;
                res+=dp[i][j];
            }
        }
    }
    return res;
}
```

一摸一样，理解一点就ok，**以`matrix[i][j]` 为右下角的最大正方形的边长，就是以这个点为右下角的正方形的数量！！！**

## [44. 通配符匹配](https://leetcode-cn.com/problems/wildcard-matching/)

给定一个字符串 (s) 和一个字符模式 (p) ，实现一个支持 '?' 和 '*' 的通配符匹配。

```java
'?' 可以匹配任何单个字符。
'*' 可以匹配任意字符串（包括空字符串）。
```

两个字符串完全匹配才算匹配成功。

**说明:**

- s 可能为空，且只包含从 a-z 的小写字母。
- p 可能为空，且只包含从 a-z 的小写字母，以及字符 ? 和 *。

**示例 1:**

```java
输入:
s = "aa"
p = "a"
输出: false
解释: "a" 无法匹配 "aa" 整个字符串。
```

**示例 2:**

```java
输入:
s = "aa"
p = "*"
输出: true
解释: '*' 可以匹配任意字符串。
```

**示例 3:**

```java
输入:
s = "cb"
p = "?a"
输出: false
解释: '?' 可以匹配 'c', 但第二个 'a' 无法匹配 'b'。
```

**示例 4:**

```java
输入:
s = "adceb"
p = "*a*b"
输出: true
解释: 第一个 '*' 可以匹配空字符串, 第二个 '*' 可以匹配字符串 "dce".
```

**示例 5:**

```java
输入:
s = "acdcb"
p = "a*c?b"
输入: false
```

**解法一**

其实这题还有一道类似的题，[10. 正则表达式生成](http://imlgw.top/2019/10/10/leetcode-hui-su/#10-%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F%E5%8C%B9%E9%85%8D) 我放在回溯里了，因为dp的解法暂时还没花时间去看，感觉递归好理解一点，后面有时间再加进来

```java
public boolean isMatch(String s, String p) {
    //dp[i][j]代表s[0,i-1](s的前i个字符),p[0,j-1](p的前j个字符)是否匹配
    boolean[][] dp=new boolean[s.length()+1][p.length()+1];        
    //dp[i][0]=false dp[0][j] p[j]=="*"|"?"
    dp[0][0]=true;
    for (int j=1;j<=p.length();j++) {
        if (p.charAt(j-1)=='*') {
            dp[0][j]=dp[0][j-1]; //dp[0][j]=true是错的
        }
    }
    //1. p[i]=p[j] dp[i][j]=dp[i-1][j-1]
    //2. p[j]="?"  dp[i][j]=dp[i-1][j-1]
    //3. p[j]="*"  dp[i][j]=dp[i-1][j] | dp[i][j-1]
    for (int i=1;i<=s.length();i++) {
        for (int j=1;j<=p.length();j++) {
            if (s.charAt(i-1) == p.charAt(j-1) || p.charAt(j-1)=='?') {
                dp[i][j]=dp[i-1][j-1];
            }else if (p.charAt(j-1)=='*') { //abcd ab* | ab ab*
                dp[i][j]=dp[i-1][j]|dp[i][j-1];
            }
        }
    }
    return dp[s.length()][p.length()];
}
```

思路都在注释里面了，比正则那个要简单，比较不好想的一点就是 `dp[i][j]=dp[i-1][j]|dp[i][j-1]`其实分别对应的就是 `*` **匹配任意字符**的情况和**匹配空字符**的情况，匹配任意字符为什么不是`dp[i-1][j-1]`? 因为`*` 不仅仅只可以匹配一个字符，它可以匹配任意个数的任意字符，就比如这样的` abcd，ab* ` 的，这种情况其实最终还是转换成了匹配空字符的情况，这也是动态规划的思想所在。

## [10. 正则表达式匹配](https://leetcode-cn.com/problems/regular-expression-matching/)

给你一个字符串 `s` 和一个字符规律 `p`，请你来实现一个支持 `'.'` 和 `'*'` 的正则表达式匹配。

```java
'.' 匹配任意单个字符
'*' 匹配零个或多个前面的那一个元素
```


所谓匹配，是要涵盖 整个 字符串 s的，而不是部分字符串。

**说明:**

- s 可能为空，且只包含从 a-z 的小写字母。
- p 可能为空，且只包含从 a-z 的小写字母，以及字符 . 和 *。

**示例 1:**

```java
输入:
s = "aa"
p = "a"
输出: false
解释: "a" 无法匹配 "aa" 整个字符串。
```

**示例 2:**

```java
输入:
s = "aa"
p = "a*"
输出: true
解释: 因为 '*' 代表可以匹配零个或多个前面的那一个元素, 在这里前面的元素就是 'a'。因此，字符串 "aa" 可被视为 'a' 重复了一次。
```

**示例 3:**

```java
输入:
s = "ab"
p = ".*"
输出: true
解释: ".*" 表示可匹配零个或多个（'*'）任意字符（'.'）。
```

**示例 4:**

```java
输入:
s = "aab"
p = "c*a*b"
输出: true
解释: 因为 '*' 表示零个或多个，这里 'c' 为 0 个, 'a' 被重复一次。因此可以匹配字符串 "aab"。
```

**示例 5:**

```java
输入:
s = "mississippi"
p = "mis*is*p*."
输出: false
```

**解法一**

dfs回溯，暴力解法，挺难搞

```java
public boolean isMatch(String s, String p) {
    return match(s,0,p,0);
}

public boolean match(String s,int sIdx, String p,int pIdx) {
    if (sIdx>=s.length() && pIdx>=p.length()) {
        return true;
    }
    if (pIdx>=p.length()) {
        return false;
    }
    //这里的判断其实不太对,也不能说不对,会稍微慢一点,因为题目中是不会给**这样的case的
    //可以直接一次跳两步判断是不是*
    if (sIdx == s.length()) {
        //后面没有*的情况 aaa aaaa
        if (pIdx==p.length()-1 && p.charAt(pIdx)!='*') { //p也到尽头,并且不为*
            return false;
        }
        while(pIdx<p.length()){ //p没到尽头,检查后面的是否有两个连续的非*
            if (p.charAt(pIdx) != '*' && (pIdx+1<p.length() && p.charAt(pIdx+1)!='*')) {
                return false;
            }
            pIdx++;
        }
        //aaa aaa*c
        return p.charAt(pIdx-1)=='*';
    }

    if (pIdx+1 < p.length() && p.charAt(pIdx+1) =='*') { //pIdx下一个是 *
        //*匹配至少一个
        if ((s.charAt(sIdx) == p.charAt(pIdx) || p.charAt(pIdx)=='.') && match(s,sIdx+1,p,pIdx)) {
            return true;
        }
        //*匹配0个
        return match(s,sIdx,p,pIdx+2);
    }else{
        //pIdx下一个不是*
        if ((p.charAt(pIdx) == s.charAt(sIdx) || p.charAt(pIdx)=='.') && match(s,sIdx+1,p,pIdx+1)) {
            return true;
        }
        return false;
    }
}
```

这题还是挺麻烦的，毕竟是hard题，我也是看了题解才写出来，核心就是要把题目意思理解对，我这里没有做记忆化，懒得写

**解法二**

他来了他来了，他带着dp解法来了，终于还是把dp的解法补上来了，参考[题解 ](https://leetcode-cn.com/problems/regular-expression-matching/solution/dong-tai-gui-hua-si-yao-su-by-a380922457-5/) ，注意下标的转换，这里容易被绕进去，详细的都在注释中了

```java
//update: 2020.4.12
public boolean isMatch(String text, String pattern) {
    //dp[i][j]代表 text[0,i)和pattern[0,j)是否匹配
    boolean[][] dp=new boolean[text.length()+1][pattern.length()+1];
    //都为空的时候肯定是匹配的
    dp[0][0]=true;
    //pattern为空，text不为空肯定是无法匹配,默认false,不用处理
    //text为空,pattern不为空需要额外判断
    for(int i=2;i<=pattern.length();i++){
        dp[0][i]=pattern.charAt(i-1)=='*'&&dp[0][i-2];
    }

    for(int i=1;i<=text.length();i++){
        for(int j=1;j<=pattern.length();j++){
            if(singleMatch(text,pattern,i-1,j-1)){
                dp[i][j]=dp[i-1][j-1];
            }else if(pattern.charAt(j-1)=='*'){
                //j一定是>=2的,p如果是*开头就是错误的语法,不过lc其实也没有这样的case
                if(j<2) return false;
                //ab abb*    --> '*'匹配0个b
                //abbbbb ab* --> '*'匹配多个b
                //注意dp[i-1][j]和text.charAt(i-1)的i-1含义是不一样的
                //text.charAt(i-1)的i-1其实指的是当前元素,而dp[i-1][j]中的i-1是前一个元素
                //abc ab*
                dp[i][j]=dp[i][j-2] || (singleMatch(text,pattern,i-1,j-2) && dp[i-1][j]);
            }
        }
    }
    return dp[text.length()][pattern.length()];
}

public boolean singleMatch(String s,String p,int i,int j){
    return s.charAt(i)==p.charAt(j) || p.charAt(j)=='.';
}
```

果然还是动态规划简洁，而且搞懂之后也很好理解~~

## [1326. 灌溉花园的最少水龙头数目](https://leetcode-cn.com/problems/minimum-number-of-taps-to-open-to-water-a-garden/) 

在 x 轴上有一个一维的花园。花园长度为 n，从点 0 开始，到点 n 结束。

花园里总共有 n + 1 个水龙头，分别位于 [0, 1, ..., n] 。

给你一个整数 n 和一个长度为 n + 1 的整数数组 ranges ，其中 ranges[i] （下标从 0 开始）表示：如果打开点 i 处的水龙头，可以灌溉的区域为 [i -  ranges[i], i + ranges[i]] 。

请你返回可以灌溉整个花园的 最少水龙头数目 。如果花园始终存在无法灌溉到的地方，请你返回 -1 。

**示例 1：**

![image.png](https://i.loli.net/2020/02/03/kAPc2ELusMa9qJZ.png)

```java
输入：n = 5, ranges = [3,4,1,1,0,0]
输出：1
解释：
点 0 处的水龙头可以灌溉区间 [-3,3]
点 1 处的水龙头可以灌溉区间 [-3,5]
点 2 处的水龙头可以灌溉区间 [1,3]
点 3 处的水龙头可以灌溉区间 [2,4]
点 4 处的水龙头可以灌溉区间 [4,4]
点 5 处的水龙头可以灌溉区间 [5,5]
只需要打开点 1 处的水龙头即可灌溉整个花园 [0,5] 。
```

**示例 2：**

```java
输入：n = 3, ranges = [0,0,0,0]
输出：-1
解释：即使打开所有水龙头，你也无法灌溉整个花园。
```

**示例 3：**

```java
输入：n = 7, ranges = [1,2,1,0,2,1,0,1]
输出：3
```

**示例 4：**

```java
输入：n = 8, ranges = [4,0,0,0,0,0,0,0,4]
输出：2
```

**示例 5：**

```java
输入：n = 8, ranges = [4,0,0,0,4,0,0,0,4]
输出：1
```

**提示：**

- `1 <= n <= 10^4`
- `ranges.length == n + 1`
- `0 <= ranges[i] <= 100`

**解法一**

172周赛的最后一题

```java
public int minTaps(int n, int[] ranges) {
    int[] dp=new int[n+1]; //浇溉i位置前需要最少的水龙头
    Arrays.fill(dp,Integer.MAX_VALUE);
    dp[0]=0;
    for (int i=0;i<=n;i++) {
        int left=Math.max(0,i-ranges[i]);
        int right=Math.min(n,i+ranges[i]);
        if (dp[left]==Integer.MAX_VALUE) { //左边界不可达
            continue;
        }
        for (int j=left+1;j<=right;j++) {
            dp[j]=Math.min(dp[j],dp[left]+1); //范围内的区域dp值
        }
    }
    return dp[n]==Integer.MAX_VALUE?-1:dp[n];
}
```

动态规划的解法，对每个水龙头所覆盖的区域进行递推，dp[i]代表浇灌`0 ~ i` 位置区域所需要的最少的水头所以很容易得到递推公式`dp[j]=Math.min(dp[j],dp[left]+1)`  ，整体还是很好理解的

**解法二**

贪心算法，和[贪心专题](http://imlgw.top/2020/01/21/leetcode-tan-xin/)中的 [1024. 视频剪辑]()是一样的题目

有时间再来补吧

## [1335. 工作计划的最低难度](https://leetcode-cn.com/problems/minimum-difficulty-of-a-job-schedule/)

你需要制定一份 d 天的工作计划表。工作之间存在依赖，要想执行第 i 项工作，你必须完成全部 j 项工作（ `0 <= j < i`）。

你每天 **至少** 需要完成一项任务。工作计划的总难度是这 `d` 天每一天的难度之和，而一天的工作难度是当天应该完成工作的最大难度。

给你一个整数数组 `jobDifficulty` 和一个整数 d，分别代表工作难度和需要计划的天数。第 i 项工作的难度是 `jobDifficulty[i]`。

返回整个工作计划的 **最小难度** 。如果无法制定工作计划，则返回 -1 。

**示例 1：**

![image.png](https://i.loli.net/2020/01/27/ipKvNZQASOhVmdI.png)

```java
输入：jobDifficulty = [6,5,4,3,2,1], d = 2
输出：7
解释：第一天，您可以完成前 5 项工作，总难度 = 6.
第二天，您可以完成最后一项工作，总难度 = 1.
计划表的难度 = 6 + 1 = 7 
```

**示例 2：**

```java
输入：jobDifficulty = [9,9,9], d = 4
输出：-1
解释：就算你每天完成一项工作，仍然有一天是空闲的，你无法制定一份能够满足既定工作时间的计划表。
```

**示例 3：**

```java
输入：jobDifficulty = [1,1,1], d = 3
输出：3
解释：工作计划为每天一项工作，总难度为 3 。
```

**示例 4：**

```java
输入：jobDifficulty = [7,1,7,1,7,1], d = 3
输出：15
```

**解法一**

173周赛的最后一道dp题，事后一个小时左右独立的ac，还是挺舒服的，希望以后能更快ac

```java
public int minDifficulty(int[] jobDifficulty, int d) {
    int jlen=jobDifficulty.length;
    if(jlen<d){
        return -1;
    }
    //dp[i][j]   第i天完成前j项任务的最低难度
    int[][] dp=new int[d][jlen];
    for(int i=0;i<dp.length;i++){
        Arrays.fill(dp[i],Integer.MAX_VALUE);
    }
    int max=0;
    for(int j=0;j<jlen;j++){
        max=Math.max(max,jobDifficulty[j]);
        dp[0][j]=max;
    }
    for(int i=1;i<d;i++){
        for(int j=0;j<jlen;j++){
            for(int k=j-1;k>=i-1;k--){
                dp[i][j]=Math.min(dp[i][j], dp[i-1][k]+maxDifficulty(jobDifficulty,k+1,j));
            }
        }
    }
    return dp[d-1][jlen-1];
}

public int maxDifficulty(int[] jobDifficulty,int left,int right){
    int max=jobDifficulty[left];
    for(int i=left;i<=right;i++){
        max=Math.max(max,jobDifficulty[i]);
    }
    return max;
}
```

核心的递推公式： `dp[i][j]=min(dp[i][j],dp[i-1][0~k]+max(k+1,j))` 

其实也不是一开始就想到了这个递推公式，首先需要确定的是用几维数组和递推数组的含义，这个题目两个变量：天数和工作量；所以应该是二维的结构，`dp[i][j]` 然后对应题目所要求的东西，其实就可以确定dp数组的含义

`dp[i][j]` 的含义是前`i`天完成前`j`项工作的最低难度，最后返回的结果就是`dp[d-1][jlen-1]`

但是上面这个解法时间复杂度比较高，`O(dN^3)`，其实可以做一下预处理降低时间复杂度

**解法二**

预处理的地方也写成了动态规划hahaha~ 其实就是求一个**区间和**

```java
public int minDifficulty(int[] jobDifficulty, int d) {
    int jlen=jobDifficulty.length;
    if(jlen<d){
        return -1;
    }
    //dp[i][j]   前i天完成前j项任务的最低难度
    int[][] dp=new int[d][jlen];
    for(int i=0;i<dp.length;i++){
        Arrays.fill(dp[i],Integer.MAX_VALUE);
    }
    //预处理
    int[][] maxRange=new int[jlen][jlen];
    for(int i=0;i<jlen;i++){
        maxRange[i][i]=jobDifficulty[i];
        for (int j=i+1;j<jlen;j++) {
            //这里也是个动态规划hahaha~
            maxRange[i][j]=Math.max(maxRange[i][j-1],jobDifficulty[j]);
        }
    }
    for(int j=0;j<jlen;j++){
        dp[0][j]=maxRange[0][j];
    }
    for(int i=1;i<d;i++){
        for(int j=0;j<jlen;j++){
            for(int k=j-1;k>=i-1;k--){ //注意这里k>=i-1的含义
                dp[i][j]=Math.min(dp[i][j], dp[i-1][k]+maxRange[k+1][j]);
            }
        }
    }
    return dp[d-1][jlen-1];
}
```

这样时间复杂度就降低为`O(dN^2)` ，当然还是有一点不完美，就是对边界的处理可以通过对dp数组+1来规避，这个问题我之前也说过，这我尝试改了下，没改好，不改了，哈哈哈~

## [5331. 跳跃游戏 V](https://leetcode-cn.com/problems/jump-game-v/)

给你一个整数数组 arr 和一个整数 d 。每一步你可以从下标 i 跳到：

- `i + x` ，其中 `i + x < arr.length 且 0 < x <= d` 。
- `i - x` ，其中 `i - x >= 0 且 0 < x <= d` 。

除此以外，你从下标 i 跳到下标 j 需要满足：`arr[i] > arr[j] 且 arr[i] > arr[k]` ，其中下标 k 是所有 i 到 j 之间的数字（更正式的，min(i, j) < k < max(i, j)）。

你可以选择数组的任意下标开始跳跃。请你返回你 **最多** 可以访问多少个下标。

请注意，任何时刻你都不能跳到数组的外面

**示例 1：**

![image.png](https://i.loli.net/2020/02/02/lroNpJhPkVc46Ya.png)

```java
输入：arr = [6,4,14,6,8,13,9,7,10,6,12], d = 2
输出：4
解释：你可以从下标 10 出发，然后如上图依次经过 10 --> 8 --> 6 --> 7 。
注意，如果你从下标 6 开始，你只能跳到下标 7 处。你不能跳到下标 5 处因为 13 > 9 。你也不能跳到下标 4 处，因为下标 5 在下标 4 和 6 之间且 13 > 9 。
类似的，你不能从下标 3 处跳到下标 2 或者下标 1 处。
```

**示例 2：**

```java
输入：arr = [3,3,3,3,3], d = 3
输出：1
解释：你可以从任意下标处开始且你永远无法跳到任何其他坐标。
```

**示例 3：**

```java
输入：arr = [7,6,5,4,3,2,1], d = 1
输出：7
解释：从下标 0 处开始，你可以按照数值从大到小，访问所有的下标。
```

**提示：**

- `1 <= arr.length <= 1000`
- `1 <= arr[i] <= 10^5`
- `1 <= d <= arr.length`

**解法一**

预处理+回溯+记忆化

```java
private Integer[] cache=null;

public int maxJumps(int[] arr, int d) {
    int[][] max=new int[arr.length][arr.length];
    cache=new Integer[arr.length];
    for (int i=0;i<arr.length;i++) {
        max[i][i]=arr[i];
        for (int j=i+1;j<arr.length;j++) {
            max[i][j]=Math.max(max[i][j-1],arr[j]);
        }
    }
    int res=0;
    for (int i=0;i<arr.length;i++) {
        res=Math.max(jump(arr,d,i,max),res);
    }
    return res+1;
}

public int jump(int[] arr,int d,int index,int[][] max){ //从index起跳能跳多远
    if (cache[index]!=null) {
        return cache[index];
    }
    int res=0;
    for (int i=Math.max(index-d,0);i<index;i++) {
        if (arr[index]>arr[i] && arr[index]>max[i+1][index-1]) {
            res=Math.max(jump(arr,d,i,max)+1,res);
        }
    }
    //index=0, i=1 , i<=1
    for (int i=index+1;i<=Math.min(index+d,arr.length-1);i++) {
        if (arr[index]>arr[i] && arr[index]>max[index+1][i-1]) {
            res=Math.max(jump(arr,d,i,max)+1,res);
        }
    }
    return cache[index]=res;
}
```

其实已经在超时的边缘了。。。没有必要做预处理，一开始被题括号里面的信息给迷惑了，不然也不会搞个预处理，其实题目的意思就是只能向低处跳，我开始还搞了两个区间值haha，脑壳有点不亲白

**解法二**

改良后的回溯+记忆化

```java
private Integer[] cache=null;

public int maxJumps(int[] arr, int d) {
    cache=new Integer[arr.length];
    int res=0;
    for (int i=0;i<arr.length;i++) {
        res=Math.max(jump(arr,d,i),res);
    }
    return res+1;
}

public int jump(int[] arr,int d,int index){ 
    if (cache[index]!=null) {
        return cache[index];
    }
    int res=0;
    //挨个跳,有一个跳不到,后面所有的就都跳不到了
    for (int i=index-1;i>=Math.max(index-d,0) && arr[index]>arr[i];i--) {
        res=Math.max(jump(arr,d,i)+1,res);
    }
    for (int i=index+1;i<=Math.min(index+d,arr.length-1) && arr[index]>arr[i];i++) {
        res=Math.max(jump(arr,d,i)+1,res);
    }
    return cache[index]=res;
}
```

**解法三**

自底向上动态规划

```java
public int maxJumps(int[] arr, int d) {
    int res=0,len=arr.length;
    Pair[] pair=new Pair[len];
    int[] dp=new int[len];
    Arrays.fill(dp,1);
    for (int i=0;i<len;i++) pair[i]=new Pair(i,arr[i]);
    Arrays.sort(pair,(p1,p2)->p1.value-p2.value);
    for (int i=0;i<len;i++) {
        int index=pair[i].index;
        //向左
        for (int j=index-1;j>=Math.max(index-d,0) && arr[index]>arr[j];j--) {
            dp[index]=Math.max(dp[j]+1,dp[index]);
        }
        //向右
        for (int j=index+1;j<=Math.min(index+d,arr.length-1) && arr[index]>arr[j];j++) {
            dp[index]=Math.max(dp[j]+1,dp[index]);
        }
        res=Math.max(dp[index],res);
    }
    return res;
}

class Pair{
    int index;
    int value;
    public Pair(int index,int value){
        this.index=index;
        this.value=value;
    }
}
```

这里其实动态规划还有麻烦一点，因为需要从高向低跳，所以我们需要先保证低处的dp值先计算完成，所以我们需要先排序，同时要保留原索引，所以我们需要建立一个`Pair`数组来辅助，跳的方式和上面回溯的一样

## [1262. 可被三整除的最大和](https://leetcode-cn.com/problems/greatest-sum-divisible-by-three/)

给你一个整数数组 nums，请你找出并返回能被三整除的元素最大和。

**示例 1：**

```java
输入：nums = [3,6,5,1,8]
输出：18
解释：选出数字 3, 6, 1 和 8，它们的和是 18（可被 3 整除的最大和）。
```


**示例 2：**

```java
输入：nums = [4]
输出：0
解释：4 不能被 3 整除，所以无法选出数字，返回 0。
```

**示例 3：**

```java
输入：nums = [1,2,3,4,4]
输出：12
解释：选出数字 1, 3, 4 以及 4，它们的和是 12（可被 3 整除的最大和）。
```

**提示：**

- `1 <= nums.length <= 4 * 10^4`
- `1 <= nums[i] <= 10^4`

**解法一**

动态规划的解法，很久之前的一次周赛的题，好像是第一次双周赛的题？

```java
public int maxSumDivThree(int[] nums) {
    if (nums==null || nums.length<=0) {
        return 0;
    }
    //dp[i][k] 0~i 之间/3余k的最大元素和
    int[][] dp=new int[nums.length+1][3]; 
    dp[0][0]=0; //初始状态定义
    dp[0][1]=Integer.MIN_VALUE;
    dp[0][2]=Integer.MIN_VALUE;
    for (int i=1;i<=nums.length;i++) {
        if (nums[i-1]%3==0) {
            dp[i][0]=Math.max(dp[i-1][0],dp[i-1][0]+nums[i-1]);
            dp[i][1]=Math.max(dp[i-1][1],dp[i-1][1]+nums[i-1]);
            dp[i][2]=Math.max(dp[i-1][2],dp[i-1][2]+nums[i-1]);
        }
        if (nums[i-1]%3==1) {
            dp[i][0]=Math.max(dp[i-1][0],dp[i-1][2]+nums[i-1]);
            dp[i][1]=Math.max(dp[i-1][1],dp[i-1][0]+nums[i-1]);
            dp[i][2]=Math.max(dp[i-1][2],dp[i-1][1]+nums[i-1]);   
        }
        if (nums[i-1]%3==2) {
            dp[i][0]=Math.max(dp[i-1][0],dp[i-1][1]+nums[i-1]);
            dp[i][1]=Math.max(dp[i-1][1],dp[i-1][2]+nums[i-1]);
            dp[i][2]=Math.max(dp[i-1][2],dp[i-1][0]+nums[i-1]);   
        }
    }
    return  Math.max(0,dp[nums.length][0]);
}
```

看一下数据的范围`40000`，所以时间复杂度肯定是`O(N)` 或`O(NlogN)`的，`O(N^2)`的肯定超时啦

想了很久没想出来，然后瞄了一眼评论区，发现dp数组定义的是和余数相关的，然后就反应过来了，余0余1余2， 他们的组合就可以得到新的最大余数和，我们最后要求的就是最后`[0,nums.length]` 余数为0的最大和

递推公式就不用多说了，看代码就行了，递推公式很容易得到，关键就是初始的状态定义，第一步的递推，我一开始没有设置边界，直接开始dp然后后面怎么改都不对，一开始定义的base dp

```java
dp[0][0]=nums[0]%3==0?nums[0]:Integer.MIN_VALUE;
dp[0][1]=nums[0]%3==1?nums[0]:Integer.MIN_VALUE;
dp[0][2]=nums[0]%3==2?nums[0]:Integer.MIN_VALUE;
```

然后过了几十后卡在了个case上

```java
[2,19,6,16,5,10,7,4,11,6]
```

递推到第二个其实就有问题了

```java
      0     1      2 
2    MIN   MIN     2 
19   21    MIN+19  2
```

这里的19的1明显应该就是19，这里直接根据`dp[i-1][1]+19`递推得到的，后面的肯定都不对了，那我们这里将初始值设置为0可以么？

```java
dp[0][0]=nums[0]%3==0?nums[0]:0;
```

对这个case确实可以，但是对于其他的就不对了，这容易想到，你设置为0下一层的dp值就可以从这个值递推过来，但是这明显是不对的，举个例子： `3，4` 你将第一个3的余2最大值设置为0，那么下一个4就可以通过上一个余2的最大值来更新当前的余0最大值，变成4，这很明显就不对了，4根本就不能被3整除

所以这里我们最好的处理方式是添加一个边界，相当于在数组的最前面添加一个0，0%3=0

然后我们只需要将`dp[0][0]`设置为0就可以了，而1和2就设置为`INT_MIN`，让下一层不能通它来转移

> dp边界的处理方式还是要多练啊

**解法二**

优化空间为O(1)

```java
public int maxSumDivThree2(int[] nums) {
    int[] dp=new int[3];
    dp[1]=Integer.MIN_VALUE;
    dp[2]=Integer.MIN_VALUE;
    for (int i=1;i<=nums.length;i++) {
        if (nums[i-1]%3==0) {
            dp[0]=Math.max(dp[0],dp[0]+nums[i-1]);
            dp[1]=Math.max(dp[1],dp[1]+nums[i-1]);
            dp[2]=Math.max(dp[2],dp[2]+nums[i-1]);
        }
        if (nums[i-1]%3==1) {
            int temp0=dp[0];
            dp[0]=Math.max(dp[0],dp[2]+nums[i-1]);
            dp[2]=Math.max(dp[2],dp[1]+nums[i-1]); //按依赖关系调整顺序,减少变量
            dp[1]=Math.max(dp[1],temp0+nums[i-1]);
        }
        if (nums[i-1]%3==2) {
            int temp0=dp[0];
            dp[0]=Math.max(dp[0],dp[1]+nums[i-1]);
            dp[1]=Math.max(dp[1],dp[2]+nums[i-1]);
            dp[2]=Math.max(dp[2],temp0+nums[i-1]);
        }
    }
    return dp[0];
}
```

可以看出代码还是可以简化的很短的，为了表达的清晰就不改了

**解法三**

贪心，放在 [贪心专题](http://imlgw.top/2020/01/21/leetcode-tan-xin/#406-%E6%A0%B9%E6%8D%AE%E8%BA%AB%E9%AB%98%E9%87%8D%E5%BB%BA%E9%98%9F%E5%88%97) 中

## [面试题60. n个骰子的点数](https://leetcode-cn.com/problems/nge-tou-zi-de-dian-shu-lcof/)

把n个骰子扔在地上，所有骰子朝上一面的点数之和为s。输入n，打印出s的所有可能的值出现的概率。

你需要用一个浮点数数组返回答案，其中第 i 个元素代表这 n 个骰子所能掷出的点数集合中第 i 小的那个的概率。

**示例 1:**

```java
输入: 1
输出: [0.16667,0.16667,0.16667,0.16667,0.16667,0.16667]
```

**示例 2:**

```java
输入: 2
输出: [0.02778,0.05556,0.08333,0.11111,0.13889,0.16667,0.13889,0.11111,0.08333,0.05556,0.02778]
```

**限制：**

`1 <= n <= 11`

**解法一**

很有意思的题，看返回值还不容易想到用动态规划

```java
public double[] twoSum(int n) {
    //dp[i][j]代表i枚色子和为j的概率
    double[][] dp=new double[n+1][6*n+1];
    double probability=1.0/6.0;
    //base初始化
    for(int i=1;i<=6;i++) dp[1][i]=probability;
    for(int i=2;i<=n;i++){ //枚举色子
        for(int j=i;j<=i*6;j++){ //枚举点数
            for(int k=1;k<=j && k<=6;k++){ //枚举当前色子的点数
                dp[i][j]+=(probability*dp[i-1][j-k]);
            }
        }
    }
    double[] res=new double[5*n+1];//
    System.arraycopy(dp[n],n,res,0,res.length);
    return res;
}
```

`dp[i][j]`代表**i**枚色子和为**j**的概率 递推公式很容易想到 `dp[i][j]= 1/6(dp[i-1][j-1]+dp[i-1][j-2]+dp[i-1][j-3]...dp[i-1][j-6])` 然后我们枚举各个状态就ok了

## [264. 丑数 II](https://leetcode-cn.com/problems/ugly-number-ii/)

编写一个程序，找出第 n 个丑数。

丑数就是只包含质因数 2, 3, 5 的正整数。

**示例:**

```java
输入: n = 10
输出: 12
解释: 1, 2, 3, 4, 5, 6, 8, 9, 10, 12 是前 10 个丑数。
```


**说明:**  

1. 是丑数。
2. n 不超过1690。

**解法一**

小根堆，将base每次乘2乘3乘5的结果放入小根堆，然后将堆顶拿出来base，继续重复这个过程，中间要注意去重

```java
public int nthUglyNumber3(int n) {
    PriorityQueue<Long> queue=new PriorityQueue<>();
    n-=1;
    long base=1;
    while(n-- >0){
        queue.add(base*2);
        queue.add(base*3);
        queue.add(base*5);
        base=queue.poll();
        //去重
        while(!queue.isEmpty()&&base==queue.peek()){
            queue.poll();
        }
    }
    return (int)base;
}
```

用`HashSet`去重

```java
public int nthUglyNumber(int n) {
    PriorityQueue<Long> queue=new PriorityQueue<>();
    HashSet<Long> set=new HashSet<>();
    long base=1;
    long[] ugly={2,3,5};
    n-=1;
    set.add(1L);
    while(n-->0){
        for(int i=0;i<3;i++){
            if (!set.contains(ugly[i]*base)) {
                queue.add(ugly[i]*base);
                set.add(ugly[i]*base);
            }
        }
        base=queue.poll();
    }
    return (int)base;
}
```

**解法二**

这个解法还是很有技巧性的，其实就是按顺序生成丑数，一开始观察的时候就发现了但是确实想不到三指针的解法

```java
public int nthUglyNumber4(int n) {
    int[] dp=new int[n];
    dp[0]=1;
    int index1=0,index2=0,index3=0;
    for (int i=1;i<n;i++) {
        dp[i]=Math.min(dp[index1]*2,Math.min(dp[index2]*3,dp[index3]*5));
        index1+=(dp[index1]*2==dp[i]?1:0);
        index2+=(dp[index2]*3==dp[i]?1:0);
        index3+=(dp[index3]*5==dp[i]?1:0);
    }
    return dp[n-1];
}
```
## [面试题62. 圆圈中最后剩下的数字](https://leetcode-cn.com/problems/yuan-quan-zhong-zui-hou-sheng-xia-de-shu-zi-lcof/) 

0,1,,n-1这n个数字排成一个圆圈，从数字0开始，每次从这个圆圈里删除第m个数字。求出这个圆圈里剩下的最后一个数字。

例如，0、1、2、3、4这5个数字组成一个圆圈，从数字0开始每次删除第3个数字，则删除的前4个数字依次是2、0、4、1，因此最后剩下的数字是3。

**示例 1：**

```java
输入: n = 5, m = 3
输出: 3
```


**示例 2：**

```java
输入: n = 10, m = 17
输出: 2
```

**限制：**

- 1 <= n <= 10^5
- 1 <= m <= 10^6

**解法一**

用链表模拟的解法，在数据太大的时候会TLE

```java
/*
链表模拟解法
0 1 2 3 4 5 m=3
3 4 5 0 1
*/
public int lastRemaining(int n, int m) {
    Deque<Integer> queue=new LinkedList<>();
    for(int i=0;i<n;i++)
        queue.addLast(i);
    while(queue.size()>1){
        for(int i=0;i<m;i++){
            int temp=queue.removeFirst();
            if(i!=m-1){
                queue.addLast(temp);
            }
        }
    }
    return queue.getFirst();
}
```
**解法二**

数学归纳法，加上一点递推dp

```java
/*
    0 ~ n-1 每次kill第k个人
    k=(m-1)%n

    kill k之后, 在n-1个人中重建新索引
    原始索引   新的index
    k+1         0
    k+2         1
    k+3         2
    ...         ...
    n-1         n-k-2
    0           n-k-1
    1           n-k
    2           n-k+1
    ...         ...
    k-1         n-2

    新索引(x) ---> 原始索引(y)
    y=(x+k+1)%n  eg. (n-2+k+1)%n=(n+k-1)%n=k-1
    设在n-1个人中最后剩下的人是 f(n-1,m)按照上面的公式转换成原始索引就是
    f(n,m)=(f(n-1,m)+k+1)%n
 */
//数学解法
public int lastRemaining(int n, int m) {
    int last=0;//一个人的时候存活者index f(1,m)=0
    for(int i=2;i<=n;i++){ //枚举人数
        last=(last+m)%i;
    }
    return last;
}
```
参考了LeetCode的[大佬题解](https://leetcode-cn.com/u/yuanninesuns/)，讲的挺好的

**递归解法**

```java
//递归的挺好理解
public int lastRemaining(int n, int m) {
    if(n==0) return 0;
    return (lastRemaining(n-1,m)+m)%n;
}
```
## [1014. 最佳观光组合](https://leetcode-cn.com/problems/best-sightseeing-pair/)

给定正整数数组 A，A[i] 表示第 i 个观光景点的评分，并且两个景点 i 和 j 之间的距离为 j - i。

一对景点（i < j）组成的观光组合的得分为（A[i] + A[j] + i - j）：景点的评分之和减去它们两者之间的距离。

返回一对观光景点能取得的最高分。

**示例：**

```java
输入：[8,1,5,2,6]
输出：11
解释：i = 0, j = 2, A[i] + A[j] + i - j = 8 + 5 + 0 - 2 = 11
```

**提示：**

- `2 <= A.length <= 50000`
- `1 <= A[i] <= 1000`

**解法一**

暴力解法，`O(N^2)`，这题数据5w，肯定会T

```java
//A[i] + A[j] + i - j (i<j)
public int maxScoreSightseeingPair(int[] A) {
    int res=-1;
    for (int i=0;i<A.length;i++) {
        for (int j=i+1;j<A.length;j++) {
            res=Math.max(A[i]+A[j]+i-j,res);
        }
    }
    return res;
}
```
**解法二**

```java
// A[i]+A[j] - (j-i) i<j
// A[i]+i + (A[j]-j)
public int maxScoreSightseeingPair(int[] A) {
    int res=-1;
    int maxPre=A[0]+0;
    for (int j=1;j<A.length;j++) {           
        res=Math.max(maxPre+A[j]-j,res);
        maxPre=Math.max(maxPre,A[j]+j);
    }
    return res;
}
```
我们把式子合并一下 `(A[i]+i)+(A[j]-j)` ，然后发现其实上面的暴力解法中，我们想要结果最大实际上就是要两部分都最大，然后内层循环里面，`A[j]-j` 是个变量，而`A[i]+i`不变的，代表`j`位置之前，所有的`A[i]+i` 但是我们只需要最大的，所以我们完全可以用一个变量来保存这个最大值，然后遍历的过程中不断的更新它就ok了

## [983. 最低票价](https://leetcode-cn.com/problems/minimum-cost-for-tickets/)

在一个火车旅行很受欢迎的国度，你提前一年计划了一些火车旅行。在接下来的一年里，你要旅行的日子将以一个名为 `days` 的数组给出。每一项是一个从 `1` 到 `365` 的整数。

火车票有三种不同的销售方式：

- 一张为期一天的通行证售价为 `costs[0]` 美元；
- 一张为期七天的通行证售价为 `costs[1]` 美元；
- 一张为期三十天的通行证售价为 `costs[2]` 美元。

通行证允许数天无限制的旅行。 例如，如果我们在第 2 天获得一张为期 7 天的通行证，那么我们可以连着旅行 7 天：第 2 天、第 3 天、第 4 天、第 5 天、第 6 天、第 7 天和第 8 天。

返回你想要完成在给定的列表 `days` 中列出的每一天的旅行所需要的最低消费。

**示例 1：**

```java
输入：days = [1,4,6,7,8,20], costs = [2,7,15]
输出：11
解释： 
例如，这里有一种购买通行证的方法，可以让你完成你的旅行计划：
在第 1 天，你花了 costs[0] = $2 买了一张为期 1 天的通行证，它将在第 1 天生效。
在第 3 天，你花了 costs[1] = $7 买了一张为期 7 天的通行证，它将在第 3, 4, ..., 9 天生效。
在第 20 天，你花了 costs[0] = $2 买了一张为期 1 天的通行证，它将在第 20 天生效。
你总共花了 $11，并完成了你计划的每一天旅行。
```

**示例 2：**

```java
输入：days = [1,2,3,4,5,6,7,8,9,10,30,31], costs = [2,7,15]
输出：17
解释：
例如，这里有一种购买通行证的方法，可以让你完成你的旅行计划： 
在第 1 天，你花了 costs[2] = $15 买了一张为期 30 天的通行证，它将在第 1, 2, ..., 30 天生效。
在第 31 天，你花了 costs[0] = $2 买了一张为期 1 天的通行证，它将在第 31 天生效。 
你总共花了 $17，并完成了你计划的每一天旅行。
```

**提示：**

1. `1 <= days.length <= 365`
2. `1 <= days[i] <= 365`
3. `days` 按顺序严格递增
4. `costs.length == 3`
5. `1 <= costs[i] <= 1000`

**解法一**

这题的递推关系其实很容易就想到：`dp[i]=Min(dp[i-dt[k]]+costs[k],dp[i])`，可是菜鸡的我还是搞了半天才AC

```java
public int mincostTickets(int[] days, int[] costs) {
    int n=days.length;
    int[] dp=new int[days[n-1]+1];
    boolean[] visit=new boolean[days[n-1]+1];
    for (int i:days) {
        visit[i]=true;
    }
    Arrays.fill(dp,0x3f3f3f3f);
    int [] dt = {1,7,30};
    dp[days[0]-1]=0;//初始状态
    //枚举所有的day
    for (int d=days[0];d<=days[n-1];d++) {
        //一开始没想到这里，dubug的时候才看出来，不在旅游计划中的应该直接延续前面的
        if (!visit[d]) { 
            dp[d]=dp[d-1];
            continue;
        }
        //三种票
        for (int k=0;k<3;k++) {
            dp[d]=Math.min((d-dt[k]>=days[0]-1?dp[d-dt[k]]:0)+costs[k],dp[d]);
        }
    }
    return dp[days[n-1]];
}
```
一开始写了二维的，然后发现一维的状态并没有用，改成一维之后又dubug了半天，主要就在那个非旅行计划中的`d`没处理好，这题的状态定义不是特别清晰，然后搞混了，把非旅行计划内的也算进去了

## _博弈型动态规划_

## [292. Nim 游戏](https://leetcode-cn.com/problems/nim-game/)

你和你的朋友，两个人一起玩 Nim 游戏：桌子上有一堆石头，每次你们轮流拿掉 1 - 3 块石头。 拿掉最后一块石头的人就是获胜者。你作为先手。

你们是聪明人，每一步都是最优解。 编写一个函数，来判断你是否可以在给定石头数量的情况下赢得游戏。

**示例:**

```java
输入: 4
输出: false 
解释: 如果堆中有 4 块石头，那么你永远不会赢得比赛；
     因为无论你拿走 1 块、2 块 还是 3 块石头，最后一块石头总是会被你的朋友拿走。
```

**解法一**

博弈型动态规划入门题

```java
public boolean canWinNim(int n) {
    boolean[] dp=new boolean[n];
    dp[0]=true;
    dp[1]=true;
    dp[2]=true;
    //t t t f t t t f .....
    //每个状态取决于前3个状态
    for (int i=3;i<n;i++) {
        //分别拿1,2,3个看对面能不能赢
        dp[i]=!dp[i-1] || !dp[i-2] || !dp[i-3];
    }
    return dp[n-1];
}
```
这个解法在数据过大的时候还是会TLE

**解法二**

规律

```java
public boolean canWinNim(int n) {
    return n%4!=0;
}
```
##  _字符串类型的动态规划_

## [1143. 最长公共子序列](https://leetcode-cn.com/problems/longest-common-subsequence/)

给定两个字符串 text1 和 text2，返回这两个字符串的最长公共子序列。

一个字符串的 子序列 是指这样一个新的字符串：它是由原字符串在不改变字符的相对顺序的情况下删除某些字符（也可以不删除任何字符）后组成的新字符串。
例如，"ace" 是 "abcde" 的子序列，但 "aec" 不是 "abcde" 的子序列。两个字符串的「公共子序列」是这两个字符串所共同拥有的子序列。 若这两个字符串没有公共子序列，则返回 0 

> 最长公共子序列问题是在一组序列（通常2个）中找到最长公共子序列（注意：不同于子串，LCS不需要是连续的子串）。该问题是典型的计算机科学问题，是文件差异比较程序的基础，在生物信息学中也有所应用

**示例 1:**

```java
输入：text1 = "abcde", text2 = "ace" 
输出：3  
解释：最长公共子序列是 "ace"，它的长度为 3。
```

**示例 2:**

```java
输入：text1 = "abc", text2 = "abc"
输出：3
解释：最长公共子序列是 "abc"，它的长度为 3。
```

**示例 3:**

```java
输入：text1 = "abc", text2 = "def"
输出：0
解释：两个字符串没有公共子序列，返回 0。
```

**解法一**

2019.10.19力扣终于有这题了，重新来做一遍，这题也就是经常说的LCS，dp的经典问题

```java
public int longestCommonSubsequence(String A, String B) {
    int[][] dp=new int[A.length()+1][B.length()+1];
    for (int i=1;i<=A.length();i++) {
        for (int j=1;j<=B.length();j++) {
            if (A.charAt(i-1)==B.charAt(j-1)) {
                dp[i][j]=dp[i-1][j-1]+1;
            }else{
                dp[i][j]=Math.max(dp[i-1][j],dp[i][j-1]);
            }
        }
    }
    return dp[A.length()][B.length()];
}
```
其实看见这种什么最长的多半和dp有关系，这题只要明白几个要点就可以推出dp方程

- `dp[i][j]` 代表A序列前`i`个字符（记为`Ai`）， 和B序列前`j`个字符（记为`Bj`） 的LCS

- 两层for循环遍历两个序列，当`A[i]==B[j]`的时候，LCS长度就会增加 1，进而得到dp方程

  `dp[i][j]=dp[i-1][j-1]+1`   dp[i-1] [j-1]代表的就是不包含`Ai/Bj`的`Ai-1`和`Bj-1` 的LCS

- `A[i]!=B[j]`的时候就有两种情况了，首先`A[i]`和`B[j]`已经**不可能同时出现在最终的LCS里面**了，所以我们考虑另外的两种情况`dp[i][j-1]`和`dp[i-1][j]`取个最大值

除此之外，还有一个值得注意的小细节，这里循环都是从1开始的，这样就不用处理初始值，如果从0开始的话，处理初始情况（i，j=0的情况）会麻烦一点

**解法二**

2019.10.19重写了一种记忆化递归的方法，没有dp快，但是思路其实是类似的

```java
Integer[][] cache=null;

public int longestCommonSubsequence(String text1, String text2) {
    if(text2==null || text1==null || text1.length()<=0 ||text2.length()<=0){
        return 0;
    }
    int len1=text1.length();
    int len2=text2.length();
    cache=new Integer[len1][len2];
    return lcs(text1,len1-1,text2,len2-1);
}

//lcs定义: 求text1[0,a]和text2[0,b]的最长公共子序列
public int lcs(String text1, int a,String text2,int b) {
    if(a==-1 || b==-1){
        return 0;
    }
    if (cache[a][b]!=null) {
        return cache[a][b];
    }
    if (text1.charAt(a)==text2.charAt(b)) {
        cache[a][b]=lcs(text1,a-1,text2,b-1)+1;
        return cache[a][b];
    }else{
        cache[a][b]=Math.max(lcs(text1,a-1,text2,b),lcs(text1,a,text2,b-1));
        return cache[a][b];
    }
}
```
## [718. 最长重复子数组](https://leetcode-cn.com/problems/maximum-length-of-repeated-subarray/)

给两个整数数组 `A` 和 `B` ，返回两个数组中公共的、长度最长的子数组的长度。

**示例 1:**

```java
输入:
A: [1,2,3,2,1]
B: [3,2,1,4,7]
输出: 3
解释: 
长度最长的公共子数组是 [3, 2, 1]。
```

**说明:**

1. 1 <= len(A), len(B) <= 1000
2. 0 <= A[i], B[i] < 100

**解法一**

简单的递推，注意别写成上面的lcs了，这里的`dp[i][j]` 代表的其实是**公共子串**以`A[i]`和`B[i]` 结尾的最长重复子数组长度，如果`A[i]`和`B[i]`不相等那就是0

```java
public int findLength(int[] A, int[] B) {
    int lenA=A.length;
    int lenB=B.length;
    int[][] dp=new int[lenA+1][lenB+1];
    int res=0;
    for(int i=1;i<=lenA;i++){
        for(int j=1;j<=lenB;j++){
            if(A[i-1]==B[j-1]){
                dp[i][j]=dp[i-1][j-1]+1;
                res=Math.max(dp[i][j],res);
            }
        }
    }
    return res;
}
```
**解法二**

> 本来是想练练二分，在二分的tag里面看见的这题，结果发现这题dp似乎更容易，看了官方的二分的解法，其实就是二分公共子串的长度，然后检查，检查这一步有一些操作，不是很理解，以后有机会再来补充

## [72. 编辑距离](https://leetcode-cn.com/problems/edit-distance/)

给定两个单词 word1 和 word2，计算出将 word1 转换成 word2 所使用的最少操作数 。

你可以对一个单词进行如下三种操作：

- 插入一个字符
- 删除一个字符
- 替换一个字符

**示例 1:**

```java
输入: word1 = "horse", word2 = "ros"
输出: 3
解释: 
horse -> rorse (将 'h' 替换为 'r')
rorse -> rose (删除 'r')
rose -> ros (删除 'e')
```


**示例 2:**

```java
输入: word1 = "intention", word2 = "execution"
输出: 5
解释: 
intention -> inention (删除 't')
inention -> enention (将 'i' 替换为 'e')
enention -> exention (将 'n' 替换为 'x')
exention -> exection (将 'n' 替换为 'c')
exection -> execution (插入 'u')
```

**解法一**

很经典的动态规划的问题，做之前看了动态规划的思路，然后很快写出了递归的解法。。。

```java
Integer[][] cache=null;

public int minDistance(String word1, String word2) {
    cache=new Integer[word1.length()][word2.length()];
    return minDistance(word1,word1.length()-1,word2,word2.length()-1);
}

//递归的定义: word1[0,idx1] 和 word2[0,idx2] 的最短编辑距离
public int minDistance(String word1,int idx1,String word2,int idx2) {
    if (idx1<0) {
        return idx2+1;
    }
    if (idx2<0) {
        return idx1+1;
    }
    if (cache[idx1][idx2]!=null) {
        return cache[idx1][idx2];
    }
    if (word1.charAt(idx1) == word2.charAt(idx2)) {
        return cache[idx1][idx2]=minDistance(word1,idx1-1,word2,idx2-1);
    }else{
        return cache[idx1][idx2]=1+Math.min(minDistance(word1,idx1-1,word2,idx2),Math.min(minDistance(word1,idx1,word2,idx2-1),minDistance(word1,idx1-1,word2,idx2-1)));
    }
}
```
核心的递推公式就是 

`minDis(i,j)=minDis(i-1,j-1)  word1[i]==word2[j]`

`minDis(i,j)=min(minDis(i-1,j),minDis(i,j-1),minDis(i-1,j-1))+1  word1[i]!=word2[j]`

第一个公式好说，相等时候就别动，延续之前的状态，关键是第二个，不相等的时候，简单来说就是求对应的三种操作增，删，改的最小值，但是这里，第二种删的操作，这里要想清楚，其实word1删除一个其实就等价于word2增加一个，所以是`minDis(i,j-1)`，虽然题目说的是从word1转换为word2，但是其实结果都是一样的，这个转换的过程其实是可逆的

**解法二**

标准的动态规划写法，感觉还是上面的记忆化递归好写，动态规划要考虑的边界比递归要多

```java
public int minDistance2(String word1, String word2) {
    if (word1.length()<=0 || word2.length()<=0) {
        return word2.length()==0?word1.length():word2.length();
    }
    //dp[i][j]: word1前i个和word2前j个的最短编辑距离
    int[][] dp=new int[word1.length()+1][word2.length()+1];
    for (int i=1;i<=word2.length();i++) {
        dp[0][i]=i;
    }
    for (int i=1;i<=word1.length();i++) {
        dp[i][0]=i;
    }
    for (int i=1; i<=word1.length();i++) {
        for (int j=1;j<=word2.length();j++) {
            if (word1.charAt(i-1)==word2.charAt(j-1)) {
                dp[i][j]=dp[i-1][j-1];
            }else{
                dp[i][j]=1+Math.min(dp[i-1][j],Math.min(dp[i][j-1],dp[i-1][j-1]));
            }
        }
    }
    return dp[word1.length()][word2.length()];
}
```

## [647. 回文子串](https://leetcode-cn.com/problems/palindromic-substrings/)

给定一个字符串，你的任务是计算这个字符串中有多少个回文子串。

具有不同开始位置或结束位置的子串，即使是由相同的字符组成，也会被计为是不同的子串。

**示例 1:**

```java
输入: "abc"
输出: 3
解释: 三个回文子串: "a", "b", "c".
```

**示例 2:**

```java
输入: "aaa"
输出: 6
说明: 6个回文子串: "a", "a", "a", "aa", "aa", "aaa".
```


**注意:**

1. 输入的字符串长度不会超过1000。

**解法一**

动态规划的解法，其实我是没想到的，`dp[i][j]`代表的是`s[i,j]` 是不是回文子串

```java
public int countSubstrings(String s) { 
    if (s==null || s.length()<=0) {
        return 0;
    }
    int n=s.length();
    //aaaba
    //dp[i][j] i~j 是不是回文
    boolean [][] dp=new boolean[n][n];
    int res=n; //下面的循环没有统计单个字符的情况
    for (int i=n-1;i>=0;i--) {
        for (int j=i+1;j<n;j++) { //统一写法，这里改成j=i就可以统计到单个字符的情况
            if (s.charAt(i) == s.charAt(j) && (j-i<=2 || dp[i+1][j-1])) {
                dp[i][j]=true;
                res++;
            }
        }
    }
    return res;
}
```

**解法二**

很熟悉的解法，没错就是我们之前 [最长回文子串中](https://leetcode-cn.com/problems/longest-palindromic-substring/) 的解法，比上面的动态规划会快很多，也比较自然

```java
//中心扩展法
public int countSubstrings(String s) {
    if (s==null || s.length()<=0) {
        return 0;
    }
    int n=s.length();
    int count=0;
    for (int i=0;i<n;i++) {
        count+=spread(s,i,i+1)+spread(s,i,i);
    }
    return count;
}

public int spread(String s,int i,int j){
    int res=0;
    while(i>=0 && j<s.length()){
        if (s.charAt(i)==s.charAt(j)) {
            i--;j++;
            res++;
        }else{
            return res;
        }
    }
    return res;
}
```


## [32. 最长有效括号](https://leetcode-cn.com/problems/longest-valid-parentheses/)

给定一个只包含 '(' 和 ')' 的字符串，找出最长的包含有效括号的子串的长度。

**示例 1:**

```java
输入: "(()"
输出: 2
解释: 最长有效括号子串为 "()"
```

**示例 2:**

```java
输入: ")()())"
输出: 4
解释: 最长有效括号子串为 "()()"
```

**解法一**

栈+dp的思路

```java
public int longestValidParentheses(String s) {
    if (s==null || s.length()<=0) {
        return  0;
    }
    Stack<Integer> stack=new Stack<>();
    int[] dp=new int[s.length()]; //以i位置括号结尾的最长有效括号
    dp[0]=0;
    int res=0;
    for (int i=0;i<s.length();i++) {
        if (s.charAt(i)=='(') {
            stack.push(i); //dp[i]=0
        }else{
            if(!stack.isEmpty()){
                int left=stack.pop();
                dp[i]= left==0?i-left+1:dp[left-1]+i-left+1;
                res=Math.max(res,dp[i]);
            }//else dp[i]=0  
        }
    }
    return res;
}
```

首先想到的就是这种解法，感觉还是很好理解的，栈中存索引，`dp[i]` 代表以`i`位置括号结尾的最长有效括号，当遇到右括号的时候将栈顶的左括号的索引`left`弹出来，两者形成的括号长度再加上左括号索引前一个元素`dp[left-1]`就是当前位置的`dp[i]`

**Update**

回头做第二遍地时候竟然写了个不一样的dp思路hahaha~，还是挺有意思的，遇到右括号不是直接用`i-left+1` 算长度，而是直接根据前一个字符`dp[i-1]` 进行转化，其实是一样的，只不过是累加的，前面的是一次就全算出来了

```java
//update: 2020.4.13
public int longestValidParentheses(String s) {
    if(s==null || s.length()<=0) return 0;
    Deque<Integer> stack=new ArrayDeque<>();
    int[] dp=new int[s.length()];
    int res=0;
    for(int i=0;i<s.length();i++){
        if(s.charAt(i)=='('){
            stack.push(i);
        }else{
            if(!stack.isEmpty()){
                int left=stack.pop();
                //dp[i]=(left>1?dp[left-1]+dp[i-1]:dp[i-1])+2;
                dp[i]=dp[i-1]+(left>1?dp[left-1]:0)+2;
            }
            res=Math.max(res,dp[i]);
        }
    }
    return res;
}
```
**解法二**

纯dp解法，上面的解法虽然也用到了一点动态规划，但是还不够纯粹

```java
public int longestValidParentheses(String s) {
    if (s==null || s.length()<=0) {
        return  0;
    }
    int[] dp=new int[s.length()];
    dp[0]=0;
    int res=0;
    for (int i=1;i<s.length();i++) {
        if (s.charAt(i)==')') {
            if (s.charAt(i-1)=='(') { //前一个是左括号
                dp[i]=i-2>=0?dp[i-2]+2:2;
            }else{ //前一个是右括号 ()(())
                if (i-dp[i-1]-1>=0 && s.charAt(i-dp[i-1]-1)=='(') {//前一个右括号的合法序列的前一个是'('
                    //前一个的前一个的dp[i-dp[i-1]-2]+dp[i-1]+2
                    dp[i]=(i-dp[i-1]-2>=0?dp[i-dp[i-1]-2]:0)+dp[i-1]+2; 
                }
                //前一个右括号的合法序列的前一个是')',没救了
            }
        }
        res=Math.max(res,dp[i]);
    }
    return res;
}
```

解释都在代码注释中，感觉没那么容易直接想到，毕竟hard题

> 其实还有两种方法，一种利用纯利用栈的，还有一种很神奇的方法，没什么通用性，很难直接想出来，就不做记录了，纯栈的解法后面再来补充

**解法三**

~~纯利用栈的解法以后再来补充~~ 算了，直接写了吧

```java
public int longestValidParentheses(String s) {
    if (s==null || s.length()<=0) {
        return  0;
    }
    Deque<Integer> stack=new ArrayDeque<>();
    stack.push(-1); //临界点
    int res=0;
    for(int i=0;i<s.length();i++){
        if(s.charAt(i)=='('){
            stack.push(i);
        }else{
            stack.pop();
            if(stack.isEmpty()){ 
                //栈为空,说明当前的')'肯定是不合法的
                //其实相当于一个分界点,这个位置之前的字符不可能再构成合法的序列了
                //后面的栈空的时候就不能再根据-1来算长度了,需要一个新的临界点
                stack.push(i);
            }else{
                res=Math.max(res,i-stack.peek());
            }
        }
    }
    return res;
}
```
关键的地方就在于将**非法的右括号**入栈，作为一个分界点便于后面计算，初始的-1也很关键

---

后面有时间会单独将这些题目分类整理成文章，目前暂时先这样

## _区间DP_

## [312. 戳气球](https://leetcode-cn.com/problems/burst-balloons/)

有 n 个气球，编号为0 到 n-1，每个气球上都标有一个数字，这些数字存在数组 nums 中。

现在要求你戳破所有的气球。每当你戳破一个气球 i 时，你可以获得 `nums[left] * nums[i] * nums[right]` 个硬币。 这里的 `left` 和 `right` 代表和 i 相邻的两个气球的序号。注意当你戳破了气球 i 后，气球 left 和气球 right 就变成了相邻的气球。

求所能获得硬币的最大数量。

**说明:**

- 你可以假设 nums[-1] = nums[n] = 1，但注意它们不是真实存在的所以并不能被戳破。
- 0 ≤ n ≤ 500, 0 ≤ nums[i] ≤ 100

**示例:**

```java
输入: [3,1,5,8]
输出: 167 
解释: nums = [3,1,5,8] --> [3,5,8] -->   [3,8]   -->  [8]  --> []
     coins =  3*1*5      +  3*5*8    +  1*3*8      + 1*8*1   = 167
```

**解法一**

暴力回溯，其实就是尝试所有的组合，这样时间复杂度`O(N!)` ，数据超过十几就不行了

```java
public int maxCoinsTLE(int[] nums) {
    LinkedList<Integer> list=new LinkedList<>();
    for (int n:nums) list.add(n);
    dfs(list,0);
    return max;
}

private int max=0;

public void dfs(LinkedList<Integer> list,int sum) {
    if (list.size()==0) {
        max=Math.max(sum,max);
        return;
    }
    for (int i=0;i<list.size();i++) {
        int temp=list.get(i);
        //这个值要先算
        int cur=(i-1<0?1:list.get(i-1))*(i+1>=list.size()?1:list.get(i+1))*temp;
        list.remove(i);
        dfs(list,sum+cur);
        list.add(i,temp);
    }
}
```

**解法二**

区间型动态规划，也是我第一次听说这个名词，但是之前其实已经做了不少的区间型的dp了

```java
public int maxCoins(int[] nums) {
    if (nums==null || nums.length<=0) {
        return 0;
    }
    int[] A=new int[nums.length+2];
    A[0]=1;A[A.length-1]=1;
    for (int i=0;i<nums.length;i++) {
        A[i+1]=nums[i]; //copy一个新数组
    }
    //区间DP
    int n=A.length;
    int[][] dp=new int[n][n]; //dp[i][j]代表的是不包含边界i,j的最大得分
    for (int len=2;len<=n;len++) { //枚举区间长度
        for (int i=0;i<=n-len;i++) { //枚举起点
            int j=i+len-1; //区间终点
            for (int k=i+1;k<j;k++) { //枚举分割点
                dp[i][j]=Math.max(dp[i][j],dp[i][k]+dp[k][j]+A[k]*A[i]*A[j]);
            }
        }
    }
    return dp[0][n-1];
}
```

这里为了处理边界将数组头尾插入了1，copy了一个新的数组，下面的区间型dp也是在看了题解，然后对着板子写的，当然这题的关键不是这里，关键是如何规划，这里搬运 [题解区大佬](https://leetcode-cn.com/problems/burst-balloons/solution/dong-tai-gui-hua-qiu-jie-by-feng-ji-wei-wang/) 的解释

我们来分析一下按照上面的那个递归的思路状态转移方程能写吗？如果按照上面的递归的思路，我们定义dp[i][j]表示对于i-j的气球的最大的收益，那状态转移方程就是

`dp[i][j]=max(coins[k]* coins[k-1] * coins[k+1]+dp[i][k-1]+dp[k+1][j]) k∈[i,j]`,

就按上面的那个例子，[3,1,5,8],来写一下过程

```java
扎爆3 剩下[]和[1,5,8]
扎爆1 剩下[3]和[5,8]
扎爆5 剩下[3,1]和[8]
扎爆8 剩下[3,1,5]和[]
```

然后在这些里面找到一个最大值返回，这其中对于扎爆1和8的最大的收益这样定义是没有问题的，因为两者都在边界，但是对于扎爆1和5就有问题了，就拿先扎爆1来说，扎爆1，剩下的最大的收益变为了[3]和[5,8]这两个区间的最大的收益的和，这个肯定不对，因为剩下的[3,5,8]区间的最大的收益的和不会是[3]的最大收益和与[5,8]的最大收益和的总和构成的，所以这个状态转移方程是不对的

那应该怎么定义状态转移方程呢，这也就是这道题巧妙的地方，它使用逆向思维，也就是上面的状态转移方程是由某个气球先爆得出来的，那么这里我们让指定的气球最后爆，这样的话状态转移方程为

`dp[i][j]=max(coins[k]*coins[i-1]*coins[j+1]+dp[i][k-1]+dp[k+1][j]) k∈[i,j]` 

再来看上面的例子，就对了，也就是对于[i,j]的气球，我们让某个位置的气球最后再爆，求出它左区间的最大的收益和右区间的最大的收益，加上这个气球最后爆掉所获得的收益，一 一比较，返回一个最大值就好了，这里要注意的是，要按区间的长度来进行状态的转移，也就是先求长度为1的，然后2的依次类推，因为这里状态转移方程涉及到了当前位置的后面的区间的最大的收益，但是后面的区间的长度是小于当前区间的长度的，故应该按长度来进行状态的转移，这也是典型的区间型动态规划的套路

**解法三**

在解法二的基础上简化了代码，`dp[i][j]`代表`i`和`j`之间，包含`i`和`j`位置，戳爆所有气球的最大得分

```java
public int maxCoins(int[] nums) {
    if (nums==null || nums.length<=0) {
        return 0;
    }
    int n=nums.length;
    //dp[i][j]代表的是包含边界i,j的最大得分
    int[][] dp=new int[n][n]; 
    for (int len=1;len<=n;len++) { //枚举区间长度
        for (int i=0;i<=n-len;i++) { //枚举起点
            int j=i+len-1; //区间终点
            for (int k=i;k<=j;k++) { //枚举分割点
                dp[i][j]=Math.max(dp[i][j],(k-1>=0?dp[i][k-1]:0)+(k+1<n?dp[k+1][j]:0)+nums[k]*(i-1>=0?nums[i-1]:1)*(j+1<n?nums[j+1]:1));
            }
        }
    }
    return dp[0][n-1];
}
```

## [877. 石子游戏](https://leetcode-cn.com/problems/stone-game/)

亚历克斯和李用几堆石子在做游戏。偶数堆石子**排成一行**，每堆都有正整数颗石子 `piles[i]` 。

游戏以谁手中的石子最多来决出胜负。石子的总数是奇数，所以没有平局。

亚历克斯和李轮流进行，亚历克斯先开始。 每回合，玩家从行的开始或结束处取走整堆石头。 这种情况一直持续到没有更多的石子堆为止，此时手中石子最多的玩家获胜。

假设亚历克斯和李都发挥出最佳水平，当亚历克斯赢得比赛时返回 `true` ，当李赢得比赛时返回 `false` 。

**示例：**

```java
输入：[5,3,4,5]
输出：true
解释：
亚历克斯先开始，只能拿前 5 颗或后 5 颗石子 。
假设他取了前 5 颗，这一行就变成了 [3,4,5] 。
如果李拿走前 3 颗，那么剩下的是 [4,5]，亚历克斯拿走后 5 颗赢得 10 分。
如果李拿走后 5 颗，那么剩下的是 [3,4]，亚历克斯拿走后 4 颗赢得 9 分。
这表明，取前 5 颗石子对亚历克斯来说是一个胜利的举动，所以我们返回 true 。
```

**提示：**

1. `2 <= piles.length <= 500`
2. `piles.length` 是偶数。
3. `1 <= piles[i] <= 500`
4. `sum(piles)` 是奇数。

**解法一**

博弈型dp，先采用区间dp的解法试试（区间dp是真的妙啊！！！)

```java
//通用的区间DP
public boolean stoneGame(int[] piles) {
    //[5,3,4,5]
    int N=piles.length;
    //(i~j) 先手(0),后手(1)的最大收益
    int[][][] dp=new int[N][N][2];
    //base init len=1的情况
    for(int i=0;i<N;i++){
        dp[i][i][0]=piles[i];
        dp[i][i][1]=0;
    }
    for (int len=2;len<=N;len++) { //枚举区间长度
        for (int left=0;left<=N-len;left++) { //枚举所有区间
            //left+len-1<N
            int right=left+len-1;
            //先手拿left或者right的最大收益
            //我先手拿了left或者right之后，剩下[left+1,right]或[left,right-1]区间
            //我在剩下的区间中继续选其实就成为了后手，所以我们加上剩下区间的后手最大值
            int firLeft=piles[left]+dp[left+1][right][1];
            int firRight=piles[right]+dp[left][right-1][1];
            if(firLeft>firRight){
                dp[left][right][0]=firLeft;
                //先手选left那么就相当于让另一个人（后手）从[left+1,right]中先手取最大值
                dp[left][right][1]=dp[left+1][right][0];
            }else{
                dp[left][right][0]=firRight;
                //同上
                dp[left][right][1]=dp[left][right-1][0];
            }
        }
    }
    return dp[0][N-1][0]-dp[0][N-1][1]>0;
}
```
**解法二**

数学推理的方法，具体的证明给不出来，但是很容易理解它的正确性

```java
//数学
public boolean stoneGame(int[] piles) {
    return true;
}
```
首先题目有两个特殊的条件（这两个条件我解法一没用到）`piles.length` 是偶数，`sum(piles)` 是奇数，这其实说明了不会有平局，因为是偶数，我们把所有的柱子用奇偶来划分，那么奇数柱子和偶数**石头堆的数量**一定是相同的，而且**两者总的石头数一定有一个数量的差异**，不可能相等，我先手拿的话我就可以只考虑拿较大的那一类石头堆，那我能保证我拿到的一定是奇数或偶数的堆吗？其实随便举一个例子就知道了，比如4堆石头，先手的人总是能控制自己一定拿到1，3或者2，4所以先手的一定是可以赢得，推广到N堆石头也同样适用

## [1039. 多边形三角剖分的最低得分](https://leetcode-cn.com/problems/minimum-score-triangulation-of-polygon/)

给定 N，想象一个凸 N 边多边形，其顶点按顺时针顺序依次标记为 `A[0], A[i], ..., A[N-1]`。

假设您将多边形剖分为 N-2 个三角形。对于每个三角形，该三角形的值是顶点标记的乘积，三角剖分的分数是进行三角剖分后所有 N-2 个三角形的值之和。

返回多边形进行三角剖分后可以得到的最低分。

**示例 1：**

```java
输入：[1,2,3]
输出：6
解释：多边形已经三角化，唯一三角形的分数为 6。
```

**实例2：**

![J8IK8U.png](https://s1.ax1x.com/2020/04/21/J8IK8U.png)

```java
输入：[3,7,4,5]
输出：144
解释：有两种三角剖分，可能得分分别为：3*7*5 + 4*5*7 = 245，或 3*4*5 + 3*4*7 = 144。最低分数为 144。
```

**解法一**

区间DP套路，枚举所有的区间和区间内的点，`区间内的点`和`区间左右端点`将整个区间又划分为两个`已知的子区间`，典型的区间dp

```java
//环形的结构
public int minScoreTriangulation(int[] A) {
    if(A==null || A.length<=0) return 0;
    int N=A.length;
    //从 0 ~ N-1 形成一个环
    //    1-3 
    //   /    \
    //  5      1
    //   \    /
    //    1—4
    // dp[left][right] 代表left~right区间形成的环的最小得分值
    int[][] dp=new int[N][N];
    for (int len=3;len<=N;len++) { //枚举长度,从3开始
        for (int left=0;left<=N-len;left++) { //枚举左端点
            //left+len-1<N
            int right=left+len-1;
            //init初始化区间值
            dp[left][right]=Integer.MAX_VALUE;
            for (int i=left+1;i<right;i++) { //枚举区间内的所有的点(不包括端点)),将环分割成左右两部分
                dp[left][right]=Math.min(dp[left][right],dp[left][i]+dp[i][right]+A[i]*A[left]*A[right]);
            }
        }
    }
    return dp[0][N-1];
}
```

## [516. 最长回文子序列](https://leetcode-cn.com/problems/longest-palindromic-subsequence/)

给定一个字符串s，找到其中最长的回文子序列。可以假设s的最大长度为1000。

**示例 1:**

```java
输入:

"bbbab"
输出:

4
一个可能的最长回文子序列为 "bbbb"。
```

**示例 2:**

```java
输入:

"cbbd"
输出:

2
一个可能的最长回文子序列为 "bb"。
```

**解法一**

动态规划，`dp[i][j]` 代表i~j的最长回文串

```java
public int longestPalindromeSubseq(String s) {
    if (s==null || s.length()<=0) {
        return 0;
    }
    int n=s.length();
    int[][] dp=new int[n][n]; //dp[i][j] 代表i~j的最长回文子串
    for (int i=0;i<n;i++) {
        dp[i][i]=1;
    }
    //bbbab
    for (int i=n-1;i>=0;i--) {
        for (int j=i+1;j<n;j++) { 
            if (s.charAt(i)==s.charAt(j)) {
                dp[i][j]=dp[i+1][j-1]+2;
            }else{
                dp[i][j]=Math.max(dp[i+1][j],dp[i][j-1]);
            }
        }
    }
    return dp[0][n-1];
}
```

核心的dp公式：

1. `s[i]==p[j]`，`dp[i][j]=dp[i+1][j-1]+2` 
2. `s[i]!=p[j]` ，`dp[i][j]=max(dp[i+1][j],dp[i][j-1])`

是不是有的眼熟？没错，和LCS的dp思路很相似

这里其实还有个需要注意的地方，就是两次循环的方向，我们要保证每次计算dp值的时候右值都是已经计算过的，所以需要调整循环的执行方向让 `i`从右往左，让`j`从`i`向右，这样i+1和j-1的dp就都是计算过的了

**UPDATE**

```java
//update: 2020.4.19
//区间dp写法,更加套路化，其实思路是一样的
public int longestPalindromeSubseq(String s) {
    if(s==null || s.length()<=0) return 0;
    int N=s.length();
    int[][] dp=new int[N][N];
    //base len=1
    for(int i=0;i<N;i++){
        dp[i][i]=1;
    }
    for(int len=2;len<=N;len++){
        for(int i=0;i<=N-len;i++){
            //j=i+len-1<N
            int j=i+len-1;
            if(s.charAt(i)==s.charAt(j)){
                dp[i][j]=dp[i+1][j-1]+2;
            }else{
                dp[i][j]=Math.max(dp[i+1][j],dp[i][j-1]);
            }
        }
    }
    return dp[0][N-1];
}
```

**解法二**

对比一下递归解法是真的简洁，没有那么多边界考虑

```java
//递归解法,自底向上
private Integer[][] cache=null;

public int longestPalindromeSubseq(String s) {
    if (s==null || s.length()<=0) {
        return 0;
    }
    int n=s.length();
    cache=new Integer[n][n];
    return helper(s,0,n-1);
}

public int helper(String s,int left,int right){
    if (left>right) {
        return 0;
    }
    if (left==right) {
        return 1;
    }
    if (cache[left][right]!=null) {
        return cache[left][right];
    }
    if (s.charAt(left) == s.charAt(right)) {
        return cache[left][right]=helper(s,left+1,right-1)+2; //由外向内缩进
    }
    return cache[left][right]=Math.max(helper(s,left+1,right),helper(s,left,right-1));
}
```
**解法三**

将问题转换为LCS的问题

```java
//LCS的解法
public int longestPalindromeSubseq(String s) {
    if (s==null || s.length()<=0) {
        return 0;
    }
    int n=s.length();
    int[][] dp=new int[n+1][n+1]; //dp[i][j] 代表s[0,i]~rs[0,j]的最长子序列
    String rs=new StringBuilder(s).reverse().toString();
    for (int i=1;i<=s.length();i++) {
        for (int j=1;j<=rs.length();j++) {
            if (s.charAt(i-1)==rs.charAt(j-1)) {
                dp[i][j]=dp[i-1][j-1]+1;
            }else{
                dp[i][j]=Math.max(dp[i-1][j],dp[i][j-1]);
            }
        }
    }
    return dp[n][n];
}
```
最长的回文序列其实就是求这个字符串和它翻转后的字符串的最长公共子序列！很妙的解法

## [1312. 让字符串成为回文串的最少插入次数](https://leetcode-cn.com/problems/minimum-insertion-steps-to-make-a-string-palindrome/)

给你一个字符串 s ，每一次操作你都可以在字符串的任意位置插入任意字符。

请你返回让 s 成为回文串的 **最少操作次数** 。

「回文串」是正读和反读都相同的字符串。

**示例 1：**

```java
输入：s = "zzazz"
输出：0
解释：字符串 "zzazz" 已经是回文串了，所以不需要做任何插入操作。
```

**示例 2：**

```java
输入：s = "mbadm"
输出：2
解释：字符串可变为 "mbdadbm" 或者 "mdbabdm" 。
```


**示例 3：**

```java
输入：s = "leetcode"
输出：5
解释：插入 5 个字符后字符串变为 "leetcodocteel" 。
```

**示例 4：**

```java
输入：s = "g"
输出：0
```


**示例 5：**

```java
输入：s = "no"
输出：1
```

**提示：**

- 1 <= s.length <= 500
- s 中所有字符都是小写字母。 

**解法一**

170周赛的压轴题，其实和前面的[516. 最长回文子序列](#516-最长回文子序列)是一样的。。。没啥好说的

```java
public int minInsertions(String s) {
    if (s==null || s.length()<=0) {
        return 0;
    }
    int n=s.length();
    int[][] dp=new int[n][n];
    for (int i=n-1;i>=0;i--) {
        for (int j=i+1;j<n;j++) {
            if (s.charAt(i) == s.charAt(j)) {
                dp[i][j]=dp[i+1][j-1];   
            }else{
                //不相等就可以在头部插入s[j]或者尾部插入s[i]
                //头部插入s[j]则子区间就是dp[i][j-1]
                //尾部插入s[i]则子区间就是dp[i+1][j]
                dp[i][j]=1+Math.min(dp[i+1][j],dp[i][j-1]);
            }
        }
    }
    return dp[0][n-1];
}
```
**解法二**

UPDATE：2020.5.18，区间dp的写法

```java
public int minInsertions(String s) {
    int[][] dp=new int[s.length()][s.length()];
    for(int len=2;len<=s.length();len++){
        for(int left=0;left+len-1<s.length();left++){
            int right=left+len-1;
            if(s.charAt(left)==s.charAt(right)){
                dp[left][right]=dp[left+1][right-1];
            }else{
                dp[left][right]=Math.min(dp[left][right-1],dp[left+1][right])+1;
            }
        }
    }
    return dp[0][s.length()-1];
}
```

这题也可以用LCS的解法解，`n-lcs[n][n]`就是最少的插入次数 

## [664. 奇怪的打印机](https://leetcode-cn.com/problems/strange-printer/)

有台奇怪的打印机有以下两个特殊要求：

打印机每次只能打印同一个字符序列。
每次可以在任意起始和结束位置打印新字符，并且会覆盖掉原来已有的字符。
给定一个只包含小写英文字母的字符串，你的任务是计算这个打印机打印它需要的最少次数。

**示例 1:**

```java
输入: "aaabbb"
输出: 2
解释: 首先打印 "aaa" 然后打印 "bbb"。
```

**示例 2:**

```java
输入: "aba"
输出: 2
解释: 首先打印 "aaa" 然后在第二个位置打印 "b" 覆盖掉原来的字符 'a'。
```

**提示:** 输入字符串的长度不会超过 100。

**解法一**

前一天看了大致的思路，隔了一天独立的写了一下，将前天思路的很多细节都想明白了

```java
public int strangePrinter(String s) {
    if(s==null || s.length()<=0) return 0;
    int N=s.length();
    int[][] dp=new int[N][N];
    for (int i=0;i<N;i++) {
        dp[i][i]=1;
    }
    //aa        1
    //aab       2
    //aaba      2
    //aabab     3
    //aababa    4
    for (int len=2;len<=N;len++) {
        for (int left=0;left+len-1<N;left++) { //左端点
            int right=left+len-1; //右端点
            //和前一个区间[left,right-1]的第一个字符相等,可以直接顺带打印出来
            if(s.charAt(right)==s.charAt(left)){ 
                //至少是dp[left][right-1]
                dp[left][right]=dp[left][right-1];
                continue;
            }
            //最多是前一个区间+1(直接打印right位置的字符)
            dp[left][right]=dp[left][right-1]+1; 
            //枚举[left,right)中的所有字符,如果有和right相等的就考虑从这个位置分割整个字符
            //比如 abc b 这个最后的b就可以连同前面的bc一起顺带打印出来
            //所以当前区间的值就是可以是dp[a][a]+dp[b][c],我们从所有的这种情况中取最小值就ok了
            for (int i=left;i<right;i++) { 
                if(s.charAt(right)==s.charAt(i)){
                    dp[left][right]=Math.min(dp[left][i-1]+dp[i][right-1],dp[left][right]);
                }
            }
        }
    }
    return dp[0][N-1];
}
```
`dp[left][right]`代表left到right的最少打印次数，然后我们要理解一点：`dp[left][right]` 其实只可能有两个取值，`dp[left][right-1]`或者`dp[left][right-1]+1`，我们求最小打印次数其实就是希望增加一个字符后打印次数不变

> **eg:**  abc b 这个最后的b就可以连同前面的bc一起顺带打印出来，打印次数不变

枚举所有的区间`[left,right]`，然后枚举区间`[left,right-1]`内的字符，如果区间`[left,right-1]`内的某个字符`i`和`right`相等，那么`right`就可以跟随`[i,right-1]`一起顺带打印出来，这样该区间的dp值为

`dp[left][right]=dp[left][i-1]+dp[i][right-1]`
这样就有可能让打印次数不变，我们求一下所有情况的最小值就可以了

> 这题的代码还可以更简洁，不过我懒得改了，感觉上面的代码已经很清晰了
>
> ps:  这几天做了不少有点难度的DP题，发现确实自己对边界的处理，递推公式的理解有了一点提升，加油💪

## _数位DP_

## [233. 数字 1 的个数](https://leetcode-cn.com/problems/number-of-digit-one/)

给定一个整数 n，计算所有小于等于 n 的非负整数中数字 1 出现的个数。

**示例:**

```java
输入: 13
输出: 6 
解释: 数字 1 出现在以下数字中: 1, 10, 11, 12, 13 。
```

**解法一**

这个题其实困扰了我很长时间，之前看了好几次都放弃了，感觉有些找规律的数学解法太难想到了，所以这里直接采用**数位DP**，相对于其他的解法，这种解法会更加套路模板化
```java
//dp[pos][sumOne]代表从高位枚举到pos位置，前面1出现的个数sumOne
int [][] dp=null;

public int countDigitOne(int n) {
    int len=0;
    int[] num=new int[64]; //64位肯定够了
    while(n!=0){
        num[len++]=n%10;
        n/=10;
    }
    dp=new int[len+1][len+1];
    //从高位向低位枚举
    return dfs(num,len-1,0,true,true);
}

public int dfs(int[] num,int pos,int sumOne,boolean leadZero,boolean limit){
    //枚举完所有的数位，直接返回sumOne
    if(pos==-1) return sumOne;
    //状态重叠，状态要完全一致
    if(!leadZero && !limit && dp[pos][sumOne]!=0) return dp[pos][sumOne];
    int res=0;
    int up=limit?num[pos]:9;
    for (int i=0;i<=up;i++) {
        res+=dfs(num,pos-1,sumOne+(i==1?1:0),leadZero&&i==0,i==up&&limit);
    }
    if(!leadZero && !limit) dp[pos][sumOne]=res;
    return res;
}
```

优化下代码，前导0对状态存取没有影响，有**前导0**和**没有前导0**的`sumOne`肯定不一样

```java
//简化下代码，前导0其实没有影响
//dp[pos][sumOne]代表从高位枚举到pos位置，前面1的个数sumOne
int [][] dp=null;

public int countDigitOne(int n) {
    int len=0;
    int[] num=new int[64]; //64位肯定够了
    while(n!=0){
        num[len++]=n%10;
        n/=10;
    }
    dp=new int[len+1][len+1];
    //从高位向低位枚举
    return dfs(num,len-1,0,true);
}

public int dfs(int[] num,int pos,int sumOne,boolean limit){
    //枚举完所有的数位，直接返回sumOne
    if(pos==-1) return sumOne;
    //状态重叠，状态要完全一致
    if(!limit && dp[pos][sumOne]!=0) return dp[pos][sumOne];
    int res=0;
    int up=limit?num[pos]:9;
    for (int i=0;i<=up;i++) {
        res+=dfs(num,pos-1,sumOne+(i==1?1:0),i==up&&limit);
    }
    if(!limit) dp[pos][sumOne]=res;
    return res;
}
```

**数位DP**的概念和模板其实我也是下午在网上查的，然后找到了两个写的比较好的文章 [数位dp总结 之 从入门到模板](https://blog.csdn.net/wust_zzwh/article/details/52100392)，[数字组成的奥妙——数位dp](https://www.luogu.com.cn/blog/virus2017/shuweidp) 以后遇到类似的题又多了一种解法（数学的解法至今仍没学会🤣）只有这一题想搞懂**数位DP**还是不太够，这题还是比较简单的，还是要多做题，多总结才能慢慢体会，lc上好像数位dp好像并不多，过两天都做一下试试

## [1420. 生成数组](https://leetcode-cn.com/problems/build-array-where-you-can-find-the-maximum-exactly-k-comparisons/) 

给你三个整数 `n`、`m` 和 `k` 。下图描述的算法用于找出正整数数组中最大的元素。

![JdU4w8.png](https://s1.ax1x.com/2020/04/23/JdU4w8.png)

请你生成一个具有下述属性的数组 arr ：

- arr 中有 n 个整数。
- 1 <= arr[i] <= m 其中 (0 <= i < n) 。
- 将上面提到的算法应用于 arr ，search_cost 的值等于 k 。

返回上述条件下生成数组 arr 的 方法数 ，由于答案可能会很大，所以 必须 对 10^9 + 7 取余。

**示例 1：**

```java
输入：n = 2, m = 3, k = 1
输出：6
解释：可能的数组分别为 [1, 1], [2, 1], [2, 2], [3, 1], [3, 2] [3, 3]
```


**示例 2：**

```java
输入：n = 5, m = 2, k = 3
输出：0
解释：没有数组可以满足上述条件
```


**示例 3：**

```java
输入：n = 9, m = 1, k = 1
输出：1
解释：可能的数组只有 [1, 1, 1, 1, 1, 1, 1, 1, 1]
```


**示例 4：**

```java
输入：n = 50, m = 100, k = 25
输出：34549172
解释：不要忘了对 1000000007 取余
```


**示例 5：**

```java
输入：n = 37, m = 17, k = 7
输出：418930126
```

**提示：**

- 1 <= n <= 50
- 1 <= m <= 100
- 0 <= k <= n 

**解法一**

185周赛T4，类似于数位DP，记忆化递归的方式很好理解

```java
int mod=(int)1e9+7;

Integer [][][] dp=null;

public int numOfArrays(int n, int m, int k) {
    dp=new Integer[n+1][m+1][k+1];
    return dfs(n,0,m,k);
}

public int dfs(int pos,int preMax,int m,int k){
    if(k==0 && pos==0) return dp[pos][preMax][k]=1;
    //k=0但是pos不为0并没有结束,还可以继续找比preMax小的元素
    //if(pos==0 || k==0) return 0; 
    if(pos==0 || k<0) return 0;
    if(dp[pos][preMax][k]!=null){
        return dp[pos][preMax][k];
    }
    long res=0;
    for (int i=1;i<=m;i++) {
        if(i>preMax){ //大于preMax,代价-1,更新最大值
            res=(res+dfs(pos-1,i,m,k-1))%mod;
        }else{ //小于preMax,代价不变,最大值不变
            res=(res+dfs(pos-1,preMax,m,k))%mod;
        }
    }
    return dp[pos][preMax][k]=(int)(res%mod);
}
```
一开始没注意给dp数组赋初始值，直接按照0判断的，结果T了半天，我都懵了了，后来才意识到，很多状态的结果就是0，你直接按照0判断，很多相同的dp值为0的状态都没有被取出来用

**优化**

```java
int mod=(int)1e9+7;

Long [][][] dp=null;

public int numOfArrays(int n, int m, int k) {
    dp=new Long[n+1][m+1][k+1];
    return (int)dfs(n,0,m,k);
}

public long dfs(int pos,int preMax,int m,int k){
    if(k==0 && pos==0) return dp[pos][preMax][k]=1L;
    //k=0但是pos不为0并没有结束,还可以继续找比preMax小的元素
    //if(pos==0 || k==0) return 0; 
    if(pos==0 || k<0) return 0L;
    if(dp[pos][preMax][k]!=null){
        return dp[pos][preMax][k];
    }
    long res=0;
    //小于等于preMax的部分可以通过dfs(pos-1,preMax,m,k)算出来,减少递归次数
    res=(res+(preMax*dfs(pos-1,preMax,m,k))%mod)%mod;
    for (int i=preMax+1;i<=m;i++) {
        res=(res+dfs(pos-1,i,m,k-1))%mod;
    }
    return dp[pos][preMax][k]=res%mod;
}
```
**解法二**

递推的方式，其实思想是一样的，但是个人感觉并不是特别容易直接想出来

```java
public int numOfArrays(int n, int m, int k) {
    int mod=1000_000_007;

    long[][][] dp=new long[n+1][m+1][k+1];
    for(int i=1;i<=m;i++){
        dp[1][i][1]=1;
    }
    for(int i=2;i<=n;i++){
        for(int j=1;j<=m;j++){
            for(int cost=1;cost<=k;cost++){
                //新增加的元素小于等于j也就是原数组的最大值,对应上面递归的i<=preMax
                //所以新增加的元素在新数组的末尾有1~j种选择
                dp[i][j][cost]=(j*dp[i-1][j][cost])%mod;
                for(int nm=1;nm<j;nm++){
                    //新增加的元素大于原数组的最大值j,对应上面递归的i>preMax
                    dp[i][j][cost]=(dp[i][j][cost]+dp[i-1][nm][cost-1])%mod;
                }
            }
        }
    }
    long res=0;
    for(int i=1;i<=m;i++){
        res+=dp[n][i][k];
        res%=mod;
    }
    return (int)(res%mod);
}
```
## _状压DP_

## [464. 我能赢吗](https://leetcode-cn.com/problems/can-i-win/)

在 "100 game" 这个游戏中，两名玩家轮流选择从 1 到 10 的任意整数，累计整数和，先使得累计整数和达到 100 的玩家，即为胜者。

如果我们将游戏规则改为 “玩家不能重复使用整数” 呢？

例如，两个玩家可以轮流从公共整数池中抽取从 1 到 15 的整数（不放回），直到累计整数和 >= 100。

给定一个整数 `maxChoosableInteger` （整数池中可选择的最大数）和另一个整数 `desiredTotal`（累计和），判断先出手的玩家是否能稳赢（假设两位玩家游戏时都表现最佳）？

你可以假设 `maxChoosableInteger` 不会大于 20， `desiredTotal` 不会大于 300。

**示例：**

```java
输入：
maxChoosableInteger = 10
desiredTotal = 11

输出：
false

解释：
无论第一个玩家选择哪个整数，他都会失败。
第一个玩家可以选择从 1 到 10 的整数。
如果第一个玩家选择 1，那么第二个玩家只能选择从 2 到 10 的整数。
第二个玩家可以通过选择整数 10（那么累积和为 11 >= desiredTotal），从而取得胜利.
同样地，第一个玩家选择任意其他整数，第二个玩家都会赢。
```

**解法一**

从 [力扣上的 DP 问题分类汇总](https://leetcode-cn.com/circle/article/NfHhXD/) 过来的，所以首先知道这个是状压DP，但是菜鸡的我并不知道状压DP是啥🤣，所以先写了个暴力的搜索，然后不出意料的t了

```java
//TLE case 20 210 41/45
public boolean canIWin(int maxChoosableInteger, int desiredTotal) {
    if(desiredTotal<=maxChoosableInteger) return true;
    if((1+maxChoosableInteger)*(maxChoosableInteger)/2<desiredTotal) return false;
    boolean[] visit=new boolean[maxChoosableInteger+1];
    return dfs(maxChoosableInteger,desiredTotal,visit);
}

public boolean dfs(int max,int total,boolean[] visit){
    if(total<=0) return false; //前人已经拿完了
    for (int i=1;i<=max;i++) {
        if(!visit[i]){
            visit[i]=true;
            boolean temp=dfs(max,total-i,visit);
            visit[i]=false;
            if(!temp){
                return true;
            }
        }
    }
    return false;
}
```
注意这个写法的返回时机，有些语句是不能合并的，比如下面这样就是错的（别问我为啥知道的🤣）

```java
public boolean dfs(int max,int total,boolean[] visit){
    if(total<=0) return false;
    for (int i=1;i<=max;i++) {
        if(!visit[i]){
            visit[i]=true;
            if(!dfs(max,total-i,visit)){
                return true;
            }
            visit[i]=false;
        }
    }
    return false;
}
```

像上面这样写乍一看好像没问题，对比一下两种写法，其实就是回溯`visit`的时机不同，第一个是另一个人`dfs` 不管成功与否都会回溯状态，而第二种是另一个人`dfs` 成功后才会回溯状态，失败后就不会回溯状态了，但是在这题中不管另一个人失败与否都需要回溯状态，首先另一个人成功后我肯定是要回溯，另一个人成功意味着我失败了，我还要继续尝试找必胜的选取方法，但是另一个人失败后还是需要回溯状态，仔细想想，下一层dfs(另一个人)失败是因为下下一层dfs(我)成功了，我在那一次成功后并没有回溯`visit`，这样下一层（另一个人）就用不了被下下层占用的数字，就无法枚举所有情况，自然就错了

> 各位大佬见谅，表达能力有限，上面的都是我胡言乱语，大佬们跳过就行了

**解法二**

根据昨天写的一个数位DP，加上自己的yy，写了个状压DP出来了🤣，上面的解法明显会有很多的重复解，而确定是否重复的关键其实就是数字的选取情况，上面我们是用的一个`visit`数组来记录这选取的情况，要直接通过这个`visit`来判断状态是否相同是很耗时耗力的，所以就引入了状态压缩，我们可以用二进制的第`i`位的0或者1来表示`i`这个数字的选取与否，这样所有数字的选取状态就可以用一个数来很方便的表示，题目说了不超过20位，所以这里就可以用一个`int`来表示状态`state`，通过`state`来判断状态是否一致，进而进行记忆化的存取

```java
Boolean[] dp=null; //用Boolean比较方便判断是否记忆化

public boolean canIWin(int maxChoosableInteger, int desiredTotal) {
    //小于最大值先手可以直接拿
    if(desiredTotal<=maxChoosableInteger) return true;
    //前N项和还不够desiredTotal
    if((1+maxChoosableInteger)*(maxChoosableInteger)/2<desiredTotal) return false;
    //20位二进制 1<<21
    dp=new Boolean[1<<21];
    return dfs(maxChoosableInteger,desiredTotal,0);
}

public boolean dfs(int max,int total,int state){
    if(total<=0) return false; //前人已经拿完了
    if(dp[state]!=null){
        return dp[state];
    }
    for (int i=max;i>=1;i--) {
        //参数传递的，就不用回溯了，代码变的简洁多了
        if((state>>i&1)==0 && !dfs(max,total-i,state|(1<<i))){
            return dp[state]=true;
        }
    }
    return dp[state]=false;
}
```
其实一开始开了个二维的 `new Boolean[desiredTotal] [1<<21]`，然后就直接MLE了😅 太菜了hahaha，然后看了下，发现`state`一样的话，`total`肯定是一样的（反过来就不一定了），所以只需要一维的就可以了

其实也可以用`int[]`做状压

```java
//int做状压，似乎性能没有提升
int[] dp=null; //0:未初始化 1:true 2:false

public boolean canIWin(int maxChoosableInteger, int desiredTotal) {
    //小于最大值先手可以直接拿
    if(desiredTotal<=maxChoosableInteger) return true;
    //前N项和还不够desiredTotal
    if((1+maxChoosableInteger)*(maxChoosableInteger)/2<desiredTotal) return false;
    //20位二进制 1<<21
    dp=new int[1<<21];
    return dfs(maxChoosableInteger,desiredTotal,0);
}

public boolean dfs(int max,int total,int state){
    if(dp[state]!=0){
        return dp[state]==1;
    }
    for (int i=max;i>=1;i--) {
        //参数传递的，就不用回溯了，代码变的简洁多了
        if((state>>i&1)==0 && (i>=total || !dfs(max,total-i,state|(1<<i)))){
            dp[state]=1;
            return true;
        }
    }
    dp[state]=2;
    return false;
}
```

