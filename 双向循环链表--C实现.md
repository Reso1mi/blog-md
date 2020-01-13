---
title: 
 双向循环链表
tags: 
  [数据结构,算法]
date: 2018/8/18
categories:
		[算法]
---
## C实现的双向循环链表
很久没有用C了，都忘了，昨天下午又复习了一下然后实现了这个双向循环链表，后面每种数据结构都会在这里实现记录下来。
开发环境   : sublime+MinGW 
```java
#include <stdio.h>
#include <stdlib.h>
//定义结构体
typedef struct Node
{
	int data;
	struct Node *perv;
	struct Node *next;
}Node;

int length;

//初始化节点
Node* createNode(){
	Node * node;
	//开辟空间
	node =(Node*)malloc(sizeof(Node));
	if(node==NULL){
		printf("动态开辟空间失败");
	}
	scanf("%d",&(node->data));
	node->perv=NULL;
	node->next=NULL;
	return node;
}

//初始化链表 
Node* createList(int n)
{
    Node *tail,*p,*head;
    //初始化头结点 (这个节点只是个标志，标志链表的头并不存储数据,只是为了操作的统一性)
    head=(Node*)malloc(sizeof(Node));
    int i;
    if(n >= 1)   //结点的个数 >= 1 的时候
    {
        p = createNode();
        head->next = p;
        p->perv=head;
        tail = p;
    }
    for(i = 2;i <= n;i++)    //生成第一个结点以后的结点，并建立双向链表的关系 
    {
        p = createNode();
        tail->next = p;
        p->perv = tail;
        //尾指针后移
        tail = p;
    }
    //连接头尾
    head->perv=tail;
    tail->next=head;
    //链表的长度
    length = n;
    if(n >= 1)
        return (head);
    else
        return 0;    
} 


//在头尾插入节点   （实际上这两个方法都可以通过下面的insAnywhere完成，主要为了效率,如果是尾结点插入getEle()时间复杂度过高）
//insAnywhere(head,0);
void insHead(Node* head){
	Node *p=createNode();
	Node *q;
	//保存第一个节点
	q=head->next;
	//连接头结点
	head->next=p;
	p->perv=head;
	//连接之前的第一个节点
	p->next=q;
	q->perv=p;
	++length;
}

//insAnywhere(head,length);
void insTail(Node* head){
	Node *p=createNode();
	//先保存下之前的尾指针
	Node *tail=head->perv;
	//连接头尾
	head->perv=p;
	p->next=head;
	//连接之前的尾指针
	tail->next=p;
	p->perv=tail;
	++length;
}

//取得某一位置的节点  时间复杂度为 O(n)
Node * getEle(Node* head,int n){
	//将第一个节点赋值给p
	Node *p=head;
	for (int i = 0; i < n; ++i)
	{
		p=p->next;
	}
	return p;
}

//获取链表的长度
int getLength(){
	return length;
}

//任意位置插入，因为设置了头节点所以插入的操作具有一致性
void insAnywhere(Node *head,int n){
	Node *newNode=createNode();
	//取得对应位置的值
	Node *currentNode=getEle(head,n);
	//保存当前位置的下一个
	Node *nextNode;
	//保存下一个节点
	nextNode=currentNode->next;
	//连接当前结点
	currentNode->next=newNode;
	newNode->perv=currentNode;
	//连接之前的下一个节点
	newNode->next=nextNode;
	nextNode->perv=newNode;
	++length;
}

//任意位置的删除
void delNode(Node *head,int n){
	Node *delNode=getEle(head,n);
	/*printf("%d\n", delNode->data);
	printf("--------------\n");*/
	//先保存当前节点的后一节点
	Node *nextNode=delNode->next;
	//将后一个节点接在当前节点的前一个的后面
	delNode->perv->next=nextNode;
	nextNode->perv=delNode->perv;
	//free这个节点
	free(delNode);
}

//遍历链表
void printlnAll(Node *head){
	Node *p=head->next;
   do{
   	  printf("%d\n", p->data);
   	  p=p->next;
   }while(p!=head);
	// while(p->next!=head){
	// 	printf("%d\n", p->data);
	// 	//指针后移
	// 	p=p->next;
	// }
}


int main(int argc, char const *argv[])
{

	Node* createList(int n);
	Node* createNode();

	Node *head=createList(3);
	printf("遍历链表\n");
	printlnAll(head);
	
	insHead(head);
	//insAnywhere(head,0);
	printf("在头插入节点后\n");
	printlnAll(head);

	insTail(head);    //下面的也可以但是效率比较低
	//insAnywhere(head,getLength());
	printf("在尾插入节点后\n");
	printlnAll(head);

	//在第一个元素后面插入元素
	printf("---------在第一个元素后面插入元素-----\n");
	insAnywhere(head,1);
	printlnAll(head);

	//删除最后面的节点
	printf("---------删除最后面的节点-------\n");
	delNode(head,6);
	printlnAll(head);

	//删除第一个后面的节点
	printf("-------删除第一个后面的节点---------\n");
	delNode(head,2);
	//printf("%d\n", head->next->data);
	printlnAll(head);
	return 0;
}
```
- 线性表
![image](http://p0.cdn.img9.top/ipfs/QmUS62kkfTx4trGRmMukDCFuJNxrKDR2P2DdoERRqf1pcr?0.png)
这里面的链式存储结构里面的 *静态链表* 挺有意思的，不用指针实现链式结构。
线性表的这两种结构实际上是后面其他数据结构的基础，顺序储存结构和链式储存结构也各有优劣。
![image](http://p1.cdn.img9.top/ipfs/QmUWbnXLv86uwuCrWkp2ft6fax7TgZ1kryaCPKrWTGidsy?1.PNG)

注：代码中的String是我自定义的。

```java
typedef struct{
	//长度
	int length;
	//内容
	char *str;
	//最大值
	int maxLength;
}String;

void init(String *s,int max,char * string){
	int i;
	s->maxLength=max;
	s->length=strlen(string);
	//开辟空间
	s->str=(char*)malloc(sizeof(char)*max);
	//赋值
	for(i=0;i<s->length;i++){
		s->str[i]=string[i];
	}
}

```

