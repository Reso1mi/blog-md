---
title: 
 Rabin-Karp算法
tags: 
  [数据结构,算法]
categories:
	[算法]
---
 > 见到好几次了，感觉不是很难，学一手，本来想详细的写一下完整的Rabin-Karp解析的，但是目前确实时间有点紧，加上自己也没做几题，理解的可能还不到位，等后面有时间再来补吧
 ## [1044. 最长重复子串](https://leetcode-cn.com/problems/longest-duplicate-substring/)

Difficulty: **困难**


给出一个字符串 `S`，考虑其所有**重复子串**（`S` 的连续子串，出现两次或多次，可能会有重叠）。

返回**任何**具有最长可能长度的重复子串。（如果 `S` 不含重复子串，那么答案为 `""`。）

**示例 1：**

```
输入："banana"
输出："ana"
```

**示例 2：**

```
输入："abcd"
输出：""
```

**提示：**

1.  `2 <= S.length <= 10^5`
2.  `S` 由小写英文字母组成。


**解法一**

看题解区学习了一下，整体的思路倒是不难，主要就是一个`rolling hash`的过程，然后就是关于MOD的选取，到现在也不是很清楚怎么选MOD。。。
```java
public String longestDupSubstring(String S) {
    int n = S.length();
    int [] nums =new int[n];
    for (int i = 0; i < S.length(); i++) {
        nums[i] =  S.charAt(i) - 'a';
    }
    long MOD =  1L<<32;
    int B = 26; 
    int left = 1;
    int right = n - 1;
    int start = -1, mlen = 0;
    while (left <= right){
        int mid = left + (right - left)/2;
        int temp = RabinKarp(mid, B, nums, MOD);
        if (temp != -1){
            if (mid > mlen){
                mlen = mid;
                start = temp;
            }
            left = mid + 1;
        }else{
            right = mid - 1;
        }
    }
    return start == -1 ? "" : S.substring(start, start + mlen);
}

public int RabinKarp(int len, int B, int[] nums, long MOD){
    //hash(S) = s[0] * B^(len-1) + s[1] * B^(len-2) + ... + s[n-1] *1
    //B^(len-1) (移除左端点时需要的值)
    long BL = 1;
    for (int i = 0; i < len - 1; i++){
        BL = (BL * B) % MOD;
    }
    //[0,len-1]的hash值
    long h = 0;
    for (int i = 0; i < len; i++){
        h = (h * B + nums[i]) % MOD;
    }
    HashSet<Long> set = new HashSet<>();
    set.add(h);
    //rolling hash
    for (int i = 1; i <= nums.length - len; i++){
        //+MOD是为了负数取模
        h = (h - nums[i - 1] * BL % MOD + MOD) % MOD;
        h = (h * B + nums[i + len - 1]) % MOD;
        if (set.contains(h)){
            return i;
        }
        set.add(h);
    }
    return -1;
}
```

**解法二**

补充一下冲突检测，猜mod也太玄学了😂
> 这个冲突检测的方法有问题，留下做个印证，正确的检测请直接看 解法三

```java
//写一波检测冲突的
public String longestDupSubstring(String S) {
    int n = S.length();
    long MOD = (long) 1e9+7;
    int B = 26; 
    int left = 1;
    int right = n - 1;
    int start = -1, mlen = 0;
    while (left <= right){
        int mid = left + (right - left)/2;
        int temp = RabinKarp(mid, B, S, MOD);
        if (temp != -1){
            if (mid > mlen){
                mlen = mid;
                start = temp;
            }
            left = mid + 1;
        }else{
            right = mid - 1;
        }
    }
    return start == -1 ? "" : S.substring(start, start + mlen);
}

public int RabinKarp(int len, int B, String S, long MOD){
    //hash(S) = s[0] * B^(len-1) + s[1] * B^(len-2) + ... + s[n-1] *1
    //B^(len-1) (移除左端点时需要的值)
    long BL = 1;
    for (int i = 0; i < len - 1; i++){
        BL = (BL * B) % MOD;
    }
    //[0,len-1]的hash值
    long h = 0;
    for (int i = 0; i < len; i++){
        h = (h * B + S.charAt(i) - 'a') % MOD;
    }
    //这里肯定不能直接存字符串做冲突检测，太大了会MLE
    //存一个起始地址就可以了len已知
    HashMap<Long,Integer> map = new HashMap<>();
    map.put(h, 0);
    //rolling hash
    for (int i = 1; i <= S.length() - len; i++){
        //+MOD是为了负数取模
        h = (h - (S.charAt(i-1) - 'a') * BL % MOD + MOD) % MOD;
        h = (h * B + (S.charAt(len + i -1) - 'a')) % MOD;
        Integer start = map.get(h);
        if (start != null && S.substring(start, start + len).equals(S.substring(i, i + len))){
            return i;
        }
        map.put(h, i);
    }
    return -1;
}
```

> 关于这个冲突检测有一个小问题，我尝试减小了`MOD`的大小，比如101，这样计算出来的结果在数据量较大的就不对了，得到的字符虽然确实是重复了，但是并不是最长的，按道理写了冲突检测后即使我MOD取1应该都是可以找出来的啊？======>
> 在写上面这段话的时候突然想明白了，因为`MOD`取的太小，导致冲突的概率大大增加，而这里我做冲突检测的时候只保存了一个值，也就是说会有很多值被舍弃掉，也许你舍弃的值可能恰好就是最后的答案，字符越长，发生这种情况的概率就越高，所以说我这里冲突检测做的并不完全，能过也纯属运气，正确的冲突检测应该保存一个`List`链表，然后在发生冲突的时候在List中找有没有和当前字符相等的，这样一来，时间复杂度就会上去（其实这个过程就是设计Hash表的过程，链地址法解决冲突）

**解法三**

下面的应该就没什么问题了，链地址法，时间复杂度会增大，耗时增加到了500ms+，但是确保了100%的正确率，这是值得的，这次把mod调小也不会出错了，总体来说，这几番折腾收获还是挺大的，好事还是要多磨啊
```java
//再写一波检测冲突。。。
//上面的写的有问题，检测不完全
public String longestDupSubstring(String S) {
    int n = S.length();
    long MOD = (long) 1e9+7;
    int B = 26; 
    int left = 1;
    int right = n - 1;
    int start = -1, mlen = 0;
    while (left <= right){
        int mid = left + (right - left)/2;
        int temp = RabinKarp(mid, B, S, MOD);
        if (temp != -1){
            if (mid > mlen){
                mlen = mid;
                start = temp;
            }
            left = mid + 1;
        }else{
            right = mid - 1;
        }
    }
    return start == -1 ? "" : S.substring(start, start + mlen);
}

public int RabinKarp(int len, int B, String S, long MOD){
    //hash(S) = s[0] * B^(len-1) + s[1] * B^(len-2) + ... + s[n-1] *1
    //B^(len-1) (移除左端点时需要的值)
    long BL = 1;
    for (int i = 0; i < len - 1; i++){
        BL = (BL * B) % MOD;
    }
    //[0,len-1]的hash值
    long h = 0;
    for (int i = 0; i < len; i++){
        h = (h * B + S.charAt(i) - 'a') % MOD;
    }
    //这里肯定不能直接存字符串做冲突检测，太大了会MLE
    //存一个起始地址就可以了len已知
    HashMap<Long,List<Integer>> map = new HashMap<>();
    map.put(h, new ArrayList(){{add(0);}});
    //rolling hash
    for (int i = 1; i <= S.length() - len; i++){
        //+MOD是为了负数取模
        h = (h - (S.charAt(i-1) - 'a') * BL % MOD + MOD) % MOD;
        h = (h * B + (S.charAt(len + i -1) - 'a')) % MOD;
        List<Integer> starts = map.get(h);
        if (check(starts, i, S, len)){
            return i;
        }
        if (starts == null){
            List<Integer> lis = new ArrayList<>();
            lis.add(i);
            map.put(h, lis);
        }else{
            starts.add(i);
        }
    }
    return -1;
}

public boolean check(List<Integer> starts, int i, String s, int len){
    if(starts == null || starts.size() <= 0){
        return false;
    }
    for (int left : starts) {
        if(s.substring(left, left + len).equals(s.substring(i, i + len))){
            return true;
        }
    }
    return false;
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

dp的解法是O(N^2)的，略显暴力，动态规划的解法放在[dp专题](http://imlgw.top/2019/09/01/leetcode-dong-tai-gui-hua/#718-%E6%9C%80%E9%95%BF%E9%87%8D%E5%A4%8D%E5%AD%90%E6%95%B0%E7%BB%84)中，这里要介绍的是二分+字符串Hash的`O(NlogN)`的解法

这里直接莽过了，且效率极高（16ms），就不写冲突检测了
```java
public int findLength(int[] A, int[] B) {
    int lenA = A.length, lenB = B.length;
    int left = 1;
    int right = Math.min(lenA, lenB);
    int res = 0;
    while (left <= right){
        int mid = left +(right - left) / 2;
        if(RabinKarp(A, B, mid)){
            res = Math.max(res, mid);
            left = mid + 1;
        }else{
            right = mid - 1;
        }
    }
    return res;
}

public boolean RabinKarp(int[] A,int[] B, int L){
    int MOD = (int) 1e9+7, BASE = 101;
    long BL = 1;
    for(int i = 0; i < L-1; i++){
        BL = (BL * BASE) % MOD;
    }
    //hash(A[0,L-1]),hash(B[0,L-1])
    long hA = 0, hB = 0;
    for(int i = 0; i < L; i++){
        hA = (hA * BASE + A[i]) % MOD;
        hB = (hB * BASE + B[i]) % MOD;
    }
    HashSet<Long> set =new HashSet<>();
    set.add(hA);
    //rolling hash A
    for(int i = 1; i <= A.length - L; i++){
        hA = (hA - A[i-1] * BL % MOD + MOD) % MOD;
        hA = (hA * BASE + A[L + i -1]) % MOD;
        set.add(hA);
    }
    if(set.contains(hB)) return true;
    //rolling hash B
    for(int i = 1; i <= B.length - L; i++){
        hB = (hB - B[i-1] * BL % MOD + MOD) % MOD;
        hB = (hB * BASE + B[L + i -1]) % MOD;
        //这里还可以做一下冲突检测，set中需要多存一些信息
        if(set.contains(hB)){
            return true;
        }
    }
    return false;
}
```