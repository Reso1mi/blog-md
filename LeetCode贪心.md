---
title: 
   LeetCode贪心
tags: 
  [LeetCode,贪心]
categories:
	[算法]
date: 2020/1/21
---


## [455. 分发饼干](https://leetcode-cn.com/problems/assign-cookies/)

假设你是一位很棒的家长，想要给你的孩子们一些小饼干。但是，每个孩子最多只能给一块饼干。对每个孩子 i ，都有一个胃口值 gi ，这是能让孩子们满足胃口的饼干的最小尺寸；并且每块饼干 j ，都有一个尺寸 sj 。如果 sj >= gi ，我们可以将这个饼干 j 分配给孩子 i ，这个孩子会得到满足。你的目标是尽可能满足越多数量的孩子，并输出这个最大数值。

**注意：**

你可以假设胃口值为正。
一个小朋友最多只能拥有一块饼干

**示例 1:**

```java
输入: [1,2,3], [1,1]

输出: 1

解释: 
你有三个孩子和两块小饼干，3个孩子的胃口值分别是：1,2,3。
虽然你有两块小饼干，由于他们的尺寸都是1，你只能让胃口值是1的孩子满足。
所以你应该输出1。
```


**示例 2:**

```java
输入: [1,2], [1,2,3]

输出: 2

解释: 
你有两个孩子和三块小饼干，2个孩子的胃口值分别是1,2。
你拥有的饼干数量和尺寸都足以让所有孩子满足。
所以你应该输出2.
```

**解法一**

贪就完事儿了

```java
public int findContentChildren(int[] g, int[] s) {
    if (g==null || s==null) {
        return 0;
    }
    Arrays.sort(g);
    Arrays.sort(s);
    int res=0,index=0;
    for (int i=0;i<g.length;i++) {
        while(index<s.length){
            if (g[i]<=s[index]) {
                res++;
                index++;
                break;
            }
            index++;
        }
    }
    return res;
}
```

## [392. 判断子序列](https://leetcode-cn.com/problems/is-subsequence/)

给定字符串 s 和 t ，判断 s 是否为 t 的子序列。

你可以认为 s 和 t 中仅包含英文小写字母。字符串 t 可能会很长（长度 ~= 500,000），而 s 是个短字符串（长度 <=100）。

字符串的一个子序列是原始字符串删除一些（也可以不删除）字符而不改变剩余字符相对位置形成的新字符串。（例如，`"ace"`是`"abcde"`的一个子序列，而`"aec"`不是）。

**示例 1:**

```java
s = "abc", t = "ahbgdc"
返回 true.
```

**示例 2:**

```java
s = "axc", t = "ahbgdc"
返回 false.
```

**后续挑战 :**

如果有大量输入的 S，称作S1, S2, ... , Sk 其中 k >= 10亿，你需要依次检查它们是否为 T 的子序列。在这种情况下，你会怎样改变代码？

**解法一**

```java
public boolean isSubsequence(String s, String t) {
    if (s==null || t==null) {
        return false;
    }
    int sindex=0,tindex=0;
    while(sindex<s.length()) {
        while(tindex<t.length() && sindex<s.length()){
            if (s.charAt(sindex)==t.charAt(tindex)) {
                sindex++;
            }
            tindex++;
        }
        if (tindex==t.length()) {
            break; 
        }
    }
    return sindex==s.length();
}
```
可以改成递归（多练习递归）

```java
public boolean isSubsequence(String s,String t){
    return subsequence(s,t,0,0);
}

public boolean subsequence(String s,String t,int sindex,int tindex){
    if (sindex == s.length()) {
        return true;
    }
    //上下if不能交换,可能最后一个才相等
    if (tindex == t.length()) {
        return false;
    }
    return s.charAt(sindex)==t.charAt(tindex)?subsequence(s,t,sindex+1,tindex+1):subsequence(s,t,sindex,tindex+1);
}
```
**解法二**

```java
//大量的s字符串 处理
public boolean isSubsequence3(String s, String t) {
    //预处理
    ArrayList<ArrayList<Integer>> hash=new ArrayList<>();
    for (int i=0;i<26;i++) {
        hash.add(new ArrayList());
    }
    for (int i=0;i<t.length();i++) {
        hash.get(t.charAt(i)-'a').add(i);
    }
    //经过上面的预处理,后面的处理就会很快,不用再遍历t字符串
    int lastIndex=-1;
    for (int i=0;i<s.length();i++) {
        List<Integer> indexList=hash.get(s.charAt(i)-'a');
        int temp=binarySearch(indexList,lastIndex);
        if (temp==indexList.size()) {
            return false;
        }
        lastIndex=indexList.get(temp);
    }
    return true;
}

//找到第一个比target大的元素
public int binarySearch(List<Integer> list,int target){
    int left=0,right=list.size()-1;
    while(left<=right){
        int mid=left+(right-left)/2;
        if (list.get(mid)>target) {
            right=mid-1;
        }else{
            left=mid+1;
        }
    }
    return left;
}
```

## [621. 任务调度器](https://leetcode-cn.com/problems/task-scheduler/)

给定一个用字符数组表示的 CPU 需要执行的任务列表。其中包含使用大写的 A - Z 字母表示的26 种不同种类的任务。任务可以以任意顺序执行，并且每个任务都可以在 1 个单位时间内执行完。CPU 在任何一个单位时间内都可以执行一个任务，或者在待命状态。

然而，两个**相同种类**的任务之间必须有长度为 n 的冷却时间，因此至少有连续 n 个单位时间内 CPU 在执行不同的任务，或者在待命状态。

你需要计算完成所有任务所需要的**最短时间**

**示例 1：**

```java
输入: tasks = ["A","A","A","B","B","B"], n = 2
输出: 8
执行顺序: A -> B -> (待命) -> A -> B -> (待命) -> A -> B.
```

**注：**

- 任务的总个数为 `[1, 10000]`
- n 的取值范围为 `[0, 100]`

**解法一**

```java
public int leastInterval(char[] tasks, int n) {
    int[] map=new int[26];
    for (int i=0;i<tasks.length;i++) {
        map[tasks[i]-'A']++;
    }
    //找最大值
    int max=-1;
    for (int i=0;i<map.length;i++) {
        max=Math.max(map[i],max);
    }
    int maxCount=0;
    for (int i=0;i<map.length;i++) {
        if (map[i]==max) {
            maxCount++;
        }
    }
    //比如 a b c d e f g,n=1
    return Math.max((n+1)*(max-1)+maxCount,tasks.length);
}
```
核心思想就是将出现次数最多的任务优先执行并且尽可能的分散，比如  `A A A B B C n=2` 最短的时间就是`A X X A X X A` ，最终的时间就是`(n+1)*(max-1)+1` 也就是 `(2+1) *(3-1)+1=7`， 但是可能会有多个最多次数的任务，所以我们还需要加上最多的相同的个数，最后就是 `(n+1)*(max-1)+maxCount` ，但是还不够，还是有可能会出现代码中的例子，也就是最后得到的结果比我们的任务列表还有短，所以我们需要取一个最大值

## [45. 跳跃游戏 II](https://leetcode-cn.com/problems/jump-game-ii/)

给定一个非负整数数组，你最初位于数组的第一个位置。

数组中的每个元素代表你在该位置可以跳跃的最大长度。

你的目标是使用最少的跳跃次数到达数组的最后一个位置。

**示例:**

```java
输入: [2,3,1,1,4]
输出: 2
解释: 跳到最后一个位置的最小跳跃数是 2。
     从下标为 0 跳到下标为 1 的位置，跳 1 步，然后跳 3 步到达数组的最后一个位置。
```

**说明:**

假设你总是可以到达数组的最后一个位置。

**解法一**

兴致勃勃写了个dp

```java
public int jump(int[] nums) {
    if (nums==null || nums.length<=0) {
        return 0;
    }
    int[] dp=new int[nums.length];
    for (int i=1;i<nums.length;i++) {
        dp[i]=Integer.MAX_VALUE;
        for (int j=0;j<i;j++) {
            if (nums[j]>=i-j) {
                dp[i]=Math.min(dp[j]+1,dp[i]);    
            }
        }
    }
    return dp[nums.length-1];
}
```

如果这就过了那你也太小瞧这题了😂人家可是hard题，那能这么容易就让你过了？

没错，这里直接TLE了，最后一个CASE过不去

**解法二**

贪心，核心思想不是每次都跳到最远的地方，**而是跳到当前位置能跳到的最远的位置**

```java
//每次选能跳的位置中跳的最远的
public int jump(int[] nums){
    if (nums==null || nums.length<=0) {
        return 0;
    }
    int max=0;//最大边界
    int step=0,curMaxIndex=0;
    for (int i=0;i<nums.length-1;i++) {
        curMaxIndex=Math.max(curMaxIndex,nums[i]+i); //i能跳的位置中,跳的最远的
        if (i==max) {//走到边界就++
            step++;
            max=curMaxIndex;
        }
    }
    return step;
}
```

代码需要细细品，一下可能看不太明白

## [435. 无重叠区间](https://leetcode-cn.com/problems/non-overlapping-intervals/)

给定一个区间的集合，找到需要移除区间的最小数量，使剩余区间互不重叠。

注意:

- 可以认为区间的终点总是大于它的起点。
- 区间 [1,2] 和 [2,3] 的边界相互“接触”，但没有相互重叠。

**示例 1:**

```java
输入: [ [1,2], [2,3], [3,4], [1,3] ]

输出: 1

解释: 移除 [1,3] 后，剩下的区间没有重叠。
```

**示例 2:**

```java
输入: [ [1,2], [1,2], [1,2] ]

输出: 2

解释: 你需要移除两个 [1,2] 来使剩下的区间没有重叠。
```

**示例 3:**

```java
输入: [ [1,2], [2,3] ]

输出: 0

解释: 你不需要移除任何区间，因为它们已经是无重叠的了。
```

**解法一**

动态规划，其实和最长递增子序列是一样的

```java
public int eraseOverlapIntervals(int[][] intervals) {
    if (intervals==null || intervals.length<=0) {
        return 0;
    }
    Arrays.sort(intervals,(a,b)->a[0]-b[0]);
    int[]dp=new int[intervals.length];
    int max=-1;
    for (int i=0;i<intervals.length;i++) {
        dp[i]=1;
        for (int j=0;j<i;j++) {
            if(intervals[i][0]>=intervals[j][1]){
                dp[i]=Math.max(dp[j]+1,dp[i]);
            }
        }
        max=Math.max(max,dp[i]);
    }
    return intervals.length-max;
}
```
171ms，8%，感觉快要过不了了。。。本来是是写的记忆化递归的，结果过不了。。。卡在倒数第二个case上

```java
HashMap<Pair,Integer> cache=new HashMap<>();//TLE

public int eraseOverlapIntervals2(int[][] intervals) {
    Arrays.sort(intervals,(a,b)->a[0]-b[0]);
    return intervals.length-dfs(intervals,0,Integer.MIN_VALUE);
}

//背包问题,返回最多可以留下的区间
public int dfs(int[][] intervals,int index,int prev) {
    if (index==intervals.length) {
        return 0;
    }
    Pair key=new Pair(index,prev);
    if (cache.containsKey(key)) {
        return cache.get(key);
    }
    int res=dfs(intervals,index+1,prev);
    if (intervals[index][0]>=prev) {
        res=Math.max(res,dfs(intervals,index+1,intervals[index][1])+1);
    }
    cache.put(key,res);
    return res;
}
```
**解法二**

贪心，时间复杂度降低为线性

```java
public int eraseOverlapIntervals(int[][] intervals) {
    if (intervals==null || intervals.length<=0) {
        return 0;
    }
    //按照起点排序,重叠的时候选择保留结尾小的那一个
    //Arrays.sort(intervals,(a,b)->a[0]-b[0]); lambda初始化效率会低一点
    Arrays.sort(intervals,new Comparator<int[]>(){
        @Override
        public int compare(int[] a,int[] b){
            return a[0]-b[0];
        }
    });
    int res=1;
    int prev=0;
    for (int i=1;i<intervals.length;i++) {
        if (intervals[i][0]>=intervals[prev][1]) {
            res++;
            prev=i;
        }else if(intervals[i][1]<intervals[prev][1]){
            prev=i; //选择结尾小的那一个
        }
    }
    return intervals.length-res;
}
```
按照起点排序，在重叠的时候优先选择结尾小的哪一个，这样就可能得到更多的区间组合，关于这个算法的正确性我就不证明了
