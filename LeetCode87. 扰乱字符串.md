---
title: 
   LeetCode87.扰乱字符串
tags: 
  [LeetCode,动态规划]
categories:
	[算法]
---

### [87\. 扰乱字符串](https://leetcode-cn.com/problems/scramble-string/)

Difficulty: **困难**

使用下面描述的算法可以扰乱字符串 `s` 得到字符串 `t` ：

1.  如果字符串的长度为 1 ，算法停止
2.  如果字符串的长度 > 1 ，执行下述步骤：
    *   在一个随机下标处将字符串分割成两个非空的子字符串。即，如果已知字符串 `s` ，则可以将其分成两个子字符串 `x` 和 `y` ，且满足 `s = x + y` 。
    *   **随机** 决定是要「交换两个子字符串」还是要「保持这两个子字符串的顺序不变」。即，在执行这一步骤之后，`s` 可能是 `s = x + y` 或者 `s = y + x` 。
    *   在 `x` 和 `y` 这两个子字符串上继续从步骤 1 开始递归执行此算法。

给你两个 **长度相等** 的字符串 `s1`和 `s2`，判断 `s2`是否是 `s1`的扰乱字符串。如果是，返回 `true` ；否则，返回 `false` 。

**示例 1：**

```c
输入：s1 = "great", s2 = "rgeat"
输出：true
解释：s1 上可能发生的一种情形是：
"great" --> "gr/eat" // 在一个随机下标处分割得到两个子字符串
"gr/eat" --> "gr/eat" // 随机决定：「保持这两个子字符串的顺序不变」
"gr/eat" --> "g/r / e/at" // 在子字符串上递归执行此算法。两个子字符串分别在随机下标处进行一轮分割
"g/r / e/at" --> "r/g / e/at" // 随机决定：第一组「交换两个子字符串」，第二组「保持这两个子字符串的顺序不变」
"r/g / e/at" --> "r/g / e/ a/t" // 继续递归执行此算法，将 "at" 分割得到 "a/t"
"r/g / e/ a/t" --> "r/g / e/ a/t" // 随机决定：「保持这两个子字符串的顺序不变」
算法终止，结果字符串和 s2 相同，都是 "rgeat"
这是一种能够扰乱 s1 得到 s2 的情形，可以认为 s2 是 s1 的扰乱字符串，返回 true
```

**示例 2：**
```c
输入：s1 = "abcde", s2 = "caebd"
输出：false
```

**示例 3：**

```c
输入：s1 = "a", s2 = "a"
输出：true
```

**提示：**

*   $\text{s1.length = s2.length}$
*   $1 \leq s1.length \leq 30$
*   $s_1$ 和 $s_2$ 由小写英文字母组成

### 解法一
记忆化搜索，一开始没看题解磨了好一会磨出来的，没经过优化
<details>

```java
​class Solution {

    char[] c1, c2;
    
    Boolean[][][][] dp = null;

    public boolean isScramble(String s1, String s2) {
        int m = s1.length();
        int n = s2.length();
        dp = new Boolean[m+1][m+1][n+1][n+1];
        c1 = s1.toCharArray();
        c2 = s2.toCharArray();
        if (!check(0, m-1, 0, n-1)) return false;
        return dfs(0, m-1, 0, n-1);
    }

    public boolean dfs(int i1, int j1, int i2, int j2) {
        if (!check(i1, j1, i2, j2)) {
            return dp[i1][j1][i2][j2] = false;
        }
        if (i1 == j1) {
            if (c1[i1] == c2[i2]) {
                return true;
            }
            return false;
        }
        if (new String(c1, i1, j1-i1+1).equals(new String(c2, i2, j1-i1+1))) {
            // System.out.println(new String(c1, i1, j1-i1+1));
            return true;
        }
        if (dp[i1][j1][i2][j2] != null) {
            return dp[i1][j1][i2][j2];
        }
        int k = i1, p = i2;
        while (k < j1 && p < j2) {
            if (dfs(i1, k, i2, p) && dfs(k+1, j1, p+1, j2)) {
                dp[i1][k][i2][p] = true;
                dp[k+1][j1][p+1][j2] = true;
                return true;
            }
            k++; p++;
        }
        k = i1; p = j2;
        while (k < j1 && p > 0) {
            if (dfs(i1, k, p, j2) && dfs(k+1, j1, i2, p-1)) {
                dp[i1][k][p][j2] = true;
                dp[k+1][j1][i2][p-1] = true;
                return true;
            }
            k++; p--;
        }
        return dp[i1][j1][i2][j2] = false;
    }

    public boolean check(int i1, int j1, int i2, int j2) {
        if (j1-i1 != j2-i2) {
            return false;
        }
        int[] cnt1 = new int[26];
        int[] cnt2 = new int[26];
        while (i1 <= j1 && i2 <= j2) {
            cnt1[c1[i1++]-'a']++;
            cnt2[c2[i2++]-'a']++;
        }
        for (int i = 0; i < 26; i++) {
            if (cnt1[i] != cnt2[i]) {
                return false;
            }
        }
        return true;
    }
}
```
</details>

优化后的写法
