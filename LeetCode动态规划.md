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

简单的dp题，dp方程很容易得到，`dp[i]=max(dp[i-2]+nums[i],dp[i-1])`

```java
//  Max[i]=max(nums[i-2]+nums[i],nums[i-1])
public int rob(int[] nums) {
    if(nums==null||nums.length<=0){
        return 0;
    }
    if(nums.length==1){
        return nums[0];
    }
    int []dp=new int[nums.length];
    dp[0]=nums[0];
    dp[1]=nums[1]>nums[0]?nums[1]:nums[0];
    int i;
    for (i=2;i<nums.length;i++) {
        dp[i]=Math.max(dp[i-2]+nums[i],dp[i-1]);
    }
    return dp[i-1];
}
```

时间空间都是O(N)，需要注意边界条件，这题还有另一种写法，空间复杂度是O(1)的

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

## [337. 打家劫舍 III](https://leetcode-cn.com/problems/house-robber-iii/)

在上次打劫完一条街道之后和一圈房屋后，小偷又发现了一个新的可行窃的地区。这个地区只有一个入口，我们称之为“根”。 除了“根”之外，每栋房子有且只有一个“父“房子与之相连。一番侦察之后，聪明的小偷意识到“这个地方的所有房屋的排列类似于一棵二叉树”。 如果两个直接相连的房子在同一天晚上被打劫，房屋将自动报警。

计算在不触动警报的情况下，小偷一晚能够盗取的最高金额。

**示例 1:**

```java
输入: [3,2,3,null,3,null,1]
 	 3
	/ \
   2   3
    \   \ 
     3   1

输出: 7 
解释: 小偷一晚能够盗取的最高金额 = 3 + 3 + 1 = 7.
```

**示例 2:**

~~~java
输入: [3,4,5,1,3,null,1]

 	 3
	/ \
   4   5
  / \   \ 
 1   3   1

输出: 9
解释: 小偷一晚能够盗取的最高金额 = 4 + 5 = 9.
~~~

**解法一**

暴力递归，应该还是写得出来

```java
//AC了,但是效率很低
//可以用hashMap缓存一下每个节点rob的值,但是没必要
public int rob(TreeNode root) {
    return tryRob(root);
}

public int tryRob(TreeNode root) {
    if (root==null) {
        return 0;
    }
    if (root.left==null && root.right==null) {
        return root.val;
    }
    //偷取当前节点
    int res=root.val;
    if (root.left!=null) {
        res+=tryRob(root.left.left)+tryRob(root.left.right);
    }
    if (root.right!=null) {
        res+=tryRob(root.right.left)+tryRob(root.right.right);
    }
    //不偷当前节点
    int res2=0;
    res2=tryRob(root.left)+tryRob(root.right);
    return Math.max(res,res2);
}
```
**解法二**

看评论区说是啥树状dp ? 知识盲区了hahaha

```java
public int rob(TreeNode root) {
    int[] res=tryRob(root);
    return Math.max(res[0],res[1]);
}

//树形dp???
//看的懂，但是肯定写不出来 。。。。
public int[] tryRob(TreeNode root) {
    int[] dp=new int[2];
    if (root==null) {
        return dp;
    }

    int[] left=tryRob(root.left);
    int[] right=tryRob(root.right);
    //不包含当前节点的最大值
    dp[0]=Math.max(left[0],left[1])+Math.max(right[0],right[1]);
    //包含当前节点的最大值
    dp[1]=left[0]+right[0]+root.val;
    return dp;
}
```
不是我吹，就这样的题目，再遇见多少次我都写不出来这样的解（笑

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

------

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

------

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

------

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
这里我又发现了leetcode的case有问题，我开始没有缓存前一天的empty值，直接将empty带到hold去了，结果还过了。。。不过我懒得去想case了，反正提交了leecode也不会理我😅

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

递推公式：`dp[i][j]=1+min(dp[i-1][j-1],dp[i-1][j],dp[i][j-1])`  嗯，想不到就很可惜

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
2.  `s[i]!=p[j]` ，`dp[i][j]=max(dp[i+1][j],dp[i][j-1])`

是不是有的眼熟？没错，和LCS的dp思路很相似

这里其实还有个需要注意的地方，就是两次循环的方向，我们要保证每次计算dp值的时候右值都是已经计算过的，所以需要调整循环的执行方向让 `i`从右往左，让`j`从`i`向右，这样i+1和j-1的dp就都是计算过的了

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

170周赛的压轴题，其实和前面的最长回文是一样的。。。没啥好说的

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
                //不相等就可以在中间插入一个字符 +1
                dp[i][j]=1+Math.min(dp[i+1][j],dp[i][j-1]);
            }
        }
    }
    return dp[0][n-1];
}
```
这题也可以用LCS的解法解，`n-lcs[n][n]`就是最少的插入次数 