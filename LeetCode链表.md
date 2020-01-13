---
title: 
  LeetCode链表
tags:
  [链表,LeetCode]
categories:
    [算法]
date: 2019/2/27
cover: http://static.imlgw.top///20190227/gPprrxbrdwAx.jpg?imageslim
---

##  2. 两数相加

给出两个 **非空** 的链表用来表示两个非负的整数。其中，它们各自的位数是按照 **逆序** 的方式存储的，并且它们的每个节点只能存储 **一位** 数字。
 如果，我们将这两个数相加起来，则会返回一个新的链表来表示它们的和。
您可以假设除了数字 0 之外，这两个数都不会以 0 开头。
**示例：**
**输入：**(2 -> 4 -> 3) + (5 -> 6 -> 4)
**输出**：7 -> 0 -> 8
**原因**：342 + 465 = 807

**解法一**

```java
 public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
      // 先用0补齐短的哪一个
        ListNode n1 = l1;
        ListNode n2 = l2;
        while (n1 != null && n2 != null) {
            if (n1.next == null && n2.next != null) {
                n1.next = new ListNode(0);
            } else if (n1.next != null && n2.next == null) {
                n2.next = new ListNode(0);
            } else if (n1.next == null && n2.next == null) {
                break;
            }
            n1 = n1.next;
            n2 = n2.next;
        }
        // 从头开始加
        ListNode res = null;
        //实际操作的链表
        ListNode real=null;
        // 进位
        int n = 0;
        int m = 0;
        int i = 0;
        while (l1 != null) {
            // 当前位
            m = (l1.val + l2.val) % 10+n;
            // 判断进位后是否为10
            if(m==10) {
                m=0;
                n=1;
            }else {         
             // 进位 0 or 1
             n = (l1.val + l2.val) / 10;
            }
            // 添加到原链表的后面
            // 判断是不是第一次
            if (i == 0) {
                real=res = new ListNode(m);
            } else {
                ListNode temp = new ListNode(m);
                real.next = temp;
                real=temp;
            }
            //判断位数相同的情况下最后一位和大于10的情况
            if(n==1&&l1.next==null) {
                //在最后加个1
                real.next=new ListNode(1);
            }
            l2 = l2.next;
            l1 = l1.next;
            i++;
        }
        return res;
 }
```
很久之前写的代码了，代码很乱，用0补齐短的那个然后对应相加注意进位就行了。

## 445. 两数相加Ⅱ

给定两个**非空**链表来代表两个非负整数。数字最高位位于链表开始位置。它们的每个节点只存储单个数字。将这两数相加会返回一个新的链表。
你可以假设除了数字 0 之外，这两个数字都不会以零开头。
**进阶:**
如果输入链表不能修改该如何处理？换句话说，你不能对列表中的节点进行翻转。
**示例:**
**输入:** (7 -> 2 -> 4 -> 3) + (5 -> 6 -> 4)
**输出:** 7 -> 8 -> 0 -> 7

```java
public static ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    BigInteger b1 = new BigInteger(list2num(l1));
    BigInteger b2 = new BigInteger(list2num(l2));
    String resStr=b1.add(b2).toString();
    //再变成字符串存到连表里面
    ListNode res=new ListNode(1);
    ListNode real=res;
    for (int i=0;i<resStr.length();i++) {
        real.next=new ListNode(Integer.valueOf(resStr.charAt(i)-48));
        real=real.next;
    }
    return res.next;
}
public static String  list2num(ListNode l){
    String num="";
    while(l!=null){
        num=num+l.val;
        l=l.next;
    }
    return num;
}
```
这两题方法很多，下面那题实际上是上面那一题反过来的，但是题目要求不改变链表所以可以利用栈来反转，然后就跟上面的类似了，然后这里我偷了个懒用的`BigInteger`搞的速度也还行 77%beat。

---

## **876. 链表的中间节点**

```java
public static ListNode middleNode(ListNode head) {
        if(head==null||head.next==null)return head;
        ListNode fast=head;
        ListNode slow=head;
        // 1 2 3 4 5 6 7
        while(fast!=null&&fast.next!=null){
            fast=fast.next.next;
            slow=slow.next;
        }
        return slow;
}
```
> `快慢指针`，很常见很经典的做法后面很多题会用到这个。

---
## **206. 反转链表**
反转一个单链表。

**示例:**

**输入:** 1->2->3->4->5->NULL
**输出:** 5->4->3->2->1->NULL

**进阶:**
你可以迭代或递归地反转链表。你能否用两种方法解决这道题？

**解法一:**

```java
//递归
public static ListNode reverseList(ListNode head) {
    if (head == null || head.next == null) {
        return head;
    }
    ListNode newHead = reverseList(head.next);
    //从后往前
    head.next.next = head;
    head.next = null;
    return newHead;
}
```
**解法二:**

```java
//三指针迭代
public static ListNode reverseList2(ListNode head) {
    if(head==null)return head;
    ListNode pre=head;
    ListNode cur=head.next;
    ListNode temp;
    while(cur!=null){
        temp=cur.next;
        cur.next=pre;
        pre=cur;
        cur=temp;
    }
    head.next=null;
    return pre;
}
```

---
## **92. 反转链表Ⅱ**
反转从位置 _m_ 到 _n_ 的链表。请使用一趟扫描完成反转。

**说明:**
1 ≤ _m_ ≤ _n_ ≤ 链表长度。

**示例:**

**输入:** 1->2->3->4->5->NULL, _m_ = 2, _n_ = 4
**输出:** 1->4->3->2->5->NULL

```java
public static ListNode reverseBetween(ListNode head, int m, int n) {
    if(m==n) return head;
    //用来遍历
    ListNode pre=head;
    ListNode mid=head.next;
    ListNode rear=null;
    //在遍历的中间连接这个表 时间复杂厚度O(N)
    //所以需要先保存 m前的节点用于后面到n的时候连接n和前面的部分 preM
    //还要保存m节点，在后面遍历到n的时候将M节点和后面的部分连接
    //中间段的前后节点 
    ListNode preM =null;
    ListNode valM=head;
    //ListNode nNext=null;
    int count =1;
    while(count <=n-1){
        //count的位置实际上是指的pre的位置因为只有pre是从head开始走的
        //尾指针后移
        rear=mid.next;
        if(count==m-1){
            //保存M点前面的节点和M节点
            preM=pre;
            valM=mid;
            //System.out.println("preM :"+preM.val);
        }
        if(count==n-1){
            //连接n后面节点的值
            valM.next=rear;
            //在这里判断下m前有没有元素
            if(m==1){
                head=mid;
            } else{
                preM.next=mid;
            }
        }
        if(count >= m && count <=n-1){
            //只有mid的位置大于m小于等于n才会将节点next域反转
            mid.next=pre;
        }
        //其他两个指针也向后移动
        pre=mid;
        mid=rear;
        count++;
    }
    return head;
}
```
代码写的比较烂但是思路还是比较清晰，只扫描了一遍链表 2ms beat 100%，但是创建的指针有点多，抠边界要细心。

---
## **[725. 分隔链表](https://leetcode-cn.com/problems/split-linked-list-in-parts/)**
Given a (singly) linked list with head node `root`, write a function to split the linked list into `k` consecutive linked list "parts".

The length of each part should be as equal as possible: no two parts should have a size differing by more than 1. This may lead to some parts being null.

The parts should be in order of occurrence in the input list, and parts occurring earlier should always have a size greater than or equal parts occurring later.

Return a List of ListNode's representing the linked list parts that are formed.

Examples 1->2->3->4, k = 5 // 5 equal parts [ [1], [2], [3], [4], null ]

**Example 1:**

```java
Input: 
root = [1, 2, 3], k = 5
Output: [[1],[2],[3],[],[]]
Explanation:
The input and each element of the output are ListNodes, not arrays.
For example, the input root has root.val = 1, root.next.val = 2, \root.next.next.val = 3, and root.next.next.next = null.
The first element output[0] has output[0].val = 1, output[0].next = null.
The last element output[4] is null, but it's string representation as a ListNode is [].
```

**Example 2:**

```java
Input: 
root = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10], k = 3
Output: [[1, 2, 3, 4], [5, 6, 7], [8, 9, 10]]
Explanation:
The input has been split into consecutive parts with size difference at most 1, and earlier parts are a larger size than the later parts.
```

**Note:**

The length of `root` will be in the range `[0, 1000]`.

Each value of a node in the input will be an integer in the range `[0, 999]`.

`k` will be an integer in the range `[1, 50]`.

```java
public static ListNode[] splitListToParts(ListNode root, int k) {
    //先要获取下链表的长度
    ListNode temp=root;
    ListNode next=root;
    ListNode [] result=new ListNode[k];
    int count=0;
    while(temp!=null){
        temp=temp.next;
        count++;
    }
    temp=root;
    //任意两部分差距不能大于1，大的在前，小的在后面
    //其实就是对count进行分配
    //注意: 有null的情况一定是 k>count 直接按 1 切分就完事了
    //k<count的情况只要在 count/k 的前几个元素上加上 count/k 的余数就行了
    int size=count/k;
    int num=count%k;
    result[0]=root;
    int index=1;
    if(k<=count){
        for (int i=1;temp!=null && index < k;i++){
            next=temp.next;
            if(i<=(size+1)*num && i%(size+1)==0){
                //前几个res的分割点
                result[index++]=next;
                //切断
                temp.next=null;
            } else if(i>(size+1)*num && (i-num)%size==0){
                result[index++]=next;
                temp.next=null;
            }
            temp=next;
        }
    } else{
        //剩下的情况就是后面要补null的情况
        // 这里两种情况应该是可以合并的，但是k>count num>0 懒得去抠边界
        for (int i=1;i<k;i++){
            if(temp==null){
                //这个if其实没必要
                result[i]=null;
            } else{
                next=temp.next;
                result[i]=next;
                temp.next=null;
                temp=next;
            }
        }
    }
    return result;
}
```
3ms beat 89% 这题也比较简单主要是边界要抠好

---
## **86. 分隔(割)链表**
给定一个链表和一个特定值_ x_，对链表进行分隔，使得所有小于 _x_ 的节点都在大于或等于 _x_ 的节点之前。

你应当保留两个分区中每个节点的初始相对位置。

**示例:**

**输入:** head = 1->4->3->2->5->2, _x_ = 3
**输出:** 1->2->2->4->3->5

```java
public ListNode partition(ListNode head, int x) {
    //先在头部加一个dummy节点统一操作
    ListNode dummyNode=new ListNode(-1);
    dummyNode.next=head;
    //分割点
    ListNode pre=cutNode=dummyNode;
    ListNode cur=head;
    int cut=0;
    while(cur!=null){
        if(cur.val>=x&&cut==0){
            //只会执行一次在找到第一个val>=x的节点的时候---保存分割点
            cutNode=pre;
            cut=1;
        } else if(cur.val<x && cut==1){
            //找到分割点后 遍历到val<x的节点的情况---将cur连接到cutNode的后面 处理好cur相邻的两个节点
            //先处理好cur相邻的节点
            pre.next=cur.next;
            //连接cutNode
            cur.next=cutNode.next;
            cutNode.next=cur;
            //cutNode后移
            cutNode=cur;
        }
        pre=cur;
        cur=cur.next;
    }
    return dummyNode.next
}
```
这题也挺简单和上面的那题名字一样是在整理这篇博客的时候现场做的(2019.2.27)前后大概半个小时orz。。。比较菜，但是这个我没有在本地跑直接在LeetCode上提交的然后就过了 1ms beat84% 感觉思路比较清晰就没有本地跑，提交记录上最快的居然是用了额外空间new了两个链表然后连起来的。。。醉了可能是测试用例太少了。

---
## **160. 相交链表**
编写一个程序，找到两个单链表相交的起始节点。
如下面的两个链表：
![mark](http://static.imlgw.top///20190303/NVEXndTcF1R6.png?imageslim)

在节点 c1 开始相交。

**示例 1：**

![mark](http://static.imlgw.top///20190303/ymln2djUVieT.png?imageslim)

**输入**: intersectVal = 8, listA = [4,1,8,4,5], listB = [5,0,1,8,4,5], skipA = 2, skipB = 3
**输出**: Reference of the node with value = 8
**输入解释**:相交节点的值为 8 （注意，如果两个列表相交则不能为 0）。从各自的表头开始算起，链表 A 为 [4,1,8,4,5]，链表 B 为 [5,0,1,8,4,5]。在 A 中，相交节点前有 2 个节点；在 B 中，相交节点前有 3 个节点。

**示例 2：**

![mark](http://static.imlgw.top///20190303/2F7qqhkUIWog.png?imageslim)
**输入**:intersectVal = 2, listA = [0,9,1,2,4], listB = [3,2,4], skipA = 3, skipB = 1
**输出**:Reference of the node with value = 2
**输入解释**:相交节点的值为 2 （注意，如果两个列表相交则不能为 0）。从各自的表头开始算起，链表 A 为 [0,9,1,2,4]，链表 B 为 [3,2,4]。在 A 中，相交节点前有 3 个节点；在 B 中，相交节点前有 1 个节点。

**示例 3：**

![mark](http://static.imlgw.top///20190303/hKSAelGSE1TY.png?imageslim)

**输入**:intersectVal = 0, listA = [2,6,4], listB = [1,5], skipA = 3, skipB = 2
**输出**:null
**输入解释**:从各自的表头开始算起，链表 A 为 [2,6,4]，链表 B 为 [1,5]。由于这两个链表不相交，所以 intersectVal 必须为 0，而 skipA 和 skipB 可以是任意值。
**解释**:这两个链表不相交，因此返回 null。

**注意：**

*   如果两个链表没有交点，返回 `null`.
*   在返回结果后，两个链表仍须保持原有的结构。
*   可假定整个链表结构中没有循环。
*   程序尽量满足 O(_n_) 时间复杂度，且仅用 O(_1_) 内存。

**解法一：**

```java
public static ListNode getIntersectionNode(ListNode headA, ListNode headB) {
    ListNode pA=headA;
    ListNode pB=headB;
    //计算两个链表长度然后计算差距然后向后对齐
    int lenA=0;
    int lenB=0;
    while(pA!=null){
        pA=pA.next;
        lenA++;
    }
    while(pB!=null){
        pB=pB.next;
        lenB++;
    }
    int dis=lenB>lenA?lenB-lenA:lenA-lenB;
    if(lenB>lenA){
        while(dis-->0){
            headB=headB.next;
        }
    } else{
        while(dis-->0){
            headA=headA.next;
        }
    }
    //不相等就一直向后移
    while(headA!=headB){
        //如果有一条为空说明没有交点
        if(headA.next==null){
            return null;
        }
        headA=headA.next;
        headB=headB.next;
    }
    return headA;
}
```
这种方法比较直接直接计算两个链表的长度然后计算差值然后将长的那个移动到对应的位置让`两条链表尾对齐`然后一起向后移动
**解法二：**

```java
//方法二 
public static ListNode getIntersectionNode2(ListNode headA, ListNode headB) {
    //当一个指针到结尾时转到另一个链表头再向后移动 ，这样做的目的和就是可以直接消除链表之间的长度差，向后对齐，方法还是很巧妙的。
    ListNode pA=headA;
    ListNode pB=headB;
    if(headB==null || headA==null)
                 return null;
    //while(pA!=null && pB!=null ){
    while(pA!=pB){
        //要保证两个==null的时候都只能执行一次不然如果没有交点就会死循环
        //改变while的条件
        //改变pA，pB跳转的条件
        //这样就可以保证最后没交点的时候 第二遍循环pA和pB最后会同时等于null会有出口不会死循环
        pA=pA==null?headB:pA.next;
        pB=pB==null?headA:pB.next;
    }
    return pA;
}
```
>  同时遍历两个链表，当一个指针到结尾时转到另一个链表头再向后移动 ，这样做的目的和就是可以直接消除链表之间的长度差，使之尾对齐，方法还是很巧妙的，代码也比较简洁。

---
## **234. 回文链表**
请判断一个链表是否为回文链表。

**示例 1:**

**输入:** 1->2
**输出:** false

**示例 2:**

**输入:** 1->2->2->1
**输出:** true

**进阶：**
你能否用 O(n) 时间复杂度和 O(1) 空间复杂度解决此题？

```java
public boolean isPalindrome(ListNode head) {
    if(head==null || head.next==null){
        return true;
    }
    // 利用快慢指针找到中点
    ListNode fast = head;
    ListNode slow = head;
    while (fast.next != null) {
        fast = fast.next.next == null ? fast.next : fast.next.next;
        slow = slow.next;
    }
    // 如果是偶数节点，slow就是偏右的那个 奇数就是正中间的 奇数在正中间不用管
    // fast是尾节点 1 1 1 1 1 1
    resverList(slow);
    //slow.next==null
    // check
    while (fast != null && head != null) {
        if (fast.val != head.val) {
            return false;
        }
        fast = fast.next;
        head = head.next;
    }
    return true;

}

public static void resverList(ListNode node) {
    ListNode cur = node;
    ListNode pre = null;
    ListNode next = node;
    while (cur != null) {
        next = next.next;
        cur.next = pre;
        pre = cur;
        cur = next;
    }
}
```
这题就用到了上面的翻转链表的方法，不过这里只翻转了一半，翻转了后半段然后从两边到中间逐个节点对比

---
## **237. 删除链表中的节点**
请编写一个函数，使其可以删除某个链表中给定的（非末尾）节点，你将只被给定要求被删除的节点。

现有一个链表 -- head = [4,5,1,9]，它可以表示为:

![mark](http://static.imlgw.top///20190303/9MUvMzlcAN0G.png?imageslim)

**示例 1:**

**输入:** head = [4,5,1,9], node = 5
**输出:** [4,1,9]
**解释:** 给定你链表中值为 5 的第二个节点，那么在调用了你的函数之后，该链表应变为 4 -> 1 -> 9.

**示例 2:**

**输入:** head = [4,5,1,9], node = 1
**输出:** [4,5,9]
**解释:** 给定你链表中值为 1 的第三个节点，那么在调用了你的函数之后，该链表应变为 4 -> 5 -> 9.

**说明:**

*   链表至少包含两个节点。
*   链表中所有节点的值都是唯一的。
*   给定的节点为非末尾节点并且一定是链表中的一个有效节点。
*   不要从你的函数中返回任何结果。

**二货做法**

```java
public  static void deleteNodelow(ListNode node) {
    //思路就是和node后面的元素一直交换，就像冒泡排序一样
    ListNode next;
    ListNode temp=new ListNode(0);
    ListNode pre=temp;
    while(node.next!=null){
        next=node.next;
        if(node.next.next==null){
            pre=node;
        }
        //先保存最后一个节点前的节点
        //交换当前节点和后一个节点
        temp.val=node.val;
        node.val=next.val;
        next.val=temp.val;
        node=next;
    }
    pre.next=null;
}
```
首先想到的愚蠢的做法,怎么这么蠢？？？？

**正确做法**

```java
  public  static void deleteNode(ListNode node) {
        node.val=node.next.val;
        node.next=node.next.next;
  }
```

---
## **203. 移除链表元素**
删除链表中等于给定值 **_val _**的所有节点。

**示例:**

**输入:** 1->2->6->3->4->5->6, _**val**_ = 6
**输出:** 1->2->3->4->5

**解法一：**
双指针 + 虚拟头节点

```java
 public static ListNode removeElements(ListNode head, int val) {
        if (head == null) return null;
        ListNode dummyHead = new ListNode(0);
        //再头接待你之前加了新的节点
        dummyHead.next = head;
        ListNode pre = dummyHead, cur = head;
        while (cur != null) {
            if (cur.val == val) {
                pre.next = cur.next;
            } else {
                //不是相等的值就向后移动
                pre = cur;
            }
            cur = cur.next;
        }
        return dummyHead.next;
    }
```
**解法二：**

递归方法比较简洁

```java
//递归
public static ListNode removeElements2(ListNode head, int val){
    if (head == null)
       return null;
    //将下一个元素放进递归如果是==val的就会把下一个的下一个元素返回连接到当前元素
    head.next = removeElements2(head.next, val);
    return head.val == val ? head.next : head;
}
```
---
## **19. 删除链表的倒数第N个节点**
给定一个链表，删除链表的倒数第 n 个节点，并且返回链表的头结点。
示例：
给定一个链表: 1->2->3->4->5, 和 n = 2.
当删除了倒数第二个节点后，链表变为 1->2->3->5.
说明：
给定的 n 保证是有效的。
进阶：
你能尝试使用一趟扫描实现吗？

```java
public ListNode removeNthFromEnd(ListNode head, int n) {
    if(head==null&&head.next==null) return null;
    //双指针 主要是头和尾的删除需要抠一下边界
    //  -1 | 1 2 3 4 5 6
    ListNode fast=head;
    ListNode dummyNode=new ListNode(-1);
    dummyNode.next=head;
    ListNode slow=dummyNode;
    //加了哑节点，直接先加1
    int count=1;
    while(fast!=null){
        fast=fast.next;
        slow=count<n?slow:slow.next;
        count++;
        if(fast.next==null){
            //slow到达需要删除的位置的前一个
            slow.next=slow.next.next;
            return dummyNode.next;
        }
    }
    return dummyNode.next;
}
```
看了评论才知道咋一遍循环，主要就是控制slow指针走`length-n`步，让快指针先走n步，然后快慢一起走，快指针到头时慢指针就到`length-n`的位置了，这题也可以用List保存每个节点让然后把待删除的节点的前一个拿出来操作，遍历两遍的方法比较简单就不写了，感觉这种方法比较好 , 这题的OJ case比较少所以没什么可比性 , 前几个都是跑了两遍的，我把最快的拷过来跑的比我还慢。。。。然后我又提交了一次 beat 90%.......



---
## **82. 删除链表中的重复元素Ⅱ**
给定一个排序链表，删除所有含有重复数字的节点，只保留原始链表中 _没有重复出现 _的数字。

**示例 1:**

**输入:** 1->2->3->3->4->4->5
**输出:** 1->2->5

**示例 2:**

**输入:** 1->1->1->2->3
**输出:** 2->3
>乍一看跟上面那一题一样？这题是排序链表上面那题是无序的，而且这题不给定元素

```java
public  static ListNode deleteDuplicates(ListNode head) {
    if(head==null)return null;
    //首先想到的思路是3指针，然后遍历的过程中后面的指针遇到==val的情况就让后面的指针一直后移走到！=val
    //先添加个哑节点
    ListNode dummyNode=new ListNode(-1);
    dummyNode.next=head;
    ListNode cur=head;
    ListNode next=head.next;
    ListNode pre=dummyNode;
    while(next!=null){
        while(next.val==cur.val){
            next=next.next;
            if(next==null){
                pre.next=null;
                return dummyNode.next;
            }
            if(next.val!=cur.val){
                pre.next=next;
                //cur跟上
                cur=next;
                break;
            }
        }
        //关键就是pre移动这里有坑 不能直接将pre移到cur,因为会有连续的连续存在
        pre=cur.next!=null&&cur.val==cur.next.val?pre:cur;
        cur=next;
        next=next.next;
    }
    return dummyNode.next;
}
```
整体来说还是挺简单的只跑了一趟 1ms beta98%，评论里面大都只用了两个指针我用了三个这样感觉比较清晰
怎么好理解怎么来。貌似最快的是一个递归的，递归写起来确实玄学还要多练练啊

```java
  public ListNode deleteDuplicates(ListNode head) {
        if (head == null) {
            return null;
        }
        if (head.next != null && head.val == head.next.val) {
            while (head.next != null && head.val == head.next.val) {
                head = head.next;
            }
            return deleteDuplicates(head.next);
        } else {
            head.next = deleteDuplicates(head.next);
        }
        return head;
    }
```
这题还有个简单版的那个删除后会留下一个，那样向我上面这样写pre的跳转就会更简单一点，这里就不写了

---
## **143. [重排链表](https://leetcode-cn.com/problems/reorder-list/)**
给定一个单链表 _L_：L0→L1→…→Ln-1→Ln 
将其重新排列后变为： L0→Ln→L1→Ln-1→L2→Ln-2→…

你不能只是单纯的改变节点内部的值，而是需要实际的进行节点交换。

**示例 1:**

给定链表 1->2->3->4, 重新排列为 1->4->2->3.

**示例 2:**

给定链表 1->2->3->4->5, 重新排列为 1->5->2->4->3.

> 这题和上面的回文链表有点类似，都是快慢指针不过这题稍微复杂点

**解法一**

```java
public static void reorderList(ListNode head) {
    if(head==null){
        return;
    }
    ListNode right = head;
    ListNode slow = head;
    // 1 1 1 1 1 1 1
    while (right.next != null) {
        right = right.next.next != null ? right.next.next : right.next;
        slow = slow.next;
    }
    // 从slow开始翻转
    res(slow);
    //左半部分
    ListNode left = head;
    //下一个节点
    ListNode rnext = right;
    ListNode lnext = left;
    // 1 2 3 4 5 6 7 8 
    // 1 8 2 7 3 6 4 5
    while (right != null && left != null) {
        // 要保存right的下一个节点 , left也需要,不然无法导航到下一个节点
        lnext = lnext.next;
        rnext = rnext.next;
        // 偶数个数节点,如果遍历到right链表的最后一个节点
        // 偶数的话right链表会短一点 最后连接的时候
        // left: 1->2->3->4->5 right: 8->7->6->5   
        // 像这样会将5加到left的4和5之间,但是明显只有一个5这样添加就是有问题的
        if(right.next==null){
            //所以这里吧lnext赋值为null,后面就不会重复连接5这个节点
            lnext=null;
        }
        //奇数个数的时候这样连接没问题
        // 1 2 3 4 5 
        // 9 8 7 6 5
        //5.next=5
        left.next = right;
        //如果奇数个数到最后这一步 right和left是同一个节点都为值是5的节点
        //所以这里下面的直接覆盖了上面的
        //5.next=null
        right.next = lnext;
        left = lnext;
        right = rnext;
    }
}

//反转
public static void res(ListNode node) {
    ListNode pre = null;
    ListNode cur = node;
    ListNode nex = node;
    while (cur != null) {
        nex = nex.next;
        cur.next = pre;
        pre = cur;
        cur = nex;
    }
}
```

也是之前的代码了，写的比较烂，但是思路还是比较清晰的，边界需要注意，速度还行 4ms  77% 。

---
## [21. 合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)

将两个有序链表合并为一个新的有序链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 

**示例：**

```java
输入： 1->2->4, 1->3->4
输出： 1->1->2->3->4->4
```

**解法一**

常规迭代的做法

```java
 public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        ListNode temp=new ListNode(0);
        ListNode res=temp;
        while(l1!=null&&l2!=null){
            if(l2.val<l1.val){
                temp.next=l2;
                l2=l2.next;
                temp=temp.next;
            }else{
                temp.next=l1;
                l1=l1.next;
                temp=temp.next;
            }
        }
        temp.next=l1==null?l2:l1;
        return res.next;
}
```
归并分治的思想，期末考试的一道题

**解法二**

递归的做法

```java
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    ListNode dummyNode=new ListNode(-1);
    mergeTwoLists(l1,l2,dummyNode);
    return dummyNode.next;
}

public void mergeTwoLists(ListNode l1, ListNode l2,ListNode res) {
    if(l1==null){
        res.next=l2;
        return;
    }

    if(l2==null){
        res.next=l1;
        return;
    }

    if(l1.val>l2.val){
        res.next=l2;
        mergeTwoLists(l1,l2.next,res.next);
    }else{
        res.next=l1;
        mergeTwoLists(l1.next,l2,res.next);
    }
}
```

上面是我一开始自己写的，一点也不_递归_ 

```java
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    if(l1==null) return l2;
    if(l2==null) return l1;
    ListNode dummyNode=new ListNode(-1);
    if(l1.val<l2.val){
        l1.next=mergeTwoLists(l1.next,l2);
        return l1;
    }else{
        l2.next=mergeTwoLists(l1,l2.next);
        return l2;
    }
}
```

这种看着就很简洁

---
## [23. 合并K个排序链表](https://leetcode-cn.com/problems/merge-k-sorted-lists/)

合并 k 个排序链表，返回合并后的排序链表。请分析和描述算法的复杂度。

**示例:**

```java
输入:
[
  1->4->5,
  1->3->4,
  2->6
]
输出: 1->1->2->3->4->4->5->6
```

**解法一：**

```java
public ListNode mergeKLists(ListNode[] lists) {
    if(lists.length==0)return null;
    ListNode temp=lists[0];
    for (int i=0;i<lists.length-1;i++){
        temp=merge2List(temp,lists[i+1]);
    }
    return temp;
}

public static ListNode merge2List(ListNode headA,ListNode headB){
    ListNode dummyNode=new ListNode(-1);
    ListNode res=dummyNode;
    ListNode tempA=headA;
    ListNode tempB=headB;
    while(tempB!=null&&tempA!=null){
        if(tempB.val>tempA.val){
            res.next=tempA;
            tempA=tempA.next;
        } else{
            res.next=tempB;
            tempB=tempB.next;
        }
        res=res.next;
    }
    res.next=tempA==null?tempB:tempA;
    return dummyNode.next;
}
```
最先想到的方法，和前面的二路归并一样，把前两个归并的结果和后面的继续归并。速度太慢了200ms左右.....时间复杂度是`O(N^2)`.
**解法二：**

```java
public ListNode mergeKLists2(ListNode[] lists) {
    if(lists.length==0)return null;
    return divide(lists,0,lists.length-1);
}

public static ListNode divide(ListNode[] lists,int left,int right){
    if(left>=right)return lists[left];
    int mid=left+((right-left)>>1);
    ListNode l = divide(lists,left,mid);
    ListNode r = divide(lists,mid+1,right);
    return merge2List(l,r);
}

public static ListNode merge2List(ListNode headA,ListNode headB){
    if(headA==null)return headB;
    if(headB==null)return headA;
    ListNode dummyNode=new ListNode(-1);
    ListNode res=dummyNode;
    ListNode tempA=headA;
    ListNode tempB=headB;
    while(tempB!=null&&tempA!=null){
        if(tempB.val>tempA.val){
            res.next=tempA;
            tempA=tempA.next;
        } else{
            res.next=tempB;
            tempB=tempB.next;
        }
        res=res.next;
    }
    res.next=tempA==null?tempB:tempA;
    return dummyNode.next;
}
```
看起来很眼熟？没错就是归并排序的思路，利用分治的思想，先归并左边，再归并右边，然后merge左右的结果，时间复杂度为`O(NlogK)` (递归树深度为logK，归并每一层时间复杂度都是N)， 10ms左右，N是链表的元素个数，K是链表个数。而且因为是链表空间复杂度也不高。另外这题也可以改成非递归的方式如下:
```java
public ListNode mergeKLists3(ListNode [] lists){
    if (lists.length == 0) {
        return null;
    }
    int k = lists.length;
    while (k > 1) {
        for (int i = 0; i < k / 2; i++) {
            //两两合并将结果保存在前半部分的节点中然后缩小一半的范围
            lists[i] = merge2Lists(lists[i], lists[i + (k + 1) / 2]);
        }
        //缩小一半的范围
        k = (k + 1) / 2;
    }
    return lists[0];
}
```
**解法三：**

```java
//小根堆的方法
public ListNode mergeKLists4(ListNode[] lists) {
    //利用一个按节点值最小次序排列的优先队列, 每次取最小的节点加入返回链表中
    if(lists.length < 1) return null;
    Queue<ListNode> pq = new PriorityQueue<>((a, b) -> (a.val - b.val));
    ListNode head = new ListNode(0);
    ListNode cur = head;
    for (ListNode p : lists){
        if(p != null)
        pq.offer(p);
    }
    while(!pq.isEmpty()) {
        cur.next = pq.poll();
        cur = cur.next;
        if(cur.next != null)
        //讲当前节点后一个节点加入队列
        pq.offer(cur.next);
    }
    return head.next;
}
```

利用了小根堆，Java里面有小根堆可以直接用，思路就是每次把每条链表的头元素都放进小根堆里面然后找出最小的加到新链表中然后，最小的那个节点的链表向后移再加到小根堆里面，方法还是相当简洁的。但是用了90ms左右比较慢，我想了下感觉这个时间复杂度好像是NKlogK,但是评论说是NlogK。。。算法渣渣有点迷。。。

---

## 430. 扁平化多级双向链表

您将获得一个双向链表，除了下一个和前一个指针之外，它还有一个子指针，可能指向单独的双向链表。这些子列表可能有一个或多个自己的子项，依此类推，生成多级数据结构，如下面的示例所示。

扁平化列表，使所有结点出现在单级双链表中。您将获得列表第一级的头部。

**示例:**

**输入:**
 1---2---3---4---5---6--NULL
         |
         7---8---9---10--NULL
             |
             11--12--NULL

**输出:**
1-2-3-7-8-11-12-9-10-4-5-6-NULL
**以上示例的说明:**

给出以下多级双向链表:

![mark](http://static.imlgw.top///20190303/DqC2qKF5h63V.png?imageslim)

我们应该返回如下所示的扁平双向链表:

![mark](http://static.imlgw.top///20190303/qOSn0TLmMuCt.png?imageslim)

**解法一：**

```java
/*
        这种解法思路还是比较清晰的
        每次有子链的时候就直接把子链遍历到尾 然后添加到链表中形成新主链 把子链的入口节点child指定为null
        然后主链继续向后遍历所以整个遍历的次数就是整个链表元素的个数 O(M)
*/
public static Node flatten1(Node head) {
    if(head==null || head.next==null&&head.child==null){
        return head;
    }
    Node cur=head;
    Node nNext;
    Node child;
    while(cur!=null){
        if(cur.child!=null){
            //有子链表
            child=cur.child;
            //子链表的表头
            nNext=cur.next;
            //主链的下一节点
            //连接子链表
            cur.next=child;
            //主链的下一节点为子链表头
            child.prev=cur;
            //子链表的前驱节点
            //已经拼接到主链，孩子链置为空 （这步还很关键我开始一直忘设置为null）
            cur.child=null;
            if(nNext==null){
                //遍历到主链最后一个了
                //所以没有下一个节点，后面的步骤不用继续但是也不能Break 因为最后一个节点有可能还有子链表
                continue;
            }
            while(child.next!=null){
                //找到新主链的下一节点 (子链的最后一个)
                child=child.next;
            }
            //连接以前的主链
            child.next=nNext;
            nNext.prev=child;
        }
        //主链表向后移动
        cur=cur.next;
    }
    return head;
}
```
**解法二：**

```java
    //标准的DFS
    public static Node flatten2(Node node) {
       if(node==null){
            return node;
        }
        Node head = node;
        while (head!=null){
            //我感觉这样会快一些
             Node next = head.next;
            if(head.child!=null){
                next = head.next;
                //子链表扁平化 返回头节点
                Node nextLayer = flatten2(head.child);//子链表的头节点
                //连接子链表头和主链
                head.next = nextLayer;
                nextLayer.prev = head;
                //然后子链表置为null
                head.child = null;
                //遍历到子链表的结尾
                while (nextLayer.next!=null){
                    nextLayer = nextLayer.next;
                }
                //连接子链表的尾部
                nextLayer.next = next;
                if(next!=null){
                    next.prev = nextLayer;
                }
            }
            //这里就直接跳过子链表 之前的是head=head.next; 但是因为之前的子链表已经加到主链表中所以会浪费一些时间（子链表肯定是已经扁平化的肯定都没有子链表）
            head = next;
        }
        return node;
    }
```

两种方法都是看的评论里面的第二种我稍微改了下，直接跳过子链表效率会高很多，不过这题case也比较少看不出差异。

---
## **141. 环形链表**

给定一个链表，判断链表中是否有环。

为了表示给定链表中的环，我们使用整数 `pos` 来表示链表尾连接到链表中的位置（索引从 0 开始）。 如果 `pos` 是 `-1`，则在该链表中没有环。

**示例 1：**

**输入：**head = [3,2,0,-4], pos = 1
**输出：**true
**解释：**链表中有一个环，其尾部连接到第二个节点。

![mark](http://static.imlgw.top///20190303/H2wmyaG9uoiz.png?imageslim)

**示例 2：**
**输入：** head = [1,2], pos = 0
**输出：** true
**解释：** 链表中有一个环，其尾部连接到第一个节点。

![mark](http://static.imlgw.top///20190303/UqIc2XWwtxbo.png?imageslim)

**示例 3：**

**输入：** head = [1], pos = -1
**输出：** false
**解释：** 链表中没有环。

![mark](http://static.imlgw.top///20190303/dAg8QrxqJJga.png?imageslim)

**进阶：**

你能用 _O(1)_（即，常量）内存解决此问题吗？

**有一点需要注意的是只有一个节点的情况应该是不考虑的直接 false 。**

```java
public static Boolean hasCycle(ListNode head) {
    if (head == null || head.next == null)
                  return false;
    //快慢指针 相遇的时候快指针回到头部step改为1 再次相遇的时候就是环的pos
    //这题只是判断有没有环所以只要相遇就有环
    ListNode slow=head;
    ListNode fast=head.next;
    while(slow!=fast){
        //有环是不会走到尽头的
        if(fast.next==null || fast.next.next==null){
            return false;
        }
        fast=fast.next.next;
        slow=slow.next;
    }
    return true;
}
```
---

## 141. 环形链表Ⅱ

给定一个链表，返回链表开始入环的第一个节点。 如果链表无环，则返回 `null`。

为了表示给定链表中的环，我们使用整数 `pos` 来表示链表尾连接到链表中的位置（索引从 0 开始）。 如果 `pos` 是 `-1`，则在该链表中没有环。

**说明：** 不允许修改给定的链表。 ps:上题的基础上返回入环的第一个节点

```java
public static ListNode detectCycle(ListNode head) {
    if(head==null||head.next==null){
        return null;
    }
    ListNode fast=head;
    ListNode slow=head;
    Boolean isMeet=false;
    //快指针没有到尽头
    while(fast!=null&&fast.next!=null){
        fast=isMeet?fast.next:fast.next.next;
        slow=slow.next;
        if(fast==slow){
            if(!isMeet){
                //第一次相遇
                //我这种写法开始没考虑到这种情况,入环节点就是头节点就不能继续走了
                if(fast==head) return;
                fast=head;
                isMeet=true;
            } else{
                return fast;
            }
        }
    }
    return null;
}
```
> 这种解法还是挺有意思的`快慢指针`，快指针一次走两步慢指针一次走一步在环上相遇的时候快指针回到头节点步数调整为1，再>次相遇的>时候（这里有可能重合，当头节点就是入环节点的时候）就是入环节点。
> 原理 :
> A---->B---->C      分别为`头节点`，`入环节点`，`第一次相遇的节点`
> 分析第一次相遇时快慢指针走过的路径可得
> AB+BC+CB+BC=2(AB+BC)  快指针走过的路程肯定是慢指针的两倍
> 化简最后就得到AB=CB 所以他们`再次相遇`就是入环的节点

---
## **61. [旋转链表](https://leetcode-cn.com/problems/rotate-list/)**
给定一个链表，旋转链表，将链表每个节点向右移动  _k_个位置，其中 _k_是非负数。

**示例 1:**

```java
输入: 1->2->3->4->5->NULL, k = 2
输出: 4->5->1->2->3->NULL
解释:
向右旋转 1 步: 5->1->2->3->4->NULL
向右旋转 2 步: 4->5->1->2->3->NULL
```

**示例 2:**

```java
输入: 0->1->2->NULL, k = 4
输出: 2->0->1->NULL
解释:
向右旋转 1 步: 2->0->1->NULL
向右旋转 2 步: 1->2->0->NULL
向右旋转 3 步: 0->1->2->NULL
向右旋转 4 步: 2->0->1->NULL
```

**解法一**


```java
public static ListNode rotateRight(ListNode head, int k) {
    if(head==null||head.next==null){
        return head;
    }
    //先获取下链表的长度，顺便记录tail的值
    ListNode temp=head;
    ListNode tail=head;
    int length=0;
    while(temp!=null){
        length++;
        if(temp.next==null){
            tail=temp;
            break;
        }
        temp=temp.next;
    }
    //将K化简
    k=k%length;
    if(k==0) return head;
    temp=head;
    int count=0;
    //然后再遍历一遍链表在 length-k 的地方断开
    while(temp!=null){
        if(count==(length-k-1)){
            tail.next=head;
            head=temp.next;
            temp.next=null;
            return head;
        }
        count++;
        temp=temp.next;
    }
    return head;
}
```
虽然难度是mid，但是感觉这题还是比较简单，我看见有一种比较好点的方法是在第一遍循环完之后将链表转换为`双向链表`然后再移动还是比较有意思的

---
## **328. 奇偶链表**
给定一个单链表，把所有的奇数节点和偶数节点分别排在一起。请注意，这里的奇数节点和偶数节点指的是节点编号的奇偶性，而不是节点的值的奇偶性。

请尝试使用原地算法完成。你的算法的空间复杂度应为 O(1)，时间复杂度应为 O(nodes)，nodes 为节点总数。

**示例 1:**

**输入:** `1->2->3->4->5->NULL`
**输出:** `1->3->5->2->4->NULL`

**示例 2:**

**输入:** `2->1->3->5->6->4->7->NULL`
**输出:** `2->3->6->7->1->5->4->NULL`

**说明:**

*   应当保持奇数节点和偶数节点的相对顺序。
*   链表的第一个节点视为奇数节点，第二个节点视为偶数节点，以此类推。

```java
//奇偶链表
public static ListNode oddEvenList(ListNode head) {
    if(head==null||head.next==null||head.next.next==null)return head;
    // 1 2 3 4 5 6 7
    // 1 2 3 4 5 6
    ListNode pOdd=head;
    ListNode pEven=head.next;
    ListNode temp=pEven;
    while(pEven!=null&&pEven.next!=null){
        pOdd.next=pEven.next;
        //奇数先走
        pOdd=pOdd.next;
        pEven.next=pOdd.next;
        pEven=pEven.next;
    }
    pOdd.next=temp;
    return head;
}
```
> 很像踩石头过河的游戏，一道很简单的mid，不知道为啥一开始抠了半天的边界。。。果然
> 还是不熟悉啊。Add oil ! ! ! 👍👍👍👍👍

---
## **147. 对链表进行插入排序**

插入排序的动画演示如上篇文章。
每次迭代时，从输入数据中移除一个元素（用红色表示），并原地将其插入到已排好序的链表中。
插入排序算法：
插入排序是迭代的，每次只移动一个元素，直到所有元素可以形成一个有序的输出列表。
每次迭代中，插入排序只从输入数据中移除一个待排序的元素，找到它在序列中适当的位置，并将其插入。
重复直到所有输入数据插入完为止。
示例 1：
输入: 4->2->1->3
输出: 1->2->3->4
示例 2：
输入: -1->5->3->4->0
输出: -1->0->3->4->5

```java
//beat 50%
public static  ListNode insertionSortList(ListNode head) {
    if(head==null||head.next==null)return head;
    //哑节点
    ListNode dummyNode=new ListNode(Integer.MIN_VALUE);
    System.out.println(dummyNode.val);
    dummyNode.next=head;
    ListNode tempNode=head;
    //外循环内的指针
    ListNode loopVariable=head.next;
    ListNode loopPre=head;
    //内循环的指针
    ListNode tempPre=dummyNode;
    while(loopVariable!=null){
        //头插法
        for (tempNode=dummyNode;tempNode!=loopVariable;tempNode=tempNode.next){
            if(tempNode.val>loopVariable.val){
                //System.out.println(loopVariable.val);
                //printList(dummyNode.next);
                //先处理好loopVariable的前后节点
                loopPre.next=loopVariable.next;
                //再处理tempNode前后的节点
                loopVariable.next=tempNode;
                tempPre.next=loopVariable;
                //loopVariable 归位
                loopVariable=loopPre;
                break;
            }
            tempPre=tempNode;
        }
        loopPre=loopVariable;
        loopVariable=loopVariable.next;
    }
    return dummyNode.next;
}
```
上面的是我开始自己写的，但是提交后发现速度有点慢40ms 50%左右 然后我有点不信把比较靠前的拷了一个 10ms😂前几名10ms以内的都是用的方法不是插入....
在研究别人10ms的代码时突然意识到了问题所在 我在进行插入的时候没有判断就时没有关心是不是应该进行插入操作，对于数组的插入排序是不用关心这个问题的，因为是反向遍历的 而这里是链表只能正向的遍历如果不判断就会浪费很多时间

```java
// 16ms  beat  70% 开始少写了一个if判断
public static  ListNode insertionSortList(ListNode head) {
    if(head==null||head.next==null)return head;
    //哑节点
    ListNode dummyNode=new ListNode(Integer.MIN_VALUE);
    System.out.println(dummyNode.val);
    dummyNode.next=head;
    ListNode tempNode=head;
    //外循环内的指针
    ListNode loopVariable=head.next;
    ListNode loopPre=head;
    //内循环的指针
    ListNode tempPre=dummyNode;
    while(loopVariable!=null){
        //头插法
        if(loopVariable.val<loopPre.val){
            for (tempNode=dummyNode;tempNode!=loopVariable;tempNode=tempNode.next){
                if(tempNode.val>loopVariable.val){
                    //System.out.println(loopVariable.val);
                    //printList(dummyNode.next);
                    //先处理好loopVariable的前后节点
                    loopPre.next=loopVariable.next;
                    //再处理tempNode前后的节点
                    loopVariable.next=tempNode;
                    tempPre.next=loopVariable;
                    //loopVariable 归位
                    loopVariable=loopPre;
                    break;
                }
                tempPre=tempNode;
            }
        }
        loopPre=loopVariable;
        loopVariable=loopVariable.next;
    }
    return dummyNode.next;
}
```
这题整体思路就是按照插入排序的思路来的，值得注意的就是链表只能正向遍历，而且需要考虑保存的节点有两个，插入位置的前一个，以及待插入的前一个(头插法)。

---
## <font color=red size=5 face="黑体">148. 排序链表</font>

在 O(n log n) 时间复杂度和常数级空间复杂度下，对链表进行排序。

示例 1:

输入: 4->2->1->3
输出: 1->2->3->4
示例 2:

输入: -1->5->3->4->0
输出: -1->0->3->4->5

>上面那题时间复杂度明显是O(n2)最坏，这题要求是O(nlogn)和常数空间上面的插入肯定不适合了

```java
//归并排法
public static  ListNode sortList(ListNode head) {
    if(head==null||head.next==null)return null;
    return mergeSort(head);
}

public static ListNode mergeSort(ListNode head){
    if(head.next==null){
        return head;
    }
    ListNode fast=head;
    ListNode slow=head;
    ListNode pre=head;
    while(fast!=null&&fast.next!=null){
        pre=slow;
        fast=fast.next.next;
        slow=slow.next;
    }
    pre.next=null;
    //这里要注意断开两条链表不然后面不方便找中点
    ListNode left = mergeSort(head);
    //归并左边
    ListNode right = mergeSort(slow);
    //归并右边
    return merge2list(left,right);
    //返回 左右两条链表归并结果
}

public static ListNode merge2list(ListNode headA,ListNode headB){
    if(headA==null)return headB;
    if(headB==null)return headA;
    ListNode dummyNode=new ListNode(-1);
    dummyNode.next=headA;
    ListNode temp=dummyNode;
    while(headA!=null&&headB!=null){
        if(headA.val>headB.val){
            temp.next=headB;
            headB=headB.next;
        } else{
            temp.next=headA;
            headA=headA.next;
        }
        temp=temp.next;
    }
    temp.next=headA==null?headB:headA;
    return dummyNode.next;
}
```

​        4ms 85%  标准的`归并操作`*分治*的思想，但是我还是扣了好长时间，最后还是看了别人的代码才知道，其实一开始就想到了`快慢指针找中点`但是感觉时间复杂度可能会变得更高就没那样做。。。还是太菜了时间复杂度都不会分析。。。这里有一个小地方就是找到中点之后要记得断开中点和后面链表的连接，这样会方便后面归并，不然就需要传递一个边界的指针那样又会有很多问题（没错我开始就是这么做的😭）

   下面这种是后来又写的`经典快排`，400ms，12% 我都怀疑我到底写了个啥？后来把插入拿来试了下884+ms然后又看了一遍才相信我写的是快排。

```java
//我自己写的快排
public static ListNode sortList4(ListNode head){
    if(head==null||head.next==null)return head;
    sortList(head,null);
    return head;
}

//快排实现
public static void sortList(ListNode head,ListNode tail) {
    if(tail==head){
        return;
    }
    //确定枢纽元素
    ListNode base=partion(head,tail);
    sortList(head,base);
    sortList(base.next,tail);
}

//看了下别人的博客也学到了一种快排的新思路
//慢指针左边都是小于base枢纽元素的，快指针和慢指针中间都是大于等于base枢纽元素的
//慢指针后面的都是未知区域
public static ListNode partion(ListNode head,ListNode tail){
    ListNode base=head;
    ListNode fast=head.next;
    ListNode slow=head;
    // 3  1  3  2  5  -1 0
    //    s  f
    while(fast!=tail){
        if(fast.val<=base.val){
            //交换两个节点的值
            swap(fast,slow.next);
            slow=slow.next;
        }
        fast=fast.next;
    }
    //归位
    swap(head,slow);
    //应该可以试试返回区间
    return slow;
}

public static void swap(ListNode a,ListNode b){
    int temp=a.val;
    a.val=b.val;
    b.val=temp;
}
```

> 实际上快排确实不适合链表（下面光速打脸）因为毕竟不是数组可以从两边开始遍历，链表每次都需要遍历整个链表才能划分好基准位置。

看了下前几的代码发现了这个，`非标准的三向切分的快排`，为啥说是非标准呢？看下面代码就知道了，我给加了注释

```java
public static ListNode sortList3(ListNode head) {
    ListNode node = new ListNode(0);
    node.next = head;
    sort(node, null);
    return node.next;
}

private  static void sort(ListNode from, ListNode to) {
    if (from == null || from == to || from.next == to || from.next.next == to)return;
    int v = from.next.val;
    //基准元素
    ListNode mid = from;
    //切分点指针
    ListNode equal = from.next;
    //等于区域右边界
    ListNode node = from.next;
    //遍历用的指针
    while (node.next != to) {
        //node不到头  左开右开区间（from,to）
        if (node.next.val < v) {
            //小于基准位置元素
            //保存当前节点的下一个元素，用于插入节点
            //小于基准元素的节点
            ListNode currentNext = node.next.next;
            //保存切分点的下一个元素，作用同上
            ListNode midNext = mid.next;
            //交换node.next和mid
            //纸上画一下就了解了
            mid.next = node.next;
            node.next.next = midNext;
            node.next = currentNext;
            //切分点后移
            mid = mid.next;
        } else if (node.next.val == v) {
            //node的下一个等于基准元素
            //3 1 2 3 4 5 6
            if (equal == node) {
                //等于区域和node.next==val相邻了，直接跳过
                equal = node.next;
                node = node.next;
            } else {
                //将node.next插入equal后面
                //然后equal向后移动
                //和上面的类似
                ListNode nodeNext = node.next.next;
                ListNode equalNext = equal.next;
                equal.next = node.next;
                node.next.next = equalNext;
                node.next = nodeNext;
                equal = equal.next;
            }
        } else {
            //大于直接跳过
            node = node.next;
        }
    }
    // [mid.next---equal] 为等于val的节点
    sort(from, mid.next);
    sort(equal, to);
}
```

整体思路就是一共有三个指针，`mid`(切分点)  `equal`(等于区) `node`(遍历指针) node从from遍历到to，注意这里是`左开右开区间` 就是说头from和尾to都取不到，然后将小与base的节点插入到mid的后面，然后mid后移，等于区插入到equal的后面，最后形成的就是`[mid.next---equal]` 为等于val的节点，然后对子区域递归就ok了，这个用时 `4ms` 。。。。。。还要继续加油啊！！！

---
## <font color=red size=5 face="黑体">138.复制带随机指针的链表</font>

给定一个链表，每个节点包含一个额外增加的随机指针，该指针可以指向链表中的任何节点或空节点。

要求返回这个链表的深拷贝。 
示例：
![mark](http://static.imlgw.top///20190308/tR8e3eu2yqaq.png?imageslim)
输入：
`{"$id":"1","next":{"$id":"2","next":null,"random":{"$ref":"2"},"val":2},"random":{"$ref":"2"},"val":1}`
解释：
节点 1 的值是 1，它的下一个指针和随机指针都指向节点 2 。
节点 2 的值是 2，它的下一个指针指向 null，随机指针指向它自己。

提示：
你必须返回给定头的拷贝作为对克隆列表的引用。

**解法一：** 利用Map

```java
public static Node copyRandomList(Node head) {
    Map<Node,Node> copNode =new HashMap<>();
    Node temp=head;
    while(temp!=null){
        //建立对应关系
        copNode.put(temp,new Node(temp.val,null,null));
        temp=temp.next;
    }
    //再循环一次复制next和Radom节点
    temp=head;
    while(temp!=null){
        copNode.get(temp).next=copNode.get(temp.next);
        copNode.get(temp).random=copNode.get(temp.random);
        temp=temp.next;
    }
    return copNode.get(head);
}
```
第一个循环利用Map将原链表和拷贝链表形成对应关系，第二个循环就是直接给拷贝链表的next域和random域赋值。

**解法二：**`奥义 影分身`

```java
public static Node copyRandomList2(Node head) {
    if(head==null)return head;
    Node temp=head;
    //链表  奥义 - 影分身
    while(temp!=null){
        //这里直接将next域传入构造器完成和后面元素的连接
        temp.next=new Node(temp.val,temp.next,null);
        temp=temp.next.next;
    }
    temp=head;
    //连接random域
    while(temp!=null){
        if(temp.random!=null){
            temp.next.random=temp.random.next;
        }
        temp=temp.next.next;
    }
    temp=head;
    // 分离
    Node newHead=head.next;
    Node next=null;
    while(temp!=null){
        next=temp.next;
        if(next!=null){
            temp.next=next.next;
        }
        temp=next;
    }
    return newHead;
}
```
为啥要叫影分身？因为帅...这种方法比上面的要快一点可能是创建hashMap比较耗时间，其实分析这两种方法其实都是先把链表拷贝了一份，然后通过对应关系来连接拷贝链表的next和random域，map是通过键值对的方式对应拷贝链表，这样可以方便的通过原链表找到拷贝链表的random. 然后上面这种方法也是一样，在原链表每个节点后面copy一个节点，然后根据前一个节点的random来找拷贝节点的random(前一个节点的random的next) 主要就是找到一个对应关系.
> tips: 这题OJ上的0ms是有问题的，这题本意肯定也不是这个
>
> 
>
> ![mark](http://static.imlgw.top///20190308/p1GPgJVYaURp.png?imageslim)

最开始能通过主要是OJ后台只判断了val的值，可以看出现在题目已经改了。现在肯定是跑不过的，可能是判断了random是不是new出来的(我试了下看了下返回这个)
输入
`{"$id":"1","next":{"$id":"2","next":null,"random":{"$ref":"2"},"val":2},"random":{"$ref":"2"},"val":1}`
输出
`{"$id":"1","next":{"$id":"2","next":null,"random":`

`{"$id":"3","next":null,"random":null,"val":2},"val":2},`

`"random":{"$id":"4","next":null,"random":null,"val":2},"val":1}`
预期结果
`{"$id":"1","next":{"$id":"2","next":null,"random":{"$ref":"2"},"val":2},"random":{"$ref":"2"},"val":1}`

---
## **24. [两两交换链表中的节点](https://leetcode-cn.com/problems/swap-nodes-in-pairs/)**
给定一个链表，两两交换其中相邻的节点，并返回交换后的链表。

你不能只是单纯的改变节点内部的值，而是需要实际的进行节点交换。

**示例:**

给定 1->2->3->4, 你应该返回 2->1->4->3.

**解法一**

```java
public ListNode swapPairs(ListNode head) {
    if(head==null||head.next==null)return head;
    ListNode dummyNode=new ListNode(-1);
    dummyNode.next=head;
    ListNode nex=head.next;
    ListNode cur=head;
    ListNode pre=dummyNode;
    // -1|1 2 3 4
    while(nex!=null){
        pre.next=nex;
        cur.next=nex.next;
        nex.next=cur;
        pre=cur;
        if(cur.next==null){
            return dummyNode.next;
        }
        cur=cur.next;
        nex=cur.next;
    }
    return dummyNode.next;
}
```
其实跟反转链表是一样的，三个指针分别记录前 中 后三个节点然后逆序，只不过步长不一样，这里step为2，一次走两步， 我上面的代码可能写的有些乱，思路还是一样的

**解法二**

```java
//递归版本
public ListNode swapPairs(ListNode head) {
    if(head==null||head.next==null){
        return head;
    }
    ListNode next=head.next;
    head.next=swapPairs(next.next);
    next.next=head;
    return next;
}
```
递归是真的简洁，我最开始写反转链表的递归就是这么写的😂

---
## <font color=red size=5 face="黑体">25.K个一组翻转链表</font>

给出一个链表，每 k 个节点一组进行翻转，并返回翻转后的链表。

k 是一个正整数，它的值小于或等于链表的长度。如果节点总数不是 k 的整数倍，那么将最后剩余节点保持原有顺序。

**示例 :**

```java
给定这个链表：1->2->3->4->5
当 k = 2 时，应当返回: 2->1->4->3->5
当 k = 3 时，应当返回: 3->2->1->4->5
```

**说明 :**

- 你的算法只能使用常数的额外空间。
- 你不能只是单纯的改变节点内部的值，而是需要实际的进行节点交换。

**解法一**

```java
//非递归，理一下思路： 记录每次翻转前后的节点 然后翻转返回头 将每 K 个元素当成一个整体
public static ListNode reverseKGroup(ListNode head,int k) {
    if(head==null||head.next==null)return head;
    ListNode dummyNode=new ListNode(-1);
    dummyNode.next=head;
    ListNode pre=dummyNode;
    ListNode cur=head;
    ListNode next=head;
    ListNode temp;
    while(next!=null){
        //temp 保存 cur 方便后面连接  K+1位置的元素
        temp=cur;
        //k 个一组翻转
        int step=k;
        while(step>0 && next!=null){
            //next走到 K+1 位置节点
            next=next.next;
            step--;
            //小细节 k>链表长度时应该直接返回（我认为）等下提交了看看
            //所以直接应该直接返回 (掉了k的值判断 因为有可能刚好有k个元素)
            if(next==null&& step!=0){
                return dummyNode.next;
            }
        }
        //翻转 cur--next.prev 返回头节点
        //连接 反转后的头节点
        pre.next=reverse(cur,k);
        temp.next=next;
        //pre temp向后移动
        pre=temp;
        cur=next;
    }
    return dummyNode.next;
}

// -1| 1 2 3 | 4 5 6 | 7 8
//翻转链表并返回子链表
public static ListNode reverse(ListNode node,int k){
    ListNode pre=null;
    ListNode cur=node;
    ListNode next=node;
    while(k>0&&next!=null){
        next=next.next;
        cur.next=pre;
        pre=cur;
        cur=next;
        k--;
    }
    //返回反转后的头节点
    System.out.println("头"+pre.val);
    return pre;
}
```

> 7ms 74% 也是我第一道做出来的困难题，[上一道困难题超时了](http://imlgw.top/2018/10/31/%E4%B8%80%E9%81%93LeetCode%E5%BC%95%E5%8F%91%E7%9A%84%E6%83%A8%E6%A1%88/)
> 这题虽然说是困难题但是其实也不难，感觉就mid左右的水平，确实比上一题要复杂一点，但是只要思路理清楚其实也挺简单的，下面是我用OneNote画的一张图片。
> 4个指针分别对应 K 链表的 前一个(pre)  K链表的头节点(cur) 没有翻转前的K链表的头节点and`翻转后的尾节点(temp)`   K链表的后一个节点(next)。然后其实就简单了，写一个翻转链表的函数然后返回头节点(也可以多加一个指针指向`翻转前的头节点`)，然后就简单了，next指针一次走K步，走到 K+1 位置 同时也是`下一次K链表的头节点`，而temp则为下一次K链表的pre...然后循环这个过程就行了，其实写成递归会很简洁，但是我是真的不会写递归，太菜了Orz

![mark](http://static.imlgw.top///20190312/jl7moiy7PbjH.png?imageslim)

---
## **817. 链表组件**
给定一个链表（链表结点包含一个整型值）的头结点 head。
同时给定列表 G，该列表是上述链表中整型值的一个子集。
返回列表 G 中组件的个数，这里对组件的定义为：链表中一段最长连续结点的值（该值必须在列表 G 中）构成的集合。
示例 1：
输入: 
head: 0->1->2->3
G = [0, 1, 3]
输出: 2
解释: 
链表中,0 和 1 是相连接的，且 G 中不包含 2，所以 [0, 1] 是 G 的一个组件，同理 [3] 也是一个组件，故返回 2。
示例 2：
输入: 
head: 0->1->2->3->4
G = [0, 3, 1, 4]
输出: 2
解释: 
链表中，0 和 1 是相连接的，3 和 4 是相连接的，所以 [0, 1] 和 [3, 4] 是两个组件，故返回 2。
注意:
如果 N 是给定链表 head 的长度，1 <= N <= 10000。
链表中每个结点的值所在范围为 [0, N - 1]。
1 <= G.length <= 10000
G 是链表中所有结点的值的一个子集.

```java
public int numComponents(ListNode head, int[] G) {
    if(head==null||head.next==null) return head;
    ListNode temp=head;
    int res=0;
    while(temp!=null){
        if(isInG(temp.val,G)){
            while(temp!=null&&isInG(temp.val,G)){
                temp=temp.next;
            }
            res++;
            if(temp==null){
                return res;
            }
        }
        temp=temp.next;
    }
    return res;
}

public static Boolean isInG(int val,int[] G){
    for (int i=0;i<G.length;i++){
        if(G[i]==val){
            return true;
        }
    }
    return false;
}
```
首先想到的方法 91ms 19%..... 有点慢了，然后我稍微改了下。

```java
public int numComponents2(ListNode head, int[] G) {
    ListNode temp=head;
    int res=0;
    Boolean [] isInG=new Boolean[10000];
    int j=0;
    while(temp!=null){
        for (int i=0;i<G.length;i++){
            if(G[i]==temp.val){
                isInG[j]=true;
                break;
            }
        }
        j++;
        temp=temp.next;
    }
    temp=head;
    for (int i=0;temp!=null;temp=temp.next,i++){
        if(isInG[i]){
            while(temp!=null&&isInG[i]){
                temp=temp.next;
                i++;
            }
            res++;
            if(temp==null){
                return res;
            }
        }
    }
    return res;
}
```
   用了一个数组保存了每个位置的状态速度跟前面的差不多。。。主要问题就是那个数组的创建，这种创建方式用连续的下标来对应连续的链表的每个元素，每次都要遍历G才知道当前位置是不是在G中。

   其实可以直接把当前节点的val作为数组的下标这样既有了对应关系也不用遍历G.可以说是很优秀了，但是实际上这样做是有前提条件的那就是链表中的元素值应该`没有负数`，还是题做少了啊 Orz。

```java
public int numComponents3(ListNode head, int[] G) {
    ListNode temp=head;
    int res=0;
    Boolean [] isInG=new Boolean[10000];
    int j=0;
    //换一种方式 以node.val作为数组的下标
    for (int i:G){
        isInG[i]=true;
    }
    while(temp!=null){
        if(isInG[temp.val]){
            while(temp!=null&&isInG[temp.val]){
                temp=temp.next;
            }
            res++;
            if(temp==null){
                return res;
            }
        }
        temp = temp.next;
    }
    return res;
}
```

---

## **109. 有序链表转换二叉搜索树**

给定一个单链表，其中的元素按升序排序，将其转换为高度平衡的二叉搜索树。

本题中，一个高度平衡二叉树是指一个二叉树*每个节点* 的左右两个子树的高度差的绝对值不超过 1。

**示例:**

```sql
给定的有序链表： [-10, -3, 0, 5, 9],

一个可能的答案是：[0, -3, 9, -10, null, 5], 它可以表示下面这个高度平衡二叉搜索树：

      0
     / \
   -3   9
   /   /
 -10  5
```

BST不太熟看了下评论写出来的

```java
public TreeNode sortedListToBST(ListNode head) {
    return build(head,null);
}

public static TreeNode build(ListNode head,ListNode tail){
    if(head==tail){
        return null;
    }
    //快慢指针找中点
    ListNode fast=head,slow=head;
    while(fast!=tail&&fast.next!=tail){
        //左闭右开
        fast=fast.next.next;
        slow=slow.next;
    }
    //slow为中点或中点后一个
    //1 2 3 4
    TreeNode root=new TreeNode(slow.val);
    root.left=build(head,slow);
    root.right=build(slow.next,tail);
    return root;
}
```

---

## [1019. 链表中的下一个更大节点](https://leetcode-cn.com/problems/next-greater-node-in-linked-list/)

给出一个以头节点 `head` 作为第一个节点的链表。链表中的节点分别编号为：`node_1, node_2, node_3, ...` 。

每个节点都可能有下一个更大值（*next larger* **value**）：对于 `node_i`，如果其 `next_larger(node_i)` 是 `node_j.val`，那么就有 `j > i` 且  `node_j.val > node_i.val`，而 `j` 是可能的选项中最小的那个。如果不存在这样的 `j`，那么下一个更大值为 `0` 。

返回整数答案数组 `answer`，其中 `answer[i] = next_larger(node_{i+1})` 。

**注意：**在下面的示例中，诸如 `[2,1,5]` 这样的**输入**（不是输出）是链表的序列化表示，其头节点的值为 2，第二个节点值为 1，第三个节点值为 5 。

**示例 1：**

```java
输入：[2,1,5]
输出：[5,5,0]
```

**示例 2：**

```java
输入：[2,7,4,3,5]
输出：[7,0,5,5,0]
```

**示例 3：**

```java
输入：[1,7,5,1,9,2,5,1]
输出：[7,9,9,9,0,5,0,0]
```

 

**提示：**

1. 对于链表中的每个节点，`1 <= node.val <= 10^9`
2. 给定列表的长度在 `[0, 10000]` 范围内

```java
public static int[] nextLargerNodes(ListNode head) {
    //list里面存元素
    ArrayList<Integer> A = new ArrayList<>();
    for (ListNode node = head; node != null; node = node.next)
                A.add(node.val);
    int[] res = new int[A.size()];
    //栈里面存索引
    Stack<Integer> stack = new Stack<>();
    for (int i = 0; i < A.size(); ++i) {
        while (!stack.isEmpty() && A.get(stack.peek()) < A.get(i))
             res[stack.pop()] = A.get(i);
        stack.push(i);
    }
    return res;
}
```

新题，磨了好长时间，没做出来。。。真是菜啊 Orz，70ms，因为跑两遍。下面这个`14ms`可以说是相当快了，但，时间可能耗费在建立栈和list上了，看了下提交上的前几个都是用的数组，用数组模拟的栈。

```java
//和上面的方法差不多,但这个更快，上面那个跑了两遍
public static int[] nextLargerNodes3(ListNode head) {
    int[] stack = new int[10000];
    int[] res = new int[10000];
    //temp是链表的副本，相当于上面的list
    int[] temp = new int[10000];
    //top 栈顶
    int top = -1, i = 0;
    ListNode node = head;
    while (node != null) {
        while (top != -1 && temp[stack[top]] < node.val){
            //后一个大于当前节点, 栈不为空
            //pop出比它小的元素并赋值res，重新生成单调栈
            res[stack[top--]] = node.val;
        }
        stack[++top] = i;
        temp[i++] = node.val;
        node = node.next;
    }
    return Arrays.copyOf(res, i);
}
```

思路就是利用`单调栈`，栈里面存的索引对应的元素都是单调递减的，遇到不递减的就会一直pop()直到再次单调递减。这样很容易就找到了每个元素的下一个最大元素了。

19.7.21 重新做了一遍这道题，第一遍还是没想出来，还是看了之前的代码

```java
public static int[] nextLargerNodes5(ListNode head) {
        List<Integer> list=new ArrayList<>();
        ListNode temp=head;
        while(temp!=null){
            list.add(temp.val);
            temp=temp.next;
        }
        int [] stack=new int[10000];
        int stackIndex=-1;
        int [] res=new int[10000];
        for (int i=0;i<list.size();i++) {
            while(stackIndex!=-1 && list.get(i)>list.get(stack[stackIndex])){
                res[stack[stackIndex--]]=list.get(i);
            }
            //维护一个递减的栈
            stack[++stackIndex]=i;
        }
        return  Arrays.copyOf(res, list.size());
}
```

相比上面的方法一，采用了数组模拟队列(数据范围已经给定了)，30ms，80% 。仔细看看代码发现其实第一个循环完全没有必要，可以一边遍历一边存进去。

**一次遍历**

```java
    public static int[] nextLargerNodes6(ListNode head) {
        List<Integer> list=new ArrayList<>();
        ListNode temp=head;
        int [] stack=new int[10000];
        int stackIndex=-1;
        int [] res=new int[10000];

        for (int i=0;temp!=null;i++) {
            list.add(temp.val);
            while(stackIndex!=-1 && list.get(i)>list.get(stack[stackIndex])){
                res[stack[stackIndex--]]=list.get(i);
            }
            //维护一个递减的栈
            stack[++stackIndex]=i;
            temp=temp.next;
        }
        return  Arrays.copyOf(res, list.size());
    }
```

优化后发现比之前还慢了。。。

**数组模拟链表**

```java
 public static int[] nextLargerNodes7(ListNode head) {
        int [] list=new int[10000];
        ListNode temp=head;
        int [] stack=new int[10000];
        int stackIndex=-1;
        int [] res=new int[10000];

        int i;
        for ( i=0;temp!=null;i++) {
            list[i]=temp.val;
            while(stackIndex!=-1 && list[i]>list[stack[stackIndex]]){
                res[stack[stackIndex--]]=list[i];
            }
            //维护一个递减的栈
            stack[++stackIndex]=i;
            temp=temp.next;
        }
        return  Arrays.copyOf(res, i);
  }
```

这次提交了几次直接 8ms 100%了。。。

