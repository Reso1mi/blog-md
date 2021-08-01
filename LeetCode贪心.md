---
title: LeetCode 贪心
tags:
  - LeetCode
  - 贪心
categories:
  - 算法
date: 2020/1/21
abbrlink: a91acf16
---

## [455. 分发饼干](https://leetcode-cn.com/problems/assign-cookies/)

假设你是一位很棒的家长，想要给你的孩子们一些小饼干。但是，每个孩子最多只能给一块饼干。对每个孩子 i ，都有一个胃口值 gi ，这是能让孩子们满足胃口的饼干的最小尺寸；并且每块饼干 j ，都有一个尺寸 sj 。如果 sj >= gi ，我们可以将这个饼干 j 分配给孩子 i ，这个孩子会得到满足。你的目标是尽可能满足越多数量的孩子，并输出这个最大数值。

**注意：**

你可以假设胃口值为正。
一个小朋友最多只能拥有一块饼干

**示例 1:**

```java
输入：[1,2,3], [1,1]

输出：1

解释：
你有三个孩子和两块小饼干，3 个孩子的胃口值分别是：1,2,3。
虽然你有两块小饼干，由于他们的尺寸都是 1，你只能让胃口值是 1 的孩子满足。
所以你应该输出 1。
```

**示例 2:**

```java
输入：[1,2], [1,2,3]

输出：2

解释：
你有两个孩子和三块小饼干，2 个孩子的胃口值分别是 1,2。
你拥有的饼干数量和尺寸都足以让所有孩子满足。
所以你应该输出 2.
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

## [274. H 指数](https://leetcode-cn.com/problems/h-index/)

给定一位研究者论文被引用次数的数组（被引用次数是非负整数）。编写一个方法，计算出研究者的 h 指数。

h 指数的定义：“h 代表“高引用次数”（high citations），一名科研人员的 h 指数是指他（她）的 （N 篇论文中）~~至多~~有 h 篇论文分别被引用了至少 h 次。（其余的 N - h 篇论文每篇被引用次数不多于 h 次。）”

**示例：**

```java
输入：citations = [3,0,6,1,5]
输出：3 
解释：给定数组表示研究者总共有 5 篇论文，每篇论文相应的被引用了 3, 0, 6, 1, 5 次。
     由于研究者有 3 篇论文每篇至少被引用了 3 次，其余两篇论文每篇被引用不多于 3 次，所以她的 h 指数是 3。
```

**说明：** 如果 h 有多种可能的值，h 指数是其中最大的那个。

**解法一**

题目意思搞懂就 ok

```java
public int hIndex(int[] citations) {
    int len=citations.length;
    Arrays.sort(citations);
    int count=0;
    for (int i=len-1;i>=0;i--) {
        if (citations[i]<=len-(i+1)) {
            return len-(i+1);
        }
    }
    return len;
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

如果有大量输入的 S，称作 S1, S2, ... , Sk 其中 k >= 10 亿，你需要依次检查它们是否为 T 的子序列。在这种情况下，你会怎样改变代码？

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
    //上下 if 不能交换，可能最后一个才相等
    if (tindex == t.length()) {
        return false;
    }
    return s.charAt(sindex)==t.charAt(tindex)?subsequence(s,t,sindex+1,tindex+1):subsequence(s,t,sindex,tindex+1);
}
```
**解法二**

```java
//大量的 s 字符串 处理
public boolean isSubsequence3(String s, String t) {
    //预处理
    ArrayList<ArrayList<Integer>> hash=new ArrayList<>();
    for (int i=0;i<26;i++) {
        hash.add(new ArrayList());
    }
    for (int i=0;i<t.length();i++) {
        hash.get(t.charAt(i)-'a').add(i);
    }
    //经过上面的预处理，后面的处理就会很快，不用再遍历 t 字符串
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

//找到第一个比 target 大的元素
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

给定一个用字符数组表示的 CPU 需要执行的任务列表。其中包含使用大写的 A - Z 字母表示的 26 种不同种类的任务。任务可以以任意顺序执行，并且每个任务都可以在 1 个单位时间内执行完。CPU 在任何一个单位时间内都可以执行一个任务，或者在待命状态。

然而，两个**相同种类**的任务之间必须有长度为 n 的冷却时间，因此至少有连续 n 个单位时间内 CPU 在执行不同的任务，或者在待命状态。

你需要计算完成所有任务所需要的**最短时间**

**示例 1：**

```java
输入：tasks = ["A","A","A","B","B","B"], n = 2
输出：8
执行顺序：A -> B -> （待命） -> A -> B -> （待命） -> A -> B.
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

## [55. 跳跃游戏](https://leetcode-cn.com/problems/jump-game/)

给定一个非负整数数组，你最初位于数组的第一个位置。

数组中的每个元素代表你在该位置可以跳跃的最大长度。

判断你是否能够到达最后一个位置。

**示例 1:**

```java
输入：[2,3,1,1,4]
输出：true
解释：我们可以先跳 1 步，从位置 0 到达 位置 1, 然后再从位置 1 跳 3 步到达最后一个位置。
```

**示例 2:**

```java
输入：[3,2,1,0,4]
输出：false
解释：无论怎样，你总会到达索引为 3 的位置。但该位置的最大跳跃长度是 0 ， 所以你永远不可能到达最后一个位置。
```

**解法一**

回溯，勉强能过。太蠢了，为啥想不到简单的方法，就非得往复杂了想？就这么傻么？

```java
Boolean[] cache=null;

public boolean canJump(int[] nums) {
    if (nums==null || nums.length<=0) return false;
    cache=new Boolean[nums.length];
    return jump(nums,0);
}

public boolean jump(int[] nums,int index) {
    if (nums[index] >= nums.length-1 -index) {
        return true;
    }
    if (cache[index]!=null) {
        return cache[index];
    }
    for (int i=nums[index];i>=1;i--) {
        if (index+i<nums.length && jump(nums,index+i)) {
            return cache[index]=true;
        }
    }
    return cache[index]=false;
}
```
**解法二**

不用多说了，遍历数组，不断更新能到达的最远距离，如果**某个位置的 index 大于当前能到达的最远距离就直接返回 false**

```java
//MDZZ
public boolean canJump(int[] nums) {
    int maxIndex=nums[0];
    for (int i=1;i<nums.length-1;i++) {
        if(maxIndex >= nums.length-1) return true;
        if (i>maxIndex) {
            return false;
        }
        maxIndex=Math.max(maxIndex,i+nums[i]);
    }
    return true;
}
```

## [45. 跳跃游戏 II](https://leetcode-cn.com/problems/jump-game-ii/)

给定一个非负整数数组，你最初位于数组的第一个位置。

数组中的每个元素代表你在该位置可以跳跃的最大长度。

你的目标是使用最少的跳跃次数到达数组的最后一个位置。

**示例：**

```java
输入：[2,3,1,1,4]
输出：2
解释：跳到最后一个位置的最小跳跃数是 2。
     从下标为 0 跳到下标为 1 的位置，跳 1 步，然后跳 3 步到达数组的最后一个位置。
```

**说明：**

假设你总是可以到达数组的最后一个位置。

**解法一**

兴致勃勃写了个 dp

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

如果这就过了那你也太小瞧这题了😂人家可是 hard 题，那能这么容易就让你过了？

没错，这里直接 TLE 了，最后一个 CASE 过不去

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
        curMaxIndex=Math.max(curMaxIndex,nums[i]+i); //i 能跳的位置中，跳的最远的
        if (i==max) {//走到边界就++
            step++;
            max=curMaxIndex;
        }
    }
    return step;
}
```

代码需要细细品，一下可能看不太明白

**解法三**

回顾的时候这道题始终是没搞清楚，[看了一个大佬的题解](https://leetcode-cn.com/problems/jump-game-ii/solution/xun-huan-bu-bian-shi-fen-xi-cban-by-huai-an-2/) （这个大佬好像是个初中的妹子）后明白了

```java
//参考了一个大佬循环不变表达式的分析
public int jump(int[] nums){
    if (nums==null || nums.length<=0) {
        return 0;
    }
    //当前这一跳能选择的最远距离
    int left=0;
    //目前能达到的最远距离
    int right=0;
    int ptr=0,step=0;
    while (right<nums.length-1) {
        left=right;
        while(ptr<nums.length && ptr<=left) {
            right=Math.max(right,nums[ptr]+ptr);
            ptr++;
        }
        step++;
    }
    return step;
}
```
## [56. 合并区间](https://leetcode-cn.com/problems/merge-intervals/)

给出一个区间的集合，请合并所有重叠的区间。

**示例 1:**

```java
输入：[[1,3],[2,6],[8,10],[15,18]]
输出：[[1,6],[8,10],[15,18]]
解释：区间 [1,3] 和 [2,6] 重叠，将它们合并为 [1,6].
```

**示例 2:**

```java
输入：[[1,4],[4,5]]
输出：[[1,5]]
解释：区间 [1,4] 和 [4,5] 可被视为重叠区间。
```

**解法一**

思路也没啥好说的，类似贪心吧

```java
public int[][] merge(int[][] intervals) {
    if (intervals ==null || intervals.length<=0) {
        return new int[][]{};
    }
    Arrays.sort(intervals,(a,b)->a[0]-b[0]);
    LinkedList<int[]> list=new LinkedList<>();
    for (int i=1;i<intervals.length;i++) {
        if (intervals[i][0]<=intervals[i-1][1]) {
            if (intervals[i][1]>intervals[i-1][1]) {
                intervals[i][0]=intervals[i-1][0];   
            }else{
                intervals[i][0]=intervals[i-1][0];
                intervals[i][1]=intervals[i-1][1];
            }
        }else{
            list.add(intervals[i-1]);
        }
    }
    list.add(intervals[intervals.length-1]);
    /*  int[][] res=new int[list.size()][2];
        for (int i=0;i<list.size();i++) {
            res[i][0]=list.get(i)[0];
            res[i][1]=list.get(i)[1];
        }*/
    return list.toArray(new int[0][0]); //题解哪里学到一招
}
```

最大的收获就是学到了一招 list 转 array 的方法😁

偶然看到，简化下代码

```java
//update：2020.4.16
//偶然看到，简化下代码
public int[][] merge(int[][] intervals) {
    if (intervals ==null || intervals.length<=0) {
        return new int[][]{};
    }
    Arrays.sort(intervals,(a,b)->a[0]-b[0]);
    LinkedList<int[]> list=new LinkedList<>();
    for (int i=1;i<intervals.length;i++) {
        if (intervals[i][0]<=intervals[i-1][1]) {
            intervals[i][0]=intervals[i-1][0];
            intervals[i][1]=Math.max(intervals[i-1][1],intervals[i][1]);
        }else{
            list.add(intervals[i-1]);
        }
    }
    list.add(intervals[intervals.length-1]);
    return list.toArray(new int[0][0]);
}
```
一开始还没注意这个解法，现在回头看看这个方法挺妙的，当无法覆盖的时候将`intervals[i-1]` 入栈，当可以覆盖的时候修改当前元素值，在下一轮继续添加或覆盖，其实还是有一点点不好理解，刚刚又重写了一个，思路很直白

```java
public int[][] merge(int[][] intervals) {
    if(intervals==null || intervals.length<=0) return intervals;
    Arrays.sort(intervals,(a,b)->a[0]!=b[0]?a[0]-b[0]:a[1]-b[1]);
    List<int[]> res=new ArrayList<>();
    res.add(intervals[0]);
    for(int i=1;i<intervals.length;i++){
        int[] pre=res.get(res.size()-1);
        if(intervals[i][0]<=pre[1]){
            if(intervals[i][1]>=pre[1]){
                pre[1]=intervals[i][1];
            }
        }else{
            res.add(intervals[i]);
        }
    }
    return res.toArray(new int[0][0]);
}
```
## [763. 划分字母区间](https://leetcode-cn.com/problems/partition-labels/)

Difficulty: **中等**

字符串 `S` 由小写字母组成。我们要把这个字符串划分为尽可能多的片段，同一个字母只会出现在其中的一个片段。返回一个表示每个字符串片段的长度的列表。

**示例 1：**

```go
输入：S = "ababcbacadefegdehijhklij"
输出：[9,7,8]
解释：
划分结果为 "ababcbaca", "defegde", "hijhklij"。
每个字母最多出现在一个片段中。
像 "ababcbacadefegde", "hijhklij" 的划分是错误的，因为划分的片段数较少。
```

**提示：**

*   `S`的长度在`[1, 500]`之间。
*   `S`只包含小写字母 `'a'` 到 `'z'` 。

**解法一**

我一开始的想法就是先统计出所有 26 个字母出现的首位置和末位置，然后题目就变成了 [合并区间](#56-合并区间)，但是其实不需要真正的合并，这里只需要求长度就行了
```golang
func partitionLabels(S string) []int {
    var m = make(map[byte]int)
    var Max = func(a, b int) int {if a>b{return a};return b}
    for i := 0; i < len(S); i++ {
        m[S[i]] = i
    }
    var start, end = 0, 0
    var res []int
    for i := 0; i < len(S); i++ {
        //更新当前区间结尾最大值
        end = Max(end, m[S[i]])
        //走到当前区间结尾，当前区间结束
        if i==end {
            res = append(res, end-start+1)
            start = i+1
        }
    }
    return res
}
```
代码意思很明确，多看看就明白了

## [435. 无重叠区间](https://leetcode-cn.com/problems/non-overlapping-intervals/)

给定一个区间的集合，找到需要移除区间的最小数量，使剩余区间互不重叠。

注意：

- 可以认为区间的终点总是大于它的起点。
- 区间 [1,2] 和 [2,3] 的边界相互“接触”，但没有相互重叠。

**示例 1:**

```java
输入：[ [1,2], [2,3], [3,4], [1,3] ]

输出：1

解释：移除 [1,3] 后，剩下的区间没有重叠。
```

**示例 2:**

```java
输入：[ [1,2], [1,2], [1,2] ]

输出：2

解释：你需要移除两个 [1,2] 来使剩下的区间没有重叠。
```

**示例 3:**

```java
输入：[ [1,2], [2,3] ]

输出：0

解释：你不需要移除任何区间，因为它们已经是无重叠的了。
```

**解法一**

动态规划，其实和最长递增子序列是一样的

> 和 [最长数对链](http://imlgw.top/2019/09/01/leetcode-dong-tai-gui-hua/#646-%E6%9C%80%E9%95%BF%E6%95%B0%E5%AF%B9%E9%93%BE) 一摸一样

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
先根据左边界排个序，保证后面的不会覆盖前面的，然后反手求一下最长的无重叠区间长度，和最长递增子序列一样，最后用总长度减去这个最长的区间长度结果就是答案

171ms，8%，感觉快要过不了了。本来是是写的记忆化递归的，结果过不了。卡在倒数第二个 case 上

**记忆化递归写法**

```java
HashMap<Pair,Integer> cache=new HashMap<>();//TLE

public int eraseOverlapIntervals2(int[][] intervals) {
    Arrays.sort(intervals,(a,b)->a[0]-b[0]);
    return intervals.length-dfs(intervals,0,Integer.MIN_VALUE);
}

//背包问题，返回最多可以留下的区间
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
    //按照起点排序，重叠的时候选择保留结尾小的那一个
    //Arrays.sort(intervals,(a,b)->a[0]-b[0]); lambda 初始化效率会低一点
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
按照起点排序，在重叠的时候优先选择结尾小的哪一个，这样就可能得到更多的区间组合

## [1288. 删除被覆盖区间](https://leetcode-cn.com/problems/remove-covered-intervals/)

Difficulty: **中等**

给你一个区间列表，请你删除列表中被其他区间所覆盖的区间。

只有当 `c <= a` 且 `b <= d` 时，我们才认为区间 `[a,b)` 被区间 `[c,d)` 覆盖。

在完成所有删除操作后，请你返回列表中剩余区间的数目。

**示例：**

```go
输入：intervals = [[1,4],[3,6],[2,8]]
输出：2
解释：区间 [3,6] 被区间 [2,8] 覆盖，所以它被删除了。
```

**提示：**​​​​​​

*   `1 <= intervals.length <= 1000`
*   `0 <= intervals[i][0] < intervals[i][1] <= 10^5`
*   对于所有的 `i != j`：`intervals[i] != intervals[j]`

**解法一**

思路比较直白
```golang
func removeCoveredIntervals(intervals [][]int) int {
    sort.Slice(intervals, func (i int, j int) bool {
        if intervals[i][0] == intervals[j][0] {
            return intervals[i][1] > intervals[j][1]
        }
        return intervals[i][0] < intervals[j][0]
    })
    var Max = func(a, b int) int {if a>b {return a};return b}
    var count = 0
    var maxRight = intervals[0][1]
    for i := 1; i < len(intervals); i++ {
        if intervals[i][1] <= maxRight {
            count++
        }
        maxRight = Max(maxRight, intervals[i][1])
    }
    return len(intervals)-count
}
```

## [452. 用最少数量的箭引爆气球](https://leetcode-cn.com/problems/minimum-number-of-arrows-to-burst-balloons/)

Difficulty: **中等**

在二维空间中有许多球形的气球。对于每个气球，提供的输入是水平方向上，气球直径的开始和结束坐标。由于它是水平的，所以 y 坐标并不重要，因此只要知道开始和结束的 x 坐标就足够了。开始坐标总是小于结束坐标。平面内最多存在 10<sup>4</sup>个气球。

一支弓箭可以沿着 x 轴从不同点完全垂直地射出。在坐标 x 处射出一支箭，若有一个气球的直径的开始和结束坐标为 x<sub style="display: inline;">start，</sub>x<sub style="display: inline;">end，</sub> 且满足  x<sub style="display: inline;">start</sub> ≤ x ≤ x<sub style="display: inline;">end，</sub>则该气球会被引爆<sub style="display: inline;">。</sub>可以射出的弓箭的数量没有限制。 弓箭一旦被射出之后，可以无限地前进。我们想找到使得所有气球全部被引爆，所需的弓箭的最小数量。

**Example:**

```go
输入：
[[10,16], [2,8], [1,6], [7,12]]

输出：
2

解释：
对于该样例，我们可以在 x = 6（射爆 [2,8],[1,6] 两个气球）和 x = 11（射爆另外两个气球）。
```

**解法一**

和前面几题一样，按照起点排序，发生重叠时记录小的 Xend，实际上 end 的含义就是当前这一箭能射穿前面所有气球的最远距离，后面的气球如果大于这个距离就需要加一箭，否则就可以一并射穿
```golang
func findMinArrowShots(points [][]int) int {
    if len(points) <= 0 {
        return 0
    }
    sort.Slice(points, func(i int, j int) bool {
        return points[i][0] < points[j][0]
    })
    var end = points[0][1]
    var res = 1
    for i := 1; i < len(points); i++ {
        if points[i][0] > end {
            res++
            end = points[i][1]
        }else{
            end = Min(end, points[i][1])
        }
    }
    return res
}
​
func Min(a, b int) int {
    if a > b {
        return b
    }
    return a
}
```
**解法二**

按照终点排序
```golang
func findMinArrowShots(points [][]int) int {
    if len(points) <= 0 {
        return 0
    }
    sort.Slice(points, func(i int, j int) bool {
        return points[i][1] < points[j][1]
    })
    var end = points[0][1]
    var res = 1
    for i := 1; i < len(points); i++ {
        if points[i][0] > end {
            res++
            end = points[i][1]
        }
    }
    return res
}

func Min(a, b int) int {
    if a > b {
        return b
    }
    return a
}
```

## [1024. 视频拼接](https://leetcode-cn.com/problems/video-stitching/)

你将会获得一系列视频片段，这些片段来自于一项持续时长为 T 秒的体育赛事。这些片段可能有所重叠，也可能长度不一。

视频片段 `clips[i]` 都用区间进行表示：开始于 `clips[i][0]` 并于 `clips[i][1]` 结束。我们甚至可以对这些片段自由地再剪辑，例如片段 [0, 7] 可以剪切成 `[0, 1] + [1, 3] + [3, 7]` 三部分。

我们需要将这些片段进行再剪辑，并将剪辑后的内容拼接成覆盖整个运动过程的片段（[0, T]）。返回所需片段的最小数目，如果无法完成该任务，则返回 -1 。

**示例 1：**

```java
输入：clips = [[0,2],[4,6],[8,10],[1,9],[1,5],[5,9]], T = 10
输出：3
解释：
我们选中 [0,2], [8,10], [1,9] 这三个片段。
然后，按下面的方案重制比赛片段：
将 [1,9] 再剪辑为 [1,2] + [2,8] + [8,9] 。
现在我们手上有 [0,2] + [2,8] + [8,10]，而这些涵盖了整场比赛 [0, 10]。
```

**示例 2：**

```java
输入：clips = [[0,1],[1,2]], T = 5
输出：-1
解释：
我们无法只用 [0,1] 和 [0,2] 覆盖 [0,5] 的整个过程。
```

**示例 3：**

```java
输入：clips = [[0,1],[6,8],[0,2],[5,6],[0,4],[0,3],[6,7],[1,3],[4,7],[1,4],[2,5],[2,6],[3,4],[4,5],[5,7],[6,9]], T = 9
输出：3
解释： 
我们选取片段 [0,4], [4,7] 和 [6,9] 。
```

**示例 4：**

```java
输入：clips = [[0,4],[2,8]], T = 5
输出：2
解释：
注意，你可能录制超过比赛结束时间的视频。
```

**提示：**

- `1 <= clips.length <= 100`
- `0 <= clips[i][0], clips[i][1] <= 100`
- `0 <= T <= 100`

**解法一**

感觉这个贪心还是很经典的，很多题都是这个思路，上面的 跳跃游戏 2，包括 172 周赛的最后一题，都是这个类似的区间覆盖问题

```java
public int videoStitching(int[][] clips, int T) {
    Arrays.sort(clips,(a,b)->a[0]-b[0]);
    int i=0,res=0,last=0;
    while(i<clips.length) {
        int temp=last;
        while(i<clips.length&&clips[i][0]<=temp) {
            last=Math.max(last,clips[i][1]);
            i++;
        }
        if (last==temp) { //没有找到能覆盖的
            return -1;
        }
        res++;
        if (last>=T) {
            return res;
        }
    }
    return -1;
}
```

首先按照左边界排序，然后找的时候**每次都在序列中找能覆盖`overlap`上一次右边界的最长区间** ，第一次覆盖其实就是找的左边界能覆盖 0 的最长的区间，然后下一次就要找能覆盖这个区间右边界的最长的区间。最终的结果就是最少的区间数目，正确性这里其实思考一下就知道了，每次都选择最优区间，对后面的选择没有负面影响，具体如何证明还是留给大佬们吧

## [406. 根据身高重建队列](https://leetcode-cn.com/problems/queue-reconstruction-by-height/)

假设有打乱顺序的一群人站成一个队列。 每个人由一个整数对` (h, k) `表示，其中 h 是这个人的身高，k 是排在这个人前面且身高大于或等于 h 的人数。 编写一个算法来重建这个队列。

注意：
总人数少于 1100 人。

**示例**

```java
输入：
[[7,0], [4,4], [7,1], [5,0], [6,1], [5,2]]

输出：
[[5,0], [7,0], [5,2], [6,1], [4,4], [7,1]]
```

**解法一**

贪心，没想出来

```java
public int[][] reconstructQueue(int[][] people) {
    if (people==null ||people.length<=0) {
        return new int[0][0];
    }
    List<int[]> res=new LinkedList<>();
    Arrays.sort(people,(p1,p2)->p1[0]!=p2[0]?p2[0]-p1[0]:p1[1]-p2[1]);
    for (int i=0;i<people.length;i++) {
        res.add(people[i][1],people[i]);
    }
    return res.toArray(new int[0][0]);
}
```

首先对身高 h 降序，k 升序进行排列得到，然后将元素`（h，k）`插入前面比它大的元素中的第 k 个位置，保证该元素前面有 k 个比当前元素大的，使之合法，**后面的比它矮的元素的移动对前面其实的没有任何影响**，这个算法的正确性很容易想到，身高高的人是看不到身高矮的人的~，也就是身高矮的人在身高高的人前或后对身高高的人是没有任何影响的

```java
[7,0] [7,1] [6,1] [5,0] [5,2] [4,4]

[]                (7,0) -> []
0                 (7,1) -> [7,0]
0 1               (6,1) -> [7,0] [7,1]
0 1 2             (5,0) -> [7,0] [6,1] [7,1]
0 1 2 3           (5,2) -> [5,0] [7,0] [6,1] [7,1]
0 1 2 3 4         (4,4) -> [5,0] [7,0] [5,2] [6,1] [7,1]
0 1 2 3 4 5                [5,0] [7,0] [5,2] [6,1] [4,4] [7,1]
```

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

**解法三**

O(NlogN) 贪心，最优解法应该是 dp，放在 dp[专题中](http://imlgw.top/2019/09/01/leetcode-dong-tai-gui-hua/)

```java
public int maxSumDivThree3(int[] nums) {
    int sum=0;
    List<Integer> one=new ArrayList<>();
    List<Integer> two=new ArrayList<>();
    for (int n:nums) {
        sum+=n;
        if (n%3==1) one.add(n);
        if (n%3==2) two.add(n);
    }
    Collections.sort(one);
    Collections.sort(two);
    if (sum%3==1) { //移除一个余数为 1 的 或者两个余数为 2 的
        return Math.max(one.size()>=1?sum-one.get(0):0,two.size()>=2?sum-two.get(0)-two.get(1):0);
    }
    if (sum%3==2) { //移除一个余数为 2 或者两个余数为 1 的
        return Math.max(two.size()>=1?sum-two.get(0):0,one.size()>=2?sum-one.get(0)-one.get(1):0);   
    }
    return sum;
}
```

如果总和%3=1 我们就可以移除数组中%3=1 的最小那个或者移除两个%3=2 的最小的，同理，总和%3=2，我们可以移除一个最小的%2=0 的元素，或者移除两个%2=1 的最小元素

这里我们需要记录的仅仅是数组中%3=1 和%3=2 的最小的 4 个值就 ok，其实不用排序就可以，直接 O(N) 遍历就行，嫌麻烦没改，后面有时间来改改

**解法二**

履行上面的承诺，改好了一版 O(N) 的贪心解法

```java
public int maxSumDivThree(int[] nums) {
    int M=0x3f3f3f3f;
    //余 1 最小值
    int min1_0=M,min1_1=M;
    //余 2 最小值
    int min2_0=M,min2_1=M;
    int sum=0;
    for(int i=0;i<nums.length;i++){
        sum+=nums[i];
        if(nums[i]%3==1){
            if(nums[i]<=min1_0){
                min1_1=min1_0; //更新次小值
                min1_0=nums[i]; //更新最小值
            }else if(nums[i]<=min1_1){
                min1_1=nums[i]; //更新次小值
            }
        }
        if(nums[i]%3==2){
            if(nums[i]<=min2_0){
                min2_1=min2_0;
                min2_0=nums[i];
            }else if(nums[i]<=min2_1){
                min2_1=nums[i];
            }
        }
    }
    if(sum%3==1) return sum-Math.min(min2_0+min2_1,min1_0);
    if(sum%3==2) return sum-Math.min(min1_0+min1_1,min2_0);
    return sum;
}
```

## [5172. 形成三的最大倍数](https://leetcode-cn.com/problems/largest-multiple-of-three/)

给你一个整数数组 `digits`，你可以通过按任意顺序连接其中某些数字来形成 3 的倍数，请你返回所能得到的最大的 3 的倍数。

由于答案可能不在整数数据类型范围内，请以字符串形式返回答案。

如果无法得到答案，请返回一个空字符串。

**示例 1：**

```java
输入：digits = [8,1,9]
输出："981"
```

**示例 2：**

```java
输入：digits = [8,6,7,1,0]
输出："8760"
```

**示例 3：**

```java
输入：digits = [1]
输出：""
```

**示例 4：**

```java
输入：digits = [0,0,0,0,0,0]
输出："0"
```

**提示：**

- 1 <= digits.length <= 10^4
- 0 <= digits[i] <= 9
- 返回的结果不应包含不必要的前导零。

**解法一**

177 周赛的 T4，时隔多日，周赛又出了这一题，和上面一样，思路差不多的，需要优先考虑只删除一个的情况

```java
public String largestMultipleOfThree(int[] digits) {
    int sum=0;
    int[] freq=new int[10];
    for(int i=0;i<digits.length;i++) {
        sum+=digits[i];
        freq[digits[i]]++;
    }
    if(sum==0) return "0";
    //删除一个余 1 的或者两个余 2 的，优先删除一个余 1 的
    //删除 1 个得到的结果肯定比删除 2 个大
    if(sum%3==1){ 
        if(!deleteMin(freq,1)){ 
            deleteMin(freq,2);
        }
    }
    if(sum%3==2){ //删除一个余 2 的或者两个余 1 的
        if(!deleteMin(freq,2)){
            deleteMin(freq,1);
        }   
    }
    StringBuilder res=new StringBuilder();
    //逆序构建结果
    for(int i=9;i>=0;i--){
        int count=freq[i];
        while(count-- >0){
            res.append(i);
        }
    }
    return res.toString();
}

public boolean deleteMin(int[] freq,int y){
    for (int i=y;i<9;i+=3) {
        if (freq[i]!=0) {
            freq[i]--;
            return true;
        }
    }
    return false;
}
```

## [1111. 有效括号的嵌套深度](https://leetcode-cn.com/problems/maximum-nesting-depth-of-two-valid-parentheses-strings/)

迷惑题，不想粘题目了

**解法一**

按照深度的奇偶划分两个子串。

```java
public int[] maxDepthAfterSplit(String seq) {
    //Deque<Character> stack=new ArrayDeque<>();
    int depth=0;
    int[] res=new int[seq.length()];
    for (int i=0;i<seq.length();i++) {
        if(seq.charAt(i)=='('){
            res[i]=depth++%2;
        }else{
            //根据左括号奇偶判断
            res[i]=--depth%2;
        }
    }
    return res;
}
```

## [1353. 最多可以参加的会议数目](https://leetcode-cn.com/problems/maximum-number-of-events-that-can-be-attended/) 

给你一个数组 events，其中 events[i] = [startDayi, endDayi] ，表示会议 i 开始于 startDayi ，结束于 endDayi 。

你可以在满足 startDayi <= d <= endDayi 中的任意一天 d 参加会议 i 。注意，一天只能参加一个会议。

请你返回你可以参加的 最大 会议数目。

**示例 1：**

![JELWN9.png](https://s1.ax1x.com/2020/04/17/JELWN9.png)

```java
输入：events = [[1,2],[2,3],[3,4]]
输出：3
解释：你可以参加所有的三个会议。
安排会议的一种方案如上图。
第 1 天参加第一个会议。
第 2 天参加第二个会议。
第 3 天参加第三个会议。
```

**示例 2：**

```java
输入：events= [[1,2],[2,3],[3,4],[1,2]]
输出：4
```

**示例 3：**

```java
输入：events = [[1,4],[4,4],[2,2],[3,4],[1,1]]
输出：4
```

**示例 4：**

```java
输入：events = [[1,100000]]
输出：1
```

**示例 5：**

```java
输入：events = [[1,1],[1,2],[1,3],[1,4],[1,5],[1,6],[1,7]]
输出：7
```

**提示：**

- `1 <= events.length <= 10^5`
- `events[i].length == 2`
- `1 <= events[i][0] <= events[i][1] <= 10^5`

**解法一**

暴力贪心

```java
//[[1,4],[4,4],[2,2],[3,4],[1,1]]
// 1,1  2,2  1,4  3,4  4,4
// 暴力贪心，按结束时间排序，优先安排结束时间短的，O(N^2)
public int maxEvents(int[][] events) {
    if(events==null || events.length<=0) return 0;
    Arrays.sort(events,(e1,e2)->e1[1]-e2[1]);
    //当天有没有安排会议
    HashSet<Integer> set=new HashSet<>();
    int count=0;
    for(int i=0;i<events.length;i++){
        int start=events[i][0];
        int end=events[i][1];
        for(int j=start;j<=end;j++){ //在对应时间段内进行安排
            if(!set.contains(j)){
                set.add(j);
                count++;
                break;
            }
        }
    }
    return count;
}
```
**解法二**

```java
//优先队列优化，NlogN
public int maxEvents(int[][] events) {
    if(events==null || events.length<=0) return 0;
    Arrays.sort(events,(e1,e2)->e1[0]-e2[0]);
    //结束时间构建小根堆
    PriorityQueue<Integer> pq=new PriorityQueue<>();
    int index=0,res=0,n=events.length;
    int curDay=1;
    while(index<n || !pq.isEmpty()){
        //将当天开始的会议的结束时间加入小根堆
        while(index<n && curDay==events[index][0]){
            pq.add(events[index++][1]);
        }
        //将过期会议的移除
        while(!pq.isEmpty() && pq.peek()<curDay){
            pq.poll();
        }
        //优先选择结束时间最短的
        if(!pq.isEmpty()){
            pq.poll();
            res++;
        }
        curDay++; //安排下一天
    }
    return res;
}
```

## [134. 加油站](https://leetcode-cn.com/problems/gas-station/)

在一条环路上有 N 个加油站，其中第 i 个加油站有汽油 gas[i] 升。

你有一辆油箱容量无限的的汽车，从第 i 个加油站开往第 i+1 个加油站需要消耗汽油 cost[i] 升。你从其中的一个加油站出发，开始时油箱为空。

如果你可以绕环路行驶一周，则返回出发时加油站的编号，否则返回 -1。

**说明：** 

- 如果题目有解，该答案即为唯一答案。
- 输入数组均为非空数组，且长度相同。
- 输入数组中的元素均为非负数。

**示例 1:**

```java
输入：
gas  = [1,2,3,4,5]
cost = [3,4,5,1,2]

输出：3

解释：
从 3 号加油站（索引为 3 处）出发，可获得 4 升汽油。此时油箱有 = 0 + 4 = 4 升汽油
开往 4 号加油站，此时油箱有 4 - 1 + 5 = 8 升汽油
开往 0 号加油站，此时油箱有 8 - 2 + 1 = 7 升汽油
开往 1 号加油站，此时油箱有 7 - 3 + 2 = 6 升汽油
开往 2 号加油站，此时油箱有 6 - 4 + 3 = 5 升汽油
开往 3 号加油站，你需要消耗 5 升汽油，正好足够你返回到 3 号加油站。
因此，3 可为起始索引。
```

**示例 2:**

```java
输入：
gas  = [2,3,4]
cost = [3,4,3]

输出：-1

解释：
你不能从 0 号或 1 号加油站出发，因为没有足够的汽油可以让你行驶到下一个加油站。
我们从 2 号加油站出发，可以获得 4 升汽油。 此时油箱有 = 0 + 4 = 4 升汽油
开往 0 号加油站，此时油箱有 4 - 3 + 2 = 3 升汽油
开往 1 号加油站，此时油箱有 3 - 3 + 3 = 3 升汽油
你无法返回 2 号加油站，因为返程需要消耗 4 升汽油，但是你的油箱只有 3 升汽油。
因此，无论怎样，你都不可能绕环路行驶一周。
```

**解法一**

```go
func canCompleteCircuit(gas []int, cost []int) int {
    n:=len(gas)
    curGas:=0 //当前油量
    start:=0 //起点
    total:=0 //gas 和 cost 之差，小于 0 的话肯定无法绕圈
    for i:=start;i<n;i++{
        curGas+=(gas[i]-cost[i])
        total+=(gas[i]-cost[i])
        //油量不够，i 无法继续前进到 i+1, 说明从 start~i 无法绕环
        if curGas<0{ 
            start=i+1
            curGas=0
        }
    }
    if total<0{
        return -1
    }
    return start
}
```

## [1046. 最后一块石头的重量](https://leetcode-cn.com/problems/last-stone-weight/)

有一堆石头，每块石头的重量都是正整数。

每一回合，从中选出两块 **最重的** 石头，然后将它们一起粉碎。假设石头的重量分别为 `x` 和 `y`，且 `x <= y`。那么粉碎的可能结果如下：

- 如果 `x == y`，那么两块石头都会被完全粉碎；
- 如果 `x != y`，那么重量为 `x` 的石头将会完全粉碎，而重量为 `y` 的石头新重量为 `y-x`。

最后，最多只会剩下一块石头。返回此石头的重量。如果没有石头剩下，就返回 `0`。

**示例：**

```java
输入：[2,7,4,1,8,1]
输出：1
解释：
先选出 7 和 8，得到 1，所以数组转换为 [2,4,1,1,1]，
再选出 2 和 4，得到 2，所以数组转换为 [2,1,1,1]，
接着是 2 和 1，得到 1，所以数组转换为 [1,1,1]，
最后选出 1 和 1，得到 0，最终数组转换为 [1]，这就是最后剩下那块石头的重量。
```

**提示：**

1. `1 <= stones.length <= 30`
2. `1 <= stones[i] <= 1000`

**解法一**

虽然 tag 有贪心，但是并不是贪心。直接模拟就行了，反而是这题的 [进阶版本](https://leetcode-cn.com/problems/last-stone-weight-ii/)，我以为可以这样贪心过，结果发现不对

```java
public int lastStoneWeight(int[] stones) {
    if(stones==null ||stones.length<=0){
        return 0;
    }
    PriorityQueue<Integer> pq=new PriorityQueue<>((a,b)->b-a);
    for(int i=0;i<stones.length;i++){
        pq.add(stones[i]);
    }
    //NlogN
    while(pq.size()>1){
        int y=pq.poll();
        int x=pq.poll();
        pq.add(y-x);
    }
    return pq.poll();
}
```
## [920. 会议室 (LintCode)](https://www.lintcode.com/problem/meeting-rooms/description)

给定一系列的会议时间间隔，包括起始和结束时间 [[s1,e1]，[s2,e2]，…(si < ei)，确定一个人是否可以参加所有会议。
- (0,8),(8,10) 在 8 这这一时刻不冲突

**样例 1**

```go
输入：intervals = [(0,30),(5,10),(15,20)]
输出：false
解释：
(0,30), (5,10) 和 (0,30),(15,20) 这两对会议会冲突
```
**样例 2**
```go
输入：intervals = [(5,8),(9,15)]
输出：true
解释：
这两个时间段不会冲突
```

**解法一**

这个比较简单，排序后判断相邻的区间是否会覆盖就行了
```golang
import "sort"

func canAttendMeetings (intervals []*Interval) bool {
    // Write your code here
    sort.Slice(intervals, func(i int, j int) bool {
        return intervals[i].Start < intervals[j].Start
    })
    for i := 1; i < len(intervals); i++ {
        if intervals[i].Start < intervals[i-1].End {
            return false
        }
    }
    return true
}
```

## [919. 会议室 II(LintCode)](https://www.lintcode.com/problem/meeting-rooms-ii/description)

给定一系列的会议时间间隔 intervals，包括起始和结束时间 [[s1,e1],[s2,e2],...] (si < ei)，找到所需的最小的会议室数量。

**样例 1**
```go
输入：intervals = [(0,30),(5,10),(15,20)]
输出：2
解释：
需要两个会议室
会议室 1:(0,30)
会议室 2:(5,10),(15,20)
```

**样例 2**
```go
输入：intervals = [(2,7)]
输出：1
解释：
只需要 1 个会议室就够了
```

**解法一**

扫描线的做法，感觉比较简单，也比较好理解（这应该属于最简单的扫描线吧，我看了其他的一些扫描线啥的都是 acm 里面的内容）
![mark](http://static.imlgw.top/blog/20200810/nQveo3X6eKxI.png?imageslim)
类似就是上图样子，求一个最大的有重合的区间数量，先将起点终点打散后排序，扫描的时候就按照排序后的节点来一个个扫描，然后根据节点的属性来判断是应该+1 还是-1，如果是起点就+1，如果遇到终点就-1，整个过程就像是一条线从左往右扫描过去一样
```golang
/**
 * Definition of Interval:
 * type Interval struct {
 *     Start, End int
 * }
 */

/**
 * @param intervals: an array of meeting time intervals
 * @return: the minimum number of conference rooms required
 */
import "sort"

type Pair struct{
    time int
    isEnd bool
}

func minMeetingRooms (intervals []*Interval) int {
    var n = len(intervals)
    var list []*Pair
    var Max = func (a,b int) int {if a>b {return a};return b}
    for i := 0; i < n; i++ {
        list = append(list, &Pair{intervals[i].Start,false})
        list = append(list, &Pair{intervals[i].End,true})
    }
    sort.Slice(list, func (i int, j int) bool {
        return list[i].time < list[j].time
    })
    var res = 0
    var count = 0
    for _, p := range list {
        if p.isEnd{
            count--
        }else{
            count++
        }
        res = Max(count, res)
    }
    return res
}
```
**解法二**

排序+小根堆，按起点排序，然后遍历所有区间，如果某个区间的 start 大于堆顶的结束时间，说明这两个会议可以公用一个会议室，所以将堆顶弹出，然后将当前会议加入堆中，所以最后堆的大小就是会议室的数量
```java
//小根堆的思路
public int minMeetingRooms(List<Interval> intervals) {
    // Write your code here
    Collections.sort(intervals,(i1,i2)->i1.start-i2.start);
    PriorityQueue<Integer> pq = new PriorityQueue<>();
    pq.add(intervals.get(0).end);
    for (int i = 1; i < intervals.size(); i++) {
        if (intervals.get(i).start > pq.peek()) {
            pq.poll();
        }
        pq.add(intervals.get(i).end);
    }
    return pq.size();
}
```
个人感觉这个思路还是没有上面扫描线简单好理解

## [391. 数飞机 (LintCode)](https://www.lintcode.com/problem/number-of-airplanes-in-the-sky/description)

给出飞机的起飞和降落时间的列表，用序列 interval 表示。请计算出天上同时最多有多少架飞机？

**样例 1:**
```go
输入：[(1, 10), (2, 3), (5, 8), (4, 7)]
输出：3
解释：
第一架飞机在 1 时刻起飞，10 时刻降落。
第二架飞机在 2 时刻起飞，3 时刻降落。
第三架飞机在 5 时刻起飞，8 时刻降落。
第四架飞机在 4 时刻起飞，7 时刻降落。
在 5 时刻到 6 时刻之间，天空中有三架飞机。
```
**样例 2:**
```go
输入：[(1, 2), (2, 3), (3, 4)]
输出：1
解释：降落优先于起飞。
```

**解法一**

和会议室一摸一样，代码稍微改动一点，排序规则需要遵循降落有限
```golang
/**
 * Definition of Interval:
 * type Interval struct {
 *     Start, End int
 * }
 */

/**
 * @param airplanes: An interval array
 * @return: Count of airplanes are in the sky.
 */
import "sort"

type Pair struct {
    time  int
    isEnd bool
}

func countOfAirplanes(airplanes []*Interval) int {
    var n = len(airplanes)
    var list []*Pair
    var Max = func(a, b int) int {
        if a > b {
            return a
        }
        return b
    }
    for i := 0; i < n; i++ {
        list = append(list, &Pair{airplanes[i].Start, false})
        list = append(list, &Pair{airplanes[i].End, true})
    }
    //[(1, 2), (2, 3), (3, 4)]
    //随意排序： 1 2start 2end 3start 3end 4 这样最大值就是 2，不对
    //所以应该降落优先，将降落时间点的排在前面
    sort.Slice(list, func(i int, j int) bool {
        if list[i].time == list[j].time {
            //将 end 放在前面
            return list[i].isEnd
        }
        return list[i].time < list[j].time
    })
    var res = 0
    var count = 0
    for _, p := range list {
        if p.isEnd {
            count--
        } else {
            count++
        }
        res = Max(count, res)
    }
    return res
}
```
当然同样可以使用堆，这里就不多写了

## [NC531. 递增数组](https://www.nowcoder.com/practice/d0907f3982874b489edde5071c96754a)

牛牛有一个数组 array，牛牛可以每次选择一个连续的区间，让区间的数都加 1，他想知道把这个数组变为严格单调递增，最少需要操作多少次？
- 1 <= array.size <= 2*10^5
- 1 <= array[i] <= 1*10^9

**示例 1**
```go
输入：[1,2,1]
输出：2
说明：把第三个数字+2 可以构成 1，2，3
```
**解法一**

```java
public long IncreasingArray (int[] array) {
    // write code here
    long res = 0;
    for (int i = 1; i < array.length; i++){
        if (array[i] <= array[i-1]) {
            res += array[i-1]-array[i]+1;
        }
    }
    return res;
}
```
虽然是 easy，还是想了一会儿，我的想法就是当增加一个数的时候就连同**后面的所有数**一起增加，而增加一个数肯定是增加到前一个数+1 的次数是最少的（当然我们也不用真的去加，因为后面的区间是整体++，而我们要求的操作次数只是个差值）
其实很详细的证明我也给不出来，我只考虑了几种情况
1. `i`后面的部分是单调递增的 3 1(i)  2  3，那么很明显这里和后面一起增加是最优选择
2. `i`后面是部分是单调递减的 3 3(i)  2  1，那么同样，和后面的一起增加是最优选择，单独选择某一个区间都会导致整体的落差变大，使得后面没增加的部分需要增加的次数增加
3. `i`后面先递增后递减 3 1(i)  2  1 同上 相当于 递增+递减 看做两部分，（1 2）同增，那么（2，1）也应该随之同增
4. `i`后面先递减后递增 5 4(i) 3 5  递减+递增 也分成两部分

## [738. 单调递增的数字](https://leetcode-cn.com/problems/monotone-increasing-digits/)

Difficulty: **中等**

给定一个非负整数&amp;nbsp;<code>N</code>，找出小于或等于&amp;nbsp;<code>N</code>&amp;nbsp; 的最大的整数，同时这个整数需要满足其各个位数上的数字是单调递增。</p>

<p>（当且仅当每个相邻位数上的数字&amp;nbsp;<code>x</code>&amp;nbsp; 和&amp;nbsp;<code>y</code>&amp;nbsp; 满足&amp;nbsp;<code>x &amp;lt;= y</code>&amp;nbsp; 时，我们称这个整数是单调递增的。）</p>

<p>

给定一个非负整数 `N`，找出小于或等于 `N` 的最大的整数，同时这个整数需要满足其各个位数上的数字是单调递增。

（当且仅当每个相邻位数上的数字 `x` 和 `y` 满足 `x <= y` 时，我们称这个整数是单调递增的。）

**示例 1:**

```go
输入：N = 10
输出：9
```

**示例 2:**

```go
输入：N = 1234
输出：1234
```

**示例 3:**

```go
输入：N = 332
输出：299
```

**说明：** `N` 是在 `[0, 10^9]` 范围内的一个整数。

<strong>示例 1:</strong></p>

```go
输入：N = 10
输出：9
```

<p><strong>示例 2:</strong></p>

```go
输入：N = 1234
输出：1234
```

<p><strong>示例 3:</strong></p>

```go
输入：N = 332
输出：299
```

<p><strong>说明：</strong> N 是在<code>[0, 10^9]</code>范围内的一个整数。</p>

**解法一**

这题写了好几版，总感觉很简单，但是总是有 case 能把我卡住，最终的思路就是，**逆向遍历**这个数，如果某个数小于前面（左边）的数，那么将前面的数减一，然后记录下当前的下标，最终这个下标后面的数都要变成 9
```golang
//322 -> 299
//243 -> 239
//3524 -> 499
//332 -> 299
//235854 235799
func monotoneIncreasingDigits(N int) int {
    var nums []int
    //123
    for N > 0 {
        nums = append(nums, N%10)
        N /= 10
    }
    var res = 0
    var idx = -1
    //213
    for i := 0; i < len(nums)-1; i++ {
        if nums[i] < nums[i+1] {
            idx = i
            nums[i+1]--
        }
    }
    for i := len(nums) - 1; i >= 0; i-- {
        if i <= idx {
            res = res*10 + 9
        } else {
            res = res*10 + nums[i]
        }
    }
    return res
}
```