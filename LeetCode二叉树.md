---
title: 
   LeetCode二叉树
tags: 
  [LeetCode,二叉树]
categories:
	[算法]
date: 2019/11/06
cover: http://static.imlgw.top/blog/20191106/pcmvnEK8Hwg8.png?imageslim
---

## [144. 二叉树的前序遍历](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)

Given a binary tree, return the *preorder* traversal of its nodes' values.

**Example:**

```java
Input: [1,null,2,3]
   1
    \
     2
    /
   3

Output: [1,2,3]
```

**Follow up:** Recursive solution is trivial, could you do it iteratively?

**解法一**

递归，没啥好说的

```java
private List<Integer> res=new ArrayList<>();

public List<Integer> preorderTraversal(TreeNode root) {
    if(root!=null){
        res.add(root.val);
        preorderTraversal(root.left);
        preorderTraversal(root.right);
    }
    return res;
}
```
**解法二**

教科书上的写法，经典的前序遍历非递归实现方式

```java
public List<Integer> preorderTraversal(TreeNode root) {
    List<Integer> res=new ArrayList<>();
    if (root==null) return res;
    Stack<TreeNode> stack=new Stack<>();
    stack.push(root);
    while(!stack.isEmpty()){
        TreeNode top=stack.pop();
        res.add(top.val);
        //注意顺序
        if (top.right!=null) {
            stack.push(top.right);
        }
        if (top.left!=null) {
            stack.push(top.left);
        }
    }
    return res;
}
```
**解法三**

非递归，模拟递归栈的方式，记录节点以及是否需要继续寻找子节点

```java
public List<Integer> preorderTraversal(TreeNode root) {
    List<Integer> res=new ArrayList<>();
    if (root==null) return res;
    Stack<Command> stack=new Stack<>();
    stack.push(new Command(true,root));
    while(!stack.isEmpty()){
        Command command=stack.pop();
        if (!command.isGo) {
            res.add(command.node.val);
        }else{
            TreeNode node=command.node;
            //逆序进栈
            if (node.right!=null) {
                stack.push(new Command(true,node.right));
            }
            if (node.left!=null) {
                stack.push(new Command(true,node.left));
            }    
            stack.push(new Command(false,node));
        }
    }
    return res;
}

static class Command{
    boolean  isGo; //是否继续寻找子节点
    TreeNode node; //当前节点
    public Command(boolean isGo,TreeNode node){
        this.isGo=isGo;
        this.node=node;
    }
}
```
bobo老师的一种思路，可以说是相当妙了👏，一下就解决了三种遍历的非递归实现，另外两种只需要调整一下进栈的顺序就可以了！

**解法四**

找到一个板子，可以很好的解决三种遍历

```java
//经典的非递归实现方式
public List<Integer> preorderTraversal4(TreeNode root) {
    List<Integer> res=new ArrayList<>();
    if (root==null) return res;
    Stack<TreeNode> stack=new Stack<>();
    TreeNode cur=root;
    while(cur!=null||!stack.isEmpty()){
        while (cur!=null) {
            res.add(cur.val);
            stack.push(cur);
            cur=cur.left;
        }
        //没有左子树了
        cur=stack.pop();
        //切换为右子树
        cur=cur.right;
        
    }
    return res;
}
```
> 关于 `while(cur!=null||!stack.isEmpty())`，其实栈中存的只是某一个根节点的所有左子树，并不是所有的节点，所以栈为空不代表已经遍历完所有节点了，只能代表当前节点的左子树都遍历完了，还有右子树还没遍历，只有当右子树也为空也就是`cur==null` 的时候才是遍历完了，具体看一下下面这颗树就明白了
>
> ```java
>       5
>      / \
>     4    6 
>   /      \
> 3       8
> ```
>
> 

## [589. N叉树的前序遍历](https://leetcode-cn.com/problems/n-ary-tree-preorder-traversal/)

给定一个 N 叉树，返回其节点值的*前序遍历*。

例如，给定一个 `3叉树` :

![MWwEt0.png](https://s2.ax1x.com/2019/11/20/MWwEt0.png)

返回其前序遍历: `[1,3,5,6,2,4]`。

**说明:** 递归法很简单，你可以使用迭代法完成此题吗?

**解法一**

递归没啥好说的

```java
//递归的方式
List<Integer> res=new LinkedList<>();

public List<Integer> preorder(Node root) {
    dfs(root);
    return res;
}

public void dfs(Node root) {
    if (root==null) {
        return;
    }
    List<Node> children=root.children;
    res.add(root.val);
    for (Node node:children) {
        preorder(node);
    }
}
```
**解法二**

迭代的方式

```java
public List<Integer> preorder(Node root) {
    List<Integer> res=new LinkedList<>();
    if (root==null) {
        return res;
    }
    Stack<Node> stack=new Stack<>();
    stack.add(root);
    while(!stack.isEmpty()){
        Node node=stack.pop();
        res.add(node.val);
        List<Node> children=node.children;
        //逆序添加
        for (int i=children.size()-1;i>=0;i--) {
            stack.add(children.get(i));
        }
    }
    return res;
}
```
到这里我是真的对遍历的那个板子无感了，这里我开始想用板子写，结果发现并不好写，无从下手（可能是我太菜），所以采用了经典的前序遍历方式，果然经典就是经典，通用性很强，而且相当好理解，所以以后遇到遍历的题目，尽量还是自己写，别套板子（对后序的板子也一直不是特别理解，所以也一直没记住，套板子还是要建立在理解的基础上啊，不然永远不会做！）

## [94. 二叉树的中序遍历](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)

Given a binary tree, return the *inorder* traversal of its nodes' values.

**Example:**

```java
Input: [1,null,2,3]
   1
    \
     2
    /
   3

Output: [1,3,2]
```

**Follow up:** Recursive solution is trivial, could you do it iteratively?

**解法一**

递归的方式和模拟栈的方式就不记录了，重点看一下这个板子

```java
//经典的非递归实现方式
public List<Integer> inorderTraversal3(TreeNode root) {
    List<Integer> res=new ArrayList<>();
    if (root==null) return res;
    Stack<TreeNode> stack=new Stack<>();
    TreeNode cur=root;
    while(cur!=null||!stack.isEmpty()){
        while (cur!=null) {
            stack.push(cur);
            cur=cur.left;
        }
        //没有左子树了
        cur=stack.pop();
        //将当前节点添加到res中
        res.add(cur.val);
        //切换为右子树
        cur=cur.right;
    }
    return res;
}
```
## [145. 二叉树的后序遍历](https://leetcode-cn.com/problems/binary-tree-postorder-traversal/)

Given a binary tree, return the *postorder* traversal of its nodes' values.

**Example:**

```java
Input: [1,null,2,3]
   1
    \
     2
    /
   3

Output: [3,2,1]
```

**Follow up:** Recursive solution is trivial, could you do it iteratively?

**解法一**

这题是个hard题，没那么容易（不过根据bobo老师的方式来做确实简单😂）

```java
public List<Integer> postorderTraversal3(TreeNode root) {
    List<Integer> res=new ArrayList<>();
    if (root==null) return res;
    Stack<TreeNode> stack=new Stack<>();
    TreeNode cur=root,lastNode=null; //lastNode为上一次访问的节点
    while(cur!=null||!stack.isEmpty()){
        while (cur!=null) {
            stack.push(cur);
            cur=cur.left;
        }
        //没有左子树了,把后一个左节点拿出来
        cur=stack.peek();
        //如果没有右节点,或者右节点访问过了
        if (cur.right==null||cur.right==lastNode) {
            //添加节点
            res.add(cur.val);
            //记录当前节点为lastNode
            lastNode=cur;
            //将他pop出去
            stack.pop();
            //节点已经弹出
            //指向null,不然就死循环了
            cur=null;
        }else{
            //右节点不为空,并且没访问过
            //切换为右子树,重复上面的步骤
            cur=cur.right;
        }
        
    }
    return res;
}
```
这种题一定要记住 "招式"，乱写只会越写越乱

**解法二**

这种解法似乎更加容易理解！！！

```java
public List<Integer> postorderTraversals(TreeNode root) {
    List<Integer> res=new ArrayList<>();
    if (root==null) return res;
    Stack<TreeNode> stack=new Stack<>();
    stack.push(root);
    TreeNode lastNode=null;
    while(!stack.isEmpty()){
        TreeNode cur=stack.peek();
        if ((cur.left==null && cur.right ==null) || (lastNode!=null &&( cur.left==lastNode || cur.right==lastNode))) {
            stack.pop();
            res.add(cur.val);
            lastNode=cur;
        }else{
            if (cur.right!=null) {
                stack.push(cur.right);
            }
            if (cur.left!=null) {
                stack.push(cur.left);
            }
        }
    }
    return res;
}
```
## [590. N叉树的后序遍历](https://leetcode-cn.com/problems/n-ary-tree-postorder-traversal/)

给定一个 N 叉树，返回其节点值的*后序遍历*。

例如，给定一个 `3叉树` :

![NTree](https://i.loli.net/2019/12/01/KAQP9UNfV5bau7J.png)

返回其后序遍历: `[5,6,3,2,4,1]`.

**说明:** 递归法很简单，你可以使用迭代法完成此题吗?

**解法一**

递归的解法，没啥好说的

```java
List<Integer> res=new LinkedList<>();

public List<Integer> postorder(Node root) {
    if (root==null) {
        return res;
    }
    dfs(root);
    return res;
}

public void dfs(Node root) {
    List<Node> children=root.children;
    for (Node node:children) {
        dfs(node);
    }
    res.add(root.val);
}
```

**解法二**

锁了！这才是树遍历的板子

```java
public List<Integer> postorder(Node root) {
    List<Integer> res=new LinkedList<>();
    if (root==null) {
        return res;
    }
    Stack<Node> stack=new Stack<>();
    stack.push(root);
    Node lastNode=null;
    while(!stack.isEmpty()){
        Node node=stack.peek();
        List<Node> children=node.children;
        if (children.isEmpty() || (lastNode!=null && lastNode == children.get(children.size()-1))) {
            res.add(node.val);
            stack.pop();
            lastNode=node;
        }else{
            for (int i=children.size()-1;i>=0;i--) {
                stack.push(children.get(i));
            }
        }
    }
    return res;
}
```

这题开始因为一个空的case把我搞晕了，搞了半天才发现

## [102. 二叉树的层次遍历](https://leetcode-cn.com/problems/binary-tree-level-order-traversal/)

Given a binary tree, return the *level order* traversal of its nodes' values. (ie, from left to right, level by level).

For example:
Given binary tree `[3,9,20,null,null,15,7]`,

```java
    3
   / \
  9  20
    /  \
   15   7
```

return its level order traversal as:

```java
[
  [3],
  [9,20],
  [15,7]
]
```

**解法一**

BFS，利用队列

```java
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> res=new ArrayList<>();
    if (root==null)return res;
    Queue<TreeNode> queue=new LinkedList<>();
    queue.add(root);
    while(!queue.isEmpty()){
        //count代表的其实就是每一层的节点个数
        int count=queue.size();
        List<Integer> list=new ArrayList<>();
        while(count>0){
            //取出当前节点,并将其左右子节点入队列
            TreeNode node=queue.poll();
            list.add(node.val);
            if (node.left!=null) {
                queue.add(node.left);
            }
            if (node.right!=null) {
                queue.add(node.right);
            }
            count--;
        }
        res.add(list);
    }
    return res;
}
```

**解法二**

递归DFS，这种其实还是挺有意思的，可以看下

```java
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> res = new ArrayList<>();
    helper(res, root, 0);
    return res;
}

private void helper(List<List<Integer>> res, TreeNode root, int depth) {
    if (root == null) return;
    //需要增加一层
    if (res.size() == depth) res.add(new LinkedList<>());
    res.get(depth).add(root.val);
    helper(res, root.left, depth + 1);
    helper(res, root.right, depth + 1);
}
```
## [429. N叉树的层序遍历](https://leetcode-cn.com/problems/n-ary-tree-level-order-traversal/)

给定一个 N 叉树，返回其节点值的*层序遍历*。 (即从左到右，逐层遍历)。

例如，给定一个 `3叉树` :

![NTee](https://i.loli.net/2019/12/01/He8KVlms1jynbvr.png)



返回其层序遍历:

```java
[
     [1],
     [3,2,4],
     [5,6]
]
```

**说明:**

1. 树的深度不会超过 `1000`。
2. 树的节点总数不会超过 `5000`。

**解法一**

```java
public List<List<Integer>> levelOrder(Node root) {
    List<List<Integer>> res=new LinkedList<>();
    if (root==null) {
        return res;
    }
    Queue<Node> queue=new LinkedList<>();
    queue.add(root);
    while(!queue.isEmpty()){
        int count=queue.size();
        List<Integer> temp=new LinkedList<>();
        while(count>0){
            Node node=queue.poll();
            temp.add(node.val);
            for (Node child:node.children) {
                queue.add(child);
            }
            count--;
        }
        if (!temp.isEmpty()) {
            res.add(temp);   
        }
    }
    return res;
}
```
## [107. 二叉树的层次遍历 II](https://leetcode-cn.com/problems/binary-tree-level-order-traversal-ii/)

Given a binary tree, return the *bottom-up level order* traversal of its nodes' values. (ie, from left to right, level by level from leaf to root).

For example:
Given binary tree `[3,9,20,null,null,15,7]`,

```java
    3
   / \
  9  20
    /  \
   15   7
```



return its bottom-up level order traversal as:

```java
[
  [15,7],
  [9,20],
  [3]
]
```

**解法一**

```java
public List<List<Integer>> levelOrderBottom(TreeNode root) {
    List<List<Integer>> res=new ArrayList<>();
    if(root==null)return res;
    Queue<TreeNode> queue=new LinkedList<>();
    queue.add(root);
    while(!queue.isEmpty()){
        List<Integer> list=new ArrayList<>();
        int count=queue.size();
        while(count>0){
            TreeNode top=queue.poll();
            if (top.left!=null) {
                queue.add(top.left);
            }
            if (top.right!=null) {
                queue.add(top.right);
            }
            list.add(top.val);
            count--;
        }
        //从头添加
        res.add(0,list);
    }
    return res;
}
```
主要是复习下层次遍历，相比上面就多了 `res.add(0,list)` 从头部添加

## [103. 二叉树的锯齿形层次遍历](https://leetcode-cn.com/problems/binary-tree-zigzag-level-order-traversal/)

Given a binary tree, return the *zigzag level order* traversal of its nodes' values. (ie, from left to right, then right to left for the next level and alternate between).

For example:
Given binary tree `[3,9,20,null,null,15,7]`,

```java
    3
   / \
  9  20
    /  \
   15   7
```

return its zigzag level order traversal as:

```java
[
  [3],
  [20,9],
  [15,7]
]
```

**解法一**

```java
public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
    List<List<Integer>> res=new ArrayList<>();
    if(root==null)return res;
    Queue<TreeNode> queue=new LinkedList<>();
    queue.add(root);
    boolean reverse=false;
    while(!queue.isEmpty()){
        LinkedList<Integer> list=new LinkedList<>();
        int count=queue.size();
        while(count>0){
            TreeNode node=queue.poll();
            if (reverse) {
                //从头添加，相当于逆序了
                list.addFirst(node.val);
            }else{
                list.add(node.val);
            }
            if (node.left!=null) {
                queue.add(node.left);
            }
            if (node.right!=null) {
                queue.add(node.right);
            }
            count--;
        }
        reverse=!reverse;
        res.add(list);
    }
    return res;
}
```
和上面一题一样，老想着怎么去按照题目的要求去遍历节点，哎，太蠢了，灵活一点啊

## [199. 二叉树的右视图](https://leetcode-cn.com/problems/binary-tree-right-side-view/)

给定一棵二叉树，想象自己站在它的右侧，按照从顶部到底部的顺序，返回从右侧所能看到的节点值。

**示例:**

```java
输入: [1,2,3,null,5,null,4]
输出: [1, 3, 4]
解释:

   1            <---
 /   \
2     3         <---
 \     \
  5     4       <---
```

**解法一**

还是和上面一样，一上午做了三道一样的题，这题吸取了上面的教训没有去想怎么遍历了

```java
public List<Integer> rightSideView(TreeNode root) {
    List<Integer> res=new ArrayList<>();
    if(root==null) return res;
    LinkedList<TreeNode> queue=new LinkedList<>();
    queue.add(root);
    while(!queue.isEmpty()){
        int count=queue.size();
        while(count>0){
            TreeNode node=queue.poll();
            if (node.left!=null) {
                queue.add(node.left);
            }
            if (node.right!=null) {
                queue.add(node.right);
            }
            //取每一层最后一个节点
            if (count==1) {
                res.add(node.val);
            }
            count--;
        }
    }
    return  res;
}
```
只记录每一层最后一个节点，最后得到的就是右视图



## [104. 二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)

给定一个二叉树，找出其最大深度。

二叉树的深度为根节点到最远叶子节点的最长路径上的节点数。

**说明:** 叶子节点是指没有子节点的节点。

**示例：**
给定二叉树 [3,9,20,null,null,15,7]

```java
    3
   / \
  9  20
    /  \
   15   7
```


返回它的最大深度 3 。

**解法一**

递归解法，很简洁

```java
//maxDepth(root)=1+max(maxDepth(root.left),maxDepth(root.right));
public int maxDepth(TreeNode root) {
    if(root==null){
        return 0;
    }
    int maxLeft=maxDepth(root.left);
    int maxRight=maxDepth(root.right);
    return (maxLeft>maxRight?maxLeft:maxRight)+1;
}
```
**解法二**

BFS，广度优先搜索

```java
public int maxDepth(TreeNode root) {
    if (root==null) return 0;
    Queue<TreeNode> queue=new LinkedList<>();
    queue.add(root);
    int max=0;//注意初始值
    while(!queue.isEmpty()){
        int count=queue.size();
        while(count>0){
            TreeNode node=queue.poll();
            if (node.left!=null) {
                queue.add(node.left);
            }
            if (node.right!=null) {
                queue.add(node.right);
            }
            count--;
        }
        max++;
    }
    return max;
}
```
## [559. N叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-n-ary-tree/)

给定一个 N 叉树，找到其最大深度。

最大深度是指从根节点到最远叶子节点的最长路径上的节点总数。

例如，给定一个 `3叉树` :

![3叉树](https://s2.ax1x.com/2019/11/20/MWwEt0.png)

我们应返回其最大深度，3。

**说明:**

1. 树的深度不会超过 `1000`。
2. 树的节点总不会超过 `5000`。

**解法一**

没啥好说的

```java
public int maxDepth(Node root) {
    if (root==null) {
        return 0;
    }
    int max=0;
    List<Node> children=root.children;
    for (Node node:children) {
        max=Math.max(max,maxDepth(node));
    }
    return max+1;
}
```
## [111. 二叉树的最小深度](https://leetcode-cn.com/problems/minimum-depth-of-binary-tree/)

给定一个二叉树，找出其最小深度。

最小深度是从根节点到最近叶子节点的最短路径上的节点数量。

**说明:** 叶子节点是指没有子节点的节点。

**示例:**

给定二叉树 `[3,9,20,null,null,15,7]`   

```java
    3
   / \
  9  20
    /  \
   15   7
```


返回它的最小深度  2.

**解法一**

最大都求了，最小也来一发，经典BFS做法，求最短路径

```java
public int minDepth(TreeNode root) {
    if (root==null) return 0;
    Queue<TreeNode> queue=new LinkedList<>();
    queue.add(root);
    int min=0;
    while(!queue.isEmpty()){
        int count=queue.size();
        min++;
        while(count>0){
            TreeNode node=queue.poll();
            if (node.left==null && node.right==null) {
                return min;
            }
            if (node.left!=null) {
                queue.add(node.left);
            }
            if (node.right!=null) {
                queue.add(node.right);
            }
            count--;
        }
    }
    return min;
}
```
**解法二**

递归

```java
public int minDepth(TreeNode root) {
    if (root==null) return 0;
    if (root.left==null) {
        return minDepth(root.right)+1;
    }
    if (root.right==null) {
        return minDepth(root.left)+1;
    }
    return Math.min(minDepth(root.left),minDepth(root.right))+1;
}
```
很上面最大的相反，但是有个细节需要注意，如果一个根节点左右子树，**有一颗为空**，如果不处理，按照之前的逻辑，这颗空子树下一次就会返回0，肯定会比另一颗小最后返回的就是到这颗子树的路径，但是仔细想想这样是正确的么？明显不是，最短路径的尽头一定是叶子节点也就是左右子树都为空的时候，所以这里需要特别注意


## [226. 翻转二叉树](https://leetcode-cn.com/problems/invert-binary-tree/)

翻转一棵二叉树。

**示例：**

输入：

```java
     4
   /   \
  2     7
 / \   / \
1   3 6   9
```


输出：

```java
     4
   /   \
  7     2
 / \   / \
9   6 3   1
```

**备注:**
这个问题是受到 Max Howell 的 原问题 启发的 ：

> 谷歌：我们90％的工程师使用您编写的软件(Homebrew)，但是您却无法在面试时在白板上写出翻转二叉树这道题，这太糟糕了。

**解法一**

```java
public TreeNode invertTree(TreeNode root) {
    if (root==null) {
        return null;
    }
    invertTree(root.left);
    invertTree(root.right);
    //交换左右节点
    TreeNode temp=root.left;
    root.left=root.right;
    root.right=temp;
    return root;
}
```
注意递归调用和交换节点的顺序，不能搞反了

```java
public TreeNode invertTree(TreeNode root) {
    if (root==null) {
        return null;
    }
    TreeNode right=root.right;
    root.right=invertTree(root.left);
    root.left=invertTree(right);
    return root;
}
```
比较简洁也比较符合递归的做法

## [100. 相同的树](https://leetcode-cn.com/problems/same-tree/)

给定两个二叉树，编写一个函数来检验它们是否相同。

如果两个树在结构上相同，并且节点具有相同的值，则认为它们是相同的。

**示例 1:**

```java
输入:       1         1
          / \       / \
         2   3     2   3

        [1,2,3],   [1,2,3]

输出: true
```

**示例 2:**

```java
输入:      1          1
          /           \
         2             2
    
        [1,2],     [1,null,2]

输出: false
```

**示例 3:**

```java
输入:       1         1
          / \       / \
         2   1     1   2
        
        [1,2,1],   [1,1,2]
输出: false
```

**解法一**

```java
public boolean isSameTree(TreeNode p, TreeNode q) {
    if (p==null && q==null) {
        return true;
    }
    if (p!=null && q!=null && p.val==q.val) {
        return isSameTree(p.right,q.right)&&isSameTree(p.left,q.left);
    }
    return false;
}
```

## [572. 另一个树的子树](https://leetcode-cn.com/problems/subtree-of-another-tree/)

给定两个非空二叉树 s 和 t，检验 s 中是否包含和 t 具有相同结构和节点值的子树。s 的一个子树包括 s 的一个节点和这个节点的所有子孙。s 也可以看做它自身的一棵子树。

**示例 1:**
给定的树 s:

```java
     3
    / \
   4   5
  / \
 1   2
```


给定的树 t：

```java
   4 
  / \
 1   2
```

返回 **true**，因为 t 与 s 的一个子树拥有相同的结构和节点值。

**示例 2:**
给定的树 s：

```java
     3
    / \
   4   5
  / \
 1   2
    /
   0
```

给定的树 t：

```java
   4
  / \
 1   2
```

返回 **false**

**解法一**

先上一个错误解法

```java
public boolean isSubtree(TreeNode s, TreeNode t) {
    if (t==null && s==null) {
        return true;
    }
    if (s==null) {
        return false;
    }
    if (s!=null&& t!=null && s.val == t.val) {
        return isSubtree(s.left,t.left) && isSubtree(s.right,t.right);
    }
    return isSubtree(s.left,t) | isSubtree(s.right,t);
}
```
过了146/176的case，但是这个明显是错的，不过核心的递归还是大概雏形写出来了

**解法二**

思考了一会，光速瞄了一眼评论区，隐约看到了有人说双递归，然后想到了下面的解

```java
//简化代码
public boolean isSubtree(TreeNode s, TreeNode t) {
    if (s==null) {
        return false;
    }
    return isSame(s,t)| isSubtree(s.left,t) | isSubtree(s.right,t);
}

public boolean isSame(TreeNode s, TreeNode t){
    if (s==null && t==null) {
        return true;        
    }
    if (s==null || t==null) {
        return false;
    }
    return s.val==t.val && isSame(s.left,t.left) && isSame(s.right,t.right);
}
```

开始的代码没这么简洁，比较罗嗦，要判断一棵树是不是另一颗的子树很好判断，要么s和t直接相等，要么t是s左子树的子树，或者右子树的子树，所以我们还需要一个函数判断两个两棵树是否相等，只用一个函数确实不好实现

**解法三**

其实还有一种解法，也是最开始想到的，就是直接中序遍历和前序遍历，得到两个序列，然后用kmp匹配两棵树，kmp很久没看了，不会写了，后面有时间再来写

## [222. 完全二叉树的节点个数](https://leetcode-cn.com/problems/count-complete-tree-nodes/)

给出一个**完全二叉树**，求出该树的节点个数。

**说明：**

完全二叉树的定义如下：在完全二叉树中，除了最底层节点可能没填满外，其余每层节点数都达到最大值，并且最下面一层的节点都集中在该层最左边的若干位置。若最底层为第 h 层，则该层包含 1~ 2h 个节点。

**示例:**

```java
输入: 
    1
   / \
  2   3
 / \  /
4  5 6

输出: 6
```

**解法一**

BFS，权当复习了

```java
//BFS
public int countNodes(TreeNode root) {
    Queue<TreeNode> queue=new LinkedList<>();
    if (root == null) return 0;
    queue.add(root);
    int count=1;
    while(!queue.isEmpty()){
        int nextLevel=queue.size();
        while(nextLevel>0){
            TreeNode node=queue.poll(); 
            count++;
            if (node.left!=null) {
                queue.add(node.left);
            }
            if (node.right!=null) {
                queue.add(node.right);
            }
            nextLevel--;
        }
    }
    return count;
}
```
**解法二**

递归解法

```java
//暴力
public int countNodes(TreeNode root) {
    if (root==null) return 0;
    return countNodes(root.left)+countNodes(root.right)+1;
}
```
更加精简点可以缩减成一行

**解法三**

这题是mid难度，而且题目给的条件还没用上：**这是一颗完全二叉树**，所以我们可以利用它的性质来做，众所周知，**满二叉树的节点个数**可以直接根据公式 `2^H-1` 计算得来，所以我们只要判断当前的完全二叉树是不是**满二叉树**，如果是直接算出来，这样就可以省去中间很多节点的遍历

```java
//利用完全二叉树的性质
public int countNodes(TreeNode root) {
    if (root==null) return 0;
    TreeNode left=root.left;
    TreeNode right=root.right;
    int hight=0;
    while(left!=null && right!=null){
        left=left.left;
        right=right.right;
        hight++;
    }
    //同时向左向右走，走到最后left==null就说明这颗树是满二叉树，可以利用公式直接求出节点个数
    //否则就对其左右子树递归求解
    return left==null?(1<<hight)-1:countNodes(root.left)+countNodes(root.right)+1;
}
```
不得不说这样的方式还是挺巧妙的，时间复杂度应该是`O(2logN)`? 

## [110. 平衡二叉树](https://leetcode-cn.com/problems/balanced-binary-tree/)

给定一个二叉树，判断它是否是高度平衡的二叉树。

本题中，一棵高度平衡二叉树定义为：

>  一个二叉树每个节点 的左右两个子树的高度差的绝对值不超过1

**示例 1:**

给定二叉树 [3,9,20,null,null,15,7]

```java
    3
   / \
  9  20
    /  \
   15   7
```

返回 true 。

**示例 2:**

给定二叉树 [1,2,2,3,3,null,null,4,4]

```java
       1
      / \
     2   2
    / \
   3   3
  / \
 4   4
```


返回 false 。

**解法一**

暴力法，结合上面的[二叉树最大深度](#104. 二叉树的最大深度)，**自顶向下**，求左右子树的高度差

```java
//top 2 bottom
public boolean isBalanced(TreeNode root) {
    if (root==null) return true;
    if (Math.abs(hight(root.left)-hight(root.right))>1) {
        return false;
    }
    return isBalanced(root.left) && isBalanced(root.right);
}

public int hight(TreeNode root){
    if (root==null) {
        return 0;
    }
    return Math.max(hight(root.right),hight(root.left))+1;
}
```
自顶向下，先判断根节点，然后判断左右子树，很明显。在判断左右子树的时候，会重复的遍历判断根节点的时候已经遍历过的节点，时间复杂度应该是`O(N^2)`

**解法二**

自底向上，利用一个实例变量保存结果，其实就是在上面的求heigh过程中将左右子树的高度先取出来直接比较，如果差距大于1就直接记录下结果false，但是其实这里还是可以优化下

```java
private boolean ans=true;

//buttom 2 top
public boolean isBalanced(TreeNode root) {
    if (root==null) return true;
    hight(root);
    return ans;
}

public int hight(TreeNode root){
    if (root==null) {
        return 0;
    }
    //递归分治，自底向上，在求高度的过程中计算左右高度差
    int left=hight(root.left);
    int right=hight(root.right);
    if (Math.abs(left-right)>1) {
        ans=false;
    }
    return Math.max(left,right)+1;
}
```
自底向上，只需要遍历一遍二叉树就可以得到结果，时间复杂度`O(N)` 

**解法三**

```java
public boolean isBalanced(TreeNode root) {
    if (root==null) return true;
    return hight(root)!=-1;
}

public int hight(TreeNode root){
    if (root==null) {
        return 0;
    }
    int left=hight(root.left);
    if (left==-1) {
        return -1;
    }
    int right=hight(root.right);
    if (right==-1) {
        return -1;
    }
    return Math.abs(left-right)>1?-1:Math.max(left,right)+1;
}
```

在不符合的时候一路`return -1` 节省后面的计算

## [563. 二叉树的坡度](https://leetcode-cn.com/problems/binary-tree-tilt/)

给定一个二叉树，计算整个树的坡度。

一个树的节点的坡度定义即为，该节点左子树的结点之和和右子树结点之和的差的绝对值。空结点的的坡度是0。

整个树的坡度就是其所有节点的坡度之和。

**示例:**

```java
输入: 
         1
       /   \
      2     3
输出: 1
解释: 
结点的坡度 2 : 0
结点的坡度 3 : 0
结点的坡度 1 : |2-3| = 1
树的坡度 : 0 + 0 + 1 = 1
```

**注意:**

1. 任何子树的结点的和不会超过32位整数的范围。
2. 坡度的值不会超过32位整数的范围。. 

**解法一**

很快写出来的解法，发现这题和上面的 **平衡二叉树** 有异曲同工之妙！

```java
//首先想到的解法
public int findTilt(TreeNode root) {
    if (root==null) {
        return 0;
    }
    return findTilt(root.left)+findTilt(root.right)+Math.abs(childSum(root.left)-childSum(root.right));
}

public int childSum(TreeNode root) {
    if (root==null) {
        return 0;
    }
    return childSum(root.left)+childSum(root.right)+root.val;
}
```
嵌套递归，相当暴力

**解法二**

上面的做法确实有点可惜，其实在计算childSum的时候就可以字节把坡度算出来然后累加就是整体的坡度

```java
int tilt=0;

//结果发现上面的做法傻逼了。。。其实我知道是不对的,但是不知道咋改,不过写了个嵌套递归也还行hahaha
public int findTilt(TreeNode root) {
    childSum(root);
    return tilt;
}

public int childSum(TreeNode root) {
    if (root==null) {
        return 0;
    }
    int left=childSum(root.left);
    int right=childSum(root.right);
    tilt+=Math.abs(left-right);
    return left+right+root.val;
}
```

## [112. 路径总和](https://leetcode-cn.com/problems/path-sum/)

给定一个二叉树和一个目标和，判断该树中是否存在**根节点到叶子节点**的路径，这条路径上所有节点值相加等于目标和。

**说明:** 叶子节点是指没有子节点的节点。

**示例:** 
给定如下二叉树，以及目标和 `sum = 22`

```java
          5
         / \
        4   8
       /   / \
      11  13  4
     /  \      \
    7    2      1
```
返回 true, 因为存在目标和为 22 的根节点到叶子节点的路径 `5->4->11->2`

**解法一**

递归解法

```java
public boolean hasPathSum(TreeNode root, int sum) {
    if (root==null) {
        return false;
    }
    //需要注意这里的叶节点判断
    if (root.left==null && root.right==null&&root.val==sum) {
        return true;
    }
    return hasPathSum(root.left,sum-root.val) || hasPathSum(root.right,sum-root.val);
}
```
值得注意的地方就是这个叶子节点的判断，一开始没注意到，直接写的 `root.val==sum` ，其实如果不是叶子节点的话，其实是不成立的，比如

```java
  1
 /
2       sum=1
```

其实这就是`false` ，因为他没有右子树，而题目要求的是从**根节点到叶子节点**

## [404. 左叶子之和](https://leetcode-cn.com/problems/sum-of-left-leaves/)

计算给定二叉树的所有左叶子之和。

**示例：**

```java
    3
   / \
  9  20
    /  \
   15   7
```

在这个二叉树中，有两个左叶子，分别是 9 和 15，所以返回 24

**解法一**

说实话，这些题给的例子都挺误导人的，会让人不自觉地忽略**叶子节点**这个条件😂

```java
private int sum=0;

public int sumOfLeftLeaves(TreeNode root){
    sumOfLeft(root);
    return sum;
};

public void sumOfLeft(TreeNode root) {
    if (root==null) {
        return;
    }
    //注意这里的条件！！！！
    if (root.left!=null && root.left.left==null &&root.left.right==null) {
        sum+=root.left.val;
    }
    sumOfLeft(root.left);
    sumOfLeft(root.right);
}
```
## [257. 二叉树的所有路径](https://leetcode-cn.com/problems/binary-tree-paths/)

给定一个二叉树，返回所有从根节点到叶子节点的路径。

**说明:** 叶子节点是指没有子节点的节点。

**示例:**

```java
 输入:

   1
 /   \
2     3
 \
  5

输出: ["1->2->5", "1->3"]

解释: 所有根节点到叶子节点的路径为: 1->2->5, 1->3
```

**解法一**

递归DFS的解法

```java
//DFS
public List<String> binaryTreePaths(TreeNode root) {
    List<String> res=new ArrayList<>();
    if (root==null) return res;

    if (root!=null&&root.left==null&&root.right==null) {
        res.add(String.valueOf(root.val));
        return res;
    }
    //左子树的所有路径
    List<String> lefts=binaryTreePaths(root.left);
    //右子树的所有路径
    List<String> rights=binaryTreePaths(root.right);
    
    //在每条路径前面加上当前根节点
    for (int i=0;i<lefts.size();i++) {
        res.add(root.val+"->"+lefts.get(i));
    }
    for (int i=0;i<rights.size();i++) {
        res.add(root.val+"->"+rights.get(i));
    }
    return res;
}
```
比上面的递归稍微复杂点，核心思想还是要抓住递归的本质，不要去纠结递归每一步都是怎么得到的，从宏观上去写代码，还是要多练啊

**解法二**

BFS广搜

```java
public List<String> binaryTreePaths(TreeNode root) {
    List<String> res=new ArrayList<>();
    if (root==null) return res;
    Stack<TreeNode> node_stack=new Stack<>();
    Stack<String> path_stack=new Stack<>();
    node_stack.add(root);
    path_stack.add(String.valueOf(root.val));
    String path="";
    while(!node_stack.isEmpty()){
        TreeNode node=node_stack.pop();
        path=path_stack.pop();
        //叶子节点，这条路径搜索结束，添加到res中
        if (node.left==null&&node.right==null) {
            res.add(path);
        }
        
        if (node.left!=null) {
            node_stack.add(node.left);
            path_stack.add(path+"->"+node.left.val);
        }
        if (node.right!=null) {
            node_stack.add(node.right);
            path_stack.add(path+"->"+node.right.val);
        }
    }
    return res;
}
```
这里和传统的BFS不太一样，是用的栈来遍历的

## [113. 路径总和 II](https://leetcode-cn.com/problems/path-sum-ii/)

给定一个二叉树和一个目标和，找到所有从根节点到叶子节点路径总和等于给定目标和的路径。

**说明:** 叶子节点是指没有子节点的节点

**示例:**
给定如下二叉树，以及目标和 `sum = 22`，

```java
          5
         / \
        4   8
       /   / \
      11  13  4
     /  \    / \
    7    2  5   1
```
返回:

```java
[
   [5,4,11,2],
   [5,8,4,5]
]
```

**解法一**

和上一题的做法基本一致，本来应该是一遍bugfree的，编译错误整了半天

```java
public List<List<Integer>> pathSum(TreeNode root, int sum) {
    List<List<Integer>> res=new LinkedList<>();
    if (root==null) {
        return res;
    }
    if (root!=null && root.left==null && root.right==null && root.val==sum) {
        LinkedList<Integer>  lis= new LinkedList<>(); 
        lis.add(root.val);
        res.add(lis);
        return res;
    }
    //左右子树符合条件的路径
    List<List<Integer>> lefts=pathSum(root.left,sum-root.val);
    List<List<Integer>> rights=pathSum(root.right,sum-root.val);

    for (int i=0;i<lefts.size();i++) {
        ((LinkedList<Integer>)lefts.get(i)).addFirst(root.val);
        res.add(lefts.get(i));
    }

    for (int i=0;i<rights.size();i++) {
        ((LinkedList<Integer>)rights.get(i)).addFirst(root.val);
        res.add(rights.get(i));
    }
    return res;
}
```
**解法二**

废了老大劲终于把BFS写出来了。。。可以看出还是借鉴的上面的思路

```java
public List<List<Integer>> pathSum2(TreeNode root,int sum) {
    List<List<Integer>> res=new LinkedList<>();
    if (root==null) return res;
    //节点栈
    Stack<TreeNode> node_stack=new Stack<>();
    //路径栈
    Stack<List<Integer>> path_stack=new Stack<>();
    //节点sum栈
    Stack<Integer> sum_stack=new Stack<>();
    //给每个栈存入初始值
    node_stack.add(root);
    path_stack.add(new LinkedList(){{
        add(root.val);
    }});
    sum_stack.add(root.val);
    //BFS
    while(!node_stack.isEmpty()){
        TreeNode node=node_stack.pop();
        List<Integer> pathList=path_stack.pop();
        int tempS=sum_stack.pop();
        //终止条件
        if (node.left==null && node.right==null&&tempS==sum) {
            res.add(pathList);
            continue;
        }
        if (node.left!=null) {
            //这三个栈是同步的,node栈存放当前节点
            //path栈存放根节点到当前节点的路径
            //sum栈存放的是path栈中所有节点的val和
            node_stack.add(node.left);
            //这里不要直接操作pathList,否则左右的路径会混在一起
            LinkedList<Integer> tlis= new LinkedList(pathList);
            tlis.add(node.left.val);
            path_stack.add(tlis);
            //累加路径上的节点值
            sum_stack.add(tempS+node.left.val);
        }
        if (node.right!=null) {
            node_stack.add(node.right);
            //同上
            LinkedList<Integer> tlis= new LinkedList(pathList);
            tlis.add(node.right.val);
            path_stack.add(tlis);
            sum_stack.add(tempS+node.right.val);
        }
    }
    return res;
}
```
用到三个栈，同步保存节点的信息，还是挺简单的

## [129. 求根到叶子节点数字之和](https://leetcode-cn.com/problems/sum-root-to-leaf-numbers/)

给定一个二叉树，它的每个结点都存放一个 `0-9` 的数字，每条从根到叶子节点的路径都代表一个数字。

例如，从根到叶子节点路径 `1->2->3` 代表数字 `123`

计算从根到叶子节点生成的所有数字之和。

**说明:** 叶子节点是指没有子节点的节点。

**示例 1:**

```java
输入: [1,2,3]
    1
   / \
  2   3
输出: 25
解释:
从根到叶子节点路径 1->2 代表数字 12.
从根到叶子节点路径 1->3 代表数字 13.
因此，数字总和 = 12 + 13 = 25.
```

**示例 2:**

```java
输入: [4,9,0,5,1]
    4
   / \
  9   0
 / \
5   1
输出: 1026
解释:
从根到叶子节点路径 4->9->5 代表数字 495.
从根到叶子节点路径 4->9->1 代表数字 491.
从根到叶子节点路径 4->0 代表数字 40.
因此，数字总和 = 495 + 491 + 40 = 1026.
```

**解法一**

BFS，延续上面的做法

```java
public int sumNumbers(TreeNode root) {
    int res=0;
    if (root == null ) return res;
    Stack<TreeNode> node_stack=new Stack<>();
    Stack<Integer> sum_stack=new Stack<>();
    node_stack.add(root);
    sum_stack.add(root.val);
    while(!node_stack.isEmpty()){
        TreeNode node=node_stack.pop();
        int tempS=sum_stack.pop();
        if (node.left==null && node.right==null) {
            res+=tempS;
            continue;
        }
        if (node.left!=null) {
            node_stack.add(node.left);
             //注意*10,在上一层的基础上*10
            sum_stack.add(tempS*10+node.left.val);
        }
        if (node.right!=null) {
            node_stack.add(node.right);
            sum_stack.add(tempS*10+node.right.val);
        }
    }
    return res;
}
```
**解法二**

DFS解法，一开始没想出来。。。

```java
private int sum=0;
//DFS
public int sumNumbers2(TreeNode root) {
    sumNumber(0,root);
    return sum;
}

public void sumNumber(int parent,TreeNode root) {
    if (root==null) {
        return;
    }
    int cur=parent*10+root.val;
    //叶子节点
    if (root!=null && root.left==null&& root.right==null) {
        sum+=cur;
    }
    sumNumber(cur,root.left);
    sumNumber(cur,root.right);
}
```

## [437. 路径总和 III](https://leetcode-cn.com/problems/path-sum-iii/)

给定一个二叉树，它的每个结点都存放着一个整数值。

找出路径和等于给定数值的路径总数。

路径不需要从根节点开始，也不需要在叶子节点结束，但是路径方向必须是向下的（只能从父节点到子节点）。

二叉树不超过1000个节点，且节点数值范围是 [-1000000,1000000] 的整数。

**示例：**

```java
root = [10,5,-3,3,2,null,11,3,-2,null,1], sum = 8
      10
     /  \
    5   -3
   / \    \
  3   2   11
 / \   \
3  -2   1
```

返回 3。和等于 8 的路径有:

1.  5 -> 3
2.  5 -> 2 -> 1
3.  -3 -> 11

**解法一**

```java
public int pathSum(TreeNode root, int sum) {
    if (root == null ) {
        return 0;
    }
    int res=findPath(root,sum);
    res+=pathSum(root.left,sum);
    res+=pathSum(root.right,sum);
    return res;
}

public int findPath(TreeNode node,int sum){
    int res=0;
    if (node==null) {
        return res;
    }
    if (node.val==sum) {
        res++;
    }
    res+=findPath(node.left,sum-node.val);
    res+=findPath(node.right,sum-node.val);
    return res;
}
```
emmmm，这题分类是easy确实太迷了，嵌套的递归，看了解法确实看的懂，但是写是绝对写不出来的（眼睛：我懂了，脑子：你懂个锤子）除非能记住

## [235. 二叉搜索树的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-search-tree/)

给定一个二叉搜索树, 找到该树中两个指定节点的最近公共祖先。

百度百科中最近公共祖先的定义为：对于有根树 T 的两个结点 p、q，最近公共祖先表示为一个结点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（**一个节点也可以是它自己的祖先**）

例如，给定如下二叉搜索树:  `root = [6,2,8,0,4,7,9,null,null,3,5]`

![mark](http://static.imlgw.top/blog/20191001/KlQJmqmdWmP3.png?imageslim)

**示例 1:**

```java
输入: root = [6,2,8,0,4,7,9,null,null,3,5], p = 2, q = 8
输出: 6 
解释: 节点 2 和节点 8 的最近公共祖先是 6
```

**示例 2:**

```java
输入: root = [6,2,8,0,4,7,9,null,null,3,5], p = 2, q = 4
输出: 2
解释: 节点 2 和节点 4 的最近公共祖先是 2, 因为根据定义最近公共祖先节点可以为节点本身。
```

**说明:**

- 所有节点的值都是唯一的。
- p、q 为不同节点且均存在于给定的二叉搜索树中。

**解法一**

看了一点点思路，然后bugfree

```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    //特殊情况,其中一个已经是另一个的祖先了
    //if (p==root || q==root) return root;
    //都小于根节点
    if (p.val<root.val && q.val<root.val) {
        return lowestCommonAncestor(root.left,p,q);
    }else if (p.val > root.val && q.val > root.val) {
        //都大于根节点
        return lowestCommonAncestor(root.right,p,q);
    }else{
        //一大一小 或者有一个是root
        return root;
    }
}
```
其实核心就是利用好BST的性质，左子树一定小于根节点，右子树一定大于根节点，求公共祖先，如果一个节点在左子树，一个在右子树，那么最近的公共祖先一定是root，除此之外，还有一种特殊情况就是当两个节点已经有祖先关系的时候，那么直接返回祖先节点就可以了

> 这里其实前面的`if`可以去掉，题目中说到了所有节点的值都是唯一的，所以节点值相等就说明是同一个节点，就已经包含在最后一个else的情况中了

## [98. 验证二叉搜索树](https://leetcode-cn.com/problems/validate-binary-search-tree/)

给定一个二叉树，判断其是否是一个有效的二叉搜索树。

假设一个二叉搜索树具有如下特征：

- 节点的左子树只包含**小于**当前节点的数。
- 节点的右子树只包含**大于**当前节点的数。
- 所有左子树和右子树自身必须也是二叉搜索树。

**示例 1:**

```java
输入:
    2
   / \
  1   3
输出: true
```

**示例 2:**

```java
输入:
    5
   / \
  1   4
     / \
    3   6
输出: false
解释: 输入为: [5,1,4,null,null,3,6]。
     根节点的值为 5 ，但是其右子节点值为 4 
```

**解法一**

递归解法，很巧妙

```java
public boolean isValidBST(TreeNode root) {
    if (root==null) {
        return true;
    }
    return isValidBST(root,null,null);
}

public boolean isValidBST(TreeNode node,Integer low,Integer high){
    if (node==null) return true;
    if (low!=null && low>=node.val || high!=null && high<=node.val) {
        return false;
    }
    return isValidBST(node.left,low,node.val) && isValidBST(node.right,node.val,high);
}
```
一定要注意BST的性质是根节点**大于所有** 右子树的节点，**小于所有**左子树的节点，而不是简单的验证当前节点和左右节点的大小关系就可以了，所以我们在验证的时候传入对应的**上界**和**下界**，节点必须要大于下界，小于上界，那么上界和下界从哪里来？_当前节点就是左子树的上界，右子树的下界！_  然后递归左右子树就ok了

> 这题其实还有一个坑，只不过我这个做法直接跳过了，题目的case中有的节点值是`Integer.MIN_VALUE`，和`Integer.MAX_VALUE`  ，如果上界下界直接用int来传递的话，很有可能递归初始调用就是这样的
>
> `return isValidBST(root,Integer.MIN_VALUE,Integer.MAX_VALUE);` 这就正中出题人下怀，所以我们这里用一个包装类型，这样我们只需要检测上界下界是不是null就可以了

**解法二**

这个就利用了BST和中序遍历的关系，我们知道中序遍历是 `左->根->右`  这个顺序放到 BST中恰好就是一个升序的序列，所以我们就可以利用这个性质来判断二叉树是不是BST

```java
//BST的中序遍历一定是升序的
public boolean isValidBST(TreeNode root){
    LinkedList<Integer> order=new LinkedList<>();
    if (root==null) {
        return true;
    }
    Stack<TreeNode> stack=new Stack<>();
    TreeNode cur=root;
    while(cur!=null || !stack.isEmpty()){
        while(cur!=null){
            stack.add(cur);
            cur=cur.left;
        }
        cur=stack.pop();
        //和上一次的最后一个节点值比较
        if (!order.isEmpty() && order.getLast()>= cur.val) {
            return false;
        }
        order.add(cur.val);
        cur=cur.right;
    }
    return true;
}
```
这里其实可以不用list保存结果，用一个int保存上一次的节点值就行了

## [108. 将有序数组转换为二叉搜索树](https://leetcode-cn.com/problems/convert-sorted-array-to-binary-search-tree/)

将一个按照升序排列的有序数组，转换为一棵**高度平衡二叉搜索树**

本题中，一个高度平衡二叉树是指一个二叉树每个节点 的左右两个子树的高度差的绝对值不超过 1。

**示例:**

```java
给定有序数组: [-10,-3,0,5,9]
一个可能的答案是：[0,-3,9,-10,null,5]，它可以表示下面这个高度平衡二叉搜索树：
      0
     / \
   -3   9
   /   /
 -10  5
```

**解法一**

```java
public TreeNode sortedArrayToBST(int[] nums) {
    return sortedArrayToBST(nums,0,nums.length-1);
}

public TreeNode sortedArrayToBST(int[] nums,int left,int right) {
    if (left>right) {
        return null;
    }
    int mid=(right-left)/2+left;
    TreeNode node=new TreeNode(nums[mid]);
    node.left=sortedArrayToBST(nums,left,mid-1);
    node.right=sortedArrayToBST(nums,mid+1,right);
    return node;
}
```
这题最开始终止条件写错了，思路是对的，对递归运用的还是不够熟练，终止条件其实只需要想一下极端情况就可以了

## [230. 二叉搜索树中第K小的元素](https://leetcode-cn.com/problems/kth-smallest-element-in-a-bst/)

给定一个二叉搜索树，编写一个函数 `kthSmallest` 来查找其中第 k 个最小的元素

**说明：**
你可以假设 k 总是有效的，1 ≤ k ≤ 二叉搜索树元素个数。

**示例 1:**

```java
输入: root = [3,1,4,null,2], k = 1
   3
  / \
 1   4
  \
   2
输出: 1
```


**示例 2:**

```java
输入: root = [5,3,6,2,4,null,null,1], k = 3
       5
      / \
     3   6
    / \
   2   4
  /
 1
输出: 3
```


**进阶：**
如果二叉搜索树经常被修改（插入/删除操作）并且你需要频繁地查找第 k 小的值，你将如何优化 kthSmallest 函数？

**解法一**

非递归中序遍历

```java
public int kthSmallest(TreeNode root, int k) {
    Stack<TreeNode> stack=new Stack<>();
    TreeNode cur=root;
    int count=0;
    while(cur!=null || !stack.isEmpty()){
        while(cur!=null){
            stack.add(cur);
            cur=cur.left;
        }
        cur=stack.pop();
        if (count==k-1) {
            return cur.val;
        }
        cur=cur.right;
        count++;
    }
    //没找到
    return -1;
}
```
还是利用BST中序遍历是升序的性质，在取到第k个元素的时候就直接`break`

**解法二**

递归的方式，加了两个额外的实例变量其实不太好

```java
public int kthSmallest(TreeNode root, int k) {
    kthSmallest(root,k);
    return res;
}

private int count=0;

private int res=0;

public void kthSmall(TreeNode root, int k) {
    if (root==null) {
        return;
    }
    kthSmall(root.left,k);
    if (count==k-1) {
        res=root.val;
        return;
    }
    count++;
    kthSmall(root.right,k);
}
```
**进阶**

可以维护一个大根堆，就和最小栈一样，每次对BST操作的时候同步操作这个大根堆

## [236. 二叉树的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree/)

给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

百度百科中最近公共祖先的定义为：“对于有根树 T 的两个结点 p、q，最近公共祖先表示为一个结点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（一个节点也可以是它自己的祖先）。”

例如，给定如下二叉树:  root = [3,5,1,6,2,0,8,null,null,7,4]

 ![mark](http://static.imlgw.top/blog/20191003/m0bWNSMUQWy2.png?imageslim)

**示例 1:**

```java
输入: root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1
输出: 3
解释: 节点 5 和节点 1 的最近公共祖先是节点 3。
```

**示例 2:**

```java
输入: root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 4
输出: 5
解释: 节点 5 和节点 4 的最近公共祖先是节点 5。因为根据定义最近公共祖先节点可以为节点本身。
```

**说明:**

- 所有节点的值都是唯一的。
- p、q 为不同节点且均存在于给定的二叉树中。

**解法一**

```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    if (root == null || p==root ||q==root) {
        return root;
    }
    TreeNode left=lowestCommonAncestor(root.left,p,q);
    TreeNode right=lowestCommonAncestor(root.right,p,q);
    if (left!=null && right!=null) {
        return root;
    }else if (left!=null) {
        return left;
    }else if (right!=null) {
        return right;
    }
    return null;
}
```
这题，说实话，我是想不出来

在左、右子树中分别查找是否包含p或q：

- 如果以下两种情况（左子树包含p，右子树包含q/左子树包含q，右子树包含p），那么此时的根节点就是最近公共祖先
- 如果左子树包含p和q，那么到root->left中继续查找，最近公共祖先在左子树里面
- 如果右子树包含p和q，那么到root->right中继续查找，最近公共祖先在右子树里面 

## [101. 对称二叉树](https://leetcode-cn.com/problems/symmetric-tree/)

给定一个二叉树，检查它是否是镜像对称的。

例如，二叉树 `[1,2,2,3,4,4,3]` 是对称的

```java
    1
   / \
  2   2
 / \ / \
3  4 4  3
```


但是下面这个 `[1,2,2,null,3,null,3]` 则不是**镜像对称**的:

```java
    1
   / \
  2   2
   \   \
   3    3
```

**说明:**

如果你可以运用递归和迭代两种方法解决这个问题，会很加分。

**解法一**

哎，感觉刷题还是得在白天，脑子清醒点，下午就感觉做题老是出问题，一开始题都没看清就开始做

其实一开始是想BFS层次遍历然后判断每一层是不是镜像对称的，然后发现有些case是过不了的，比如

`[1,3,3,2,null,2]` 这样的case

```java
    1
   / \
  3   3
 /   /
2   2
```

~~层序遍历判断不出了这样的case~~   下面解法四打脸

然后换一种遍历方式，其实一开始就想到了前序遍历，如果是镜像对称的话，前序遍历刚好就是对称的，但是！！！还是有case过不了！！！我们再看上面的case，我们改一改

```java
    1
   / \
  2   2
 /   /
2   2
```

第`192/195个case`，我惊了，居然还有这种操作！！！实在没办法翻了下解答，发现有位老兄也是这样做的，然后他很巧妙的在每个节点值后面加了一个**层数**，他好像是直接当作字符串添加的，我感觉不太好，改用了数组，最后判断的时候需要保证层数和值都相同才行，完整代码如下

```java
//[1,2,2,2,null,2] 忘了还有这样的case了,哭了
public boolean isSymmetric(TreeNode root) {
    if (root == null) {
        return true;
    }
    List<Integer[]> lis=new ArrayList<>();
    preTravle(root,lis,0); 
    for (int i=0,j=lis.size()-1;i<=j;i++,j--) {
        if (lis.get(i)[0]!= lis.get(j)[0] ||  lis.get(j)[1]!= lis.get(i)[1]) {
            return false;
        }
    }
    return true;
}

//前序遍历
public void preTravle(TreeNode node,List<Integer[]> lis,int k){
    if (node!=null) {
        preTravle(node.left,lis,k+1);
        //这里其实用个字符串就可以，但是感觉拼接的效率不高，而且，是不是有可能出现问题？
        // 11+3 ==1+13 ？？？是不是有可能出现类似这样的情况
        Integer[] temp=new Integer[2];
        temp[0]=node.val;
        temp[1]=k;
        lis.add(temp);
        preTravle(node.right,lis,k+1);
    }
}
```

**解法二**

递归的解法，应该算是官解了，一开始也是想用递归写的，没抓住问题的本质，太菜了

```java
public boolean isSymmetric(TreeNode root) {
    if (root ==null) {
        return true;
    }
    //转换为求左右子树是否镜像对称的问题
    return isSymmetric(root.left,root.right);
    //return isSymmetric(root,root);
}

//dfs
public boolean isSymmetric(TreeNode t1,TreeNode t2) {
    if (t1==null && t2==null) {
        return true;
    }
    //有一个为null
    if (t1== null || t2==null) {
        return false;
    }
    //都不为null
    return t1.val==t2.val && isSymmetric(t1.left,t2.right) && isSymmetric(t1.right,t2.left);
}
```
一棵树是镜像对称，说明左右子树左右对称，所以这个问题就可以转换为，判断左右两颗子树是否是镜像对称的问题

![mark](http://static.imlgw.top/blog/20191107/Ae6daB6SuXdl.png?imageslim)

判断两颗树是否成镜像对称的话，其实就和照镜子一样的，如上图，判断左子树和右子树是否成镜像对称，就需要判断`t1的左子树和t2的右子树是否镜像对称,t1的右子树和t2的左子树是否镜像对称`，根据这个就可以写出递归函数，还是挺妙的

**解法三**

类似于层次遍历，其实就是根据上面的递归方法改来的，核心思想和上面递归的是一样的

```java
//非递归解法
public boolean isSymmetric(TreeNode root) {
    if (root ==null) {
        return true;
    }
    Stack<TreeNode> stack=new Stack<>();
    stack.push(root.left);
    stack.push(root.right);
    while(!stack.isEmpty()){
        TreeNode t1=stack.pop();
        TreeNode t2=stack.pop();
        if (t1==null && t2==null) {
            continue;
        }
        if (t1==null||t2==null || t1.val!=t2.val) {
            return false;
        }
        /*if (t1.val!=t2.val) {
                return false;
         }*/
        stack.push(t1.left);
        stack.push(t2.right);
        stack.push(t1.right);
        stack.push(t2.left);
    }
    return true;
}       
```

**解法四**

前序遍历，其实我一开始也想到了用占位的方式，但是因为之前遍历方式不同，导致没想好在哪里加

```java
public boolean isSymmetric(TreeNode root) {
    if (root ==null) {
        return true;
    }
    Queue<TreeNode> queue=new LinkedList<>();
    queue.add(root);
    while(!queue.isEmpty()){
        int count=queue.size();
        ArrayList<Integer> lis=new ArrayList<>();
        while(count>0){
            TreeNode node=queue.poll();
            //lis.add(node.val);
            if (node.left!=null) {
                queue.add(node.left);
                lis.add(node.left.val);
            }else{
                //-1占位
                lis.add(-1);
            }
            
            if (node.right!=null) {
                queue.add(node.right);
                lis.add(node.right.val);
            }else{
                //为空加-1占位
                lis.add(-1);
            }
            count--;
        }
        //对每一层经常判断
        for (int i=0,j=lis.size()-1;i<=j;i++,j--) {
            if (lis.get(i)!=lis.get(j)) {
                return false;
            }
        }
    }
    return true;
}
```
这样的做法明显时间复杂度会之前要高，不仅遍历了整颗树一遍，还对每一层遍历了一遍，一共遍历了两遍

## [513. 找树左下角的值](https://leetcode-cn.com/problems/find-bottom-left-tree-value/)

给定一个二叉树，在树的最后一行找到最左边的值。

示例 1:
```java
输入:
    2
   / \
  1   3
输出:
1
```

示例 2:
```java
输入:
        1
       / \
      2   3
     /   / \
    4   5   6
       /
      7
输出:
7
```
**注意:** 您可以假设树（即给定的根节点）不为 NULL。

**解法一**

这种题写一百遍了😂，然而我还是没有bugfree

```java
//其实这种层序遍历的方式对这题有一点小题大作，不过我还是比较习惯这种方式
public int findBottomLeftValue(TreeNode root) {
    Queue<TreeNode> queue=new LinkedList<>();
    queue.add(root);
    int res=-1;
    while(!queue.isEmpty()){
        int count=queue.size();
        int temp=count;
        while(count>0){
            TreeNode node=queue.poll();
            if (count==temp) {
                res=node.val;
            }
            if (node.left!=null) {
                queue.add(node.left);
            }
            if (node.right!=null) {
                queue.add(node.right);
            }
            count--;
        }
    }
    return res;   
}
```
**解法二**

最左边的值，也就是最后一行的第一个元素，dfs深度优先，深度每增加一次就更新一次res

```java
//DFS
public int findBottomLeftValue2(TreeNode root) {
    dfs(root,0);
    return res;
}

int res=-1,max=Integer.MIN_VALUE;

public void dfs(TreeNode node,int depth){
    if (node==null) return;
    if (depth>max) {
        max=depth;
        res=node.val;
    }
    dfs(node.left,depth+1);
    dfs(node.right,depth+1);
}
```

## [105. 从前序与中序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

根据一棵树的前序遍历与中序遍历构造二叉树。

**注意:**
你可以假设树中没有重复的元素。

例如，给出

```java
前序遍历 preorder = [3,9,20,15,7]
中序遍历 inorder = [9,3,15,20,7]
```


返回如下的二叉树：

```java
    3
   / \
  9  20
    /  \
   15   7
```
**解法一**

```java
public TreeNode buildTree(int[] preorder, int[] inorder) {
    if (preorder==null) {
        return null;
    }
    return buildTree(preorder,0,preorder.length-1,inorder,0,inorder.length-1);
}

public TreeNode buildTree(int[] preorder,int preleft,int preright,int[] inorder,int inleft,int inright) {
    //递归出口只需要想一下边界,比如只要一个节点的时候,很明显只有一个节点的时候这几个值都是 0
    //但是此时肯定不能返回null,所以这里递归出口不是大于等于,而是大于
    if (preleft>preright || inleft>inright) {
        return null;
    }
    TreeNode root=new TreeNode(preorder[preleft]);
    int index=inleft;
    while(inorder[index] != preorder[preleft]) {
        index++;
    }
    root.left=buildTree(preorder,preleft+1,preleft+index-inleft,inorder,inleft,index-1);
    root.right=buildTree(preorder,preleft+index-inleft+1,preright,inorder,index+1,inright);
    return root;
}
```
这题核心思想就是利用这几种遍历的性质，文字总是苍白的，看个图吧

![mark](http://static.imlgw.top/blog/20191106/pcmvnEK8Hwg8.png?imageslim)

这样一看就清晰了，前序遍历左边第一个节点 `1` 一定是根节点，所以我们首先确定了根节点，然后我们去中序遍历中去找这个根节点（一定有），如上图，我们找到了中间的 `1`然后再根据中序遍历的性质，我们可以就知道，中序遍历中，这个`1` 的左边是 `1` 的左子树，右边是`1` 的右子树，到这里我们就确定了根节点及其左右子树，剩下的就交给递归去完成了😁，我们只需要对左右子树分别递归该过程就可以得到一颗完整的树了

当然这里值得注意的地方就是下标的变换，要十分注意，自己带入几个值试试

## [106. 从中序与后序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)

根据一棵树的中序遍历与后序遍历构造二叉树。

**注意:**
你可以假设树中没有重复的元素。

例如，给出

```java
中序遍历 inorder = [9,3,15,20,7]
后序遍历 postorder = [9,15,7,20,3]
```


返回如下的二叉树：

```java
    3
   / \
  9  20
    /  \
   15   7
```
**解法一**

方法同上，只不过是从后往前了

```java
public TreeNode buildTree(int[] inorder, int[] postorder) {
    if (inorder==null || inorder.length<=0) {
        return null;
    }
    return buildTree(inorder,0,inorder.length-1,postorder,0,postorder.length-1);
}

public TreeNode buildTree(int[] inorder,int inL,int inR, int[] postorder,int pL,int pR) {
    //递归出口只需要想一下边界,比如只要一个节点的时候,很明显只有一个节点的时候这几个值都是0
    //但是此时肯定不能返回null,所以这里递归出口不是大于等于,而是大于
    if (inL>inR || pL>pR) { 
        return null;
    }
    TreeNode root=new TreeNode(postorder[pR]);
    int index=inL;
    while(inorder[index]!=postorder[pR]){
        index++; //一定有,所以不用担心越界的问题
    }
    root.left=buildTree(inorder,inL,index-1,postorder,pL,pL+index-inL-1);
    root.right=buildTree(inorder,index+1,inR,postorder,pL+index-inL,pR-1);
    return root;
}
```
和上面一样没啥好说的

**解法二**

上面两种解法提交后效率都不高，这里去中序遍历中找根节点的操作其实可以用Hash表代替

```java
//hash表优化
HashMap<Integer,Integer> map=new HashMap<>();

public TreeNode buildTree(int[] inorder, int[] postorder) {
    if (inorder==null || inorder.length<=0) {
        return null;
    }
    for (int i=0;i<inorder.length;i++) {
        map.put(inorder[i],i);
    }
    return buildTree(inorder,0,inorder.length-1,postorder,0,postorder.length-1);
}

public TreeNode buildTree(int[] inorder,int inL,int inR, int[] postorder,int pL,int pR) {
    //递归出口只需要想一下边界,比如只要一个节点的时候,很明显只有一个节点的时候这几个值都是相等的
    //但是此时肯定不能返回null,所以这里递归出口不是大于等于,而是大于
    if (inL>inR || pL>pR) { 
        return null;
    }
    TreeNode root=new TreeNode(postorder[pR]);
    int index=map.get(postorder[pR]);
    root.left=buildTree(inorder,inL,index-1,postorder,pL,pL+index-inL-1);
    root.right=buildTree(inorder,index+1,inR,postorder,pL+index-inL,pR-1);
    return root;
}
```

## [889. 根据前序和后序遍历构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-postorder-traversal/)

返回与给定的前序和后序遍历匹配的任何二叉树。

 pre 和 post 遍历中的值是不同的正整数。

**示例：**

```java
输入：pre = [1,2,4,5,3,6,7], post = [4,5,2,6,7,3,1]
输出：[1,2,3,4,5,6,7]
```

**提示：**

- `1 <= pre.length == post.length <= 30`
- `pre[]` 和 `post[]` 都是 1, 2, ..., pre.length 的排列
- 每个输入保证至少有一个答案。如果有多个答案，可以返回其中一个。

**解法一**

和上面的有一点点区别

```java
HashMap<Integer,Integer> map=new HashMap<>();

public TreeNode constructFromPrePost(int[] pre, int[] post) {
    for (int i=0;i<post.length;i++) {
        map.put(post[i],i);
    }
    return constructFromPrePost(pre,0,pre.length-1,post,0,post.length-1);
}

public TreeNode constructFromPrePost(int[] pre,int preL,int preR,int[] post,int postL,int postR){
    if (preL>preR || postL>preR) {
        return null;
    }
    TreeNode root=new TreeNode(pre[preL]);
    if (preL==preR) {
        return root;
    }
    int postIndex=map.get(pre[preL+1]); //这种位置一定要注意溢出
    int len=postIndex-postL;
    root.left=constructFromPrePost(pre,preL+1,preL+1+len,post,postL,postIndex);
    root.right=constructFromPrePost(pre,preL+2+len,preR,post,postIndex+1,postR-1);
    return root;
}
```
![mark](http://static.imlgw.top/blog/20191106/pcmvnEK8Hwg8.png?imageslim)

还是这张图，前序的第一个是根节点，后序的最后一个是根节点，而我们要找的是左右子树的分界线，这里没有中序遍历，乍一看似乎不好确定，其实不然，注意观察前序的第二个节点，也就是左子树的根节点，比如上面的2，对应到后序遍历中其实正好就可以作为左子树的分界线，这样一来就和上面一样了，所以这里的关键就是找到一个划分点

🔔 **有一点需要注意，题目说了这题的结果可能是不唯一的，数据结构的课程里面也讲过，仅凭前序和后序是无法确定一颗二叉树的，但是一定么？**

并不一定，我们题目的case就是个反例，它就可以通过前序和后序唯一的确定这颗二叉树，那什么时候可以确定，什么时候无法确定呢？

无法确定的例子好说，【1，2】，【2，1】这个就无法确定

 ```java
  1              1   
   \            /
    2          2        两种可能都有
 ```

 但是如果是这样的【1，2，3】【2，3，1】这种就可以唯一的确定

 ```java
  1
 / \
2   3  只可能是这种情况
 ```
归纳总结一下，可以发现，**如果这颗二叉树每个节点的度都是0或者2** 那么他就可以通过前序和后序确定，反之就不一定了，因为你只有一个子节点那么就无法确定这个节点是左节点还是右节点，如果没有或者两个都有那么就可以确定了（根据顺序确定，前面的是左后面的是右，但是你只有一个我就不知道是左还是右了）

## [114. 二叉树展开为链表](https://leetcode-cn.com/problems/flatten-binary-tree-to-linked-list/)

给定一个二叉树，原地将它展开为链表。

例如，给定二叉树

```java
	1
   / \
  2   5
 / \   \
3   4   6
```

将其展开为：

```java
1
 \
  2
   \
    3
     \
      4
       \
        5
         \
          6
```

> 题目没抄错，就是这样的，确实题目没有说明按照什么方式展开，但是看case能猜到是前序遍历的方式展开（靠猜的？）

**解法一**

前序遍历，递归的解法，用一个全局变量保存链表的结尾，每次将节点添加到last的后面

```java
TreeNode last=null;

public void flatten(TreeNode root) {
    if (root==null) {
        return;
    }
    if (last!=null) {
        last.left=null;
        last.right=root;
    }
    last=root;
    TreeNode right=root.right;//保存右子树
    flatten(root.left);
    flatten(right);
}
```
需要注意的地方就是需要保存右子树，因为前面的操作将左子树添加到根节点右子树的时候，会导致原本的右子树丢失

**解法二**

变形的后序遍历，递归解法

```java
TreeNode pre=null;

public void flatten(TreeNode root) {
    if (root==null) {
        return;
    }
    flatten(root.right);
    flatten(root.left);
    root.right=pre;
    root.left=null;
    pre=root;
}
```
相比前面的解法，为了不丢失右子树，先遍历右子树，再遍历左子树，整个序列就是`6 5 4 3 2 1`  我们只需要将每个节点的right指向前一个节点就ok了

**解法三**

迭代，我觉得这种解法应该来说是最容易理解的，而且是完全的 `in-place`

```java
public void flatten(TreeNode root) {
    TreeNode mRight=null;
    while(root!=null){
        if (root.left!=null) {
            mRight=root.left;
            //找到左子树的最右节点
            while(mRight.right!=null){
                mRight=mRight.right;
            }
            //将根的右节点接在 mRight.right
            mRight.right=root.right;
            //将root.left接在root.right
            root.right=root.left;
            //左节点置为null
            root.left=null;
        }
        //重复该过程
        root=root.right;
    }
}
```
画个图就是这样

![mark](http://static.imlgw.top/blog/20191108/BIx12P1AYjeX.png?imageslim)

## [116. 填充每个节点的下一个右侧节点指针](https://leetcode-cn.com/problems/populating-next-right-pointers-in-each-node/)

给定一个**完美二叉树**，其所有叶子节点都在同一层，每个父节点都有两个子节点。二叉树定义如下：

```java
struct Node {
  int val;
  Node *left;
  Node *right;
  Node *next;
}
```


填充它的每个 next 指针，让这个指针指向其下一个右侧节点。如果找不到下一个右侧节点，则将 next 指针设置为 NULL

初始状态下，所有 next 指针都被设置为 NULL

![leet](https://i.loli.net/2019/11/10/eC4VBqXmwuspZlG.png)

**解法一**

开始没做出来，菜！！！然后特意留到今天总结，又在web上提交了一遍

```java
public Node connect(Node root) {
    if (root ==null ||root.left==null) {
        return root;
    }
    root.left.next=root.right;
    if (root.next!=null) {
        root.right.next=root.next.left;   
    }
    connect(root.left);
    connect(root.right);
    return root;
}
```
**解法二**

这个解法梳理还是很清奇的，类似拉拉链的过程

```java
public Node connect(Node root) {
    if (root ==null ||root.left==null) {
        return root;
    }
    Node left=root.left;
    Node right=root.right;
    //有的像拉拉链的过程
    while(left!=null){
        left.next=right;
        left=left.right;
        right=right.left;
    }
    connect(root.left);
    connect(root.right);
    return root;
}
```

## [450. 删除二叉搜索树中的节点](https://leetcode-cn.com/problems/delete-node-in-a-bst/)

给定一个二叉搜索树的根节点 root 和一个值 key，删除二叉搜索树中的 key 对应的节点，并保证二叉搜索树的性质不变。返回二叉搜索树（有可能被更新）的根节点的引用。

一般来说，删除节点可分为两个步骤：

1. 首先找到需要删除的节点；

2. 如果找到了，删除它。

**说明：** 要求算法时间复杂度为 O(h)，h 为树的高度。

**示例:**

```java
root = [5,3,6,2,4,null,7]
key = 3

    5
   / \
  3   6
 / \   \
2   4   7

给定需要删除的节点值是 3，所以我们首先找到 3 这个节点，然后删除它。

一个正确的答案是 [5,4,6,2,null,null,7], 如下图所示。

    5
   / \
  4   6
 /     \
2       7

另一个正确答案是 [5,2,6,null,4,null,7]。

    5
   / \
  2   6
   \   \
    4   7

```

**解法一**

更多解释看另一篇 [二叉搜索树](http://imlgw.top/2019/11/08/er-fen-sou-suo-shu/#%E5%88%A0%E9%99%A4%E4%BB%BB%E6%84%8F%E5%80%BC)

```java
public TreeNode deleteNode(TreeNode root, int key) {
    if (root ==null) {
        return null;           
    }
    if (root.val>key) {
        root.left=deleteNode(root.left,key);
    }else if (root.val<key) {
        root.right=deleteNode(root.right,key);
    }else{
        if (root.left==null) {
            return root.right;
        }
        if (root.right==null) {
            return root.left;
        }
        //用右子树的最小值填补删除的元素
        TreeNode delNode=root;
        root=getMin(root.right);
        //下面的left和right不能交换,还好刚开始写错了一波,不然也不会发现,哈哈啊哈哈哈
        //这里的deleteMin是为了删除delNode的最小值root,如果你先把delNode.left连接到了root.left
        //那么root就不再是最小值了,再进行deleteMin就会导致root无法删除,最后返回root,导致root.right=root形成环
        //结果无法打印
        root.right=deleteMin(delNode.right);
        root.left=delNode.left;
    }
    return root;
}

public TreeNode deleteMin(TreeNode node){
    if (node.left==null) {
        return node.right;
    }
    node.left=deleteMin(node.left);
    return node;
}

public TreeNode getMin(TreeNode node){
    if (node.left==null) {
        return node;
    }
    return getMin(node.left);
}
```

## [701. 二叉搜索树中的插入操作](https://leetcode-cn.com/problems/insert-into-a-binary-search-tree/)

给定二叉搜索树（BST）的根节点和要插入树中的值，将值插入二叉搜索树。 返回插入后二叉搜索树的根节点。 保证原始二叉搜索树中不存在新值。

注意，可能存在多种有效的插入方式，只要树在插入后仍保持为二叉搜索树即可。 你可以返回任意有效的结果。

例如, 

给定二叉搜索树:

```java
    4
   / \
  2   7
 / \
1   3

和 插入的值: 5
```


你可以返回这个二叉搜索树:

```java
     4
   /   \
  2     7
 / \   /
1   3 5
```
或者这个树也是有效的:

```java
     5
   /   \
  2     7
 / \   
1   3
     \
      4
```

**解法一**

```java
public TreeNode insertIntoBST(TreeNode root, int val) {
    if (root==null) {
        return new TreeNode(val);
    }
    if (root.val>val) {
        root.left=insertIntoBST(root.left,val);   
    }else if (root.val<val) {
        root.right=insertIntoBST(root.right,val);   
    }
    return root;
}
```
没啥好说的，看代码就懂了



## [662. 二叉树最大宽度](https://leetcode-cn.com/problems/maximum-width-of-binary-tree/)

给定一个二叉树，编写一个函数来获取这个树的最大宽度。树的宽度是所有层中的最大宽度。**这个二叉树与满二叉树（full binary tree）结构相同，但一些节点为空。**

每一层的宽度被定义为两个端点（该层最左和最右的非空节点，两端点间的null节点也计入长度）之间的长度。

**Example 1:**

```java
Input: 

           1
         /   \
        3     2
       / \     \  
      5   3     9 

Output: 4
Explanation: The maximum width existing in the third level with the length 4 (5,3,null,9).
```

**Example 2:**

```java
Input: 

          1
         /  
        3    
       / \       
      5   3     

Output: 2
Explanation: The maximum width existing in the third level with the length 2 (5,3).
```

**Example 3:**

```java
Input: 

          1
         / \
        3   2 
       /        
      5      

Output: 2
Explanation: The maximum width existing in the second level with the length 2 (3,2).
```

**Example 4:**

```java
Input: 

          1
         / \
        3   2
       /     \  
      5       9 
     /         \
    6           7
Output: 8
Explanation:The maximum width existing in the fourth level with the length 8 (6,null,null,null,null,null,null,7).
```

**Note:** Answer will in the range of 32-bit signed integer.

**解法一**

一开始居然没想到，哎😐还是菜啊

```java
public int widthOfBinaryTree(TreeNode root) {
    if (root==null) {
        return 0;
    }
    Queue<TreeNode> queue=new LinkedList<>();
    LinkedList<Integer> idxs=new LinkedList<>();
    int max=1;
    idxs.add(1);
    queue.add(root);
    while(!queue.isEmpty()){
        int size=queue.size();
        while(size>0){
            TreeNode top=queue.poll();
            int index=idxs.removeFirst();
            if (top.left!=null) {
                queue.add(top.left);
                idxs.add(index*2);
            }
            if (top.right!=null) {
                queue.add(top.right);
                idxs.add(index*2+1);
            }
            size--;
        }
        if (idxs.size()!=0) {
            max=Math.max(idxs.getLast()-idxs.getFirst()+1,max);    
        }
    }
    return max;
}
```
还是层次遍历的思路，不过需要额外添加一个索引列表，用来**记录每个节点对应在完全二叉树中的索引**，这个索引值完全可以根据上一层父节点的索引的到，我们初始化定义根节点的index为1，然后进行层次遍历记录每一层的每个节点的index就ok，当遍历完一层之后统计列表最左和最右两个节点之差，这个值就是当前层的宽度，最后求个最大值就ok了，很可惜，看了答案才知道

**解法二**

递归版本

```java
public int widthOfBinaryTree(TreeNode root) {
    if (root==null) {
        return 0;
    }
    dfs(root,0,0,new LinkedList<>());
    return max;
}

int max=1;

public void dfs(TreeNode node,int depth,int index,List<Integer> leftIdxs){
    if (node==null) {
        return;
    }
    if (depth>=leftIdxs.size()) {
        leftIdxs.add(index);
    }
    //记录当前节点和当前层最左节点的差
    max=Math.max(index-leftIdxs.get(depth)+1,max);
    dfs(node.left,depth+1,index*2,leftIdxs);
    dfs(node.right,depth+1,index*2+1,leftIdxs);
}
```
这个版本在空间复杂度可能会低一点，list中只存每个层最左的节点，当深度大于等于list的长度时候说明当前节点一定是新一层的最左节点，这个时候添加进去就ok，然后求每个节点和当前层最左的节点index差值就最后更新最大值就ok，这个解法还是没有那么自然，还是上面的BFS好理解一点

## [938. 二叉搜索树的范围和](https://leetcode-cn.com/problems/range-sum-of-bst/)

给定二叉搜索树的根结点 root，返回 L 和 R（含）之间的所有结点的值的和。

二叉搜索树保证具有唯一的值。

**示例 1：**

```java
输入：root = [10,5,15,3,7,null,18], L = 7, R = 15
输出：32
```


**示例 2：**

```java
输入：root = [10,5,15,3,7,13,18,1,null,6], L = 6, R = 10
输出：23
```

**提示：**

- 树中的结点数量最多为 10000 个。
- 最终的答案保证小于 2^31

**解法一**

还行，这题反应过来了，中序遍历

```java
private int sum=0;

public int rangeSumBST(TreeNode root, int L, int R) {
    preorder(root,L,R);
    return sum;
}

public void preorder(TreeNode root, int L, int R) {
    if (root==null) {
        return;
    }
    rangeSumBST(root.left,L,R);
    if (root.val>=L && root.val<=R) {
        sum+=root.val;
    }
    rangeSumBST(root.right,L,R);
}
```

## [617. 合并二叉树](https://leetcode-cn.com/problems/merge-two-binary-trees/)

给定两个二叉树，想象当你将它们中的一个覆盖到另一个上时，两个二叉树的一些节点便会重叠。

你需要将他们合并为一个新的二叉树。合并的规则是如果两个节点重叠，那么将他们的值相加作为节点合并后的新值，否则不为 NULL 的节点将直接作为新二叉树的节点。

**示例 1:**

```java
输入: 
	Tree 1                     Tree 2                  
          1                         2                             
         / \                       / \                            
        3   2                     1   3                        
       /                           \   \                      
      5                             4   7                  
输出: 
合并后的树:
	     3
	    / \
	   4   5
	  / \   \ 
	 5   4   7
```

**注意:** 合并必须从两个树的根节点开始。

**解法一**

```java
public TreeNode mergeTrees(TreeNode t1, TreeNode t2) {
    if (t2==null) {
        return  t1;
    }
    if (t1==null) {
        return t2;
    }
    t1.val+=t2.val;
    t1.left=mergeTrees(t1.left,t2.left);
    t1.right=mergeTrees(t1.right,t2.right);
    return t1;
}
```

## [543. 二叉树的直径](https://leetcode-cn.com/problems/diameter-of-binary-tree/)

给定一棵二叉树，你需要计算它的直径长度。一棵二叉树的直径长度是任意两个结点路径长度中的最大值。这条路径可能穿过根结点。

**示例 :**
给定二叉树

```java
      1
     / \
    2   3
   / \     
  4   5    
```
返回 3, 它的长度是路径 [4,2,1,3] 或者 [5,2,1,3]。

**注意：**两结点之间的路径长度是以它们之间边的数目表示

**解法一**

树的题目做多了，发现其实也就几种题型，都很熟悉，这题就和上面的 [二叉树的坡度]() ，[平衡二叉树]() 很类似，这题需要注意**直径不一定过根节点**

```java
int max=Integer.MIN_VALUE;

public int diameterOfBinaryTree(TreeNode root) {
    if (root==null) {
        return 0;
    }
    hight(root);
    return max;
}

public int hight(TreeNode node){
    if (node==null) {
        return 0;
    }
    int left=hight(node.left);
    int right=hight(node.right);
    max=Math.max(left+right,max);
    return Math.max(left,right)+1;
}
```

**解法二**

和之前一样，先写了个暴力的嵌套递归😂，代码确实简介，难道这就是暴力美学么，i了

```java
public int diameterOfBinaryTree(TreeNode root) {
    return root==null?0:Math.max(hight(root.left)+hight(root.right),Math.max(diameterOfBinaryTree(root.right),diameterOfBinaryTree(root.left)));
}

public int hight(TreeNode node){
    return node==null?0:Math.max(hight(node.left),hight(node.right))+1;
}
```

## [530. 二叉搜索树的最小绝对差](https://leetcode-cn.com/problems/minimum-absolute-difference-in-bst/)

给定一个所有节点为非负值的二叉搜索树，求树中任意两节点的差的绝对值的最小值。

**示例 :**

```java
输入:

   1
    \
     3
    /
   2

输出:
1

解释:
最小绝对差为1，其中 2 和 1 的差的绝对值为 1（或者 2 和 3）。
```

**注意:** 树中至少有2个节点。

**解法一**

很可惜，这题还WA了一次。。。

```java
public int getMinimumDifference(TreeNode root) {
    Stack<TreeNode> stack=new Stack<>();
    TreeNode cur=root;
    int diff=Integer.MAX_VALUE,last=-1;
    while(!stack.isEmpty() || cur!=null){
        while(cur!=null){
            stack.push(cur);
            cur=cur.left;
        }
        cur=stack.pop();
        if (last!=-1) {
            diff=Math.min(diff,Math.abs(last-cur.val));   
        }
        last=cur.val;
        cur=cur.right;
    }
    return diff;
}
```
**解法二**

递归的方式

```java
private int diff = Integer.MAX_VALUE;

private int last = -1;

public int getMinimumDifference(TreeNode root) {
    inorder(root);
    return diff;
}

public void inorder(TreeNode root){
    if (root==null) {
        return;
    }
    inorder(root.left);
    diff = last==-1?diff:Math.min(diff,Math.abs(last-root.val));
    last = root.val;
    inorder(root.right);
}
```

## [515. 在每个树行中找最大值](https://leetcode-cn.com/problems/find-largest-value-in-each-tree-row/)

您需要在二叉树的每一行中找到最大的值。

**示例：**

```java
输入: 
	  1
     / \
    3   2
   / \   \  
  5   3   9 
输出: [1, 3, 9]
```

**解法一**

娱乐题

```java
public List<Integer> largestValues(TreeNode root) {
    List<Integer> res=new ArrayList<>();
    dfs(root,res,0);
    return res;
}

public void dfs(TreeNode node,List<Integer> list,int level){
    if (node==null) {
        return;
    }
    if (level>=list.size()) {
        list.add(node.val);
    }
    if(node.val>list.get(level)){
        list.set(level,node.val);
    }
    dfs(node.left,list,level+1);
    dfs(node.right,list,level+1);
}
```

## [173. 二叉搜索树迭代器](https://leetcode-cn.com/problems/binary-search-tree-iterator/)

实现一个二叉搜索树迭代器。你将使用二叉搜索树的根节点初始化迭代器。

调用 `next()` 将返回二叉搜索树中的下一个最小的数。

 **示例：**

![leetcode](https://i.loli.net/2019/12/25/TXK397rI5hwjG4B.png)

```java
BSTIterator iterator = new BSTIterator(root);
iterator.next();    // 返回 3
iterator.next();    // 返回 7
iterator.hasNext(); // 返回 true
iterator.next();    // 返回 9
iterator.hasNext(); // 返回 true
iterator.next();    // 返回 15
iterator.hasNext(); // 返回 true
iterator.next();    // 返回 20
iterator.hasNext(); // 返回 false
```

**提示：**

- next() 和 hasNext() 操作的时间复杂度是 O(1)，并使用 **O(h)** 内存，其中 h 是树的高度。
- 你可以假设 next() 调用总是有效的，也就是说，当调用 next() 时，BST 中至少存在一个下一个最小的数。 

**解法一**

注意这题空间复杂度要求是`O(h)` ，并不是憨憨题

```java
Stack<TreeNode> stack=new Stack<>();

public BSTIterator(TreeNode root) {
    pushLeft(root);
}

public int next() {
    TreeNode node=stack.pop();
    if (node.right!=null) {
        pushLeft(node.right);
    }
    return node.val;
}

public boolean hasNext() {
    return !stack.isEmpty();
}

public void pushLeft(TreeNode node){
    while(node!=null){
        stack.add(node);
        node=node.left;
    }
}
```

我们可以用一个stack存储BST的左链，当取最小值就从stack中直接取，如果取出来的node还有右子树就将右子树的左链也添加进来，是不是有点熟悉？其实就是中序遍历的过程

## [95. 不同的二叉搜索树 II](https://leetcode-cn.com/problems/unique-binary-search-trees-ii/)

给定一个整数 n，生成所有由 1 ... n 为节点所组成的**二叉搜索树**。

**示例:**

```java
输入: 3
输出:
[
  [1,null,3,2],
  [3,2,null,1],
  [3,1,null,null,2],
  [2,1,3],
  [1,null,2,null,3]
]
解释:
以上的输出对应以下 5 种不同结构的二叉搜索树：

   1         3     3      2      1
    \       /     /      / \      \
     3     2     1      1   3      2
    /     /       \                 \
   2     1         2                 3
```

**解法一**

```java
public  List<TreeNode> generateTrees(int n) {
    if(n<=0){
        return new ArrayList<>();
    }
    return generateTrees(1,n);
}

public List<TreeNode> generateTrees(int start,int end) {
    List<TreeNode> res=new ArrayList<>();
    if (start>end) {
        //null也是一种情况，左右子树为空
        res.add(null);
        return res;
    }
    for (int i=start;i<=end;i++) {
        List<TreeNode> left=generateTrees(start,i-1);
        List<TreeNode> right=generateTrees(i+1,end);
        for (TreeNode l:left) {
            for (TreeNode r:right) {
                TreeNode currentNode=new TreeNode(i);
                currentNode.left=l;
                currentNode.right=r;
                res.add(currentNode);
            }
        }
    }
    return res;
}
```
很久之前做过的题，今天又拿出来看看，其实属于分治思路

## [538. 把二叉搜索树转换为累加树](https://leetcode-cn.com/problems/convert-bst-to-greater-tree/)

给定一个二叉搜索树（Binary Search Tree），把它转换成为累加树（Greater Tree)，使得每个节点的值是原来的节点值加上所有大于它的节点值之和。

**例如：**

```java
输入: 二叉搜索树:
              5
            /   \
           2     13

输出: 转换为累加树:
             18
            /   \
          20     13
```

**解法一**

这里我想先上一个**错误的解法**

```java
//以下代码纯属娱乐
public TreeNode convertBST(TreeNode root) {
    if (root==null) {
        return new TreeNode(0);//这里肯定是错的，null应该直接返回null
    }
    if (root.left==null && root.right==null) {
        return root;
    }
    root.val+= convertBST(root.right).val;
    convertBST(root.left).val+=root.val;
    return root;
}
```

忽略返回值的部分，乍一看好像是对的😂，其实问题大了，首先是左边的值算的不对，因为是DFS会从最左边开始算，都只加了他的父节点原始的值，而父节点的累加值还没有算出来，其次有些情况是算不出来的比如左子树的某一个右节点你就算不出来

**解法二**

其实一开始就知道可以直接中序遍历做，只是想玩一些其他的方法，可惜没搞出来😂

```java
public TreeNode convertBST(TreeNode root) {
    dfs(root);
    return root;
}

private int sum=0;

public void dfs(TreeNode root){
    if (root==null) {
        return;
    }
    dfs(root.right);
    sum+=root.val;
    root.val=sum;
    dfs(root.left);
}
```
这里需要注意的就是要翻过来遍历，从大到小，因为它求的是比它大的节点的值

## [508. 出现次数最多的子树元素和](https://leetcode-cn.com/problems/most-frequent-subtree-sum/)

给出二叉树的根，找出出现次数最多的子树元素和。一个结点的子树元素和定义为以该结点为根的二叉树上所有结点的元素之和（包括结点本身）。然后求出出现次数最多的子树元素和。如果有多个元素出现的次数相同，返回所有出现次数最多的元素（不限顺序）。

**示例 1**

```java
输入:

  5
 /  \
2   -3
返回 [2, -3, 4]，所有的值均只出现一次，以任意顺序返回所有值。
```

**示例 2**

```java
输入:

  5
 /  \
2   -5
返回 [2]，只有 2 出现两次，-5 只出现 1 次。
```

**解法一**

左子树和+右子树和，HashMap记录出现的次数，记录最大值然后取出出现次数最多的

```java
private Map<Integer,Integer> map = new HashMap<>();

private int maxCount=0;

public int[] findFrequentTreeSum(TreeNode root) {
    if (root ==null) {
        return new int[]{};
    }
    dfs(root);
    List<Integer> res=new ArrayList<>();
    map.keySet().stream().filter(val->map.get(val)==maxCount).forEach(res::add);
    return res.stream().mapToInt(Integer::valueOf).toArray(); //list转数组的又一个小技巧，单纯的toArray只能转换成Integer[],还需要转
}

public int dfs(TreeNode root){
    if (root==null) {
        return 0;
    }
    int value=root.val+dfs(root.right)+dfs(root.left);
    map.put(value,map.getOrDefault(value,0)+1);
    maxCount=Math.max(maxCount,map.get(value));
    return value;
}
```

写法是基于Lambda的，函数式写起来真的舒服

## [606. 根据二叉树创建字符串](https://leetcode-cn.com/problems/construct-string-from-binary-tree/)

你需要采用前序遍历的方式，将一个二叉树转换成一个由括号和整数组成的字符串。

空节点则用一对空括号 "()" 表示。而且你需要省略所有不影响字符串与原始二叉树之间的一对一映射关系的空括号对。

**示例 1:**

```java
输入: 二叉树: [1,2,3,4]
       1
     /   \
    2     3
   /    
  4     

输出: "1(2(4))(3)"

解释: 原本将是“1(2(4)())(3())”，
在你省略所有不必要的空括号对之后，
它将是“1(2(4))(3)”。
```

**示例 2:**

```java
输入: 二叉树: [1,2,3,null,4]
       1
     /   \
    2     3
     \  
      4 

输出: "1(2()(4))(3)"

解释: 和第一个示例相似，
除了我们不能省略第一个对括号来中断输入和输出之间的一对一映射关系。
```

**解法一**

```java
public String tree2str(TreeNode t) {
    StringBuilder s=new StringBuilder();
    dfs(t,s);
    return s.toString();
}

public void dfs(TreeNode node,StringBuilder s){
    if (node==null) {
        return;
    }
    s.append(node.val);
    if(node.left==null && node.right==null){//没有子节点
        return;
    }
    s.append("(");
    dfs(node.left,s);
    s.append(")");
    if (node.right==null) { //没有右节点
        return;
    }
    s.append("(");
    dfs(node.right,s);
    s.append(")");
}
```

没啥好说的，搞清楚题目意思然后注意递归的几个出口就行了