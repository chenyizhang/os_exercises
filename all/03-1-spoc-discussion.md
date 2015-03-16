# lec5 SPOC思考题


NOTICE
- 有"w3l1"标记的题是助教要提交到学堂在线上的。
- 有"w3l1"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。


## 个人思考题
---

请简要分析最优匹配，最差匹配，最先匹配，buddy systemm分配算法的优势和劣势，并尝试提出一种更有效的连续内存分配算法 (w3l1)
```
  + 采分点：说明四种算法的优点和缺点
  - 答案没有涉及如下3点；（0分）
  - 正确描述了二种分配算法的优势和劣势（1分）
  - 正确描述了四种分配算法的优势和劣势（2分）
  - 除上述两点外，进一步描述了一种更有效的分配算法（3分）
 ```
- [x]  

>  

## 小组思考题

请参考ucore lab2代码，采用`struct pmm_manager` 根据你的`学号 mod 4`的结果值，选择四种（0:最优匹配，1:最差匹配，2:最先匹配，3:buddy systemm）分配算法中的一种或多种，在应用程序层面(可以 用python,ruby,C++，C，LISP等高语言)来实现，给出你的设思路，并给出测试用例。 (spoc)

best_fit :
#include <iostream>
#include <stdlib.h>
using namespace std;


#define MAX_length 640 //最大内存空间为640KB
#define Free 0 //空闲状态
#define Busy 1 //已用状态
#define OK 1
#define ERROR 0

struct freearea
{
	long size;
	long address;
	int state;
};

typedef struct DulNode
{
	freearea data;
	struct DulNode *prior;
	struct DulNode *next;
}DuLNode,*DuLinkList;

DuLinkList block_first;
DuLinkList block_last;

int best_fit(int request);
int initblock();
int alloc();
int free();


int initblock()
{
	block_first=(DuLinkList)malloc(sizeof(DuLNode));
	block_last=(DuLinkList)malloc(sizeof(DuLNode));
	block_first->prior=NULL;
	block_first->next=block_last;
	block_last->prior=block_first;
	block_last->next=NULL;
	block_last->data.address=0;
	block_last->data.size=MAX_length;
	block_last->data.state=Free;
	return OK;
}


int alloc()
{
	int request = 0;
	cout<<"请输入需要分配的主存大小(单位:KB)："; 
	cin>>request;
	if(request<0 ||request==0) 
	{
		cout<<"分配大小不合适，请重试！"<<endl;
		return ERROR;
	}

	if(best_fit(request)==OK) cout<<"分配成功！"<<endl;
	else cout<<"内存不足，分配失败！"<<endl;
	return OK;
}


int best_fit(int request)
{
	int ch; //记录最小剩余空间
	DuLinkList temp=(DuLinkList)malloc(sizeof(DuLNode)); 
	temp->data.size=request;
	temp->data.state=Busy;
	DuLNode *p=block_first->next;
	DuLNode *q=NULL; //记录最佳插入位置

	while(p) //初始化最小空间和最佳位置
	{
		if(p->data.state==Free && (p->data.size>=request) )
		{
			if(q==NULL)
			{
				q=p;
				ch=p->data.size-request;
			}
			else if(q->data.size > p->data.size)
			{
				q=p;
				ch=p->data.size-request;
			}
		}
		p=p->next;
	}

	if(q==NULL) return ERROR;//没有找到空闲块

	else if(q->data.size==request)
	{
		q->data.state=Busy;
		return OK;
	}
	else
	{
		temp->prior=q->prior;
		temp->next=q;
		temp->data.address=q->data.address;
		q->prior->next=temp;
		q->prior=temp;
		q->data.address+=request;
		q->data.size=ch;
		return OK;
	}
	return OK;
}

int free(int flag)
{
	DuLNode *p=block_first;
	for(int i= 0; i <= flag; i++)
		if(p!=NULL)
			p=p->next;
		else
			return ERROR;

	p->data.state=Free;
	if(p->prior!=block_first && p->prior->data.state==Free)//与前面的空闲块相连
	{
		p->prior->data.size+=p->data.size;
		p->prior->next=p->next;
		p->next->prior=p->prior;
		p=p->prior;
	}
	if(p->next!=block_last && p->next->data.state==Free)//与后面的空闲块相连
	{
		p->data.size+=p->next->data.size;
		p->next->next->prior=p;
		p->next=p->next->next; 
	}
	if(p->next==block_last && p->next->data.state==Free)//与最后的空闲块相连
	{
		p->data.size+=p->next->data.size;
		p->next=NULL; 
	}

	return OK;
}

void show()
{
	int flag = 0;
	cout<<"\n主存分配情况:\n";
	cout<<"++++++++++++++++++++++++++++++++++++++++++++++\n\n";
	DuLNode *p=block_first->next;
	cout<<"分区号\t起始地址\t分区大小\t状态\n\n";
	while(p)
	{
		cout<<"  "<<flag++<<"\t";
		cout<<"  "<<p->data.address<<"\t\t";
		cout<<" "<<p->data.size<<"KB\t\t";
		if(p->data.state==Free) cout<<"空闲\n\n";
		else cout<<"已分配\n\n";
		p=p->next;
	}
	cout<<"++++++++++++++++++++++++++++++++++++++++++++++\n\n";
}

int main()
{
	initblock();
	
	while(1)
	{
		int choice;
		show();
		cout<<"请输入您的操作：";
		cout<<"\n1: 分配内存\n2: 回收内存\n0: 退出\n";

		cin>>choice;
		if(choice==1) 
			alloc(); // 分配内存
		else if(choice==2)  // 内存回收
		{
			int flag;
			cout<<"请输入您要释放的分区号：";
			cin>>flag;
			free(flag);
		}
		else if(choice==0) break; //退出
		else //输入操作有误
		{
			cout<<"输入有误，请重试！"<<endl;
			continue;
		}

	}
	return 0;
}

```
如何表示空闲块？ 如何表示空闲块列表？ 
[(start0, size0),(start1,size1)...]
在一次malloc后，如果根据某种顺序查找符合malloc要求的空闲块？如何把一个空闲块改变成另外一个空闲块，或消除这个空闲块？如何更新空闲块列表？
在一次free后，如何把已使用块转变成空闲块，并按照某种顺序（起始地址，块大小）插入到空闲块列表中？考虑需要合并相邻空闲块，形成更大的空闲块？
如果考虑地址对齐（比如按照4字节对齐），应该如何设计？
如果考虑空闲/使用块列表组织中有部分元数据，比如表示链接信息，如何给malloc返回有效可用的空闲块地址而不破坏
元数据信息？
伙伴分配器的一个极简实现
http://coolshell.cn/tag/buddy
```

--- 

## 扩展思考题

阅读[slab分配算法](http://en.wikipedia.org/wiki/Slab_allocation)，尝试在应用程序中实现slab分配算法，给出设计方案和测试用例。

## “连续内存分配”与视频相关的课堂练习

### 5.1 计算机体系结构和内存层次
MMU的工作机理？

- [x]  

>  http://en.wikipedia.org/wiki/Memory_management_unit

L1和L2高速缓存有什么区别？

- [x]  

>  http://superuser.com/questions/196143/where-exactly-l1-l2-and-l3-caches-located-in-computer
>  Where exactly L1, L2 and L3 Caches located in computer?

>  http://en.wikipedia.org/wiki/CPU_cache
>  CPU cache

### 5.2 地址空间和地址生成
编译、链接和加载的过程了解？

- [x]  

>  

动态链接如何使用？

- [x]  

>  


### 5.3 连续内存分配
什么是内碎片、外碎片？

- [x]  

>  

为什么最先匹配会越用越慢？

- [x]  

>  

为什么最差匹配会的外碎片少？

- [x]  

>  

在几种算法中分区释放后的合并处理如何做？

- [x]  

>  

### 5.4 碎片整理
一个处于等待状态的进程被对换到外存（对换等待状态）后，等待事件出现了。操作系统需要如何响应？

- [x]  

>  

### 5.5 伙伴系统
伙伴系统的空闲块如何组织？

- [x]  

>  

伙伴系统的内存分配流程？

- [x]  

>  

伙伴系统的内存回收流程？

- [x]  

>  

struct list_entry是如何把数据元素组织成链表的？

- [x]  

>  



