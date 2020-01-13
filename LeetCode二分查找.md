---
title: 
   LeetCode二分查找
tags: 
  [LeetCode,二分]
categories:
	[算法]
date: 2019/12/6
cover: http://static.imlgw.top/5.jpg
---

>  从 [数组专题](http://imlgw.top/2019/05/04/leetcode-shu-zu/) 中抽取出来的 

## [704. 二分查找](https://leetcode-cn.com/problems/binary-search/)

给定一个 n 个元素有序的（升序）整型数组 nums 和一个目标值 target  ，写一个函数搜索 nums 中的 target，如果目标值存在返回下标，否则返回 -1。

**示例 1:**

```java
输入: nums = [-1,0,3,5,9,12], target = 9
输出: 4
解释: 9 出现在 nums 中并且下标为 4
```


**示例 2:**

```java
输入: nums = [-1,0,3,5,9,12], target = 2
输出: -1
解释: 2 不存在 nums 中因此返回 -1
```

**提示：**

- 你可以假设 nums 中的所有元素是不重复的
- n 将在 [1, 10000]之间
- nums 的每个元素都将在 [-9999, 9999]之间

**解法一**

比较经典的二分

```java
public int search(int[] nums, int target) {
    int left=0,right=nums.length;
    while(left<right){
        int mid=left+(right-left)/2;
        if(nums[mid]<target){
            left=mid+1;
        }else if(nums[mid] > target){
            right=mid;
        }else{
            return mid;
        }
    }
    return -1;
}
```
**解法二**

按照板子来的二分，最后需要后处理一下不存在的情况

```java
//模板二分
public int search(int[] nums, int target) {
    int left=0,right=nums.length;
    while(left<right){
        int mid=left+(right-left)/2;
        if(nums[mid]<target){ //排除mid
            left=mid+1;
        }else{
            right=mid;
        }
    }
    return left!=nums.length&&nums[left]==target?left:-1;
}
```
## [35. 搜索插入位置](https://leetcode-cn.com/problems/search-insert-position/)

给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。

你可以假设数组中无重复元素。

**示例 1:**

```java
输入: [1,3,5,6], 5
输出: 2
```

**示例 2:**

```java
输入: [1,3,5,6], 2
输出: 1
```

**示例 3:**

```java
输入: [1,3,5,6], 7
输出: 4
```

**示例 4:**

```java
输入: [1,3,5,6], 0
输出: 0
```

**解法一**

```java
public static  int searchInsert(int[] nums, int target) {
    int len=nums.length;
    int lo=0,hi=len-1;
    if(nums[hi]<target){
        return len;
    }
    if(nums[0]>=target){
        return 0;
    }
    while(lo<=hi){
        int mid=lo+(hi-lo)/2;
        if(nums[mid]<target){
            lo=mid+1;
            if(lo<nums.length&&nums[lo]>target){
                return mid+1;
            }
        } else if(nums[mid]>target){
            hi=mid-1;
            if(hi>=0&&nums[hi]<target){
                return hi+1;
            }
        } else{
            return mid;
            //相等的情况，直接返回这个index
        }
    }
    return 0;
    //到不了这里
}
```

其实上面的代码写的并不好，写的很奇怪。

```java
public int searchInsert(int[] nums, int target) {
    int len=nums.length;
    int lo=0,hi=len-1;
    if(nums[hi]<target){
        return len;
    }
    if(nums[0]>=target){
        return 0;
    }
    while(lo<=hi){
        int mid=lo+(hi-lo)/2;
        if(nums[mid]<target){
            lo=mid+1;
        } else if(nums[mid]>target){
            hi=mid-1;
        } else{
            return mid;
        }
    }
    return lo;
    //return hi+1
}
```

如果熟悉二分的过程，其实最后返回`lo`或者`hi+1`就可以了。

## [153. 寻找旋转排序数组中的最小值](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/)

假设按照升序排序的数组在预先未知的某个点上进行了旋转。

( 例如，数组 `[0,1,2,4,5,6,7]` 可能变为 `[4,5,6,7,0,1,2]` )。

请找出其中最小的元素。

你可以假设数组中不存在重复元素。

**示例 1:**

```java
输入: [3,4,5,1,2]
输出: 1
```


**示例 2:**

```java
输入: [4,5,6,7,0,1,2]
输出: 0
```

**解法一**

把最开始写的拉跨解法也放上来吧

```java
public int findMin(int[] nums) {
    if (nums==null||nums.length<=0) {
        return 0;
    }
    if (nums.length==1||nums[0]<nums[nums.length-1]) {
        return nums[0];
    }
    int left=1,right=nums.length-1;
    while(left<right){
        int mid=left+(right-left)/2+1;
        if (nums[mid]>nums[mid-1]) {
            if (nums[mid]>nums[0]) {
                left=mid;
            }else{
                right=mid-1;
            }
        }else{
            return nums[mid];
        }
    }
    return nums[left];
}
```
说实话，我都不知道咋对的。。。

**解法二**

模板解法，还是模板写起来清晰舒服

```java
public int findMin(int[] nums) {
    if (nums==null||nums.length<=0) {
        return 0;
    }
    int left=0,right=nums.length-1;
    while(left<right){
        int mid=left+(right-left)/2;
        if (nums[mid]>nums[right]) { //排除mid的分支
            left=mid+1;
        }else{
            right=mid;
        }
    }
    return nums[left];
}
```
需要注意要和右边界比较，和左边界比较不一定正确

比如 `1 2 3 4 5` 和`2 3 4 5 1` 两个的中点都大于左边界，但是你无法确定此时应该如果缩短区间，除非做特判，但是那样就麻烦了

## [154. 寻找旋转排序数组中的最小值 II](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array-ii/) 

假设按照升序排序的数组在预先未知的某个点上进行了旋转。

( 例如，数组 `[0,1,2,4,5,6,7]` 可能变为 `[4,5,6,7,0,1,2]` )。

请找出其中最小的元素。

注意数组中可能存在重复的元素。

**示例 1：**

```java
输入: [1,3,5]
输出: 1
```

**示例 2：**

```java
输入: [2,2,2,0,1]
输出: 0
```


**说明：**

- 这道题是 寻找旋转排序数组中的最小值 的延伸题目。
- 允许重复会影响算法的时间复杂度吗？会如何影响，为什么？  

**解法一**

相比上一题有了重复的元素，在跳转的时候需要分清楚情况，在mid和中点相等的时候只排除右边界一个元素

```java
public int findMin(int[] nums) {
    int left=0,right=nums.length-1;
    while(left<right){
        int mid=left+(right-left)/2;
        if(nums[mid] > nums[right]){
            left=mid+1;
        }else if(nums[mid] < nums[right]){
            right=mid;
        }else{
            right--; //和右边界相等,无法判断,只缩减一步
        }
    }
    return nums[left];
}
```
## [33. 搜索旋转排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/)

假设按照升序排序的数组在预先未知的某个点上进行了旋转。

( 例如，数组 `[0,1,2,4,5,6,7]` 可能变为 `[4,5,6,7,0,1,2]` )。

搜索一个给定的目标值，如果数组中存在这个目标值，则返回它的索引，否则返回 `-1` 。

你可以假设数组中不存在重复的元素。

你的算法时间复杂度必须是 *O*(log *n*) 级别。

**示例 1:**

```java
输入: nums = [4,5,6,7,0,1,2], target = 0
输出: 4
```

**示例 2:**

```java
输入: nums = [4,5,6,7,0,1,2], target = 3
输出: -1
```

**解法一**

题目明确要求了时间复杂度O(logn)，所以肯定还是要二分，先上代码吧

```java
public static int search2(int[] nums, int target) {
        int len = nums.length;

        if ((nums == null) || (len <= 0)) {
            return -1;
        }

        int lo = 0;
        int hi = len - 1;

        while (lo <= hi) {
            int mid = lo + ((hi - lo) / 2);
            // 左, 右 指的是旋转点左右
            if (nums[mid] > target) { //首先是大于target的情况
                
                if (target < nums[lo]) {
                    //target在右边
                    //mid未知还需要判断下 画一个折线图就很清楚了
                    if (nums[mid] <= nums[hi]) { //mid也在右边
                        hi = mid - 1;
                    } else {
                        //mid在左边
                        lo = mid + 1;
                    }
                } else if (target > nums[lo]) {
                    //说明mid在左边, target也在左边
                    hi = mid - 1;
                } else {
                    return lo;
                }
            } else if (nums[mid] < target) { //小于target的情况

                if (target < nums[hi]) {
                    //mid在右边，target在右边
                    lo = mid + 1;
                } else if (target > nums[hi]) {
                    //target在左边
                    //mid未知还需要判断下
                    if (nums[mid] > nums[hi]) { //mid在左边
                        lo = mid + 1;
                    } else {
                        hi = mid - 1;
                    }
                } else {
                    return hi;
                }
            } else {
                return mid;
            }

            /*if(hi>=0&&lo<len&&nums[lo]<nums[hi]){
                   //切换成有序的二分
                   while(lo<=hi){
                         mid=lo+(hi-lo)/2;
                         if(nums[mid]>target){
                                    hi=mid-1;
                         }else if(nums[mid]<target){
                                    lo=mid+1;
                       }else return mid;
                   }
            }*/
        }
        return -1;
    }

```

1ms，99% 纯if判断**target**和**mid**的位置，然后选择移动**lo**还是**hi**，一开始我随便找了几组数然后就开始写，写到后面发现都是bug😂，这里画个图很方便

![mark](http://static.imlgw.top///20190507/vQgFb8yle0FH.png?imageslim)

在里面找点会很清晰

**解法二**

当然还有一种更加简单也不用这么复杂的方法

```java
public static int search(int[] nums, int target) {
    int lo=0,hi=nums.length-1;
    if(nums==null||nums.length<=0){
        return  -1;
    }
    int index=-1;
    while(lo<=hi){
        int mid=lo+(hi-lo)/2;
        if(nums[mid]>=nums[lo]){
            //左半部分有序
            index=binarySearch(nums,target,lo,mid);
            //对右半部分二分
            if(index==-1){
                lo=mid+1;
                //lo-->mid 没找到就对右半部分继续划分
            } else return index;
        } else if(nums[mid]<nums[lo]){
            //右半部分有序
            index=binarySearch(nums,target,mid,hi);
            if(index==-1){
                hi=mid-1;
            } else return index;
        }
    }
    return  index;
}

public static int binarySearch(int []nums,int target,int lo,int hi){
    while(lo<=hi){
        int mid=lo+(hi-lo)/2;
        if(nums[mid]>target){
            hi=mid-1;
        } else if(nums[mid]<target){
            lo=mid+1;
        } else return mid;
    }
    return -1;
}
```

这个应该比上一个慢一点，最好情况下是_**O(logN)**_直接将**target**划分到有序的那一边，如果没划分到有序的那一边就会花费时间去二分尝试切割数组，时间复杂度应该是`logN+log(N/2)+log(N/4)+...log(N/N)` 最后整体复杂度应该是`O(logN*logN)` ，虽然比 `logN` 好很多，但是并不是我们想要的算法

## [34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

给定一个按照升序排列的整数数组 `nums`，和一个目标值 `target`。找出给定目标值在数组中的开始位置和结束位置。

你的算法时间复杂度必须是 *O*(log *n*) 级别。

如果数组中不存在目标值，返回 `[-1, -1]`。

**示例 1:**

```java
输入: nums = [5,7,7,8,8,10], target = 8
输出: [3,4]
```

**示例 2:**

```java
输入: nums = [5,7,7,8,8,10], target = 6
输出: [-1,-1]
```

**解法一**

时间复杂度O(logN)，肯定还是要二分

```java
//两次二分
public int[] searchRange(int[] nums, int target) {
    if(nums.length<=0){
        return new int[]{-1,-1};
    }
    return new int[]{left(nums,target,0,nums.length-1),right(nums,target,0,nums.length-1)};
}

//5,7,7,8,8,8,8,10,10
public int left(int []nums,int target,int lo,int hi){
    while(lo<=hi){
        int mid=lo+(hi-lo)/2;
        //System.out.println("lo: "+nums[lo]+"mid: "+nums[mid] +"hi: "+nums[hi]);
        if(nums[mid]<target){
            lo=mid+1;
        }else if(nums[mid]>target){
            hi=mid-1;
        }else if(mid>0){ //nums[mid]=target
            if(nums[mid-1]!=target){
                return mid;
            }else{
                //控制向左找
                hi=mid-1;
            }
        }else{
            return mid; //0
        }
    }
    return -1;
}

public int right(int []nums,int target,int lo,int hi){
    while(lo<=hi){
        int mid=lo+(hi-lo)/2;
        if(nums[mid]<target){
            lo=mid+1;
        }else if(nums[mid]>target){
            hi=mid-1;
        }else if(mid<nums.length-1){
            if(nums[mid+1]!=target){
                return mid;
            }else{
                //控制向右找
                lo=mid+1;
            }
        }else{
            return mid; //nums.length
        }
    }
    return -1;
}
```

1ms ，99% 核心就是两次二分，分别向左和向后二分整个数组， 在相等的时候并不返回，多判断一下，左边的就控制hi向左边继续找，右边就控制lo向右边继续找，直到下一个不等于target就返回，和上面一题一样都是二分的变种

**解法二**

统一的解法，上面的做法虽然直白，但是没有通用性，这里借鉴评论区大佬 [liweiwei1419](https://www.liwei.party/)的[讲解](https://leetcode-cn.com/problems/search-insert-position/solution/te-bie-hao-yong-de-er-fen-cha-fa-fa-mo-ban-python-/)写一个通用的解法，之前写二分一直都是凭感觉，不注意细节，有错误就debug，东改一改，西改一改，然后就过了。。。毫无章法，以后要统一写法了

```java
//两次二分
public int[] searchRange(int[] nums, int target) {
    if(nums.length<=0){
        return new int[]{-1,-1};
    }
    return new int[]{left(nums,target,0,nums.length-1),right(nums,target,0,nums.length-1)};
}

//找大于等于target的第一个元素,小于肯定不符合
public int left(int []nums,int target,int lo,int hi){
    while(lo<hi){
        int mid=lo+(hi-lo)/2;
        if(nums[mid]<target){ //排除小于target的,剩下【lo,hi】都是大于等于的
            lo=mid+1;
        }else{
            hi=mid;
        }
    }
    return nums[hi]==target?hi:-1;
}

//找小于等于target的最后一个元素,大于肯定不符合
public int right(int []nums,int target,int lo,int hi){
    while(lo<hi){
        //选取右中值
        int mid=lo+(hi-lo)/2+1;
        if(nums[mid]>target){ //排除大于target,剩下[lo,hi]都是小于等于的
            hi=mid-1;
        }else{
            //根据这个判断需要选取右中值
            lo=mid;
        }
    }
    return nums[hi]==target?hi:-1;
}
```

## [287. 寻找重复数](https://leetcode-cn.com/problems/find-the-duplicate-number/)

给定一个包含 n + 1 个整数的数组 `nums`，其数字都在 1 到 n 之间（包括 1 和 n），可知至少存在一个重复的整数。假设只有一个重复的整数，找出这个重复的数。

**示例 1:**

```java
输入: [1,3,4,2,2]
输出: 2
```


**示例 2:**

```java
输入: [3,1,3,4,2]
输出: 3
```


**说明：**

- 不能更改原数组（假设数组是只读的）。
- 只能使用额外的 O(1) 的空间。
- 时间复杂度小于 O(n2) 。
- 数组中只有一个重复的数字，但它可能不止重复出现一次 

**解法一**

这题还是挺有意思的，题目要求了数组nums是只读的，且不能使用额外的空间，且时间复杂度还要小于O(N^2)，否则的话其实可以排序，或者使用Hash表来做，这里我们使用二分来做

```java
public int findDuplicate(int[] nums){
    int left=1,right=nums.length-1; //左右边界
    //这里实际上是对【1,2,3,4,...n-1】这个区间进行二分
    //在过程中对mid检测每个数在nums数组中出现的次数
    //1 3 4 2 2实际上是对【1,2,3,4】区间进行二分
    while(left<right){
        int mid=left+(right-left)/2;
        int temp=count(nums,mid);
        //排除中位数,小于mid的数<=mid,一定不是,说明重复元素一定在右边
        if(temp<=mid){ //1 2 3 4
            left=mid+1;
        }else{
            right=mid;
        }
    }
    return left;
}

//n-1个整数 , 1~n有n个数     
//1 2 2 3 4     1~4之间, 1 2 3 4
public int count(int[] nums,int n){
    int res=0;
    for (int i=0;i<nums.length;i++) {
        if (nums[i]<=n) {
            res++;          
        }
    }
    return res;
}
```
这样的解法还是很巧妙的，对nums数组的**取值范围**进行二分，二分的核心就是，nums数组中，小于取值范围中mid的元素应该小于等于mid

举个例子：`[1 3 4 2 2]` 取值范围是`[1 2 3 4]` ，取中点2，正常情况下nums中小于等于2的元素，应该最多有2个，也就是`[1 2]`2个，但是这里在nums中，有3个`[1 2 2]` 大于2了，这就说明一定有重复的元素，而且一定是小于中点2的，也就是在左半边，下一步就应该舍弃右半边，在`[1,2]`中继续查找 

这里按照我们之前的模板来写，先找排除mid的条件，**在nums中小于mid的元素的数量小于等于mid的时候，包括mid在内的右边界都会被排除，肯定都不是重复的元素** ，然后就按照模板写出二分就行了

**解法二**

快慢指针的做法，技巧性很强，一般人第一次做是很难想到这种做法的，其实和 [链表专题](http://imlgw.top/2019/02/27/leetcode-lian-biao-tag/#141-%E7%8E%AF%E5%BD%A2%E9%93%BE%E8%A1%A8) 中的环形链表是一样的做法，然后按照那个思路走就行了，不清楚原理可以看看上面环形链表的解法

```java
public int findDuplicate(int[] nums){
    int slow=0,fast=0;
    boolean isMeet=false;
    while(true){
        fast=isMeet?nums[fast]:nums[nums[fast]];
        slow=nums[slow];
        if (fast==slow) {
            if (isMeet) {
                return slow;
            }
            fast=0;
            isMeet=true;
        }
    }
}
```
这种解法的关键是将数组值看作索引然后再数组像链表一样移动，比如 `[1,2,3,4,5,6,7,8,9,5]`用值作为索引连接起来就是`1 2 3 4 [5 6 7 8 9] [5 6 7 8 9] ....` ，时间复杂度`O(N)` 技巧性比较强，如果面试管不追问的话其实答出上面的二分就ok了

## [852. 山脉数组的峰顶索引](https://leetcode-cn.com/problems/peak-index-in-a-mountain-array/)

我们把符合下列属性的数组 A 称作山脉：

- A.length >= 3
- 存在 0 < i < A.length - 1 使得A[0] < A[1] < ... A[i-1] < A[i] > A[i+1] > ... > A[A.length - 1]


给定一个确定为山脉的数组，返回任何满足 `A[0] < A[1] < ... A[i-1] < A[i] > A[i+1] > ... > A[A.length - 1]` 的 i 的值。

 **示例 1：**

```java
输入：[0,1,0]
输出：1
```


**示例 2：**

```java
输入：[0,2,1,0]
输出：1
```

**解法一**

其实还是上面的模板，只不过做了一点点改动而已，很傻逼的WA了一发，我也是服了自己了

```java
public static int peakIndexInMountainArray(int[] A) {
    int left=0,right=A.length-1;
    while(left<right){
        int mid=left+(right-left)/2;
        //System.out.println(mid);
        if (mid>0 && mid<A.length && A[mid] > A[mid-1] && A[mid]<A[mid+1]) {
            left=mid+1;
        }else if (mid>0 && mid<A.length && A[mid]< A[mid-1] && A[mid]>A[mid+1]){
            right=mid-1;
        }else{
            return mid;
        }
    }
    return left;
}
```

## [1283. 使结果不超过阈值的最小除数](https://leetcode-cn.com/problems/find-the-smallest-divisor-given-a-threshold/)

给你一个整数数组 `nums` 和一个正整数 `threshold`  ，你需要选择一个正整数作为除数，然后将数组里每个数都除以它，并对除法结果求和。

请你找出能够使上述结果小于等于阈值 `threshold` 的除数中 最小 的那个。

每个数除以除数后都向上取整，比方说 7/3 = 3 ， 10/2 = 5 。

题目保证一定有解。

**示例 1：**

```java
输入：nums = [1,2,5,9], threshold = 6
输出：5
解释：如果除数为 1 ，我们可以得到和为 17 （1+2+5+9）。
如果除数为 4 ，我们可以得到和为 7 (1+1+2+3) 。如果除数为 5 ，和为 5 (1+1+1+2)。
```

**示例 2：**

```java
输入：nums = [2,3,5,7,11], threshold = 11
输出：3
```

**示例 3：**

```java
输入：nums = [19], threshold = 5
输出：4
```

**提示：**

- `1 <= nums.length <= 5 * 10^4`
- `1 <= nums[i] <= 10^6`
- `nums.length <= threshold <= 10^6`

**解法一**

周赛的题，太蠢了，没做出来。。。。

```java
public int smallestDivisor(int[] nums, int threshold) {
    int left=1,right=1000000;
    while(left<right){
        int mid=left+(right-left)/2;
        int sum=0;
        for (int i=0;i<nums.length;i++) {
            sum+=(nums[i]+mid-1)/mid; //向上取整
        }
        if (sum>threshold) {
            left=mid+1;
        }else{
            right=mid;
        }
    }
    return left;
}
```
其实只要明确一点这题就很容易想到二分，解空间为：`[1，max(nums[i])]` 我们只需要在这个区间之内做二分搜索就ok了，再然后就是向上取整的一个小技巧

## [74. 搜索二维矩阵](https://leetcode-cn.com/problems/search-a-2d-matrix/)

编写一个高效的算法来判断 `m x n` 矩阵中，是否存在一个目标值。该矩阵具有如下特性：

每行中的整数从左到右按升序排列。
每行的第一个整数大于前一行的最后一个整数。
**示例 1:**

```java
输入:
matrix = [
  [1,   3,  5,  7],
  [10, 11, 16, 20],
  [23, 30, 34, 50]
]
target = 3
输出: true
```


**示例 2:**

```java
输入:
matrix = [
  [1,   3,  5,  7],
  [10, 11, 16, 20],
  [23, 30, 34, 50]
]
target = 13
输出: false
```

**解法一**

```java
public boolean searchMatrix(int[][] matrix, int target) {
    if(matrix==null || matrix.length<=0 || matrix[0].length<=0){
        return false;
    }
    int m=matrix.length;
    int n=matrix[0].length;
    int low=0,high=m-1;
    if (target>matrix[m-1][n-1] || target < matrix[0][0]) {
        return false;
    }
    while(low<=high){ //二分确定在哪一行
        int mid=low+(high-low)/2;
        if (target == matrix[mid][0]) {
            return true;
        }else if(matrix[mid][0]<target){
            low=mid+1;
        }else{
            high=mid-1;
        }
    }
    int column=low!=0?low-1:low;
    low=0;
    high=n-1;
    while(low<high){
        int mid=low+(high-low)/2;
        if (matrix[column][mid]==target) {
            return true;
        }else if(matrix[column][mid] < target){
            low=mid+1; 
        }else{
            high=mid-1;
        }
    }
    return target==matrix[column][low];
}
```

😔，这题wa了11次，是的，11次，可想而知我有多彩，最后写出来的解法还是如此的难看，主要就是在确定在哪一行的时候写出了好多问题，可以看到我上下的两种二分方法是不一样的，前期就揪着一种写，按照上面的板子写，结果写出了一堆bug... 以后写二分还是要注意啊，1s确定思路，代码写了3h。。。。

**解法二**

看了评论区写出来的，利用取模和除将二维数组拉成一维的，相当的优秀，也不用考虑那些边界，时间复杂度和上面一样`log(nm)` 

```java
public boolean searchMatrix(int[][] matrix, int target) {
    if(matrix==null || matrix.length<=0 || matrix[0].length<=0){
        return false;
    }
    int m=matrix.length;
    int n=matrix[0].length;
    int left=0,right=m*n-1;
    while(left<right){
        int mid=left+(right-left)/2;
        if(matrix[mid/n][mid%n]<target){
            left=mid+1;
        }else{
            right=mid;
        }
    }
    return matrix[left/n][left%n]==target;
}
```

## [4. 寻找两个有序数组的中位数](https://leetcode-cn.com/problems/median-of-two-sorted-arrays/)

给定两个大小为 m 和 n 的有序数组 `nums1` 和 `nums2`。

请你找出这两个有序数组的中位数，并且要求算法的时间复杂度为 `O(log(m + n))`。

你可以假设 `nums1` 和 `nums2` 不会同时为空。

**示例 1:**

```java
nums1 = [1, 3]
nums2 = [2]

则中位数是 2.0
```

**示例 2:**

```java
nums1 = [1, 2]
nums2 = [3, 4]

则中位数是 (2 + 3)/2 = 2.5
```

**解法一**

Hard题，首先想到的是归并，但是时间复杂度不符合要求，最低要求 `O(log(m+n))`，想了好一会儿实在是想不出来（菜）然后看了评论区的解法

```java
//find nums1+nums2 /2 大的数
public double findMedianSortedArrays(int[] nums1, int[] nums2) {
    int m=nums1.length;
    int n=nums2.length;
    int leftMid=(m+n+1)/2;
    int rightMid=(m+n+2)/2;
    return (findMedian(nums1,0,m-1,nums2,0,n-1,leftMid) + findMedian(nums1,0,m-1,nums2,0,n-1,rightMid)) * 0.5;
}
//    i
//1 2 3 5
//    j
//1 2 4 6 7 8 9     k=6 find k/2=3
//
//        i
//*1 2 3* 5
//  j  
//1 2 4 6 7 8 9     k=3 find k/2=1  res=4
public double findMedian(int[] nums1,int left1,int right1, int[] nums2,int left2,int right2,int k) {
    int len1=right1-left1+1;
    int len2=right2-left2+1;
    if (len1==0) {
        return nums2[left2+k-1];
    }
    if (len2==0) {
        return nums1[left1+k-1];
    }
    if (k==1) {
        return Math.min(nums1[left1],nums2[left2]);
    }
    int i=left1+Math.min(len1,k/2)-1;
    int j=left2+Math.min(len2,k/2)-1;
    if (nums1[i] < nums2[j]) {
        return findMedian(nums1,i+1,right1,nums2,left2,right2,k-(i-left1+1));
    }else{
        return findMedian(nums1,left1,right1,nums2,j+1,right2,k-(j-left2+1));
    }
}
```
这种解法还是挺妙的，求第k小的思路，两个数组都是有序的，我们要求第k小，我们可以将k一分为二，看看两个数组的 `k/2` 位置的元素哪个大哪个小，小的哪个数组前 `k/2` 个元素就可以直接排除掉，因为他们必不可能是第k小的元素，举个例子就很容易理解

```java
1 2 3 5
1 2 4 6 7 8 9  k=6
k/2=3,分别在两数组中找第三个元素，也即是3，4明显3比较小，所以我们可以直接排除第一个数组的1，2，3三个元素，他们必不可能是第k小的元素！
*1 2 3* 5
1 2 4 6 7 8 9  k=3
```

然后重复上面的过程，每次排除`k/2` 的元素，最后在`log(k)` 的时间复杂度下就能找到两个数组的mid，而这里`k=(m+n+1)/2` 所以是符合题目要求的，除此之外，我们还需要考虑奇数和偶数的情况，那我们就可以分别计算一下，我们求一下左中位数和右中位数，如果是奇数左中和右中就是同一个`(k)/2==(k+1)/2` ，偶数的话就是`(k)/2`和`(k+1)/2`分别就是左中和右中，然后我们直接/2就得到了解

