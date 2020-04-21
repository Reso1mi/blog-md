---
title: 
 KMP算法及优化
tags: 
  [数据结构,算法]
date: 2018/9/6
categories:
		[算法]
---
## KMP算法及优化
> 后面有时间再来重写一下，kmp还是挺重要的

KMP算法是一种改进的[字符串匹配](https://baike.so.com/doc/9018958-9348545.html)算法，由D.E.Knuth，J.H.Morris和V.R.Pratt同时发现，因此人们称它为克努特--莫里斯--普拉特操作(简称KMP算法)。KMP算法的关键是利用匹配失败后的信息，尽量减少模式串与主串的匹配次数以达到快速匹配的目的。具体实现就是实现一个next()函数，函数本身包含了模式串的局部匹配信息。时间复杂度O(m+n)。
 开发环境   : sublime+MinGW 

- 暴力匹配法

```c
//暴力匹配   
int BruteForce(String S, String T,int begin){ 
	int i=begin,j=0;
	while(i<S.length&&j<T.length){
		if(S.str[i]==T.str[j]){
			j++;
			i++;
		}else{
			//母串的当前位置+1向后移动
			i=i-j+1;
			//子串从0开始
			j=0;
		}
	}
	if (i<s(S.length-T.length))
		return i-j-begin;
	return -1;  //没找到
}
```
暴力匹配比较简单粗暴，就是一个个的比对如果不对母串就回溯直到配成功。
这个算法无疑时间复杂度较高，最糟糕情况为 O(m*n);而这个算法的缺陷就在于每次母串的不必要的回溯，于是KMP算法出现了
- KMP算法

```c
void getNext(String S,int next[]){
	//自己和自己匹配其实和kmp是一个道理
	int i=1,j=0;
	//和标准的KMP不台一样,标准的KMP是从1开始的其实原理也一样，这样更习惯
	next[0]=-1;
	next[1]=0;
	//循环 n-2 次
	while(i<S.length-1){
		if(S.str[i]==S.str[j]){
			++j;
			++i;
			next[i]=j;
		}else if(j==0){
			//如果和前缀第一个字符就不等，i后移下一个位置的next为0
			++i;
			next[i]=0;
		}else
		    //j回退到之前的位置
			j=next[j];
	}
} 

//返回第一次出现的位置
int KMP(String S,String T,int begin){
	int i=begin,j=0;
	int next[100];
	getNext(T,next);
	while(i<S.length&&j<T.length){
		if(S.str[i]==T.str[j]){
			++i;
			++j;
		}else if(j==0){
			i++;
		}
		else
		{
		 j=next[j];
		}
	}
	if (i<s(S.length-T.length))
		return i-j-begin;
	return -1;   //没找到
}
```
这个算法的原理其实很好理解 ,母串不回溯，如果不相等字串就会跳到对应的位置继续比对
 # a b a b c
 # a b c
比如上面这种，c和a不相等 ，如果是暴力匹配那母串就会回溯到第二个字符b的位置继续比对，但是这并没有意义，因为子串的字符都不相等，正确的做法肯定是直接将字串右滑，或者说将子串回溯，回溯到a的位置，然后计较母串的第三个字符和子串的第一个字符...... 分析一下可以看出来这个算法最坏时间复杂度为O(n+m)
那么问题来了回溯的位置如何确定？这也是这个算法的关键之处。要得到一个对应每个位置的next数组储存当失配时回溯的位置，这个数组的确定只和子串本身的结构有关其代表的时最长的公共前后缀的长度，关于next数组的求法就不说了，人眼基本上一眼就能看出来。
- next数组编程实现

```c
void getNext(String S,int next[]){
	//自己和自己匹配其实和kmp是一个道理
	int i=1,j=0;
	//和标准的KMP不台一样,标准的KMP是从1开始的其实原理也一样，这样更习惯
	next[0]=-1;
	next[1]=0;
	//循环 n-2 次
	while(i<S.length-1){
		if(S.str[i]==S.str[j]){
			++j;
			++i;
			next[i]=j;
		}else if(j==0){
			//如果和前缀第一个字符就不等，i后移下一个位置的next为0
			++i;
			next[i]=0;
		}else
		    //j回退到之前的位置
			j=next[j];
	}
} 
```
一开始看到这个我是比较懵的怎么这么短？仔细看了下这几行代码前面的其实都还好理解,相当于自己和自己匹配
# ！！！关键是后面的不相等的情况
这里我也纠结了小半天随后在网上看到了一个图讲解这个的瞬间就开朗了这里就直接搬过来了（对[原文](https://blog.csdn.net/qq_30974369/article/details/74276186)加了一点自己的直观理解并另外加上了参考《大话数据结构》里面的对于KMP算法的优化）
#  a b  a  c  f  g a  b a b h
![image](https://p1.cdn.img9.top/ipfs/QmQLpe9fWukFT9is6XnAmmAyBdTCVYSeV6oj7mn5yGVsy8?1.png)
# **a b  a**  c  f  g **a  b a** *b*  h
红色的是当前匹配上的最长的前后缀，蓝色为当前匹配位置，也就是*i* 的位置 与上面对应的就是b 的位置了
![image2](https://p3.cdn.img9.top/ipfs/QmaFp1PaEY5Ednc4wUdLatsuJ7e4fibKv7jXwroGGb8rXN?3.png)
# **a b  a**  *c*  f  g **a  b  a** *b*  h
绿色为当前匹配到的最长前缀的后一位也就是 *j*  的位置 对应 c 的位置
显然c b不相等  如果不相等那就不能继续往后找了 那像 aba 这么长的公共前后缀就用不了了，也就是你的next数组会变小，前后缀相似度会减小。
![image3](https://p1.cdn.img9.top/ipfs/QmURa6y33pwvpWjoBZ6eSiyZzraBd9QPWSBF13gq2xdPnT?1.png)
![image4](https://p2.cdn.img9.top/ipfs/QmVfW9VQ2JYhNigxskotFVRtgAYcEhksBBEKzSztKn3p9b?2.png)
# **a**  *b*  **a**  c  f  g  **a**  b  **a** b  h
如图四块灰色区域完全相等，关键步骤 j=next[j];之前的j在绿色区域 现在的 j 回溯到了紫色区域 也就是对应b的位置
![image5](https://p1.cdn.img9.top/ipfs/QmYo2cscPKdCL9tGGn2frfrn6p4WuGXuma6W6yQSEw4bdx?1.png)
相信到这里应该就看明白了,四块区域相等，如果蓝色部分和紫色相等是不是就又有了公共的前后缀了呢？那如果不同就会继续递推。
- KMP优化
  什么？这么吊的算法还可以优化？的确，KMP还可以优化，这里直接拿《大话数据结构》里面的例子来说明 (ps : 这里大话数据结构和KMP原始的是一样的，也就是next是从1开始的，我是从0开始的)     
![image6](https://p0.cdn.img9.top/ipfs/QmSum26wHswwEgRmR79oM75o8afbAMCy2ZMQcremrXpfka?0.png)    
  - next优化

```c
void getNexval(String S,int next[]){
	//自己和自己匹配其实和kmp是一个道理
	int i=1,j=0;
	//和标准的KMP不台一样,标准的KMP是从1开始的其实原理也一样，这样更习惯
	next[0]=-1;
	next[1]=0;
	//循环n-2次
	while(i<S.length-1){
		if(S.str[i]==S.str[j]){
			++j;
			++i;
			//下一对对应的位置（也就是正在求next值的位置）
			if(S.str[j]==S.str[i]){
				//这一步实际上是跳过了重复的部分 如果和这个位置的next值的字符相同就可以将这个位置的next字符设置为next位置字符的next值
				next[i]=next[j];
			}else
			next[i]=j;
		}else if(j==0){
			//如果和前缀第一个字符就不等，i后移下一个位置的next为0
			++i;
			next[i]=0;
		}else
		    //j回退到之前的位置
			j=next[j];
	}
} 
```
其实优化的理由就是在进行和子串匹配的时候如果失配，在子串回溯时回溯到的那个值和当前的值相同那么我们可以直接回溯到 那个回溯值的回溯值，可能有点绕但是仔细想想就明白了。
 # **a b a b a a a b a**
这个字符串的next应该是 next=[-1,0,0,1,2,3,1,1,2] 
nextval=[-1,0,0,0,0,3,1,0,0] （这里有一个小问题，那就是前两位是不用考虑的，永远是-1 0，所以如果按照上面的说法可能会认为第三个是-1，其实不是的具体的可以看代码，如果时官方的那种从1开始就没有这种顾虑）。
拿这个实际的来说 ,  **a b a b a a a b a**当比对到该子串最后一个a时 不相等 ，按照之前的方法回溯，回溯到index=2的a位置，发现这货也是a那肯定也不相等，然后又回溯回溯到第一个a，这中间不就多回溯了一次么？为什么不一次到位呢？所以我们直接判断将要回溯的值和当前值是不是相等，相等就把将要回溯的值的回溯值赋给当前值。



