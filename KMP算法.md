---
title: KMP 算法
tags:
  - 数据结构
  - 算法
categories:
  - 算法
date: 2020/5/13
abbrlink: 2da0528d
---

## 前言

之前大一的时候写过一篇关于 KMP 的博客，写的也是乱七八糟的，自己看着都费劲，最近看左神的书《程序员代码面试指南》有讲，重新学了一遍，感觉写的还是挺好的，这篇就不从 0 开始讲解`KMP`了，简单说一些要点，方便以后回顾

> 我记得大一为了搞懂 kmp 好像花了挺长时间的，网上翻各种博客，搞了大半天吧，结果还是没咋搞清楚，前几天看书大概只花了一个小时左右就都搞清楚了，可能是左神写的比较好吧😂

## next 数组是什么？

`kmp`最关键的地方就是这个`next`数组了，`next`数组就是各个子串的前缀后缀最大匹配（相等）长度，具体一点就是`next[i]`代表的就是子串`i`位置前字符（不包括`i`位置）的前后缀最大匹配长度

举个例子： 子串`abababc`  

next[3] = "aba" 的前后缀最大匹配长度 = 1 （开头的 a 和结尾的 a 匹配）

next[6]= "ababab" 的前后缀最大匹配长度 = 4 （开头的 abab 和结尾的 abab 匹配）

## next 数组的作用？

他的作用其实就是在两个字符串比较失配的时候避免目标串回溯，常规的暴力匹配在字符失配的时候就会回退到首字符重新匹配，整体时间复杂度就是`O(m*n)`，而 next 数组就是为了避免回退，简单举一个例子

![mark](http://static.imlgw.top/blog/20200513/ugvgLhmyaeBr.png?imageslim)

用 PPT 简单的画了个图（PPT 真好用），当匹配到**母串和子串**`index=6`的位置时，发现两者的字符不相同，按照暴力匹配，下一步就是母串回溯到`index=1`也就是 b 字符位置，子串回溯到首字符，重新开始匹配，直到匹配到子串，或者匹配完母串所有子串，但是当我们有了`next`数组，我们的母串就不必再回退了，而子串也不必再回退到首字符了，图中的黄色下划线和绿色下划线代表的就是**子串 index=6 **位置前的最长前后缀匹配长度，也就是`ababab`的**最长前后缀匹配字符**，当子串 index=6 位置的字符匹配不上的时候我们我们就可以直接将`index`跳到`next[6]`，也就是将子串索引移动到 index=4 的位置，就变成下面这样

![mark](http://static.imlgw.top/blog/20200513/6sBvml0hXrr6.png?imageslim)

母串并没有回退，继续匹配`母串 index=6`和子串`index=4`位置的元素，然后重复上面的过程

## 为什么子串可以直接滑动 next[i] 步？

前面我们知道了如何使用`next`数组，但是为什么子串可以一下子从滑动到`next[i]`位置呢？万一中间有能匹配的字符不就滑过了么？

我们假设在母串中间存在某一个位置能匹配出子串，且该位置在**子串最长匹配后缀之前**，也就是说这部分**比当前的最长前后缀匹配长度还要长**对应到下图就是黑色虚线框框出来的部分

![mark](http://static.imlgw.top/blog/20200513/EbC0E1JvbkHh.png?imageslim)

既然能从这个位置匹配出子串，那么说明我的**子串的前缀**和这一部分相等，同时，由于失配前子串和母串是完全匹配的，所以我**子串的后缀**和这一部分肯定也是相等的，诶？我子串前后缀都和这部分相等，那它肯定是我的匹配前后缀啊，但是这部分又比我的最长前后缀长度要长，是不是矛盾了？所以原假设是不成立的，不存在这样的位置！所以子串可以放心的滑到`next[i]`位置

## next 数组如何求？

说了这么多 next 数组，那么 next 数组究竟怎么求呢？其实整个的求解过程有点像动态规划，有一些细节需要注意下

> 想起来了再写

## 时间复杂度为什么是线性的？

> 想起来了再写

## 例题

### [28. 实现 strStr()](https://leetcode-cn.com/problems/implement-strstr/)

实现 [strStr()](https://baike.baidu.com/item/strstr/811469) 函数。

给定一个 haystack 字符串和一个 needle 字符串，在 haystack 字符串中找出 needle 字符串出现的第一个位置 （从 0 开始）。如果不存在，则返回  **-1**。

**示例 1:**

```java
输入：haystack = "hello", needle = "ll"
输出：2
```

**示例 2:**

```java
输入：haystack = "aaaaa", needle = "bba"
输出：-1
```

**说明：**

当 `needle` 是空字符串时，我们应当返回什么值呢？这是一个在面试中很好的问题。

对于本题而言，当 `needle` 是空字符串时我们应当返回 0 。这与 C 语言的 [strstr()](https://baike.baidu.com/item/strstr/811469) 以及 Java 的 [indexOf()](https://docs.oracle.com/javase/7/docs/api/java/lang/String.html#indexOf(java.lang.String)) 定义相符。

**解法一**

标注的是简单，很多人都是直接调的 API，但是我感觉没啥意义，所以这题应该直接上 kmp

```java
public int strStr(String haystack, String needle) {
    if(needle==null || needle.length()<=0) return 0;
    if(haystack==null ||haystack.length()<=0) return -1;
    int[] next=getNext(needle);
    int tidx=0,sidx=0;
    while (sidx<haystack.length() && tidx<needle.length()) {
        if(needle.charAt(tidx) == haystack.charAt(sidx)){
            tidx++;sidx++;
            if(tidx==needle.length()){
                return sidx-tidx;
            }
        }else if(next[tidx]==-1){
            //完全失配 sidx 需要后移
            sidx++;
        }else{
            tidx=next[tidx];
        }
    }
    return -1;
}

//求 t 的 next
//abadabac
//ac
public int[] getNext(String t){
    int[] next= new int[t.length()];
    next[0]=-1;
    if(t.length()<=1) return next;
    next[1]=0;
    int left=0,right=2;
    while(right<t.length()){
        if(t.charAt(left)==t.charAt(right-1)){
            left++;
            next[right++]=left;
        }else if(next[left]==-1){ //无法匹配了
            right++; //next[right]=0
        }else{
            left=next[left];
        }
    }
    return next;
}
```
没啥好说的，裸 kmp

### [459. 重复的子字符串](https://leetcode-cn.com/problems/repeated-substring-pattern/)

给定一个非空的字符串，判断它是否可以由它的一个子串重复多次构成。给定的字符串只含有小写英文字母，并且长度不超过 10000。

**示例 1:**

```java
输入："abab"

输出：True

解释：可由子字符串 "ab" 重复两次构成。
```

**示例 2:**

```java
输入："aba"

输出：False
```

**示例 3:**

```java
输入："abcabcabcabc"

输出：True

解释：可由子字符串 "abc" 重复四次构成。 （或者子字符串 "abcabc" 重复两次构成。)
```

**解法一**

这题标注的也是简单，确实暴力的解法不难想到，但是复杂度比较高，所以就直接上 kmp

```java
//20ms，做复杂了，构造了一个 s+s 然后去掉头，再在里面 kmp 找
public boolean repeatedSubstringPattern(String s) {
    if(s==null || s.length()<=0) return false;
    String t=s+s;
    int[] next=getNext(s);
    int i=0,j=1; //去掉头
    while(i<s.length() && j<t.length()){
        if(s.charAt(i)==t.charAt(j)){
            i++;j++;
        }else if(next[i]==-1){
            j++;
        }else{
            i=next[i];
        }
    }
    return j-i!=s.length();
}

public int[] getNext(String s){
    if(s.length()==1){
        return new int[]{-1};
    }
    int[] next=new int[s.length()];
    next[0]=-1;
    next[1]=0;
    int left=0,right=2; 
    while(right<s.length()){
        if(s.charAt(left)==s.charAt(right-1)){
            next[right++]=++left;
        }else if(next[left]==-1){
            next[right++]=0;
        }else{
            left=next[left];
        }
    }
    return next;
}
```
这个解法其实构造了一个`s+s`的字符`t`，然后去掉头，在`t[1:]`中找原`s`，最后找到的位置只要不是`s+s`连接处，也就是`s.length()`位置，那么就肯定是重复的有循环的，其实这个结论在写的时候并没有证明，完全是猜的😂，简单证明下，一图胜前言

![mark](http://static.imlgw.top/blog/20200513/O4Nmz343e9Li.png?imageslim)

**解法二**

其实只需要构造 next 数组，根据 next 数组就可以判断是不是重复的，我们看几组数据

```java
   s:  a b c a b c a b c a b c \0
next: -1 0 0 0 1 2 3 4 5 6 7 8  9
   s:  a a b a b d \0
next: -1 0 1 0 1 0  0
   s:  a b c d a b \0
next: -1 0 0 0 0 1 2 
```

相比常规的 KMP 算法，我们在字符最后最后也加了`next`位，其实是为了区别下面的情况

```
   s:  a b a b \0
next: -1 0 0 1 2
   s:  a b a c \0
next: -1 0 0 1 0
```

然后我们就可以发现，next 数组在过了一定的范围后就开始逐渐递增了，而这个递增的拐点就是在第一个循环节结束的时候，至于为什么我就不详细证明了，其实也很好想因为过了循环节，后面的都是和前面重复的，所以每多一个字符`next[i]=next[i-1]+1`，所以我们用字符的长度减去`next[slen]`就可以得到循环结的长度，我们只需要验证这个循环节能否被`s`字符串长度整除就可以了，同时需要防止循环节长度等于字符串长度的情况

```java
public boolean repeatedSubstringPattern(String s) {
    if(s==null || s.length()<=1) return false;
    int[] next=getNext(s);
    int replen=s.length()-next[s.length()];
    //循环结长度等于字符长度
    return replen!=s.length() && s.length()%replen==0;
}

public int[] getNext(String s){
    if(s.length()==1){
        return new int[]{-1};
    }
    int[] next=new int[s.length()+1];
    next[0]=-1;
    next[1]=0;
    int left=0,right=2; 
    while(right<=s.length()){
        if(s.charAt(left)==s.charAt(right-1)){
            next[right++]=++left;
        }else if(next[left]==-1){
            next[right++]=0;
        }else{
            left=next[left];
        }
    }
    return next;
}
```

实在不行把这个记住就行了，反正我是记住了😂（过几天就忘了

### [214. 最短回文串](https://leetcode-cn.com/problems/shortest-palindrome/)

给定一个字符串 ***s***，你可以通过在字符串前面添加字符将其转换为回文串。找到并返回可以用这种方式转换的最短回文串。

**示例 1:**

```java
输入："aacecaaa"
输出："aaacecaaa"
```

**示例 2:**

```java
输入："abcd"
输出："dcbabcd"
```

**解法一**

这题很关键的一个点就是最短回文串其实就是原字符`s`，减去`s[0]`开头的最长回文串，剩下的部分再放到`s`前，这就是最短回文串

所以问题就变成了如何求`s[0]`开头的最长回文串，朴素的思路可以使用中心扩散法，枚举所有的字符和间隙，或者使用"马拉车"，等高效算法，这里不多介绍，主要介绍 kmp 的做法

我们把`s`串翻转变成`rs`，然后将两部分拼接起来变为`s+rs`，这个时候我们要求`s[0]`开头的最长回文子串，实际上就变成了求`s+rs`的最长公共前后缀

![mark](http://static.imlgw.top/blog/20200518/CtFIYht31mIL.png?imageslim)

这里还有一点需要注意，就是`s+rs`的中间应该加分隔符，这是为了避免公共前后缀过长，甚至比原字符`s`还要长，这肯定是不对的，就比如`aaaaaaa`这样的 case，加了分割符之后最长的前后缀就不会超过`s`了，

```java
public String shortestPalindrome(String s) {
    String rs=new StringBuilder(s).reverse().toString();
    //#是为了避免前后缀过长超过原字符 s 的长度，比如 aaaaaaa 这种
    String t=s+"#"+rs; 
    int[] next=new int[t.length()+1];
    next[0]=-1;
    next[1]=0;
    int left=0;
    int i=2;
    while(i<=t.length()){
        if(t.charAt(i-1)==t.charAt(left)){
            next[i++]=++left;
        }else if(next[left]==-1){
            next[i++]=0;
        }else{
            left=next[left];
        }
    }
    //System.out.println(next[t.length()]);
    return rs.substring(0,s.length()-next[t.length()])+s;
}
```

### [1392. 最长快乐前缀](https://leetcode-cn.com/problems/longest-happy-prefix/)

「快乐前缀」是在原字符串中既是 非空 前缀也是后缀（不包括原字符串自身）的字符串。

给你一个字符串 s，请你返回它的**最长快乐前缀**。

如果不存在满足题意的前缀，则返回一个空字符串。

**示例 1：**

```java
输入：s = "level"
输出："l"
解释：不包括 s 自己，一共有 4 个前缀（"l", "le", "lev", "leve"）和 4 个后缀（"l", "el", "vel", "evel"）。最长的既是前缀也是后缀的字符串是 "l" 。

```
**示例 2：**

```java
输入：s = "ababab"
输出："abab"
解释："abab" 是最长的既是前缀也是后缀的字符串。题目允许前后缀在原字符串中重叠。
```
**示例 3：**

```java
输入：s = "leetcodeleet"
输出："leet"
```
**示例 4：**

```java
输入：s = "a"
输出：""
```

**提示：**
- 1 <= s.length <= 10^5
- s 只含有小写英文字母

**解法一**

最近某次周赛的 T4，没参加，今天看见群里有人提到了，看了下发现是裸 KMP... 正好复习下，结果写出 bug 了。

```golang
func longestPrefix(s string) string {
    if len(s) == 1 {
        return ""
    }
    //裸 KMP
    next := make([]int, len(s)+1)
    next[0] = -1
    next[1] = 0
    var left = 0
    var i = 2
    for i <= len(s) {
        if s[i-1] == s[left] {
            left++
            next[i] = left
            i++
        } else if next[left] == -1 {
            i++
        } else {
            left = next[left]
        }
    }
    return s[0:next[len(s)]]
}
```
确实也长时间没有复习 kmp 了，kmp 的细节几乎都忘了，上面的都是凭借着一点理解和记忆写的
> 检查了前面 kmp 的写法，稍微改进了一下，目前统一了写法

### [796. 旋转字符串](https://leetcode-cn.com/problems/rotate-string/)

给定两个字符串，`A` 和 `B`。

`A` 的旋转操作就是将 `A` 最左边的字符移动到最右边。 例如，若 `A = 'abcde'`，在移动一次之后结果就是`'bcdea'` 。如果在若干次旋转操作之后，`A` 能变成`B`，那么返回`True`。

```java
示例 1:
输入：A = 'abcde', B = 'cdeab'
输出：true
```
```java
示例 2:
输入：A = 'abcde', B = 'abced'
输出：false
```

**注意：**

*   `A` 和 `B` 长度不超过 `100`。

**解法一**

经典 easy 题当 hard 做，这个题数据量很小，直接暴力就行了，但是我们还是要追求更好的解法

一开始是在一篇文章中看到了这个题，里面说了这个题是 kmp，我看了下没想到什么好的思路，只想到了一个 NlogN 的做法，二分+kmp 找旋转点，然后 kmp 判断旋转点后时候也存在于 A 字符中（类似二分答案）
```golang
//不够聪明的做法：二分+KMP 时间复杂度 O(NlogN)
func rotateString(A string, B string) bool {
    if len(A) != len(B) {
        return false
    }
    if A == B {
        return true
    }
    var left = 0
    var right = len(B) - 1
    var rotate = -1
    //二分找旋转点
    for left <= right {
        mid := left + (right-left)/2
        if kmp(A, B[:mid+1]) != -1 {
            rotate = mid
            left++
        } else {
            right--
        }
    }
    if rotate == -1 {
        return false
    }
    return kmp(A, B[rotate+1:]) != -1
}

func kmp(A string, t string) int {
    var next = getNext(t)
    var Ai = 0
    var ti = 0
    for Ai < len(A) && ti < len(t) {
        if A[Ai] == t[ti] {
            Ai++
            ti++
        } else if next[ti] == -1 {
            Ai++
        } else {
            ti = next[ti]
        }
    }
    if ti == len(t) {
        return Ai - 1
    }
    return -1
}

func getNext(t string) []int {
    if len(t) < 2 {
        return []int{-1}
    }
    var next = make([]int, len(t))
    var left = 0
    next[0] = -1
    next[1] = 0
    var i = 2
    for i < len(t) {
        if t[left] == t[i-1] {
            left++
            next[i] = left
            i++
        } else if next[left] == -1 {
            i++
        } else {
            left = next[left]
        }
    }
    return next
}
```

**解法二**

看了评论区的大佬的做法，实际上`A+A`就包含了所有的旋转`A`的结果子串，`A+A`就相当于首位相连，所以我们可以直接在`A+A`中 kmp 找`B`就可以了，时间复杂度`O(N)`

```golang
//聪明的解法：A+A 包含了所有可能的旋转情况，直接对 A+A 和 B 做 kmp 就行了
//abcdeabcde
func rotateString(A string, B string) bool {
    if len(A) != len(B) {
        return false
    }
    if A == B {
        return true
    }
    return kmp(A+A, B) != -1
}
```