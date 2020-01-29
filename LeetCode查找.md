---
title: 
   LeetCode查找
tags: 
  [LeetCode,查找]
categories:
	[算法]
cover: http://static.imlgw.top/blog/20190915/vqTRbmP6PbOo.jpg?imageslim
date: 2019/9/15
---

## [1. 两数之和](https://leetcode-cn.com/problems/two-sum/)

给定一个整数数组 `nums` 和一个目标值 `target`，请你在该数组中找出和为目标值的那 **两个** 整数，并返回他们的数组下标。

你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。

**示例:**

```java
给定 nums = [2, 7, 11, 15], target = 9

因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]
```

> 平生不识**TwoSum**，做遍LeetCode也枉然

**解法一**

```java
public int[] twoSum(int[] nums, int target) {
    int length = nums.length;
    for (int i = 0; i < length - 1; i++) {
        for (int j = 1; j < length - i; j++) {
            int result = nums[i] + nums[i + j];
            if (result == target) {
                return new int[] { i, i + j };
            }
        }
    }
    return null;
}
```

最开始的做法，直接暴力求解，简单，但是效率很低，50ms，41% beats，其实在笔试或者其它对效率要求没那么严格的地方用暴力法也没毛病节约很多时间，能直接写出最优解肯定好，但是实在没办法了暴力法也不失为一种好方法，最优解可以下来后再研究。

**解法二**

hash查找

```java
public int[] twoSum(int[] nums, int target) {
    HashMap<Integer,Integer> map=new HashMap<>();
    //第一遍把所有的元素和索引存到hashMap中
    for (int i=0;i<nums.length;i++) {
        map.put(nums[i],i);
    }
    //再查找hash
    for (int i=0;i<nums.length;i++) {
        //不能重复所以 下标需要限制下
        if(map.containsKey(target-nums[i]) && map.get(target-nums[i])!=i){
            return new int[]{i,map.get(target-nums[i])};
        }
    }
    return new int[]{};
}
```

其实可以只hash一遍，hash两遍主要考虑顺序的问题。直接利用hashMap查找，效率很高。

```java
public int[] twoSum2(int[] nums, int target) {
    HashMap<Integer,Integer> map=new HashMap<>();
    for (int i=0;i<nums.length;i++) {
        //不能重复所以 下标需要限制下
        if(map.containsKey(target-nums[i]) && map.get(target-nums[i])!=i){
            return new int[]{i,map.get(target-nums[i])};
        }
        map.put(nums[i],i);
    }

    return new int[]{};
}
```

提交记录上最快的做法

```java
public int[] twoSum(int[] nums, int target) {
    int index;
    int indexArrayMax=2047;
    int[] indexArrays=new int[indexArrayMax+1];
    int diff;
    for(int i=1;i<nums.length;i++){
        diff=target-nums[i];
        //i=0时索引无效,所以单独处理
        if(diff==nums[0]){
            return new int[]{0,i};
        }
        index=diff&indexArrayMax;
        if(indexArrays[index]!=0){
            return new int[]{indexArrays[index],i};
        }
        indexArrays[nums[i]&indexArrayMax]=i;   
    }   
    return new int[2];
}
```

没看懂。。。群里问了下，手动hash。。。。以后再来研究吧.

## [349. 两个数组的交集](https://leetcode-cn.com/problems/intersection-of-two-arrays/)

给定两个数组，编写一个函数来计算它们的交集。

**示例 1:**

```java
输入: nums1 = [1,2,2,1], nums2 = [2,2]
输出: [2]
```

**示例 2:**

```java
输入: nums1 = [4,9,5], nums2 = [9,4,9,8,4]
输出: [9,4]
```

**说明:**

- 输出结果中的每个元素一定是唯一的。
- 我们可以不考虑输出结果的顺序。

**解法一**

```java
public int[] intersection(int[] nums1, int[] nums2) {
    Set<Integer> s1=new HashSet<>();
    ArrayList<Integer> res=new ArrayList<>();
    int index=0;
    for (int a:nums1 ) {
        s1.add(a);
    }

    for (int i=0;i<nums2.length;i++) {
        if(s1.contains(nums2[i])){
            res.add(nums2[i]);
            s1.remove(nums2[i]);//别忘了remove掉
        }
    }

    int [] res2=new int[res.size()];
    for (int i=0;i<res.size();i++) {
        res2[i]=res.get(i);
    }
    return res2;
}
```
没啥好说的，这种题确实不难，仔细想想就可以

## [350. 两个数组的交集 II](https://leetcode-cn.com/problems/intersection-of-two-arrays-ii/)

给定两个数组，编写一个函数来计算它们的交集。

**示例 1:**

```java
输入: nums1 = [1,2,2,1], nums2 = [2,2]
输出: [2,2]
```

**示例 2:**

```java
输入: nums1 = [4,9,5], nums2 = [9,4,9,8,4]
输出: [4,9]
```

**说明：**

- 输出结果中每个元素出现的次数，应与元素在两个数组中出现的次数一致。
- 我们可以不考虑输出结果的顺序。

**进阶:**

- 如果给定的数组已经排好序呢？你将如何优化你的算法？
- 如果 nums1 的大小比 nums2 小很多，哪种方法更优？
- 如果 nums2 的元素存储在磁盘上，磁盘内存是有限的，并且你不能一次加载所有的元素到内存中，你该怎么办？

**解法一**

这题和上面的区别就是需要输出所有的交集，重复的也算，所以可以用map的结构记录字符出现的次数

```java
public int[] intersect(int[] nums1, int[] nums2) {
    HashMap<Integer,Integer> map=new HashMap<>();
    for (int i=0;i<nums1.length;i++) {
        map.put(nums1[i],map.getOrDefault(nums1[i],0)+1);
    }
    ArrayList<Integer> res=new ArrayList<>();
    for (int i=0;i<nums2.length;i++) {
        if (map.containsKey(nums2[i])) {
            if (map.get(nums2[i])!=0) {
                //有交集
                res.add(nums2[i]); //添加到结果中
                map.put(nums2[i],map.get(nums2[i])-1); //map映射减一
            }
        }
    }
    int []res2=new int[res.size()];
    for (int i=0;i<res2.length;i++) {
        res2[i]=res.get(i);
    }
    return res2;
}
```

思路也很直白，和上一题的做法类似

**进阶**

**Q1:** 排好序的话就可以直接利用双指针，两个指针分别指向两个数组的头，相等就加入list，不相等就移动小的哪一个，直到有一个指针走到末尾

**Q2:** 这个就很明显了，肯定先把小的哪一个用map映射起来，这样map查找的效率会更高 ？

**Q3:** 这个参考英文版的 [讨论区](https://leetcode.com/problems/intersection-of-two-arrays-ii/discuss/82243/Solution-to-3rd-follow-up-question) 

## [242. 有效的字母异位词](https://leetcode-cn.com/problems/valid-anagram/)

给定两个字符串 s 和 t ，编写一个函数来判断 t 是否是 s 的字母异位词。

**示例 1:**

```java
输入: s = "anagram", t = "nagaram"
输出: true
```

**示例 2:**

```java
输入: s = "rat", t = "car"
输出: false
```

**说明:**

- 你可以假设字符串只包含小写字母。

**进阶:**

- 如果输入字符串包含 unicode 字符怎么办？你能否调整你的解法来应对这种情况？

**解法一**

```java
public boolean isAnagram(String s, String t) {
    if (s.length()!=t.length())return false;
    int[] freq=new int[256];
    for (int i=0;i<s.length();i++) {
        freq[s.charAt(i)]++;
    }
    int count=0,match=0;
    for (int a:freq) {
        if(a!=0){
            count++;
        }
    }
    for (int i=0;i<t.length();i++) {
        if(freq[t.charAt(i)]>0){
            freq[t.charAt(i)]--;
            if(freq[t.charAt(i)]==0){
                match++;
            }
        }
    }
    return match==count;
}
```

这里其实空间还可以优化，题目说了字符串只包含小写字符所以只需要26个int就行了，可以在freq操作的时候 `-'A'` 优化空间

**进阶**

字符包含`unicode` 的话如果再使用int数组就不合适了，这个范围会变得很大，更加通用的方式是采用`HashMap`

## [202. 快乐数](https://leetcode-cn.com/problems/happy-number/)

编写一个算法来判断一个数是不是“快乐数”。

一个“快乐数”定义为：对于一个正整数，每一次将该数替换为它每个位置上的数字的平方和，然后重复这个过程直到这个数变为 1，也可能是无限循环但始终变不到 1。如果可以变为 1，那么这个数就是快乐数。

**示例:** 

```java
输入: 19
输出: true
解释: 
12 + 92 = 82
82 + 22 = 68
62 + 82 = 100
12 + 02 + 02 = 1
```

**解法一**

```java
public static boolean isHappy(int n) {
    char[] nums=null;
    int sum=n;
    while(true) {
        nums=String.valueOf(sum).toCharArray();
        sum=0;
        for (int i=0;i<nums.length;i++) {
            sum+=(nums[i]-48)*(nums[i]-48);
        }
        if (sum==4) {
            return false;
        }else if (sum==1) {
            return true;
        }
    }
}
```

找到了规律，所有不快乐的数(😅，都会进入4 → 16 → 37 → 58 → 89 → 145 → 42 → 20 → 4 的循环，可以直接在sum和这些值相等的时候就return我懒得写那么多，比较取巧但是效率还是挺高的

**解法二**

```java
public static boolean isHappy(int n) {
    char[] nums=null;
    int sum=n;
    HashSet<Integer> set=new HashSet<>();
    while(true) {
        nums=String.valueOf(sum).toCharArray();
        sum=0;
        for (int i=0;i<nums.length;i++) {
            sum+=(nums[i]-48)*(nums[i]-48);
        }
        if (sum==1) {
            return true;
        }else if (set.contain(sum)){
            return false;
        }else{
            set.add(sum);    
        }
    }
}
```
这种做法就比较常规，也是符合这篇主题**查找**的解法，代码比较简单就不啰嗦了

## [290. 单词规律](https://leetcode-cn.com/problems/word-pattern/)

给定一种规律 pattern 和一个字符串 str ，判断 str 是否遵循相同的规律。

这里的 **遵循** 指完全匹配，例如， pattern 里的每个字母和字符串 str 中的每个非空单词之间存在着双向连接的对应规律。

**示例1:**

```java
输入: pattern = "abba", str = "dog cat cat dog"
输出: true
```

**示例 2:**

```java
输入:pattern = "abba", str = "dog cat cat fish"
输出: false
```

**示例 3:**

```java
输入: pattern = "aaaa", str = "dog cat cat dog"
输出: false
```

**示例 4:**

```java
输入: pattern = "abba", str = "dog dog dog dog"
输出: false
```

**说明:**
你可以假设 `pattern` 只包含小写字母， `str` 包含了由单个空格分隔的小写字母

**解法一**

```java
public static boolean wordPattern(String pattern, String str) {
    HashMap<Character,String> map=new LinkedHashMap<>();
    String[] strs=str.split(" ");
    char[] p=pattern.toCharArray();
    if (strs.length!=p.length) {
        return false;
    }
    for (int i=0;i<p.length;i++) {
        if (map.containsKey(p[i])) {
            if (!map.get(p[i]).equals(strs[i])) {
                return false;
            }
        }else{
            //这里直接和前一个比较的，正确做法是用map.containsValue判断是否已经添加
            /*if (strs[i].equals(strs[i-1])) {
                return false;
            }*/
            if (map.containsValue(strs[i])) {
                return false;           
            }
            map.put(p[i],strs[i]);
        }
    }
    return true;
}
```
很简单的题，需要对两个字符串的模式进行匹配，借助Hash表直接将两个String进行一对一的映射，既然要匹配那么`同一个key字符对应的value字符肯定是一样的`，还有一点需要注意的是在遇到一个新的key字符的时候，需要判断对应位置的value字符出现过没有，出现过就直接return false，这一点第一遍的时候没考虑到，`不同的key字符对应的value字符肯定是不一样的`

> 因为第一次没考虑到第二种情况，提交后竟然跑过了`31/33` 个case，然后就感觉这题case可能有点问题，然后自己写了个错的算法居然也跑过了，具体的代码在上面的注释中，感兴趣的可以去试试，我已经提交case了但是还没回应我

## [205. 同构字符串](https://leetcode-cn.com/problems/isomorphic-strings/)

给定两个字符串 s 和 t，判断它们是否是同构的。

如果 s 中的字符可以被替换得到 t ，那么这两个字符串是同构的。

所有出现的字符都必须用另一个字符替换，同时保留字符的顺序。两个字符不能映射到同一个字符上，但字符可以映射自己本身。

**示例 1:**

```java
输入: s = "egg", t = "add"
输出: true
```


**示例 2:**

```java
输入: s = "foo", t = "bar"
输出: false
```

**示例 3:**

```java
输入: s = "paper", t = "title"
输出: true
```

**说明:**

- 你可以假设 s 和 t 具有相同的长度。

**解法一**

这题和上面一模一样，Hash表的解法就不写了，这题都是单个的字符，可以不用Hash表，可以用数组优化

```java
public boolean isIsomorphic2(String s, String t) {
    if (s.length()!=t.length()) {
        return false;
    }
    int[] key=new int[256];
    int[] value=new int[256];

    for (int i=0;i<s.length();i++) {
        int cs=s.charAt(i);
        int ct=t.charAt(i);
        if(key[cs]!=0){ //cs出现过
            if (key[cs]!=ct) {
                return false;
            }
        }else{//cs没出现过
            if (value[ct]!=0) {
                return false;
            }
            key[cs]=ct;
            value[ct]=cs;
        }
    }
    return true;
}
```

💬 同样的，这题和上面的290一样，case也有问题，直接和前一个字符比较就可以过

## [454. 四数相加 II](https://leetcode-cn.com/problems/4sum-ii/)

给定四个包含整数的数组列表 A , B , C , D ,计算有多少个元组 (i, j, k, l) ，使得 A[i] + B[j] + C[k] + D[l] = 0。

为了使问题简单化，所有的 A, B, C, D 具有相同的长度 N，且 `0 ≤ N ≤ 500` 。所有整数的范围在 -2^28 到 2^28 - 1 之间，最终结果不会超过 2^31 - 1 。

**例如:**

```java
输入:
A = [ 1, 2]
B = [-2,-1]
C = [-1, 2]
D = [ 0, 2]

输出:
2

解释:
两个元组如下:

1. (0, 0, 0, 1) -> A[0] + B[0] + C[0] + D[1] = 1 + (-2) + (-1) + 2 = 0
2. (1, 1, 0, 0) -> A[1] + B[1] + C[0] + D[0] = 2 + (-1) + (-1) + 0 = 0
```

**解法一**

这题其实看数据规模就知道应该写一个什么样复杂度的算法了`0~500`，暴力的话会很恐怖`O(N^4)`，这里可以考虑将其中一个放到hash表中，然后遍历其他的3个，时间复杂度优化到了`O(N^3)`，但是时间复杂度还是很恐怖，所以可以考虑将两个数组的和放到hash表中，这样就可以将时间复杂度优化到`O(N^2)`

```java
public int fourSumCount(int[] A, int[] B, int[] C, int[] D) {
    HashMap<Integer,Integer> map=new HashMap<>();
    for (int i=0;i<C.length;i++) {
        for (int j=0;j<D.length;j++) {
            int key=C[i]+D[j];
            map.put(key,map.getOrDefault(key,0)+1);
        }
    }
    int res=0;
    for (int i=0;i<A.length;i++) {
        for (int j=0;j<B.length;j++) {
            int key=A[i]+B[j];
            if(map.containsKey(-key)){
                res+=map.get(-key);
            }
        }
    }
    return res;
}
```

## [451. 根据字符出现频率排序](https://leetcode-cn.com/problems/sort-characters-by-frequency/)

给定一个字符串，请将字符串里的字符按照出现的频率降序排列。

**示例 1:**

```java
输入:
"tree"

输出:
"eert"

解释:
'e'出现两次，'r'和't'都只出现一次。
因此'e'必须出现在'r'和't'之前。此外，"eetr"也是一个有效的答案。
```

**示例 2:**

```java
输入:
"cccaaa"

输出:
"cccaaa"

解释:
'c'和'a'都出现三次。此外，"aaaccc"也是有效的答案。
注意"cacaca"是不正确的，因为相同的字母必须放在一起。
```

**示例 3:**

```java
输入:
"Aabb"

输出:
"bbAa"

解释:
此外，"bbaA"也是一个有效的答案，但"Aabb"是不正确的。
注意'A'和'a'被认为是两种不同的字符。
```

**解法一**

```java
public static String frequencySort(String s) {
    if (s==null || s.length()<1) {
        return s;
    }
    HashMap<Character,Integer> map=new HashMap<>();
    for (int i=0;i<s.length();i++) {
        map.put(s.charAt(i),map.getOrDefault(s.charAt(i),0)+1);
    }
    ArrayList<HashMap.Entry> list=new ArrayList<>();
    for(HashMap.Entry entry:map.entrySet()){
        list.add(entry);
    }
    list.sort((e1,e2)->(Integer)e2.getValue()-(Integer)e1.getValue());
    StringBuilder res=new StringBuilder();
    for (int i = 0; i < list.size(); i++) {
        Integer value = (Integer)list.get(i).getValue();
        while (value>0){
            res.append(list.get(i).getKey());
            value--;
        }
    }
    return res.toString();
}
```
这题其实也是TopK问题，直接的想法就是用hashMap统计各个字符出现的个数，然后排序再拼接为结果，其实这题一开始是TLE了的，一开始没注意直接用的String拼接的，效率很低，改用StringBuilder后就过了，虽然效率还是很低 138ms，垫底

**解法二**

```java
public  static String frequencySort2(String s) {
    if (s==null || s.length()<1) {
        return s;
    }
    int[] freq=new int[256];
    for (int i=0;i<s.length();i++) {
        freq[s.charAt(i)]++;
    }
    int[] freq_bak=freq.clone();
    Arrays.sort(freq);
    StringBuilder res=new StringBuilder();
    //从大到小
    for (int i = 255; i>=0 && freq[i]!=0; i--) {
        for (int j=0;j<255;j++) {
            //找到原数组中对应的字符
            //只要出现次数一样的就行了
            if(freq_bak[j]==freq[i]){
                //根据freq_bak[j]构造结果
                while(freq_bak[j]>0){
                    res.append((char)j);
                    freq_bak[j]--;
                }
                break;
            }
        }
    }
    return res.toString();
}
```
15ms，90% 其实思路和上面是一样的，都是统计数量后进行排序，然后重建字符串，但是用数组的方式明显会比HashMap效率会更高的多，后面的两层循环都是在常数时间内，主要是重建字符串和排序消耗时间，时间复杂度应该是`O(NlogN)`

**解法三**

```java
public  static String frequencySort3(String s) {
    if (s==null || s.length()<1) {
        return s;
    }
    ArrayList<Character> [] bucket=new ArrayList[s.length()+1];

    int[] freq=new int[256];

    for (int i=0;i<s.length();i++) {
        freq[s.charAt(i)]++;
    }

    for (int i=0;i<s.length();i++) {
        if (bucket[freq[s.charAt(i)]]==null) {
            bucket[freq[s.charAt(i)]]=new ArrayList<>();
        }
        //每个元素只进入一次
        if (!bucket[freq[s.charAt(i)]].contains(s.charAt(i))) {
            bucket[freq[s.charAt(i)]].add(s.charAt(i));
        }
    } 
    //printArray(bucket);
    StringBuilder res=new StringBuilder();
    for (int i=bucket.length-1;i>=0;i--) {
        //过滤0
        if (bucket[i]==null) {
            continue;
        }
        //出现i次的字符list
        ArrayList<Character> temp=bucket[i];
        //遍历出现次数相同的list()
        for (int j=0;j<temp.size();j++) { 
            //遍历出现的次数
            for (int count=0;count<i;count++) {
                res.append(temp.get(j));
            }
        }
    }
    return res.toString();
}
```
50ms，50% 这个是根据 [前k个高频元素](http://imlgw.top/2019/05/04/leetcode-shu-zu-tag/#347-%E5%89%8D-K-%E4%B8%AA%E9%AB%98%E9%A2%91%E5%85%83%E7%B4%A0) 中桶排序的解法来的，当然这里并不是最优解，只是一种思路，其实写起来还是挺麻烦的，时间复杂度略高，主要是在桶排序的时候添加元素做不到O(N)需要判断元素是否添加，一个元素只能在list中添加一次，否则后面重建字符串的时候就会有问题

## [49. 字母异位词分组](https://leetcode-cn.com/problems/group-anagrams/)

给定一个字符串数组，将字母异位词组合在一起。字母异位词指字母相同，但排列不同的字符串。

**示例:**

```java
输入: ["eat", "tea", "tan", "ate", "nat", "bat"],
输出:
[
  ["ate","eat","tea"],
  ["nat","tan"],
  ["bat"]
]
```

**说明：**

- 所有输入均为小写字母
- 不考虑答案输出的顺序

**解法一**

算是暴力法了，借助上面的同构题思路来遍历判断

```java
public List<List<String>> groupAnagrams(String[] strs) {
    ArrayList<List<String>> res=new ArrayList<>();
    for (int i=0;i<strs.length;i++) {
        if ("7"==strs[i]) {
            continue;
        }
        ArrayList<String> group=new ArrayList<String>();
        group.add(strs[i]);
        for (int j=i+1;j<strs.length;j++) {
            if ("7"==strs[j]) {
                continue;
            }
            if(isAnagram(strs[i],strs[j])){
                group.add(strs[j]);
                //有分组了
                strs[j]="7";
            }
        }
        res.add(group);
    }
    return res;
}

public boolean isAnagram(String str1,String str2){
    if(str1.length()!=str2.length()){
        return false;
    }
    int[] freq=new int[26];
    for (int i=0;i<str1.length();i++) {
        freq[str1.charAt(i)-'a']++;
    }
    for (int i=0;i<str2.length();i++) {
        freq[str2.charAt(i)-'a']--;
    }
    for (int i=0;i<freq.length;i++) {
        if (freq[i]!=0) {
            return false;
        }
    }
    return true;
}
```
可以看到里面有一个`7` 其实没什么含义就是为了表示这个字符已经有分组了，这里一开始我是用的equals来比较的这个7结果超时了，然后换成了==勉强跑过了，可能是个例，因为我后来用boolean数组也没跑过。。。

> 这里用==可以比较的原因可能是strs和字面量 "7"都在字符常量池中，但是这里并不建议这样比较，这里可以说是个反例了，比较字符串请用`equals` ！！！

```java
public List<List<String>> groupAnagrams(String[] strs) {
    ArrayList<List<String>> res=new ArrayList<>();
    boolean[] flag=new boolean[strs.length()];
    for (int i=0;i<strs.length;i++) {
        if (flag[i]) continue;
        ArrayList<String> group=new ArrayList<String>();
        group.add(strs[i]);
        for (int j=i+1;j<strs.length;j++) {
            if(flag[j])continue;
            if(isAnagram(strs[i],strs[j])){
                group.add(strs[j]);
                flag[j]=true;
            }
        }
        res.add(group);
    }
    return res;
}
```
**解法二**

利用排序结果来作为key，将排序结果相同的str映射到一起

```java
//排序解法
public List<List<String>> groupAnagrams(String[] strs) {
    HashMap<String,List<String>> map=new HashMap<>();
    for (int i=0;i<strs.length;i++) {
        char[] strs_i=strs[i].toCharArray();
        //排序，将结果作为key
        Arrays.sort(strs_i);
        String key=String.valueOf(strs_i);

        if(map.containsKey(key)){
            //存在同构的key，直接添加进去
            map.get(key).add(strs[i]);
        }else{
            //不存在就创建一个，然后将自己添加进去
            map.put(key,new ArrayList<>());
            map.get(key).add(strs[i]);
        }
    }
    return new ArrayList(map.values());
}
```
时间复杂度`O(NKlogK)`，K为字符数组中最长的字符串，`O(KlogK)` 是给这个字符串排序的结果

**解法三**

根据出现频次构成的字符串作为key，比如`aba`以及所有的异位词都会被映射为`2#1#` 

```java
public List<List<String>> groupAnagrams(String[] strs) {
    HashMap<String,List<String>> map=new HashMap<>();
    int[] freq=new int[26];
    for (int i=0;i<strs.length;i++) {
        //Arrays.fill(freq,0);
        //统计字符出现的频次
        for (int j=0;j<strs[i].length();j++) {
            freq[strs[i].charAt(j)-'a']++;
        }
        //构建唯一映射的key
        StringBuilder key=new StringBuilder();
        //这个其实类似桶排序,依次取abcde...
        for (int j=0;j<26;j++) {
            key.append(freq[j]);
            //这个#很关键,为了防止重复,因为有的字符可能出现两位数的次数,仅仅对比数字是无法确定的
            key.append("#");
            //重置为0方便后面重复使用
            freq[j]=0;
        }
        String skey=key.toString();
        if(map.containsKey(skey)){
            map.get(skey).add(strs[i]);
        }else{
            map.put(skey,new ArrayList());
            map.get(skey).add(strs[i]);
        }
    }
    return new ArrayList(map.values());
}
```
45ms，21% 时间复杂度`O(NK)` K为字符数组中最长的字符串的长度，很玄学，讲道理应该不会这么慢，看了leetcode上前几名跟我的差不多，开始做的时候直接用 `StringBuilder` 对象作为了key结果肯定不对，StringBuilder没有覆盖equals方法，key永远不会相等，每次都是新的key

**解法四**

**算术基本定理**，又称为正整数的唯一分解定理，即：每个大于1的自然数，要么本身就是质数，要么可以写为2个以上的质数的积，而且这些质因子按大小排列之后，写法仅有一种方式

```java
public List<List<String>> groupAnagrams(String[] strs) {
    HashMap<Integer, List<String>> hash = new HashMap<>();
    //每个字母对应一个质数
    int[] prime = { 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97, 101, 103 };
    for (int i = 0; i < strs.length; i++) {
        int key = 1;
        //累乘得到 key
        for (int j = 0; j < strs[i].length(); j++) {
            key *= prime[strs[i].charAt(j) - 'a'];
        } 
        if (hash.containsKey(key)) {
            hash.get(key).add(strs[i]);
        } else {
            List<String> temp = new ArrayList<String>();
            temp.add(strs[i]);
            hash.put(key, temp);
        }
    }
    return new ArrayList<List<String>>(hash.values());
}
```

时间复杂度`O(NK)`，强的8行

> 分析完上面三种解法后其实很同意得出这题的关键：`给同组的异位词找到一个相同的映射key`，尽量的缩短求这个映射的时间就可优化整个算法

## [447. 回旋镖的数量](https://leetcode-cn.com/problems/number-of-boomerangs/)

给定平面上 n 对不同的点，“回旋镖” 是由点表示的元组 `(i, j, k)` ，其中 i 和 j 之间的距离和 i 和 k 之间的距离相等（**需要考虑元组的顺序**）

找到所有回旋镖的数量。你可以假设 n 最大为 500，所有点的坐标在闭区间 `[-10000, 10000]` 中。

**示例:**

```java
输入:
[[0,0],[1,0],[2,0]]

输出:
2

解释:
两个回旋镖为 [[1,0],[0,0],[2,0]] 和 [[1,0],[2,0],[0,0]]
```

**解法一**

```java
public static int numberOfBoomerangs(int[][] points) {
    int res=0;
    HashMap<Integer,Integer> map=new HashMap<>();
    for (int i=0;i<points.length;i++) {
        for (int j=0;j<points.length;j++) {
            if (i!=j){
                int dis=dis(points[i],points[j]);
                map.put(dis,map.getOrDefault(dis,0)+1);
            }
        }
        //C2m 组合问题
        for (Integer count:map.values()) {
            if (count>1) {
                res+=count*(count-1);
            }
        }
        map.clear();
    }
    return res;
}

public static int dis(int[] a,int[] b){
    return (a[0]-b[0])*(a[0]-b[0])+(a[1]-b[1])*(a[1]-b[1]);
}
```
看一下给的数据量`500`就知道复杂度只能是`O(N^2)` 的，用Hash表统计到当前点的距离相同的点有多少个，然后利用组合数求多少种组合，一开始并没有想到这种方法，我想的是利用坐标系的对称来做，太菜了

这里还有个小细节，一开始将HashMap的创建放在内循环中，发现效率很低，300ms左右，然后将创建HashMap移出去后用clear清空，瞬间快了100ms左右，创建HashMap的成本果然还是挺大的

这题其实还可以减少内循环的数量，

## [217. 存在重复元素](https://leetcode-cn.com/problems/contains-duplicate/)

给定一个整数数组，判断是否存在重复元素。

如果任何值在数组中出现至少两次，函数返回 true。如果数组中每个元素都不相同，则返回 false。

**示例 1:**

```java
输入: [1,2,3,1]
输出: true
```

**示例 2:**

```java
输入: [1,2,3,4]
输出: false
```

**示例 3:**

```java
输入: [1,1,1,3,3,4,3,2,4,2]
输出: true
```

**解法一**

借助Hash表，很简单的题

```java
public static boolean containsDuplicate(int[] nums) {
    if (nums==null) return false;
    HashSet<Integer> set=new HashSet<>();
    for (int i=0;i<nums.length;i++) {
        if (set.contains(nums[i])) {
            return true;
        }
        set.add(nums[i]);
    }
    return false;
}
```
其实还可以优化下

```java
public static boolean containsDuplicate(int[] nums) {
    if (nums==null) return false;
    HashSet<Integer> set=new HashSet<>();
    for (int i=0;i<nums.length;i++) {
        if (!set.add(nums[i])) {
            return true;
        }
    }
    return false;
}
```
`set.add()` 本身就带有返回值，可以减少很多判断，这题还有一个进阶版 [219. 存在重复元素 II](https://leetcode-cn.com/problems/contains-duplicate-ii/) 也不难，我放到 [滑动窗口专题](http://imlgw.top/2019/07/20/leetcode-hua-dong-chuang-kou-tag/#3-%E6%97%A0%E9%87%8D%E5%A4%8D%E5%AD%97%E7%AC%A6%E7%9A%84%E6%9C%80%E9%95%BF%E5%AD%90%E4%B8%B2) 中去了

## [220. 存在重复元素 III](https://leetcode-cn.com/problems/contains-duplicate-iii/)

给定一个整数数组，判断数组中是否有两个不同的索引 i 和 j，使得 `nums [i]` 和 `nums [j]` 的差的绝对值最大为 t，并且 i 和 j 之间的差的绝对值最大为 k

**示例 1:**

```java
输入: nums = [1,2,3,1], k = 3, t = 0
输出: true
```

**示例 2:**

```java
输入: nums = [1,0,1,1], k = 1, t = 2
输出: true
```

**示例 3:**

```java
输入: nums = [1,5,9,1,5,9], k = 2, t = 3
输出: false
```

**解法一**

这里其实可以算难题了，通过率只有20%，利用Java中提供的TreeMap，有顺序而且插入和删除等操作效率都很高（logN），然后查找指定范围的元素，看符不符合题目要求

```java
public boolean containsNearbyAlmostDuplicate(int[] nums, int k, int t) {
    TreeSet<Integer> set=new TreeSet<>();
    for (int i=0;i<nums.length;i++) {
        //大于nums[i]的最小元素
        Integer ceiling=set.ceiling(nums[i]);
        //小于nums[i]的最大元素
        Integer floor=set.floor(nums[i]);
        //防止溢出
        long temp1=Long.valueOf(nums[i])+Long.valueOf(t);
        long temp2=Long.valueOf(nums[i])-Long.valueOf(t);
        if((ceiling!=null && ceiling<=temp1) || (floor!=null && floor>=temp2)) {
            return true;
        }
        set.add(nums[i]);
        if (set.size()>k) {
            //移除左边界
            set.remove(nums[i-k]);
        }
    }
    return false;
}
```
72ms，5%左右，整体时间复杂度`O(NlogN)` 

**解法二**

看了下评论区，发现其实可以直接比较两个边界，这样可以少一次查找的操作，效率有很大提升

```java
public boolean containsNearbyAlmostDuplicate2(int[] nums, int k, int t) {
    TreeSet<Long> set=new TreeSet<>();
    for (int i=0;i<nums.length;i++) {
        //大于nums[i]-t的最小元素
        Long ceil=set.ceiling((long)nums[i]-(long)t);
        if(ceil!=null && ceil<=(long)nums[i]+(long)t) {
            return true;
        }
        set.add((long)nums[i]);
        if (set.size()>k) {
            set.remove((long)nums[i-k]);
        }
    }
    return false;
}
```
43ms，56%左右，依然要注意溢出的问题

## [1282. 用户分组](https://leetcode-cn.com/problems/group-the-people-given-the-group-size-they-belong-to/)

有 n 位用户参加活动，他们的 ID 从 0 到 n - 1，每位用户都 恰好 属于某一用户组。给你一个长度为 n 的数组 groupSizes，其中包含每位用户所处的用户组的大小，请你返回用户分组情况（存在的用户组以及每个组中用户的 ID）。

你可以任何顺序返回解决方案，ID 的顺序也不受限制。此外，题目给出的数据保证至少存在一种解决方案。

**示例 1：**

```java
输入：groupSizes = [3,3,3,3,3,1,3]
输出：[[5],[0,1,2],[3,4,6]]
解释： 
其他可能的解决方案有 [[2,1,6],[5],[0,4,3]] 和 [[5],[0,6,2],[4,3,1]]。
```

**示例 2：**

```java
输入：groupSizes = [2,1,3,3,3,2]
输出：[[1],[0,5],[2,3,4]]
```

**提示：**

- `groupSizes.length == n`
- `1 <= n <= 500`
- `1 <= groupSizes[i] <= n`

**解法一**

12.8周赛的题，我是模拟做的，看了别人的做法还是感觉这种比较优雅

```java
public List<List<Integer>> groupThePeople(int[] groupSizes) {
    HashMap<Integer,List<Integer>> map=new HashMap<>();
    List<List<Integer>> res=new LinkedList<>();
    for (int i=0;i<groupSizes.length;i++) {
        if (!map.containsKey(groupSizes[i])) {
            List<Integer> list=new LinkedList();
            map.put(groupSizes[i],list);
        }
        List<Integer> gl=map.get(groupSizes[i]);
        gl.add(i);
        if (gl.size()==groupSizes[i]) {
            res.add(gl);
            map.remove(groupSizes[i]);
        }
    }
    return res;
}
```
时间复杂度`O(N)`

**解法二**

模拟的解法，时间复杂度`O(N^2)`

```java
public List<List<Integer>> groupThePeople(int[] groupSizes) {
    boolean[] visit=new boolean[groupSizes.length];
    List<List<Integer>> res=new LinkedList<>();
    for (int i=0;i<groupSizes.length;i++) {
        if (visit[i]) {
            continue;
        }
        List<Integer> list= new LinkedList<>();
        list.add(i);
        for (int j=i+1;j<groupSizes.length;j++) {
            if (visit[j]) {
                continue;
            }
            if (list.size()==groupSizes[i]) {
                break;
            }
            if (groupSizes[j]==groupSizes[i]) {
                list.add(j);
                visit[j]=true;
            }
        }
        res.add(list);
    }
    return res;
}
```

## [128. 最长连续序列](https://leetcode-cn.com/problems/longest-consecutive-sequence/)

给定一个未排序的整数数组，找出最长连续序列的长度。

要求算法的时间复杂度为 `O(n)`。

**示例:**

```java
输入: [100, 4, 200, 1, 3, 2]
输出: 4
解释: 最长连续序列是 [1, 2, 3, 4]。它的长度为 4。
```

**解法一**

借助Hash表的暴力解法

```java
public int longestConsecutive(int[] nums) {
    if (nums ==null || nums.length<=0) {
        return 0;
    }
    HashSet<Integer> set=new HashSet();
    for (int n:nums) {
        set.add(n);
    }
    int max=0;
    for (int i=0;i<nums.length;i++) {
        int num=nums[i];
        int res=1;
        while(set.contains(num+1)){
            res++;
            num++;
        }
        max=Math.max(max,res);
    }
    return max;
}
```

对于每个元素在Hash表中查找它的下一个连续的元素`num+1` 有没有，有的话就继续往下找，最后求的以每个元素开头的最长子序列长度，时间复杂度`O(N^2)`不符合题目的要求

所以我们需要优化我们的算法，其实上面的过程我们很容易就看出啦里面会有重复的计算

`eg. 5，4，6，7，8`  我们在第一个5的时候计算了以5开头的 5，6，7，8这条路径，然后转而计算第二个4，计算了以4开头的4，5，6，7，8这里其实就发生了重复的计算，那么我们这里求的是最长的序列，所以我们需要舍弃第一个，也就是说我们遍历第一个5的时候直接跳过，跳过的依据就是判断 5-1 在不在集合中，如果在那么以它开头的序列一定不是不会是最长的，反之则有可能是最长的，我们统计所有这样的序列长度，最后求一个最大值就可以了

**解法二**

每个元素最多遍历2次，所以时间复杂度为O(N)符合要求

```java
public int longestConsecutive(int[] nums) {
    if (nums ==null || nums.length<=0) {
        return 0;
    }
    HashSet<Integer> set=new HashSet();
    for (int n:nums) {
        set.add(n);
    }
    int max=0;
    for (int i=0;i<nums.length;i++) {
        int num=nums[i];
        if (!set.contains(num-1)) {
            int res=1;
            while(set.contains(num+1)){
                res++;
                num++;
            }
            max=Math.max(max,res);
        }
    }
    return max;
}
```

**解法三**

并查集的解法，略微麻烦点，但是毕竟这题的tag就是并查集，还是实现一下

```java
//并查集
HashMap<Integer,Integer> parent;

HashMap<Integer,Integer> size;

int max=1;

public int find(int index){
    while(parent.get(index)!=index){
        //parent[index]=parent[parent[index]];
        parent.put(index,parent.get(index));
        index=parent.get(index);
    }
    return index;
}

public void union(int p,int q){
    int pID=find(p);
    int qID=find(q);
    if (pID==qID) {
        return;
    }
    int pSize=size.get(pID);
    int qSize=size.get(qID);
    if (pSize > qSize) {
        //parent[qID]=pID;
        parent.put(qID,pID);
        //size[pID]+=size[qID];
        size.put(pID,pSize+qSize);
    }else{
        //parent[pID]=qID;
        parent.put(pID,qID);
        //size[qID]+=size[pID];
        size.put(qID,pSize+qSize);
    }
    max=Math.max(max,pSize+qSize); //统计最大值
}

public void initUnionFind(int[]nums){
    parent=new HashMap<>();
    size=new HashMap<>();
    for (int i=0;i<nums.length;i++) {
        parent.put(nums[i],nums[i]);
        size.put(nums[i],1);
    }
}

public int longestConsecutive(int[] nums) {
    if (nums ==null || nums.length<=0) {
        return 0;
    }
    HashSet<Integer> set=new HashSet();
    for (int i=0;i<nums.length;i++) {
        set.add(nums[i]);
    }
    initUnionFind(nums);
    for (int i=0;i<nums.length;i++) {
        if (set.contains(nums[i]-1)) { //判断-1或者+1都可以
            union(nums[i],nums[i]-1);
        }
    }
    return max;
}
```

## _前缀和相关_

## [560. 和为K的子数组](https://leetcode-cn.com/problems/subarray-sum-equals-k/)

给定一个整数数组和一个整数 k，你需要找到该数组中和为 k 的连续的子数组的个数。

**示例 1 :**

```java
输入:nums = [1,1,1], k = 2
输出: 2 , [1,1] 与 [1,1] 为两种不同的情况。
```


**说明 :**

- 数组的长度为 [1, 20,000]。
- 数组中元素的范围是 [-1000, 1000] ，且整数 k 的范围是 [-1e7, 1e7]。

**解法一**

前缀和 + hash表

```java
public int subarraySum(int[] nums, int k) {
    HashMap<Integer,Integer> map = new HashMap<>();
    map.put(0,1);//初始化头哨兵,避免下标转换
    int sum=0,res=0;
    for (int i=0;i<nums.length;i++) {
        sum+=nums[i];
        //-1 -1 1 | 0
        if (/*sum>=k && */map.containsKey(sum-k)) {
            res+=map.get(sum-k);
        }
        map.put(sum,map.getOrDefault(sum,0)+1);
    }
    return res;
}
```
我们将各个位置的前缀和作为键，这个前缀和在**当前位置之前出现的次数作为键** (这一点保证了连续，不会找到后面去)

然后我们的目标就是找到和为k区间有多少个，区间和利用前缀和可以直接算出，也就是 

`sum[i~j] = sum[j] -sum[i]= k` 然后这个问题就可以转换为，当我们遍历到某个元素的时候，我们在map中查找前缀和为`sum[j] - k`的元素有几个，这样就可以得到区间和为k的区间有多少个！

值得注意的地方就是，需要添加一个初始的`sum=0`的值，避免下标的转换

画个图就像下面这样：

![img](http://static.imlgw.top/blog/20191104/ECiyYqso8i2K.png?imageslim)

## [1248. 统计「优美子数组」](https://leetcode-cn.com/problems/count-number-of-nice-subarrays/)

给你一个整数数组 nums 和一个整数 k。

如果某个子数组中恰好有 k 个奇数数字，我们就认为这个子数组是**「优美子数组」**。

请返回这个数组中「优美子数组」的数目。

**示例 1：**

```java
输入：nums = [1,1,2,1,1], k = 3
输出：2
解释：包含 3 个奇数的子数组是 [1,1,2,1] 和 [1,2,1,1] 。
```


**示例 2：**

```java
输入：nums = [2,4,6], k = 1
输出：0
解释：数列中不包含任何奇数，所以不存在优美子数组。
```


**示例 3：**

```java
输入：nums = [2,2,2,1,2,2,1,2,2,2], k = 2
输出：16
```

**提示：**

- `1 <= nums.length <= 50000`
- `1 <= nums[i] <= 10^5`
- `1 <= k <= nums.length`

**解法一**

11.3 周赛第二题，没做出来，以为是滑动窗口，滑了半天没滑出来，后来看了解答知道了是利用前缀和 + Hash，其实和上面一题是类似的，相当于上一题的进阶

```java
public int numberOfSubarrays(int[] nums, int k) {
    HashMap<Integer,Integer> map=new HashMap<>();
    int sum=0,res=0;
    map.put(0,1);
    for (int i=0;i<nums.length;i++) {
        if (nums[i]%2==1) {
            sum++;
        }
        //题目说了都是正数，所以可以优化下
        if (sum>=k && map.containsKey(sum-k)) {
            res+=map.get(sum-k);
        }
        map.put(sum,map.getOrDefault(sum,0)+1);
    }
    return res;
}
```
其实这里就是把奇数都看作 1，偶数都看作0，这样问题就变成了求和为k的区间个数有多少个，然后在根据上面的前缀和+Hash表，就可以很容易的得到答案，还有一点就是题目说了数据都是正数，所以在判断的时候可以加一个`sum>=k` 来减少一点判断，当然这题题目指定了数据的范围，所以还可以直接用数组做map映射

```java
public static int numberOfSubarrays(int[] nums, int k) {
    int res=0;
    int sum=0;
    //HashMap<Integer,Integer> map=new HashMap<>();
    int[] map=new int[100001];
    map[0]=1;
    for (int i=0;i<nums.length;i++) {
        if (nums[i]%2==1) {
            sum++;
        }
        if (sum>=k) {
            res+=map[sum-k];
        }
        map[sum]++;
        //System.out.println(sum+":"+map[sum]);
    }
    return res;
}
```

