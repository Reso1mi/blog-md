---
title: 
  LeetCode数组
tags:
  [数组,LeetCode]
categories:
	[算法]
date: 2019/5/4
---

## LeetCode 数组

面试中的算法问题，有很多并不需要复杂的数据结构支撑。就是用数组，就能考察出很多东西了。其实，经典的排序问题，二分搜索等等问题，就是在数组这种最基础的结构中处理问题的。 

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

**hash查找**

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

## [167. 两数之和 II - 输入有序数组](https://leetcode-cn.com/problems/two-sum-ii-input-array-is-sorted/)

给定一个已按照**升序排列** 的有序数组，找到两个数使得它们相加之和等于目标数。

函数应该返回这两个下标值 index1 和 index2，其中 index1 必须小于 index2*。*

**说明:**

- 返回的下标值（index1 和 index2）不是从零开始的。
- 你可以假设每个输入只对应唯一的答案，而且你不可以重复使用相同的元素。

**示例:**

```java
输入: numbers = [2, 7, 11, 15], target = 9
输出: [1,2]
解释: 2 与 7 之和等于目标数 9 。因此 index1 = 1, index2 = 2 。
```

两数之和的变种，看见**有序**其实也可以使用二分来做，但是时间复杂度是`O(NlogN)`，相对较高

```java
public int[] twoSum(int[] numbers, int target) {
        if(numbers==null||numbers.length<=0){
            return null;
        }
        int left=0,right=numbers.length-1;
        while(right>left){
            int sum=numbers[right]+numbers[left];
            if(sum==target){
                return new int[]{left+1,right+1};
            }if(sum<target){
                left++;
            }else{
                right--;
            }
        }
        return null;
}
```

**对撞指针**，很基础的题。

## [11. 盛最多水的容器](https://leetcode-cn.com/problems/container-with-most-water/)

给定 *n* 个非负整数 *a*1，*a*2，...，*a*n，每个数代表坐标中的一个点 (*i*, *ai*) 。在坐标内画 *n* 条垂直线，垂直线 *i* 的两个端点分别为 (*i*, *ai*) 和 (*i*, 0)。找出其中的两条线，使得它们与 *x* 轴共同构成的容器可以容纳最多的水。

**说明：**你不能倾斜容器，且 *n* 的值至少为 2。

![mark](http://static.imlgw.top///20190505/1uze479AHHLo.png?imageslim)

图中垂直线代表输入数组 [1,8,6,2,5,4,8,3,7]。在此情况下，容器能够容纳水（表示为蓝色部分）的最大值为 49。

**示例:**

```java
输入: [1,8,6,2,5,4,8,3,7]
输出: 49
```

**解法一**

```java
public int maxArea(int[] height) {
        int len=height.length;
        if(len==0){
            return 0;
        }
        int max=Integer.MIN_VALUE;
        for(int i=0;i<len-1;i++){
            for(int j=i+1;j<len;j++){
                int minHight=height[i]>height[j]?height[j]:height[i];
                max=max>(j-i)*minHight ? max:(j-i)*minHight;
            }
        }
        return max;
 }
```

522ms，13% 垫底了，别问，问就是暴力🤣

```java
public int maxArea(int[] height) {
        int len=height.length;
        if(len==0){
            return 0;
        }
        int head=0,tail=len-1;
        int max=Integer.MIN_VALUE;
        while(head<len){
            tail=len-1; //开始改的时候这一句忘了加
            while(head!=tail){
                int minHight=height[tail]>height[head]?height[head]:height[tail];
                max=max>(tail-head)*minHight ? max:(tail-head)*minHight;
                if(height[head]<=height[tail]){
                    break;
                }else{
                    tail--;
                }
            }
            head++;
        }
        return max;
}
```

212ms，40%，利用双指针稍微优化了下，依然是遍历找每个柱的最大值，但是尾指针在移动时先判断下，如果比头指针大就直接break，因为**已经是最大值**了，tail是从右向左移动的

> 开始改的时候忘了将尾指针归位，结果还对了，而且90%的beats.....哈哈哈，误打误撞搞了个最优解出来。

**解法二**

上面两种其实都是暴力，时间复杂度都很高

```java
public int maxArea(int[] height) {
        int len=height.length;
        if(len==0){
            return 0;
        }
        int left=0,right=len-1;
        int max=Integer.MIN_VALUE;
        while(left<right) {
            int minHight=height[left]>height[right]?height[right]:height[left];
            max=max>(right-left)*minHight ? max:(right-left)*minHight;
            if(height[left]<=height[right]){
                left++;
            }else{
                right--;
            }
        }
        return max;
}
```

**标准**的最优解，这题主要考察的就是双指针，两个指针一头一尾，先算出这个头尾的面积大小，然后下一步思考怎么扩大这个区域的面积，结合题上面的图（最左边为头，最右边为尾）

![mark](http://static.imlgw.top///20190505/1uze479AHHLo.png?imageslim)

这个时候如果移动尾指针，明显面积只可能减小，所以只有移动头指针才有可能增大这个区域的面积，这样一来就可以省掉很多没必要的计算，有点像贪心，时间复杂度O(N)

## [15. 三数之和](https://leetcode-cn.com/problems/3sum/)

给定一个包含 *n* 个整数的数组 `nums`，判断 `nums` 中是否存在三个元素 *a，b，c ，*使得 *a + b + c =* 0 ？找出所有满足条件且不重复的三元组。

**注意：**答案中不可以包含重复的三元组。

```java
例如, 给定数组 nums = [-1, 0, 1, 2, -1, -4]，

满足要求的三元组集合为：
[
  [-1, 0, 1],
  [-1, -1, 2]
]
```

想太多了，没做出来，看了评论才做出来。

```java
public static List<List<Integer>> threeSum(int[] nums) {
    List<List<Integer>> list = new ArrayList();
    Arrays.sort(nums);
    // 先排序  o(nlogn)
    int len = nums.length;
    if(nums == null || len < 3) return list;
    // 完备性
    for (int i = 0; i < len-2; i++) {
        if(nums[i]>0){
            //大于0了，后面的和加起来肯定>0了
            break;
        }
        //遍历数组，相同的元素只需要遍历一遍，不然会重复
        if(i > 0 && nums[i] == nums[i-1]) continue;
        // 一次去重优化
        //当前元素的下一个元素。
        int L = i+1;
        //尾元素
        int R = len-1;
        while(L<R){
            int sum = nums[i] + nums[L] + nums[R];
            if(sum == 0){
                list.add(Arrays.asList(nums[i],nums[L],nums[R]));
                //-4 -1 -1 0 1 2
                while (L<R && nums[L] == nums[L+1]) L++;
                //二次去重优化
                while (L<R && nums[R] == nums[R-1]) R--;
                L++;
                R--;
            } else if (sum < 0){ //小于0所以要增大L,逼近0 else R--;
                 L++;   
            } else R--; //大于0就减小R
        }
    }
    return list;
}
```

代码思路就是遍历数组，然后从**i**位置后面的数组中找能和**i**凑成一对的元素，这里关键就是这里怎么找这两个元素 满足nums[L]+nums[R]=-nums[i]，问题就转化成了上面的**两数之和**，但是这里用暴力法肯定是过不了的，hashMap这里也不好用，所以这里我们可以先给数组排个序，然后利用**双指针对撞**，逐渐逼近0，还有一个很需要注意的地方就是二次去重，如下图

![mark](http://static.imlgw.top///20190505/5YlNbCLe57fb.png?imageslim)

当找到一组时有可能L，R的下一个位置的值没变这样就会导致重复。

## [16. 最接近的三数之和](https://leetcode-cn.com/problems/3sum-closest/)

给定一个包括 *n* 个整数的数组 `nums` 和 一个目标值 `target`。找出 `nums` 中的三个整数，使得它们的和与 `target` 最接近。返回这三个数的和。假定每组输入只存在唯一答案。

```java
例如，给定数组 nums = [-1，2，1，-4], 和 target = 1.

与 target 最接近的三个数的和为 2. (-1 + 2 + 1 = 2).
```

跟上面的题其实是一样的，这里主要时为了检测下自己上面的搞懂了没

```java
public int threeSumClosest(int[] nums, int target) {
    int len=nums.length;
    if(nums==null||len<3) return 0;
    Arrays.sort(nums);
    int closest=nums[0]+nums[1]+nums[2];
    for (int i=0;i<len-2;i++) {
        if(i!=0&&nums[i]==nums[i-1])continue;
        //跳过重复元素提高效率
        int L=i+1;
        int R=len-1;
        while(L<R){
            int sum=nums[L]+nums[R]+nums[i];
            closest=Math.abs(closest-target)>Math.abs(sum-target)?sum:closest;
            if(sum==target){
                return target;
            } else if(sum>target){
                while(L<R && nums[R]==nums[R-1])R--;
                R--;
            } else{
                while(L<R && nums[L]==nums[L+1])L++;
                L++;
            }
        }
    }
    return closest;
}
```

一遍**bugfree**，其实都挺简单，这两题我一直在考虑别的算法，我想的是排序后从两遍向中间然后...就不bb了，反之很多没考虑到的地方。

## [18. 四数之和](https://leetcode-cn.com/problems/4sum/)

给定一个包含 n 个整数的数组 `nums` 和一个目标值 `target`，判断 `nums` 中是否存在四个元素 a，b，c 和 d ，使得 `a + b + c + d` 的值与 `target` 相等？找出所有满足条件且不重复的四元组。

**注意：**

答案中不可以包含重复的四元组。

**示例：**

```java
给定数组 nums = [1, 0, -1, 0, -2, 2]，和 target = 0。

满足要求的四元组集合为：
[
  [-1,  0, 0, 1],
  [-2, -1, 1, 2],
  [-2,  0, 0, 2]
]
```

**解法一**

和三数之和一样，但是更加繁琐了，提交了5，6次才AC，还是看了别人的代码的

```java
public static List<List<Integer>> fourSum(int[] nums, int target) {
    List<List<Integer>> res=new ArrayList<>();
    Arrays.sort(nums);
    int n=nums.length;
    //0 0 -1 1
    for (int i=0;i<n-3;i++) {
        //这里我开始写的是和后一个比较，0，0，0，0这种过不了
        if(i>0 && nums[i]==nums[i-1])continue;
        if (nums[i]+nums[i+1]+nums[i+2]+nums[i+3]>target) break;
        if (nums[i]+nums[n-1]+nums[n-2]+nums[n-3]<target) continue;
        for (int j=i+1;j<n-2;j++) {
            //同上
            if(j>i+1&&nums[j]==nums[j-1])continue;
            if (nums[i]+nums[j]+nums[j+2]+nums[j+1]>target) break;
            if (nums[i]+nums[j]+nums[n-2]+nums[n-1]<target) continue;
            int two=nums[i]+nums[j];
            //左右边界
            int left=j+1,right=n-1;
            while(left<right){
                if (target-two==nums[left]+nums[right]) {
                    res.add(Arrays.asList(nums[i],nums[j],nums[left],nums[right]));
                    //想清楚什么时候跳,放外面就错了
                    while(left<right && nums[left]==nums[left+1]){left++;};
                    while(left<right && nums[right]==nums[right-1]){right--;};
                    left++;
                    right--;
                }else if (target-two>nums[left]+nums[right]) {
                    left++;
                }else{
                    right--;
                }
            }
        }
    }
    return res;
}
```


## [26. 删除排序数组中的重复项](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/)

给定一个排序数组，你需要在**原地**删除重复出现的元素，使得每个元素只出现一次，返回移除后数组的新长度。

不要使用额外的数组空间，你必须在**原地修改输入数组**并在使用 O(1) 额外空间的条件下完成。

**示例 1:**

```java
给定数组 nums = [1,1,2], 

函数应该返回新的长度 2, 并且原数组 nums 的前两个元素被修改为 1, 2。 

你不需要考虑数组中超出新长度后面的元素。
```

**示例 2:**

```java
给定 nums = [0,0,1,1,1,2,2,3,3,4],

函数应该返回新的长度 5, 并且原数组 nums 的前五个元素被修改为 0, 1, 2, 3, 4。

你不需要考虑数组中超出新长度后面的元素。
```

**解法一**

实不相瞒，这题一开始我暴力做的，冒泡的思想，太蠢了😅 ，注意题目要求空间复杂度O(1)

```java
public int removeDuplicates(int[] nums) {
    if (nums.length == 0) return 0;
    int i = 0;
    for (int j = 1; j < nums.length; j++) {
        if (nums[j] != nums[i]) {
            nums[i++] = nums[j];
        }
    }
    return i + 1;
}
```

双指针，真的用的挺多的。

## [80. 删除排序数组中的重复项 II](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array-ii/)

给定一个排序数组，你需要在**原地**删除重复出现的元素，使得每个元素最多出现两次，返回移除后数组的新长度。

不要使用额外的数组空间，你必须在**原地修改输入数组**并在使用 O(1) 额外空间的条件下完成。

**示例 1:**

```java
给定 nums = [1,1,1,2,2,3],

函数应返回新长度 length = 5, 并且原数组的前五个元素被修改为 1, 1, 2, 2, 3 。

你不需要考虑数组中超出新长度后面的元素。
```

**示例 2:**

```java
给定 nums = [0,0,1,1,1,1,2,3,3],

函数应返回新长度 length = 7, 并且原数组的前五个元素被修改为 0, 0, 1, 1, 2, 3, 3 。

你不需要考虑数组中超出新长度后面的元素。
```

**说明:**

为什么返回数值是整数，但输出的答案是数组呢?

请注意，输入数组是以**“引用”**方式传递的，这意味着在函数里修改输入数组对于调用者是可见的。

你可以想象内部操作如下:

```java
// nums 是以“引用”方式传递的。也就是说，不对实参做任何拷贝
int len = removeDuplicates(nums);

// 在函数里修改输入数组对于调用者是可见的。
// 根据你的函数返回的长度, 它会打印出数组中该长度范围内的所有元素。
for (int i = 0; i < len; i++) {
    print(nums[i]);
}
```

上面题目加一点，在前后相等的时候判断index前是否已经有两个相等

```java
public int removeDuplicates(int[] nums) {
    int index=2;
    for (int i=2;i<nums.length;i++){
        if(nums[i]!=nums[i-1] || (nums[i]==nums[i-1] && nums[index-2]!=nums[index-1])){
            nums[index++]=nums[i];
        }
    }
    return index;
}
```

## [27. 移除元素](https://leetcode-cn.com/problems/remove-element/)

给定一个数组 *nums* 和一个值 *val*，你需要**原地**移除所有数值等于 *val* 的元素，返回移除后数组的新长度。

不要使用额外的数组空间，你必须在**原地修改输入数组**并在使用 O(1) 额外空间的条件下完成。

元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。

**示例 1:**

```java
给定 nums = [3,2,2,3], val = 3,

函数应该返回新的长度 2, 并且 nums 中的前两个元素均为 2。

你不需要考虑数组中超出新长度后面的元素。
```

**示例 2:**

```java
给定 nums = [0,1,2,2,3,0,4,2], val = 2,

函数应该返回新的长度 5, 并且 nums 中的前五个元素为 0, 1, 3, 0, 4。

注意这五个元素可为任意顺序。

你不需要考虑数组中超出新长度后面的元素。
```

目标元素多时

```java
public int removeElement(int[] nums, int val) {
    int i = 0;
    for (int j = 0; j < nums.length; j++) {
        if (nums[j] != val) {
            nums[i++] = nums[j];
        }
    }
    return i;
}
```

目标元素少时

```java
public int removeElement2(int[] nums, int val) {
    int i = 0;
    int n = nums.length;
    while (i < n) {
        if (nums[i] == val) {
            nums[i] = nums[--n];
        } else {
            i++;
        }
    }
    return n;
}
```

## [283. 移动零](https://leetcode-cn.com/problems/move-zeroes/)

给定一个数组 `nums`，编写一个函数将所有 `0` 移动到数组的末尾，同时保持非零元素的相对顺序。

**示例:**

```java
输入: [0,1,0,3,12]
输出: [1,3,12,0,0]
```

**说明**:

- 必须在原数组上操作，不能拷贝额外的数组。

- 尽量减少操作次数。

**解法一**

```java
public void moveZeroes(int[] nums) {
        if(nums==null||nums.length<=1){
            return;
        }
        int index=0;
        for(int i=0;i<nums.length;i++){
            if(nums[i]!=0){
                nums[index++]=nums[i];
            }
        }
        for(int i=index;i<nums.length;i++){
            nums[i]=0;
        }
}
```

其实就是借助上面题目的思路，最后再补0就ok了，其实也还可以优化下

```java
public void moveZeroes(int[] nums) {
    int m=0; //[0,k)为非0元素
    for(int i=0;i<nums.length;i++){
        if(nums[i]!=0){
            if(i!=m){
                int temp=nums[i];
                nums[i]=nums[m];
                nums[m]=temp;   
            }
            m++;
        }
    }
}
```

## [31. 下一个排列](https://leetcode-cn.com/problems/next-permutation/)

实现获取下一个排列的函数，算法需要将给定数字序列重新排列成字典序中下一个更大的排列。

如果不存在下一个更大的排列，则将数字重新排列成最小的排列（即升序排列）。

必须**原地**修改，只允许使用额外常数空间。

以下是一些例子，输入位于左侧列，其相应输出位于右侧列。

```java
1,2,3 → 1,3,2
3,2,1 → 1,2,3
1,1,5 → 1,5,1
```

直接上最优解吧，这题暴力法O(N!)，空间也超过了

```java
public void nextPermutation(int[] nums) {
    int len=nums.length;
    if(nums==null||len<=1){
        return;
    }
    for (int i=len-2;i>=0;i--) {
        while(i>=0 &&nums[i]>=nums[i+1]){
            //找到第一个峰值左相邻的元素（从左到右）
            i--;
        }
        //逆序的, 没有最大值
        if(i==-1){
            reverse(nums,0);
            return;
        }
        //找到峰值右边 [i+1 , len-1] 最后一个比i 大的元素
        for (int j=len-1;j>i;j--) {
            if(nums[j]>nums[i]){
                swap(nums,j,i);
                reverse(nums,i+1);
                return;
            }
        }
    }
}
//翻转数组
private void reverse(int[] nums, int start) {
    for (int i=start,j=nums.length-1;i<j;i++,j--) {
        swap(nums,i,j);
    }
}
private  static void swap(int[] nums, int i, int j) {
    int temp = nums[i];
    nums[i] = nums[j];
    nums[j] = temp;
}
```

- 第一步，逆序找到第一个峰值的左边第一个元素 `a[i-1]`。

- 将峰值右边的**最小的**比`a[i-1]`大的`a[j]`(其实就是`右边最后一个比它大的元素`)元素与**a[i-1]**交换。
- 翻转刚刚调整过`a[i-1]`后面的逆序的数组(`a[i]-->a[len-1]`)。

![mark](http://static.imlgw.top/blog/20190728/G6uqlPyjPLdV.png?imageslim)

至于为什么这样做自己模拟下就懂了，逆序部分是没有下一个比它大的排列的，所以如果想让整个排列变大只能从这个逆序的排列里面选一个比逆序前最后一个''稍微''大一点的元素与之交换，然后将整个逆序的部分翻转就是下一个排列，这题看了题解后处理边界又处理了半天，**循环里面的循环边界条件一定要注意**

## [556. 下一个更大元素 III](https://leetcode-cn.com/problems/next-greater-element-iii/)

给定一个32位正整数 n，你需要找到最小的32位整数，其与 n 中存在的位数完全相同，并且其值大于n。如果不存在这样的32位整数，则返回-1。

**示例 1:**

```java
输入: 12
输出: 21
```

**示例 2:**

```java
输入: 21
输出: -1
```

**解法一**

和上面那一题一样，权当复习了一下

```java
public int nextGreaterElement(int n) {
    StringBuilder sb=new StringBuilder();
    while(n/10>0){
        sb.append(n%10);
        n/=10;
    }
    sb.append(n);
    System.out.println(sb);
    char[] nums=sb.reverse().toString().toCharArray();
    int len=nums.length;
    for (int i=len-1;i>0;i--) {
        if (nums[i]>nums[i-1]) { //逆序的峰值i
            if (i==0) return -1; 
            for (int j=len-1;j>=i;j--) {
                if (nums[j]>nums[i-1]) {
                    swap(nums,j,i-1);
                    reverse(nums,i,len-1);
                    return Long.valueOf(new String(nums))>Integer.MAX_VALUE?-1:Integer.valueOf(new String(nums));
                }
            }
        }
    }
    return -1;
}

public void reverse(char[] nums,int begin,int end){
    for (int i=begin,j=end;i<j;i++,j--) {
        swap(nums,i,j);
    }
}

public void swap(char[] nums,int a,int b){
    char temp=nums[a];
    nums[a]=nums[b];
    nums[b]=temp;
}
```
## [136. 只出现一次的数字](https://leetcode-cn.com/problems/single-number/)

给定一个**非空整数**数组，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。

**说明：**

你的算法应该具有线性时间复杂度。 你可以不使用额外空间来实现吗？

**示例 1:**

```java
输入: [2,2,1]
输出: 1
```


**示例 2:**

```java
输入: [4,1,2,1,2]
输出: 4
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

## [55. 跳跃游戏](https://leetcode-cn.com/problems/jump-game/)

给定一个非负整数数组，你最初位于数组的第一个位置。

数组中的每个元素代表你在该位置可以跳跃的最大长度。

判断你是否能够到达最后一个位置。

**示例 1:**

```java
输入: [2,3,1,1,4]
输出: true
解释: 我们可以先跳 1 步，从位置 0 到达 位置 1, 然后再从位置 1 跳 3 步到达最后一个位置。
```

**示例 2:**

```java
输入: [3,2,1,0,4]
输出: false
解释: 无论怎样，你总会到达索引为 3 的位置。但该位置的最大跳跃长度是 0 ， 所以你永远不可能到达最后一个位置。
```

**解法一**

回溯，勉强能过。。。太蠢了，为啥想不到简单的方法，就非得往复杂了想？就这么傻么？

```java
Boolean[] cache=null;

public boolean canJump(int[] nums) {
    if (nums==null || nums.length<=0) return false;
    cache=new Boolean[nums.length];
    return jump(nums,0);
}

public boolean jump(int[] nums,int index) {
    if (nums[index] >= nums.length-1 -index) {
        return true;
    }
    if (cache[index]!=null) {
        return cache[index];
    }
    for (int i=nums[index];i>=1;i--) {
        if (index+i<nums.length && jump(nums,index+i)) {
            return cache[index]=true;
        }
    }
    return cache[index]=false;
}
```
**解法二**

不用多说了，遍历数组，不断更新能到达的最远距离，如果**某个位置的index大于当前能到达的最远距离就直接返回false**

```java
//MDZZ
public boolean canJump(int[] nums) {
    int maxIndex=nums[0];
    for (int i=1;i<nums.length-1;i++) {
        if(maxIndex >= nums.length-1) return true;
        if (i>maxIndex) {
            return false;
        }
        maxIndex=Math.max(maxIndex,i+nums[i]);
    }
    return true;
}
```
## [169. 多数元素](https://leetcode-cn.com/problems/majority-element/)

给定一个大小为 n 的数组，找到其中的多数元素。多数元素是指在数组中出现次数大于 ⌊ n/2 ⌋ 的元素。

你可以假设数组是非空的，并且给定的数组总是存在多数元素。

**示例 1:**

```java
输入: [3,2,3]
输出: 3
```


**示例 2:**

```java
输入: [2,2,1,1,1,2,2]
输出: 2
```

**解法一**

分治法， (`HashMap`或者排序什么的方法就不说了，笔试可以那样写，面试就不能这样了)

```java
public int majorityElement(int[] nums) {
    return majorityElement(nums,0,nums.length-1);
}

public int majorityElement(int[] nums,int lo,int hi) {
    if (lo==hi) {
        return nums[lo];
    }
    int mid=lo+(hi-lo)/2;
    int leftMode=majorityElement(nums,lo,mid);
    int rightMode=majorityElement(nums,mid+1,hi);
    if (leftMode==rightMode) {
        return rightMode;
    }
    return countMode(nums,lo,mid,leftMode)>countMode(nums,mid+1,hi,rightMode)?leftMode:rightMode;
}

public int countMode(int[] nums,int left,int right,int mode){
    int count=0;
    for (int i=left;i<=right;i++) {
        if (mode==nums[i]) {
            count++;
        }
    }
    return count;
}
```

并不是最优解，时间复杂度`O(NlogN)`，只是一种思路吧

**解法二**

摩尔投票法

```java
public int majorityElement(int[] nums) {
    int sum=1;
    int res=nums[0]; 
    for (int i=1;i<nums.length;i++) {
        if (sum==0) {
            res=nums[i];
        }
        //将众数看做1,其他的看作-1,最后和一定是大于0的
        if (res!=nums[i]) {
            sum--;
        }else{
            sum++;
        }
    }
    return res;
}
```
## [229. 求众数 II](https://leetcode-cn.com/problems/majority-element-ii/)

给定一个大小为 n 的数组，找出其中所有出现超过 ⌊ n/3 ⌋ 次的元素。

**说明:** 要求算法的时间复杂度为 O(n)，空间复杂度为 O(1)。

**示例 1:**

```java
输入: [3,2,3]
输出: [3]
```


**示例 2:**

```java
输入: [1,1,1,3,3,2,2,2]
输出: [1,2]
```

**解法一**

和上面的方法一样，抵消去除三个不同的元素对众数没有任何影响

```java
public static List<Integer> majorityElement(int[] nums) {
    List<Integer> res = new ArrayList<>();
    if (nums == null || nums.length == 0)
        return res;
    int candidateA = nums[0];
    int candidateB = nums[0];
    int countA = 0;
    int countB = 0;

    for (int num : nums) {
        if (num == candidateA) {
            countA++;//投A
            continue;
        }
        if (num == candidateB) {
            countB++;//投B
            continue;
        }
        if (countA == 0) {
            candidateA = num;
            countA++;
            continue;
        }
        if (countB == 0) {
            candidateB = num;
            countB++;
            continue;
        }
        countA--;
        countB--;
    }
    countA = 0;
    countB = 0;
    for (int num : nums) {
        if (num == candidateA)
            countA++;
        else if (num == candidateB)
            countB++;
    }
    if (countA > nums.length / 3)
        res.add(candidateA);
    if (countB > nums.length / 3)
        res.add(candidateB);
    return res;
}
```

## [41. 缺失的第一个正数](https://leetcode-cn.com/problems/first-missing-positive/)

给定一个未排序的整数数组，找出其中没有出现的最小的正整数。

**示例 1:**

```java
输入: [1,2,0]
输出: 3
```

**示例 2:**

```java
输入: [3,4,-1,1]
输出: 2
```

**示例 3:**

```java
输入: [7,8,9,11,12]
输出: 1
```

**解法一**

Head题，想到了桶排序，但是空间不符合要求，看了评论扣了半天边界也没抠出来

```java
public int firstMissingPositive(int[] nums) {
    if(nums==null||nums.length<=0){
        return 1;
    }
    for (int i=0;i<nums.length;++i){
        //将每个元素归位，我开始只有一层循环，那样会漏掉很多元素（可能被交换的元素 后面也需要交换），这样的就是一次直接到位。
        while(nums[i]>=1&&nums[i]<=nums.length&&nums[nums[i]-1]!=nums[i])
        {
            int temp=nums[nums[i]-1];
            nums[nums[i]-1]=nums[i];
            nums[i]=temp;
        }
    }
    for (int i=0;i<nums.length;++i){
        if(nums[i]!=i+1)
            return i+1;
    }
    return nums.length+1;
}
```

不考虑空间复杂度利用桶排序的思想

```java
public int firstMissingPositive2(int[] nums) {
        if(nums==null||nums.length<=0){
            return 1;
        }
        int [] bucket=new int[nums.length];
        for(int i=0;i<nums.length;++i){
            if(nums[i]>0 && nums[i]<=nums.length){
                bucket[nums[i]-1]=1; //代表这个桶有元素了
            }
        }
        for(int i=0;i<bucket.length;++i){
            if(bucket[i]==0)
                return i+1;
        }
        return nums.length+1;
}
```

lc上提交后的空间消耗居然比上面的还小一点😂

## [42. 接雨水](https://leetcode-cn.com/problems/trapping-rain-water/)

给定 *n* 个非负整数表示每个宽度为 1 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

![rainwatertrap.png](https://i.loli.net/2019/05/14/5cda71129045d93180.png)




上面是由数组 [0,1,0,2,1,0,1,3,2,1,2,1] 表示的高度图，在这种情况下，可以接 6 个单位的雨水（蓝色部分表示雨水）。

**示例:**

```java
输入: [0,1,0,2,1,0,1,3,2,1,2,1]
输出: 6
```

**解法一**

这种解法的关键就是每个柱子能接的水是**左右最长柱子(都大于当前柱子)中的较小的那个减去当前柱子**。

```java
//双指针
public static int trap3(int []height){
    if (height == null || height.length <= 1) return 0;
    int left = 0;
    int right = height.length - 1;
    int left_max = 0;
    int right_max = 0;
    int res = 0;
    while (left < right) {
        if (height[left] < height[right]) {
            if (height[left] < left_max) //当前左边元素 比右边最大值小，比左边最大值小是个“凹点”
                res += left_max - height[left]; 
            else left_max = height[left]; //更新左边柱子最大值
            left++;
        } else {
            if (height[right] < right_max) //当前左边元素 比左边最大值小，比右边最大值小是个“凹点”
                res += right_max - height[right]; 
            else right_max = height[right]; //更新右边柱子最大值
            right--;
        }
    }
    return res; 
}
```

> 我最开始思路是填满后用总面积减数组和，跑过了130+个，有一种特殊的跑不过了，懒得去处理那个边界了，不太优雅

**解法二**

还有一种很巧妙的方法，也比较好理解，找到最大值，然后分别对两边的柱子进行遍历，如果当前的柱子小于前面柱子的最大值，就说明一定可以接到水，这个过程中需要记录柱子左边的最大值，用于计算可以接水的量，最后计算总和

```java
public static int trap5(int []height){
    //
    int n=height.length,idx=0,lefth=0,righth=0,area=0;
    for (int i=0;i<n;i++) idx=height[idx]<=height[i]?i:idx;
    for (int i=0;i<idx;i++){
        if(height[i]<lefth) area+=lefth-height[i]; 
        else lefth=height[i]; //更新最大值
    }
    for (int i=n-1;i>idx;i--){
        if(height[i]<righth) area+=righth-height[i]; 
        else righth=height[i]; //更新最大值
    }
    return area;
}
```

**解法三**

利用栈的

```java
public static int trap6(int[] height) {
        if (height == null || height.length == 0) return 0;
        Deque<Integer> stack = new ArrayDeque<>(); //栈里面维护一个递减序列
        int res = 0;
        for (int i = 0; i < height.length; i++){
            while ( ! stack.isEmpty() && height[stack.peek()] < height[i]) { //当遍历的元素大于栈顶元素
                int tmp = stack.pop(); //栈顶弹出来
                if (stack.isEmpty()) break;
                res += (Math.min(height[i],height[stack.peek()]) - height[tmp]) * (i - stack.peek() - 1);
            }
            //维护递减序列
            stack.push(i);
        }
        return res;
}
```

这种有点不好理解，其实是按照层来计算的，栈里面是递减的元素，如果读到比栈顶大的元素就**按层**计算递减栈**底部元素**到**当前元素**能蓄水的面积。

## [75. 颜色分类](https://leetcode-cn.com/problems/sort-colors/)

给定一个包含红色、白色和蓝色，一共 *n* 个元素的数组，**原地**对它们进行排序，使得相同颜色的元素相邻，并按照红色、白色、蓝色顺序排列。

此题中，我们使用整数 0、 1 和 2 分别表示红色、白色和蓝色。

**注意:**
不能使用代码库中的排序函数来解决这道题。

**示例:**

```java
输入: [2,0,2,1,1,0]
输出: [0,0,1,1,2,2]
```

**进阶：**

- 一个直观的解决方案是使用计数排序的两趟扫描算法。
  首先，迭代计算出0、1 和 2 元素的个数，然后按照0、1、2的排序，重写当前数组。
- 你能想出一个仅使用常数空间的一趟扫描算法吗？

题目上已经有了提示，很直观的做法就是利用桶排序的方法

```java
public static void sortColors(int[] nums) {
    int [] bucket=new int[3];
    //基于桶排序
    for (int i=0;i<nums.length;i++){
        bucket[nums[i]]++;
    }
    int index=0;
    //重新构造出来
    for (int i=0;i<nums.length;i++) {
        while (bucket[index]<=0) {
            index++;
        }
        nums[i]=index;
        bucket[index]--;
    }
}
```

当然还有更优秀的做法，利用**三向切分快排**的思想(荷兰国旗问题)

```java
public static void sortColors(int[] nums) {
    int less=-1,more=nums.length-1;
    int l=0;
    while(l<=more){
        if(nums[l]<1){
            swap(nums,++less,l++);
        } else if(nums[l]>1){
            swap(nums,more--,l);
        } else{ 
            l++;
        }
    }
}

public static swap(int []nums,int a,int b){
    int temp=nums[a];
    nums[a]=nums[b];
    nums[b]=temp;
}
```

## [125. 验证回文串](https://leetcode-cn.com/problems/valid-palindrome/)

给定一个字符串，验证它是否是回文串，只考虑字母和数字字符，可以忽略字母的大小写。

**说明：**本题中，我们将空字符串定义为有效的回文串。

**示例 1:**

```java
输入: "A man, a plan, a canal: Panama"
输出: true
```

**示例 2:**

```java
输入: "race a car"
输出: false
```

easy题，对撞指针

```java
public Boolean isPalindrome(String s) {
    if(s==null||s.length()<=1){
        return true;
    }
    s=s.toLowerCase();
    int left=0,right=s.length()-1;
    while(left<right){
        char lch=s.charAt(left);
        char rch=s.charAt(right);
        if(isNumOrchar(lch) && isNumOrchar(rch)){
            //System.out.println(lch+","+rch);
            if(lch==rch){
                left++;
                right--;
            } else{
                return false;
            }
        } else if((!isNumOrchar(lch)) && isNumOrchar(rch)){
            left++;
        } else if(isNumOrchar(lch) && !isNumOrchar(rch)){
            right--;
        } else{
            left++;
            right--;
        }
    }
    return true;
}

public Boolean isNumOrchar(char ch){
    if((ch>='0' && ch<='9') || (ch>='a' && ch<='z') || (ch>='A' &&  ch<='Z')){
        return true;
    }
    return false;
}
```

代码写多了，不够简洁，其实可以直接用**Character**的API

```java
public Boolean isPalindrome(String s) {
    if (s == null) return false;
    if (s.length() == 0) return true;
    int i = 0;
    int j = s.length() - 1;
    while (i < j) {
        while (i < j && !Character.isLetterOrDigit(s.charAt(i))) i++;
        while (i < j && !Character.isLetterOrDigit(s.charAt(j))) j--;
        if (Character.toLowerCase(s.charAt(i)) != Character.toLowerCase(s.charAt(j))){
            return false;
        }
        i++;
        j--;
    }
    return true;
}
```

## [215.数组中的第K个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)

Find the **k**th largest element in an unsorted array. Note that it is the kth largest element in the sorted order, not the kth distinct element.

**Example 1:**

```
Input: [3,2,1,5,6,4] and k = 2
Output: 5
```

**Example 2:**

```
Input: [3,2,3,1,2,4,5,5,6] and k = 4
Output: 4
```

**Note:** 
You may assume k is always valid, 1 ≤ k ≤ array's length.

> 这题必须多说几句

**解法一**

大根堆的做法（首先想到的方法，不是常规用堆的做法）

```java
public static int findKthLargest(int[] nums, int k) {
    //构建了大根堆
    for (int i=0;i<nums.length;i++){
        siftUp(nums,i);
    }
    int size=nums.length-1;
    for (int i=0;i<k-1;i++) {
        swap(nums,0,size);//和堆顶交换K次
        siftDown(nums,0,--size);//重新调整堆
    }
    return nums[0];
}

public static void siftUp(int[] nums,int i){
    while(nums[i]>nums[(i-1)/2]){
        swap(nums,i,(i-1)/2);
        i=(i-1)/2;
    }
}

//i 变小 下沉
public static void siftDown(int[] nums,int i,int size){
    //判断有没有子节点（左孩子）
    int left=i*2+1;
    while(left<size){
        int right=left+1;
        //左右节点最大值
        int larger=left+1<size && nums[left]<nums[left+1] ?left+1:left;
        if(nums[larger]>nums[i]){
            swap(nums,larger,i);
            i=larger;
            left=larger*2+1;
        } else{
            break;
        }
    }
}

public static  void  swap(int []nums,int a,int b){
    int temp=nums[a];
    nums[a]=nums[b];
    nums[b]=temp;
}
```

70%左右的beat，当时感觉还行，时间复杂度应该是`O(KlogN)`，后来越想越不对，又去看了下堆排序，发现我之前写的堆排序都是有问题的

**优化后的大根堆做法**

```java
public static int findKthLargest(int[] nums, int k) {
    int last=nums.length-1;
    for (int i=nums.length/2-1;i>=0;i--) {
        siftDown(nums,i,last);
    }
    for (int i=0;i<k-1;i++) {
        swap(nums,0,last);
        siftDown(nums,0,--last);
    }
    return nums[0];
}

//i 变小 下沉
public static void siftDown(int[] nums,int i,int last){
    //判断有没有子节点（左孩子）
    int left=i*2+1;
    while(left<=last){
        int right=left+1;
        //左右节点最大值
        int larger=right<=last && nums[right] > nums[left]?right:left;
        if(nums[larger]>nums[i]){
            swap(nums,larger,i);
            i=larger;
            left=larger*2+1;
        } else{
            break;
        }
    }
}

public static  void  swap(int []nums,int a,int b){
    int temp=nums[a];
    nums[a]=nums[b];
    nums[b]=temp;
}
```

95% beat，比上面的要快很多，相比之前的方法，构造堆的方式发生了变化，上面那种通过自上而下的insert方式时间复杂度是O(NlogN)，其实想想，这两种方式是完全相反的，insert的方式，最后一层每个元素最坏都可能调整`logN`次，而最后一层也是元素最多的一层，这样一来复杂度就会大大增加，相反如果采用从底向上的`swim`方式最后一层都只需要调整`1`次，而根节点需要调整`logN`次，而根节点只有一个，时间复杂度就会大大降低，最终的时间复杂度就是`O(N)`，[具体推算可以看这篇文章](https://www.zhihu.com/question/20729324)， 现在的时间复杂度才真的是`O(KlogN)`

![mark](http://static.imlgw.top/image/20190617/kVuvnMfjuSns.png?imageslim)

> 💥💥 上面这两种做法是有问题的，失去了用堆的优势，大根堆的做法必须要阿将整个堆构建完成后才能去找topk这样的话内存消耗比较大，应该维护一个小根堆，这样如果数据量很大的时候不用全读入内存中，  这题因为是我自己实现的堆，所以建堆的复杂度是O(N)（如果使用官方的API，建堆的时间复杂度就是NlogN），最终大根堆小根堆复杂度取决于K和N的大小关系，但是面试的时候最好不要说用大根堆的做法

**解法二**

小根堆的做法

```java
public int findKthLargest(int[] nums, int k) {
    int size=nums.length;
    //先维护一个大小为k的小根堆 ,这里要注意k不是下标，k=index+1
    for (int i = k/2; i >=0; i--) {
        heapIfy(nums,i,k);
    }
	//再从k开始向里面插入元素
    for (int i=k;i<size;i++) {
        if(nums[i]>nums[0]) { //大于小根堆堆顶,进取代它
            //小根堆求第K大,保证这个堆的元素是整个堆的前k大的元素，堆顶就是第k大
            swap(nums,i,0);
            heapIfy(nums,0,k);
        }
        //小于堆顶就不用管了
    }
    return nums[0];
}

//小根堆调整
public  void heapIfy(int[] nums, int i, int size) {
    int left = 2 * i + 1;
    while (left < size) {
        int right = left + 1;
        int small = right < size && nums[right] < nums[left] ? right: left;
        if(nums[small]<nums[i]) {
            swap(nums,small,i);
            i=small;
            left=2*i+1;
        } else {
            return;
        }
    }
}

private  void swap(int[] nums, int l, int r) {
    int temp = nums[l];
    nums[l] = nums[r];
    nums[r] = temp;
}
```

2ms，99%beat，一般情况下的topK问题，如果用堆解决的话应该都是采用**小根堆**这种做法来做，时间复杂度为`O(NlogK)`，维护一个大小为k的小根堆，然后再遍历后面n-k个元素，依次和当前最小堆的堆顶比较（当前topK中的最小元素，堆顶），如果比它小就和它交换然后调整堆，这样就始终保持了这个堆是当前的topK小，最后的堆顶就是第K大的元素。

**解法三**

其实还有一类做法，利用`快排+二分`的思想

```java
public static int findKthLargest(int[] nums, int k) {
    int n=nums.length;
    int left=0,right=nums.length-1;
    while(left<=right){
        //分治
        int base=partion(nums,left,right); //拿到划分点
        if(base<n-k){
            left=base+1;
        } else if(base>n-k){
            right=base-1;
        } else{
            return nums[base];
        }
    }
    return -1;
}

public static int partion(int []nums,int left,int right){
    //随机取值
    swap(nums,left,left+(int) (Math.random() * (right - left + 1)));
    int base=left;
    while(left<right){
        while(left<right&&nums[right]>nums[base]){
            right--;
        }
        while(left<right&&nums[left]<=nums[base]){
            left++;
        }
        if(left<right){
            swap(nums,left,right);
        }
    }
    //归位
    swap(nums,left,base);
    return left;
}

public static  void  swap(int []nums,int a,int b){
    int temp=nums[a];
    nums[a]=nums[b];
    nums[b]=temp;
}
```

这里最好用**随机**的**partition**，我试了下**不随机**大概`50+ms 30%beat`，这种随机的大概`3ms 97%beats `，差距还是很大的，时间复杂度是**O(N)**

> 至于为什么是O(N)，我们可以来分析下，这里**假设每次划分都是差不多中点的位置**，如果是快排，那么在**partition**之后依然需要两边的子数组进行**partition**，分治整个递归栈的高度就是`logN`，每层都是N，所以整体的复杂度就**O(NlogN)**....扯远了，回到正题
>
> 来说说我们这里为什么是O(N)，这里我们沿用前面的分析过程，递归栈深度依然是`logN`，但是我们在这里第一次确定划分点的相对**k**的位置后，下一步**只需要划分其中一边的元素，不用对另一边的元素继续**，也就是n/2，再往下就是n/4，n/8，n/16 ....   而 `(1+1/2+1/4+1/8+......1/2^n)n <=2n` ，也就是说整体的复杂度是低于O(2N)的，所以这里复杂度就是O(N)

**解法四**

[BFPRT算法](https://zhuanlan.zhihu.com/p/31498036) 大佬们提出来的根据上面快排改进而来，其实面试把小根堆和快排的解法答出来应该就差不多了，这个解法还是有些不容易写出来

```java
public static int findKthLargest(int []nums,int k){
    return findKthLargest(nums,0,nums.length-1,k);
}

public static int findKthLargest(int[] nums,int l,int r,int k) {
    int mid=findMid(nums,l,r);
    swap(nums,mid,l);
    int m=partition(nums,l,r);
    if(m==nums.length-k){
        return nums[m];
    }
    //下面的类似了
    if(m>nums.length-k){
        return findKthLargest(nums,l,m-1,k);
    }
    return findKthLargest(nums,m+1,r,k);
}

//中位数的中位数，主要的核心就是在这里
public static int  findMid(int []nums,int l,int r){
    int leftSub=l;
    //分组求中位数，5等分
    for (int i=l;i<r-4;i+=5) {
        insertSort(nums,i,i+4);
        //将每一组的中位数统一放到左侧，用于递归
        swap(nums,leftSub++,i+2);
    }
    //处理剩下的不足5个的
    if (r-l<4) {
        insertSort(nums,l,r);
        swap(nums,leftSub,l+(r-l)/2);
    }
    //找到了
    if(l==leftSub){
        return l;
    }
    return findMid(nums,l,leftSub);
}

//五等分的插入
public static void insertSort(int []nums,int l,int r){
    for (int i=0;i<r;i++) {
        for (int j=i+1;j>=l&&nums[j]<nums[i];j--) {
            swap(nums,j,i);
        }
    }
}

//快排partition
public static int partition(int []nums,int left,int right){
    int base=left;
    while(left<right){
        while(left<right&&nums[right]>nums[base]){
            right--;
        }
        while(left<right&&nums[left]<=nums[base]){
            left++;
        }
        if(left<right){
            swap(nums,left,right);
        }
    }
    //归位
    swap(nums,left,base);
    return left;
}

public static void swap(int []nums,int a,int b){
    int temp=nums[a];
    nums[a]=nums[b];
    nums[b]=temp;
}
```

[具体的时间复杂度证明](https://zhuanlan.zhihu.com/p/31498036)，当n取5时候，在划分的时候**至少**会大于**3n/10**的元素，避免了极端情况，保证在最坏情况下也不会太坏。

![mark](http://static.imlgw.top/image/20190617/Lc4M5f2qkegH.png?imageslim)

如上图，每一列为分好的一组元素，中间黄色部分为每组的中位数，红色块为**中位数的中位数**，这个中位数至少会大于等于左上角黑框框住的部分，所以在划分的时候会保证至少减小大约3n/10 的规模。

所以时间复杂度   `T(N)<=T(n/5)+T( 7n/10)+c*n`  总体时间复杂度**O(N)**，至于为什么不用其他的元素可以看看上面的那篇文章。

## [345. 反转字符串中的元音字母](https://leetcode-cn.com/problems/reverse-vowels-of-a-string/)

Write a function that takes a string as input and reverse only the vowels of a string.

**Example 1:**

```
Input: "hello"
Output: "holle"
```

**Example 2:**

```
Input: "leetcode"
Output: "leotcede"
```

**Note:**
The vowels does not include the letter "y".

```java
public String reverseVowels(String s) {
    if(s==null||s.length()<=0){
        return s;
    }
    char[] ss=s.toCharArray();
    int left=0,right=s.length()-1;
    while(left<right){
        while(left<right && !isYy(ss[left])){
            left++;
        }
        while(left<right && !isYy(ss[right])){
            right--;
        }
        swap(left++,right--,ss);
    }
    return new String(ss);
}

public Boolean isYy(char ch){
    char temp=Character.toLowerCase(ch);
    return temp=='a'|| temp=='e'||temp=='i'||temp=='o'||temp=='u';
}

public void swap(int a,int b,char[] s){
    char temp=s[a];
    s[a]=s[b];
    s[b]=temp;
}
```

很简单的对撞指针题

## [88. 合并两个有序数组](https://leetcode-cn.com/problems/merge-sorted-array/)

Given two sorted integer arrays *nums1* and *nums2*, merge *nums2* into *nums1* as one sorted array.

**Note:**

- The number of elements initialized in *nums1* and *nums2* are *m* and *n* respectively.
- You may assume that *nums1* has enough space (size that is greater or equal to *m* + *n*) to hold additional elements from *nums2*.

**Example:**

```java
Input:
nums1 = [1,2,3,0,0,0], m = 3
nums2 = [2,5,6],       n = 3

Output: [1,2,2,3,5,6]
```

典型的二路归并

```java
public static void merge(int[] nums1, int m, int[] nums2, int n) {
    if(nums1.length<=0||nums2.length<=0){
        return;
    }
    int []res=new int[m+n];
    int i1=0,i2=0;
    for (int i=0;i1<m&&i2<n;i++) {
        if(nums1[i1]<=nums2[i2]) {
            res[i]=nums1[i1++];
        } else if(nums1[i1]>nums2[i2] ){
            res[i]=nums2[i2++];
        }
    }
    if(i1>=m){
        System.arraycopy(nums2,i2,res,i2+m,n-i2);
    } else{
        System.arraycopy(nums1,i1,res,i1+n,m-i1);
    }
    System.arraycopy(res,0,nums1,0,res.length);
}
```

1ms ，98%beats.

看了下评论区发现自己还是太年轻了，原来这题是可以在**O(1)**的空间复杂度下完成的

```java
public static void merge3(int[] nums1, int m, int[] nums2, int n) {
    if(nums1.length<=0||nums2.length<=0){
        return;
    }
    int i1=m-1,i2=n-1;
    for (int i=m+n-1;i>=0;i--) {
        if(i1<0){
            nums1[i]=nums2[i2--];
        } else if(i2<0){
            nums1[i]=nums1[i1--];
        } else if(nums1[i1]>nums2[i2]) {
            nums1[i]=nums1[i1--];
        } else if(nums1[i1]<=nums2[i2] ){
            nums1[i]=nums2[i2--];
        }
    }
}
```

合并后的长度确定，nums1的空间也足够，所以完全可以从后往前，从大到小，从而避免了使用额外的空间储存结果，学到了学到了👏



## [118. 杨辉三角](https://leetcode-cn.com/problems/pascals-triangle/)

Given a non-negative integer *numRows*, generate the first *numRows* of Pascal's triangle.

![img](https://upload.wikimedia.org/wikipedia/commons/0/0d/PascalTriangleAnimated2.gif)
In Pascal's triangle, each number is the sum of the two numbers directly above it.

**Example:**

```java
Input: 5
Output:
[
     [1],
    [1,1],
   [1,2,1],
  [1,3,3,1],
 [1,4,6,4,1]
]
```

递归专题里面的题目，所以直接用递归来实现了下。

```java
public static List<List<Integer>> generate(int numRows) {
    if(numRows<=0) {
        return new ArrayList();
    }
    List<List<Integer>> res = new ArrayList<>();
    res.add(new ArrayList<Integer>() {
        {
            add(1);
        }
    }
    );
    generate(1, res.get(0), res, numRows);
    return res;
}

public static void generate(int numRow, List<Integer> preRow, List<List<Integer>> res, int rowMax) {
    if (rowMax == numRow) {
        return;
    }
    List<Integer> row = new ArrayList<>();
    row.add(1);
    for (int i = 1; i < preRow.size(); i++) {
        row.add(preRow.get(i - 1) + preRow.get(i));
    }
    row.add(1);
    res.add(row);
    generate(numRow + 1,row,res,rowMax);
}
```

尾递归，很鸡肋。

## [119. 杨辉三角 II](https://leetcode-cn.com/problems/pascals-triangle-ii/)

Given a non-negative index *k* where *k* ≤ 33, return the *k*th index row of the Pascal's triangle.

Note that the row index starts from 0.

![img](https://upload.wikimedia.org/wikipedia/commons/0/0d/PascalTriangleAnimated2.gif)
In Pascal's triangle, each number is the sum of the two numbers directly above it.

**Example:**

```java
Input: 3
Output: [1,3,3,1]
```

**Follow up:**

Could you optimize your algorithm to use only *O*(*k*) extra space?

```java
 public List<Integer> getRow(int rowIndex) {
        List<Integer> res=new ArrayList<>();
        long cur=1;
        res.add((int)cur);
        for(int i=1;i<=rowIndex;i++){
            cur=cur*(rowIndex-i+1)/i;
            res.add((int)cur);
        }
        return res;
 }
```

直接利用组合数的公式，m列第n个元素等于C(n-1,M-1)

## [54. 螺旋矩阵](https://leetcode-cn.com/problems/spiral-matrix/)

给定一个包含 m x n 个元素的矩阵（m 行, n 列），请按照顺时针螺旋顺序，返回矩阵中的所有元素。

**示例 1:**

```java
输入:
[
 [ 1, 2, 3 ],
 [ 4, 5, 6 ],
 [ 7, 8, 9 ]
]
输出: [1,2,3,6,9,8,7,4,5]
```

**示例 2:**

```java
输入:
[
  [1, 2, 3, 4],
  [5, 6, 7, 8],
  [9,10,11,12]
]
输出: [1,2,3,4,8,12,11,10,9,5,6,7]
```

**解法一**

这题很久之前做过，这次又来做的时候还是没做出来，忘了之前咋做的了，用模拟的方法搞了半天，没搞出来，然后瞄了一眼之前写的才写出来....

```java
public static List<Integer> spiralOrder2(int[][] matrix) {
    List<Integer> res=new ArrayList<>();
    if(matrix.length<=0){
        return res;
    }
    //a: 行
    //b: 列
    int la=0,lb=0,ra=matrix.length-1,rb=matrix[0].length-1;
    //终止条件
    while(lb<=rb && la<=ra){
        //缓存各个坐标
        int tla=la,tlb=lb,tra=ra,trb=rb;
        //特殊情况，特殊处理
        if(tla==tra){//同一行
            while(tlb<=trb){
                res.add(matrix[tla][tlb++]);
            }
            return res;
        }else if(tlb==trb){//同一列
            while(tla<=tra){
                res.add(matrix[tla++][tlb]);
            }
            return res;
        }else{
            //向左
            while(tlb<rb){
                res.add(matrix[tla][tlb++]);
            }
			//向下
            while(tla<ra){
                res.add(matrix[tla++][tlb]);
            }
			//向右
            while(trb>lb){
                res.add(matrix[tra][trb--]);
            }
			//向上
            while(tra>la){
                res.add(matrix[tra--][trb]);
            }
        }
        //向内靠拢(缩圈)
        la++;
        lb++;
        ra--;
        rb--;
    }
    return res;
}
```

模拟的方式相对要复杂点，需要记录每个节点是否访问然后在选择，这里的方式就很巧妙，直接按层遍历，由外到内，不用考虑那么多。时间复杂度`O(NM)`空间复杂度`O(NM)`。

## [48. 旋转图像](https://leetcode-cn.com/problems/rotate-image/)

给定一个 *n* × *n* 的二维矩阵表示一个图像。

将图像顺时针旋转 90 度。

**说明：**

你必须在**原地**旋转图像，这意味着你需要直接修改输入的二维矩阵。**请不要**使用另一个矩阵来旋转图像。

**Example 1:**

```java
Given input matrix = 
[
  [1,2,3],
  [4,5,6],
  [7,8,9]
],

rotate the input matrix in-place such that it becomes:
[
  [7,4,1],
  [8,5,2],
  [9,6,3]
]
```

**Example 2:**

```java
Given input matrix =
[
  [ 5, 1, 9,11],
  [ 2, 4, 8,10],
  [13, 3, 6, 7],
  [15,14,12,16]
], 

rotate the input matrix in-place such that it becomes:
[
  [15,13, 2, 5],
  [14, 3, 4, 1],
  [12, 6, 8, 9],
  [16, 7,10,11]
]
```

**解法一**

这题和上面哪一题放在一起很有必要，很类似的题型

```java
public void rotate(int[][] matrix) {
    if (matrix==null || matrix.length==0) {
        return;
    }
    int len=matrix.length-1;
    int lx=0,ly=0,rx=len,ry=len;
    while(lx<=rx){
        //len=ry-ly;
        for (int i=0;i<len;i++) {
            int temp=matrix[lx][ly+i];
            matrix[lx][ly+i]=matrix[rx-i][ly];
            matrix[rx-i][ly]=matrix[rx][ry-i];
            matrix[rx][ry-i]=matrix[lx+i][ry];
            matrix[lx+i][ry]=temp;
        }
        //缩圈
        len-=2; //写ry-ly可能会好一点，无所谓
        lx++;ly++;
        rx--;ry--;
    }
}
```
和上一题一样，都是从整体出发，从外层到内层，考虑每一层的前`n-1`个节点的旋转过程，这个过程需要自己在纸上画一画，空想容易搞错

## [347. 前 K 个高频元素](https://leetcode-cn.com/problems/top-k-frequent-elements/)

给定一个非空的整数数组，返回其中出现频率前 k 高的元素。

**示例 1:**

```java
输入: nums = [1,1,1,2,2,3], k = 2
输出: [1,2]
```

**示例 2:**

```java
输入: nums = [1], k = 1
输出: [1]
```

**说明：**

```java
你可以假设给定的 k 总是合理的，且 1 ≤ k ≤ 数组中不相同的元素的个数。
你的算法的时间复杂度必须优于 O(n log n) , n 是数组的大小。
```

也是TopK问题，但是这题其实还有个条件，`不会给出有歧义的数据` ，举个例子

`nums=[1,1,1,2,2,2,3,3,3] ，k=2` 这样的就是有歧义的

但是题目中也没有规定这样的如何处理，经过测试，发现官方的解在遇到这种情况会抛一个异常。

**解法一**

大根堆的做法

```java
public static List<Integer> topKFrequent(int[] nums, int k) {
    if(nums==null||nums.length<=0){
        return null;
    }
    HashMap<Integer,Integer> fre=new HashMap<>();
    for (int i=0;i<nums.length;i++) {
        //fre.get(i) nums[i]出现的频次
        fre.put(nums[i],fre.getOrDefault(nums[i],0)+1);
    }
    //1:3,2:3,3:1
    PriorityQueue<HashMap.Entry<Integer,Integer>> pq=new PriorityQueue(new ComparatorMap());
    for (HashMap.Entry ent:fre.entrySet()) {
        pq.add(ent);
    }
    ArrayList<Integer> res=new ArrayList<>();
    for (int i=0;i<k;i++) {
        res.add(pq.poll().getKey());
    }
    return res;
}

//比较器
static class ComparatorMap implements Comparator<HashMap.Entry<Integer,Integer>>{
    @Override
    public int compare(Map.Entry<Integer, Integer> o1, Map.Entry<Integer, Integer> o2) {
        return o2.getValue()-o1.getValue();
    }
}
```
用大根堆不太好，容易爆内存，但是在这一题可以保证顺序，`但是题目并没有要求顺序`，时间复杂度~~O(KlogN)~~

这里错了，建堆的时间复杂度就是`O(NlogN)`了，只有自己手写的堆，采用自底向上的方式建堆时间复杂度才是O(N) ，可以参考 [之前的文章](http://imlgw.top/2018/12/11/chang-jian-pai-xu-suan-fa-zong-jie/#%E5%A0%86%E6%8E%92%E5%BA%8F%E6%9B%B4%E4%BC%98%E7%9A%84%E5%81%9A%E6%B3%95) ，这也是上面topK问题中提到的

**解法二**

小根堆的做法

```java
public static List<Integer> topKFrequent(int[] nums, int k) {
    if(nums==null||nums.length<=0){
        return null;
    }
    HashMap<Integer,Integer> fre=new HashMap<>();
    for (int i=0;i<nums.length;i++) {
        //fre.get(i) nums[i]出现的频次
        fre.put(nums[i],fre.getOrDefault(nums[i],0)+1);
    }
    //1:3,2:3,3:1
    //其实可以这样写，但是对比了一下lambda大概要60ms，而直接构造比较器只要20ms
    //PriorityQueue<HashMap.Entry<Integer,Integer>> pq=new PriorityQueue<>((o1,o2)->o1.getValue()-o2.getValue());
    PriorityQueue<HashMap.Entry<Integer,Integer>> pq=new PriorityQueue(new ComparatorMap());
    for (HashMap.Entry ent:fre.entrySet()) {
        pq.add(ent);
        if(pq.size()>k){
            pq.poll();
        }
    }
    ArrayList<Integer> res=new ArrayList<>();
    while (!pq.isEmpty()) {
        res.add(pq.poll().getKey());
    }
    return res;
}

static class ComparatorMap implements Comparator<HashMap.Entry<Integer,Integer>>{
    @Override
    public int compare(Map.Entry<Integer, Integer> o1, Map.Entry<Integer, Integer> o2) {
        return o1.getValue()-o2.getValue();
    }
}
```

时间复杂度`O(NlogK)`因为只维护了一个K大小的小根堆 ，时间复杂度和大根堆~~O(KlogN)~~ `O(NlogN)`相比会快很多，除此之外，如果N和K很接近的话可以考虑`O(Nlog(N-K))` 的做法，维护一个N-K的大根堆，里面存频率最低的那些元素，最后返回其他的元素（no code， just talk）

**解法三**

桶排序，这题的最优解应该就是桶排序

```java
public static List<Integer> topKFrequent(int[] nums, int k) {
    if(nums==null||nums.length<=0){
        return null;
    }
    HashMap<Integer,Integer> fre=new HashMap<>();
    for (int i=0;i<nums.length;i++) {
        //记录nums[i]出现的频次
        fre.put(nums[i],fre.getOrDefault(nums[i],0)+1);
    }
    ArrayList<Integer> [] bucket=new ArrayList[nums.length+1];
    for (Integer num:fre.keySet()) {
        if(bucket[fre.get(num)]==null){
            bucket[fre.get(num)]=new ArrayList<>();
        }
        //桶排序
        bucket[fre.get(num)].add(num); //所有出现fre.get(num)次的元素构成一条链表
    }
    ArrayList<Integer> res=new ArrayList<>();
    int topk=bucket.length-1;
    while (true) {
        //从后向前遍历（从频次大到小）
        //指针移动到合适的位置
        while(bucket[topk]==null&&topk>0){
            topk--;
        }
        res.addAll(bucket[topk--]);
        if(res.size()==k){
            return res;
        }
    }
}
```

桶排序的思路，时间复杂度`O(N)`，空间复杂度也是`O(N)`，在leetcode提交三种方法的差距不大，可能是数据量太少了

## [67. 二进制求和](https://leetcode-cn.com/problems/add-binary/) 

给定两个二进制字符串，返回他们的和（用二进制表示）。

输入为非空字符串且只包含数字 1 和 0。

**示例 1:**

```java
输入: a = "11", b = "1"
输出: "100"
```

**示例 2:**

```java
输入: a = "1010", b = "1011"
输出: "10101"
```

**解法一**

这题和下面的题目是我有意放在一起的，这题也可以作为大数相加的模板

```java
public String addBinary(String a, String b) {
    StringBuilder res=new StringBuilder(); 
    int idxA=a.length()-1;
    int idxB=b.length()-1;
    boolean carry=false;
    //int carry=0;
    while(idxA >=0 || idxB >=0){
        char bina=idxA>=0?a.charAt(idxA):'0';
        char binb=idxB>=0?b.charAt(idxB):'0';
        if(bina == '1' && binb =='1'){
            res.append(carry?1:0);
            carry=true;
        }else if((bina == '1' && binb =='0') ||(bina == '0' && binb =='1')){
            res.append(carry?0:1);
        }else{
            res.append(carry?1:0);
            carry=false;
        }
        idxA--;idxB--;
    }
    if(carry) res.append(1);
    return res.reverse().toString();
}
```
## [415. 字符串相加](https://leetcode-cn.com/problems/add-strings/)

给定两个字符串形式的非负整数 `num1` 和`num2` ，计算它们的和。

**注意：**

1. num1 和num2 的长度都小于 5100.
2. num1 和num2 都只包含数字 0-9.
3. num1 和num2 都不包含任何前导零。
4. 你不能使用任何內建 BigInteger 库， 也不能直接将输入的字符串转换为整数形式。

**解法一**

一开始没找到这题，后面偶然发现的，随手写一下

```java
public String addStrings(String num1, String num2) {
    StringBuilder sb=new StringBuilder();
    int m=num1.length()-1;
    int n=num2.length()-1;
    int carry=0;
    while(n>=0 || m>=0){
        int a= m>=0?num1.charAt(m)-48:0;
        int b= n>=0?num2.charAt(n)-48:0;
        int sum=a+b+carry;
        carry=sum/10;
        sb.append(sum%10);
        m--;n--;
    }
    if (carry==1) {
        sb.append("1");
    }
    return sb.reverse().toString();
}
```

## [43. 字符串相乘](https://leetcode-cn.com/problems/multiply-strings/)

给定两个以字符串形式表示的非负整数 num1 和 num2，返回 num1 和 num2 的乘积，它们的乘积也表示为字符串形式。

**示例 1:**

```java
输入: num1 = "2", num2 = "3"
输出: "6"
```

**示例 2:**

```java
输入: num1 = "123", num2 = "456"
输出: "56088"
```


**说明：**

1. num1 和 num2 的长度小于110。
2. num1 和 num2 只包含数字 0-9。
3. num1 和 num2 均不以零开头，除非是数字 0 本身。
4. 不能使用任何标准库的大数类型（比如 BigInteger）或直接将输入转换为整数来处理。

**解法一**

```java
public static String multiply2(String num1, String num2) {
    // 1 2 3 
    // 4 5 6
    // 501 6
    int n1=num1.length();
    int n2=num2.length();
    //n1*n2 结果最长为 n1+n2
    int[] res=new int[n1+n2];
    for (int i=n1-1;i>=0;i--) {
        for (int j=n2-1;j>=0;j--) {
            //主要就是对这个i+j+1的理解
            res[i+j+1]+=(num1.charAt(i)-48)*(num2.charAt(j)-48);
        }
    }
    //处理进位
    for(int i=res.length-1;i>=0;i--) {
        if(res[i]>=10){
            res[i-1]+=res[i]/10;
            res[i]%=10;
        }
    }
    //去掉前面多余的0
    int index=0;
    while (index<res.length-1&&res[index]==0) { 
        index++;
    }
    StringBuilder sb=new StringBuilder();
    for (int i=index;i<res.length;i++) {
        sb.append(res[i]);
    }
    return sb.toString();
}
```
其实就是模拟的手算的过程，关键的地方就是 `i+j+1` 的理解

![mark](http://static.imlgw.top/blog/20190928/4xnHi4yd4hwA.png?imageslim)

**解法二**

其实仔细分析，会发现上面的代码其实有很多多余的操作，比如去掉前面的0，因为两个**非0的数相乘**，最后的结果最多n1+n2位，最少n1+n2-1位，所以前面的0**最多就一个**

```java
public static String multiply(String num1, String num2) {
    if (num1.equals("0") || num2.equals("0")) {
        return "0";
    }
    int n1=num1.length();
    int n2=num2.length();
    int[] res=new int[n1+n2];
    for (int i=n1-1;i>=0;i--) {
        for (int j=n2-1;j>=0;j--) {
            //注意这里的i+j+1
            res[i+j+1]+=(num1.charAt(i)-48)*(num2.charAt(j)-48);
        }
    }
    //处理进位(其实这里res[0]是不可能大于10的)，模拟下知道了
    for(int i=res.length-1;i>=0;i--) {
        if(res[i]>=10){
            res[i-1]+=res[i]/10;
            res[i]%=10;
        }
    }
    StringBuilder sb=new StringBuilder();
    for (int i=0;i<res.length;i++) {
        //前面最多只有一个0(除了两个数中有一个为0的时候)
        if (i==0 && res[i]==0) continue;
        sb.append(res[i]);
    }
    return sb.toString();
}
```
**解法三**

其实上面的进位和计算对应位置的值可以同时处理，这是最接近人手算的思路了

```java
public static String multiply3(String num1, String num2) {
    if (num1.equals("0") || num2.equals("0")) {
        return "0";
    }
    int n1=num1.length();
    int n2=num2.length();
    int[] res=new int[n1+n2];
    for (int i=n1-1;i>=0;i--) {
        for (int j=n2-1;j>=0;j--) {
            int sum=res[i+j+1]+(num1.charAt(i)-48)*(num2.charAt(j)-48);
            res[i+j+1]=sum%10;
            res[i+j]+=sum/10;
        }
    }
    StringBuilder sb=new StringBuilder();
    for (int i=0;i<res.length;i++) {
        //前面最多只有一个0(除了两个数中有一个为0的时候)
        if (i==0 && res[i]==0) continue;
        sb.append(res[i]);
    }
    return sb.toString();
}
```

## [6. Z 字形变换](https://leetcode-cn.com/problems/zigzag-conversion/)

将一个给定字符串根据给定的行数，以从上往下、从左到右进行 Z 字形排列。

比如输入字符串为 `"LEETCODEISHIRING"` 行数为 3 时，排列如下：

```java
L   C   I   R
E T O E S I I G
E   D   H   N
```


之后，你的输出需要从左往右逐行读取，产生出一个新的字符串，比如："LCIRETOESIIGEDHN"。

请你实现这个将字符串进行指定行数变换的函数：

```java
string convert(string s, int numRows);
```

**示例 1:**

```java
输入: s = "LEETCODEISHIRING", numRows = 3
输出: "LCIRETOESIIGEDHN"
```

**示例 2:**

```java
输入: s = "LEETCODEISHIRING", numRows = 4
输出: "LDREOEIIECIHNTSG"
解释:
L     D     R
E   O E   I I
E C   I H   N
T     S     G
```

**解法一**

比较脑残，但是勉强还是过了

```java
public String convert(String s, int numRows) {
    if (s==null || s.length()<=0 || numRows==1) {
        return s;
    }
    int len=s.length();
    //足够的空间
    int[][] strs=new int[((len/((numRows-1)*2))+1)*(numRows-1)][numRows];
    int index=0,x=0,y=0;
    boolean flag=false;
    while(index < s.length()) {
        if (!flag) {
            strs[x][y++]=s.charAt(index++);
            if (y==numRows-1) {
                flag=true;
            }
        }else{
            strs[x++][y--]=s.charAt(index++);
            if (y==0) {
                flag=false;
            }
        }
    }
    StringBuilder sb=new StringBuilder();
    for (int j=0;j<strs[0].length;j++) {
        for (int i=0;i<strs.length;i++) {
            if (strs[i][j]!=0) {
                sb.append((char)strs[i][j]);
            }
        }
    }
    return sb.toString();
}
```

就是将字符按照之字形填入一个二维数组中，然后按规则取出来就ok，最优解看了，明天再来写！

**解法二**

今天还是不够清晰，后天再写

## [392. 判断子序列](https://leetcode-cn.com/problems/is-subsequence/)

给定字符串 s 和 t ，判断 s 是否为 t 的子序列。

你可以认为 s 和 t 中仅包含英文小写字母。字符串 t 可能会很长（长度 ~= 500,000），而 s 是个短字符串（长度 <=100）。

字符串的一个子序列是原始字符串删除一些（也可以不删除）字符而不改变剩余字符相对位置形成的新字符串。（例如，`"ace"`是`"abcde"`的一个子序列，而`"aec"`不是）。

**示例 1:**

```java
s = "abc", t = "ahbgdc"
返回 true.
```

**示例 2:**

```java
s = "axc", t = "ahbgdc"
返回 false.
```

**后续挑战 :**

如果有大量输入的 S，称作S1, S2, ... , Sk 其中 k >= 10亿，你需要依次检查它们是否为 T 的子序列。在这种情况下，你会怎样改变代码？

**解法一**

```java
public boolean isSubsequence(String s, String t) {
    if (s==null || t==null) {
        return false;
    }
    int sindex=0,tindex=0;
    while(sindex<s.length()) {
        while(tindex<t.length() && sindex<s.length()){
            if (s.charAt(sindex)==t.charAt(tindex)) {
                sindex++;
            }
            tindex++;
        }
        if (tindex==t.length()) {
            break; 
        }
    }
    return sindex==s.length();
}
```
可以改成递归（多练习递归）

```java
public boolean isSubsequence(String s,String t){
    return subsequence(s,t,0,0);
}

public boolean subsequence(String s,String t,int sindex,int tindex){
    if (sindex == s.length()) {
        return true;
    }
    //上下if不能交换,可能最后一个才相等
    if (tindex == t.length()) {
        return false;
    }
    return s.charAt(sindex)==t.charAt(tindex)?subsequence(s,t,sindex+1,tindex+1):subsequence(s,t,sindex,tindex+1);
}
```
**解法二**

```java
//大量的s字符串 处理
public boolean isSubsequence3(String s, String t) {
    //预处理
    ArrayList<ArrayList<Integer>> hash=new ArrayList<>();
    for (int i=0;i<26;i++) {
        hash.add(new ArrayList());
    }
    for (int i=0;i<t.length();i++) {
        hash.get(t.charAt(i)-'a').add(i);
    }
    //经过上面的预处理,后面的处理就会很快,不用再遍历t字符串
    int lastIndex=-1;
    for (int i=0;i<s.length();i++) {
        List<Integer> indexList=hash.get(s.charAt(i)-'a');
        int temp=binarySearch(indexList,lastIndex);
        if (temp==indexList.size()) {
            return false;
        }
        lastIndex=indexList.get(temp);
    }
    return true;
}

//找到第一个比target大的元素
public int binarySearch(List<Integer> list,int target){
    int left=0,right=list.size()-1;
    while(left<=right){
        int mid=left+(right-left)/2;
        if (list.get(mid)>target) {
            right=mid-1;
        }else{
            left=mid+1;
        }
    }
    return left;
}
```

## [189. 旋转数组](https://leetcode-cn.com/problems/rotate-array/)

给定一个数组，将数组中的元素向右移动 k 个位置，其中 k 是非负数。

**示例 1:**

```java
输入: [1,2,3,4,5,6,7] 和 k = 3
输出: [5,6,7,1,2,3,4]
解释:
向右旋转 1 步: [7,1,2,3,4,5,6]
向右旋转 2 步: [6,7,1,2,3,4,5]
向右旋转 3 步: [5,6,7,1,2,3,4]
```


**示例 2:**

```java
输入: [-1,-100,3,99] 和 k = 2
输出: [3,99,-1,-100]
解释: 
向右旋转 1 步: [99,-1,-100,3]
向右旋转 2 步: [3,99,-1,-100]
```

**说明:**

- 尽可能想出更多的解决方案，至少有三种不同的方法可以解决这个问题。
- 要求使用空间复杂度为 O(1) 的 **原地** 算法 

**解法一**

常规解法，每次保留数组最后一个元素，从后往前将每个元素赋值为前一个元素的值，这样就相当于将数组整体向后循环移动一次，循环移动k次就是最后的结果

```java
public void rotate(int[] nums, int k) {
    if(nums==null||nums.length<=1||k==0){
        return;
    }
    int len=nums.length;
    k=k%len;
    for (int i=0;i<k;i++) {
        int temp=nums[len-1];
        for (int j=len-1;j>=0;j--) {
            nums[j]=nums[j-1];
        }
        nums[0]=temp;
    }
}
```
时间复杂度较高，`O(NK)` Java可以过，但是C/C++可能过不了

**解法二**

这个做法就相当巧妙了，三次翻转🐂🍺

```java
//翻转的方法
public void rotate(int[] nums, int k) {
    if(nums==null||nums.length<=1||k==0){
        return;
    }
    int len=nums.length;
    k=k%len;
    if(k==0)return;
    reverse(nums,0,len-k-1);
    reverse(nums,len-k,len-1);
    reverse(nums,0,nums.length-1);
}

public void reverse(int []nums,int left,int right){
    while(left<right){
        swap(nums,left++,right--);
    }
}

public void swap(int []nums,int a,int b){
    int temp=nums[a];
    nums[a]=nums[b];
    nums[b]=temp;
}
```
`O(N)` 应该是最优解了

## [1232. 缀点成线](https://leetcode-cn.com/problems/check-if-it-is-a-straight-line/)

在一个 XY 坐标系中有一些点，我们用数组 coordinates 来分别记录它们的坐标，其中 coordinates[i] = [x, y] 表示横坐标为 x、纵坐标为 y 的点。

请你来判断，这些点是否在该坐标系中属于同一条直线上，是则返回 `true`，否则请返回  `false`

**解法一**

10.20竞赛第一题，判断给定的点是不是再一条直线上，判断和前两个点是不是在一条直线上，注意不要直接除算斜率，那样是不准确的

```java
public boolean checkStraightLine(int[][] coordinates) {
    for (int i=2;i<coordinates.length;i++) {
        if((coordinates[i][1]-coordinates[i-1][1])*(coordinates[i-1][0]-coordinates[i-2][0])!=
           (coordinates[i][0]-coordinates[i-1][0])*(coordinates[i-1][1]-coordinates[i-2][1])){
            return false;
        }
    }
    return true;
}
```

## [1233. 删除子文件夹](https://leetcode-cn.com/problems/remove-sub-folders-from-the-filesystem/)

你是一位系统管理员，手里有一份文件夹列表 folder，你的任务是要删除该列表中的所有 子文件夹，并以 任意顺序 返回剩下的文件夹。

我们这样定义「子文件夹」：

- 如果文件夹 `folder[i]` 位于另一个文件夹 `folder[j]` 下，那么 `folder[i]` 就是 `folder[j]` 的子文件夹。
  文件夹的「路径」是由一个或多个按以下格式串联形成的字符串：

- `/` 后跟一个或者多个小写英文字母。
  例如，`/leetcode` 和 `/leetcode/problems` 都是有效的路径，而空字符串和 `/` 不是。

 **示例 1：**

```java
输入：folder = ["/a","/a/b","/c/d","/c/d/e","/c/f"]
输出：["/a","/c/d","/c/f"]
解释："/a/b/" 是 "/a" 的子文件夹，而 "/c/d/e" 是 "/c/d" 的子文件夹。
```


**示例 2：**

```java
输入：folder = ["/a","/a/b/c","/a/b/d"]
输出：["/a"]
解释：文件夹 "/a/b/c" 和 "/a/b/d/" 都会被删除，因为它们都是 "/a" 的子文件夹。
```


**示例 3：**

```java
输入：folder = ["/a/b/c","/a/b/d","/a/b/ca"]
输出：["/a/b/c","/a/b/ca","/a/b/d"]
```

**提示：**

- 1 <= folder.length <= 4 * 10^4
- 2 <= folder[i].length <= 100
- folder[i] 只包含小写字母和 /
- folder[i] 总是以字符 / 起始
- 每个文件夹名都是唯一的

**解法一**

`2019.10.20`的竞赛题，当时没做出来。。。一直超时，太菜了

```java
public List<String> removeSubfolders(String[] folder) {
    Arrays.sort(folder);
    List<String> res=new LinkedList<>();
    int root=0;
    res.add(folder[0]);
    for (int i=1;i<folder.length;i++) {
        if (!folder[i].startsWith(folder[root]+"/")) {
            res.add(folder[i]);
            root=i;
        }
    }
    return res;
}
```
当时我想到了排序，但是并没处理好，排序之后还是傻傻的一个个去对比，其实排序后就很清楚了

`folder = ["/a","/a/b","/c/d","/c/d/e","/c/f"]` 题目其实也在暗示我们要排序，给的case都是排好序的

当然这里很精髓的一步就是在对比的时候在 `folder[root]` 后面加上一个 `"/"` ，这样就不会将 `a/b/c` 判断为 `a/b/ca` 的根目录了~

## [5. 最长回文子串](https://leetcode-cn.com/problems/longest-palindromic-substring/)

给定一个字符串 s，找到 s 中最长的回文子串。你可以假设 s 的最大长度为 1000。

**示例 1：**

```java
输入: "babad"
输出: "bab"
注意: "aba" 也是一个有效答案。
```


**示例 2：**

```java
输入: "cbbd"
输出: "bb"
```

**解法一**

是面试经常考的一题，还是挺有意思的，除了这个还有几道回文的题我放在动态规划专题中

```java
public String longestPalindrome(String s) {
    if (s==null || s.length()<=0) {
        return "";
    }
    String res=s.charAt(0)+"";//只有1个字符
    for (int i=1;i<s.length();i++) {
        String even=palindrome(s,i-1,i); //偶数长度回文,从两个字符中间开始扩散
        String odd=palindrome(s,i,i); //奇数长度回文,从某一个字符开始扩散
        String temp=odd.length()>even.length()?odd:even;
        if (temp.length()>res.length()) {
            res=temp;
        }
    }
    return res;
}

public String palindrome(String s,int i,int j){
    while(i>=0 && j<=s.length()-1 && s.charAt(i)==s.charAt(j)){
        i--;
        j++;
    }
    return s.substring(i+1,j);
}
```
如果采用暴力法的话就是枚举所有子串，判断是不是回文串，最后求个最长的，时间复杂度`O(N^3)` ，但是我们可以利用回文的特征，利用中心扩散法，以`str`的**各个位置**作为中心，向两边扩散，最后求得最大值，注意得这里说的是**各个位置**，这个里面其实就包含了元素之间的间隙，其实整体思路还是挺简单的，但经过我们小小的转换思路，时间复杂度就降低到了`O(N^2)`，当然，这里还不是最优解，最优应该是[Manacher](https://oi-wiki.org/string/manacher/) （马拉车）算法，等后面有时间我再来研究这种算法

## [56. 合并区间](https://leetcode-cn.com/problems/merge-intervals/)

给出一个区间的集合，请合并所有重叠的区间。

**示例 1:**

```java
输入: [[1,3],[2,6],[8,10],[15,18]]
输出: [[1,6],[8,10],[15,18]]
解释: 区间 [1,3] 和 [2,6] 重叠, 将它们合并为 [1,6].
```


**示例 2:**

```java
输入: [[1,4],[4,5]]
输出: [[1,5]]
解释: 区间 [1,4] 和 [4,5] 可被视为重叠区间。
```

**解法一**

思路也没啥好说的，类似贪心吧

```java
public int[][] merge(int[][] intervals) {
    if (intervals ==null || intervals.length<=0) {
        return new int[][]{};
    }
    Arrays.sort(intervals,(a,b)->a[0]-b[0]);
    LinkedList<int[]> list=new LinkedList<>();
    for (int i=1;i<intervals.length;i++) {
        if (intervals[i][0]<=intervals[i-1][1]) {
            if (intervals[i][1]>intervals[i-1][1]) {
                intervals[i][0]=intervals[i-1][0];   
            }else{
                intervals[i][0]=intervals[i-1][0];
                intervals[i][1]=intervals[i-1][1];
            }
        }else{
            list.add(intervals[i-1]);
        }
    }
    list.add(intervals[intervals.length-1]);
    /*  int[][] res=new int[list.size()][2];
        for (int i=0;i<list.size();i++) {
            res[i][0]=list.get(i)[0];
            res[i][1]=list.get(i)[1];
        }*/
    return list.toArray(new int[0][0]); //题解哪里学到一招
}
```

最大的收获就是学到了一招list转array的方法😁

## [435. 无重叠区间](https://leetcode-cn.com/problems/non-overlapping-intervals/)

给定一个区间的集合，找到需要移除区间的最小数量，使剩余区间互不重叠。

注意:

- 可以认为区间的终点总是大于它的起点。
- 区间 [1,2] 和 [2,3] 的边界相互“接触”，但没有相互重叠。

**示例 1:**

```java
输入: [ [1,2], [2,3], [3,4], [1,3] ]

输出: 1

解释: 移除 [1,3] 后，剩下的区间没有重叠。
```

**示例 2:**

```java
输入: [ [1,2], [1,2], [1,2] ]

输出: 2

解释: 你需要移除两个 [1,2] 来使剩下的区间没有重叠。
```

**示例 3:**

```java
输入: [ [1,2], [2,3] ]

输出: 0

解释: 你不需要移除任何区间，因为它们已经是无重叠的了。
```

**解法一**

动态规划，其实和最长递增子序列是一样的

```java
public int eraseOverlapIntervals(int[][] intervals) {
    if (intervals==null || intervals.length<=0) {
        return 0;
    }
    Arrays.sort(intervals,(a,b)->a[0]-b[0]);
    int[]dp=new int[intervals.length];
    int max=-1;
    for (int i=0;i<intervals.length;i++) {
        dp[i]=1;
        for (int j=0;j<i;j++) {
            if(intervals[i][0]>=intervals[j][1]){
                dp[i]=Math.max(dp[j]+1,dp[i]);
            }
        }
        max=Math.max(max,dp[i]);
    }
    return intervals.length-max;
}
```
171ms，8%，感觉快要过不了了。。。本来是是写的记忆化递归的，结果过不了。。。卡在倒数第二个case上

```java
HashMap<Pair,Integer> cache=new HashMap<>();//TLE

public int eraseOverlapIntervals2(int[][] intervals) {
    Arrays.sort(intervals,(a,b)->a[0]-b[0]);
    return intervals.length-dfs(intervals,0,Integer.MIN_VALUE);
}

//背包问题,返回最多可以留下的区间
public int dfs(int[][] intervals,int index,int prev) {
    if (index==intervals.length) {
        return 0;
    }
    Pair key=new Pair(index,prev);
    if (cache.containsKey(key)) {
        return cache.get(key);
    }
    int res=dfs(intervals,index+1,prev);
    if (intervals[index][0]>=prev) {
        res=Math.max(res,dfs(intervals,index+1,intervals[index][1])+1);
    }
    cache.put(key,res);
    return res;
}
```
**解法二**

贪心，时间复杂度降低为线性

```java
public int eraseOverlapIntervals(int[][] intervals) {
    if (intervals==null || intervals.length<=0) {
        return 0;
    }
    //按照起点排序,重叠的时候选择保留结尾小的那一个
    //Arrays.sort(intervals,(a,b)->a[0]-b[0]); lambda初始化效率会低一点
    Arrays.sort(intervals,new Comparator<int[]>(){
        @Override
        public int compare(int[] a,int[] b){
            return a[0]-b[0];
        }
    });
    int res=1;
    int prev=0;
    for (int i=1;i<intervals.length;i++) {
        if (intervals[i][0]>=intervals[prev][1]) {
            res++;
            prev=i;
        }else if(intervals[i][1]<intervals[prev][1]){
            prev=i; //选择结尾小的那一个
        }
    }
    return intervals.length-res;
}
```
按照起点排序，在重叠的时候优先选择结尾小的哪一个，这样就可能得到更多的区间组合，关于这个算法的正确性我就不证明了

## [240. 搜索二维矩阵 II](https://leetcode-cn.com/problems/search-a-2d-matrix-ii/)

编写一个高效的算法来搜索 m x n 矩阵 matrix 中的一个目标值 target。该矩阵具有以下特性：

- 每行的元素从左到右升序排列。
- 每列的元素从上到下升序排列。

**示例:**
现有矩阵 matrix 如下：

```java
[
  [1,   4,  7, 11, 15],
  [2,   5,  8, 12, 19],
  [3,   6,  9, 16, 22],
  [10, 13, 14, 17, 24],
  [18, 21, 23, 26, 30]
]
```

给定 target = `5`，返回 `true`。

给定 target = `20`，返回 `false`。

**解法一**

看上一题 [240. 搜索二维矩阵](https://leetcode-cn.com/problems/search-a-2d-matrix/)  的评论区的时候看到了这种解法

```java
public boolean searchMatrix(int[][] matrix, int target) {
    if(matrix==null || matrix.length<=0 || matrix[0].length<=0){
        return false;
    }
    int m=matrix.length;
    int n=matrix[0].length;
    int  column=0,row=m-1;
    while(column<n && row>=0){
        //System.out.println(row+","+column);
        if (matrix[row][column]==target) {
            return true;
        }
        if (matrix[row][column] > target) {
            row--;
        }else{
            column++;
        }
    }
    return false;
}
```
整个矩阵从左上到右下，其实就分为了两块，每个元素的左上一定小于当前元素，右下一定大于当前元素，所以很自然就可以写出类似的O(m+n)的解法，这题好像也可以二分，但是还是这种比较好理解

## [532. 逆序对](https://www.lintcode.com/problem/reverse-pairs/description)

（来自领扣）

在数组中的两个数字如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。给你一个数组，求出这个数组中逆序对的总数。
概括：如果a[i] > a[j] 且 i < j， a[i] 和 a[j] 构成一个逆序对。

**样例1**

```java
输入: A = [2, 4, 1, 3, 5]
输出: 3
解释:
(2, 1), (4, 1), (4, 3) 是逆序对
```

**样例2**

```java
输入: A = [1, 2, 3, 4]
输出: 0
解释:
没有逆序对
```

**解法一**

```java
public long reversePairs(int[] A) {
    if (A==null || A.length<=0) {
        return 0;
    }
    return reversePairs(A,0,A.length-1);
}

public long reversePairs(int[] A,int left,int right) {
    if (left == right) {
        return 0;
    }
    int mid=left+(right-left)/2;
    long l=reversePairs(A,left,mid);
    long r=reversePairs(A,mid+1,right);
    return merge(A,left,mid,right)+l+r;
}

public long merge(int[] nums,int left,int mid,int right){
    long res=0;
    int[] help=new int[right-left+1];
    int i=left,j=mid+1;
    int index=0;
    while(i<=mid && j<=right){
        //小于等于的时候让i先进栈
        //help[index++]=nums[i]<=nums[j] ? nums[i++]:nums[j++];
        if (nums[i]<=nums[j]) {
            help[index++] = nums[i++];
        }else{
            help[index++] = nums[j++];
            res+= mid-i+1; //j和i-mid间的所有元素形成逆序对
        }
    }
    while(i<=mid){
        help[index++]=nums[i++];
    }
    while(j<=right){
        help[index++]=nums[j++];
    }

    for (int k=0;k<help.length;k++) {
        nums[left+k]=help[k];
    }
    return res;
}
```

归并排序的思路，最开始我是在每次i>j和最后收尾的时候res++，然后结果总是不对，然后取查了答案才意识到不能这样算，当`nums[i] > nums[j]` 的时候，`i~j` 形成的逆序对其实不只一个，而是`[i,mid]` 区间的所有元素，如果你只是+1的话就会漏掉许多情况，因为下一步 `j++` 就会将 `j` 向后移动，那些情况就考虑不到了

## [263. 丑数](https://leetcode-cn.com/problems/ugly-number/)

编写一个程序判断给定的数是否为丑数。

丑数就是只包含质因数 2, 3, 5 的正整数。

**示例 1:**

```java
输入: 6
输出: true
解释: 6 = 2 × 3
```

**示例 2:**

```java
输入: 8
输出: true
解释: 8 = 2 × 2 × 2
```

**示例 3:**

```java
输入: 14
输出: false 
解释: 14 不是丑数，因为它包含了另外一个质因数 7。
```

**说明：**

1. `1` 是丑数。
2. 输入不会超过 `32` 位有符号整数的范围: `[−231,  231 − 1]`。

**解法一**

直接暴力，还是比较简单

```java
public boolean isUgly(int num) {
    if (num<=0) {
        return false;
    }
    if(num==1) {
        return true;
    }
    return num%2==0?isUgly(num/2):false || num%3==0?isUgly(num/3):false || num%5==0?isUgly(num/5):false;
}
```
## 贪心

先暂时放在这里，以后多了之后在考虑分开的问题

## [621. 任务调度器](https://leetcode-cn.com/problems/task-scheduler/)

给定一个用字符数组表示的 CPU 需要执行的任务列表。其中包含使用大写的 A - Z 字母表示的26 种不同种类的任务。任务可以以任意顺序执行，并且每个任务都可以在 1 个单位时间内执行完。CPU 在任何一个单位时间内都可以执行一个任务，或者在待命状态。

然而，两个**相同种类**的任务之间必须有长度为 n 的冷却时间，因此至少有连续 n 个单位时间内 CPU 在执行不同的任务，或者在待命状态。

你需要计算完成所有任务所需要的**最短时间**

**示例 1：**

```java
输入: tasks = ["A","A","A","B","B","B"], n = 2
输出: 8
执行顺序: A -> B -> (待命) -> A -> B -> (待命) -> A -> B.
```

**注：**

- 任务的总个数为 `[1, 10000]`
- n 的取值范围为 `[0, 100]`

**解法一**

```java
public int leastInterval(char[] tasks, int n) {
    int[] map=new int[26];
    for (int i=0;i<tasks.length;i++) {
        map[tasks[i]-'A']++;
    }
    //找最大值
    int max=-1;
    for (int i=0;i<map.length;i++) {
        max=Math.max(map[i],max);
    }
    int maxCount=0;
    for (int i=0;i<map.length;i++) {
        if (map[i]==max) {
            maxCount++;
        }
    }
    //比如 a b c d e f g,n=1
    return Math.max((n+1)*(max-1)+maxCount,tasks.length);
}
```
核心思想就是将出现次数最多的任务优先执行并且尽可能的分散，比如  `A A A B B C n=2` 最短的时间就是`A X X A X X A` ，最终的时间就是`(n+1)*(max-1)+1` 也就是 `(2+1) *(3-1)+1=7`， 但是可能会有多个最多次数的任务，所以我们还需要加上最多的相同的个数，最后就是 `(n+1)*(max-1)+maxCount` ，但是还不够，还是有可能会出现代码中的例子，也就是最后得到的结果比我们的任务列表还有短，所以我们需要取一个最大值

## [45. 跳跃游戏 II](https://leetcode-cn.com/problems/jump-game-ii/)

给定一个非负整数数组，你最初位于数组的第一个位置。

数组中的每个元素代表你在该位置可以跳跃的最大长度。

你的目标是使用最少的跳跃次数到达数组的最后一个位置。

**示例:**

```java
输入: [2,3,1,1,4]
输出: 2
解释: 跳到最后一个位置的最小跳跃数是 2。
     从下标为 0 跳到下标为 1 的位置，跳 1 步，然后跳 3 步到达数组的最后一个位置。
```

**说明:**

假设你总是可以到达数组的最后一个位置。

**解法一**

兴致勃勃写了个dp

```java
public int jump(int[] nums) {
    if (nums==null || nums.length<=0) {
        return 0;
    }
    int[] dp=new int[nums.length];
    for (int i=1;i<nums.length;i++) {
        dp[i]=Integer.MAX_VALUE;
        for (int j=0;j<i;j++) {
            if (nums[j]>=i-j) {
                dp[i]=Math.min(dp[j]+1,dp[i]);    
            }
        }
    }
    return dp[nums.length-1];
}
```

如果这就过了那你也太小瞧这题了😂人家可是hard题，那能这么容易就让你过了？

没错，这里直接TLE了，最后一个CASE过不去

**解法二**

贪心，核心思想不是每次都跳到最远的地方，**而是跳到当前位置能跳到的最远的位置**

```java
//每次选能跳的位置中跳的最远的
public int jump(int[] nums){
    if (nums==null || nums.length<=0) {
        return 0;
    }
    int max=0;//最大边界
    int step=0,curMaxIndex=0;
    for (int i=0;i<nums.length-1;i++) {
        curMaxIndex=Math.max(curMaxIndex,nums[i]+i); //i能跳的位置中,跳的最远的
        if (i==max) {//走到边界就++
            step++;
            max=curMaxIndex;
        }
    }
    return step;
}
```

代码需要细细品，一下可能看不太明白