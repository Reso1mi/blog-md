 > 见到好几次了，感觉不是很难，学一手
 ## [1044\. 最长重复子串](https://leetcode-cn.com/problems/longest-duplicate-substring/)

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