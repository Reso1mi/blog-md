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

## _双指针_ 

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
//update: 2020.4.18
public int maxArea(int[] height) {
    if(height==null || height.length<=0) return 0;
    int left=0,right=height.length-1;
    int max=0;
    while(left<right){
        max=Math.max((right-left)*Math.min(height[left],height[right]),max);
        //if(left<right){ 隐约记得之前也这样写过。。。没想到这次又在这里WA了
        if(height[left]<height[right]){
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

## [42. 接雨水](https://leetcode-cn.com/problems/trapping-rain-water/)

给定 *n* 个非负整数表示每个宽度为 1 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

![rainwatertrap.png](https://i.loli.net/2019/05/14/5cda71129045d93180.png)




上面是由数组 `[0,1,0,2,1,0,1,3,2,1,2,1]` 表示的高度图，在这种情况下，可以接 6 个单位的雨水（蓝色部分表示雨水）。

**示例:**

```java
输入: [0,1,0,2,1,0,1,3,2,1,2,1]
输出: 6
```

**解法一**

> 我最开始思路是填满后用总面积减数组和，跑过了130+个，有一种特殊的跑不过了，懒得去处理那个边界了，不太优雅

这个题目的关键就是每个柱子能接的水是**左右最长柱子(都大于当前柱子)中的较小的那个减去当前柱子**。

所以我们可以用两个数组分别存储每个柱子左右的最长柱子（做预处理），这样就得到了一种有点动态规划意思的解法

```java
public static int trap(int []height){
    if (height==null || height.length<=0) {
        return 0;
    }
    int len=height.length;
    int[] leftMax=new int[len];
    leftMax[0]=height[0];
    int[] rightMax=new int[len];
    rightMax[len-1]=height[len-1];
    int res=0;
    //左右最大柱子包含当前柱子
    for (int i=1;i<len;i++) {
        leftMax[i]=Math.max(leftMax[i-1],height[i]);
    }
    for (int i=len-2;i>=0;i--) {
        rightMax[i]=Math.max(rightMax[i+1],height[i]);
    }
    for (int i=0;i<len;i++) {
        res+=Math.min(rightMax[i],leftMax[i])-height[i];
    }
    return res;
}
```

利用**双指针**就行空间的优化

```java
public static int trap(int []height){
    if (height==null || height.length<=0) {
        return 0;
    }
    int len=height.length;
    int leftMax=0,rightMax=0;
    int left=0,right=len-1,res=0;
    while(left<=right){
        leftMax=Math.max(leftMax,height[left]);
        rightMax=Math.max(rightMax,height[right]);
        //leftMax小于rightMax,那么靠近leftMax的柱子left可以接的雨水就可以确定了
        if (leftMax<rightMax) {
            res+=leftMax-height[left]; 
            left++;
        }else{ //反之leftMax大于rightMax,那么考近rightMax的柱子right可以接的最多的雨水就可以i确定了
            res+=rightMax-height[right];
            right--;
        }
    }
    return res;
}
```

个人感觉这个是最好理解的版本，我这里最开始的哪个版本不是这样写的，当时自己肯定也没搞懂，包括现在我也没搞懂那种写法

![[图片来自liweiwei1419大佬](https://leetcode-cn.com/u/liweiwei1419/)](http://static.imlgw.top/blog/20200129/Dy8M19G4XwSn.png?imageslim)

这两种情况对应的就是循环中的if的两个分支，双指针向中间靠拢，当`leftMax`小于`rightMax`的时候我们不用去考虑当前`left`柱子右边实际的最大的右边的柱子是谁，我们只需要知道`left`柱子 左边最大值`leftMax`的值就ok，因为此时`left` 柱子能接水的量是由`leftMax`决定的，反之对应第二种情况，`right`柱子的接水量则是由`rightMax` 决定的，最后遍历完所有的柱子就可以确定整体的接水量

> 这里的if分支的条件有的解法中写的是leftMax < nums[right]甚至nums[left] < nums[right] 这也是我上面说的不理解的地方，因为这样写也是可以AC的😅，后面有时间再回头看看吧

**解法二**

还有一种很巧妙的方法，也比较好理解，找到最大值，然后分别对两边的柱子进行遍历，如果当前的柱子小于前面柱子的最大值，就说明一定可以接到水，这个过程中需要记录柱子左边和右边的最大值，用于计算可以接水的量，最后计算总和

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

> 2020/1/29回顾
>
> 这个解法其实就是单调栈😂，当时还是菜鸟根本就不懂，现在回头一看就懂了hahaha~ 
>
> 放到[单调栈专题](http://imlgw.top/2019/10/01/leetcode-zhan-dui-lie/#%E5%8D%95%E8%B0%83%E6%A0%88)里面解释了

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

**解法一**

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

**解法一**

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

**解法一**

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

**解法一**

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

**解法二**

保持`[0,m)` 为非0元素，遇到非0元素就和右边界进行交换

```java
public void moveZeroes(int[] nums) {
    int m=0; //[0,m)为非0元素
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

**解法一**

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

并不是最优解，时间复杂度`O(NlogN)`，只是一种思路吧，而且是通用的求众数的方法

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
**解法三**

刚刚看见一种解法，当作求第k大，用快选就行了，时间复杂度`O(N)`

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

和上面的方法一样，抵消去除三个不同的元素对众数没有任何影响，但是最后需要判断是否都是符合条件的

```go
//update：2020.4.23 用go在web上随手写了一个，感觉比之前java写的哪个好理解
func majorityElement(nums []int) []int {
    var res []int
    //超过n/3的元素最多2个
    cand1:=-1 //设置成nums中不存在的值比较好,比如-1
    count1:=0
    cand2:=-1
    count2:=0
    for _,num:=range nums{
        if num==cand1{ //投1
            count1++
        }else if num==cand2{ //投2
            count2++
        }else { //都不投
            if count1==0 {
                cand1=num
                count1=1
            }else if count2==0{
                cand2=num
                count2=1
            }else{
                count1--
                count2--
            }
        }
    }
    temp1:=0
    temp2:=0
    for _,num:= range nums{
        if num==cand1{
            temp1++
        }
        if num==cand2{
            temp2++
        }
    }

    if temp1>len(nums)/3 {
        res=append(res,cand1)
    }
    if temp2>len(nums)/3 {
        res=append(res,cand2)
    }
    return res
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
        //将每个元素归位，我开始只有一层循环，那样会漏掉很多元素（可能被交换的元素后面也需要交换），这样的就是一次直接到位。
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

其实也是桶排序的思想，不过这里是利用交换来定位每个元素，首相我们将原数组看作桶，题目要求的正整数，所以我们桶中存的应该是`【1，nums.length】`，也就是0位置应该存放的是1，1位置存放的应该是2....再归位后重新遍历数组，如果某个位置的`nums[i]!=i+1` 就说明这个是第一个缺失的正数，遍历完了之后没有找到，全部对应上了，那就说明我们缺少的第一个正数是`nums.length+1`

**解法二**

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

## [442. 数组中重复的数据](https://leetcode-cn.com/problems/find-all-duplicates-in-an-array/)

给定一个整数数组 a，其中1 ≤ a[i] ≤ n （n为数组长度）, 其中有些元素出现两次而其他元素出现一次。

找到所有出现两次的元素。

你可以不用到任何额外空间并在O(n)时间复杂度内解决这个问题吗？

**示例：**

```java
输入:
[4,3,2,7,8,2,3,1]

输出:
[2,3]
```

**解法一**

同上，抽屉原理，直接秒掉这三题 hard，mid，easy

```java
public List<Integer> findDuplicates(int[] nums) {
    for (int i=0;i<nums.length;i++) {
        while(nums[i]!=i+1 && nums[i]!=nums[nums[i]-1]){
            int temp=nums[nums[i]-1];
            nums[nums[i]-1]=nums[i];
            nums[i]=temp;
        }
    }
    List<Integer> res=new LinkedList<>();
    for (int i=0;i<nums.length;i++) {
        if (nums[i]!=i+1) {
            res.add(nums[i]);
        }
    }
    return res;
}
```

**解法二**

技巧性的思路，和上一题一样，将对应位置置反，如果遇到已经置反的就说明当前位置重复了

```java
//5 1 1 3 2
public List<Integer> findDuplicates(int[] nums) {
    List<Integer> res=new LinkedList<>();
    for (int i=0;i<nums.length;i++) {
        if (nums[Math.abs(nums[i])-1]<0) {
            res.add(Math.abs(nums[i]));
        }
        nums[Math.abs(nums[i])-1]=-Math.abs(nums[Math.abs(nums[i])-1]);
    }
    return res;
}
```

## [448. 找到所有数组中消失的数字](https://leetcode-cn.com/problems/find-all-numbers-disappeared-in-an-array/)

给定一个范围在  1 ≤ a[i] ≤ n ( n = 数组大小 ) 的 整型数组，数组中的元素一些出现了两次，另一些只出现一次。

找到所有在 [1, n] 范围之间没有出现在数组中的数字。

您能在不使用额外空间且时间复杂度为O(n)的情况下完成这个任务吗? 你可以假定返回的数组不算在额外空间内。

**示例:**

```java
输入:
[4,3,2,7,8,2,3,1]

输出:
[5,6]
```

**解法一**

首先想到的解法，利用的和上面缺失的第一个正数一样的思路，抽屉原理，归位每个数字，最后没有归为的index就是消失的数字

```java
public List<Integer> findDisappearedNumbers(int[] nums) {
    //nums[i]=i+1
    for (int i=0;i<nums.length;i++) {
        while(nums[i]!=i+1 && nums[nums[i]-1]!=nums[i]){
            int temp=nums[i];
            nums[i]=nums[temp-1];
            nums[temp-1]=temp;
            //nums[i]=nums[nums[i]-1]; 最开始的错误写法
            //nums[nums[i]-1]=temp;
        }
    }
    List<Integer> res=new LinkedList<>();
    for (int i=0;i<nums.length;i++) {
        if (nums[i]!=i+1) {
            res.add(i+1);
        }
    }
    return res;
}
```

中间写出了一个小`bug`，交换两个元素的时候先交换了`nums[i]`，导致了后面的`nums[nums[i]+1]` 发生了变化，然后就死循环了😂，调试了一下才看出来，太菜了

**解法二**

很巧妙的方法

```java
//很巧妙
public List<Integer> findDisappearedNumbers(int[] nums) {
    //nums[i]=i+1
    //5 1 4 2 3
    for (int i=0;i<nums.length;i++) {
        nums[Math.abs(nums[i])-1]=-Math.abs(nums[Math.abs(nums[i])-1]);
    }
    List<Integer> res=new LinkedList<>();
    for (int i=0;i<nums.length;i++) {
        if (nums[i]>0) {
            res.add(i+1);
        }
    }
    return res;
}
```

题目给定了数值的范围就是`[1,n]`所以可以遍历每个元素，将该元素正确位置的值取反置为负数

比如 `5 1 1 3 2` 遍历到5的时候就会将末尾的2变为-2，依次类推，最后得到的就是`[-5,-1,-1,3,-2]` ，最后再遍历一遍，其中值为正数的元素的索引+1就是消失的数字

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

**解法一**

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

**解法一**

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

**解法一**

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

**解法二**

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

**解法三**

时隔多年，LeetCode打卡又做了一遍，这次直接想到了最优解，而且代码很简洁

```java
public void merge(int[] A, int m, int[] B, int n) {
    if(n==0 )  return;
    int len=A.length,ai=m-1,bi=n-1,i=len-1;
    while(ai>=0 && bi>=0) A[i--]=A[ai] > B[bi] ? A[ai--]:B[bi--];
    while(bi>=0) A[i--]=B[bi--];
    //ai剩余的不用管
}
```
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

## [315. 计算右侧小于当前元素的个数](https://leetcode-cn.com/problems/count-of-smaller-numbers-after-self/)

给定一个整数数组 nums，按要求返回一个新数组 counts。数组 counts 有该性质： counts[i] 的值是  nums[i] 右侧小于 nums[i] 的元素的数量。

**示例:**

```java
输入: [5,2,6,1]
输出: [2,1,1,0] 
解释:
5 的右侧有 2 个更小的元素 (2 和 1).
2 的右侧仅有 1 个更小的元素 (1).
6 的右侧有 1 个更小的元素 (1).
1 的右侧有 0 个更小的元素.
```

**错误解法一**

这个bug我大概看了有两三个小时，人都看傻了，必须放上来纪念下

```go
import(
    "fmt"
)
type Element struct{
    idx int
    val int
}

func countSmaller(nums []int) []int {
    n:=len(nums)
    count:=make([]int,n)
    elements:=make([]Element,n)
    for i,num:=range nums{
        elements[i].idx=i
        elements[i].val=num
    }
    fmt.Println(elements);
    mergeSort(elements,0,n-1,count)
    return count
}

func mergeSort(num []Element,left int,right int,count []int){
    if left>=right{
        return
    }
    mid:=left+(right-left)/2
    mergeSort(num,left,mid,count)
    mergeSort(num,mid+1,right,count)
    merge(num,left,mid,right,count)
}

func merge(num []Element,left int,mid int,right int,count []int){
    help:=make([]int,right-left+1)
    i:=left
    j:=mid+1
    index:=0
    for i<=mid && j<=right {
        if num[i].val<=num[j].val{ //说明j前面的元素都小于i
            count[num[i].idx]+=(j-mid-1)
            help[index]=num[i].val
            i++
        }else{
            help[index]=num[j].val
            j++
        }
        index++
    }
    for i<=mid{
        count[num[i].idx]+=(j-mid-1)
        help[index]=num[i].val
        index++
        i++
    }
    for j<=right{
        help[index]=num[j].val
        index++
        j++
    }
    for i:=left;i<=right;i++{
        num[i].val=help[i-left]
    }
}
```

一开始用go写的，调了半天没调出来，我以为是go的啥问题（刚学go不太熟）然后用Java又写了一遍

```java
class Solution {
    public List<Integer> countSmaller(int[] nums) {
        Pair[] pair=new Pair[nums.length];
        for(int i=0;i<nums.length;i++){
            pair[i]=new Pair(i,nums[i]);
        }
        int[] count=new int[nums.length];
        mergeSort(pair,0,nums.length-1,count);
        List<Integer> res=new ArrayList<>();
        for(int i=0;i<count.length;i++){
            res.add(count[i]);
        }
        return res;
    }

    public void mergeSort(Pair[] nums,int left,int right,int[] count){
        if(left>=right){
            return;
        }
        int mid=left+(right-left)/2;
        mergeSort(nums,left,mid,count);
        mergeSort(nums,mid+1,right,count);
        merge(nums,left,mid,right,count);
    }

    public void merge(Pair [] nums,int left,int mid,int right,int[] count){
        int i=left,j=mid+1;
        //出Bug的地方，应该用 Pair[] 
        int[] helper=new int[right-left+1];
        int index=0;
        while(i<=mid && j<=right){
            if(nums[i].value>nums[j].value){
                helper[index++]=nums[j++].value;
            }else{
                count[nums[i].index]+=j-mid-1;
                helper[index++]=nums[i++].value;
            }
        }
        while(i<=mid){
            count[nums[i].index]+=j-mid-1;
            helper[index++]=nums[i++].value;
        }
        while(j<=right){
            helper[index++]=nums[j++].value;
        }
        for(int k=0;k<helper.length;k++){
            //这里无形之中将索引和数据的对应关系打乱了。。。。。
            nums[left+k].value=helper[k];
        }
    }

    class Pair{
        int index;
        int value;
        public Pair(int i,int v){
            index=i;
            value=v;
        }
    }
}
```

还是不对，和之前go的结果是一样的，这段代码我反复地看了3个小时，楞是没看出来哪里写错了，我是真的菜啊！！！！！！！！！

**解法一**

其实和逆序对的解法是类似的，思路都在注释中

```java
public List<Integer> countSmaller(int[] nums) {
    Pair[] pair=new Pair[nums.length];
    for(int i=0;i<nums.length;i++){
        pair[i]=new Pair(i,nums[i]);
    }
    int[] count=new int[nums.length];
    mergeSort(pair,0,nums.length-1,count);
    List<Integer> res=new ArrayList<>();
    for(int i=0;i<count.length;i++){
        res.add(count[i]);
    }
    return res;
}

public void mergeSort(Pair[] nums,int left,int right,int[] count){
    if(left>=right){
        return;
    }
    int mid=left+(right-left)/2;
    mergeSort(nums,left,mid,count);
    mergeSort(nums,mid+1,right,count);
    merge(nums,left,mid,right,count);
}

public void merge(Pair [] nums,int left,int mid,int right,int[] count){
    int i=left,j=mid+1;
    Pair[] helper=new Pair[right-left+1];
    int index=0;
    while(i<=mid && j<=right){
        if(nums[i].value>nums[j].value){
            helper[index++]=nums[j++];
        }else{
            //i<=j 那么mid+1~j-1的肯定都比i小
            //(j-1)-(mid+1)+1=j-mid-1
            count[nums[i].index]+=j-mid-1;
            helper[index++]=nums[i++];
        }
    }
    while(i<=mid){
        //j没了，那么所有的j的元素都比i小
        //等价于right-mid
        count[nums[i].index]+=j-mid-1;
        helper[index++]=nums[i++];
    }
    while(j<=right){
        helper[index++]=nums[j++];
    }
    for(int k=0;k<helper.length;k++){
        nums[left+k]=helper[k];
    }
}

class Pair{
    int index;
    int value;
    public Pair(int i,int v){
        index=i;
        value=v;
    }
}
```

> 这题还可以用**树状数组**解，但是我暂时还不会，后面有时间学了再来补充，其实还可以用线段树，二叉搜索树等等，有点麻烦，算了

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

## [59. 螺旋矩阵 II](https://leetcode-cn.com/problems/spiral-matrix-ii/)

给定一个正整数 n，生成一个包含 1 到 n^2 所有元素，且元素按顺时针顺序螺旋排列的正方形矩阵。

**示例:**

```java
输入: 3
输出:
[
 [ 1, 2, 3 ],
 [ 8, 9, 4 ],
 [ 7, 6, 5 ]
]
```

**解法一**

```java
public int[][] generateMatrix(int n) {
    int[][] res=new int[n][n];
    int left=0,right=n-1;
    int val=1;
    while(left<=right){
        int a=left,b=left;
        int c=right,d=right;
        if (left==right) {
            res[left][right]=val;
        }
        while(b<right){
            res[left][b++]=val++;
        }
        while(a<right){
            res[a++][right]=val++;
        }
        while(d>left){
            res[right][d--]=val++;
        }
        while(c>left){
            res[c--][left]=val++;
        }
        left++;right--;
    }
    return  res;
}
```

上一题的简化版，2020.2.11白板写的，还行

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

**解法二**

新学到的解法，挺有意思的，整体沿对角线交换，然后每行沿中点交换，这个其实可以通过观察数组结构得到

```java
public void rotate(int[][] matrix) {
    if(matrix==null || matrix.length<=0) return;
    int N=matrix.length;
    for(int i=0;i<N;i++){
        for(int j=i+1;j<N;j++){
            int temp=matrix[i][j];
            matrix[i][j]=matrix[j][i];
            matrix[j][i]=temp;
        }
    }

    for(int i=0;i<N;i++){
        for(int j=0,k=N-1;j<k;j++,k--){
            int temp=matrix[i][j];
            matrix[i][j]=matrix[i][k];
            matrix[i][k]=temp;
        }
    }
}
```

## [498. 对角线遍历](https://leetcode-cn.com/problems/diagonal-traverse/)

给定一个含有 M x N 个元素的矩阵（M 行，N 列），请以对角线遍历的顺序返回这个矩阵中的所有元素，对角线遍历如下图所示。

**示例:**

```
输入:
[
 [ 1, 2, 3 ],
 [ 4, 5, 6 ],
 [ 7, 8, 9 ]
]

输出:  [1,2,4,7,5,3,6,8,9]
```

 ![JXlfOg.png](https://s1.ax1x.com/2020/05/01/JXlfOg.png)

**说明:**

1. 给定矩阵中的元素总数不会超过 100000 。

**解法一**

因为是先做的下面的那一题，所以我这里直接延用了前面的思路，借助了map额外的空间，其实做复杂了

```go
//借助额外空间
func findDiagonalOrder(matrix [][]int) []int {
    var res []int
    m := len(matrix)
    if matrix == nil || m == 0 {
        return res
    }
    n := len(matrix[0])
    hmap := make(map[int][]int)
    flagRow := false //行开头标志位
    for i, row := range matrix {
        flagColumn := flagRow //列标志位
        for j, num := range row {
            if flagColumn {
                hmap[i+j] = append(hmap[i+j], num)
            } else {
                hmap[i+j] = append([]int{num}, hmap[i+j]...)
            }
            flagColumn = !flagColumn
        }
        flagRow = !flagRow
    }
    for i := 0; i <= m*n; i++ {
        res = append(res, hmap[i]...)
    }
    return res
}
```

**解法二**

模拟，不过是从整体上模拟，比较好的解法，不借助map

```go
//比较好的解法
func findDiagonalOrder(matrix [][]int) []int {
    var res []int
    m := len(matrix)
    if matrix == nil || m == 0 {
        return res
    }
    n := len(matrix[0])
    leftX := 0
    leftY := 0
    rightX := 0
    rightY := 0
    flag := true
    //左右端点沿着矩形边缘移动就行了
    for leftX < m && leftY < n {
        help(matrix, leftX, leftY, rightX, rightY, flag, &res)
        if leftX == m-1 { //左端点到达边界
            leftY++
        } else {
            leftX++
        }
        if rightY == n-1 { //右端点到达边界
            rightX++
        } else {
            rightY++
        }
        flag = !flag //反转
    }
    return res
}

//获取 (lx,ly) 和 (rx,ry)之间的点
func help(matrix [][]int, lx, ly, rx, ry int, flag bool, res *[]int) {
    for lx >= rx && ly <= ry {
        if flag {
            *res = append(*res, matrix[lx][ly])
            lx--
            ly++
        } else {
            *res = append(*res, matrix[rx][ry])
            rx++
            ry--
        }
    }
}
```

## [1424. 对角线遍历 II](https://leetcode-cn.com/problems/diagonal-traverse-ii/)

给你一个列表 `nums` ，里面每一个元素都是一个整数列表。请你依照下面各图的规则，按顺序返回 `nums` 中对角线上的整数。

 

**示例 1：**

![JOOOEt.png](https://s1.ax1x.com/2020/05/01/JOOOEt.png)

```java
输入：nums = [[1,2,3],[4,5,6],[7,8,9]]
输出：[1,4,2,7,5,3,8,6,9]
```

**示例 2：**

![JOXkEq.png](https://s1.ax1x.com/2020/05/01/JOXkEq.png)

```go
输入：nums = [[1,2,3,4,5],[6,7],[8],[9,10,11],[12,13,14,15,16]]
输出：[1,6,2,8,7,3,9,4,12,10,5,13,11,14,15,16]
```

**示例 3：**

```go
输入：nums = [[1,2,3],[4],[5,6,7],[8],[9,10,11]]
输出：[1,4,2,5,3,8,6,9,7,10,11]
```

**示例 4：**

```go
输入：nums = [[1,2,3,4,5,6]]
输出：[1,2,3,4,5,6]
```

**提示：**

- `1 <= nums.length <= 10^5`
- `1 <= nums[i].length <= 10^5`
- `1 <= nums[i][j] <= 10^9`
- `nums` 中最多有 `10^5` 个数字。

**解法一**

186th周赛的t3，还是挺有意思的，这题我拿到的第一想法其实是找一下关系直接排序，但是实际上有更好的方法

```go
func findDiagonalOrder(nums [][]int) []int {
    //最大的行列值
    n := 0
    m := make(map[int][]int)
    for i, row := range nums {
        for j, num := range row {
            //逆序添加
            m[i+j] = append([]int{num}, m[i+j]...)
            n = max(n, i+j)
        }
    }
    //res := make([]int, n)
    var res []int
    for i := 0; i <= n; i++ {
        res = append(res, m[i]...)
    }
    return res
}

func max(a, b int) int {
    if a < b {
        return b
    }
    return a
}
```

上面的解法其实和N皇后里面对行列的处理是一样的，两条对角线，一条行列和相等，一条行列差相等

> 这题我还看到了至少3种不同的方法，有一种把这个数组旋转一下，然后当成二叉树，直接做BFS层次遍历😂，脑洞挺大的

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

_关于节省空间的问题，其实很好理解，去找一个OJ试一下就懂了_

```java
import java.util.*;
public class Main{
    public static void main(String[] args){
        Scanner sc=new Scanner(System.in);
        int N=sc.nextInt();
        int K=sc.nextInt();
        PriorityQueue<Integer> queue=new PriorityQueue<>((a,b)->b-a);
        for(int i=0;i<N;i++){
            int num=sc.nextInt(); //一个个的读入,而不是一起读入
            queue.add(num);
            if(queue.size()>K){
                queue.poll();
            }
        }
        System.out.println(queue.peek());
    }
}
```

**解法三**

其实还有一类做法，利用`快排+二分`的思想，一般也被称为快选

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

**三切分快排优化**

ACWing上交的，wa了好几次，发现是二分写错了，哎，二分真难，其实还可以做一下随机处理

```java
import java.util.*;
public class Main{
    public static void main(String[] args){
        Scanner sc=new Scanner(System.in);
        int N=sc.nextInt();
        int K=sc.nextInt()-1;
        int[] nums=new int[N];
        for(int i=0;i<N;i++) nums[i]=sc.nextInt();
        int left=0,right=N-1;
        while(left<right){
            int[] equ=partition(nums,left,right);
            if(K>equ[1]){
                left=equ[1]+1;
            }else if(K<equ[0]){
                right=equ[0]-1;
            }else{
                System.out.println(nums[equ[0]]);
                return;
            }
        }
        System.out.println(nums[left]);
    }

    public static int[] partition(int[] nums,int left,int right){
        int less=left-1,more=right,base=nums[right];
        int i=left;
        while(i<more){
            if(nums[i]<base){
                swap(nums,++less,i++);
            }else if(nums[i]>base){
                swap(nums,--more,i);
            }else{
                i++;
            }
        }
        //归位
        swap(nums,right,more++);
        return new int[]{less+1,more-1}; //返回等于区域
    }

    public static void swap(int[] nums,int a,int b){
        int temp=nums[a];
        nums[a]=nums[b];
        nums[b]=temp;
    }
}
```

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

## [295. 数据流的中位数](https://leetcode-cn.com/problems/find-median-from-data-stream/)

中位数是有序列表中间的数。如果列表长度是偶数，中位数则是中间两个数的平均值。

例如，

[2,3,4] 的中位数是 3

[2,3] 的中位数是 (2 + 3) / 2 = 2.5

设计一个支持以下两种操作的数据结构：

- void addNum(int num) - 从数据流中添加一个整数到数据结构中。
- double findMedian() - 返回目前所有元素的中位数。

**示例：**

```java
addNum(1)
addNum(2)
findMedian() -> 1.5
addNum(3) 
findMedian() -> 2
```


**进阶:**

- 如果数据流中所有整数都在 0 到 100 范围内，你将如何优化你的算法？
- 如果数据流中 99% 的整数都在 0 到 100 范围内，你将如何优化你的算法？

**解法一**

这题很久之前就听人讲过，不过忘记了，最佳策略就是利用小根堆+大根堆，分别维护前半部分和后半部分的最值

```java
/** initialize your data structure here. */
PriorityQueue<Integer> minQue=null;

PriorityQueue<Integer> maxQue=null;

public MedianFinder295() {
    minQue=new PriorityQueue<>();
    maxQue=new PriorityQueue<>((a,b)->b-a);
}

public void addNum(int num) {
    minQue.add(num);
    maxQue.add(minQue.poll());
    if(minQue.size()<maxQue.size()){
        minQue.add(maxQue.poll());
    }
}

public double findMedian() {
    if(minQue.size()==maxQue.size()){
        return (minQue.peek()+maxQue.peek())/2.0;
    }
    return minQue.peek();
}
```

前半部分用大跟堆，后半部分小根堆，每次将一个堆的最值放到另一个堆中，这样保证了大跟堆的最大值一定小于小根堆的最小值，另外我们还需要保证两个堆的差距不能大于1，这里我将多的放到小根堆中，最后在奇数的时候将小根堆的堆顶弹出就可以了

## [66. 加一](https://leetcode-cn.com/problems/plus-one/)

给定一个由**整数**组成的**非空**数组所表示的非负整数，在该数的基础上加一。

最高位数字存放在数组的首位， 数组中每个元素只存储**单个**数字。

你可以假设除了整数 0 之外，这个整数不会以零开头。

**示例 1:**

```java
输入: [1,2,3]
输出: [1,2,4]
解释: 输入数组表示数字 123。
```

**示例 2:**

```java
输入: [4,3,2,1]
输出: [4,3,2,2]
解释: 输入数组表示数字 4321。
```

**解法一**

模拟进位

```go
func plusOne(digits []int) []int {
    n := len(digits) - 1
    carry := 1
    for n >= 0 {
        digits[n] += carry
        carry = digits[n] / 10
        digits[n] %= 10
        n--
    }
    if carry == 1 {
        digits = append([]int{1}, digits...)
    }
    return digits
}
```

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
//update: 2020.4.16 在web上重新推了一遍
//idx : 0 1 2
//i :   4 5 6
//j :   1 2 3
//   ——————————
//    1 3 6 8 (i+j+1)
//    9 1 2
//  4 5 6
//  ——————————
//0 1 2 3 4 5
//0 5 6 0 8 8  
public String multiply(String num1, String num2) {
    if(num1==null || num2==null) return num1;
    int n1=num1.length(),n2=num2.length();
    int[] res=new int[n1+n2];
    //如果想同时处理进位的话就必须倒推
    for(int i=n1-1;i>=0;i--){
        for(int j=n2-1;j>=0;j--){
            int sum=res[i+j+1]+(num1.charAt(i)-48)*(num2.charAt(j)-48);
            res[i+j+1]=sum%10;
            //res[i+j]会超过10,但是由于我们是倒推的,所以这个会在下一轮进行处理,否则就无法处理了
            res[i+j]+=sum/10; 
        }
    }   
    //n*m位数 乘积应该是 (m+n-1 ~ m+n)位
    //前两个为0一定是0
    if(res[0]==0 && res[1]==0) return "0";
    //去除前导0（最多一个）
    StringBuilder sb=new StringBuilder();
    for(int i=0;i<res.length;i++){
        if(res[i]==0 && i==0)continue;
        sb.append(res[i]);
    }
    return sb.toString();
}
```

## [8. 字符串转换整数 (atoi)](https://leetcode-cn.com/problems/string-to-integer-atoi/)

请你来实现一个 `atoi` 函数，使其能将字符串转换成整数。

首先，该函数会根据需要丢弃无用的开头空格字符，直到寻找到第一个非空格的字符为止。

当我们寻找到的第一个非空字符为正或者负号时，则将该符号与之后面尽可能多的连续数字组合起来，作为该整数的正负号；假如第一个非空字符是数字，则直接将其与之后连续的数字字符组合起来，形成整数。

该字符串除了有效的整数部分之后也可能会存在多余的字符，这些字符可以被忽略，它们对于函数不应该造成影响。

注意：假如该字符串中的第一个非空格字符不是一个有效整数字符、字符串为空或字符串仅包含空白字符时，则你的函数不需要进行转换。

在任何情况下，若函数不能进行有效的转换时，请返回 0。

**说明：**

假设我们的环境只能存储 32 位大小的有符号整数，那么其数值范围为 `[−231,  231 − 1]`。如果数值超过这个范围，请返回  `INT_MAX (231 − 1)` 或 `INT_MIN (−231)` 

**示例 1:**

```java
输入: "42"
输出: 42
```


**示例 2:**

```java
输入: "   -42"
输出: -42
解释: 第一个非空白字符为 '-', 它是一个负号。
     我们尽可能将负号与后面所有连续出现的数字组合起来，最后得到 -42 。
```


**示例 3:**

```java
输入: "4193 with words"
输出: 4193
解释: 转换截止于数字 '3' ，因为它的下一个字符不为数字。
```

**解法一**

之前一直很排斥这道题，知道这次朋友阿里面试问了这道题。。。

```java
public int myAtoi(String str) {
    if(str==null || str.length()<=0) return 0;
    int MAX=Integer.MAX_VALUE,MIN=Integer.MIN_VALUE;
    int res=0,index=0;
    //过滤空格
    while(index<str.length()&&str.charAt(index)==' ')index++;
    if(index==str.length()) return 0;
    //取正负号
    char firstChar=str.charAt(index);
    boolean positive=true;
    if(!isDigit(firstChar)){
        if(firstChar!='+'&&firstChar!='-') return 0;
        index++;
        positive = firstChar!='-';
    }
    //正负数的边界
    int limit=positive?-MAX:MIN;
    //过滤0
    while(index<str.length()&&str.charAt(index)=='0')index++;
    //取每一位,在非字符截止
    while(index<str.length()&&isDigit(str.charAt(index))){
        int digit=str.charAt(index++)-'0';
        if(res<(limit+digit)/10){
            return positive?MAX:MIN;
        }
        //这里的res>=limit
        res=res*10-digit;
    }
    //if(index!=str.length()) return 0; //中途遇到非数字(也是合法的)
    return positive?-res:res;
}

public boolean isDigit(char c){
    return c>='0' && c<='9';
}
```

参考了`Integer.parseInt(String s, int radix)`方法对边界的处理方式, **用负数来表示正负数的边界**

1. 这样正数的边界就是`-INT_MAX`,负数是`INT_MIN`

2. 然后我们同样也用负数来保存结果, `res=res\*10-digit`

3. 我们需要保证这个值是在`INT`范围内的, 也就是 res*10-digit>=limit (负边界)

4. 所以我们需要对`res`做判断,但是直接判断可能会溢出,所以进行移项,变换为 `res<(limit+digit)/10`

5. 最后如果是正数就返回 `-res`,负数就返回`res`

还是十分巧妙的 👏👏

## [1071. 字符串的最大公因子](https://leetcode-cn.com/problems/greatest-common-divisor-of-strings/)

对于字符串 S 和 T，只有在 S = T + ... + T（T 与自身连接 1 次或多次）时，我们才认定 “T 能除尽 S”。

返回最长字符串 X，要求满足 X 能除尽 str1 且 X 能除尽 str2。

**示例 1：**

```java
输入：str1 = "ABCABC", str2 = "ABC"
输出："ABC"
```

**示例 2：**

```java
输入：str1 = "ABABAB", str2 = "ABAB"
输出："AB"
```


**示例 3：**

```java
输入：str1 = "LEET", str2 = "CODE"
输出：""
```

**解法一**

首先想到的方法，其实也是根据辗转相除法来的（这种好像叫更相减损术 ?）

```java
//6 4 gcd(6,4)=gcd(4,2)=gcd(2,0) return 2
public String gcdOfStrings(String str1, String str2) {
    if(str1.equals(str2)){
        return str1;
    }
    int index1=0,index2=0;
    //用减法替代除法求余数
    while(str1.length()>=str2.length() && index1<str1.length() && index2<str2.length()){
        if(str1.charAt(index1)!=str2.charAt(index2)) return "";
        index2++;
        index1++;
    }
    //gcd(str2,余数)
    return gcdOfStrings(str2,str1.substring(index1,str1.length()));
}
```
**解法二**

数学方法，比较巧妙

```java
public String gcdOfStrings(String str1, String str2) {
    if(!(str1+str2).equals(str2+str1)){
        return "";
    }
    return str1.substring(gcd(str1.length(),str2.length()));
}

public int gcd(int a,int b){
    if(b==0) return a;
    return gcd(b,a%b);
}
```
第一个条件充分性的证明还是有点不太理解，不过后面的最大公因子的长度就是str1和str2长度倒是可以通过反证来证明出来，这里直接copy题解[大佬](https://leetcode-cn.com/problems/greatest-common-divisor-of-strings/solution/tan-tan-zheng-ming-wei-shi-yao-liang-zi-fu-chuan-c/) 的证明

假设两字符串的长度分别为l1, l2, 他们的最大公约数是k。
现已知道两字符串存在最大公因子（第一行代码），假设该字串的长度为`k'`。

下面开始反证，
若`k' < k`，而根据题意`k'`也为`l1, l2` 的公约数，则`k'`必能被`k`整除，这说明我们可以将该字串的长度扩充到`k`，同时保持它仍然为`str1`和`str2`的公因子，所以这种情况下这个长度为`k`的公因子就不是最大公因子。
若`k' > k`， 根据题意`k'`为`l1`, `l2` 的公约数， 而k为最大公约数，而这时出现了一个比最大公约数还大的公约数，这是矛盾的，所以这种情况也是不可能的。

综合以上，最大公因子的长度必然等于两串长度的最大公约数。

## [914. 卡牌分组](https://leetcode-cn.com/problems/x-of-a-kind-in-a-deck-of-cards/)

给定一副牌，每张牌上都写着一个整数。

此时，你需要选定一个数字 X，使我们可以将整副牌按下述规则分成 1 组或更多组：

- 每组都有 X 张牌。
- 组内所有的牌上都写着相同的整数。

仅当你可选的 X >= 2 时返回 true。

**示例 1：**

```java
输入：[1,2,3,4,4,3,2,1]
输出：true
解释：可行的分组是 [1,1]，[2,2]，[3,3]，[4,4]
```

**示例 2：**

```java
输入：[1,1,1,2,2,2,3,3]
输出：false
解释：没有满足要求的分组。
```

**示例 3：**

```java
输入：[1]
输出：false
解释：没有满足要求的分组。
```

**示例 4：**

```java
输入：[1,1]
输出：true
解释：可行的分组是 [1,1]
```

**示例 5：**

```java
输入：[1,1,2,2,2,2]
输出：true
解释：可行的分组是 [1,1]，[2,2]，[2,2]
```

**提示：**

- `1 <= deck.length <= 10000`
- `0 <= deck[i] < 10000`

**解法一**

```java
public boolean hasGroupsSizeX(int[] deck) {
    HashMap<Integer,Integer> map=new HashMap<>();
    for(int i=0;i<deck.length;i++){
        map.put(deck[i],map.getOrDefault(deck[i],0)+1);
    }
    int g=-1;
    for (Integer key:map.keySet()) {
        int freq=map.get(key);
        if(g==-1) {
            g=freq;
        }else{
            if(freq<2) return false;
            g=gcd(freq,g);
        }
    }
    return g>=2;
}

public int gcd(int a,int b){
    if(b==0) return a;
    return gcd(b,a%b);
}
```
wa一次就知道咋做了，一开始以为只要所有元素出现次数可以整除就行了，wa了之后就意识到只要求一个最大公约数就可以了

**解法二**

给定了范围，直接用数组模拟

```java
public int gcd(int a,int b){
    if(b==0) return a;
    return gcd(b,a%b);
}

public boolean hasGroupsSizeX2(int[] deck) {
    int[] hash=new int[10001];
    for(int i=0;i<deck.length;i++){
        hash[deck[i]]++;
    }
    int g=-1;
    for (int i=0;i<hash.length-1;i++) {
        if(hash[i]!=0){
            if(hash[i]<2) return false;
            g= g!=-1?gcd(g,hash[i]):hash[i];
            if(g==1) return false; //优化,提前终止
        }
    }
    return g>=2;
}
```
## [165. 比较版本号](https://leetcode-cn.com/problems/compare-version-numbers/)

比较两个版本号 version1 和 version2。
如果 `version1 > version2` 返回 1，如果 `version1 < version2` 返回 -1， 除此之外返回 0。

你可以假设版本字符串非空，并且只包含数字和 . 字符。

 . 字符不代表小数点，而是用于分隔数字序列。

例如，`2.5`  不是“两个半”，也不是“差一半到三”，而是第二版中的第五个小版本。

你可以假设版本号的每一级的默认修订版号为 0。例如，版本号 3.4 的第一级（大版本）和第二级（小版本）修订号分别为 3 和 4。其第三级和第四级修订号均为 0。

**示例 1:**

```java
输入: version1 = "0.1", version2 = "1.1"
输出: -1
```

**示例 2:**

```java
输入: version1 = "1.0.1", version2 = "1"
输出: 1
```


**示例 3:**

```java
输入: version1 = "7.5.2.4", version2 = "7.5.3"
输出: -1
```


**示例 4：**

```java
输入：version1 = "1.01", version2 = "1.001"
输出：0
解释：忽略前导零，“01” 和 “001” 表示相同的数字 “1”。
```


**示例 5：**

```java
输入：version1 = "1.0", version2 = "1.0.0"
输出：0
解释：version1 没有第三级修订号，这意味着它的第三级修订号默认为 “0”。
```

**提示：**

1. 版本字符串由以点 （.） 分隔的数字字符串组成。这个数字字符串可能有前导零。
2. 版本字符串不以点开始或结束，并且其中不会有两个连续的点。

**解法一**

貌似笔试喜欢出这题，挺简单的，用java分割的时候要注意 `"."` 是一个正则表达式，匹配任意单个字符，我们如果要将它看作一个普通字符需要加上双斜线`"\\."`

```java
public int compareVersion(String version1, String version2) {
    String[] v1=version1.split("\\.");
    String[] v2=version2.split("\\.");
    int len1=v1.length,len2=v2.length;
    int i=0,j=0;
    while(i<len1 || j<len2) {
        int a=Integer.valueOf(i<len1?v1[i++]:"0");
        int b=Integer.valueOf(j<len2?v2[j++]:"0");
        if (a<b) {
            return -1;
        }else if (a>b){
            return 1;
        }
    }
    return 0;
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

## [409. 最长回文串](https://leetcode-cn.com/problems/longest-palindrome/)

给定一个包含大写字母和小写字母的字符串，找到通过这些字母构造成的最长的回文串。

在构造过程中，请注意区分大小写。比如 `"Aa"` 不能当做一个回文字符串。

**注意:**
假设字符串的长度不会超过 1010。

**示例 1:**

```java
输入:
"abccccdd"

输出:
7

解释:
我们可以构造的最长的回文串是"dccaccd", 它的长度是 7。
```

**解法一**

和上面的题目名字一样，但是请注意审题！！！

```java
public int longestPalindrome(String s) {
    if(s==null || s.length()<=0) return 0;
    int[] hash=new int[128];
    for(int i=0;i<s.length();i++){
        hash[s.charAt(i)]++;
    }
    //Arrays.sort(hash);
    int res=0;boolean flag=false;
    for(int i=hash.length-1;i>=0;i--){
        if(hash[i]!=0){
            if(hash[i]%2==0){
                res+=hash[i]; //偶数直接加
            }else{
                flag=true;
                res+=(hash[i]-1); //奇数减一再加
            }
        }
    }
    return flag?res+1:res;
}
```

## [680. 验证回文字符串 Ⅱ](https://leetcode-cn.com/problems/valid-palindrome-ii/)

给定一个非空字符串 `s`，**最多**删除一个字符。判断是否能成为回文字符串。

**示例 1:**

```java
输入: "aba"
输出: True
```

**示例 2:**

```java
输入: "abca"
输出: True
解释: 你可以删除c字符。
```

**注意:**

1. 字符串只包含从 a-z 的小写字母。字符串的最大长度是50000。

**解法一**

模拟就行了

```java
public boolean validPalindrome(String s) {
    if(s==null || s.length()<=0) return true;
    int left=0,right=s.length()-1;
    while(left<right){
        if(s.charAt(left)==s.charAt(right)){
            left++;right--;
        }else{
            return valid(s,left+1,right) || valid(s,left,right-1);
        }
    }
    return true;
}

public boolean valid(String s,int left,int right){
    while(left<=right){
        if(s.charAt(left)==s.charAt(right)){
            left++;right--;
        }else{
            return false;
        }
    }
    return true;
}
```

## [1332. 删除回文子序列](https://leetcode-cn.com/problems/remove-palindromic-subsequences/)

给你一个字符串 s，它仅由字母 'a' 和 'b' 组成。每一次删除操作都可以从 s 中删除一个回文 **子序列**。

返回删除给定字符串中所有字符（字符串为空）的最小删除次数。

「子序列」定义：如果一个字符串可以通过删除原字符串某些字符而不改变原字符顺序得到，那么这个字符串就是原字符串的一个子序列。

「回文」定义：如果一个字符串向后和向前读是一致的，那么这个字符串就是一个回文。 

**示例 1：**

```java
输入：s = "ababa"
输出：1
解释：字符串本身就是回文序列，只需要删除一次。
```

**示例 2：**

```java
输入：s = "abb"
输出：2
解释："abb" -> "bb" -> "". 
先删除回文子序列 "a"，然后再删除 "bb"。
```


**示例 3：**

```java
输入：s = "baabb"
输出：2
解释："baabb" -> "b" -> "". 
先删除回文子序列 "baab"，然后再删除 "b"。
```


**示例 4：**

```java
输入：s = ""
输出：0
```

**提示：**

- `0 <= s.length <= 1000`
- `s` 仅包含字母 'a'  和 'b'

**解法一**

某一次周赛的第一题，乍一看最长回文子串？最长回文序列？这题当时还是难到了不少人，我那次没参加，后台听说了第一题是个坑，然后这里审题的时候就很注意，没踩坑😁

```java
public int removePalindromeSub(String s) {
    if (s==null || s.length()<=0) {
        return 0;
    }
    for(int i=0,j=s.length()-1;i<=j;i++,j--){
        if (s.charAt(i)!=s.charAt(j)) {
            return 2;
        }
    }
    return 1;
}
```

题目说了只有两个字母a和b，而且要删除的是**回文子序列**，这样一说就清楚了，这才是简单题的水准呐~还是挺有意思的，脑筋急转弯hahaha


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

## [1333. 餐厅过滤器](https://leetcode-cn.com/problems/filter-restaurants-by-vegan-friendly-price-and-distance/)

给你一个餐馆信息数组 `restaurants`，其中  `restaurants[i] = [idi, ratingi, veganFriendlyi, pricei, distancei]`。你必须使用以下三个过滤器来过滤这些餐馆信息。

其中素食者友好过滤器 `veganFriendly` 的值可以为 `true` 或者 `false`，如果为 `true` 就意味着你应该只包括 `veganFriendlyi` 为 `true` 的餐馆，为 `false` 则意味着可以包括任何餐馆。此外，我们还有最大价格 `maxPrice` 和最大距离 `maxDistance` 两个过滤器，它们分别考虑餐厅的价格因素和距离因素的最大值。

过滤后返回餐馆的 `id`，按照 `rating` 从高到低排序。如果 `rating` 相同，那么按 `id` 从高到低排序。简单起见， `veganFriendlyi` 和 `veganFriendly` 为 `true` 时取值为 1，为 `false` 时，取值为 0 。

**示例一**

```java
输入：restaurants = [[1,4,1,40,10],[2,8,0,50,5],[3,8,1,30,4],[4,10,0,10,3],[5,1,1,15,1]], veganFriendly = 1, maxPrice = 50, maxDistance = 10
输出：[3,1,5] 
解释： 
这些餐馆为：
餐馆 1 [id=1, rating=4, veganFriendly=1, price=40, distance=10]
餐馆 2 [id=2, rating=8, veganFriendly=0, price=50, distance=5]
餐馆 3 [id=3, rating=8, veganFriendly=1, price=30, distance=4]
餐馆 4 [id=4, rating=10, veganFriendly=0, price=10, distance=3]
餐馆 5 [id=5, rating=1, veganFriendly=1, price=15, distance=1] 
在按照 veganFriendly = 1, maxPrice = 50 和 maxDistance = 10 进行过滤后，我们得到了餐馆 3, 餐馆 1 和 餐馆 5（按评分从高到低排序）。 
```

**示例 2：**

```java
输入：restaurants = [[1,4,1,40,10],[2,8,0,50,5],[3,8,1,30,4],[4,10,0,10,3],[5,1,1,15,1]], veganFriendly = 0, maxPrice = 50, maxDistance = 10
输出：[4,3,2,1,5]
解释：餐馆与示例 1 相同，但在 veganFriendly = 0 的过滤条件下，应该考虑所有餐馆。
```


**示例 3：**

```java
输入：restaurants = [[1,4,1,40,10],[2,8,0,50,5],[3,8,1,30,4],[4,10,0,10,3],[5,1,1,15,1]], veganFriendly = 0, maxPrice = 30, maxDistance = 3
输出：[4,5]
```

**提示：**

- `1 <= restaurants.length <= 10^4`
- `restaurants[i].length == 5`
- `1 <= idi, ratingi, pricei, distancei <= 10^5`
- `1 <= maxPrice, maxDistance <= 10^5`
- `veganFriendlyi` 和 `veganFriendly` 的值为 0 或 1 。
- 所有 `idi` 各不相同。

**解法一**

看到这个题，javaer不用stream可太可惜了hahaha

```java
public List<Integer> filterRestaurants(int[][] restaurants, int veganFriendly, int maxPrice, int maxDistance) {
    return Stream.of(restaurants)
        .filter(r-> (veganFriendly==1?r[2]==veganFriendly:true) && r[4]<=maxDistance && r[3]<=maxPrice)
        .sorted((r1,r2)->r1[1]!=r2[1]?r2[1]-r1[1]:r2[0]-r1[0])
        .map(r->r[0])
        .collect(Collectors.toList());
}
```

## [5313. 时钟指针的夹角](https://leetcode-cn.com/problems/angle-between-hands-of-a-clock/)

给你两个数 `hour` 和 `minutes` 。请你返回在时钟上，由给定时间的时针和分针组成的较小角的角度（60 单位制）。

这题就懒得copy了，19场双周赛的第三题，不应该是mid题的。。。

**解法一**

```java
public double angleClock(int hour, int minutes) {
    double m=minutes/60.0 * 360;
    double h=((hour/12.0)*360)%360 + 30*minutes/60.0;
    return Math.min(Math.abs(m-h),360-Math.abs(m-h));
}
```

化简一下是 **h时m分的夹角为：5.5m-30h**

## [5169. 日期之间隔几天](https://leetcode-cn.com/problems/number-of-days-between-two-dates/)

请你编写一个程序来计算两个日期之间隔了多少天。

日期以字符串形式给出，格式为 `YYYY-MM-DD`。

**解法一**

177周赛的T1

```java
import java.time.*;
import java.time.temporal.ChronoUnit;
class Solution {
    public int daysBetweenDates(String date1, String date2) {
        return (int)Math.abs(LocalDate.parse(date1).until(LocalDate.parse(date2),ChronoUnit.DAYS));
    }
}
```

做LeetCode少数导包了的题🤣

**解法二**

copy大佬的原生解法

```java

private int[] months = new int[]{0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};

public int daysBetweenDates(String date1, String date2) {
    String[] d1 = date1.split("-");
    String[] d2 = date2.split("-");
    int day1 = getYears(Integer.valueOf(d1[0]) - 1) + getMonths(Integer.valueOf(d1[0]), Integer.valueOf(d1[1]) - 1) + Integer.valueOf(d1[2]);
    int day2 = getYears(Integer.valueOf(d2[0]) - 1) + getMonths(Integer.valueOf(d2[0]), Integer.valueOf(d2[1]) - 1) + Integer.valueOf(d2[2]);
    return Math.abs(day1 - day2);
}

private int getYears(int year) {
    int sum = (year - 1971) * 365;
    for (int i = 1971; i <= year; ++i) {
        if (isRun(i)) {
            ++sum;
        }
    }
    return sum;
}

private int getMonths(int year, int month) {
    int sum = 0;
    for (int i = 1; i <= month; ++i) {
        sum += months[i];
    }
    if (isRun(year) && month >= 2) {
        ++sum;
    }
    return sum;
}

private boolean isRun(int year) {
    return (year % 4 == 0 && year % 100 != 0) || (year % 400 == 0);
}
```

## [557. 反转字符串中的单词 III](https://leetcode-cn.com/problems/reverse-words-in-a-string-iii/)

给定一个字符串，你需要反转字符串中每个单词的字符顺序，同时仍保留空格和单词的初始顺序。

**示例 1:**

```java
输入: "Let's take LeetCode contest"
输出: "s'teL ekat edoCteeL tsetnoc" 
```

**注意：**在字符串中，每个单词由单个空格分隔，并且字符串中不会有任何额外的空格

**解法一**

原生的做法

```java
public String reverseWords(String s) {
    s+=" ";//统一操作
    char[] cs=s.toCharArray();
    int start=0;
    for (int i=0;i<cs.length;i++) {
        if (cs[i]==' ') {
            reverse(cs,start,i-1);
            start=i+1;
        }
    }
    return new String(cs,0,cs.length-1);
}

public void reverse(char[] s,int left,int right){
    for (int i=left,j=right;i<j;i++,j--) {
        char temp=s[i];
        s[i]=s[j];
        s[j]=temp;
    }
}
```

## [151. 翻转字符串里的单词](https://leetcode-cn.com/problems/reverse-words-in-a-string/)

给定一个字符串，逐个翻转字符串中的每个单词。

**示例 1：**

```java
输入: "the sky is blue"
输出: "blue is sky the"
```


**示例 2：**

```java
输入: "  hello world!  "
输出: "world! hello"
解释: 输入字符串可以在前面或者后面包含多余的空格，但是反转后的字符不能包括。
```


**示例 3：**

```java
输入: "a good   example"
输出: "example good a"
解释: 如果两个单词间有多余的空格，将反转后单词间的空格减少到只含一个。
```

**说明：**

- 无空格字符构成一个单词。
- 输入字符串可以在前面或者后面包含多余的空格，但是反转后的字符不能包括。
- 如果两个单词间有多余的空格，将反转后单词间的空格减少到只含一个。

**进阶：**

请选用 C 语言的用户尝试使用 O(1) 额外空间复杂度的原地解法。

**解法一**

```java
public String reverseWords(String s) {
    if(s==null || s.length()<=0) return "";
    Deque<String> stack=new ArrayDeque<>();
    for(int i=0;i<s.length();i++){
        if(s.charAt(i)==' '){
            continue;
        }
        int j=i;
        while(i<s.length() && s.charAt(i)!=' '){
            i++;
        }
        stack.push(s.substring(j,i));
    }
    if(stack.isEmpty()) return "";
    StringBuilder sb=new StringBuilder();
    sb.append(stack.pop());
    while(!stack.isEmpty()){
        sb.append(" "+stack.pop());
    }
    return sb.toString();
}
```
比较原生的做法，最开始写的借助split等方法的，比较简单就不多说了

**解法二**

原地的做法，比较繁琐，后面有时间再来实现下

1. 翻转整个字符
2. 翻转单个字符
3. 去除多余空格

## [面试题 01.06. 字符串压缩](https://leetcode-cn.com/problems/compress-string-lcci/)

字符串压缩。利用字符重复出现的次数，编写一种方法，实现基本的字符串压缩功能。比如，字符串aabcccccaaa会变为a2b1c5a3。若“压缩”后的字符串没有变短，则返回原先的字符串。你可以假设字符串中只包含大小写英文字母（a至z）。

case就不粘了

**解法一**

直接写就行了

```java
public String compressString(String S) {
    StringBuilder sb=new StringBuilder();
    int index=0;
    while(index<S.length()){
        sb.append(S.charAt(index));
        int r=1;
        while(index<S.length()-1&&S.charAt(index)==S.charAt(index+1)){
            ++index;
            r++;
        }
        sb.append(r);
        index++;
    }
    return sb.length()<S.length()?sb.toString():S;
}
```
## [238. 除自身以外数组的乘积](https://leetcode-cn.com/problems/product-of-array-except-self/)

给定长度为 n 的整数数组 nums，其中 n > 1，返回输出数组 output ，其中 output[i] 等于 nums 中除 nums[i] 之外其余各元素的乘积。

**示例:**

```java
输入: [1,2,3,4]
输出: [24,12,8,6]
说明: 请不要使用除法，且在 O(n) 时间复杂度内完成此题。
```

**进阶：**
你可以在常数空间复杂度内完成这个题目吗？（ 出于对空间复杂度分析的目的，输出数组不被视为额外空间。）

**解法一**

还行，独立的想到了解法，没啥好说的

```java
public int[] productExceptSelf(int[] nums) {
    if (nums==null || nums.length<=0) {
        return new int[0];
    }
    int[] left=new int[nums.length+1]; 
    left[0]=1;//left[i]: [0 ~ i-1]的累积
    int[] right=new int[nums.length+1];
    right[nums.length]=1; //right[i]: [i ~ nums.length-1]的累积
    for (int i=1;i<=nums.length;i++) {
        left[i]=left[i-1]*nums[i-1];
    }
    for (int i=nums.length-1;i>=0;i--) {
        right[i]=right[i+1]*nums[i];
    }
    int[] res=new int[nums.length];
    //1 2 3
    res[0]=right[1];
    for (int i=1;i<nums.length;i++) {
        res[i]=left[i]*right[i+1];
    }
    return res;
}
```

**解法二**

O(1)进阶版有时间再来补

## [5341. 最后 K 个数的乘积](https://leetcode-cn.com/problems/product-of-the-last-k-numbers/)

请你实现一个「数字乘积类」`ProductOfNumbers`，要求支持下述两种方法：

1. add(int num)

- 将数字 num 添加到当前数字列表的最后面。

2. getProduct(int k)

- 返回当前数字列表中，最后 k 个数字的乘积。
- 你可以假设当前列表中始终 至少 包含 k 个数字。
- 题目数据保证：任何时候，任一连续数字序列的乘积都在 32-bit 整数范围内，不会溢出。

**示例：**

```java
输入：
["ProductOfNumbers","add","add","add","add","add","getProduct","getProduct","getProduct","add","getProduct"]
[[],[3],[0],[2],[5],[4],[2],[3],[4],[8],[2]]

输出：
[null,null,null,null,null,null,20,40,0,null,32]

解释：
ProductOfNumbers productOfNumbers = new ProductOfNumbers();
productOfNumbers.add(3);        // [3]
productOfNumbers.add(0);        // [3,0]
productOfNumbers.add(2);        // [3,0,2]
productOfNumbers.add(5);        // [3,0,2,5]
productOfNumbers.add(4);        // [3,0,2,5,4]
productOfNumbers.getProduct(2); // 返回 20 。最后 2 个数字的乘积是 5 * 4 = 20
productOfNumbers.getProduct(3); // 返回 40 。最后 3 个数字的乘积是 2 * 5 * 4 = 40
productOfNumbers.getProduct(4); // 返回  0 。最后 4 个数字的乘积是 0 * 2 * 5 * 4 = 0
productOfNumbers.add(8);        // [3,0,2,5,4,8]
productOfNumbers.getProduct(2); // 返回 32 。最后 2 个数字的乘积是 4 * 8 = 32 
```

**提示：**

```java
add 和 getProduct 两种操作加起来总共不会超过 40000 次。
0 <= num <= 100
1 <= k <= 40000
```

**解法一**

176周赛的第二题，思路倒是很容易想到，维护一个前缀积，然后用全积除以对应k位置的就行了，但是细节的处理出了大问题haha~

```java
LinkedList<Integer> product=null;

public ProductOfNumbers() {
    product=new LinkedList<>();
    product.add(1);
}

public void add(int num) {
    if(num==0){ //重新构建
        product=new LinkedList<>();
        product.add(1);
    }else{
        product.add(num*product.getLast());
    }
}

//1| 1 0 2 3  k=3
public int getProduct(int k) {
    if(k>=product.size()){
        return 0;
    }
    return product.getLast()/product.get(product.size()-k-1);
}
```
一开始维护了所有的前缀积，结果后面的case过不了，出现了除0异常，很显然把0换成1，后面的case大了之后累乘的结果太大了，溢出为0了。

上面代码的处理方式是参考了大佬的解法，遇到0的时候就直接重置队列，最后如果k大于队列长度说明这个序列肯定是包含了0，直接返回0就可以了

## [面试题64. 求1+2+…+n](https://leetcode-cn.com/problems/qiu-12n-lcof/)

求 `1+2+...+n` ，要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）。

**示例 1：**

```java
输入: n = 3
输出: 6
```


**示例 2：**

```java
输入: n = 9
输出: 45
```

**限制：**

- 1 <= n <= 10000

**解法一**

我一开始想的是把 `n*(n-1)/2`  展开，变成平方，用pow代替，除2用移位代替，但是想了想感觉pow底层应该也是用了乘

所以还是得用递归，但是递归必须有出口，这里的关键就是怎么停止

```java
public int sumNums(int n) {
    int sum = n;
    //逻辑与短路
    boolean ans = (n > 0) && ((sum += sumNums(n - 1)) > 0);
    return sum;
}
```

## [470. 用 Rand7() 实现 Rand10()](https://leetcode-cn.com/problems/implement-rand10-using-rand7/)

已有方法 rand7 可生成 1 到 7 范围内的均匀随机整数，试写一个方法 rand10 生成 1 到 10 范围内的均匀随机整数。

不要使用系统的 Math.random() 方法。

**解法一**

拒绝采样，两次Rand7()拒绝大于40的情况

```java
public int rand10() {
    int a,b,idx;
    do{
        a=rand7();
        b=rand7();
        idx=a+(b-1)*7;
    }while(idx>40);
    return 1+(idx-1)%10;
}
```

概率论里面学过的，都忘了

![38nhYn.png](https://s2.ax1x.com/2020/02/24/38nhYn.png)

期望其实还可以更低，这里后面有时间再来研究

## [384. 打乱数组](https://leetcode-cn.com/problems/shuffle-an-array/)

打乱一个没有重复元素的数组。

**示例:**

```java
// 以数字集合 1, 2 和 3 初始化数组。
int[] nums = {1,2,3};
Solution solution = new Solution(nums);

// 打乱数组 [1,2,3] 并返回结果。任何 [1,2,3]的排列返回的概率应该相同。
solution.shuffle();

// 重设数组到它的初始状态[1,2,3]。
solution.reset();

// 随机返回数组[1,2,3]打乱后的结果。
solution.shuffle();
```

**解法一**

洗牌算法，确保至少会出现`N!`种情况，且每种情况出现的概率是相同的

```java
int[] origin=null;

int[] nums=null;

Random random=new Random();

public Solution(int[] nums) {
    this.nums=nums;
    origin=nums.clone();
}

/** Resets the array to its original configuration and return it. */
public int[] reset() {
    return origin;
}

/** Returns a random shuffling of the array. */
public int[] shuffle() {
    for (int i=nums.length-1;i>=0;i--) {
        //从尾部开始这样对于Java会简单一点点
        int rand=(int)(random.nextInt(i+1)); //随机【0,i】的元素
        swap(nums,i,rand);
    }
    return nums;
}

public void swap(int[] nums,int a,int b){
    int temp=nums[a];
    nums[a]=nums[b];
    nums[b]=temp;
}
```

## [面试题61. 扑克牌中的顺子](https://leetcode-cn.com/problems/bu-ke-pai-zhong-de-shun-zi-lcof/)

从扑克牌中随机抽5张牌，判断是不是一个顺子，即这5张牌是不是连续的。2～10为数字本身，A为1，J为11，Q为12，K为13，而大、小王为 0 ，可以看成任意数字。A 不能视为 14。

**示例 1:**

```java
输入: [1,2,3,4,5]
输出: True
```

**示例 2:**

```java
输入: [0,0,1,2,5]
输出: True
```

**限制：**

数组长度为 5 

数组的数取值为 [0, 13] .

**解法一**

只有5张牌，先排除对子，然后求最大和最小的牌面之差就行了，小于等于4就肯定是顺子

```java
public boolean isStraight(int[] nums) {
    int[] bucket=new int[14];
    for(int i=0;i<5;i++){
        bucket[nums[i]]++;
        //有非0的对子,直接false
        if(nums[i]!=0 && bucket[nums[i]] >1 ){
            return false;
        }
    }
    //记录起手牌和最大牌
    int start=-1,end=-1;
    for(int i=1,j=13;end==-1||start==-1;i++,j--){
        if(bucket[i]==1 && start==-1) start=i;
        if(bucket[j]==1 && end==-1) end=j;
    }
    //小于等于4就行,多的用0补
    return end-start<=4;
}
```
代码可以优化成一个循环内

```java
//缩减成一个循环
public boolean isStraight(int[] nums) {
    int[] bucket=new int[14];
    int min=14,max=-1;
    for(int i=0;i<nums.length;i++){
        if(nums[i]==0) continue;
        if(bucket[nums[i]]==1) return false;
        bucket[nums[i]]++;
        min=Math.min(min,nums[i]);
        max=Math.max(max,nums[i]);
    }
    return max-min<=4;
}
```
## [1103. 分糖果 II](https://leetcode-cn.com/problems/distribute-candies-to-people/)

排排坐，分糖果。

我们买了一些糖果 candies，打算把它们分给排好队的 n = num_people 个小朋友。

给第一个小朋友 1 颗糖果，第二个小朋友 2 颗，依此类推，直到给最后一个小朋友 n 颗糖果。

然后，我们再回到队伍的起点，给第一个小朋友 n + 1 颗糖果，第二个小朋友 n + 2 颗，依此类推，直到给最后一个小朋友 2 * n 颗糖果。

重复上述过程（每次都比上一次多给出一颗糖果，当到达队伍终点后再次从队伍起点开始），直到我们分完所有的糖果。注意，就算我们手中的剩下糖果数不够（不比前一次发出的糖果多），这些糖果也会全部发给当前的小朋友。

返回一个长度为 num_people、元素之和为 candies 的数组，以表示糖果的最终分发情况（即 ans[i] 表示第 i 个小朋友分到的糖果数）。

**示例 1：**

```java
输入：candies = 7, num_people = 4
输出：[1,2,3,1]
解释：
第一次，ans[0] += 1，数组变为 [1,0,0,0]。
第二次，ans[1] += 2，数组变为 [1,2,0,0]。
第三次，ans[2] += 3，数组变为 [1,2,3,0]。
第四次，ans[3] += 1（因为此时只剩下 1 颗糖果），最终数组变为 [1,2,3,1]。
```

**示例 2：**

```java
输入：candies = 10, num_people = 3
输出：[5,2,3]
解释：
第一次，ans[0] += 1，数组变为 [1,0,0]。
第二次，ans[1] += 2，数组变为 [1,2,0]。
第三次，ans[2] += 3，数组变为 [1,2,3]。
第四次，ans[0] += 4，最终数组变为 [5,2,3]。
```

**提示：**

- `1 <= candies <= 10^9`
- `1 <= num_people <= 1000`

**解法一**

```java
//暴力模拟
public int[] distributeCandies(int candies, int num_people) {
    int[] res=new int[num_people];
    int index=0;
    for (int i=1;candies>0;i++) {
        res[index%num_people]+=Math.min(candies,i);
        candies-=i;
        index++;
    }
    return res;
}
```
数学的方法：每个人得到的糖果分批次的都是一个等差数列，可以通过求和公式直接算，过程有点繁琐，后面有时间再来研究

## [1013. 将数组分成和相等的三个部分](https://leetcode-cn.com/problems/partition-array-into-three-parts-with-equal-sum/) 

  给你一个整数数组 A，只有可以将其划分为三个和相等的非空部分时才返回 true，否则返回 false。

形式上，如果可以找出索引 i+1 < j 且满足 (A[0] + A[1] + ... + A[i] == A[i+1] + A[i+2] + ... + A[j-1] == A[j] + A[j-1] + ... + A[A.length - 1]) 就可以将数组三等分。

 **示例 1：**

```java
输出：[0,2,1,-6,6,-7,9,1,2,0,1]
输出：true
解释：0 + 2 + 1 = -6 + 6 - 7 + 9 + 1 = 2 + 0 + 1
```


**示例 2：**

```java
输入：[0,2,1,-6,6,7,9,-1,2,0,1]
输出：false
```


**示例 3：**

```java
输入：[3,3,6,5,-2,2,5,1,-9,4]
输出：true
解释：3 + 3 = 6 = 5 - 2 + 2 + 5 + 1 - 9 + 4
```

**提示：**

1. 3 <= A.length <= 50000
2. -10^4 <= A[i] <= 10^4

**解法一**

这个解法算是被群友误导了的，群里面有人说这题是前缀和，然后我就往哪个上面想的，其实没必要

```java
public boolean canThreePartsEqualSum(int[] A) {
    int[] preSum=new int[A.length+1];
    preSum[0]=0;
    for(int i=1;i<=A.length;i++){
        preSum[i]=preSum[i-1]+A[i-1];
    }
    for(int i=1;i<preSum.length-2;i++){ //保证剩下2个
        if(preSum[A.length]-preSum[i]==preSum[i]*2){
            for(int j=i+1;j<preSum.length-1;j++){ //保证剩下1个
                if(preSum[A.length]-preSum[i]==(preSum[j]-preSum[i])*2){
                    //System.out.println(i+" "+j);
                    return true;
                }
            }
            return false;
        }
    }
    return false;
}
```
**解法二**

这才是正常的思路

```java
public boolean canThreePartsEqualSum(int[] A) {
    int sum=0;
    for(int i=0;i<A.length;i++) sum+=A[i];
    if(sum%3!=0) return false;
    int count=0,tempSum=0;
    //i到达A.length-1保证有第3段,否则有可能target=0 只分为两段就没了
    for(int i=0;i<A.length-1;i++){ 
        tempSum+=A[i];
        if(tempSum==sum/3){
            ++count;
            if(count==2) return true;
            tempSum=0;
        }
    }
    return false;
}
```
这题的WA点就是 sum=0的时候，有可能按照划分的逻辑只将数组划分为两段，所以要注意边界

## [调整数组顺序使奇数位于偶数前面（牛客）](https://www.nowcoder.com/practice/beb5aa231adc45b2a5dcc5b62c93f593?tpId=13&&tqId=11166&rp=2&ru=/activity/oj&qru=/ta/coding-interviews/question-ranking)

输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于数组的后半部分，并保证奇数和奇数，偶数和偶数之间的相对位置不变。

**解法一**

其实就是稳定的排序，插入，冒泡，归并都可以

```java
public void reOrderArray(int [] array) {
    for(int i=0;i<array.length;i++){
        for(int j=i;j>=1;j--){
            if(array[j]%2==1 && array[j-1]%2==0){
                swap(array,j,j-1);
            }
        }
    }
}

public void swap(int[] array,int a,int b){
    int temp=array[a];
    array[a]=array[b];
    array[b]=temp;
}
```
## [836. 矩形重叠](https://leetcode-cn.com/problems/rectangle-overlap/)

矩形以列表 `[x1, y1, x2, y2]` 的形式表示，其中 `(x1, y1)` 为左下角的坐标，`(x2, y2)` 是右上角的坐标。

如果相交的面积为正，则称两矩形重叠。需要明确的是，只在角或边接触的两个矩形不构成重叠。

给出两个矩形，判断它们是否重叠并返回结果。

**示例 1：**

```java
输入：rec1 = [0,0,2,2], rec2 = [1,1,3,3]
输出：true
```

**示例 2：**

```java
输入：rec1 = [0,0,1,1], rec2 = [1,0,2,1]
输出：false
```

**提示：**

1. 两个矩形 `rec1` 和 `rec2` 都以含有四个整数的列表的形式给出。
2. 矩形中的所有坐标都处于 `-10^9` 和 `10^9` 之间。
3. `x` 轴默认指向右，`y` 轴默认指向上。
4. 你可以仅考虑矩形是正放的情况。

**解法一**

憨憨解法，最后还被一个大case越界给卡了

```java
public boolean isRectangleOverlap(int[] rec1, int[] rec2) {
    int x=Math.max(rec2[2]-rec1[0],rec1[2]-rec2[0]);
    int y=Math.max(rec2[3]-rec1[1],rec1[3]-rec2[1]);
    long maxX=((long)rec1[2]-(long)rec1[0]+(long)rec2[2]-(long)rec2[0]);
    long maxY=((long)rec1[3]-(long)rec1[1]+(long)rec2[3]-(long)rec2[1]);
    return x<maxX && y <maxY;
}
```
**解法二**

```java
public boolean isRectangleOverlap(int[] rec1, int[] rec2) {
    return !(rec1[3]<=rec2[1]||rec2[3]<=rec1[1]||rec1[2]<=rec2[0]||rec2[2]<=rec1[0]);
}
```
## [892. 三维形体的表面积](https://leetcode-cn.com/problems/surface-area-of-3d-shapes/)

在 N * N 的网格上，我们放置一些 1 * 1 * 1  的立方体。

每个值 v = grid[i][j] 表示 v 个正方体叠放在对应单元格 (i, j) 上。

请你返回最终形体的表面积。

**示例 1：**

```java
输入：[[2]]
输出：10
```

**示例 2：**

```java
输入：[[1,2],[3,4]]
输出：34
```


**示例 3：**

```java
输入：[[1,0],[0,2]]
输出：16
```

**示例 4：**

```java
输入：[[1,1,1],[1,0,1],[1,1,1]]
输出：32
```


**示例 5：**

```java
输入：[[2,2,2],[2,1,2],[2,2,2]]
输出：46
```

**提示：**

- `1 <= N <= 50`
- `0 <= grid[i][j] <= 50` 

**解法一**

做加法

```java
//加法思路
public int surfaceArea(int[][] grid) {
    if(grid==null || grid.length<=0 ) return 0;
    int N=grid.length;
    int res=0;
    for (int i=0;i<N;i++) {
        for (int j=0;j<N;j++) {
            //正面,背面暴露的面积
            res+= Math.max(j<N-1?grid[i][j+1]-grid[i][j]:grid[i][j],0);
            res+= Math.max(j>0?grid[i][j-1]-grid[i][j]:grid[i][j],0);
            //左和右边暴露的面积
            res+= Math.max(i<N-1?grid[i+1][j]-grid[i][j]:grid[i][j],0);
            res+= Math.max(i>0?grid[i-1][j]-grid[i][j]:grid[i][j],0);
            //上和下的面积
            res+= grid[i][j]!=0?2:0;
        }
    }
    return res;
}
```
我太蠢了，开始直接分别算6个面，然后发现有坑，又去算坑的面积。。。结果就陷进去了

**解法二**

巧妙的减法思路，算贴合的时候的重合的面积

```java
//巧妙的减法思路
public int surfaceArea(int[][] grid) {
    if(grid==null || grid.length<=0 ) return 0;
    int N=grid.length;
    int x=0,y=0,count=0;
    for (int i=0;i<N;i++) {
        for (int j=0;j<N;j++) {
            if(grid[i][j]!=0){
                x+=grid[i][j]-1;
                count+=grid[i][j];
            }
            if(i>=1 && grid[i-1][j]!=0){
                y+=Math.min(grid[i][j],grid[i-1][j]);
            }
            if(j>=1 && grid[i][j-1]!=0){
                y+=Math.min(grid[i][j],grid[i][j-1]);
            }
        }
    }
    return count*6-2*x-2*y;
}
```
## [463. 岛屿的周长](https://leetcode-cn.com/problems/island-perimeter/)

给定一个包含 0 和 1 的二维网格地图，其中 1 表示陆地 0 表示水域。

网格中的格子水平和垂直方向相连（对角线方向不相连）。整个网格被水完全包围，但其中恰好有一个岛屿（或者说，一个或多个表示陆地的格子相连组成的岛屿）。

岛屿中没有“湖”（“湖” 指水域在岛屿内部且不和岛屿周围的水相连）。格子是边长为 1 的正方形。网格为长方形，且宽度和高度均不超过 100 。计算这个岛屿的周长。

**示例 :**

```java
输入:
[[0,1,0,0],
 [1,1,1,0],
 [0,1,0,0],
 [1,1,0,0]]

输出: 16
```

**解法一**

偶然翻到的题，发现和上面的是一样的

```java
//和892类似的解法，简化版
public int islandPerimeter(int[][] grid) {
    int count=0;
    int left=0,up=0;
    for(int i=0;i<grid.length;i++){
        for(int j=0;j<grid[0].length;j++){
            if(grid[i][j]==1){
                if(i-1>=0 && grid[i-1][j]==1){
                    up++;
                }
                if(j-1>=0 && grid[i][j-1]==1){
                    left++;
                }
                count++;
            }
        }
    }
    return count*4-(up+left)*2;
}
```
> 看到题解区有大佬dfs的，通过方格的变化统计有效的边，比如从1->0就可以增加一条边，而从1->边界又可以增加一条边，还是挺秒的

## [999. 车的可用捕获量](https://leetcode-cn.com/problems/available-captures-for-rook/)

题目太长，不想复制了，模拟题，题目意思搞清楚就行了

```java
public int numRookCaptures(char[][] board) {
    int[][] direction=new int[][]{{-1,0},{0,-1},{1,0},{0,1}};
    int res=0;
    for (int i=0;i<board.length;i++) {
        for (int j=0;j<board[0].length;j++) {
            if(board[i][j]=='R'){
                for (int k=0;k<4;k++) {
                    int nx=i+direction[k][0];
                    int ny=j+direction[k][1];
                    while(nx>=0 && nx<board.length && ny>=0 && ny<board[0].length){
                        if(board[nx][ny]=='B'){
                            break;
                        }
                        if(board[nx][ny]=='p'){
                            res++;
                            break;
                        }
                        nx+=direction[k][0];
                        ny+=direction[k][1];
                    }
                }
                return res;
            }
        }
    }
    return 0;
}
```
## [289. 生命游戏](https://leetcode-cn.com/problems/game-of-life/)

根据 百度百科 ，生命游戏，简称为生命，是英国数学家约翰·何顿·康威在 1970 年发明的细胞自动机。

给定一个包含 m × n 个格子的面板，每一个格子都可以看成是一个细胞。每个细胞都具有一个初始状态：1 即为活细胞（live），或 0 即为死细胞（dead）。每个细胞与其八个相邻位置（水平，垂直，对角线）的细胞都遵循以下四条生存定律：

1. 如果活细胞周围八个位置的活细胞数少于两个，则该位置活细胞死亡；

2. 如果活细胞周围八个位置有两个或三个活细胞，则该位置活细胞仍然存活；

3. 如果活细胞周围八个位置有超过三个活细胞，则该位置活细胞死亡；

4. 如果死细胞周围正好有三个活细胞，则该位置死细胞复活；

根据当前状态，写一个函数来计算面板上所有细胞的下一个（一次更新后的）状态。下一个状态是通过将上述规则同时应用于当前状态下的每个细胞所形成的，其中细胞的出生和死亡是同时发生的。

**示例：**

```java
输入： 
[
  [0,1,0],
  [0,0,1],
  [1,1,1],
  [0,0,0]
]
输出：
[
  [0,0,0],
  [1,0,1],
  [0,1,1],
  [0,1,0]
]
```

**进阶：**

- 你可以使用原地算法解决本题吗？请注意，面板上所有格子需要同时被更新：你不能先更新某些格子，然后使用它们的更新后的值再更新其他格子。
- 本题中，我们使用二维数组来表示面板。原则上，面板是无限的，但当活细胞侵占了面板边界时会造成问题。你将如何解决这些问题？

**解法一**

bugfree

```java
int[][] dir={{1,0},{0,1},{-1,0},{0,-1},{1,1},{-1,-1},{-1,1},{1,-1}};

public void gameOfLife(int[][] board) {
    if(board==null || board.length<=0) return;
    int m=board.length,n=board[0].length;
    boolean[][] change=new boolean[m][n];
    for(int i=0;i<m;i++){
        for(int j=0;j<n;j++){
            if(board[i][j]==0 && aliveCell(board,i,j,change)==3){
                board[i][j]=1;
                change[i][j]=true;
            }else if(board[i][j]==1){
                int alive=aliveCell(board,i,j,change);
                if(alive<2 || alive>3){
                    board[i][j]=0;
                    change[i][j]=true;
                }
            }
        }
    }
}

public int aliveCell(int[][] board,int x,int y,boolean[][] change){
    int alive=0;
    for(int k=0;k<dir.length;k++){
        int nx=x+dir[k][0];
        int ny=y+dir[k][1];
        if(valid(board,nx,ny)&&(board[nx][ny]==1 && !change[nx][ny] || (board[nx][ny]==0 && change[nx][ny]))){
            alive++;
        }
    }
    return alive;
}

public boolean valid(final int[][] board,int x,int y){
    return x>=0 && x<board.length && y>=0 && y<board[0].length;
}
```
我理解的原地就是在原数组上做修改，但是并没有说不能用额外空间吧。。。但是看了评论区大佬们都不是这样写的，都是用的位运算，用int空的位保存状态，最后移位，懒得写了，感觉没啥意思，水题

## [204. 计数质数](https://leetcode-cn.com/problems/count-primes/)

统计所有小于非负整数 *n* 的质数的数量。

**示例:**

```java
输入: 10
输出: 4
解释: 小于 10 的质数一共有 4 个, 它们是 2, 3, 5, 7 。
```

**解法一**

[厄拉多塞筛法](https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes) 简称埃式筛

```java
public int countPrimes(int n) {
    boolean[] prime=new boolean[n];
    //为了不那么别扭
    Arrays.fill(prime,true);
    for(int i=2;i*i<n;i++){
        if(prime[i]){
            //从i*i开始,i*(i-1)已经被前面的统计了
            for(int j=i*i;j<n;j+=i){
                prime[j]=false;
            }
        }
    }
    int res=0;
    for(int i=2;i<prime.length;i++){
        if(prime[i]) res++;
    }
    return res;
}
```
## [171. Excel表列序号](https://leetcode-cn.com/problems/excel-sheet-column-number/)

给定一个Excel表格中的列名称，返回其相应的列序号。

例如，

    A -> 1
    B -> 2
    C -> 3
    ...
    Z -> 26
    AA -> 27
    AB -> 28 
    ...
**示例 1:**

```java
输入: "A"
输出: 1
```

**示例 2:**

```java
输入: "AB"
输出: 28
```

**示例 3:**

```java
输入: "ZY"
输出: 701
```

**解法一**

朋友作业帮面试问了这道题，其实就是进制的转换，写了个回转的

```java
//26进制转10进制
public int titleToNumber(String s) {
    if(s==null || s.length()<=0) return 0;
    int res=0,n=s.length();
    int temp=1;
    for (int i=n-1;i>=0;i--) {
        res+=(s.charAt(i)-'A'+1)*temp;
        temp*=26;
    }
    return res;
}

//10进制转26进制
public String numberToTitle(int s) {
    StringBuilder res=new StringBuilder();
    while(s!=0){
        //这个s-1要注意啊woc
        res.append((char)((s-1)%26+65));
        s=(s-1)/26;
    }
    return res.reverse().toString();
}
```
## [168. Excel表列名称](https://leetcode-cn.com/problems/excel-sheet-column-title/)

给定一个正整数，返回它在 Excel 表中相对应的列名称。

例如，

```java
1 -> A
2 -> B
3 -> C
...
26 -> Z
27 -> AA
28 -> AB 
...
```
**示例 1:**

```java
输入: 1
输出: "A"
```

**示例 2:**

```java
输入: 28
输出: "AB"
```

**示例 3:**

```java
输入: 701
输出: "ZY"
```

**解法一**

```java
public String numberToTitle(int s) {
    StringBuilder res=new StringBuilder();
    while(s!=0){
        //这个s-1要注意啊woc
        res.append((char)((s-1)%26+65));
        s=(s-1)/26;
    }
    return res.reverse().toString();
}
```

go写法

```go
func convertToTitle(n int) string {
    var res string
    for n>0{
        res=string((n-1)%26+'A')+res
        n=(n-1)/26
    }
    return res
}
```

## [724. 寻找数组的中心索引](https://leetcode-cn.com/problems/find-pivot-index/)

给定一个整数类型的数组 `nums`，请编写一个能够返回数组**“中心索引”**的方法。

我们是这样定义数组**中心索引**的：数组中心索引的左侧所有元素相加的和等于右侧所有元素相加的和。

如果数组不存在中心索引，那么我们应该返回 -1。如果数组有多个中心索引，那么我们应该返回最靠近左边的那一个。

**示例 1:**

```java
输入: 
nums = [1, 7, 3, 6, 5, 6]
输出: 3
解释: 
索引3 (nums[3] = 6) 的左侧数之和(1 + 7 + 3 = 11)，与右侧数之和(5 + 6 = 11)相等。
同时, 3 也是第一个符合要求的中心索引。
```

**示例 2:**

```java
输入: 
nums = [1, 2, 3]
输出: -1
解释: 
数组中不存在满足此条件的中心索引。
```

**说明:**

- `nums` 的长度范围为 `[0, 10000]`。
- 任何一个 `nums[i]` 将会是一个范围在 `[-1000, 1000]`的整数。

**解法一**

唉，不知道为啥，直接写了个前缀后缀和判断的，很暴力直白的思路，咋就想不到简单的思路？

```go
func pivotIndex(nums []int) int {
    //前缀和，后缀和
    n:=len(nums)
    if n==0{
        return -1
    }
    pre:=make([]int,n+1) //i之前的元素和,不包含i
    pre[0]=0
    last:=make([]int,n+1) //(i-1)之后元素和,不包含(i-1)
    last[n]=0
    for i,j := 1,n-1;i<=n && j>=0; i,j = i+1,j-1 {
        //这里其实只要代入值验证第一次的转移是正确的就行了,不用考虑太多
        pre[i]=pre[i-1]+nums[i-1]
        last[j]=last[j+1]+nums[j]
    }
    for i:=0;i<n;i++{
        //联系上面数组的定义思考这里的判断
        if pre[i]==last[i+1]{
            return i;
        }
    }
    return -1
}
```

**解法二**

简单的思路，脑子瓦特了一下没想到

```go
func pivotIndex(nums []int) int {
    n:=len(nums)
    if n==0{
        return -1
    }
    sum:=0
    for _,num:= range nums{
        sum+=num
    }
    temp:=0 //包含了边界0
    for i:=0;i<n;i++{
        if temp*2+nums[i]==sum{
            return i
        }
        temp+=nums[i]
    }
    return -1
}
```

## [945. 使数组唯一的最小增量](https://leetcode-cn.com/problems/minimum-increment-to-make-array-unique/)

给定整数数组 A，每次 *move* 操作将会选择任意 `A[i]`，并将其递增 `1`。

返回使 `A` 中的每个值都是唯一的最少操作次数。

**示例 1:**

```java
输入：[1,2,2]
输出：1
解释：经过一次 move 操作，数组将变为 [1, 2, 3]。
```

**示例 2:**

```java
输入：[3,2,1,2,1,7]
输出：6
解释：经过 6 次 move 操作，数组将变为 [3, 4, 1, 2, 5, 7]。
可以看出 5 次或 5 次以下的 move 操作是不能让数组的每个值唯一的。
```

**提示：**

1. `0 <= A.length <= 40000`
2. `0 <= A[i] < 40000`

**解法一**

之前写了，没记录，这次PDD笔试考了这题

```java
public int minIncrementForUnique(int[] A) {
    Arrays.sort(A);
    int move=0;
    for(int i=1;i<A.length;i++){
        if(A[i]<=A[i-1]){
            move+=A[i-1]-A[i];
            A[i]=A[i-1]+1;
        }
    }
    return move;
}
```

暴力的解法很好想，首先肯定要排序，然后遇到小于等于前面的时候就`move+1`，直到不相等，但是这里是可以优化的，一次次的加没有啥意义，可以直接一步到位直接从`A[i]`增加到`A[i-1]+1`

> 这题还有一些方法优化，首先是排序可以用桶排序，然后还可以用并查集（比较麻烦），或者也有数学分析找规律的方法

## [面试题05. 替换空格](https://leetcode-cn.com/problems/ti-huan-kong-ge-lcof/)

请实现一个函数，把字符串 `s` 中的每个空格替换成"%20"。

**示例 1：**

```java
输入：s = "We are happy."
输出："We%20are%20happy."
```

**限制：**

```java
0 <= s 的长度 <= 10000
```

**解法一**

这题的标准做法

```java
public String replaceSpace(String s) {
    char[] res=new char[s.length()*3];
    int idx=0;
    for(int i=0;i<s.length();i++){
        if(s.charAt(i)==' '){
            res[idx++]='%';
            res[idx++]='2';
            res[idx++]='0';
        }else{
            res[idx++]=s.charAt(i);
        }
    }
    return new String(res,0,idx);
}
```

**解法二**

原题是要求O(1)空间的，这里虽然无法做到，但是可以模拟下

```java
//原题目的要求应该是在O(1)空间下,但是Java的String是不可变的
//所以不可能O(1),我们需要改一下函数签名
public String replaceSpace(/*StringBuilder*/ String ss) {
    StringBuilder s=new StringBuilder(ss); //这里是为了验证
    int oldLen=s.length();
    for (int i=0;i<oldLen;i++) {
        if(s.charAt(i)==' ') s.append("xx"); //扩充字符长度
    }
    int newLen=s.length();
    //逆序,避免覆盖
    int i=oldLen-1,j=newLen-1;
    while(i>=0){
        char c=s.charAt(i--);
        if(c==' '){
            s.setCharAt(j--,'0');
            s.setCharAt(j--,'2');
            s.setCharAt(j--,'%');
        }else{
            s.setCharAt(j--,c);
        }
    }
    return s.toString();
}
```

## [面试题45. 把数组排成最小的数](https://leetcode-cn.com/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/)

输入一个非负整数数组，把数组里所有数字拼接起来排成一个数，打印能拼接出的所有数字中最小的一个。

**示例 1:**

```java
输入: [10,2]
输出: "102"
```
**示例 2:**

```java
输入: [3,30,34,5,9]
输出: "3033459"
```

**提示:**
- `0 < nums.length <= 100`

**说明:**

输出结果可能非常大，所以你需要返回一个字符串而不是整数；拼接起来的数字可能会有前导 0，最后结果不需要去掉前导 0

**解法一**

一开始贼sb，想了一大堆有的没的😅
![UTOOLS1592838604659.png](https://upload.cc/i1/2020/06/22/zOQcTL.png)
```java
public String minNumber(int[] nums) {
    String[] strs=new String[nums.length];
    for(int i=0;i<nums.length;i++) strs[i]=nums[i]+"";
    Arrays.sort(strs,(a,b)->(a+b).compareTo(b+a));
    StringBuilder sb=new StringBuilder();
    for(String i:strs) sb.append(i);
    return sb.toString();
}
```
虽然绕了一大圈，所幸还是自己做出来了，但是为什么这样就是对的呢？其实这里我也是想当然了，严谨的应该对这个排序规则的传递性进行证明，也就是`xy>yx && yz>zy ==> xz>xz ?`这个成立，排序的结果才是对的，这里我就不copy了，证明也不是很难，大家可以去原题[题解区](https://leetcode-cn.com/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/solution/mian-shi-ti-45-ba-shu-zu-pai-cheng-zui-xiao-de-s-4/378553)看看

## [179. 最大数](https://leetcode-cn.com/problems/largest-number/)

给定一组非负整数，重新排列它们的顺序使之组成一个最大的整数。

**示例 1:**

```java
输入: [10,2]
输出: 210
```
**示例 2:**

```java
输入: [3,30,34,5,9]
输出: 9534330
说明: 输出结果可能非常大，所以你需要返回一个字符串而不是整数。
```
**解法一**

和上一题一摸一样，借机学习下golang的自定义排序
```golang
type StringSlice []string

func (p StringSlice) Len() int           { return len(p) }
func (p StringSlice) Less(i, j int) bool { return p[i]+p[j] > p[j]+p[i] }
func (p StringSlice) Swap(i, j int)      { p[i], p[j] = p[j], p[i] }

func largestNumber(nums []int) string {
    strs := make([]string, len(nums))
    for i, n := range nums {
        strs[i] = strconv.Itoa(n)
    }
    sort.Sort(StringSlice(strs))
    //return strings.Join(strs,"") 要去前导0...
    // var res = ""
    // var idx = 0
    // for idx < len(strs)-1 && strs[idx] == "0" {
    //     idx++
    // }
    // for idx < len(strs) {
    //     res += strs[idx]
    //     idx++
    // }
    res := strings.Join(strs, "")
    if res[0] == '0' { //第一个为0肯定就全部是0了...前面的写法明显没动脑子
        return "0"
    }
    return res
}
```
看了大佬们的提交记录发现go 1.8在sort包中引入了

`func Slice(slice interface{}, less func(i, j int) bool)`

通过这个就不用很麻烦的去实现3个函数了，只需要实现`Less`比较器就可以了（其实这才是正常的做法，其他语言中也都是类似的，其他两个`len`和`swap`感觉意义不大，一般不会改这两个函数，完全可以自动生成）
> 看评论区又看到一个很好的[反证的思路](https://leetcode-cn.com/problems/largest-number/solution/zui-da-shu-bi-jiao-gui-ze-chuan-di-xing-yi-ji-suan/344160)，大致意思就是：假设存在序列`"...ab..."`为最大数，且不满足该排序规则`ab>ba`，也就是说`ab<ba`，那我们交换序列中ab的位置变为`"...ba..."`很明显`"...ba..."`>`"...ab..."`，与假设矛盾，所以最大数一定满足该排序规则

##  _二进制_

> 得找时间分开了，越来越卡了，Typora快顶不住了😂

## [136. 只出现一次的数字](https://leetcode-cn.com/problems/single-number/)

给定一个**非空整数**数组，除了某个元素只出现一次以外，**其余每个元素均出现两次**。找出那个只出现了一次的元素。

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
## [260. 只出现一次的数字 III](https://leetcode-cn.com/problems/single-number-iii/)

给定一个整数数组 nums，其中恰好有两个元素只出现一次，其余所有元素均出现两次。 找出只出现一次的那两个元素。

**示例 :**

```java
输入: [1,2,1,3,2,5]
输出: [3,5]
```


**注意：**

1. 结果输出的顺序并不重要，对于上面的例子， [5, 3] 也是正确答案。
2. 你的算法应该具有线性时间复杂度。你能否仅使用常数空间复杂度来实现？

**解法一**

根据异或的结果`xor`，讲整个数组划分为两组，分别包含a，b这两个唯一的元素

```java
public int[] singleNumber(int[] nums) {
    if(nums==null || nums.length<=0) return new int[0];
    int xor=nums[0];
    for (int i=1;i<nums.length;i++) {
        xor^=nums[i];
    }
    int index=0; //ab二进制不同的index
    while((xor&1)==0){
        xor>>>=1;
        index++;
    }
    //a,b在index位置的二进制位不同，异或结果为1，然后我们就可以根据这个不同点，将整个数组按照这个划分为两部分
    //这样相同的数肯定会被分配到同一组，问题就转换成了136，这样我们再分别异或就能得到最终的a,b
    int a=0,b=0;
    for (int i=0;i<nums.length;i++) {
        if(((nums[i]>>>index)&1)==1){ //根据index位置的元素0，1来划分为两个数组
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
输入: [3,0,1]
输出: 2
```

**示例 2:**

```java
输入: [9,6,4,2,3,5,7,0,1]
输出: 8
```

**说明:**
你的算法应具有线性时间复杂度。你能否仅使用额外常数空间来实现?

**解法一**

比较直接的思路，求全序列的和然后相减就ok

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

**示例:**

```java
输入: x = 1, y = 4

输出: 2

解释:
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

## [191. 位1的个数](https://leetcode-cn.com/problems/number-of-1-bits/)

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

**进阶:**
如果多次调用这个函数，你将如何优化你的算法？ 

**解法一**

比较精妙的解法，`n&(n-1)` 就是将二进制最右边的1变为0，这样逐步的&最后数组就会变为0，我们统计下次数就是1的数量

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
        if((n&1)==1)//判断末位是不是1
            count++;
        n>>>=1; //无符号右移,避免添高位添1死循环
    }
    return count;
}
```

> 注意右移的操作，都需要使用无符号右移，不然负数右移高位补1后就错了

## [338. 比特位计数](https://leetcode-cn.com/problems/counting-bits/)

给定一个非负整数 num。对于 0 ≤ i ≤ num 范围中的每个数字 i ，计算其二进制数中的 1 的数目并将它们作为数组返回。

**示例 1:**

```java
输入: 2
输出: [0,1,1]
```


**示例 2:**

```java
输入: 5
输出: [0,1,1,2,1,2]
```


**进阶:**

- 给出时间复杂度为O(n*sizeof(integer))的解答非常容易。但你可以在线性时间O(n)内用一趟扫描做到吗？
- 要求算法的空间复杂度为O(n)。
- 你能进一步完善解法吗？要求在C++或任何其他语言中不使用任何内置函数（如 C++ 中的 __builtin_popcount）来执行此操作。 

**解法一**

这样的题肯定是直接上进阶的啦，有动态规划的意思

```java
public int[] countBits(int num) {
    int[] res=new int[num+1];
    for (int i=1;i<=num;i++) {
        //如果i二进制以0结尾,那么i>>1的countBit和i一样,i&1=0（>>1就是/2）
        //反之,那么i>>1的比i会少1个,i&1=1
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
        //i&(i-1)会去掉最右边的1
        res[i]=res[i&(i-1)]+1;
    }
    return res;
}
```

## [231. 2的幂](https://leetcode-cn.com/problems/power-of-two/)

给定一个整数，编写一个函数来判断它是否是 2 的幂次方。

**示例 1:**

```java
输入: 1
输出: true
解释: 20 = 1
```


**示例 2:**

```java
输入: 16
输出: true
解释: 24 = 16
```


**示例 3:**

```java
输入: 218
输出: false
```

**解法一**

应该是最快AC的题吧hahaha

```java
public boolean isPowerOfTwo(int n) {
    return n>0 && (n&(n-1))==0;
}
```

## [458. 可怜的小猪](https://leetcode-cn.com/problems/poor-pigs/)

有 1000 只水桶，其中有且只有一桶装的含有毒药，其余装的都是水。它们从外观看起来都一样。如果小猪喝了毒药，它会在 15 分钟内死去。

问题来了，如果需要你在一小时内，弄清楚哪只水桶含有毒药，你最少需要多少只猪？

回答这个问题，并为下列的进阶问题编写一个通用算法。

**进阶:**

假设有 `n` 只水桶，猪饮水中毒后会在 `m` 分钟内死亡，你需要多少猪（`x`）就能在 `p` 分钟内找出 “**有毒**” 水桶？这 `n` 只水桶里有且仅有一只有毒的桶。

**提示：**

1. 可以允许小猪同时饮用任意数量的桶中的水，并且该过程不需要时间。
2. 小猪喝完水后，必须有 *m* 分钟的**冷却时间**。在这段时间里，只允许观察，而不允许继续饮水。
3. 任何给定的桶都可以无限次采样（无限数量的猪）。

**解法一**

[参考题解区](https://leetcode-cn.com/problems/poor-pigs/solution/hua-jie-suan-fa-458-ke-lian-de-xiao-zhu-by-guanpen/) 我就不搬运了，简单来说就是看每个小猪能表示几进制的状态，比如题目中说的是15分钟死亡，一个小时时间，那么每只小猪可以吃4次药，可以检测出5瓶药，所以`x`只猪就能检测`state^x`个桶

```java
public int poorPigs(int buckets, int minutesToDie, int minutesToTest) {
    int state=minutesToTest/minutesToDie+1;
    return (int)Math.ceil(Math.log(buckets)/Math.log(state));
}
```

具体的做法：x只🐖，每只🐖都和对应......

> 两只猪二维的表格可以解决，三只猪三维的坐标可以解决，如果是4只猪，5只猪呢？具体的如何测量，这里我还没有点没想清楚😐

## [371. 两整数之和](https://leetcode-cn.com/problems/sum-of-two-integers/)

**不使用**运算符 `+` 和 `-` ，计算两整数 `a` 、`b` 之和。

**示例 1:**

```java
输入: a = 1, b = 2
输出: 3
```

**示例 2:**

```java
输入: a = -2, b = 3
输出: 1
```

**解法一**

两数之和 = 两数不进位和+两数进位和 ，一开始也没搞懂这个式子，后来在10进制上想了下就明白了

比如 998 + 99 = 987+110 = 97 + 1000 = 1097 ，其实就是把加法拆开来看，把进位的数和对应的数位和分开计算，而进位的数和两数的不进位和都可以通过位运算算出来，进而不使用`+/-`计算两数之和

```go
func getSum(a int, b int) int {
    if b==0{
        return a
    }
    //a^b:不进位和 a&b<<1: 进位数和(进位是进到前一位，所以需要左移，类比十进制就清楚了)
    return getSum(a^b,(a&b)<<1) 
}
```
