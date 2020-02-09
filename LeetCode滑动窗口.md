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
//用boolean数组
public static int lengthOfLongestSubstring8(String s) {
    int length=s.length();
    if(length==0) return 0;
    //ASCII码表
    boolean[] freq=new boolean[256]; 
    freq[s.charAt(0)]=true;
    int head=0,tail=0;
    int max=1;
    while(head<length){
        if(tail+1<length&&!freq[s.charAt(tail+1)]){
            tail++;
            freq[s.charAt(tail)]=true;
        }else{
            freq[s.charAt(head)]=false;
            head++;
        }
        max=max<tail-head+1?tail-head+1:max;
    }
    return max;
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

找到一个板子，统一一下写法

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
15ms，86%时间复杂度`O(M+N)`，但是这很明显并不是最优解，因为这题的窗口长度其实是可以固定的，滑动的可以更快，只用遍历一遍，等以后有时间再研究吧

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

```java
public int balancedString(String s) {
    if (s==null|| s.length()<=0) {
        return 0;
    }
    int len=s.length();
    int balance=len/4;
    //映射字符
    int[] count=new int[128];
    //统计4个字符串出现的次数
    for (int i=0;i<len;i++) {
        count[s.charAt(i)]++;
    }
    int left=0,right=0,res=len;
    while(right<len){
        count[s.charAt(right)]--;
        while(left<len && count['Q']<=balance && count['W']<=balance && count['E'] <=balance && count['R']<=balance){
            res=Math.min(res, right-left+1);
            //窗口左移，缩小窗口
            count[s.charAt(left)]++;
            left++;
        }
        //扩大右边界
        right++;
    }
    return res;
}
```
左指针追赶右指针，形成滑动窗口，感觉滑动窗口的题真的有点不好搞啊！！！