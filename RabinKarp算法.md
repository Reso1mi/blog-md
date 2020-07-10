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

下面的应该就没什么问题了，链地址法，时间复杂度会增大，耗时增加到了500ms+，但是确保了100%的正确率，这是值得的，这次把mod调小也不会出错了（但是可能会TLE，冲突变大了），总体来说，这几番折腾收获还是挺大的，好事还是要多磨啊
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

## [面试题 17.13. 恢复空格](https://leetcode-cn.com/problems/re-space-lcci/)

Difficulty: **中等**

哦，不！你不小心把一个长篇文章中的空格、标点都删掉了，并且大写也弄成了小写。像句子`"I reset the computer. It still didn’t boot!"`已经变成了`"iresetthecomputeritstilldidntboot"`。在处理标点符号和大小写之前，你得先把它断成词语。当然了，你有一本厚厚的词典`dictionary`，不过，有些词没在词典里。假设文章用`sentence`表示，设计一个算法，把文章断开，要求未识别的字符最少，返回未识别的字符数。

**注意:** 本题相对原题稍作改动，只需返回未识别的字符数

**示例：**

```go
输入：
dictionary = ["looked","just","like","her","brother"]
sentence = "jesslookedjustliketimherbrother"
输出： 7
解释： 断句后为"jess looked just like tim her brother"，共7个未识别字符。
```

**提示：**

*   `0 <= len(sentence) <= 1000`
*   `dictionary`中总字符数不超过 150000。
*   你可以认为`dictionary`和`sentence`中只包含小写字母。


**解法一**

动态规划和Trie的解法左转[动态规划](http://imlgw.top/2019/09/01/leetcode-dong-tai-gui-hua/)专题，这里记录下字符串Hash的做法，其实字符串Hash的做法相比字典树的做法会慢一点点，不过思路还是很值得学习的

这里看官方题解又学到了一点东西，这里在计算hash的时候加了一个1，这样我猜测就是为了避免0的出现，使得后面的base失效，使得冲突的概率变大，比如`aba`和`ba`可能就会被判成一样的字符，我下面的做法没有做减`a`的操作，而是取了更大的BASE，这里就不写冲突检测了，可以直接莽过，上面的第一题确实太离谱了，不按照官方题解的数据来就过不了
```java
public int respace(String[] dictionary, String s) {
    int BASE = 131;
    long MOD = Integer.MAX_VALUE;
    HashSet<Long> set = new HashSet<>();
    for (String word : dictionary ) {
        set.add(hash(word, BASE, MOD));
    }
    int n = s.length();
    int[] dp = new int[n+1];
    for (int i = 1; i <=n ; i++) {
        dp[i] = dp[i-1] + 1;
        long rollhash = 0;
        for (int j = i; j >= 1; j--) {
            rollhash = (rollhash * BASE + s.charAt(j-1)) % MOD;
            if(set.contains(rollhash)){
                //注意这里是dp[j-1]，对应s.charAt(j-1)的前一个字符
                dp[i] = Math.min(dp[i], dp[j-1]);
            }
            if(dp[i] == 0){
                break;
            }
        }
    }
    return dp[n];
}

//注意需要逆向hash，上面计算的时候是j--，是逆向的
public long hash(String s, int BASE, long MOD){
    long h = 0;
    for (int i = s.length()-1; i >=0 ; i--) {
        h = (h * BASE + s.charAt(i)) % MOD;
    }
    return h;
}
```
> 这个解法也用到了Hash表，但是相比用Hash表存字符，存数字的时间复杂度会低很多，其实字符串Hash也就是为了避免在Hash表中存大量的字符，一来空间占用会非常大，二来对于字符串来说计算字符的`hashCode()`的时间复杂度也是O(N)不可忽略的，而数字长度固定，`hashCode()`直接返回值就行了

## [1316. 不同的循环子字符串](https://leetcode-cn.com/problems/distinct-echo-substrings/)

Difficulty: **困难**


给你一个字符串 `text` ，请你返回满足下述条件的 **不同** 非空子字符串的数目：

*   可以写成某个字符串与其自身相连接的形式（即，可以写为 `a + a`，其中 `a` 是某个字符串）。

例如，`abcabc` 就是 `abc` 和它自身连接形成的。

**示例 1：**

```go
输入：text = "abcabcabc"
输出：3
解释：3 个子字符串分别为 "abcabc"，"bcabca" 和 "cabcab" 。
```

**示例 2：**

```go
输入：text = "leetcodeleetcode"
输出：2
解释：2 个子字符串为 "ee" 和 "leetcodeleetcode" 。
```

**提示：**

*   `1 <= text.length <= 2000`
*   `text` 只包含小写英文字母。


**解法一**

哭了，看着KMP的tag进来的，结果发现KMP的不太会写，学习了题解的前缀和Hash的思路，还是有点收获
```java
int BASE = 131;

long MOD = (long)1e9+7;

public int distinctEchoSubstrings(String s) {
    int n = s.length();
    //前缀hash和(前i个元素的hash值)
    long[] hashSum = new long[n+1];
    //BASE的所有幂乘
    long[] pow = new long[n+1];
    pow[0] = 1;
    hashSum[0] = 0;
    for (int i = 1; i <= n; i++) {
        hashSum[i] = (hashSum[i-1]*BASE + s.charAt(i-1)) % MOD;
        pow[i] = (pow[i-1]*BASE) % MOD;
    }
    HashSet<Long> set = new HashSet<>();
    //枚举所有偶数长度的子串
    for (int len = 2; len <= n; len+=2) {
        for (int i = 0 ; i+len-1 < n; i++) {
            int j = i + len - 1; //右边界
            int mid = i + (j-i)/2; //中点
            //0,3
            long hleft = getHash(hashSum, pow, s, i, mid);
            long hright = getHash(hashSum, pow, s, mid+1, j);
            if(hleft == hright && !set.contains(hleft)){
                set.add(hleft);
            }
        }
    }
    return set.size();
}

// 求s[i,j]区间的哈希值: s[i]*B^j-i + s[i+1]*B^j-i-1 + ... + s[j]
// 
// hashSum[i] = s[0]*B^i-1 + s[1]*B^i-2 +...+ s[i-1]
// hashSum[j+1] = s[0]*B^j + s[1]*B^j-1 +...+ s[j]
// hashSum[i]*B^j-i+1 = s[0]*B^j + s[1]*B^j-1 +...+ s[i-1]*B^j-i+1
// hash[i,j] = hashSum[j+1] - hashSum[i] * B^j-i+1
//           = s[i]*B^j-i + s[i+1]*B^j-i-1 +...+s[j]
public long getHash(long[] hashSum, long[] pow, String s, int i, int j){
    //j-i+1是[i,j]区间的长度，包含i和j，而hashSum[k]是不包含k的
    //所以这里需要转换下，j需要+1使得hashSum包含j
    return (hashSum[j+1] - (hashSum[i] * pow[j-i+1]) % MOD + MOD) % MOD;
}
```

**解法二**

KMP的做法肯定就是参考KMP的[459. 重复的子字符串](http://imlgw.top/2020/05/13/kmp-suan-fa/#459-%E9%87%8D%E5%A4%8D%E7%9A%84%E5%AD%90%E5%AD%97%E7%AC%A6%E4%B8%B2)的做法，我看了外网的discuss，只看见了一个这样写的，而且看不太懂，我自己尝试了下，感觉好多地方都会有坑，主要就是去重很麻烦，代码如下

下面为错误解法，无法通过OJ，懒得改了，感觉不是个很好的做法
```java
//KMP的做法
public int distinctEchoSubstrings(String s) {
    for (int i = 0; i < s.length(); i++) {
        getNext(s.substring(i));
    }
    return set.size();
}

HashSet<String> set = new HashSet<>();

public void getNext(String s){
    if(s.length() < 2){
        return;
    }
    int n = s.length();
    int[] next = new int[n+1];
    next[0] = -1;
    next[1] = 0;
    int left = 0, right = 2;
    while(right <= n){
        if(s.charAt(left) == s.charAt(right-1)){
            left++;
            next[right] = left;
            int replen = right-next[right];
            String rs = s.substring(0,replen);
            if (right%2==0 && replen!=right && right%replen==0 && !set.contains(rs)) {
                set.add(rs);
            }
            right++;
        }else if(next[left] == -1){
            right++;
        }else{
            left = next[left];
        }
    }
}
```