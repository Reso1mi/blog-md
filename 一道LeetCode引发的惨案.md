---
title: 
 一道LeetCode引发的惨案
tags: 
  [数据结构,算法,搜索]
date: 2018/10/31
categories:
	[算法]
---
## 一道LeetCode搜索题引发的惨案
 ### 1.先上 [题目](https://leetcode-cn.com/problems/word-ladder/) 
给定两个单词（_beginWord _和 _endWord_）和一个字典，找到从 _beginWord_ 到 _endWord_ 的最短转换序列的长度。转换需遵循如下规则：

1.  每次转换只能改变一个字母。
2.  转换过程中的中间单词必须是字典中的单词。
 [原题链接](https://leetcode-cn.com/problems/word-ladder/) 
![oss](https://imlgwpicture.oss-cn-qingdao.aliyuncs.com/blogImage/3V6EQ77R_%28%292SCR%248R%5B%245F7.png)其实这题明显是BFS(广搜) 题目类型也说了是广搜，但是我不信邪写了DFS(毕竟代码比较好写)，然后惨案就发生了。

```java
   // 标记数组
	// 默认都是0
	private static int[] mark;

	public int ladderLength(String beginWord, String endWord, List<String> wordList) {
		// 不存在
		mark = new int[wordList.size() + 1];
		if (!wordList.contains(endWord)) {
			return 0;
		}
		// dfs开始
		dfs(beginWord, endWord, wordList);
		// 无法转换
		if (min == Integer.MAX_VALUE) {
			return 0;
		}
		return min + 1;
	}

	int min = Integer.MAX_VALUE;

	// dfs
	private void dfs(String beginWord, String endWord, List<String> wordList) {
		//当相等的时候
		if (beginWord.equals(endWord)) {
			int step = 0;
			for (int i = 0; i < mark.length; i++) {
				if (mark[i] == 1) {
					step++;
				}
			}
			//更新最小值
			min = step < min ? step : min;
			return;
		}
		for (int i = 0; i < wordList.size(); i++) {
			if (mark[i] == 0 && cmp(beginWord, wordList.get(i))) {
				mark[i] = 1;
				dfs(wordList.get(i), endWord, wordList);
				mark[i] = 0;
			}
		}
		return;
	}

	// 写一个函数判段是否只变化了一个字母
	private boolean cmp(String s1, String s2) {
		int count = 0;
		for (int i = 0; i < s1.length(); i++) {
			if (s1.charAt(i) != s2.charAt(i)) {
				count++;
			}
		}
		return count == 1;
	}
```
结果直接TLE了，后来想了想DFS每次都会**尝试**所有情况，而给的例子后面的数据量也比较大，而且这题是要统计最短路径，要全部递归完才能确定最小值，我把数据拿来自己测试跑了好长时间都没跑出来，而BFS没有递归只是会耗费的空间会比较大。从这里也可以总结出来DFS跟适合判断是否存在是否可达之类的问题，BFS更适合做找最短最小之类的问题。上BFS代码

```java
// 标记数组
	// 默认都是0
	private static int[] mark;
	// 模拟队列
	class Que {
		String word;
		int step;
	}
	//BFS
	public int ladderLengthBFS(String beginWord, String endWord, List<String> wordList){
        // 不存在
		mark = new int[wordList.size() + 1];
		if (!wordList.contains(endWord)) {
			return 0;
		}
		// BFS
		int head = 0, tail = 0;
		// 初始化队列
		Que[] que = new Que[wordList.size() + 1];
		// 循环促使话述祖
		for (int i = 0; i < que.length; i++) {
			que[i] = new Que();
		}
		que[tail].word = beginWord;
		que[tail].word = beginWord;
		que[tail].step = 1;
		tail++;
		int flag=0;
		while (head < tail) {
			// 遍历字典
			for (int i = 0; i < wordList.size(); i++) {
				if (mark[i] == 0 && cmp(wordList.get(i), que[head].word)) {
					que[tail].word = wordList.get(i);
					//这里是从head开始的，所以应该是head的步数+1
					que[tail].step=que[head].step+1;
					// 标记为已经走过
					mark[i] = 1;
				   if (que[tail].word.equals(endWord)) {
					//到这里说明已经到终点了，而且是最短的，之后的最多就是相等
					//跳出循环
                       flag=1;
                       break;
			    	}
                    tail++;
				}
				
			}
            if(flag==1){
                break;
            }
			// 每次检查完一个单词就将其出队列
			head++;
		}
		return que[tail].step;
    }

	// 写一个函数判段没吃是否只变化了一个字母
	private boolean cmp(String s1, String s2) {
		int count = 0;
		for (int i = 0; i < s1.length(); i++) {
			if (s1.charAt(i) != s2.charAt(i)) {
				count++;
			}
		}
		return count == 1;
	}
```

PS：刚刚修改了下代码，我也佩服自己BFS都还没搞清楚就直接上了代码，然后结果居然还是对的，之前在if判断队尾元素是不是和endWord相等后还维护了一个最小值min，后来想了想不对，最后一个和endWord相等的元素已经进栈了，已经是最短的了，后面的即使可以转换到也最多只能和当前的相等了。 如果只是为了统计最小值就可以直接break了。那如果不仅仅要统计最小值还要记录路径要怎么搞？
### 2. [加强版](https://leetcode-cn.com/problems/word-ladder-ii/) 单词接龙 2
是一道困难等级的题，需要在上面的基础上找出所有的最短的路径。

```java
// 内部类
	class Que {
		String word;
		int step;
		Que prev;
		public String toString(){
			return (word + ":" + step);
		}		 
	}

	//返回值
	private  List<List<String>> res=new ArrayList<>();

	
	//BFS
	public List<List<String>> ladderLengthBFS(String beginWord, String endWord, List<String> wordList){
        //返回值
		//List<List<String>> res=new ArrayList<>();
		
        // 不存在
		mark = new int[wordList.size() + 1];
		if (!wordList.contains(endWord)) {
			return res;
		}
		// BFS
		int head = 0, tail = 0;
		// 初始化队列
		Que[] que = new Que[wordList.size() + 1];
		// 循环促使话述祖
		for (int i = 0; i < que.length; i++) {
			que[i] = new Que();
		}
		//先把第一个单词放进去
		que[tail].word = beginWord;
		que[tail].word = beginWord;
		que[tail].step = 1;
		tail++;
		List<Que> quelist=new ArrayList<>();
		while (head < tail) {
			// 遍历字典
			for (int i = 0; i < wordList.size(); i++) {
				if (mark[i] == 0 && cmp(wordList.get(i), que[head].word)) {
					que[tail].word = wordList.get(i);
					//这里是从head开始的，所以应该是head的步数+1
					que[tail].step=que[head].step+1;
					//que[head].next=que[tail];
					// 标记为已经走过
					mark[i] = 1;
				   if (que[tail].word.equals(endWord)) {	
				   		//记录最小值
				   		min=que[tail].step;
				   		//到这里队列后面就不用再插入元素了

				   		//2.把之前走过的路在下一个head

				   		//将队列变成list
				   		for(int j=0;j<=tail;j++){
				   			quelist.add(que[j]);
				   		}
				   		markDfs= new int[wordList.size() + 1]; 
				   		// 1. 用DFS试一下
				   		dfsBfs(que[0],endWord,quelist);
			    	}
                    tail++;
				}
			}
			// 每次检查完一个单词就将其出队列
			head++;
		}
		return res;
    }


    private void dfsBfs(Que beginWord,String endWord,List<Que> ques){
    	int step = 0;
		for (int i = 0; i < markDfs.length; i++) {
				if (markDfs[i] == 1) {
					step++;
				}
		}
		if(step>=min) return;
		if(step+1==min&&endWord.equals(beginWord.word)){
			List<String> list=new ArrayList<>();
			Stack<String> stack=new Stack<>();
			System.out.println(beginWord.word+":"+step);
			Que temp=beginWord;
				//找到一条
				for(int i=0;i<min;i++){
					stack.push(temp.word);
					System.out.print(temp.word+"<--");
					temp=temp.prev;
				}
				System.out.println();
				for(int i=0;i<min;i++){
					list.add(stack.pop());
				}
				res.add(list);
				System.out.println(res);
			return;
		}

		
		for (int i = 0; i < ques.size(); i++) {
			if (markDfs[i] == 0 && cmp(beginWord.word,ques.get(i).word)){
				markDfs[i] = 1;
				//连接两个节点
				//beginWord.next=ques.get(i);
				ques.get(i).prev=beginWord;
				dfsBfs(ques.get(i), endWord, ques);
				markDfs[i] = 0;
			}
		}
		return;
    }

	// private int step = 0;
	int min = Integer.MAX_VALUE;

	// 写一个函数判段没吃是否只变化了一个字母
	private boolean cmp(String s1, String s2) {
		int count = 0;
		for (int i = 0; i < s1.length(); i++) {
			if (s1.charAt(i) != s2.charAt(i)) {
				count++;
			}
		}
		return count == 1;
	}
```
整体上在Que上增加了一个prev的指针，遍历路径，一开始是用的BFS不过我想的太简单了，我只是把最后一个节点出队列然后再BFS，后来发现不行(居然还跑过了24个测试案例)，实际上这题我还是没有做出来，但是上面的方法应该是没问题的就是会TLE😭，大概思路就是先BFS缩短DFS需要遍历的字典然后控制每次递归的身体不能超过BFS的到的最短路径
![img9](https://imlgwpicture.oss-cn-qingdao.aliyuncs.com/blogImage/blog.PNG)一开始只能跑几个数据的，优化下能跑几十个的，但是还是太慢了，毕竟是一道难题等以后学了相关的东西再来试试看能不能做出来吧.

---
算法，学着挺有意思，就是头有点凉。