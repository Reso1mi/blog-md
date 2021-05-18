---
title: 
  LeetCode滑动窗口
tags: 
  [LeetCode,滑动窗口]
categories:
  [算法]
cover: http://static.imlgw.top/blog/20190903/ASNQ7j0W9dWO.jpg?imageslim
date: 2019/7/20
---

## LeetCode 滑动窗口

滑动问题包含一个滑动窗口，它是一个运行在一个大数组上的子列表，该数组是一个底层元素集合。假设有数组 [a b c d e f g h ]，一个大小为 3 的 **滑动窗口** 在其上滑动，则有：

```java
[a b c]
  [b c d]
    [c d e]
      [d e f]
        [e f g]
          [f g h]
```

一般情况下就是使用这个窗口在数组的 **合法区间** 内进行滑动，同时 **动态地**记录一些有用的数据，很多情况下，能够极大地提高算法地效率。

## [239. 滑动窗口最大值](https://leetcode-cn.com/problems/sliding-window-maximum/)

给定一个数组 *nums*，有一个大小为 *k* 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口 *k* 内的数字。滑动窗口每次只向右移动一位。

返回滑动窗口最大值。

**示例:**

```java
输入: nums = [1,3,-1,-3,5,3,6,7], 和 k = 3
输出: [3,3,5,5,6,7] 
解释: 

  滑动窗口的位置                最大值
---------------               -----
[1  3  -1] -3  5  3  6  7       3
 1 [3  -1  -3] 5  3  6  7       3
 1  3 [-1  -3  5] 3  6  7       5
 1  3  -1 [-3  5  3] 6  7       5
 1  3  -1  -3 [5  3  6] 7       6
 1  3  -1  -3  5 [3  6  7]      7
```

**注意：**

你可以假设 *k* 总是有效的，1 ≤ k ≤ 输入数组的大小，且输入数组不为空。

**进阶：**

你能在线性时间复杂度内解决此题吗？

**解法一**

```java
public static int[] maxSlidingWindow2(int[] nums, int k) {
    //题目上说的不为空，还是给我来了个空。。。。
    if(nums.length==0){
        return new int[]{};
    }
    if(k==1) return nums;
    LinkedList<Integer> list=new LinkedList<Integer>();
    int []res=new int[nums.length-k+1];
    int index=0;
    list.add(0);
    for (int i=1;i<nums.length;i++) {
        while(!list.isEmpty() && nums[list.getLast()]<nums[i]){
            //小于nums[i]的元素,从右边(尾)出队列 ,控制最左边(头)最大
            list.removeLast();
        }
        //然后将它加到队列中，从右边(尾)
        list.addLast(i);
        //如果队列溢出了就从右边移除一个（头）
        if(i-list.getFirst()==k){
            list.removeFirst();
        }
        if(i>=k-1){
            res[index++]=nums[list.getFirst()];
        }
    }
    return res;
}
```

> 2020.2.9 回头看了下，这不是单调队列么😂（用堆也可以啦，不过既然要在头尾都操作，直接用队列会更方便）

其实开始我也不会做，看了提示后搞了半天才弄出来，23ms，70%左右，思路就是利用一个双端队列(头尾都可以进出)，队列中存**数组下标**，然后遍历数组，在进入队列时从右向左遍历队尾，将**小于当前元素**的队尾元素去除(因为已经没用了，后面的最大值不可能是它们)，举个例子

> [2，1，-1，3，......]  k=3 ，当读到**3**这个元素的时候，窗口再向右移动最大值肯定不会是右边的元素了，所以直接剔除他们，在纸上画一画就明白了

然后很关键的一步就是什么时候移除最左边的元素，其实按照人的思路来想就是最左边的元素不在窗口内的时候，比如上面的例子，读到**3**的时候，**2**其实就应该剔除了因为它已经不在窗口内了。用代码来描述就是

`i-list.getFirst()==k`，i代表**当前元素下标**，上面的例子**3**对应的**i**就是**3**，**list.getFirst()=0**，刚好差为k，就代表**2**已经超出窗口了应该移除了。

其实这里我开始不是这样做的，我在队列里面存的不是元素索引，我存的是元素，然后在判断什么时候移除的时候发现判断不了，**index**也只是结果元素的下标，并不能代表队列最左元素的下标，对于数组存下标优先于存元素，多一个已知量有时候还是很方便的

**解法二**

属于对暴力法的优化吧，最坏情况下时间复杂度`O(NK)`，比如完全逆序的情况（2020.2.9回顾fix）

```java
public static int[] maxSlidingWindow3(int[] nums, int k) {
    //题目上说的不为空，还是给我来了个空。。。。
    int len = nums.length;
    if (len == 0) return new int[]{};
    if (len == 1) return new int[]{nums[0]};
    int localMax = Integer.MIN_VALUE;
    int[] result = new int[len - k + 1];
    for (int i = 0; i < k; i++) {
        //找到第一个窗口的最大值
        localMax = max(nums[i], localMax);
    }
    result[0] = localMax;
    for (int i = 1; i < len - k + 1; i++) {
        //窗口的下一个元素 k=3 , i=1 下一个元素下标为 3
        if (nums[i + k - 1] > localMax) {
            //判断当前窗口最大值和下一个元素的大小
            //如果比当前窗口的最大值还要大 就不用找了 就是它了
            localMax = nums[i + k - 1];
        } else if (nums[i - 1] == localMax) {
            //下一个元素比当前窗口最大值小 而且很不巧
            //当前最大值刚好是当前窗口的最左边的元素，也就是马上要超过窗口的元素
            localMax = nums[i];
            //所以就要重新找最大值
            for (int x = i; x < i + k; x++) {
                localMax = max(nums[x], localMax);
            }
        }
        //剩下的请况就是 比当前最大值小，并且最大值不是最左边的元素(还没有出界)，最大值不变
        //copy到结果中
        result[i] = localMax;
    }
    return result;
}

//其实可以用Math.max()
public static int max(int a,int b){
    if(a>=b)return a;
    return b;
}
```

2ms，99%  提交记录这种做法是最快的😂，时间复杂度最坏应该是`O(NK)`

## [5312. 大小为 K 且平均值大于等于阈值的子数组数目](https://leetcode-cn.com/problems/number-of-sub-arrays-of-size-k-and-average-greater-than-or-equal-to-threshold/) 

给你一个整数数组 `arr` 和两个整数 `k` 和 `threshold` 。

请你返回长度为 `k` 且平均值大于等于 `threshold` 的子数组数目。

 **示例 1：**

```java
输入：arr = [2,2,2,2,5,5,5,8], k = 3, threshold = 4
输出：3
解释：子数组 [2,5,5],[5,5,5] 和 [5,5,8] 的平均值分别为 4，5 和 6 。其他长度为 3 的子数组的平均值都小于 4 （threshold 的值)。
```


**示例 2：**

```java
输入：arr = [1,1,1,1,1], k = 1, threshold = 0
输出：5
```


**示例 3：**

```java
输入：arr = [11,13,17,23,29,31,7,5,2,3], k = 3, threshold = 5
输出：6
解释：前 6 个长度为 3 的子数组平均值都大于 5 。注意平均值不是整数。
```


**示例 4：**

```java
输入：arr = [7,7,7,7,7,7,7], k = 7, threshold = 7
输出：1
```


**示例 5：**

```java
输入：arr = [4,4,4,4], k = 4, threshold = 1
输出：1
```

**提示：**

- `1 <= arr.length <= 10^5`
- `1 <= arr[i] <= 10^4`
- `1 <= k <= arr.length`
- `0 <= threshold <= 10^4`

**解法一**

19双周赛的第2题，很简单的滑动窗口，枚举所有窗口计数就行了

```java
public int numOfSubarrays(int[] arr, int k, int threshold) {
    threshold*=k;
    int sum=0;
    int count=0;
    for (int i=0;i<k;i++) {
        sum+=arr[i];
    }
    for (int i=k;i<arr.length;i++) {
        if (sum>=threshold) {
            count++;
        }
        //0 1 2 3
        sum+=arr[i];
        sum-=arr[i-k];
    }
    return count;
}
```

## [3. 无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)

给定一个字符串，请你找出其中不含有重复字符的 **最长子串** 的长度。

**示例 1:**

```java
输入: "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
```

**示例 2:**

```java
输入: "bbbbb"
输出: 1
解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。
```

**示例 3:**

```java
输入: "pwwkew"
输出: 3
解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。
     请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。
```

**解法一**

这题其实很久以前(半年前)做过一次，当时用的方法很low😂

![mark](http://static.imlgw.top///20190503/zQ9MBrOhQm4S.png?imageslim)

```java
public int lengthOfLongestSubstring(String s) {
    char [] chars = s.toCharArray();
    if (chars.length == 0) {
        return 0;
    }
    int length = 1;
    String temp=new String();
    for (int i = 0; i <s.length()-1; i++) {
        if(chars[i] != chars[i+1]) {
            temp=s.substring(i, i+2);
        } else continue;
        for (int j = i+2; j < s.length(); j++) {
            if (!temp.contains(chars[j]+"")) {
                temp = s.substring(i, j+1);
            } else break;
        }
        //Break出来后判断是否是最长的k
        length=temp.length()>length?temp.length():length;
    }
    return length;
}
```

我这种完全就是利用api暴力法，居然还跑过了，时间复杂度应该是O(N3)，这题的ac率还是挺低的只有 29.3%

**解法二**

下面的是我下午重新做的

```java
public static int lengthOfLongestSubstring4(String s) {
    //边界问题永远不能忽略
    //LinkedList<Integer> list=new LinkedList<>();
    int length=s.length();
    if(length==0) return 0;
    int head=0,tail=0;
    int max=1;
    for (int i=1;i<length;i++) {
        int index=i-1; //当前元素前一个元素下标
        while(index>=head){
            if(s.charAt(i)!=s.charAt(index)){
                index--;
            }else{
                //按思路应该是把相等前所有元素移除，但是那样效率好低，所以这里我决定用数组模拟队列
                head=index+1;
                tail++;
                break;
            }
            //从尾遍历到头仍然没有相等，可以添加到队列中
            if(index==head-1){
                tail++;
                break;
            }
        }
        //每次结束循环后 尾index-头index
        max=tail-head+1 > max?tail-head+1:max;
        //System.out.println(max);
    }
    return max;
}
```

20ms左右80%左右，思路也比较清晰，遍历字符串，然后从后往前遍历字符串，判断当前字符串在前面有没有，有的话就将前面相等那个元素(index)前的元素都移除（窗口右移）

eg: `[head...index....tail] i`  向右移动`....index [head ... i]`

把当前元素添加进来，时间复杂度是O(N^2)，比上面单纯的暴力法要快多了，但是并不是最优解

**解法三**

上面的算法，其实还可以优化，每次判断前面有没有重复元素的时候可以直接用一个大小256的扩展ASCII码表数组来判断（前提是这些字符都在标准的ASCII字符中），`freq[s.charAt(i)]` 代表的就是s.charAt(i)这个字符在窗口内出现过没有，出现过就为1，否则就是0，这里如果ASCII不够其实也可以用HashMap，查找效率也很高接近O(1)

```java
public int lengthOfLongestSubstring(String s) {
    int length=s.length();
    if(length==0) return 0;
    int[] freq=new int[256]; //ASCII码表
    freq[s.charAt(0)]=1;
    int head=0,tail=0;
    int max=1;
    while(head<length){
        if(tail+1<length&&freq[s.charAt(tail+1)]==0){
            //没有重复，右边界右移
            tail++;
            freq[s.charAt(tail)]++;
        }else{
            //逐渐缩减左边界，直到不再有重复元素
            freq[s.charAt(head)]--;
            head++;
        }
        max=max<tail-head+1?tail-head+1:max;
    }
    return max;
}
```
这种做法最坏情况（没有重复的字符）下其实会遍历两遍数组tail先移动到尾部head随后有移动到尾部，但是比较好理解，解法四实际上就是对这里的优化，head每次移动都是直接移动到上一个重复元素的位置，而不是一个一个的向右移

**解法四**

提交记录上最快的方法，理解起来有点费劲，现在回头看又看不懂了。。。

```java
public int lengthOfLongestSubstring(String s) {
    int n = s.length(), ans = 0;
    //索引为元素值，这里因为元素都是字符转过来就是askll码，所以可以直接这样
    int[] index = new int[128];
    for (int j = 0, i = 0; j < n; j++) {
        //如果字符 char 在没有出现过，index[char]为0，出现则index[char]是遍历出现的最后的char的位置
        // index[s.charAt(j)] 是当前字符上一次出现的位置(从1开始)
        i = Math.max(i,index[s.charAt(j)]);
        //j - i + 1 就是舍弃s.charAt(j)重复出现之前字符的长度  如abca,当s.charAt(j) == a时，j - i + 1就是bca的长度
        //求最大值常规操作
        ans = Math.max(ans, j - i + 1);
        index[s.charAt(j)] = j + 1;
        // 从1开始
    }
    return ans;
}
```

这里最主要就是这个**i**的理解

![mark](http://static.imlgw.top///20190503/Qcj2l3E0v5sN.png?imageslim)

这个i不一定是当前元素上一次出现的位置，也有可能是离**当前**元素从右向左**最近**的**重复字符**的_位置_。而index中存的就是这个元素的索引位置+1，为什么要加1？

![mark](http://static.imlgw.top///20190503/N4GzYaft0suh.png?imageslim)

主要是因为i的默认值是0而数组默认值也是0，如果以0开始就会出现上面的情况。

其实这两种算法也比较类似，只是后面判断字符是否出现过的方式不同，前者是直接遍历这个子串，后者是利用字符为索引，其值就是上一次出现的位置(+1)，借此来计算长度。

**回首掏**

19/9/14，在网页上又写了一种不同的解法，属于解法4的变体（做的时候并没有想到解法4），`freq[]` 数组索引是字符串，但是值是该字符在s中对应的索引，不会遍历两遍数组，left可以通过索引直接跳到上一次出现的位置

```java
public static int lengthOfLongestSubstring6(String s) {
    if(s==null||s.length()<1) return 0;
    if(s.length()==1) return 1;
    int[] freq=new int[256];
    int left=0,right=0,res=0;
    Arrays.fill(freq,-1);
    while(right<s.length()){
        char sr=s.charAt(right);
        //已经存在,并且在窗口内
        if(freq[sr]!=-1 && freq[sr]>=left){
            //System.out.println(left+","+right+","+freq[sr]);
            res=Math.max(res,right-left);
            left=freq[sr]+1;
            freq[sr]=right;
        }else{
            res=Math.max(res,right-left+1);
            freq[sr]=right;
        }
        right++;
    }
    return res;
}
```
**回首掏2**

```java
public static int lengthOfLongestSubstring7(String s) {
    if(s==null||s.length()<1) return 0;
    if(s.length()==1) return 1;
    int[] freq=new int[256];
    int left=0,right=0,res=0;
    //Arrays.fill(freq,-1);
    while(right<s.length()){
        char sr=s.charAt(right);
        //已经存在(出现过),并且上一次出现在窗口内
        if(freq[sr]!=0 && freq[sr]>=left){
            //这里不包含right,所以不用加1
            res=Math.max(res,right-left);
            //left移动到重复位置元素的下一个
            //因为freq的值是存的索引+1所以这里不用+1
            left=freq[sr];
        }else{
            //这里包含right所以需要加1
            res=Math.max(res,right-left+1);
        }
        //加1是为了区别s的第一个字符
        freq[sr]=right+1;
        right++;
    }
    return res;
}
```
感觉对这题有很深的执念😅，对上面右优化了一下，但是其实还是上面的解法三比较简单，说到解法三，我又抽风改了一个boolean数组版本的

```java
//update: 2020.4.13 重写了一遍，代码更简洁了
public int lengthOfLongestSubstring(String s) {
    if(s==null ||s.length()<=0) return 0;
    boolean[] freq=new boolean[128];
    int left=0,right=0;
    int max=0;
    while(left<=right && right<s.length()){
        if(!freq[s.charAt(right)]){
            max=Math.max(max,right-left+1);
            freq[s.charAt(right++)]=true;
        }else{
            freq[s.charAt(left++)]=false;
        }
    }
    return max;
}
```
**_以上解法全部作废_**

统一使用`for-while`结构，能用`for-if`的一定可以用`for-while`，反过来就不行

```java
//2020.5.5根据自己总结的滑窗模板重写
public int lengthOfLongestSubstring(String s) {
    if(s==null || s.length()<=0) {
        return 0;
    }
    int n=s.length();
    int left=0;
    int res=1;
    boolean[] freq=new boolean[128];
    for(int right=0;right<n;right++){
        while(freq[s.charAt(right)]){
            freq[s.charAt(left++)]=false; //left不用限制
        }
        freq[s.charAt(right)]=true;
        res=Math.max(res,right-left+1);
    }
    return res;
}
```
**最优解**

map记录元素最后出现的位置，当重复的时候更新起点，只用遍历一遍
```go
func lengthOfLongestSubstring(s string) int {
    if len(s)==0{
        return 0
    }
    m:=make(map[rune]int)
    start:=0
    res:=0
    for i,ch:=range s{
        if idx,ok:=m[ch];ok && idx+1>start{
            start=idx+1
        }
        m[ch]=i
        res=Max(res,i-start+1)
    }
    return res
}

func Max(a,b int) int{
    if a>b{
        return a
    }
    return b
}
```

## [219. 存在重复元素 II](https://leetcode-cn.com/problems/contains-duplicate-ii/)

给定一个整数数组和一个整数 *k*，判断数组中是否存在两个不同的索引 *i* 和 *j*，使得 **nums [i] = nums [j]**，并且 *i* 和 *j* 的差的绝对值最大(不超过)为 *k*。

**示例 1:**

```java
输入: nums = [1,2,3,1], k = 3
输出: true
```

**示例 2:**

```java
输入: nums = [1,0,1,1], k = 1
输出: true
```

**示例 3:**

```java
输入: nums = [1,2,3,1,2,3], k = 2
输出: false
```

**解法一**

这题建议区看看原版的英文题，这里翻译过来有点误导人，应该是不超过k，写个最大搞得我有点懵

```java
public static Boolean containsNearbyDuplicate2(int[] nums, int k) {
    int len=nums.length;
    //if(k==35000) return false; 哈哈哈哈
    if(len==0||k==0)return false;
    for (int i=1;i<len;i++) {
        int index=i-1;
        while(i-index<=k){
            //k步之内
            if(index<0){
                break;
            }
            if(nums[i]==nums[index--]){
                return true;
            }
        }
    }
    return false;
}
```

暴力法，惨不忍睹，600ms，15%beats 。这题最好的做法还是借助hash表

**解法二**

```java
public static Boolean containsNearbyDuplicate3(int[] nums, int k) {
    HashMap<Integer,Integer> map=new HashMap<>();
    int len=nums.length;
    if(len==0||k==0) return false;
    for (int i=0;i<len;i++) {
        if(map.containsKey(nums[i])){
            int index=map.get(nums[i]);
            if(i-index<=k) {
                return true;
            } else {
                //大于k了那前面那个没用了
                map.replace(nums[i],i);
            }
        } else {
            map.put(nums[i],i);
        }
    }
    return false;
}
```

12ms，90%，其实效率的差距就在查找子串里有没有这个字符上，HashMap的containsKey的效率比我们遍历的不知道高到那里去了，底层源码暂时还看不太懂，以后看的时候再专门来讲

19.9.14又做了一遍，直接在网页上写的，本来应该是bugfree的，结果减反了。。。

```java
public static boolean containsNearbyDuplicate4(int[] nums, int k) {
	HashMap<Integer,Integer> hashMap=new HashMap<>();
	int left=0,right=0;
	while(right<nums.length){
		if(hashMap.containsKey(nums[right])){
			//md,重新做的时候这里减反了真是个zz
			if(right-hashMap.get(nums[right])<=k){
				return true;
			} else{
				hashMap.put(nums[right],right);
			}
		}else{
			hashMap.put(nums[right],right);
		}
		right++;
	}
	return false;
}
```
19.9.15又做了一遍，这次代码写的很简洁，思路也不同了，直接利用set集合，维护一个大小为k的连续set(窗口)

```java
public boolean containsNearbyDuplicate(int[] nums, int k) {
    HashSet<Integer> set=new HashSet<>();
    for (int i=0;i<nums.length;i++) {
        if (!set.add(nums[i])) {
            return true;
        }
        if (set.size()>k) {
            set.remove(nums[i-k]);
        }
    }
    return false;
}
```

## [209. 长度最小的子数组](https://leetcode-cn.com/problems/minimum-size-subarray-sum/)

给定一个含有 **n** 个正整数的数组和一个正整数 **s ，**找出该数组中满足其和 **≥ s** 的长度最小的连续子数组**。**如果不存在符合条件的连续子数组，返回 0。

**示例:** 

```java
输入: s = 7, nums = [2,3,1,2,4,3]
输出: 2
解释: 子数组 [4,3] 是该条件下的长度最小的连续子数组。
```

**进阶:**

如果你已经完成了*O*(*n*) 时间复杂度的解法, 请尝试 *O*(*n* log *n*) 时间复杂度的解法。

**解法一**

老规矩先上个慢的

```java
public static int minSubArrayLen(int s, int[] nums) {
    int len=nums.length;
    if(len==0)return 0;
    int minLen=Integer.MAX_VALUE;
    for (int i=1;i<len;i++){
        if(nums[i-1]>=s) return 1;
        int index=i-1;
        int sum=nums[i];
        //累加前面的元素，直到大于s或者index<0
        while(sum<s&&index>=0){
            sum+=nums[index--];
        }
        if(sum>=s){
            minLen=minLen>i-index?i-index:minLen;
        } else if(i==len-1){
            return 0;
        }
    }
    return minLen;
}
```

这个是最开始想到了思路比较清晰，遍历数组然后逆序求和直到大于S，要注意边界，比较慢，主要就是那个循环累加前面的元素会很耗费时间，111ms，14% beats🤣

**解法二**

```java
public static int minSubArrayLen2(int s, int[] nums) {
    int len =nums.length;
    if(len==0)return 0;
    //窗口左右边界
    int head=0,tail=-1;
    int minLen=Integer.MAX_VALUE;
    int sum=0;
    // 2,3,1,2,4,3 | 7
    while (tail<len) {
        if(sum>=s){
            minLen=minLen>tail-head+1?tail-head+1:minLen;
            //System.out.println(minLen);
            //删除头节点（左边界左移）
            sum-=nums[head++];
        } else{
            //尾指针到达边界了
            if(tail==len-1) break;
            //尾节点++，(右边界右移)
            sum+=nums[++tail];
            //如果有元素大于s直接返回，节约时间
            if(nums[tail]>=s){
                return 1;
            }
        }
    }
    return minLen==Integer.MAX_VALUE?0:minLen;
}
```

2ms ，99%beats，一对比差距就出来了，上面这种做法就是利用了滑动窗口的思想，很巧妙的利用了上一次计算的值，不用重复的计算累加和，上面的那种方法每次都会重新计算累加和，但是很多都是重复的计算，所以浪费了很多时间，要注意边界条件

`2 3 1 2 4 3  s=7`

![mark](http://static.imlgw.top///20190504/U9BTOJMRVhDO.jpg?imageslim)

自己在纸上画一下就懂了

**解法三**

~~找到一个模板，统一一下写法~~

```java
public static int minSubArrayLen3(int s, int[] nums) {
    int len =nums.length-1;
    if(len==0)return 0;
    int head=0,tail=-1;
    int minLen=Integer.MAX_VALUE;
    int sum=0;
    // 2,3,1,2,4,3 | 7
    while (head<=len) {
        if(tail+1<=len && sum<s){
            sum+=nums[++tail];
        }else{
            sum-=nums[head++];
        }
        if(sum>=s){
            minLen=minLen>tail-head+1?tail-head+1:minLen;
        }
    }
    return minLen==Integer.MAX_VALUE?0:minLen;
}
```

**Update(2020.5.4)**

上面的这也配叫模板？下面的是重做的时候统一的写法

```go
var INF = 1 << 31

func minSubArrayLen(s int, nums []int) int {
    left := 0
    sum := 0
    res := INF
    for right := 0; right < len(nums); right++ {
        sum += nums[right]
        for sum >= s {
            res = Min(res, right-left+1)
            sum -= nums[left]
            left++
        }
    }
    if res == INF {
        return 0
    }
    return res
}

func Min(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```

## [面试题57 - II. 和为s的连续正数序列](https://leetcode-cn.com/problems/he-wei-sde-lian-xu-zheng-shu-xu-lie-lcof/) 

输入一个正整数 target ，输出所有和为 target 的连续正整数序列（至少含有两个数）。

序列内的数字由小到大排列，不同序列按照首个数字从小到大排列。

**示例 1：**

```java
输入：target = 9
输出：[[2,3,4],[4,5]]
```


**示例 2：**

```java
输入：target = 15
输出：[[1,2,3,4,5],[4,5,6],[7,8]]
```

**限制：**

- `1 <= target <= 10^5`

**解法一**

滑动窗口，根据等差数列前n项和求sum，然后逐步的缩圈，右移

```java
public int[][] findContinuousSequence(int target) {
    List<int[]> res=new ArrayList<>();
    int left=1,right=2;
    int sum=0;
    //9 right最多到5
    while(left<=right && right<=(target+1)/2){
        //等差数列前n项和
        int n=right-left+1;
        sum=left*n+n*(n-1)/2;
        if(sum>target){
            left++; //剔除一个小的
        }else if(sum<target){
            right++; //添加一个大的
        }else{ //build结果集
            res.add(build(left,right));
            left++;//窗口左移,剔除一个小的
            right++; //回头重写发现这里还可以优化,右边界也可以扩大
        }
    }
    return res.toArray(new int[0][0]);
}

public int[] build(int left,int right){
    int[] res=new int[right-left+1];
    for(int i=left;i<=right;i++){
        res[i-left]=i;
    }
    return res;
}
```
这里其实并不是最优解，还有更好的数学解法，直接根据求和公式，枚举所有的长度，逆向求出所有的首项，这里后面有时间再来实现

## [480. 滑动窗口中位数](https://leetcode-cn.com/problems/sliding-window-median/)

中位数是有序序列最中间的那个数。如果序列的大小是偶数，则没有最中间的数；此时中位数是最中间的两个数的平均数。

例如：

```java
[2,3,4]，中位数是 3
[2,3]，中位数是 (2 + 3) / 2 = 2.5
```

给出一个数组 nums，有一个大小为 *k* 的窗口从最左端滑动到最右端。窗口中有 k 个数，每次窗口移动 1 位。你的任务是找出每次窗口移动后得到的新窗口中元素的中位数，并输出由它们组成的数组。

例如：

给出 *nums* = `[1,3,-1,-3,5,3,6,7]`，以及 *k* = 3。

```java
窗口位置                      中位数
---------------               -----
[1  3  -1] -3  5  3  6  7       1
 1 [3  -1  -3] 5  3  6  7       -1
 1  3 [-1  -3  5] 3  6  7       -1
 1  3  -1 [-3  5  3] 6  7       3
 1  3  -1  -3 [5  3  6] 7       5
 1  3  -1  -3  5 [3  6  7]      6
```

 因此，返回该滑动窗口的中位数数组 `[1,-1,-1,3,5,6]`。

**提示：**
假设`k`是合法的，即：`k` 始终小于输入的非空数组的元素个数.

head题，理清楚思路后也不难。

```java
public static double[] medianSlidingWindow(int[] nums, int k) {
    //看了一圈评论区，大概知道思路了，还是要排序
    //List<Integer> list=new ArrayList<>(); 用链表还是不方便啊
    int [] queue=new int[k];
    int head=0,tail=k-1;
    //头尾
    double [] res=new double[nums.length-k+1];
    for (int i=0;i<k;i++) {
        queue[i]=nums[i];
    }
    Arrays.sort(queue);
    printArray(queue);
    if (k%2==0) {
        res[0]=queue[(k-1)/2]/2.0+queue[(k-1)/2+1]/2.0;
        //注意除小数 .。。。。这里的测试用例Integer最大值，直接相加/2会越界
    } else {
        res[0]=queue[k/2];
    }
    for (int i=k;i<nums.length;i++) {
        //插入之前要移除上一次的头元素，这里用数组不好搞啊啊
        //System.out.println(nums[i-k]);
        deleHead(queue,nums[i-k]);
        tail--;
        //printArray(queue);
        //二分找插入点
        int index=binarySearch(queue,0,tail,nums[i]);
        System.out.println(index);
        tail++;
        //插入元素，tail++;
        for (int j=tail;j>index;j--) {
            //后一个等于前一个，给插入的元素腾出位置
            queue[j]=queue[j-1];
        }
        queue[index]=nums[i];
        //求中点
        if (k%2==0) {
            res[i-k+1]=queue[head+(tail-head)/2]/2.0+queue[head+(tail-head)/2+1]/2.0;
        } else {
            res[i-k+1]=queue[head+(tail-head)/2];
        }
    }
    return res;
}

public static void deleHead(int []nums,int target){
    for (int j=0,i=0;j<nums.length;j++,i++) {
        if(nums[j]==target){
            if (i==nums.length-1) {
                return;
            }
            while(i<nums.length-1){
                nums[i]=nums[i+1];
                i++;
            }
            return;
        }
    }
}

public  static int binarySearch(int []nums,int lo,int hi,int target){
    while(lo<=hi){
        int mid=lo+(hi-lo)/2;
        if(nums[mid]<target){
            lo=mid+1;
        } else if(nums[mid]>target){
            hi=mid-1;
        } else{
            return mid;
        }
    }
    return lo;
}
```

根据评论区提供的思路，用数组实现了一遍，当时就感觉有问题，确实，最后 164ms ，27%，很慢了，我觉得主要问题就是那个删除头的操作，但是毕竟数组，没办法，随即改用链表

```java
public static double[] medianSlidingWindow2(int[] nums, int k) {
    //看了一圈评论区，大概知道思路了，还是要排序
    List<Integer> list=new ArrayList<>();
    int head=0,tail=k-1;
    //头尾
    double [] res=new double[nums.length-k+1];
    //Arrays.sort(nums,0,k);
    int []temp=new int[k];
    for (int i=0;i<k;i++) {
        temp[i]=nums[i];
    }
    Arrays.sort(temp);
    for (int i=0;i<k;i++) {
        list.add(temp[i]);
    }
    if (k%2==0) {
        res[0]=list.get((k-1)/2)/2.0+list.get((k-1)/2+1)/2.0;
    } else {
        res[0]=list.get(k/2);
    }
    for (int i=k;i<nums.length;i++) {
        //插入之前要移除上一次的头元素，这里用数组不好搞啊啊
        //list.remove((Object)nums[i-k]); 直接删太慢了
        int dele=binarySearch(list,0,k-1,nums[i-k]);
        list.remove(dele);
        //System.out.println(list);
        //二分找插入点，找的区间为 [i-k+1, i-1]
        //int head=i-k+1,tail=i-1;
        int index=binarySearch(list,0,k-2,nums[i]);
        System.out.println(index);
        list.add(-1);
        for (int j=k-1;j>index;j--) {
            //后一个等于前一个，给插入的元素腾出位置
            list.set(j,list.get(j-1));
        }
        list.set(index,nums[i]);
        System.out.println(list);
        //求中点
        if (k%2==0) {
            res[i-k+1]=list.get((k-1)/2) / 2.0 + list.get((k-1)/2+1)/2.0;
        } else {
            res[i-k+1]=list.get(k/2);
        }
    }
    return res;
}
public  static int binarySearch(List<Integer> list,int lo,int hi,int target){
    while(lo<=hi){
        int mid=lo+(hi-lo)/2;
        if(list.get(mid)<target){
            lo=mid+1;
        } else if(list.get(mid)>target){
            hi=mid-1;
        } else{
            return mid;
        }
    }
    return lo;
}
```

47ms，70%左右，删除的时候利用二分删除，如果直接根据元素去删就跟数组效率差不多了。

## [76. 最小覆盖子串](https://leetcode-cn.com/problems/minimum-window-substring/)

给你一个字符串 S、一个字符串 T，请在字符串 S 里面找出：包含 T 所有字母的最小子串。

**示例：**

```java
输入: S = "ADOBECODEBANC", T = "ABC"
输出: "BANC"
```

**说明：**

- 如果 S 中不存这样的子串，则返回空字符串 ""。
- 如果 S 中存在这样的子串，我们保证它是唯一的答案。

**解法一**

```java
public static String minWindow(String s, String t) {
    int slen=s.length();
    int tlen=t.length();
    int l=0,r=0; //初始都为0
    int[] target=new int[256]; //A:1 B:1 C:1
    int[] window=new int[256];
    int count=0; //不同字符的数量
    for (int i=0;i<tlen;i++) {
        target[t.charAt(i)]++;
    }
    for (int a:target) {
        if (a!=0) {
            count++; //统计不同字符出现的次数
        }
    }
    int match=0; //match代表已经匹配的字符
    int[] res=new int[]{0,Integer.MAX_VALUE};
    while(r<slen){
        char c=s.charAt(r); 
        if(target[c]!=0){ //在目标子串中存在
            window[c]++; //window对应的char++
            if(window[c]==target[c]){ //到达了目标串中该char所需的数量
                match++;
            }
        }
        r++;
        while(l<r&&count==match) { //满足目标串，注意别越界
            char d=s.charAt(l);
            if (r-l<res[1]-res[0]) { //统计最小值
                res[0]=l;
                res[1]=r;
            }
            l++;
            if (target[d]!=0) {
                window[d]--;
                if (window[d]<target[d]) {//左边界左移后不再满足目标串
                    match--;
                }
            }
        }
    }
    return res[1]==Integer.MAX_VALUE?"":s.substring(res[0],res[1]);
}
```
10ms 86%，Hard题，其实大致的思路还是有的，主要是不知道怎么去和目标串对比，没想到用一个`window`数组去对比，一致想的是在目标串的数组上做手脚，但是越想越复杂。。。太蠢了😅，这题其实也可以用一个HashMap来做，但是我看了下提交记录上的普遍都是7,80ms，相对都比较慢，实际上题目没有明确的说明有特殊字符的话都是可以用一个**ASCII**数组来充当HashMap的，当然我这里用数组的时候相比HashMap要多了一步，需要统计不同字符出现的次数，不过这个操作也是常数级别的操作，并不耗时，整体时间复杂度O(N+M)，NM分别代表目标子串`t` 和源字符串 `p`的长度，首先遍历了`t` 然后滑动窗口，后面的滑动窗口左右边界最多移动2M次

**Update**

2020.4.15，在瞄了一眼之前做的之后按照之前的思路重写了一遍，感觉还行，有一个地方WA了一次

```java
//update: 2020.4.15
public String minWindow(String s, String t) {
    if(s==null || t==null) return "";
    int[] needMap=new int[128]; //需要的字符map
    int[] curMap=new int[128];  //已经匹配的字符map
    int needCount=0; //需要匹配的字符个数
    for(int i=0;i<t.length();i++){
        if(needMap[t.charAt(i)]==0){
            needCount++;
        }
        needMap[t.charAt(i)]++;
    }
    int matchCount=0; //已经匹配的个数
    int left=0,right=0;
    int minLeft=0,maxRight=Integer.MAX_VALUE;
    while(left<=right && right<s.length()){
        char c=s.charAt(right);
        if(needMap[c]!=0){
            curMap[c]++;
            if(curMap[c]==needMap[c]){
                matchCount++;
            }
        }
        while(left<=right && right<s.length() && matchCount==needCount){
            if(right-left<maxRight-minLeft){
                maxRight=right;
                minLeft=left;
            }
            char cl=s.charAt(left);
            if(curMap[cl]!=0){
                curMap[cl]--;
                //这里注意，WA点，开始写的=0
                if(curMap[cl]<needMap[cl]){
                    matchCount--;
                }
            }
            left++;
        }
        right++;
    }
    return Integer.MAX_VALUE==maxRight?"":s.substring(minLeft,maxRight+1);
}
```
> 刷题的时候发现有一道很类似的题，[最小窗口子序列](http://imlgw.top/2019/09/01/leetcode-dong-tai-gui-hua/#857-%E6%9C%80%E5%B0%8F%E7%9A%84%E7%AA%97%E5%8F%A3%E5%AD%90%E5%BA%8F%E5%88%97%EF%BC%88LintCode%EF%BC%89)唯一的区别就是这道题要求有序（子序列）

## [632. 最小区间](https://leetcode-cn.com/problems/smallest-range-covering-elements-from-k-lists/)

Difficulty: **困难**


你有 `k` 个升序排列的整数数组。找到一个**最小**区间，使得 `k` 个列表中的每个列表至少有一个数包含在其中。

我们定义如果 `b-a < d-c` 或者在 `b-a == d-c` 时 `a < c`，则区间 [a,b] 比 [c,d] 小。

**示例 1:**

```java
输入:[[4,10,15,24,26], [0,9,12,20], [5,18,22,30]]
输出: [20,24]
解释: 
列表 1：[4, 10, 15, 24, 26]，24 在区间 [20,24] 中。
列表 2：[0, 9, 12, 20]，20 在区间 [20,24] 中。
列表 3：[5, 18, 22, 30]，22 在区间 [20,24] 中。
```

**注意:**

1.  给定的列表可能包含重复元素，所以在这里升序表示 >= 。
2.  1 <= `k` <= 3500
3.  -10<sup>5</sup> <= `元素的值` <= 10<sup>5</sup>


**解法一**

```golang
func smallestRange(nums [][]int) []int {
    var n = len(nums)
    //列表中所有元素k,存在于那些数组中
    var m = make(map[int][]int)
    var Max = func (a, b int) int {if a > b {return a}; return b}
    var Min = func (a, b int) int {if a < b {return a}; return b}
    //记录最大最小值，然后在区间内滑窗，也可以直接将所有数组归并排一下，然后滑窗，比较麻烦，懒得写了
    //所以下面的做法实际上是和循序无关的，没有用到有序这个条件
    var maxV = math.MinInt32
    var minV = math.MaxInt32
    for i :=0; i < n; i++ {
        for j := 0; j < len(nums[i]); j++ {
            m[nums[i][j]] = append(m[nums[i][j]], i)
            maxV = Max(maxV, nums[i][j])
            minV = Min(minV, nums[i][j])
        }
    }
    //同 76.最小覆盖子串，这题可能思维的转换比较重要
    var count = 0
    var freq = make([]int, n+1)
    var res =[]int{minV, maxV}
    var left = minV
    for right := minV; right <= maxV; right++ {
        if lis, ok := m[right]; ok {
            for _, numIdx := range lis {
                freq[numIdx]++
                if freq[numIdx] == 1{
                    count++
                }
            }
        }
        for count == n && left <= right {
            if right-left < res[1]-res[0] {
                res[0] = left
                res[1] = right
            }
            if lis, ok := m[left]; ok{
                for _, numIdx := range lis {
                    freq[numIdx]--
                    if freq[numIdx] < 1 {
                        count--
                    }
                }
            }
            left++
        }
    }
    return res
}
```
**解法二**

上面的解法并不是最好的解法，没有用到有序的条件，比较好的解法应该是小根堆（本来打算用Go撸一个的，写一半感觉太麻烦了，不过整体小根堆的逻辑实现是对的，就是没有泛型要改很多东西，不太方便）
![](https://i.loli.net/2020/11/30/Q5KsxNi3MquZtWE.png)
维护两个最值，一个是找到一个能覆盖当前所有列表的最小的右端点（max），一个是当前列表最小的那个元素（最左的端点），然后不断缩减左端点求最小区间
```java
class Node {
    int i, j;
    public Node (int i, int j) {
        this.i = i;
        this.j = j;
    }
}

//k组链表，平均m个元素，时间复杂度 O(kmlog(k))
public int[] smallestRange(List<List<Integer>> nums) {
    PriorityQueue<Node> pq = new PriorityQueue<>((a, b) -> nums.get(a.i).get(a.j)-nums.get(b.i).get(b.j));
    int INF = (int) 1e5+1;
    int max = Integer.MIN_VALUE;
    for (int i = 0; i < nums.size(); i++) {
        pq.add(new Node(i, 0));
        max = Math.max(max, nums.get(i).get(0));
    }
    int [] res = {-INF, INF};
    while (true) {
        Node cur = pq.poll();
        //应该把val也存进去的，懒得改了
        if (max-nums.get(cur.i).get(cur.j) < res[1]-res[0]) {
            res[0] = nums.get(cur.i).get(cur.j);
            res[1] = max;
        }
        if (cur.j+1 >= nums.get(cur.i).size()) {
            break;
        }
        pq.add(new Node(cur.i, cur.j+1));
        max = Math.max(max, nums.get(cur.i).get(cur.j+1));
    }
    return res;
}
```

## [438. 找到字符串中所有字母异位词](https://leetcode-cn.com/problems/find-all-anagrams-in-a-string/)

给定一个字符串 **s** 和一个非空字符串 **p**，找到 **s** 中所有是 **p** 的字母异位词的子串，返回这些子串的起始索引。

字符串只包含小写英文字母，并且字符串 **s** 和 **p** 的长度都不超过 20100。

**说明：**

- 字母异位词指字母相同，但排列不同的字符串。
- 不考虑答案输出的顺序。

**示例 1:**

```java
输入:
s: "cbaebabacd" p: "abc"
输出:
[0, 6]

解释:
起始索引等于 0 的子串是 "cba", 它是 "abc" 的字母异位词。
起始索引等于 6 的子串是 "bac", 它是 "abc" 的字母异位词。
```

 **示例 2:**

```java
输入:
s: "abab" p: "ab"

输出:
[0, 1, 2]

解释:
起始索引等于 0 的子串是 "ab", 它是 "ab" 的字母异位词。
起始索引等于 1 的子串是 "ba", 它是 "ab" 的字母异位词。
起始索引等于 2 的子串是 "ab", 它是 "ab" 的字母异位词。
```

**解法一**

这题和上面一题其实一样，只是这里要求是连续的，在上面一题的基础上在添加结果的时候判断下长度就OK

```java
public List<Integer> findAnagrams(String s, String p) {
    int[] target=new int[256];
    int[] window=new int[256];
    int l=0,r=0;
    int plen=p.length();
    int slen=s.length();
    int count=0,match=0;
    List<Integer> res=new ArraysList<>();
    for (int i=0;i<plen;i++) {
        target[p.charAt(i)]++;
    }
    for (int a:target) {
        if(a!=0){
            count++;
        }
    }

    while(r<slen){
        char right=s.charAt(r);
        if (target[right]!=0) {
            window[right]++;
            if (window[right]==target[right]) {
                match++;
            }
        }
        r++;
        while(l<r && count==match){
            char left=s.charAt(l);
            if (r-l==res) {
                res.add(l);
            }
            l++;
            if (target[left]!=0) {
                window[left]--;
                //不满足了
                if (window[left]<target[left]) {
                    match--;
                }
            }
        }
    }
    return res;
}
```
**UPDATE:2020.7.23**

用模板重写了下
```golang
func findAnagrams(s string, p string) []int {
    var target = make([]int, 128)
    var window = make([]int, 128)
    var match = 0
    for _, sp := range p {
        if target[sp] == 0 {
            match++
        }
        target[sp]++
    }
    var left = 0
    var count = 0
    var res []int
    for right := 0; right < len(s); right++ {
        if target[s[right]] > 0 {
            window[s[right]]++
            if window[s[right]] == target[s[right]] {
                count++
            }
        }
        for count == match && left <= right {
            if right-left+1 == len(p) {
                res = append(res, left)
            }
            if window[s[left]] > 0 {
                window[s[left]]--
                if window[s[left]] < target[s[left]] {
                    count--
                }
            }
            left++
        }
    }
    return res
}
```
15ms，86%时间复杂度`O(M+N)`，其实是完全套的之前最小覆盖子串的模板，不如肯定不好写这么长

**解法二**

正常的做法，实际上上面的解法一直在避免直接比较target和window，但是实际上比较这两个数组的成本是很低的，两个数组长度固定，比较时间复杂度O(1)，具体情况具体分析，不过套模板几乎是通用的
```golang
func findAnagrams(s string, p string) []int {
    var left = 0
    //-'a'看起来太丑了，直接128
    var target [128]int //注意用数组，可以直接比较
    var window [128]int
    for _, sp := range p {
        target[sp]++
    }
    var res []int
    for right := 0; right < len(s); right++ {
        if target[s[right]] > 0 {
            window[s[right]]++
        }
        for right-left+1 > len(p) {
            if window[s[left]] > 0 {
                window[s[left]]--
            }
            left++
        }
        if window == target {
            res = append(res, left)
        }
    }
    return res
}
```

## [567. 字符串的排列](https://leetcode-cn.com/problems/permutation-in-string/)

给定两个字符串 **s1** 和 **s2**，写一个函数来判断 **s2** 是否包含 **s1** 的排列。

换句话说，第一个字符串的排列之一是第二个字符串的子串。

**示例1:**

```java
输入: s1 = "ab" s2 = "eidbaooo"
输出: True
解释: s2 包含 s1 的排列之一 ("ba").
```

**示例2:**

```java
输入: s1= "ab" s2 = "eidboaoo"
输出: False
```

**注意：**

1. 输入的字符串只包含小写字母
2. 两个字符串的长度都在 [1, 10,000] 之间

**解法一**

这题其实是 [76.最小覆盖子串](#76-最小覆盖子串) 的弱化版本，套路滑窗，但是有一些细节需要注意

```java
public boolean checkInclusion(String s1, String s2) {
    if(s1==null || s2==null || s1.length()>s2.length()){
        return false;
    }
    int n1=s1.length();
    int n2=s2.length();
    int[] freq=new int[26];
    int count=0;
    for(int i=0;i<n1;i++){
        int c=s1.charAt(i)-'a';
        if(freq[c]==0){
            count++;
        }
        freq[c]++;
    }
    int[] window=new int[26];
    int match=0;
    int left=0;
    for(int right=0;right<n2;right++){
        int cr=s2.charAt(right)-'a';
        if(freq[cr]>0){
            window[cr]++;
            if(window[cr]==freq[cr]){
                match++;
            }
        }
        while(right-left+1>n1){
            int cl=s2.charAt(left)-'a';
            if(freq[cl]>0){
                //WA点，开始写错了
                //window[cl]--;
                if(window[cl]==freq[cl]){
                    match--; //match--的前提是原本是匹配的
                }
                window[cl]--;
            }
            left++;
        } 
        if(match==count){
            return true;
        }
    }
    return false;
}
```

一开始写完了，测试了用例发现都是对的，心里一喜，难道又bugfree了？提交，结果还是WA了🤣，debug了半天，根据错误的哪个长case自己构造了一个短case（长的不好debug），然后发现了问题，这个问题我看见评论区也有人问到，顺手还回了一下😁，其实错误的原因就是套模板没套好，没理解模板的细节

这里的核心问题就是`match--`的时候有一个大前提：`cl`字符已经匹配好了，也就是`window[cl]==freq[cl]`

这个时候你`left++`将`cl`移除窗口，才能做`match--`操作，否则像之前的模板中

```java
if (target[left]!=0) {
    window[left]--;
    //不满足了
    if (window[left]<target[left]) {
        match--;
    }
}
```

这样写就有可能window[left]还没有匹配上，这个时候直接减就错了，之前的模板中缩圈都是在match==count的前提下缩圈的，所以没问题，都是匹配的，其实也可以像之前的模板一样写，就像下面这样

```java
public boolean checkInclusion(String s1, String s2) {
    if(s1==null || s2==null || s1.length()>s2.length()){
        return false;
    }
    int n1=s1.length();
    int n2=s2.length();
    int[] freq=new int[26];
    int count=0;
    for(int i=0;i<n1;i++){
        int c=s1.charAt(i)-'a';
        if(freq[c]==0){
            count++;
        }
        freq[c]++;
    }
    int[] window=new int[26];
    int match=0;
    int left=0;
    for(int right=0;right<n2;right++){
        int cr=s2.charAt(right)-'a';
        if(freq[cr]>0){
            window[cr]++;
            if(window[cr]==freq[cr]){
                match++;
            }
        }
        //****************************
        //主要就是这里不一样
        while(match==count){
            if(right-left+1==n1) return true;
            int cl=s2.charAt(left)-'a';
            if(freq[cl]>0){
                window[cl]--;
                if(window[cl]<freq[cl]){
                    match--;
                }
            }
            left++;
        } 
        //****************************
    }
    return false;
}
```
这样就不用考虑先减还是后减的问题了

**解法二**

这题其实还可以暴力做，窗口大小固定，每滑动一次就判断窗口和`freq`是不是相等，因为题目说了都是小写字母所以也是行得通的，只是常数会大一些，这里我就不写了，笔试推荐这样写，代码好写一点

## [面试题 17.18. 最短超串](https://leetcode-cn.com/problems/shortest-supersequence-lcci/)

假设你有两个数组，一个长一个短，短的元素均不相同。找到长数组中包含短数组所有的元素的最短子数组，其出现顺序无关紧要。

返回最短子数组的左端点和右端点，如有多个满足条件的子数组，返回左端点最小的一个。若不存在，返回空数组。

**示例 1:**

```java
输入:
big = [7,5,9,0,2,1,3,5,7,9,1,1,5,8,8,9,7]
small = [1,5,9]
输出: [7,10]
```

**示例 2:**

```java
输入:
big = [1,2,3]
small = [4]
输出: []
```

**提示：**

- `big.length <= 100000`
- `1 <= small.length <= 100000`

**解法一**

也是属于 [76.最小覆盖子串](#76-最小覆盖子串)的弱化，虽然解法都一样，没啥好说的，注意细节

```java
//没啥好说的，套模板就行了
public int[] shortestSeq(int[] big, int[] small) {
    if(big==null || big.length<=0) {
        return new int[0];
    }
    int slen=small.length;
    int blen=big.length;
    int[] res=new int[2];
    res[0]=-1;res[1]=-1;
    int min=Integer.MAX_VALUE;
    HashSet<Integer> set=new HashSet<>();
    for(int i:small) set.add(i);
    HashMap<Integer,Integer> window=new HashMap<>();
    int match=0;
    int left=0;
    for(int right=0;right<blen;right++){
        int wr=big[right];
        if(set.contains(wr)){
            window.put(wr,window.getOrDefault(wr,0)+1);
            if(window.get(wr)==1){
                match++;
            }
        }
        while(match==slen){
            if(right-left+1<min){
                res[0]=left;
                res[1]=right;
                min=right-left+1;
            }
            int wl=big[left];
            if(set.contains(wl)){
                window.put(wl,window.get(wl)-1);
                if(window.get(wl)==0){
                    match--;
                }
            }
            left++;
        }
    }
    return res[0]==-1?new int[0]:res;
}
```

## [1004. 最大连续1的个数 III](https://leetcode-cn.com/problems/max-consecutive-ones-iii/)

给定一个由若干 `0` 和 `1` 组成的数组 `A`，我们最多可以将 `K` 个值从 0 变成 1 。

返回仅包含 1 的最长（连续）子数组的长度。

**示例 1：**

```java
输入：A = [1,1,1,0,0,0,1,1,1,1,0], K = 2
输出：6
解释： 
[1,1,1,0,0,1,1,1,1,1,1]
粗体数字从 0 翻转到 1，最长的子数组长度为 6。
```

**示例 2：**

```java
输入：A = [0,0,1,1,0,0,1,1,1,0,1,1,0,0,0,1,1,1,1], K = 3
输出：10
解释：
[0,0,1,1,1,1,1,1,1,1,1,1,0,0,0,1,1,1,1]
粗体数字从 0 翻转到 1，最长的子数组长度为 10。
```

**提示：**

1. `1 <= A.length <= 20000`
2. `0 <= K <= A.length`
3. `A[i]` 为 `0` 或 `1` 

**解法一**

这题其实下面一题[424. 替换后的最长重复字符](#424-替换后的最长重复字符)的弱化版本

```java
//简单的滑窗
public int longestOnes(int[] A, int K) {
    if(A==null || A.length<=0) return 0;
    int N=A.length;
    int left=0,res=0,countA=0;
    for(int right=0;right<N;right++){
        countA+=(A[right]&1);
        //if也可以，个人喜欢while通用性更强
        while(right-left+1-countA>K){ 
            countA-=(A[left++]&1);
        }
        res=Math.max(res,right-left+1);
    }
    return res;
}
```
按照模板来写简直是信手拈来（其实也没有什么模板，就是规范了写法而已）

## [5434. 删掉一个元素以后全为 1 的最长子数组](https://leetcode-cn.com/problems/longest-subarray-of-1s-after-deleting-one-element/)

Difficulty: **中等**


给你一个二进制数组 `nums` ，你需要从中删掉一个元素。

请你在删掉元素的结果数组中，返回最长的且只包含 1 的非空子数组的长度。

如果不存在这样的子数组，请返回 0 。

**提示 1：**

```java
输入：nums = [1,1,0,1]
输出：3
解释：删掉位置 2 的数后，[1,1,1] 包含 3 个 1 。
```

**示例 2：**

```java
输入：nums = [0,1,1,1,0,1,1,0,1]
输出：5
解释：删掉位置 4 的数字后，[0,1,1,1,1,1,0,1] 的最长全 1 子数组为 [1,1,1,1,1] 。
```

**示例 3：**

```
输入：nums = [1,1,1]
输出：2
解释：你必须要删除一个元素。
```

**示例 4：**

```java
输入：nums = [1,1,0,0,1,1,1,0,1]
输出：4
```

**示例 5：**

```java
输入：nums = [0,0,0]
输出：0
```

**提示：**

*   `1 <= nums.length <= 10^5`
*   `nums[i]` 要么是 `0` 要么是 `1` 。

**解法一**

29th双周赛的t3，秒切，其实是上一题[1004. 最大连续1的个数 III](#1004-最大连续1的个数-iii)的弱化
```java
   public int longestSubarray(int[] nums) {
       int res=0, sum=0;
       int left=0;
       for(int right=0;right<nums.length;right++){
           sum+=(nums[right]&1);
           //区间和小于right-left-1说明中间不止一个0需要缩减窗口
           while(left<right && sum<= right-left-1){
               if(nums[left] == 1){
                   sum--;
               }
               left++;
           }
           //至少删除一个，所以不用+1
           res = Math.max(res,right-left);
       }
       return res;
   }
```
很可惜这次前3题15分钟就做完了，都是直接web上写的，本以为有机会AK，最后一题搞了半天最后交了一发还是没过，貌似很多人写了假算法，贪心莽过了（lc数据太弱了）
![UTOOLS1593333299362.png](https://upload.cc/i1/2020/06/28/mhTiQ9.png)

## [424. 替换后的最长重复字符](https://leetcode-cn.com/problems/longest-repeating-character-replacement/)

给你一个仅由大写英文字母组成的字符串，你可以将任意位置上的字符替换成另外的字符，总共可最多替换 k 次。在执行上述操作后，找到包含重复字母的最长子串的长度。

**注意:**
字符串长度 和 k 不会超过 104。

**示例 1:**

```java
输入:
s = "ABAB", k = 2
输出:
4
解释:
用两个'A'替换为两个'B',反之亦然。
```

**示例 2:**

```java
输入:
s = "AABABBA", k = 1
输出:
4
解释:
将中间的一个'A'替换为'B',字符串变为 "AABBBBA"
子串 "BBBB" 有最长重复字母, 答案为 4
```

**解法一**

上面一题的加强版

```java
public int characterReplacement(String s, int k) {
    int max = 0, start = 0, end = 0, cur = -1;
    int[] count = new int[256];
    while (end < s.length()) {
        //当前窗口出现最多的字符
        cur = Math.max(cur, ++count[s.charAt(end)]);
        //不能替换了,不同字符太多了,需要缩减窗口
        while (end - start + 1 - cur > k){
            //缩减左边界的count
            count[s.charAt(start)]--;
            start++;//不能替换了，start++
        }
        //统计最大值
        max = Math.max(max, end - start + 1);
        end++;
    }
    return max;
}
```

**UPDATE: (2020.5.5)**

按照模板重写，果然滑窗的题都是一样的

```java
public int characterReplacement(String s, int k) {
    if(s==null || s.length()<=0){
        return 0;
    }
    int n=s.length();
    int res=1;
    int left=0;
    int[] freq=new int[128];
    int maxFreq=0;
    for(int right=0;right<n;right++){
        char c=s.charAt(right);
        freq[c]++;
        maxFreq=Math.max(maxFreq,freq[c]);
        while((right-left+1-maxFreq)>k){
            freq[s.charAt(left)]--;
            left++; //这里实际上只会执行一次，改成if也是可以的，不过为了统一写法就不改了
        }
        res=Math.max(res,right-left+1);
    }
    return res;
}
```

重写这题的时候发现这题还是挺有意思的，这个里面的`maxFreq`是一个只增不减的量，是一个历史最大值，只有当出现更大的freq的时候才会更新`maxFreq`，当`maxFreq`保持不变的时候结果不会受到影响，只有出现了更大freq的时候才有可能会使结果变大

> [拷贝自题解区](https://leetcode-cn.com/problems/longest-repeating-character-replacement/solution/hua-dong-chuang-kou-chang-gui-tao-lu-by-xiaoneng/) 
>
> 因为我们只对最长有效的子字符串感兴趣，所以我们的滑动窗口不需要收缩，即使窗口可能覆盖无效的子字符串。我们可以通过在右边添加一个字符来扩展窗口，或者将整个窗口向右边移动一个字符。而且我们只在新字符的计数超过历史最大计数(来自覆盖有效子字符串的前一个窗口)时才增长窗口。也就是说，我们不需要精确的当前窗口的最大计数;我们只关心最大计数是否超过历史最大计数;这只会因为新字符而发生。

**解法二**

憨憨的解法，不过绝对是能AC的，时间复杂度并没有问题，依然是`O(N)`

```java
public int characterReplacement(String s, int k) {
    if(s==null || s.length()<=0){
        return 0;
    }
    int n=s.length();
    int res=1;
    for(int c='A';c<='Z';c++){
        int[] freq=new int[128];
        int temp=1;
        int left=0;
        for(int right=0;right<n;right++){
            if(s.charAt(right)==c){
                freq[c]++;
            }
            while((right-left+1-freq[c])>k){
                if(s.charAt(left)==c){
                    freq[c]--;
                }
                left++;
            }
            temp=Math.max(temp,right-left+1);
        }
        res=Math.max(res,temp);
    }
    return res;
}
```
既然题目说了只有大写字母，那就直接枚举所有的字符然后滑窗就行了😂，简单直白

## [1234. 替换子串得到平衡字符串](https://leetcode-cn.com/problems/replace-the-substring-for-balanced-string/)

有一个只含有 `'Q', 'W', 'E', 'R'` 四种字符，且长度为 n 的字符串。

假如在该字符串中，这四个字符都恰好出现 `n/4` 次，那么它就是一个「平衡字符串」。

给你一个这样的字符串 `s`，请通过「替换子串」的方式，使原字符串 s 变成一个「平衡字符串」。

你可以用和「待替换子串」长度相同的 任何 其他字符串来完成替换。

请返回待替换子串的最小可能长度。

如果原字符串自身就是一个平衡字符串，则返回 0 

**示例 1：**

```java
输入：s = "QWER"
输出：0
解释：s 已经是平衡的了。
```

**示例 2：**

```java
输入：s = "QQWE"
输出：1
解释：我们需要把一个 'Q' 替换成 'R'，这样得到的 "RQWE" (或 "QRWE") 是平衡的。
```

**示例 3：**

```java
输入：s = "QQQW"
输出：2
解释：我们可以把前面的 "QQ" 替换成 "ER"。 
```

**示例 4：**

```java
输入：s = "QQQQ"
输出：3
解释：我们可以替换后 3 个 'Q'，使 s = "QWER"。
```

**提示：**

- 1 <= s.length <= 10^5
- s.length 是 4 的倍数
- s 中只含有 'Q', 'W', 'E', 'R' 四种字符

**解法一**

周赛题，说实话不多做做竞赛真不知道自己多菜

(update: 2020.4.15)

我拿到这题，首先想到的是无脑套路滑窗，既然要保证平衡，那么每个字符出现的次数都应该是`N/4`，所以我们可以统计下多出来的有几个，比如`QQQW`，那么多出来的就是2个**`Q`**，也就是说我们要求的窗口内**至少**有2个Q，这样问题其实就转换成了 [76. 最小覆盖子串](#76-最小覆盖子串)（这明明是个mid题，你咋还给转换成hard了，你是不是傻🤣）

其实最小覆盖子串看起来好像挺难，但是是有套路的，我们直接套模板就可以了

```java
public int balancedString(String s) {
    if(s==null || s.length()<=0) return -1;
    int N=s.length();
    //这里用26有的浪费,为了方便写代码,就这样吧
    int[] need=new int[26];
    //初始化为-N/4这样最后得到的大于0的值就是多出来的
    Arrays.fill(need,-N/4);
    int[] cur=new int[26];
    for(int i=0;i<N;i++){
        need[s.charAt(i)-'A']++;
    }
    //有几个字符多出来了
    int needCount=0; 
    for(int i=0;i<need.length;i++){
        if(need[i]>0) needCount++;
    } 
    if(needCount==0) return 0;
    int res=N;
    int left=0,right=0;
    int matchCount=0;
    //无脑套路滑窗
    while(right<s.length()){
        char c=s.charAt(right);
        if(need[c-'A']>0){
            cur[c-'A']++;
            if(cur[c-'A']==need[c-'A']){
                matchCount++;
            }
        }
        while(left<=right && matchCount==needCount){
            res=Math.min(right-left+1,res);
            char cl=s.charAt(left);
            if(need[cl-'A']>0){
                cur[cl-'A']--;
                if(cur[cl-'A']<need[cl-'A']){
                    matchCount--;
                }
            }
            left++;
        }
        right++;
    }
    return res;
}
```

**解法二**

上面的解法是考虑`窗口内`的元素组成，窗口内至少应该有哪些元素，反过来想，我们窗口内的元素是多出来的元素，我们是把多的元素放到窗口中，那么窗口外的元素就肯定都是`小于等于N/4`的了，那么我们就可以利用这一点进行滑窗，统计符合条件的窗口的最小值，这样代码就会简洁很多

```java
//code删掉了，之前的代码有点问题，最后的返回值有些情况过不去，lc的case太弱了，让我过了
```

**UPDATE: (2020.5.4)**

按照先前的模板来分析下，这里要求的是最小的修改次数，很明显不能用`for-if`，所以采用`for-while`的结构，`for-while`最基本的结构就是外层枚举所有`right`，内层根据题目要求缩减`left`，但是这题left也需要控制边界，我这里用的是`left<=right` 但是实际上这样会有一类case结果不对，比如"QWER"这样的，返回的结果是1，~~我上面在最后做了特判~~（上面的特判是错的，比如“QWEE”这样的就返回0），其实这里还可以修改下边界，改成`left<N`，这样就没问题了，这样left就可以超过right达到right+1，这样对”QWER"就能得到正确的结果，并且根据题目信息当`left=right+1`之后`left`就不会再增加了，while条件就无法满足了，但是有的题目`left`是不用设置限制的，基本上都是在达到right+1之后就不会继续增加了

```java
class Solution {
    public int balancedString(String s) {
        if(s==null || s.length()<=0){
            return 0;
        }
        int N=s.length();
        int left=0;
        int res=N;
        int[] freq=new int[26];
        for(int i=0;i<N;i++) {
            freq[s.charAt(i)-'A']++;
        }
        for(int right=0;right<N;right++){
            freq[s.charAt(right)-'A']--;
            while(left<N && freq['Q'-'A']<=N/4 && freq['W'-'A']<=N/4 && freq['E'-'A']<=N/4 && freq['R'-'A']<=N/4){
                res=Math.min(res,right-left+1);
                freq[s.charAt(left++)-'A']++;
            }
        }
        return res;
    }
}
```

## [1358. 包含所有三种字符的子字符串数目](https://leetcode-cn.com/problems/number-of-substrings-containing-all-three-characters/)  

给你一个字符串 s ，它只包含三种字符 a, b 和 c 。

请你返回 a，b 和 c 都 **至少** 出现过一次的子字符串数目。

**示例 1：**

```java
输入：s = "abcabc"
输出：10
解释：包含 a，b 和 c 各至少一次的子字符串为 "abc", "abca", "abcab", "abcabc", "bca", "bcab", "bcabc", "cab", "cabc" 和 "abc" (相同字符串算多次)。
```

**示例 2：**

```java
输入：s = "aaacb"
输出：3
解释：包含 a，b 和 c 各至少一次的子字符串为 "aaacb", "aacb" 和 "acb" 。
```

**示例 3：**

```java
输入：s = "abc"
输出：1
```

**提示：**

- 3 <= s.length <= 5 x 10^4
- s 只包含字符 a，b 和 c 。

**解法一**

20双周赛T3

```java
public int numberOfSubstrings(String s) {
    int[] freq=new int[3];
    int left=0,right=-1,slen=s.length();
    int res=0;
    //abc
    while(left<slen-2){
        while(right+1<slen && !valid(freq)){
            freq[s.charAt(++right)-'a']++;
        }
        res+=valid(freq)?(slen-right):0;
        freq[s.charAt(left)-'a']--;
        left++;
    }
    return res;
}

public boolean valid(int[] freq){
    return freq[0]!=0 && freq[1]!=0 && freq[2]!=0;
}
```
枚举所有的左边界，然后找到最短的可以满足的右边界，那么包括右边界和之后的所有的都满足条件，直接计算就可以了，时间复杂度`O(N)`

**解法二**

2020.5.4 用自己总结的滑窗模板重写，也是`for-while`结构，right和left不能同时扩展。比如`"aaacb"`这样的case

```java
public int numberOfSubstrings(String s) {
    int left=0;
    int res=0;
    int n=s.length();
    int[] freq=new int[3];
    for(int right=0;right<n;right++){
        freq[s.charAt(right)-'a']++;
        while(freq[0]>0 && freq[1]>0 && freq[2]>0){
            res+=n-right; //后面的都符合条件
            freq[s.charAt(left)-'a']--;
            left++;
        }
    }
    return res;
}
```
## [1423. 可获得的最大点数](https://leetcode-cn.com/problems/maximum-points-you-can-obtain-from-cards/)

几张卡牌 **排成一行**，每张卡牌都有一个对应的点数。点数由整数数组 `cardPoints` 给出。

每次行动，你可以从行的开头或者末尾拿一张卡牌，最终你必须正好拿 `k` 张卡牌。

你的点数就是你拿到手中的所有卡牌的点数之和。

给你一个整数数组 `cardPoints` 和整数 `k`，请你返回可以获得的最大点数。

**示例 1：**

```java
输入：cardPoints = [1,2,3,4,5,6,1], k = 3
输出：12
解释：第一次行动，不管拿哪张牌，你的点数总是 1 。但是，先拿最右边的卡牌将会最大化你的可获得点数。最优策略是拿右边的三张牌，最终点数为 1 + 6 + 5 = 12 。
```

**示例 2：**

```java
输入：cardPoints = [2,2,2], k = 2
输出：4
解释：无论你拿起哪两张卡牌，可获得的点数总是 4 。
```

**示例 3：**

```java
输入：cardPoints = [9,7,7,9,7,7,9], k = 7
输出：55
解释：你必须拿起所有卡牌，可以获得的点数为所有卡牌的点数之和。
```

**示例 4：**

```java
输入：cardPoints = [1,1000,1], k = 1
输出：1
解释：你无法拿到中间那张卡牌，所以可以获得的最大点数为 1 。 
```

**示例 5：**

```java
输入：cardPoints = [1,79,80,1,1,1,200,1], k = 3
输出：202
```

**提示：**

- `1 <= cardPoints.length <= 10^5`
- `1 <= cardPoints[i] <= 10^4`
- `1 <= k <= cardPoints.length`

**解法一**

186周赛T2，很明显的滑动窗口，前后拿K张最大，只需要求一个最小的`[n-k]`区间值就行了，也可以用前缀和，思路都一样

```go
func maxScore(cardPoints []int, k int) int {
    if cardPoints == nil || len(cardPoints) == 0 {
        return 0
    }
    n := len(cardPoints)
    left := 0
    right := n - k - 1
    sum := 0
    windowSum := 0
    for i, num := range cardPoints {
        sum += num
        if i == right {
            windowSum = sum
        }
    }
    if k == n {
        return sum
    }
    minWin := windowSum
    for right+1 < n {
        windowSum += (cardPoints[right+1] - cardPoints[left])
        minWin = min(windowSum, minWin)
        right++
        left++
    }
    return sum - minWin
}

func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}

```

Tag里面有dp，确实这题和前面的[石子游戏](http://imlgw.top/2019/09/01/leetcode-dong-tai-gui-hua/#877-%E7%9F%B3%E5%AD%90%E6%B8%B8%E6%88%8F)有一点像，但是还是滑窗来的比较直接。。

## [1208. 尽可能使字符串相等](https://leetcode-cn.com/problems/get-equal-substrings-within-budget/)

给你两个长度相同的字符串，`s` 和 `t`。

将 `s` 中的第 `i` 个字符变到 `t` 中的第 `i` 个字符需要 `|s[i] - t[i]|` 的开销（开销可能为 0），也就是两个字符的 ASCII 码值的差的绝对值。

用于变更字符串的最大预算是 `maxCost`。在转化字符串时，总开销应当小于等于该预算，这也意味着字符串的转化可能是不完全的。

如果你可以将 `s` 的子字符串转化为它在 `t` 中对应的子字符串，则返回可以转化的最大长度。

如果 `s` 中没有子字符串可以转化成 `t` 中对应的子字符串，则返回 `0`。

**示例 1：**

```go
输入：s = "abcd", t = "bcdf", cost = 3
输出：3
解释：s 中的 "abc" 可以变为 "bcd"。开销为 3，所以最大长度为 3。
```

**示例 2：**

```go
输入：s = "abcd", t = "cdef", cost = 3
输出：1
解释：s 中的任一字符要想变成 t 中对应的字符，其开销都是 2。因此，最大长度为 1。
```

**示例 3：**

```go
输入：s = "abcd", t = "acde", cost = 0
输出：1
解释：你无法作出任何改动，所以最大长度为 1。
```

**提示：**

- `1 <= s.length, t.length <= 10^5`
- `0 <= maxCost <= 10^6`
- `s` 和 `t` 都只含小写英文字母。

**解法一**

```go
func equalSubstring(s string, t string, maxCost int) int {
    left := 0
    cost := 0
    res := 0
    for right := 0; right < len(s); right++ {
        cost += getCost(s[right], t[right])
        if cost <= maxCost {
            res = max(res, right-left+1)
        } else {
            cost -= getCost(s[left], t[left])
            left++
        }
    }
    return res
}

func getCost(a, b byte) int {
    if a < b { //a-b<0 byte是uint8直接这样减会变成正数
        return int(b - a)
    }
    return int(a - b)
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

这个题本身不难，主要是为了学习滑窗类型的题，这题我又改了好长时间，果然做滑窗的题还是有点乱，不过做了这一题之后已经有点感觉了，目前打算把所有的滑窗都重做一遍，然后总结一下套路

**思考**

下面`for-while`的结构似乎更加统一，上面`for-if`的结构只能用在求**最长，最大**的情况下，这种时候`left`和`right`允许同时加加，所以用`if`也是可以的，但是求最短的时候，比如上面[209. 长度最小的子数组](#209-长度最小的子数组)就不能用`for-if` ，当right到达边界的时候left可能还需要继续移动，所以不能用`if`

```go
func equalSubstring(s string, t string, maxCost int) int {
    left := 0
    cost := 0
    res := 0
    for right := 0; right < len(s); right++ {
        cost += getCost(s[right], t[right])
        for cost > maxCost {
            cost -= getCost(s[left], t[left])
            left++
        }
        res = max(res, right-left+1)
    }
    return res
}
```

## [1052. 爱生气的书店老板](https://leetcode-cn.com/problems/grumpy-bookstore-owner/)

今天，书店老板有一家店打算试营业 `customers.length` 分钟。每分钟都有一些顾客（`customers[i]`）会进入书店，所有这些顾客都会在那一分钟结束后离开。

在某些时候，书店老板会生气。 如果书店老板在第 `i` 分钟生气，那么 `grumpy[i] = 1`，否则 `grumpy[i] = 0`。 当书店老板生气时，那一分钟的顾客就会不满意，不生气则他们是满意的。

书店老板知道一个秘密技巧，能抑制自己的情绪，可以让自己连续 `X` 分钟不生气，但却只能使用一次。

请你返回这一天营业下来，最多有多少客户能够感到满意的数量。

**示例：**

```
输入：customers = [1,0,1,2,1,1,7,5], grumpy = [0,1,0,1,0,1,0,1], X = 3
输出：16
解释：
书店老板在最后 3 分钟保持冷静。
感到满意的最大客户数量 = 1 + 1 + 1 + 1 + 7 + 5 = 16.
```

**提示：**

- `1 <= X <= customers.length == grumpy.length <= 20000`
- `0 <= customers[i] <= 1000`
- `0 <= grumpy[i] <= 1`

**解法一**

滑动窗口的感觉来了，越来越熟练了，这题直接bugfree了😁

```java
public int maxSatisfied(int[] customers, int[] grumpy, int X) {
    if(grumpy==null || grumpy.length<0) return 0;
    int N=grumpy.length;
    int left=0;
    int window=0;//窗口内反转人数
    int max=0; //最多反转人数
    for(int right=0;right<N;right++){
        if(grumpy[right]==1){
            window+=customers[right];
        }
        //while和if都可以,个人比较喜欢while通用性比较强
        while(right-left+1>X){
            if(grumpy[left]==1){
                window-=customers[left];
            }
            left++;
        }
        max=Math.max(window,max);
    }
    int res=0;
    for(int i=0;i<N;i++){
        res+=(grumpy[i]==0?customers[i]:0);
    }
    return res+max;
}
```
可以看到仍然是前面总结的`for-while`结构，等我把所有的滑窗tag做完了再来总结一波

## [1040. 移动石子直到连续 II](https://leetcode-cn.com/problems/moving-stones-until-consecutive-ii/)

在一个长度**无限**的数轴上，第 `i` 颗石子的位置为 `stones[i]`。如果一颗石子的位置最小/最大，那么该石子被称作**端点石子**。

每个回合，你可以将一颗端点石子拿起并移动到一个未占用的位置，使得该石子不再是一颗端点石子。

值得注意的是，如果石子像 `stones = [1,2,5]` 这样，你将**无法**移动位于位置 5 的端点石子，因为无论将它移动到任何位置（例如 0 或 3），该石子都仍然会是端点石子。

当你无法进行任何移动时，即，这些石子的位置连续时，游戏结束。

要使游戏结束，你可以执行的最小和最大移动次数分别是多少？ 以长度为 2 的数组形式返回答案：`answer = [minimum_moves, maximum_moves]` 。

**示例 1：**

```java
输入：[7,4,9]
输出：[1,2]
解释：
我们可以移动一次，4 -> 8，游戏结束。
或者，我们可以移动两次 9 -> 5，4 -> 6，游戏结束。
```

**示例 2：**

```java
输入：[6,5,4,3,10]
输出：[2,3]
解释：
我们可以移动 3 -> 8，接着是 10 -> 7，游戏结束。
或者，我们可以移动 3 -> 7, 4 -> 8, 5 -> 9，游戏结束。
注意，我们无法进行 10 -> 2 这样的移动来结束游戏，因为这是不合要求的移动。
```

**示例 3：**

```java
输入：[100,101,104,102,103]
输出：[0,0]
```

**提示：**

1. `3 <= stones.length <= 10^4`
2. `1 <= stones[i] <= 10^9`
3. `stones[i]` 的值各不相同。

**解法一**

懵逼，某次周赛的T4，嗯抄，不会做，题目都差点没看懂

```java
public int[] numMovesStonesII(int[] stones) {
    if(stones==null || stones.length<=0){
        return new int[2];
    }
    //0 1 2 3 4 5 6 7 8 9 10
    //      0 1 2 3        4
    int N=stones.length;
    Arrays.sort(stones);
    int left=0;
    int[] res=new int[2];
    res[0]=Integer.MAX_VALUE;
    for(int right=0;right<N;right++){
        //整个区间范围大于N了需要缩小区间
        while(stones[right]-stones[left]+1>N){ 
            left++;
        }
        int windowStones=right-left+1;
        if(windowStones==N-1&&stones[right]-stones[left]+1==N-1){
            res[0]=Math.min(2,res[0]);
        }else{
            res[0]=Math.min(res[0],N-windowStones);
        }
    }
    res[1]=Math.max(stones[N-1]-stones[1]-N+2,stones[N-2]-stones[0]-N+2);
    return res;
}
```

## [1456. 定长子串中元音的最大数目](https://leetcode-cn.com/problems/maximum-number-of-vowels-in-a-substring-of-given-length/)

给你字符串 s 和整数 k 。

请返回字符串 s 中长度为 k 的单个子字符串中可能包含的最大元音字母数。

英文中的 元音字母 为（a, e, i, o, u）。

**示例 1：**

```java
输入：s = "abciiidef", k = 3
输出：3
解释：子字符串 "iii" 包含 3 个元音字母。
```
**示例 2：**

```java
输入：s = "aeiou", k = 2
输出：2
解释：任意长度为 2 的子字符串都包含 2 个元音字母。
```
**示例 3：**

```java
输入：s = "leetcode", k = 3
输出：2
解释："lee"、"eet" 和 "ode" 都包含 2 个元音字母。
```
**示例 4：**

```java
输入：s = "rhythms", k = 4
输出：0
解释：字符串 s 中不含任何元音字母。
```
**示例 5：**

```java
输入：s = "tryhard", k = 4
输出：1
```

**提示：**

- 1 <= s.length <= 10^5
- s 由小写英文字母组成
- 1 <= k <= s.length

**解法一**

好久没写滑窗的题了，回顾下之前的模板，`for-while`结构
```java
public int maxVowels(String s, int k) {
    int left=0;
    int res=0;
    int count=0;
    for(int right=0;right<s.length();right++){
        if(vowel(s.charAt(right))){
            count++;
        }
        while(right-left >= k){
            if(vowel(s.charAt(left))){
                count--;
            }
            left++;
        }
        res=Math.max(res,count);
    }
    return res;
}

public boolean vowel(char ch){
    return ch=='a' || ch=='e' || ch=='i' || ch=='o' || ch=='u';
}

```

## [1461. 检查一个字符串是否包含所有长度为 K 的二进制子串](https://leetcode-cn.com/problems/check-if-a-string-contains-all-binary-codes-of-size-k/)

Difficulty: **中等**


给你一个二进制字符串 `s` 和一个整数 `k` 。

如果所有长度为 `k` 的二进制字符串都是 `s` 的子串，请返回 True ，否则请返回 False 。

**示例 1：**

```go
输入：s = "00110110", k = 2
输出：true
解释：长度为 2 的二进制串包括 "00"，"01"，"10" 和 "11"。它们分别是 s 中下标为 0，1，3，2 开始的长度为 2 的子串。
```

**示例 2：**

```go
输入：s = "00110", k = 2
输出：true
```

**示例 3：**

```go
输入：s = "0110", k = 1
输出：true
解释：长度为 1 的二进制串包括 "0" 和 "1"，显然它们都是 s 的子串。
```

**示例 4：**

```go
输入：s = "0110", k = 2
输出：false
解释：长度为 2 的二进制串 "00" 没有出现在 s 中。
```

**示例 5：**

```go
输入：s = "0000000001011100", k = 4
输出：false
```

**提示：**

*   `1 <= s.length <= 5 * 10^5`
*   `s` 中只含 0 和 1 。
*   `1 <= k <= 20`


**解法一**

某次周赛的T2还是T3，忘了，我用了最暴力的方法，直接回溯生成了所有的二进制串，然后对比的，写的很快，也AC了，但是但是后面一直没时间重写，今天偶然发现了这道题，重写下

经典for-while结构
```golang
func hasAllCodes(s string, k int) bool {
    var set = make(map[int]bool)
    var left = 0
    var cur = 0
    for right := 0; right < len(s); right++{
        cur = cur * 2 + int(s[right] & 1)
        for right - left + 1 > k{
            cur &= ^(1 << k) //将首位置为0
            left++
        }
        if right - left + 1 == k{
            set[cur] = true   
        }
    }
    return len(set) == 1 << k
}
```
这个题也可以直接存字符串进去，但是存字符串的时间复杂度就不是O(N)了(N为字符长度)，而是O(KN)，因为字符串Hash的复杂度是O(K)，但是这里K很小，所以其实也无所谓，但是我们还是要追求更加优秀的解法，所以最好的做法还是将其转换成数字，然后存到哈希表中，这里看了别人的解法又学到了一手位运算的小技巧，`cur & ^(1<<k)`（k为cur长度-1）可以将cur首位置为0，也就是消去首位，原理也很简单，就不赘述了

## [1498. 满足条件的子序列数目](https://leetcode-cn.com/problems/number-of-subsequences-that-satisfy-the-given-sum-condition/)

Difficulty: **中等**


给你一个整数数组 `nums` 和一个整数 `target` 。

请你统计并返回 `nums` 中能满足其最小元素与最大元素的 **和** 小于或等于 `target` 的 **非空** 子序列的数目。

由于答案可能很大，请将结果对 10^9 + 7 取余后返回。

**示例 1：**

```go
输入：nums = [3,5,6,7], target = 9
输出：4
解释：有 4 个子序列满足该条件。
[3] -> 最小元素 + 最大元素 <= target (3 + 3 <= 9)
[3,5] -> (3 + 5 <= 9)
[3,5,6] -> (3 + 6 <= 9)
[3,6] -> (3 + 6 <= 9)
```

**示例 2：**

```go
输入：nums = [3,3,6,8], target = 10
输出：6
解释：有 6 个子序列满足该条件。（nums 中可以有重复数字）
[3] , [3] , [3,3], [3,6] , [3,6] , [3,3,6]
```

**示例 3：**

```go
输入：nums = [2,3,3,4,6,7], target = 12
输出：61
解释：共有 63 个非空子序列，其中 2 个不满足条件（[6,7], [7]）
有效序列总数为（63 - 2 = 61）
```

**示例 4：**

```go
输入：nums = [5,2,4,1,7,6,8], target = 16
输出：127
解释：所有非空子序列都满足条件 (2^7 - 1) = 127
```

**提示：**

*   `1 <= nums.length <= 10^5`
*   `1 <= nums[i] <= 10^6`
*   `1 <= target <= 10^6`

**解法一**

双指针滑窗，很关键的一步就是排序，因为我们只关心最大最小值，而且是子序列，与顺序无关
```java
//双指针
public int numSubseq(int[] nums, int target) {
    Arrays.sort(nums);
    int MOD = (int)(1e9+7);
    int n = nums.length;
    //预处理出幂值表
    int[] pow = new int[n];
    pow[0] = 1;
    for (int i = 1; i < n; i++){
        pow[i] = (pow[i-1] << 1) % MOD;
    }
    int left = 0, right = n-1;
    long count = 0;
    while(left <= right){
        while(left <= right && nums[left] + nums[right] > target) {
            right--;
        }
        if (left <= right) {
            //nums[left] + nums[right] <>= target 
            //包含left的子序列个数: left固定，在[left+1,right]选若干个，就有 2^(right-left) 种选法
            count = (count + pow[right-left]) % MOD ;
        }
        left++;
    }
    return (int)count%MOD;
}
```
**解法二**

二分
```java
//二分
public int numSubseq(int[] nums, int target) {
    Arrays.sort(nums);
    int MOD = (int)(1e9+7);
    int n = nums.length;
    //预处理出幂值表
    int[] pow = new int[n];
    pow[0] = 1;
    for (int i = 1; i < n; i++){
        pow[i] = (pow[i-1] << 1) % MOD;
    }
    long count = 0;
    for (int i = 0; i < n; i++) {
        if (target-nums[i] < 0){
            break;
        }
        int right = search(nums, target-nums[i]);
        if (right >= i){
            count = (count + pow[right-i]) % MOD;
        }
    }
    return (int) count % MOD;
}

//搜索最后一个小于等于target的值
public int search(int[] nums, int target){
    int left = 0, right = nums.length-1;
    int res = -1;
    while (left <= right) {
        int mid = left + (right-left)/2;
        if (nums[mid] <= target){
            res = mid;
            left = mid + 1;
        }else{
            right = mid - 1;
        }
    }
    return res;
}
```
## [904. 水果成篮](https://leetcode-cn.com/problems/fruit-into-baskets/)

Difficulty: **中等**


在一排树中，第 `i` 棵树产生 `tree[i]` 型的水果。  
你可以**从你选择的任何树开始**，然后重复执行以下步骤：

1.  把这棵树上的水果放进你的篮子里。如果你做不到，就停下来。
2.  移动到当前树右侧的下一棵树。如果右边没有树，就停下来。

请注意，在选择一颗树后，你没有任何选择：你必须执行步骤 1，然后执行步骤 2，然后返回步骤 1，然后执行步骤 2，依此类推，直至停止。

你有两个篮子，每个篮子可以携带任何数量的水果，但你希望每个篮子只携带一种类型的水果。  
用这个程序你能收集的水果总量是多少？
> 感觉这里想表达的应该是水果树的数量
**示例 1：**

```go
输入：[1,2,1]
输出：3
解释：我们可以收集 [1,2,1]。
```

**示例 2：**

```go
输入：[0,1,2,2]
输出：3
解释：我们可以收集 [1,2,2].
如果我们从第一棵树开始，我们将只能收集到 [0, 1]。
```

**示例 3：**

```go
输入：[1,2,3,2,2]
输出：4
解释：我们可以收集 [2,3,2,2].
如果我们从第一棵树开始，我们将只能收集到 [1, 2]。
```

**示例 4：**

```go
输入：[3,3,3,1,2,1,1,2,3,3,4]
输出：5
解释：我们可以收集 [1,2,1,1,2].
如果我们从第一棵树或第八棵树开始，我们将只能收集到 4 个水果。
```

**提示：**

1.  `1 <= tree.length <= 40000`
2.  `0 <= tree[i] < tree.length`


**解法一**

题目意思其实就是求只包含2个元素的最长子串，题目表述的不太清楚，已经反馈了
```golang
func totalFruit(tree []int) int {
    var left = 0
    var Max = func (a, b int) int {if a>b {return a};return b}
    var n = len(tree)
    var freq [40001]int
    var res, count = 0, 0
    for right := 0; right < n; right++ {
        if freq[tree[right]] == 0 {
            count++
        }
        freq[tree[right]]++
        for count > 2 {
            freq[tree[left]]--
            if freq[tree[left]] == 0 {
                count--
            }
            left++
        }
        res = Max(res, right-left+1)
    }
    return res
}
```
## [NC562.牛牛的魔法卡](https://www.nowcoder.com/practice/9b6fe52a68904c77aa81502f57ceac86)

牛牛从小就有收集魔法卡的习惯，他最大的愿望就是能够集齐 k 种不同种类的魔法卡，现在有 n 张魔法卡，这 n 张魔法卡存在于一维坐标点上，
每张魔法卡可能属于某一种类。牛牛如果想收集魔法卡就需要从当前坐标点跳跃到另外一个魔法卡所在的坐标点，花费的代价是两个跳跃坐标点之间的距离差。
牛牛可以从任意的坐标点出发，牛牛想知道他集齐 k 种魔法卡所花费的最小代价是多少，如果集不齐 k 种魔法卡，输出-1。
第一行输入两个整数 n,k, 分别表示魔法卡的个数和种类个数。
接下来有n行，每行两个数x，y 分别表示属于哪一种魔法卡和魔法卡所在的坐标

**示例1**
```go
输入: 7,3,[[0,1],[0,2],[1,5],[1,1],[0,7],[2,8],[1,3]]
输出: 3
说明: 
样例一：牛牛从坐标点5出发，经过7、8两个点就收集了3张不同种类的魔法卡，达成成就。所需代价 （7-5）+（8-7） = 3
```
**备注:**
- 1<=n<=10^6
- 1<=k<=50 0<=x<k
- 0 <= y <= 1e9

**解法一**

tag是二分，但是想了一会儿感觉好像没啥好的二分的思路，二分答案貌似可行，不过这题滑窗的思路更简单，类似[76-最小覆盖子串](#76-最小覆盖子串)滑就完事儿了
```java
public int solve (int n, int k, int[][] card) {
    // write code here
    Arrays.sort(card, (c1,c2)->c1[1]-c2[1]);
    int INF = Integer.MAX_VALUE;
    int left = 0;
    int count = 0;
    int[] freq = new int[k+1];
    int res = INF;
    for (int right = 0; right < n; right++) {
        if (freq[card[right][0]] == 0) {
            count++;
        }
        freq[card[right][0]]++;
        while(left<=right && count == k){
            res = Math.min(res, card[right][1] - card[left][1]);
            freq[card[left][0]]--;
            if (freq[card[left][0]]==0) {
                count--;
            }
            left++;
        }
    }
    if (res == INF) {
        return -1;
    }
    return res;
}
```
## [1870. 全零子串的数量（LintCode）](https://www.lintcode.com/problem/number-of-substrings-with-all-zeroes/description)

给出一个只包含0或1的字符串str,请返回这个字符串中全为0的子字符串的个数 1<=|str|<=30000

**例1:**
```go
输入:"00010011"
输出:9
解释:
"0"子字符串有5个,
"00"子字符串有3个,
"000"子字符串有1个。
所以返回9
```
**例2:**
```go
输入:"010010"
输出:5
```
**解法一**

直接滑就行了，统计所有0区间的长度，注意组合数的计算就行了
```java
// 1 1 1 1 1 (5+4+3+2+1) = n(n-1)/2 + n or n(n+1)/2
public int stringCount(String str) {
    // Write your code here.
    int left = 0, right = 0;
    int res = 0;
    while (right < str.length()) {
        while(right < str.length() && str.charAt(right) == '0'){
            right++;
        }
        // (0  0  0) 1 1
        //  l  n     r
        int n = right-left;
        //C(n+1,2)/2
        res += n*(n+1)/2;
        right++;
        left = right;
    }
    return res;
}
```

## [1529. 绝对差不超过限制的三元子数组（LintCode](https://www.lintcode.com/problem/triplet-subarray-with-absolute-diff-less-than-or-equal-to-limit/description)）

给定一个递增的整数数组nums，和一个表示限制的整数limit，请你返回满足条件的三元子数组的个数，使得该子数组中的任意两个元素之间的绝对差小于或者等于limit。

如果不存在满足条件的子数组，则返回 0 。

**数据范围：** 1 ≤ len(nums) ≤ 1e4，1 ≤ limit ≤ 1e6，0 ≤ nums[i] ≤ 1e6
由于答案可能很大，请返回它对99997867取余后的结果。

**样例 1:**
```go
输入：[1, 2, 3, 4], 3
输出：4
解释：可选方案有(1, 2, 3), (1, 2, 4), (1, 3, 4), (2, 3, 4)。因此，满足条件的三元组有4个。
```

**样例 2:**
```go
输入：[1, 10, 20, 30, 50], 19
输出：1
解释：唯一可行的三元组是(1, 10, 20)，所以答案为1。
```
**挑战**
你可以只用O(n)的时间复杂度解决这个问题吗？

**解法一**

我一开始看见是Hard想的挺复杂的，什么单调栈都搞出来了，但是仔细看题会发现题目给的数组是有序的，所以直接滑窗然后统计就行了，这里需要注意计算的方式，避免算重
```java
//LintCode上居然是Hard，感觉不是很难
public int tripletSubarray(int[] nums, int limit) {
    // write your code here
    int n = nums.length;
    int left = 0, right = 0;
    int res = 0;
    while (left <= right) {
        //找到最远的合法right
        while (right < n && nums[right]-nums[left] <= limit) {
            right++;
        }
        //  1  (2 3 4)  5
        //left   len  right
        int len = right-left-1;
        left++;
        if (len < 2) continue;
        //C(len,2) 求以left开头，包含left的所有3元组，这样不会重复
        res += len*(len-1)/2;
    }
    return res;
}
```
> 感觉自己静下心来想的话很多题目还是可以自己做出来的，但是就是想的可能有点慢，特别是竞赛中，规定了时间后一慌就更慢了。。看来还是练少了

## [1375. 至少K个不同字符的子串（LintCode）](https://www.lintcode.com/problem/substring-with-at-least-k-distinct-characters/description)

给定一个仅包含小写字母的字符串 S.

返回 S 中至少包含 k 个不同字符的子串的数量.

- 10 ≤ length(S) ≤ 1,000,000
- 1 ≤ k ≤ 26

**样例 1:**
```go
输入: S = "abcabcabca", k = 4
输出: 0
解释: 字符串中一共就只有 3 个不同的字符.
```
**样例 2:**
```go
输入: S = "abcabcabcabc", k = 3
输出: 55
解释: 任意长度不小于 3 的子串都含有 a, b, c 这三个字符.
    比如,长度为 3 的子串共有 10 个, "abc", "bca", "cab" ... "abc"
    长度为 4 的子串共有 9 个, "abca", "bcab", "cabc" ... "cabc"
    ...
    长度为 12 的子串有 1 个, 就是 S 本身.
    所以答案是 1 + 2 + ... + 10 = 55.
```

**解法一**

经典滑窗，非常套路，想好怎么统计就行了
```java
public long kDistinctCharacters(String s, int k) {
    int n = s.length();
    int left = 0;
    int[] freq = new int[128];
    int count = 0;
    long res = 0;
    for (int right = 0; right < n; right++) {
        char cr = s.charAt(right);
        if (freq[cr] == 0) {
            count++;
        }
        freq[cr]++;
        while (count >= k && left <= right) {
            // abc | abcabcabc
            // l r(2)          n(12)
            //统计以s[left,right]开头的所有子串
            //10+9+8+7+...+1
            res += n-right;
            char cl = s.charAt(left);
            freq[cl]--;
            if (freq[cl] <= 0) {
                count--;
            }
            left++;
        }
    }
    return res;
}
```

## [386. 最多有k个不同字符的最长子字符串(LintCode)](https://www.lintcode.com/problem/longest-substring-with-at-most-k-distinct-characters/description)

给定字符串S，找到最多有k个不同字符的最长子串T。

**样例 1:**
```go
输入: S = "eceba" 并且 k = 3
输出: 4
解释: T = "eceb"
```
**样例 2:**
```go
输入: S = "WORLD" 并且 k = 4
输出: 4
解释: T = "WORL" 或 "ORLD"
```
**挑战**： O(n) 时间复杂度

**解法一**

无脑滑窗就行了，太套路了
```java
public int lengthOfLongestSubstringKDistinct(String s, int k) {
    // write your code here
    int n = s.length();
    int left = 0;
    int res = 0;
    int[] freq = new int[128];
    int count = 0;
    for (int right = 0; right < n; right++) {
        char cr = s.charAt(right);
        if (freq[cr] == 0) {
            count++;
        }
        freq[cr]++;
        while (left <= right && count > k) {
            char cl = s.charAt(left);
            freq[cl]--;
            if (freq[cl] <= 0) {
                count--;
            }
            left++;
        }
        res = Math.max(res, right-left+1);
    }
    return res;
}
```
## [1675. 数组的最小偏移量](https://leetcode-cn.com/problems/minimize-deviation-in-array/)

Difficulty: **困难**


给你一个由 `n` 个正整数组成的数组 `nums` 。

你可以对数组的任意元素执行任意次数的两类操作：

*   如果元素是偶数 ，除以 `2`
    *   例如，如果数组是 `[1,2,3,4]` ，那么你可以对最后一个元素执行此操作，使其变成 `[1,2,3,2]`
*   如果元素是奇数 ，乘上 `2`
    *   例如，如果数组是 `[1,2,3,4]` ，那么你可以对第一个元素执行此操作，使其变成 `[2,2,3,4]`

数组的 **偏移量** 是数组中任意两个元素之间的 **最大差值** 。

返回数组在执行某些操作之后可以拥有的 **最小偏移量** 。

**示例 1：**

```c
输入：nums = [1,2,3,4]
输出：1
解释：你可以将数组转换为 [1,2,3,2]，然后转换成 [2,2,3,2]，偏移量是 3 - 2 = 1
```

**示例 2：**

```c
输入：nums = [4,1,5,20,3]
输出：3
解释：两次操作后，你可以将数组转换为 [4,2,5,5,3]，偏移量是 5 - 2 = 3
```

**示例 3：**

```c
输入：nums = [2,10,8]
输出：3
```

**提示：**

*   n == nums.length
*   2 <= n <= 10<sup><span style="display: inline;">5</span></sup>
*   1 <= nums[i] <= 10<sup>9</sup>

**解法一**

和上面[632-最小区间](#632-最小区间)一样，将数据变成和最小区间一样的形式，然后直接套用
```java
public int minimumDeviation(int[] nums) {
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b)->a[2]-b[2]);
    List<List<Integer>> lis = new ArrayList<>();
    int max = 0;
    for (int i = 0; i < nums.length; i++) {
        ArrayList<Integer> tmp = new ArrayList<>();
        if (nums[i] % 2 == 1) {
            pq.add(new int[]{i, 0, nums[i]});
            max = Math.max(max, nums[i]);
            tmp.add(nums[i]);
            tmp.add(nums[i] * 2);
        } else {
            tmp.add(nums[i]);
            while (nums[i] % 2 == 0) {
                tmp.add(nums[i]/2);
                nums[i]/=2;
            }
            pq.add(new int[]{i, 0, nums[i]});
            max = Math.max(max, nums[i]);
            Collections.reverse(tmp);
        }
        lis.add(tmp);
    }
    int res = Integer.MAX_VALUE;
    while (true) {
        int[] min = pq.poll();
        res = Math.min(res, max-min[2]);
        if (min[1]+1 >= lis.get(min[0]).size()) {
            break;
        }
        int next = lis.get(min[0]).get(min[1]+1);
        pq.add(new int[]{min[0], min[1]+1, next});
        max = Math.max(max, next);
    }
    return res;
}
```

**解法二**

```java
    public int minimumDeviation2(int[] nums) {
        int INF = 0x3f3f3f3f;
        PriorityQueue<Integer> pq = new PriorityQueue<>((a, b)->b-a);
        int min = INF;
        for (int i = 0; i < nums.length; i++) {
            if ((nums[i] & 1) == 1) {
                nums[i] <<= 1;
            }
            min = Math.min(min, nums[i]);
            pq.add(nums[i]);
        }
        int res = INF;
        while (true) {
            int max = pq.poll();
            res = Math.min(res, max-min);
            if ((max&1)==1) {
                break;
            }
            pq.add(max/2);
            min = Math.min(min, max/2);
        }
        return res;
    }
```