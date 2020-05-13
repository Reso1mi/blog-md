---
title: 
 KMP算法
tags: 
  [数据结构,算法]
categories:
	[算法]
---

## 前言

之前大一的时候写过一篇关于KMP的博客，写的也是乱七八糟的，自己看着都费劲，最近看左神的书《程序员代码面试指南》有讲，重新学了一遍，感觉写的还是挺好的，这篇就不从0开始讲解`KMP`了，简单说一些要点，方便以后回顾

> 我记得大一为了搞懂kmp好像花了挺长时间的，网上翻各种博客，搞了大半天吧，结果还是没咋搞清楚，前几天看书大概只花了一个小时左右就都搞清楚了，可能是左神写的比较好吧😂

## next数组是什么？

`kmp`最关键的地方就是这个`next`数组了，`next`数组就是各个子串的前缀后缀最大匹配（相等）长度，具体一点就是`next[i]`代表的就是子串`i`位置前字符（不包括`i`位置）的前后缀最大匹配长度

举个例子： 子串`abababc`  

next[3] = "aba" 的前后缀最大匹配长度 = 1 (开头的a和结尾的a匹配)

next[6]= "ababab" 的前后缀最大匹配长度 = 4 （开头的abab和结尾的abab匹配）

## next数组的作用？

他的作用其实就是在两个字符串比较失配的时候避免目标串回溯，常规的暴力匹配在字符失配的时候就会回退到首字符重新匹配，整体时间复杂度就是`O(m*n)`，而next数组就是为了避免回退，简单举一个例子

![mark](http://static.imlgw.top/blog/20200513/ugvgLhmyaeBr.png?imageslim)

用PPT简单的画了个图（PPT真好用），当匹配到**母串和子串**`index=6`的位置时，发现两者的字符不相同，按照暴力匹配，下一步就是母串回溯到`index=1`也就是b字符位置，子串回溯到首字符，重新开始匹配，直到匹配到子串，或者匹配完母串所有子串，但是当我们有了`next`数组，我们的母串就不必再回退了，而子串也不必再回退到首字符了，图中的黄色下划线和绿色下划线代表的就是**子串index=6**位置前的最长前后缀匹配长度，也就是`ababab`的**最长前后缀匹配字符**，当子串index=6位置的字符匹配不上的时候我们我们就可以直接将`index`跳到`next[6]`，也就是将子串索引移动到index=4的位置，就变成下面这样

![mark](http://static.imlgw.top/blog/20200513/6sBvml0hXrr6.png?imageslim)

母串并没有回退，继续匹配`母串index=6`和子串`index=4`位置的元素，然后重复上面的过程

## 为什么子串可以直接滑动next[i]步？

前面我们知道了如何使用`next`数组，但是为什么子串可以一下子从滑动到`next[i]`位置呢？万一中间有能匹配的字符不就滑过了么？

我们假设在母串中间存在某一个位置能匹配出子串，且该位置在**子串最长匹配后缀之前**，也就是说这部分**比当前的最长前后缀匹配长度还要长**对应到下图就是黑色虚线框框出来的部分

![mark](http://static.imlgw.top/blog/20200513/EbC0E1JvbkHh.png?imageslim)

既然能从这个位置匹配出子串，那么说明我的**子串的前缀**和这一部分相等，同时，由于失配前子串和母串是完全匹配的，所以我**子串的后缀**和这一部分肯定也是相等的，诶？我子串前后缀都和这部分相等，那它肯定是我的匹配前后缀啊，但是这部分又比我的最长前后缀长度要长，是不是矛盾了？所以原假设是不成立的，不存在这样的位置！所以子串可以放心的滑到`next[i]`位置

## next数组如何求？

说了这么多next数组，那么next数组究竟怎么求呢？其实整个的求解过程有点像动态规划，有一些细节需要注意下



## 时间复杂度为什么是线性的？

## 例题

### [28. 实现 strStr()](https://leetcode-cn.com/problems/implement-strstr/)

实现 [strStr()](https://baike.baidu.com/item/strstr/811469) 函数。

给定一个 haystack 字符串和一个 needle 字符串，在 haystack 字符串中找出 needle 字符串出现的第一个位置 (从0开始)。如果不存在，则返回  **-1**。

**示例 1:**

```java
输入: haystack = "hello", needle = "ll"
输出: 2
```

**示例 2:**

```java
输入: haystack = "aaaaa", needle = "bba"
输出: -1
```

**说明:**

当 `needle` 是空字符串时，我们应当返回什么值呢？这是一个在面试中很好的问题。

对于本题而言，当 `needle` 是空字符串时我们应当返回 0 。这与C语言的 [strstr()](https://baike.baidu.com/item/strstr/811469) 以及 Java的 [indexOf()](https://docs.oracle.com/javase/7/docs/api/java/lang/String.html#indexOf(java.lang.String)) 定义相符。

**解法一**

标注的是简单，很多人都是直接调的API，但是我感觉没啥意义，所以这题应该直接上kmp

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
            //完全失配sidx需要后移
            sidx++;
        }else{
            tidx=next[tidx];
        }
    }
    return -1;
}

//求t的next
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
        }else if(left<=0){
            next[right++]=0;
        }else{
            left=next[left];
        }
    }
    return next;
}
```
没啥好说的，裸kmp

### [459. 重复的子字符串](https://leetcode-cn.com/problems/repeated-substring-pattern/)

给定一个非空的字符串，判断它是否可以由它的一个子串重复多次构成。给定的字符串只含有小写英文字母，并且长度不超过10000。

**示例 1:**

```java
输入: "abab"

输出: True

解释: 可由子字符串 "ab" 重复两次构成。
```

**示例 2:**

```java
输入: "aba"

输出: False
```

**示例 3:**

```java
输入: "abcabcabcabc"

输出: True

解释: 可由子字符串 "abc" 重复四次构成。 (或者子字符串 "abcabc" 重复两次构成。)
```

**解法一**

这题标注的也是简单，确实暴力的解法不难想到，但是复杂度比较高，所以就直接上kmp

```java
//20ms，做复杂了，构造了一个s+s然后去掉头，再在里面kmp找
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
        }else if(left<=0){
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

其实只需要构造next数组，根据next数组就可以判断是不是重复的，我们看几组数据

```java
   s:  a b c a b c a b c a b c \0
next: -1 0 0 0 1 2 3 4 5 6 7 8  9
   s:  a a b a b d \0
next: -1 0 1 0 1 0  0
   s:  a b c d a b \0
next: -1 0 0 0 0 1 2 
```

相比常规的KMP算法，我们在字符最后最后也加了`next`位，其实是为了区别下面的情况

```
   s:  a b a b \0
next: -1 0 0 1 2
   s:  a b a c \0
next: -1 0 0 1 0
```

然后我们就可以发现，next数组在过了一定的范围后就开始逐渐递增了，而这个递增的拐点就是在第一个循环节结束的时候，至于为什么我就不详细证明了，其实也很好想因为过了循环节，后面的都是和前面重复的，所以每多一个字符`next[i]=next[i-1]+1`，所以我们用字符的长度减去`next[slen]`就可以得到循环结的长度，我们只需要验证这个循环节能否被`s`字符串长度整除就可以了，同时需要防止循环节长度等于字符串长度的情况

```java
public boolean repeatedSubstringPattern(String s) {
    if(s==null || s.length()<=1) return false;
    int[] next=getNext(s);
    int replen=s.length()-next[s.length()];
    return s.length()%replen==0;
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
        }else if(left<=0){
            next[right++]=0;
        }else{
            left=next[left];
        }
    }
    return next;
}
```

实在不行把这个记住就行了，反正我是记住了😂