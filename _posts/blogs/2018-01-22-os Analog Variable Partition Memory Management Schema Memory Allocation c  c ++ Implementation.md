
---
layout: page
title: 操作系统篇-模拟可变分区存储管理方案中的内存分配c/c++实现
    - blogs
---

# **操作系统篇-模拟可变分区存储管理方案中的内存分配c/c++实现**


----------
[TOC]

系统根据申请者的要求，按照一定的分配策略分析内存空间的使用情况，找出能满足请求的空闲区，分给申请者；当程序执行完毕或主动归还内存资源时，系统要收回它所占用的内存空间或它归还的部分内存空间，主存分配算法使用最坏适应分配算法。
##数据结构
```
//表最大记录数
const int jobMax = 100;

struct Job {
	int start;//起始地址
	int length;//空间大小
	string tag;
};

//空闲区表
Job Free[jobMax];

//空闲区计数器
int free_Quantity;

//已分配区表
Job occupy[jobMax];

//已分配区计数器
int occupy_Quantity;
```
##相关函数
```
//初始化函数
void Init();

//读取数据
void ReadData();

//空闲表根据起始地址信息进行顺序排序
void SortByStart();

//显示
void ShowView();

//最先适应分配算法
void FF();

//最优适应分配算法
void bestFit();

//最坏适应分配算法
void worstFit();

//撤销作业
void UndoJob();
```
##main
```
int main()
{
	int flag1=1, flag2;
	int chioce1, chioce2;
	
	Init();
	ReadData();

	while (flag1)
	{
		SortByStart();
		cout << "1-申请空间" << "  " << "2-撤销作业" << "  " << "3-显示空闲表和分配表"
			<< "  " << "4-退出" << "  " << endl;
		cin >> chioce1;
		switch (chioce1)
		{
			case 1:
				flag2 = 1;
				while (flag2)
				{
					cout << "1-最先适应分配算法" << "  " << "2-最优适应分配算法" 
						<< "  " << "3-最坏适应分配算法"<< "  " << "4-退出" << 
						"  " << endl;
					cin >> chioce2;
					switch (chioce2)
					{
						case 1:
							FF();
							break;
						case 2:
							bestFit();
							break;
						case 3:
							worstFit();
							break;
						case 4:
							flag2 = 0;
							break;
						default:
							break;
					}
				}
				break;
			case 2:
				UndoJob();
				break;
			case 3:
				ShowView();
				break;
			case 4:
				flag1 = 0;
				break;
			default:
				cout << "wrong"<<endl;
				break;
		}
	}
    return 0;
}
```
##最先适应分配算法
```
void FF()
{
	string jobname;
	int joblength;
	cout << "请输入新申请内存空间的作业名和空间大小" << endl;
	cin >> jobname >> joblength;
	int flag = 0;
	int i;
	for (i = 0; i < free_Quantity; i++)
	{
		//检查是否有满足要求的空闲区
		if (Free[i].length >= joblength)
		{
			flag = 1;
			break;
		}
	}
	if (!flag) cout << "当前没有满足你申请长度要求的空闲区" << endl;
	else {
		//根据最先适应分配算法进行分配
		
	
		//向分配表中添加信息
		occupy[occupy_Quantity].start = Free[i].start;
		occupy[occupy_Quantity].length = Free[i].length;
		occupy[occupy_Quantity].tag = Free[i].tag;
		occupy_Quantity++;

		
		if (Free[i].length >= joblength)
		{
			Free[i].start += joblength;
			Free[i].length -= joblength;
		}

		cout << "内存分配完成" << endl;
	}
}
```

##最优适应分配算法
```
void bestFit()
{
	string jobname;
	int joblength;
	cout << "请输入新申请内存空间的作业名和空间大小" << endl;
	cin >> jobname >> joblength;
	int flag = 0;
	int i;
	for (i = 0; i < free_Quantity; i++)
	{
		//检查是否有满足要求的空闲区
		if (Free[i].length >= joblength)
		{
			flag = 1;
			break;
		}
	}
	if (!flag) cout << "当前没有满足你申请长度要求的空闲区" << endl;
	else {
		
		if (!flag) cout << "当前没有满足你申请长度要求的空闲区" << endl;
		else {
			//根据最优适应分配算法进行分配
			for (int j = i+1; j < free_Quantity; j++)
			{
				//找到满足要求最小的一块
				if((Free[j].length>=joblength)&&(Free[j].length<Free[i].length))
				{
					i = j;
				}
			}

			//向分配表中添加信息
			occupy[occupy_Quantity].start = Free[i].start;
			occupy[occupy_Quantity].length = Free[i].length;
			occupy[occupy_Quantity].tag = Free[i].tag;
			occupy_Quantity++;


			if (Free[i].length >= joblength)
			{
				Free[i].start += joblength;
				Free[i].length -= joblength;
			}

			cout << "内存分配完成" << endl;
		}

		
	}
}
```
##最坏适应分配算法
```
void worstFit()
{
	string jobname;
	int joblength;
	cout << "请输入新申请内存空间的作业名和空间大小" << endl;
	cin >> jobname >> joblength;
	int flag = 0;
	int i;
	for (i = 0; i < free_Quantity; i++)
	{
		//检查是否有满足要求的空闲区
		if (Free[i].length >= joblength)
		{
			flag = 1;
			break;
		}
	}
	if (!flag) cout << "当前没有满足你申请长度要求的空闲区" << endl;
	else {

		if (!flag) cout << "当前没有满足你申请长度要求的空闲区" << endl;
		else {
			//根据最坏适应分配算法进行分配
			for (int j = i + 1; j < free_Quantity; j++)
			{
				//空闲块是所有申请块中最大的一块
				if ((Free[j].length >= joblength) && (Free[j].length>Free[i].length))
				{
					i = j;
				}
			}

			//向分配表中添加信息
			occupy[occupy_Quantity].start = Free[i].start;
			occupy[occupy_Quantity].length = Free[i].length;
			occupy[occupy_Quantity].tag = Free[i].tag;
			occupy_Quantity++;


			if (Free[i].length >= joblength)
			{
				Free[i].start += joblength;
				Free[i].length -= joblength;
			}

			cout << "内存分配完成" << endl; 
		}


	}
}
```

##取消内存分配
```
void UndoJob()
{
	int p;
	cout << "输入运行结束需释放空间的作业名" << endl;
	string job_name;
	cin >> job_name;
	int flag = -1;
	int start, length;
	for (int i = 0; i < occupy_Quantity; i++)
	{
		if (occupy[i].tag == job_name)
		{
			flag = i;
			start = occupy[i].start;
			length = occupy[i].length;
			break;
		}
	}
	if (flag = -1) cout << "没找到该作业名" << endl;
	else {
		for (int i = 0; i < occupy_Quantity; i++)
		{
			if (Free[i].start + Free[i].length == start)
			{
				if (i + 1 < free_Quantity&&Free[i + 1].start == start + length)
				{
					//满足要撤销的作业前后面有空闲区
					Free[i].length = Free[i].length + Free[i + 1].length + length;

					for (int j = i + 1; j < free_Quantity; j++)
					{
						Free[j] = Free[j + 1];
					}
					free_Quantity--;
					p = 1;
				}
				else {
					//满足要撤销的作业前面有空闲区
					Free[i].length += length;
					p = 1;
				}
			}
			if (Free[i].start == start + length)
			{
				//满足要撤销的作业后面有空闲区
				Free[i].start = start;
				Free[i].length += length;
				p = 1;
			}
		}
		if (p == 0)
		{
			//满足要撤销的作业前后面无空闲区
			Free[free_Quantity].start = start;
			Free[free_Quantity].length = length;
			free_Quantity++;
		}
		for (int i = flag; i < occupy_Quantity; i++)
		{
			occupy[i] = occupy[i + 1];
		}
		occupy_Quantity--;
	}
}
```

##完整代码
https://github.com/IceLanguage/OSAlgorithm/tree/master/MemoryAllocation