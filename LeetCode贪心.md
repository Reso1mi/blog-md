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

## [274. H指数](https://leetcode-cn.com/problems/h-index/)

给定一位研究者论文被引用次数的数组（被引用次数是非负整数）。编写一个方法，计算出研究者的 h 指数。

h 指数的定义: “h 代表“高引用次数”（high citations），一名科研人员的 h 指数是指他（她）的 （N 篇论文中）~~至多~~有 h 篇论文分别被引用了至少 h 次。（其余的 N - h 篇论文每篇被引用次数不多于 h 次。）”

**示例:**

```java
输入: citations = [3,0,6,1,5]
输出: 3 
解释: 给定数组表示研究者总共有 5 篇论文，每篇论文相应的被引用了 3, 0, 6, 1, 5 次。
     由于研究者有 3 篇论文每篇至少被引用了 3 次，其余两篇论文每篇被引用不多于 3 次，所以她的 h 指数是 3。
```

**说明:** 如果 h 有多种可能的值，h 指数是其中最大的那个。

**解法一**

题目意思搞懂就ok

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

## [55. 跳跃游戏](https://leetcode-cn.com/problems/jump-game/)

给定一个非负整数数组，你最初位于数组的第一个位置。

数组中的每个元素代表你在该位置可以跳跃的最大长度。

判断你是否能够到达最后一个位置。

**示例 1:**

```java
输入: [2,3,1,1,4]
输出: true
解释: 我们可以先跳 1 步，从位置 0 到达 位置 1, 然后再从位置 1 跳 3 步到达最后一个位置。
```

**示例 2:**

```java
输入: [3,2,1,0,4]
输出: false
解释: 无论怎样，你总会到达索引为 3 的位置。但该位置的最大跳跃长度是 0 ， 所以你永远不可能到达最后一个位置。
```

**解法一**

回溯，勉强能过。。。太蠢了，为啥想不到简单的方法，就非得往复杂了想？就这么傻么？

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

不用多说了，遍历数组，不断更新能到达的最远距离，如果**某个位置的index大于当前能到达的最远距离就直接返回false**

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
先根据左边界排个序，保证后面的不会覆盖前面的，然后反手求一下最长的无重叠区间长度，和最长递增子序列一样，最后用总长度减去这个最长的区间长度结果就是答案

171ms，8%，感觉快要过不了了。。。本来是是写的记忆化递归的，结果过不了。。。卡在倒数第二个case上

**记忆化递归写法**

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

感觉这个贪心还是很经典的，很多题都是这个思路，上面的 跳跃游戏2，包括172周赛的最后一题，都是这个类似的区间覆盖问题

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

首先按照左边界排序，然后找的时候**每次都在序列中找能覆盖`overlap`上一次右边界的最长区间** ，第一次覆盖其实就是找的左边界能覆盖0的最长的区间，然后下一次就要找能覆盖这个区间右边界的最长的区间。最终的结果就是最少的区间数目，正确性可以通过反证法来证明

## [406. 根据身高重建队列](https://leetcode-cn.com/problems/queue-reconstruction-by-height/)

假设有打乱顺序的一群人站成一个队列。 每个人由一个整数对` (h, k) `表示，其中h是这个人的身高，k是排在这个人前面且身高大于或等于h的人数。 编写一个算法来重建这个队列。

注意：
总人数少于1100人。

**示例**

```java
输入:
[[7,0], [4,4], [7,1], [5,0], [6,1], [5,2]]

输出:
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

首先对身高h降序，k升序进行排列得到，然后将元素`（h，k）`插入前面比它大的元素中的第k个位置，保证该元素前面有k个比当前元素大的，使之合法，**后面的比它矮的元素的移动对前面其实的没有任何影响**，这个算法的正确性很容易想到，身高高的人是看不到身高矮的人的~，也就是身高矮的人在身高高的人前或后对身高高的人是没有任何影响的

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

