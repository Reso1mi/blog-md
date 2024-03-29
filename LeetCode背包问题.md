---
title: LeetCode 背包问题
tags:
  - LeetCode
  - 背包
categories:
  - 算法
date: 2019/11/29
cover: 'http://static.imlgw.top/blog/20191117/lAgAEVtKwR4j.jpg?imageslim'
abbrlink: f68a53d9
---

> 从 [动态规划专题](http://imlgw.top/2019/09/01/leetcode-dong-tai-gui-hua/) 中抽取出来的

## [92. 背包问题（lintCode）](https://www.lintcode.com/problem/backpack/description)

在 n 个物品中挑选若干物品装入背包，最多能装多满？假设背包的大小为 m，每个物品的大小为 A[i]

- 你不可以将物品进行切割

**样例**

```java
样例 1:
	输入：[3,4,8,5], backpack size=10
	输出：9
样例 2:
	输入：[2,3,5,7], backpack size=12
	输出：12
```

**挑战**

- O(n x m) time and O(m) memory.

- O(n x m) memory is also acceptable if you do not know how to optimize memory.

**解法一**

记忆化递归，对于每个元素，有两种选择，装或者不装

```java
//用 Integer[][], 空间会超空间。lintCode 好严格
int [][] cache=null;    

public int backPack2(int m,int[] A) {
    cache=new int[A.length+1][m+1];
    for (int i=0;i<A.length;i++) {
        Arrays.fill(cache[i],-1);   
    }
    if (A==null || A.length<=0) {
        return 0;
    }
    return putPack(m,A,A.length-1);
}

//将 A[index,A.len-1] 范围内的元素装进大小为 m 的背包的最大收益
public int putPack(int m,int[] A,int index) {
    //index==0 的时候不应该返回=0 代表第一个，是可以装的
    //对于 m 也是一样，这种边界思考一下 m 就等于 0，或者就只有一个元素，index 就等于 0 这种特例就可以
    //只要这种特例是正确的那么整个递归就是正确的，并不需要去思考整个递归的结束条件
    if (index<0 || m<=0) {
        return 0;
    }
    if (cache[index][m]!=-1) {
        return cache[index][m];
    }
    //不装 index 位置的元素
    int res=putPack(m,A,index-1);
    if (A[index]<=m) {
        //说明可以装下 index 位置的元素，所以我们将 index 位置的元素装进去试试看
        //然后求出剩下的空间还最多能装多少，最后求是装 index 收益大还是不装 index 收益大
        res=Math.max(res,A[index]+putPack(m-A[index],A,index-1));
    }
    cache[index][m]=res;
    return res;
}
```

暴力递归的时间复杂度将会是`O((2^N)*N)`

其实整个递归的思路是很清晰明白的，对于每个元素，有两种情况，这也是之所以称之为 0-1 背包的原因

- 不选的话，背包的容量不变，改变为问题`putPack(m,A,index-1)`
- 选的话，背包的容量变小，改变为问题`putPack(m-A[index],A,index-1)+A[index]`

到底选还是不选，取决于两种方案哪一种更好，我们要求的，就是这个最好的方案，知道了这样的递推关系后我们就可以很容易的写出递归方程，这里在递归的过程中有可能会产生重叠的子问题（其实这里我还纠结了好一会儿，我一直感觉没有重叠的子问题，后来画一下递归树就明白了，只是重叠的不明显），所以我们可以通过缓存每次计算的结果来进行记忆化递归，整体的时间复杂度应该是`O(2^N)`，空间`O(M*N)`显然不是我们想要的结果

> 这里一开始我是想用`Integer[][]`的数组，然后就不用赋初始值，判断不为 null 就行，结果空间溢出了。lintCode 好严格，换成`int[][]`然后赋个初始值就过了

**解法二**

动态规划解法，在讲解之前，我们用一个二维表来分析下整个递推的过程

物品列表（样例 1），因为这题价值就是重量，所以 w 和 v 是一样的

![mark](http://static.imlgw.top/blog/20200227/XIvS0eAf1FqA.png?imageslim)

**DpTable（样例 1）**

![mark](http://static.imlgw.top/blog/20200227/YkPisR3fx1wa.png?imageslim)

一行一行的看，从左到右，`dp[index][m]`代表 **背包总容量不超过 m 的情况下，考虑装入`[0,index]`中的元素能获得最大收益**，比如`dp[1][7]`代表的就是背包总容量不超过 7 的情况下，考虑装入`[0,1]` 范围内的元素所能获得的最大收益，人脑思考结果自然是 7 了，下面我们分析下如果 dp 推出这个结果

前面我们已经分析过 0-1 背包的递归过程，每个元素面临两个选择，这里也一样

`dp[1][7]`如果我们选择不装入当前 index 位置的元素的话，那么最大收益就是`dp[0][7]=3`这一点应该没啥疑问

如果我们考虑装入当前 index 位置的元素的话，m 肯定会减小，那么所获得的最大收益就应该是`A[index]+dp[0][7-4]=7` 

> 注意这里当前 index 的值都是依赖于上一层`index-1`的计算结果的，也就是依赖于上一次`m,[0,index-1]`最大值的结果，所以我们需要手动的初始化第一层的值）

最后我们得到的核心状态方程就是下面这样的

`dp[index][m]=max(dp[index-1][m],A[index]+dp[index-1][j-A[index]])`

然后我们根据这个很容易就可以写出 dp 的解法

```java
//二维动态规划
public int backPack(int m,int[] A) {
    if (A==null || A.length<=0) {
        return 0;
    }
    int[][] dp=new int[A.length][m+1];
    for (int i=0;i<A.length;i++) {
        for (int j=0;j<=m;j++) {
            if (j==0) {//初始化第一列
                dp[i][j]=0;
            }else if (i==0) {//初始化第一行
                dp[i][j]=j-A[i]>=0?A[i]:0;
            }else if (i>0) {
                dp[i][j]=j-A[i]>=0?Math.max(dp[i-1][j],dp[i-1][j-A[i]]+A[i]):dp[i-1][j];
            }
        }
    }
    return dp[A.length-1][m];
}
```
当然我们肯定是不满足于这种二维的 dp 的，所以我们还得优化下空间，这里每一层都只依赖于上一层的结果，所以我么很容易就可以改成一维的，当然这里还有个小坑，如果直接按照上面的代码来改的话就是错的，我们先看看正确的改法

```java
public int backPack4(int m,int[] A) {
    if (A==null || A.length<=0) {
        return 0;
    }
    int[] dp=new int[m+1];
    for (int i=0;i<A.length;i++) {
        for (int j=m;j>=0;j--) {//从右向左，避免覆盖
            if (j==0) {//初始化第一列
                dp[j]=0;
            }else if (i==0) {//初始化第一行
                dp[j]= j-A[i]>=0?A[i]:0;
            }else{
                dp[j]=j-A[i]>=0?Math.max(dp[j],dp[j-A[i]]+A[i]):dp[j];
            }
        }
    }
    return dp[m];
}
```
可以看到，我们的内层循环不再是从左往右，而是从右往左，这样的好处就是避免了`dp[j-A[i]]`已经被**当前内层循环前面的元素**覆盖的尴尬情况，结合上面的表推一下就知道了

**解法三**

算是对之前代码的优化吧，之前写的乱七八糟的

```java
//代码优化
public int backPack(int m,int[] A) {
    if (A==null || A.length<=0) {
        return 0;
    }
    int[] dp=new int[m+1];
    for (int i=0;i<A.length;i++) {
        for (int j=m;j>=A[i];j--) {
            dp[j]=Math.max(dp[j],dp[j-A[i]]+A[i]);
        }
    }
    return dp[m];
}
```
## [完全背包问题（acwing）](https://www.acwing.com/problem/content/description/3/) 

有 N 种物品和一个容量是 V 的背包，每种物品都有无限件可用。

第 i 种物品的体积是 vi，价值是 wi。

求解将哪些物品装入背包，可使这些物品的总体积不超过背包容量，且总价值最大。
输出最大价值。

**输入格式**

第一行两个整数，N，V 用空格隔开，分别表示物品种数和背包容积。

接下来有 N 行，每行两个整数 vi,wi 用空格隔开，分别表示第 i 种物品的体积和价值。

**输出格式**

输出一个整数，表示最大价值。

**数据范围**

0<N,V≤10000<N,V≤1000
0<vi,wi≤10000<vi,wi≤1000

**输入样例**

```java
4 5
1 2
2 4
3 4
4 5
```

**输出样例：**

```java
10
```

**解法一**

相比 01 背包交换了内循环的顺序就 ok 了，当然也可以将每个物品拆分，不过复杂度会变高

```java
import java.util.*;
public class Main{
    public static void main(String[] args){
        Scanner sc=new Scanner(System.in);
        int N=sc.nextInt();
        int V=sc.nextInt();
        int[] dp=new int[V+1];
        for(int i=0;i<N;i++){
            int vi=sc.nextInt();
            int wi=sc.nextInt();
            for(int j=vi;j<=V;j++) {
                dp[j]=Math.max(dp[j],dp[j-vi]+wi);
            }
        }
        System.out.println(dp[V]);
    }
}
```

其实这个结论要直接理解还是有点难懂的，具体的推导过程可以看下面的 [零钱兑换](#322-零钱兑换)

## [416. 分割等和子集](https://leetcode-cn.com/problems/partition-equal-subset-sum/)

给定一个只包含正整数的非空数组。是否可以将这个数组分割成两个子集，使得两个子集的元素和相等。

**注意：**

1. 每个数组中的元素不会超过 100
2. 数组的大小不会超过 200

**示例 1:**

```java
输入：[1, 5, 11, 5]

输出：true

解释：数组可以分割成 [1, 5, 5] 和 [11].
```

**示例 2:**

```
输入：[1, 2, 3, 5]

输出：false

解释：数组不能分割成两个元素和相等的子集。
```

**解法一**

现在递归写起来已经有点感觉了，类似的题基本上都能写出记忆化递归的方法来

```java
//记忆化递归 37ms 44%, 开始慢是因为 stream 的原因
Boolean[][] cache=null;

public boolean canPartition(int[] nums) {
    if (nums==null || nums.length<=0) return false;
    //int sum=Arrays.stream(nums).sum();
    int sum=0;
    for (int e:nums) sum+=e; //求和
    cache=new Boolean[nums.length][sum+1];
    if (sum%2!=0) {
        return false;
    }
    return partition(nums,0,0,sum/2);
}

//尝试添加 [0,index] 位置的元素，看能否使得 half=sum （这里其实应该直接在 sum 上减，看能不能减为 0)
public boolean partition(int[] nums,int index,int half,int sum) {
    if (index==nums.length) {
        return false;
    }
    if (cache[index][half]!=null) {
        return cache[index][half];
    }

    if (half==sum) {
        return true;
    }
    cache[index][half]=partition(nums,index+1,half,sum) || 
        (half<sum&&partition(nums,index+1,half+nums[index],sum));
    return cache[index][half];
}
```
**解法二**

动态规划，依然是典型的背包问题，可以理解为用 nums 中的元素，填满 sum/2 容量大小的背包，递推公式

 `dp[i][j] =dp[i-1][j] || dp[i-1][j-nums[i]]`  选当前元素和不选当前元素，有一个能填满就 ok

`dp[i][j]` 含义为：考虑`[0,i]` 范围内的元素，能否恰好装满 `j`大小的容器

```java
//二维 dp
public boolean canPartition(int[] nums) {
    if (nums==null || nums.length<=0) return false;
    //int sum=Arrays.stream(nums).sum(); 用 stream 好慢
    int sum=0;
    for (int e:nums) sum+=e; //求和
    if (sum%2!=0) {
        return false;
    }
    int half=sum/2;
    //dp[i][j] 的含义是从 [0,i] 中选取元素，能否刚好填满 j
    boolean[][] dp=new boolean[nums.length][half+1];
    for (int j=0;j<=half;j++) {
        dp[0][j]= nums[0]==j;
    }
    for (int i=1;i<nums.length;i++) {
        for (int j=0;j<=half;j++) {
            dp[i][j]= j>=nums[i]?dp[i-1][j] || dp[i-1][j-nums[i]]:dp[i-1][j];
        }
        //如果在某个位置（每行最后一个）已经刚好填满了就直接返回
        if (dp[i][half]) {
            return true;
        }
    }
    return dp[nums.length-1][half];
}
```
空间上的优化

```java
public boolean canPartition(int[] nums) {
    if (nums==null || nums.length<=0) return false;
    int sum=0;
    for (int e:nums) sum+=e; //求和
    if (sum%2!=0) {
        return false;
    }
    int half=sum/2;
    //dp[j] 的含义是从 [0,i] 中选取元素，能否刚好填满 j
    boolean[] dp=new boolean[half+1];
    for (int j=0;j<=half;j++) {
        dp[j]= nums[0]==j;
    }

    for (int i=1;i<nums.length;i++) {
        for (int j=half;j>=nums[i];j--) {
            //dp[i][j]= j>=nums[i]?dp[i-1][j] || dp[i-1][j-nums[i]]:dp[i-1][j];
            dp[j]=dp[j]||dp[j-nums[i]];
        }
        if (dp[half]) {
            return true;
        }
    }
    return dp[half];
}
```
## [322. 零钱兑换](https://leetcode-cn.com/problems/coin-change/)

给定不同面额的硬币 coins 和一个总金额 amount。编写一个函数来计算可以凑成总金额所需的最少的硬币个数。如果没有任何一种硬币组合能组成总金额，返回 -1。

**示例 1:**

```java
输入：coins = [1, 2, 5], amount = 11
输出：3 
解释：11 = 5 + 5 + 1
```

**示例 2:**

```java
输入：coins = [2], amount = 3
输出：-1
```

**说明：**
你可以认为每种硬币的数量是无限的

**解法一**

其实就是 dfs，我最开始就是写的 dfs 只不过时间复杂度太高，没做记忆化，这里其实一开始做了记忆化也一直没跑过，一直超时，最后给的 case 是 6249 好像也不算很大吧，然后我后来把`fill` 填充数组删了，用 Integer 就跑过了。

```java
//记忆化递归 AC 50%左右
private Integer[] cache=null;

public int coinChange2(int[] coins,int amount){
    cache=new Integer[amount+1];
    //Arrays.fill(cache,-1); 这里 fill 直接 tle 了。
    return takeCoins(coins,amount);
}

public int takeCoins(int[] coins, int amount) {
    if (amount==0) {
        return 0;
    }
    if (cache[amount]!=null) {
        return cache[amount];
    }
    //int t1=coins(coins,amount,index+1);
    int res=Integer.MAX_VALUE;
    for (int i=0;i<coins.length;i++) {
        if (amount<coins[i]) continue;
        int sub=takeCoins(coins,amount-coins[i]);
        if (sub!=-1) {
            res=Math.min(sub+1,res);
        }
    }
    cache[amount]= res==Integer.MAX_VALUE?-1:res;
    return cache[amount];
}
```
**解法二**

动态规划，二维 dp，注意这里其实和前面的背包问题就有区别了，这里实际上就是个`无限背包`问题，因为这里的硬币是无限的，每个面值的硬币都可以重复的选取

**DPTable**

![mark](http://static.imlgw.top/blog/20200227/ekmjFAHgA2K4.png?imageslim)

**状态定义**

这里`dp[i][j]` 的含义为：**考虑`[0，i]` 范围内的元素，能凑成 `j` 所需的最少硬币数**，和之前的 01 背包问题状态定义没什么区别

**状态方程**

首先明确一点，这里我们对第`coins[i]`个硬币有两种选择 

1. 不拿 
2.  拿，拿 1~k 个 (k 为硬币个数的限制，这里没有限制，所以是无穷大）

进而我们可以的到状态转移的方程：

`f[i][j] = min(f[i-1][j], f[i-1][j-c]+1, f[i-1][j-2*c]+2, ..., f[i-1][j-k*c]+k)`

但是这个方程有很多计算是重复的

`f[i][j-c]=min(f[i-1][j-c], f[i-1][j-2*c]+1, ..., f[i-1][j-k*c]+k-1)`

两者合并得到

`f[i][j] = min(f[i-1]f[j], f[i][j-c]+1)`  有了状态方程，代码就好写了

```java
public int coinChange4(int[] coins,int amount){
    int[][] dp=new int[coins.length][amount+1];
    for (int j=0;j<=amount;j++) {
        dp[0][j]=j%coins[0]==0?j/coins[0]:Integer.MAX_VALUE;
    }
    for (int i=1;i<coins.length;i++) {
        for (int j=0;j<=amount;j++) {
            if (j<coins[i] || dp[i][j-coins[i]]==Integer.MAX_VALUE) {
                //放不下
                dp[i][j]=dp[i-1][j];
            }else {
                dp[i][j]=Math.min(dp[i][j-coins[i]]+1,dp[i-1][j]);
            }
        }
    }
    return dp[coins.length-1][amount]!=Integer.MAX_VALUE?dp[coins.length-1][amount]:-1;
}
```

**空间优化**

将上面的二维改成一维就是像下面一样，注意内层的循环！因为后面是 `f[i][j-c]+1` 所以需要依赖同一层前面的结果，所以必须顺序的遍历

```java
public int coinChange(int[] coins,int amount){
    int[] dp=new int[amount+1];
    //填充初始值为 Integer.MAX_VALUE, 代表不可达
    Arrays.fill(dp,Integer.MAX_VALUE);
    dp[0]=0; //除了 dp[0]
    for (int i=0;i<coins.length;i++) {
        //注意这里不能逆序！
        for (int j=coins[i];j<=amount;j++) {
            if (dp[j-coins[i]]!=Integer.MAX_VALUE) {
                dp[j]=Math.min(dp[j-coins[i]]+1,dp[j]);   
            }
        }
    }
    return dp[amount]==Integer.MAX_VALUE?-1:dp[amount];
}
```
## [518. 零钱兑换 II](https://leetcode-cn.com/problems/coin-change-2/)

给定不同面额的硬币和一个总金额。写出函数来计算可以凑成总金额的硬币组合数。假设每一种面额的硬币有无限个

**示例 1:**

```java
输入：amount = 5, coins = [1, 2, 5]
输出：4
解释：有四种方式可以凑成总金额：
5=5
5=2+2+1
5=2+1+1+1
5=1+1+1+1+1
```

**示例 2:**

```java
输入：amount = 3, coins = [2]
输出：0
解释：只用面额 2 的硬币不能凑成总金额 3。
```

**示例 3:**

```java
输入：amount = 10, coins = [10] 
输出：1
```

**注意：**

你可以假设：

- 0 <= amount （总金额） <= 5000
- 1 <= coin （硬币面额） <= 5000
- 硬币种类不超过 500 种
- 结果符合 32 位符号整数

**解法一**

求方案数，不考虑顺序

```java
public int change(int amount, int[] coins) {
    if (coins==null || coins.length<=0) {
        return amount==0?1:0;
    }
    int[][] dp=new int[coins.length][amount+1];
    for (int i=0;i<coins.length;i++) {
        for (int j=0;j<=amount;j++) {
            if (i==0) {
                dp[0][j]=j%coins[i]==0?1:0;
            }else if (j==0) {
                dp[i][0]=1;
            }else{
                  dp[i][j]= j>=coins[i]?dp[i-1][j]+dp[i][j-coins[i]]:dp[i-1][j];
                //dp[i][j]= j>=coins[i]?dp[i-1][j]+dp[i-1][j-coins[i]]:dp[i-1][j];
            }
        }
    }
    return dp[coins.length-1][amount];
}
```
**空间优化**

`f(5)=f(4)+f(3)+f(0)` 突然感觉写二维的有点多余。这种子结构要清晰的多

```java
//直接理解一维 dp 还是不太容易，但是知道递推公式后先写个二维 dp 再改为一维就很容易
public int change(int amount, int[] coins) {
    int[] dp=new int[amount+1];
    dp[0]=1;
    //这种方式相当于对 dpTable 从左向右，一行行的递推
    for (int i=0;i<coins.length;i++) {
        for (int j=0;j<=amount;j++) {
            //dp[j]+= dp[j-coins[i]]:0;
            dp[j]=j-coins[i]>=0?dp[j]+dp[j-coins[i]]:dp[j];
        }
    }
    /* 交换一下内外顺序就变成了另一个问题的解
    for (int j=0;j<=amount;j++) {
        for (int i=0;i<coins.length;i++) {
            dp[j]+= j-coins[i]>=0?dp[j-coins[i]]:0;
        }
    }*/
    return dp[amount];
}
```
**解法二**

记忆化递归，基本上 dp 能过得，记忆化递归一定能过，相比之下，我觉得记忆化递归会好写一些

```java
public int change(int amount, int[] coins) {
    if (coins==null || coins.length<=0) {
        return amount==0?1:0;
    }
    cache=new Integer[coins.length][amount+1];
    return takeCoins(amount,coins,0);
}

Integer[][] cache=null;

//[index,coins.length] 中凑成 amount 的方案数，考虑顺序
public int takeCoins(int amount,int[] coins,int index){
    if (index>=coins.length || amount<0) {
        return 0;
    }
    if (cache[index][amount]!=null) {
        return cache[index][amount];
    }
    if (amount==0) {
        return 1;
    }
    return cache[index][amount]=takeCoins(amount-coins[index],coins,index)+takeCoins(amount,coins,index+1);
}
```
## [面试题 08.11. 硬币](https://leetcode-cn.com/problems/coin-lcci/)

硬币。给定数量不限的硬币，币值为 25 分、10 分、5 分和 1 分，编写代码计算 n 分有几种表示法。（结果可能会很大，你需要将结果模上 1000000007)

**解法一**

和上面的一样的，但是这里有一些其他的方法，记录下，元素解法就不写了，和上面的一样

```java
int mod=1000000007;

public int waysToChange(int n) {
    n/=5; //余数没有影响，都用 1 补
    int[] coins={5,2,1}; //币值也/5
    long[] dp=new long[n+1];
    Arrays.fill(dp,1L); //排除 1 分的硬币，所有的面额都可以用 1 分的凑出来
    for(int i=0;i<coins.length;i++){
        for(int j=coins[i];j<=n;j++){
            dp[j]=(dp[j]+dp[j-coins[i]])%mod;
        }
    }
    return (int)(dp[n]%mod);
}
```

直接把时间从 114ms 干到了 17ms，其实时间复杂度没变，但是缩小了解空间，所以整体的时间会提高很多，当然这里能缩小的原因主要还是因为题目比较特殊

## [377. 组合总和 Ⅳ](https://leetcode-cn.com/problems/combination-sum-iv/)

给定一个由正整数组成且不存在重复数字的数组，找出和为给定目标正整数的组合的个数。

**示例：**

```java
nums = [1, 2, 3]
target = 4

所有可能的组合为：
(1, 1, 1, 1)
(1, 1, 2)
(1, 2, 1)
(1, 3)
(2, 1, 1)
(2, 2)
(3, 1)

请注意，顺序不同的序列被视作不同的组合。

因此输出为 7。
```

**进阶：**
如果给定的数组中含有负数会怎么样？
问题会产生什么变化？
我们需要在题目中添加什么限制来允许负数的出现？

**解法一**

记忆化递归，没啥好说的

```java
//记忆化递归 1ms 100%
public int combinationSum4(int[] nums, int target) {
    if (nums==null || nums.length<=0) {
        return 0;
    }
    cache=new Integer[target+1];
    return combination(nums,target);
}

Integer[] cache=null;

public int combination(int[] nums,int target){
    if (cache[target]!=null) {
        return cache[target];
    }
    if (target==0) {
        return 1;
    }
    int res=0;
    for (int i=0;i<nums.length;i++) {
        if (target-nums[i]>=0) {
            res+=combination(nums,target-nums[i]);
        }
    }
    return cache[target]=res;
}
```
**解法二**

动态规划，乍一看好像和上面一题一样，实际上并不一样，这里是考虑顺序的，最优子结构也是

`f(5)=f(4)+f(3)+f(0)` 这样的

```java
//一维 dp
public int combinationSum4(int[] nums,int target){
    int[] dp=new int[target+1];
    dp[0]=1;
    for (int i=0;i<=target;i++) {
        for (int j=0;j<nums.length;j++) {
            dp[i]+= i>=nums[j]?dp[i-nums[j]]:0;
        }
    }
    return dp[target];
}
```

这里还是要存个疑啊，没搞明白啊，为啥交换个顺序就不一样了呢？一个是按行打表，一个是按列打表？？？还是递归好写。

## [494. 目标和](https://leetcode-cn.com/problems/target-sum/)

给定一个非负整数数组，a1, a2, ..., an, 和一个目标数，S。现在你有两个符号 + 和 -。对于数组中的任意一个整数，你都可以从 + 或 -中选择一个符号添加在前面。

返回可以使最终数组和为目标数 S 的所有添加符号的方法数。

**示例 1:**

```java
输入：nums: [1, 1, 1, 1, 1], S: 3
输出：5
解释：

-1+1+1+1+1 = 3
+1-1+1+1+1 = 3
+1+1-1+1+1 = 3
+1+1+1-1+1 = 3
+1+1+1+1-1 = 3

一共有 5 种方法让最终目标和为 3。
```

**注意：**

- 数组非空，且长度不会超过 20。
- 初始的数组的和不会超过 1000。
- 保证返回的最终结果能被 32 位整数存下

**解法一**

后面的题都优先写记忆化递归了，动态规划确实有点难顶

```java
Integer[][] cache=null;

//HashMap<Integer,Integer> cache=new HashMap<>

public int findTargetSumWays(int[] nums, int S) {
    if (nums==null || nums.length<=0 || S>1000) {
        return 0;
    }
    //
    int sum=0;
    for(int n:nums)sum+=n;
    if(S>sum)return 0;
    
    //相当于平移了一下，从 [-sum,sum] --> [0,2*sum]
    cache=new Integer[nums.length][2*sum+1];
    return findTarget(nums,S,0,2*sum+1);
}

public int findTarget(int[] nums,int S,int index,int max){
    if (S==0 && index ==nums.length) {
        return 1;
    }
    if (index>=nums.length) {
        return 0;
    }
    if(S <0  && cache[index][S+max]!=null){
        return cache[index][S+max];
    }
    if (S>=0 && cache[index][S]!=null) {
        return cache[index][S];
    }
    int temp=findTarget(nums,S-nums[index],index+1,max)+findTarget(nums,S+nums[index],index+1,max);
    if (S<0) {
        cache[index][S+max]=temp;
    }else{
        cache[index][S]=temp;
    }
    return  temp;
}
```
这题还是挺有意思的，因为里面是有负数的，直接记忆化是不行的，需要转换一下，这里我是直接将 cache 数组扩大，同时保证不会有覆盖，所以直接扩大为 2sum 就 ok，这样整个 S 的范围就从`[-sum,+sum]` 变为 `[0,2sum]` 从而可以缓存所有的递归结果，其实也可以使用两个数组一个存正数，一个存负数，然后只需要符号取反就 ok 了，只不过占用的空间会大一点

**解法二**

正儿八经的 01 背包做法

```java
public int findTargetSumWays(int[] nums, int S) {
    if(nums==null || nums.length<=0) return 0;
    //nsum 负，psum 正；sum;
    //sum=psum+nsum;
    //S=psum-nsum;
    //(sum+S)/2 = psum
    int sum=0;
    for(int i=0;i<nums.length;i++) sum+=nums[i];
    if((sum+S)%2!=0 || S>sum) return 0;
    int target=(sum+S)/2;
    int[] dp=new int[target+1];
    //Arrays.fill(dp,-1);
    dp[0]=1;
    for(int i=0;i<nums.length;i++){
        for(int j=target;j>=nums[i];j--){
            dp[j]+=dp[j-nums[i]];
        }
    }
    return dp[target];
}
```

## [1049. 最后一块石头的重量 II](https://leetcode-cn.com/problems/last-stone-weight-ii/)

有一堆石头，每块石头的重量都是正整数。

每一回合，从中选出**任意两块石头**，然后将它们一起粉碎。假设石头的重量分别为 `x` 和 `y`，且 `x <= y`。那么粉碎的可能结果如下：

- 如果 `x == y`，那么两块石头都会被完全粉碎；
- 如果 `x != y`，那么重量为 `x` 的石头将会完全粉碎，而重量为 `y` 的石头新重量为 `y-x`。

最后，最多只会剩下一块石头。返回此石头**最小的可能重量**。如果没有石头剩下，就返回 `0`。

**示例：**

```go
输入：[2,7,4,1,8,1]
输出：1
解释：
组合 2 和 4，得到 2，所以数组转化为 [2,7,1,8,1]，
组合 7 和 8，得到 1，所以数组转化为 [2,1,1,1]，
组合 2 和 1，得到 1，所以数组转化为 [1,1,1]，
组合 1 和 1，得到 0，所以数组转化为 [1]，这就是最优值。
```

**提示：**

1. `1 <= stones.length <= 30`
2. `1 <= stones[i] <= 1000`

**解法一**

想了一下，其实就是在所有石头中选取部分石头，求这部分的石头和大于`sum/2`的最小值（和正常的背包思路反着来的）

```java
//   sum   = psum + nsum
//  target = psum - nsum  (psum >= nsum)
//  sum+target = 2*psum
//  target = 2*psum-sum
//  2*psum-sum>=0
//记忆化递归
Integer[][] dp=null;

public int lastStoneWeightII(int[] stones) {
    int sum=0;
    for (int i=0;i<stones.length;i++) {
        sum+=stones[i];
    }
    dp=new Integer[stones.length+1][sum];
    return 2*dfs(stones,0,0,sum)-sum;
}

public int dfs(int[] stones,int index,int psum,int sum){
    if(2*psum>=sum){
        return psum;
    }
    if(dp[index][psum]!=null){
        return dp[index][psum];
    }
    int min=Integer.MAX_VALUE;
    for (int i=index;i<stones.length;i++) {
        min=Math.min(dfs(stones,i+1,psum+stones[i],sum),min);
    }
    return dp[index][psum]=min;
}
```
当我按照这个思路 i 写出来后发现不好改递推了😂，这个思路确实有一点点怪

**解法二**

正常的 01 背包解法，其实把上面的结论反过来就行了，既然要求一个大于等于`sum/2`的最小值，其实就是求一个小于等于`sum/2` 的最大值，这样一说就很清楚了，经典的 01 背包

```java
public int lastStoneWeightII(int[] stones) {
    if(stones==null ||stones.length<=0){
        return 0;
    }
    int n=stones.length;
    int sum=0;
    for(int i=0;i<n;i++){
        sum+=stones[i];
    }
    //背包容量为 sum/2, 求最多能装多少，经典的 01 背包
    int amount=sum/2;
    int[] dp=new int[amount+1];
    for (int i=0;i<stones.length;i++) {
        for (int j=amount;j>=stones[i];j--) {
            dp[j]=Math.max(dp[j],dp[j-stones[i]]+stones[i]);
        }
    }
    //wrong: return (amount-dp[amount])*2;
    //return sum%2==0?(amount-dp[amount])*2:(amount-dp[amount])*2+1
    //nsum=dp[amount]
    //target=psum-nsum = sum-nusm-nsum
    return sum-2*dp[amount];
}
```
这里的 retrun 有两种写法，推荐第二种，第一种还要判奇偶

> 拿到这题的的第一个解法其实是贪心，每次消除两个最大的，用优先队列维护石头大小
>
> 天真的错误解法 74 / 82 个通过测试用例
> [21,26,31,33,40] ->[7,21,26,31] -> [5,7,21] -> [5,14] ->[9]
> [21,26,31,33,40] ->[19,26,31,33]->[5]
>
> 这个思路其实是 这题的弱化版本 [1046. 最后一块石头的重量](https://leetcode-cn.com/problems/last-stone-weight/) 的解法

## [474. 一和零](https://leetcode-cn.com/problems/ones-and-zeroes/)

在计算机界中，我们总是追求用有限的资源获取最大的收益。

现在，假设你分别支配着 m 个 0 和 n 个 1。另外，还有一个仅包含 0 和 1 字符串的数组。

你的任务是使用给定的 m 个 0 和 n 个 1 ，找到能拼出存在于数组中的字符串的最大数量。每个 0 和 1 至多被使用一次。

**注意：**

- 给定 0 和 1 的数量都不会超过 100。
- 给定字符串数组的长度不会超过 600。

**示例 1:**

```java
输入：Array = {"10", "0001", "111001", "1", "0"}, m = 5, n = 3
输出：4

解释：总共 4 个字符串可以通过 5 个 0 和 3 个 1 拼出，即 "10","0001","1","0" 。
```

**示例 2:**

```java
输入：Array = {"10", "0", "1"}, m = 1, n = 1
输出：2

解释：你可以拼出 "10"，但之后就没有剩余数字了。更好的选择是拼出 "0" 和 "1" 。
```

**解法一**

之前只用 Java 写了个记忆化的，补一下纯 dp 的
```golang
//之前只用 Java 写了个记忆化的，补一下纯 dp 的
func findMaxForm(strs []string, m int, n int) int {
    var sn = len(strs)
    var dp = make([][]int, m+1)
    var Max = func(a, b int) int {if a>b {return a};return b}
    for i := 0; i <= m; i++ {
        dp[i] = make([]int, n+1)
    }
    for i := 0; i < sn; i++ {
        zero, one := count(strs[i])
        for j := m; j >= zero; j-- {
            for k := n; k >= one; k-- {
                dp[j][k] = Max(dp[j][k], dp[j-zero][k-one]+1)
            }
        }
    }
    return dp[m][n]
}

func count(str string) (int, int) {
    var one, zero = 0, 0
    for i := 0; i < len(str); i++ {
        if str[i] == '0' {
            zero++
        }
        if str[i] == '1' {
            one++
        }
    }
    return zero, one
}
```

**解法二**

其实这是一个多重背包问题，一个物品有多个权值

```java
Integer [][][] cache=null;

public int findMaxForm(String[] strs, int m, int n) {
    cache=new Integer[m+1][n+1][strs.length];
    return findMax(strs,m,n,0);
}

//m:0 n:1
public int findMax(String[] strs, int m, int n,int index) {
    if (index>=strs.length) {
        return 0;
    }
    if (cache[m][n][index]!=null) {
        return cache[m][n][index];
    }
    int[] oz=count(strs[index]);
    if (oz[1]<=n && oz[0]<=m) {
        return cache[m][n][index]=Math.max(1+findMax(strs,m-oz[0],n-oz[1],index+1),findMax(strs,m,n,index+1));
    }
    return cache[m][n][index]=findMax(strs,m,n,index+1);
}

public int[] count(String str){
    int one=0,zero=0;
    char[] s=str.toCharArray();
    for (char c:s) {
        if (c=='1') {
            one++;
        }else{
            zero++;
        }
    }
    return new int[]{zero,one};
}
```

## [1255. 得分最高的单词集合](https://leetcode-cn.com/problems/maximum-score-words-formed-by-letters/)

你将会得到一份单词表 words，一个字母表 letters （可能会有重复字母），以及每个字母对应的得分情况表 score。

请你帮忙计算玩家在单词拼写游戏中所能获得的「最高得分」：能够由 letters 里的字母拼写出的 任意 属于 words 单词子集中，分数最高的单词集合的得分。

单词拼写游戏的规则概述如下：

- 玩家需要用字母表 letters 里的字母来拼写单词表 words 中的单词。
- 可以只使用字母表 letters 中的部分字母，但是每个字母最多被使用一次。
- 单词表 words 中每个单词只能计分（使用）一次。
- 根据字母得分情况表 score，字母 'a', 'b', 'c', ... , 'z' 对应的得分分别为 score[0], score[1], ..., score[25]。
- 本场游戏的「得分」是指：玩家所拼写出的单词集合里包含的所有字母的得分之和

**示例 1：**

```java
输入：words = ["dog","cat","dad","good"], letters = ["a","a","c","d","d","d","g","o","o"], score = [1,0,9,5,0,0,3,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0]
输出：23
解释：
字母得分为  a=1, c=9, d=5, g=3, o=2
使用给定的字母表 letters，我们可以拼写单词 "dad" (5+1+5) 和 "good" (3+2+2+5)，得分为 23 。
而单词 "dad" 和 "dog" 只能得到 21 分。
```

**示例 2：**

```java
输入：words = ["xxxz","ax","bx","cx"], letters = ["z","a","b","c","x","x","x"], score = [4,4,4,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,5,0,10]
输出：27
解释：
字母得分为  a=4, b=4, c=4, x=5, z=10
使用给定的字母表 letters，我们可以组成单词 "ax" (4+5)， "bx" (4+5) 和 "cx" (4+5) ，总得分为 27 。
单词 "xxxz" 的得分仅为 25 。
```

**示例 3：**

```java
输入：words = ["leetcode"], letters = ["l","e","t","c","o","d"], score = [0,0,1,1,1,0,0,0,0,0,0,1,0,0,1,0,0,0,0,1,0,0,0,0,0,0]
输出：0
解释：
字母 "e" 在字母表 letters 中只出现了一次，所以无法组成单词表 words 中的单词。
```

**提示：**

- `1 <= words.length <= 14`
- `1 <= words[i].length <= 15`
- `1 <= letters.length <= 100`
- `letters[i].length == 1`
- `score.length == 26`
- `0 <= score[i] <= 10`
- `words[i] 和 letters[i] `只包含小写的英文字母

**解法一**

看着题目就知道这题不简单😂，11.10 的周赛最后一题，1ms，用 01 背包的思路做的，很多地方其实还没处理好

```java
public int maxScoreWords(String[] words, char[] letters, int[] score) {
    int[] les=new int[26];
    for (int i=0;i<letters.length;i++) {
        les[letters[i]-'a']++;
    }
    return maxScoreWords(words,letters,score,0,les);
}

public int maxScoreWords(String[] words, char[] letters, int[] score,int index,int[] les) {
    if (index==words.length) {
        return 0;
    }
    int res=maxScoreWords(words,letters,score,index+1,les);
    String word=words[index];
    if (hasWord(les,word)) {
        int[] bak=new int[les.length];
        System.arraycopy(les,0,bak,0,les.length);
        res=Math.max(res,getScore(bak,word,score)+maxScoreWords(words,letters,score,index+1,bak));
    }
    return res;
}

public boolean hasWord(int[] les,String word){
    int[] bak=new int[les.length];
    System.arraycopy(les,0,bak,0,les.length);
    int count=0;
    for(char c:word.toCharArray()){
        if (bak[c-'a']!=0) {
            bak[c-'a']--;
            count++;
        }
    }
    return count==word.length();
}   

public int getScore(int[] les,String word,int[] score){
    int sc=0;
    for (char c:word.toCharArray()) {
        les[c-'a']--;
        sc+=score[c-'a'];
    }
    return sc;
}
```

## [HUD4501. 小明系列故事——买年货（HUDOJ）](http://acm.hdu.edu.cn/showproblem.php?pid=4501)

**Problem Description**

春节将至，小明要去超市购置年货，于是小明去了自己经常去的都尚超市。

刚到超市，小明就发现超市门口聚集一堆人。用白云女士的话说就是：“那家伙，那场面，真是人山人海，锣鼓喧天，鞭炮齐呤，红旗招展。那可真是相当的壮观啊！”。好奇的小明走过去，奋力挤过人群，发现超市门口贴了一张通知，内容如下

值此新春佳节来临之际，为了回馈广大顾客的支持和厚爱，特举行春节大酬宾、优惠大放送活动。凡是都尚会员都可用会员积分兑换商品，凡是都尚会员都可**免费拿 k 件商品**，凡是购物顾客均有好礼相送。满 100 元送 bla bla bla bla，满 200 元送 bla bla bla bla bla...blablabla....

还没看完通知，小明就高兴的要死，因为他就是都尚的会员啊。迫不及待的小明在超市逛了一圈发现超市里有** n 件他想要的商品**。小明顺便对这 n 件商品打了分，表示商品的实际价值。小明发现身上带了** v1 的人民币**，会员卡里面有** v2 的积分**。他想知道他最多能买多大价值的商品。

由于小明想要的商品实在太多了，他算了半天头都疼了也没算出来，所以请你这位聪明的程序员来帮帮他吧。
 

**Input**
```go
输入包含多组测试用例。
每组数据的第一行是四个整数 n，v1，v2，k；
然后是 n 行，每行三个整数 a，b，val，分别表示每个商品的价钱，兑换所需积分，实际价值。
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
详细信息见 Sample。
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
2013 腾讯编程马拉松初赛第〇场（3 月 20 日）

**解法一**

三维费用的背包，但是和前面的 [474. 一和零](#474-一和零)还有点不一样，三个维度的费用是无关的，而 1 和 0 中 1 的个数和 0 的个数是相关的
> 代码使用了 Petr 的输入模板，自己改进了下，增加了输入结束判断，所以看起来特别长
```java
import java.util.*;
import java.io.*;// petr 的输入模板
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
        //别忘了 flush
        out.flush();
        out.close();
    }

    //因为数据量不大，就直接 Scanner 了
    public static void solve(int n, int v1, int v2, int k, int[][] cost) {
        int[][][] dp = new int[k+1][v1+1][v2+1];
        for (int i = 0; i < n; i++) {
            for (int j = k; j >= 0; j--) {
                for (int u = v1; u >= 0; u--) {
                    for (int w = v2; w >= 0; w--) {
                        //这里不能直接 u>=cost[i][0] w >= cost[i][1]，因为积分和钱和免费拿是分开的，没有关联的
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
    //该函数也会读取一行，并初始化 tokenizer，后序直接 nextInt.. 等就可以读到该行
    public boolean EOF() {
        String str = null;
        try {
            str = reader.readLine();
            if (str == null) {
                return true;
            }
            //创建 tokenizer
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