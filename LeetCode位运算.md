---
title: LeetCode 位运算
tags:
  - LeetCode
  - 位运算
categories:
  - 算法
date: 2020/07/03
abbrlink: a9fb61a5
---

> 从 [数组专题](http://imlgw.top/2019/05/04/leetcode-shu-zu/) 抽离出来的，时间就不做矫正了，我也不知道啥时候开始做的

##  _LeetCode 二进制_

## [136. 只出现一次的数字](https://leetcode-cn.com/problems/single-number/)

给定一个**非空整数**数组，除了某个元素只出现一次以外，**其余每个元素均出现两次**。找出那个只出现了一次的元素。

**说明：**

你的算法应该具有线性时间复杂度。 你可以不使用额外空间来实现吗？

**示例 1:**

```java
输入：[2,2,1]
输出：1
```

**示例 2:**

```java
输入：[4,1,2,1,2]
输出：4
```

**解法一**

很可惜，想到了位运算，但是没试，瞄了一眼评论区，看见异或两个字马上就滚回来写了这个😂

```java
public int singleNumber(int[] nums) {
    for(int i=1;i<nums.length;i++){
        nums[i]^=nums[i-1];
    }
    return nums[nums.length-1];
}
```
## [260. 只出现一次的数字 III](https://leetcode-cn.com/problems/single-number-iii/)

给定一个整数数组 nums，其中恰好有两个元素只出现一次，其余所有元素均出现两次。 找出只出现一次的那两个元素。

**示例 :**

```java
输入：[1,2,1,3,2,5]
输出：[3,5]
```

**注意：**

1. 结果输出的顺序并不重要，对于上面的例子， [5, 3] 也是正确答案。
2. 你的算法应该具有线性时间复杂度。你能否仅使用常数空间复杂度来实现？

**解法一**

根据异或的结果`xor`，讲整个数组划分为两组，分别包含 a，b 这两个唯一的元素

```java
public int[] singleNumber(int[] nums) {
    if(nums==null || nums.length<=0) return new int[0];
    int xor=nums[0];
    for (int i=1;i<nums.length;i++) {
        xor^=nums[i];
    }
    int index=0; //ab 二进制不同的 index
    while((xor&1)==0){
        xor>>>=1;
        index++;
    }
    //a,b 在 index 位置的二进制位不同，异或结果为 1，然后我们就可以根据这个不同点，将整个数组按照这个划分为两部分
    //这样相同的数肯定会被分配到同一组，问题就转换成了 136，这样我们再分别异或就能得到最终的 a,b
    int a=0,b=0;
    for (int i=0;i<nums.length;i++) {
        if(((nums[i]>>>index)&1)==1){ //根据 index 位置的元素 0，1 来划分为两个数组
            a^=nums[i];
        }else{
            b^=nums[i];
        }
    }
    return new int[]{a,b};
}
```
> 为啥没有 只出现一次的数字Ⅱ？ 别问，问就是不会🤣
>

## [268. 缺失数字](https://leetcode-cn.com/problems/missing-number/)

给定一个包含 0, 1, 2, ..., n 中 n 个数的序列，找出 0 .. n 中没有出现在序列中的那个数。

**示例 1:**

```java
输入：[3,0,1]
输出：2
```

**示例 2:**

```java
输入：[9,6,4,2,3,5,7,0,1]
输出：8
```

**说明：**
你的算法应具有线性时间复杂度。你能否仅使用额外常数空间来实现？

**解法一**

比较直接的思路，求全序列的和然后相减就 ok

```java
public int missingNumber(int[] nums) {
    int N=nums.length;
    int sum=0;
    for (int i=0;i<N;i++) {
        sum+=nums[i];
    }
    return N*(N+1)/2-sum; //可能会溢出
}
```
**解法二**

位运算，异或，和上面一样的思路

```java
public int missingNumber(int[] nums) {
    int res=0;
    //3 0 1 
    for (int i=0;i<nums.length;i++) {
        res^=nums[i];
        res^=i;
    }
    //3^0^1^0^1^2^3=2
    return res^nums.length;
}
```
异或每个数和下标索引，结果就是缺失的数，因为缺失的只有一个，这样就和上面那一题一样了

## [461. 汉明距离](https://leetcode-cn.com/problems/hamming-distance/)

两个整数之间的汉明距离指的是这两个数字对应二进制位不同的位置的数目。

给出两个整数 x 和 y，计算它们之间的汉明距离。

**注意：**
`0 ≤ x, y < 2^31.`

**示例：**

```java
输入：x = 1, y = 4

输出：2

解释：
1   (0 0 0 1)
4   (0 1 0 0)
       ↑   ↑
```

**解法一**

```java
public int hammingDistance(int x, int y) {
    int i=x^y;
    int count=0;
    while(i!=0){
        if ((i&1)==1) { //括号不能掉
            count++;
        }
        i=i>>1;
    }
    return count;
}
```

一行`Integer.bitCount(x^y)`

## [191. 位 1 的个数](https://leetcode-cn.com/problems/number-of-1-bits/)

编写一个函数，输入是一个无符号整数，返回其二进制表达式中数字位数为 ‘1’ 的个数（也被称为汉明重量）

**示例 1：**

```java
输入：00000000000000000000000000001011
输出：3
解释：输入的二进制串 00000000000000000000000000001011 中，共有三位为 '1'。
```

**示例 2：**

```java
输入：00000000000000000000000010000000
输出：1
解释：输入的二进制串 00000000000000000000000010000000 中，共有一位为 '1'。
```

**示例 3：**

```java
输入：11111111111111111111111111111101
输出：31
解释：输入的二进制串 11111111111111111111111111111101 中，共有 31 位为 '1'。
```

**提示：**

- 请注意，在某些语言（如 Java）中，没有无符号整数类型。在这种情况下，输入和输出都将被指定为有符号整数类型，并且不应影响您的实现，因为无论整数是有符号的还是无符号的，其内部的二进制表示形式都是相同的。
- 在 Java 中，编译器使用二进制补码记法来表示有符号整数。因此，在上面的 示例 3 中，输入表示有符号整数 -3。

**进阶：**
如果多次调用这个函数，你将如何优化你的算法？ 

**解法一**

比较精妙的解法，`n&(n-1)` 就是将二进制最右边的 1 变为 0，这样逐步的&最后数组就会变为 0，我们统计下次数就是 1 的数量

```java
public int hammingWeight(int n) {
    int count=0;
    while(n!=0){
        //while(n>0){
        n&=(n-1);
        count++;
    }
    return count;
}
```

**解法二**

移位操作，相比上面会慢一些，没有那么精妙

```java
public int hammingWeight(int n) {
    int count=0;
    while(n!=0){
        if((n&1)==1)//判断末位是不是 1
            count++;
        n>>>=1; //无符号右移，避免添高位添 1 死循环
    }
    return count;
}
```

> 注意右移的操作，都需要使用无符号右移，不然负数右移高位补 1 后就错了

## [338. 比特位计数](https://leetcode-cn.com/problems/counting-bits/)

给定一个非负整数 num。对于 0 ≤ i ≤ num 范围中的每个数字 i ，计算其二进制数中的 1 的数目并将它们作为数组返回。

**示例 1:**

```java
输入：2
输出：[0,1,1]
```

**示例 2:**

```java
输入：5
输出：[0,1,1,2,1,2]
```

**进阶：**

- 给出时间复杂度为 O(n*sizeof(integer)) 的解答非常容易。但你可以在线性时间 O(n) 内用一趟扫描做到吗？
- 要求算法的空间复杂度为 O(n)。
- 你能进一步完善解法吗？要求在 C++或任何其他语言中不使用任何内置函数（如 C++ 中的 __builtin_popcount）来执行此操作。 

**解法一**

这样的题肯定是直接上进阶的啦，有动态规划的意思

```java
public int[] countBits(int num) {
    int[] res=new int[num+1];
    for (int i=1;i<=num;i++) {
        //如果 i 二进制以 0 结尾，那么 i>>1 的 countBit 和 i 一样，i&1=0（>>1 就是/2）
        //反之，那么 i>>1 的比 i 会少 1 个，i&1=1
        res[i]=res[i>>1]+(i&1); //注意括号
    }
    return res;
}
```

**解法二**

和上面类似，不过手法有点不一样

```java
public int[] countBits(int num) {
    int[] res=new int[num+1];
    for (int i=1;i<=num;i++) {
        //i&(i-1) 会去掉最右边的 1
        res[i]=res[i&(i-1)]+1;
    }
    return res;
}
```

## [231. 2 的幂](https://leetcode-cn.com/problems/power-of-two/)

给定一个整数，编写一个函数来判断它是否是 2 的幂次方。

**示例 1:**

```java
输入：1
输出：true
解释：20 = 1
```

**示例 2:**

```java
输入：16
输出：true
解释：24 = 16
```

**示例 3:**

```java
输入：218
输出：false
```

**解法一**

应该是最快 AC 的题吧 hahaha

```java
public boolean isPowerOfTwo(int n) {
    return n>0 && (n&(n-1))==0;
}
```

## [458. 可怜的小猪](https://leetcode-cn.com/problems/poor-pigs/)

有 1000 只水桶，其中有且只有一桶装的含有毒药，其余装的都是水。它们从外观看起来都一样。如果小猪喝了毒药，它会在 15 分钟内死去。

问题来了，如果需要你在一小时内，弄清楚哪只水桶含有毒药，你最少需要多少只猪？

回答这个问题，并为下列的进阶问题编写一个通用算法。

**进阶：**

假设有 `n` 只水桶，猪饮水中毒后会在 `m` 分钟内死亡，你需要多少猪（`x`）就能在 `p` 分钟内找出 “**有毒**” 水桶？这 `n` 只水桶里有且仅有一只有毒的桶。

**提示：**

1. 可以允许小猪同时饮用任意数量的桶中的水，并且该过程不需要时间。
2. 小猪喝完水后，必须有 *m* 分钟的**冷却时间**。在这段时间里，只允许观察，而不允许继续饮水。
3. 任何给定的桶都可以无限次采样（无限数量的猪）。

**解法一**

[参考题解区](https://leetcode-cn.com/problems/poor-pigs/solution/hua-jie-suan-fa-458-ke-lian-de-xiao-zhu-by-guanpen/) 我就不搬运了，简单来说就是看每个小猪能表示几进制的状态，比如题目中说的是 15 分钟死亡，一个小时时间，那么每只小猪可以吃 4 次药，可以检测出 5 瓶药，所以`x`只猪就能检测`state^x`个桶

```java
public int poorPigs(int buckets, int minutesToDie, int minutesToTest) {
    int state=minutesToTest/minutesToDie+1;
    return (int)Math.ceil(Math.log(buckets)/Math.log(state));
}
```

具体的做法：x 只🐖，每只🐖都和对应。.....

> 两只猪二维的表格可以解决，三只猪三维的坐标可以解决，如果是 4 只猪，5 只猪呢？具体的如何测量，这里我还没有点没想清楚😐

## [371. 两整数之和](https://leetcode-cn.com/problems/sum-of-two-integers/)

**不使用**运算符 `+` 和 `-` ，计算两整数 `a` 、`b` 之和。

**示例 1:**

```java
输入：a = 1, b = 2
输出：3
```

**示例 2:**

```java
输入：a = -2, b = 3
输出：1
```

**解法一**

两数之和 = 两数不进位和+两数进位和 ，一开始也没搞懂这个式子，后来在 10 进制上想了下就明白了

比如 998 + 99 = 987+110 = 97 + 1000 = 1097 ，其实就是把加法拆开来看，把进位的数和对应的数位和分开计算，而进位的数和两数的不进位和都可以通过位运算算出来，进而不使用`+/-`计算两数之和

```go
func getSum(a int, b int) int {
    if b==0{
        return a
    }
    //a^b: 不进位和 a&b<<1: 进位数和（进位是进到前一位，所以需要左移，类比十进制就清楚了）
    return getSum(a^b,(a&b)<<1) 
}
```

## [190. 颠倒二进制位](https://leetcode-cn.com/problems/reverse-bits/)

Difficulty: **简单**

颠倒给定的 32 位无符号整数的二进制位。

**示例 1：**

```
输入：00000010100101000001111010011100
输出：00111001011110000010100101000000
解释：输入的二进制串 00000010100101000001111010011100 表示无符号整数 43261596，
     因此返回 964176192，其二进制表示形式为 00111001011110000010100101000000。
```

**示例 2：**

```
输入：11111111111111111111111111111101
输出：10111111111111111111111111111111
解释：输入的二进制串 11111111111111111111111111111101 表示无符号整数 4294967293，
     因此返回 3221225471 其二进制表示形式为 10111111111111111111111111111111 。
```

**提示：**

*   请注意，在某些语言（如 Java）中，没有无符号整数类型。在这种情况下，输入和输出都将被指定为有符号整数类型，并且不应影响您的实现，因为无论整数是有符号的还是无符号的，其内部的二进制表示形式都是相同的。
*   在 Java 中，编译器使用记法来表示有符号整数。因此，在上面的 **示例 2** 中，输入表示有符号整数 `-3`，输出表示有符号整数 `-1073741825`。

**进阶**:  
如果多次调用这个函数，你将如何优化你的算法？

**解法一**

利用移位运算从后往前计算二进制的值就可以了
```java
public int reverseBits(int n) {
    int res = 0;
    int count = 32;
    while(n != 0){ //Java 中没有无符号数，所以这里不能写大于 0
        //Java 移位运算优先级低于+-
        res = (res << 1) + (n & 1);
        n >>>= 1; //无符号右移，避免高位补 1
        count--;
    }
    while(count > 0){
        res <<= 1;
        count--;
    }
    return res;
}
```
上面的解法还是不够简洁，第二个循环没有必要，不过中间有几个小知识点挺有意思的，回顾了下

**解法二**

简洁的解法
```golang
func reverseBits(num uint32) uint32 {
    var res uint32 = 0
    for i := 32; i > 0; i-- {
        //go 中移位运算符优先级高于于+-
        res = (res << 1) + (num & 1)
        num >>= 1
    }
    return res
}
```
> golang 和 java 的位运算优先级居然不一样，一开始写的 go，该成 java 的时候发现不对，调试了下才发现😂，所以任何时候能加括号的尽量加括号，即使你知道不加也可以，最好还是要加上括号！！！

## [201. 数字范围按位与](https://leetcode-cn.com/problems/bitwise-and-of-numbers-range/)

Difficulty: **中等**

给定范围 [m, n]，其中 0 <= m <= n <= 2147483647，返回此范围内所有数字的按位与（包含 m, n 两端点）。

**示例 1:**

```golang
输入：[5,7]
输出：4
```

**示例 2:**

```golang
输入：[0,1]
输出：0
```

**解法一**
最长公共前缀
```golang
//最长公共前缀
//1011 011 m
//1011 100
//1011 101
//1011 110 n
func rangeBitwiseAnd(m int, n int) int {
    var tlen = 0
    for m != n {
        m >>= 1
        n >>= 1
        tlen++
    }
    return m << tlen
}
```
**解法二**
```golang
func rangeBitwiseAnd(m int, n int) int {
    for n > m {
        n &= (n-1)
    }
    return n
}
```

## [1018. 可被 5 整除的二进制前缀](https://leetcode-cn.com/problems/binary-prefix-divisible-by-5/)

Difficulty: **简单**

给定由若干 `0` 和 `1` 组成的数组 `A`。我们定义 `N_i`：从 `A[0]` 到 `A[i]` 的第 `i` 个子数组被解释为一个二进制数（从最高有效位到最低有效位）。

返回布尔值列表 `answer`，只有当 `N_i` 可以被 `5` 整除时，答案 `answer[i]` 为 `true`，否则为 `false`。

**示例 1：**

```c
输入：[0,1,1]
输出：[true,false,false]
解释：
输入数字为 0, 01, 011；也就是十进制中的 0, 1, 3 。只有第一个数可以被 5 整除，因此 answer[0] 为真。
```

**示例 2：**

```c
输入：[1,1,1]
输出：[false,false,false]
```

**示例 3：**

```c
输入：[0,1,1,1,1,1]
输出：[true,false,false,false,true,false]
```

**示例 4：**

```c
输入：[1,1,1,0,1]
输出：[false,false,false,false,false]
```

**提示：**

1.  1 <= A.length <= 30000
2.  `A[i]` 为 `0` 或 `1`

**解法一**

一开始没考虑溢出的问题，用的是 py 所以也没报错，看了评论区才意识到会有溢出的问题，不过这里也很好处理，因为我们每次都只需要知道这个值模 5 的值就行了，所以我们只需要保留当前值模 5 的余数就行了
$$
(a * b+c) \bmod \ p=((\underbrace{(a \bmod \ p)} * (b \bmod \ p))+(c \bmod p))\bmod p
$$
这里的$a$就是上次计算的结果，根据取模运算的性质，我们只需要保存上次计算的结果模 5 后的值就行了，这样就能避免溢出的问题
```python
​class Solution:
    def prefixesDivBy5(self, A: List[int]) -> List[bool]:
        res = []
        v = 0
        # 1110(14) --> 11101 (29)
        for i in range(0, len(A)):
            v = (v*2 + A[i]) % 5
            res.append(v==0)
        return res
```