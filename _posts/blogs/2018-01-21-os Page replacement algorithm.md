---
layout: page
title: 操作系统篇-页面置换算法c/c++实现

---

[TOC]
## 数据结构

```
const int MAXSIZE = 1000;//定义最大页面数  
const int MAXQUEUE = 3;//定义页框数  

typedef struct node {
	int loaded;
	int hit;
}page;

page pages[MAXQUEUE]; //定义页框表  
int queue[MAXSIZE];
int quantity;

```

## 相关函数

```
//初始化结构函数  
void initial()
{
	int i;

	for (i = 0; i<MAXQUEUE; i++) {
		pages[i].loaded = -1;
		pages[i].hit = 0;
	}
	for (i = 0; i<MAXSIZE; i++) {
		queue[i] = -1;
	}

	quantity = 0;
}

//初始化页框函数  
void init()
{
	int i;

	for (i = 0; i<MAXQUEUE; i++) {
		pages[i].loaded = -1;
		pages[i].hit = 0;
	}
}

//读入页面流  
void readData()
{
	FILE *fp;
	char fname[20];

	int i;

	cout << "请输入页面流文件名:";
	cin >> fname;

	if ((fp = fopen(fname, "r")) == NULL) {
		cout << "错误,文件打不开,请检查文件名";
	}
	else {
		while (!feof(fp)) {
			fscanf(fp, "%d ", &queue[quantity]);
			quantity++;
		}
	}

	cout << "读入的页面流:";
	for (i = 0; i<quantity; i++) {
		cout << queue[i] << " ";
	}
}
```
## 主函数

```
void main()
{
	
	initial();

	readData();

	FIFO();
	init();

	LRU();
	init();

	LFU();
	init();

	second();

}
```
## FIFO调度算法

最简单的页面置换算法是先入先出（FIFO）法。这种算法的实质是，总是选择在主存中停留时间最长（即最老）的一页置换，即先进入内存的页，先退出内存
```
void FIFO()
{
	int i, j, p, flag;
	int absence = 0;

	p = 0;

	cout << endl << "----------------------------------------------------" << endl;
	cout << "FIFO调度算法页面调出流:";
	for (i = 0; i<quantity; i++) {
		flag = 0;
		for (j = 0; j<MAXQUEUE; j++) {
			if (pages[j].loaded == queue[i]) {
				flag = 1;
			}
		}
		if (flag == 0) {
			if (absence >= MAXQUEUE) {
				cout << pages[p].loaded << " ";
			}
			pages[p].loaded = queue[i];
			p = (p + 1) % MAXQUEUE;
			absence++;
		}
	}
	absence -= MAXQUEUE;
	cout << endl << "总缺页数:" << absence << endl;

}
```
## 最近最久未使用调度算法（LRU）

FIFO算法和OPT算法之间的主要差别是，FIFO算法利用页面进入内存后的时间长短作为置换依据，而OPT算法的依据是将来使用页面的时间。如果以最近的过去作为不久将来的近似，那么就可以把过去最长一段时间里不曾被使用的页面置换掉。它的实质是，当需要置换一页时，选择在之前一段时间里最久没有使用过的页面予以置换。这种算法就称为最久未使用算法
```
void LRU()
{
	int absence = 0;
	int i, j;
	int flag;


	for (i = 0; i<MAXQUEUE; i++) {
		pages[i].loaded = queue[i];
	}

	cout << endl << "----------------------------------------------------" << endl;
	cout << "最近最少使用调度算法(LRU)页面流:";

	for (i = MAXQUEUE; i<quantity; i++) {
		flag = -1;
		for (j = 0; j<MAXQUEUE; j++) {
			if (queue[i] == pages[j].loaded) {
				flag = j;
			}
		}
		//CAUTION pages[0]是队列头  
		if (flag == -1) {
			//缺页处理  
			cout << pages[0].loaded << " ";
			for (j = 0; j<MAXQUEUE - 1; j++) {
				pages[j] = pages[j + 1];
			}
			pages[MAXQUEUE - 1].loaded = queue[i];
			absence++;
		}
		else {
			//页面已载入  
			pages[quantity] = pages[flag];
			for (j = flag; j<MAXQUEUE - 1; j++) {
				pages[j] = pages[j + 1];
			}
			pages[MAXQUEUE - 1] = pages[quantity];
		}


	}

	cout << endl << "总缺页数:" << absence << endl;
}
```
## 最近最不常用调度算法(LFU) 

LRU (Least Recently Used): 最近最少使用调度算法，首先丢弃最近最少被使用的数据。
LFU (Least Frequently Used): 最近最不常用调度算法，软件统计数据被使用的频率，使用频率最低的数据首先被丢弃。
```
void LFU()
{
	int i, j, p;
	int absence = 0;
	int flag;

	for (i = 0; i<MAXQUEUE; i++) {
		pages[i].loaded = queue[i];
	}

	cout << endl << "----------------------------------------------------" << endl;
	cout << "最近最不常用调度算法(LFU)页面流:";

	for (i = MAXQUEUE; i<quantity; i++) {
		flag = -1;
		for (j = 0; j<MAXQUEUE; j++) {
			if (pages[j].loaded == queue[i]) {
				flag = 1;
				pages[j].hit++;
			}
		}
		if (flag == -1) {
			//缺页中断  
			p = 0;
			for (j = 0; j<MAXQUEUE; j++) {
				if (pages[j].hit<pages[p].hit) {
					p = j;
				}
			}
			cout << pages[p].loaded << " ";
			pages[p].loaded = queue[i];
			for (j = 0; j<MAXQUEUE; j++) {
				pages[j].hit = 0;
			}
			absence++;
		}
	}
	cout << endl << "总缺页数:" << absence << endl;
}
```
## 第二次机会算法

第二次机会算法的基本思想是与FIFO相同的，但是有所改进，避免把经常使用的页面置换出去。当选择置换页面时，检查它的访问位。如果是 0，就淘汰这页；如果访问位是1，就给它第二次机会，并选择下一个FIFO页面。当一个页面得到第二次机会时，它的访问位就清为0，它的到达时间就置为当前时间。如果该页在此期间被访问过，则访问位置1。这样给了第二次机会的页面将不被淘汰，直至所有其他页面被淘汰过（或者也给了第二次机会）。因此，如果一个页面经常使用，它的访问位总保持为1，它就从来不会被淘汰出去
第二次机会算法可视为一个环形队列。用一个指针指示哪一页是下面要淘汰的。当需要一个存储块时，指针就前进，直至找到访问位是0的页。随着指针的前进，把访问位就清为0。在最坏的情况下，所有的访问位都是1，指针要通过整个队列一周，每个页都给第二次机会。这时就退化成FIFO算法了。
```
void second()
{
	int i, j, t;
	int absence = 0;
	int flag, temp;

	for (i = 0; i<MAXQUEUE; i++) {
		pages[i].loaded = queue[i];
	}

	cout << endl << "----------------------------------------------------" << endl;
	cout << "第二次机会算法页面流:";

	for (i = MAXQUEUE; i<quantity; i++) {
		flag = -1;
		for (j = 0; j<MAXQUEUE; j++) {
			if (pages[j].loaded == queue[i]) {
				flag = 1;
				pages[j].hit = 1;
			}
		}
		if (flag == -1) {
			//缺页处理  
			t = 0;
			while (t == 0) {
				if (pages[0].hit == 0) {
					cout << pages[0].loaded << " ";
					for (j = 0; j<MAXQUEUE - 1; j++) {
						pages[j] = pages[j + 1];
					}
					pages[MAXQUEUE - 1].loaded = queue[i];
					pages[MAXQUEUE - 1].hit = 0;


					t = 1;
				}
				else {
					temp = pages[0].loaded;
					for (j = 0; j<MAXQUEUE - 1; j++) {
						pages[j] = pages[j + 1];
					}
					pages[MAXQUEUE - 1].loaded = temp;
					pages[MAXQUEUE - 1].hit = 0;
				}
			}
			absence++;
		}
	}
	cout << endl << "总缺页数:" << absence << endl;
}
```

代码来自http://www.doc88.com/p-9807270632375.html

