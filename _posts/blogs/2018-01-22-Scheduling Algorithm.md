
---
layout: page
title: 操作系统篇-八大调度算法实现**
    - blogs
---



# **操作系统篇-八大调度算法实现**

----------
[TOC]

听说隔壁CS专业OS的期末考试是模拟各种操作系统算法，听起来挺有趣,也不需要在linux上折腾，有空可以试着写写,把操作系统的知识打扎实。什么?网易游戏面试游戏开发应届生考操作系统也是这么考？赶紧撸起来。

## 数据结构 ##
```
//事件类型
enum EventType { ARRIVAL_EVENT, FINISH_EVENT, TIMER_EVENT };
//任务结构
struct Job
{
	string name;       //作业名
	int arriveTime=0;      //作业到达时间
	float needTime=0;        //作业所需运行时间
	int priority=0;        //作业优先级，数字越小，优先级越高
	Job(string name, int at, int nt, int p) {
		this->name = name;
		needTime = nt;
		arriveTime = at;
		priority = p;
	}
	Job(){}
};
//事件链表结点
struct Event
{
	EventType type= ARRIVAL_EVENT;            //事件类型
	int jobBeginTime=0;          //作业开始时间
	bool isFirstExe=true;           //判断是否第一次执行，用于记录作业开始时间
	int happenTime=0;            //发生时刻
	int remainTime=0;            //剩余运行时间
	double hrr=0;                //最高响应比
	Job job;                   //作业    
	Event *next=NULL;               //事件指针
	Event()
	{
		type = ARRIVAL_EVENT;
		jobBeginTime = 0;
		happenTime = 0;
		remainTime = 0;
		hrr = 0;
		job = Job();
		next = NULL;
		isFirstExe = true;
	}
	Event(int ht, int jbt, Job j)
	{
		Event();
		happenTime = ht;
		jobBeginTime = jbt;
		job = j;
		remainTime = j.needTime;
	}
};
```
## 相关函数 ##
```
//*******************************事件队列相关函数************************************
/* 作业复制函数 */
void CopyJob(Job *dest, const Job *sour);

/* 抓取头结点之外的第一个事件
* 作为返回值返回 */
Event *FetchFirstEvent(Event *head);

/* 按作业名称删除第一个匹配项，
* 删除成功返回TRUE，不存在该节点返回FALSE
*/
bool DeleteByJobName(Event *head, string jobName);

/* 删除事件队列 */
void DestroyQueue(Event *head);

//打印事件
void ShowEvent(Event* eventHead,Event *e);

//事件初始化
void InitEvent(Event* &seventHead);
//*******************************进程调度相关函数************************************
/* 插入函数 */
void InsertByHappenTime(Event *head, Event *e);
void InsertByHRR(Event *head, Event *e);

void InsertByJobTime(Event* eventHead,list<Event*> &q, Event *e);
void InsertByPriority(Event* eventHead, list<Event*> &q, Event *e);
void InsertByRemainTime(Event*  eventHead, list<Event*> &q, Event *e);
void InsertTail(Event*  eventHead, list<Event*> &head, Event *e);

/* 排序函数 */
void SortByHRR(list<Event*> &q, int currentTime);

/* 调度函数 */
void FCFS(Event *eventHead);
void SJF(Event *eventHead);
void SRTF(Event *eventHead);
void HRRF(Event *eventHead);
void Priority(Event * eventHead);    //抢占式优先级调用
void RR(Event *eventHead);      //时间片大小为1
void MFQ(Event *eventHead);
```
这里就不具体给出实现了，详情去看我的github这个项目的完整实现
https://github.com/IceLanguage/OSAlgorithm/tree/master/SchedulingAlgorithm

## 先来先服务算法(FCFS)
最简单的调度算法，既可以用于作业调度，也可以用于程序调度，当作业调度中采用该算法时，系统将按照作业到达的先后次序来进行调度，优先从后备队列中，选择一个或多个位于队列头部的作业，把他们调入内存，分配所需资源、创建进程，然后放入“就绪队列”,直到该进程运行到完成或发生某事件堵塞后，进程调度程序才将处理机分配给其他进程。为非剥夺式调度算法

简单来说就是在排队，先到先得，毫无效率

实现如下
```
void FCFS(Event *eventHead)
{
	InitEvent(eventHead);

	int currentTime = 0;            //当前时间
	
	while (eventHead->next!=NULL)
	{
		Event *firstEvent = FetchFirstEvent(eventHead);
		
		currentTime = firstEvent->happenTime;
		
		Event *finishEvent = new Event;
		finishEvent->type = FINISH_EVENT;
		finishEvent->jobBeginTime = currentTime;
		finishEvent->happenTime = currentTime + firstEvent->job.needTime;
		finishEvent->remainTime = 0;
		CopyJob(&(finishEvent->job), &(firstEvent->job));

		ShowEvent(eventHead,finishEvent);

	

	}
}
```
##最短作业优先算法（SJF）
以进入系统的作业所要求的CPU时间为标准，总选取估计计算时间最短的作业投入运行。为非剥夺式调度算法。 
不过需要提供每个任务的作用时间，现实中很难估计。而且忽视了作业等待时间，会出现饥饿现象

实现如下
```
//短作业优先算法
void SJF(Event *eventHead)
{
	InitEvent(eventHead);

	int currentTime = 0;            //当前时间
	bool isCpuBusy = false;        //CPU忙碌状态
	list<Event*> readyQueue;
	//Event *readyQueue =CreateEventQueue(); //包含头结点，就绪队列
	Event *firstEvent;
	while (eventHead->next)
	{
		firstEvent = FetchFirstEvent(eventHead);
		currentTime = firstEvent->happenTime;
		if (firstEvent->type ==ARRIVAL_EVENT)
		{
			if (isCpuBusy)
			{
				InsertByJobTime(eventHead,readyQueue, firstEvent);
				
			}
			else {
				isCpuBusy = true;

				Event *finishEvent = new Event;
				finishEvent->type = FINISH_EVENT;
				finishEvent->jobBeginTime = currentTime;
				finishEvent->happenTime = currentTime + firstEvent->job.needTime;
				finishEvent->remainTime = 0;
				CopyJob(&(finishEvent->job), &(firstEvent->job));

				

				InsertByHappenTime(eventHead, finishEvent);
			}
			
		}
		if (firstEvent->type == FINISH_EVENT)
		{
			ShowEvent(eventHead,firstEvent);
			
			isCpuBusy = false;
			if (readyQueue.empty()) return;
				
			Event *shortestEvent = readyQueue.front();
			readyQueue.pop_front();
			if (shortestEvent)
			{
				isCpuBusy = true;

				Event *finishEvent = new Event();
				finishEvent->type = FINISH_EVENT;
				finishEvent->jobBeginTime = currentTime;
				finishEvent->happenTime = currentTime + shortestEvent->job.needTime;
				finishEvent->remainTime = shortestEvent->job.needTime;
				CopyJob(&(finishEvent->job), &(shortestEvent->job));

				

				InsertByHappenTime(eventHead, finishEvent);
			}
		}
	}
}
```
##最短剩余时间优先算法
最短剩余时间优先SRTF算法把SJF算法改为剥夺式调度算法。

当一个作业正在执行时，一个新作业进入就绪状态，如果新作业需要的CPU时间比当前正在执行的作业剩余下来还需的CPU时间短，SRTF强行赶走当前正在执行的作业。此算法不仅适用于作业调度，同样也适用于进程调度
```
//最短剩余时间优先算法
void SRTF(Event* eventHead)
{
	InitEvent(eventHead);

	int currentTime = 0;            //当前时间
	bool isCpuBusy = false;        //CPU忙碌状态
	list<Event*> readyQueue;
	Event *firstEvent;
	Event *runEvent = NULL;            //正在运行事件节点

	while (eventHead->next)
	{
		firstEvent = FetchFirstEvent(eventHead);
		int frontTime = currentTime;
		currentTime = firstEvent->happenTime;
		if (firstEvent->type == ARRIVAL_EVENT)
		{
			if (isCpuBusy)
			{
				runEvent->happenTime = currentTime;
				runEvent->remainTime -= (currentTime - frontTime);//剩余时间 = 运行时间- 已运行时间（本次-上次时间）
				if (firstEvent->remainTime < runEvent->remainTime)
				{                                                        //抢断
					
					InsertByRemainTime(eventHead,readyQueue, runEvent);          //上次事件加入就绪队列
				

					runEvent = firstEvent;                               //上次事件已插入继续队列，无须释放空间,更新本次事

					Event *finishEvent = new Event;
					finishEvent->type = FINISH_EVENT;
					finishEvent->jobBeginTime = runEvent->jobBeginTime;//
					finishEvent->happenTime = currentTime + runEvent->remainTime;
					finishEvent->remainTime = runEvent->remainTime;
					CopyJob(&(finishEvent->job), &(runEvent->job));

					runEvent = finishEvent;                               //上次事件已插入继续队列，无须释放空间,更新本次事件
					runEvent->isFirstExe = false;

					
					InsertByHappenTime(eventHead, finishEvent);      //插入结束事件
				}
				else
				{
					
					InsertByRemainTime(eventHead,readyQueue, firstEvent);//等待，加入就绪事件队列
					
				}
			}
			else {
				isCpuBusy = true;
				firstEvent->isFirstExe = false;

				Event *finishEvent = new Event;
				finishEvent->type = FINISH_EVENT;
				finishEvent->jobBeginTime = firstEvent->jobBeginTime;    //
				finishEvent->isFirstExe = false;
				finishEvent->happenTime = currentTime + firstEvent->remainTime;
				finishEvent->remainTime = firstEvent->remainTime;
				CopyJob(&(finishEvent->job), &(firstEvent->job));

				runEvent = firstEvent;

			
				InsertByHappenTime(eventHead, finishEvent);
			}
			continue;

		}
		if (firstEvent->type == FINISH_EVENT)
		{
			ShowEvent(eventHead,firstEvent);

		

			isCpuBusy = false;
			if (readyQueue.empty()) return;

			Event *shortestEvent = readyQueue.front();
			readyQueue.pop_front();
			if (shortestEvent)
			{
				isCpuBusy = true;

				if (shortestEvent->isFirstExe)    //在就绪队列中，若作业首次执行，必然是延迟发生的
					shortestEvent->jobBeginTime = currentTime;
				shortestEvent->isFirstExe = false;

				Event *finishEvent = new Event();
				finishEvent->type = FINISH_EVENT;
				finishEvent->jobBeginTime = currentTime;
				finishEvent->happenTime = currentTime + shortestEvent->job.needTime;
				finishEvent->remainTime = shortestEvent->job.needTime;
				CopyJob(&(finishEvent->job), &(shortestEvent->job));


				runEvent = shortestEvent;
				InsertByHappenTime(eventHead, finishEvent);
			}
		}
	}
}
```
##响应比最高者优先算法
FCFS、SJF算法的缺点：先来先服务算法FCFS与最短作业优先SJF算法都是片面的调度算法。FCFS只考虑作业等候时间而忽视了作业的计算时间，SJF算法只考虑用户估计的作业计算时间而忽略了作业等待时间。

响应比调度思想：响应比最高者优先（HRRF）算法是介乎这两者之间的折中算法，既考虑作业等待时间，又考虑作业的运行时间，既照顾短作业又不使长作业的等待时间过长，改善了调度性能。

缺点：每次计算各道作业的响应比会有一定的时间开销，性能比SJF略差。
响应比 = 1+已等待时间/作业处理时间

这里给出按最高响应比排序的代码
```
bool EventHRRCompareRules(Event* _X, Event* _Y) // 回调函数  
{
	if (_X->hrr <_Y->hrr) return false;
	return true;
}
//按最高响应比排序
void SortByHRR(list<Event*> &q, int currentTime)
{
	list<Event*> ::iterator it;
	
	it = q.begin();
	for(; it != q.end(); it++)
	{		
		Event* e = *it;
		e->happenTime = currentTime;
		e->hrr = (e->happenTime - e->job.arriveTime) / e->job.needTime*0.1;
		*it = e;
	}
	q.sort(EventHRRCompareRules);
}
```
算法实现
```
//响应比最高者优先算法
void HRRF(Event * eventHead)
{
	InitEvent(eventHead);

	int currentTime = 0;            //当前时间
	bool  isCpuBusy = false;        //CPU忙碌状态
	list<Event*> readyQueue;
	Event *firstEvent;
	while (eventHead->next)
	{
		firstEvent = FetchFirstEvent(eventHead);
		currentTime = firstEvent->happenTime;
		if (firstEvent->type == ARRIVAL_EVENT)
		{
			if (isCpuBusy == true)
			{
				InsertTail(eventHead,readyQueue, firstEvent);
			}
			else
			{
				
				isCpuBusy = true;

				Event *finishEvent = new Event;
				finishEvent->type = FINISH_EVENT;
				finishEvent->jobBeginTime = currentTime;
				finishEvent->happenTime = currentTime + firstEvent->job.needTime;
				CopyJob(&(finishEvent->job), &(firstEvent->job));

				InsertByHappenTime(eventHead, finishEvent);
			}
			continue;
		}
		if (firstEvent->type == FINISH_EVENT)
		{
			ShowEvent(eventHead,firstEvent);

			isCpuBusy = false;
			SortByHRR(readyQueue, currentTime);
			if (readyQueue.empty()) return;

			Event *hrrEvent = readyQueue.front();
			readyQueue.pop_front();
			if (hrrEvent != NULL)
			{
				isCpuBusy = true;
				Event *finishEvent = new Event();
				finishEvent->type = FINISH_EVENT;
				finishEvent->jobBeginTime = currentTime;
				finishEvent->happenTime = currentTime + hrrEvent->job.needTime;
				finishEvent->remainTime = hrrEvent->job.needTime;
				CopyJob(&(finishEvent->job), &(hrrEvent->job));

				InsertByHappenTime(eventHead, finishEvent);
			}
		}
	}
}
```
##优先级调度算法
根据确定的优先级来选取进程/线程，每次总是选择就绪队列中优先级最高者运行。是变相的FCFS
```
//优先级调度算法
void Priority(Event * eventHead)
{
	InitEvent(eventHead);

	int currentTime = 0;            //当前时间
	bool isCpuBusy = false;        //CPU忙碌状态
	list<Event*> readyQueue;
	Event *firstEvent;
	Event *runEvent = NULL;            //正在运行事件节点
	while (eventHead->next)
	{
		firstEvent = FetchFirstEvent(eventHead);
		int frontTime = currentTime;
		currentTime = firstEvent->happenTime;
		if (firstEvent->type == ARRIVAL_EVENT)
		{
			if (isCpuBusy)
			{
				runEvent->happenTime = currentTime;
				runEvent->remainTime -= (currentTime - frontTime);//剩余时间 = 运行时间- 已运行时间（本次-上次时间）
				if (firstEvent->job.priority < runEvent->job.priority)
				{                                                      //抢断

					InsertByPriority(eventHead, readyQueue, runEvent);          //上次事件加入就绪队列

					runEvent = firstEvent;                              //上次事件已插入继续队列，无须释放空间,更新本次事件
					runEvent->isFirstExe = false;                      //

					Event *finishEvent = new Event;
					finishEvent->type = FINISH_EVENT;
					finishEvent->jobBeginTime = runEvent->jobBeginTime;//
					finishEvent->happenTime = currentTime + runEvent->remainTime;
					finishEvent->remainTime = 0;
					CopyJob(&(finishEvent->job), &(runEvent->job));

					InsertByHappenTime(eventHead, finishEvent);      //插入结束事件
				}
				else {
				
					InsertByRemainTime(eventHead, readyQueue, firstEvent);//等待，加入就绪事件队列
				}
			}
			else {
				isCpuBusy = true;
				firstEvent->isFirstExe = false;

				Event *finishEvent = new Event;
				finishEvent->type = FINISH_EVENT;
				finishEvent->jobBeginTime = firstEvent->jobBeginTime;
				finishEvent->happenTime = currentTime + firstEvent->remainTime;
				finishEvent->remainTime = 0;
				CopyJob(&(finishEvent->job), &(firstEvent->job));

				runEvent = firstEvent;

				InsertByHappenTime(eventHead, finishEvent);
			}
		

		}
		if (firstEvent->type == FINISH_EVENT)
		{
			ShowEvent(eventHead, firstEvent);

			isCpuBusy = false;
	
			if (readyQueue.empty()) return;

			Event *priorityEvent = readyQueue.front();
			readyQueue.pop_front();
			if (priorityEvent != NULL)
			{
				isCpuBusy = true;
				Event *finishEvent = new Event();
				finishEvent->type = FINISH_EVENT;
				finishEvent->jobBeginTime = priorityEvent->jobBeginTime;
				finishEvent->happenTime = currentTime + priorityEvent->remainTime;
				finishEvent->remainTime = 0;
				CopyJob(&(finishEvent->job), &(priorityEvent->job));

				runEvent = priorityEvent;
				InsertByHappenTime(eventHead, finishEvent);
			}
			
		}
	}
}
```

##轮转调度算法
轮转法调度也称为时间片调度，其做法是：调度程序每次把CPU分配给就绪队列首进程/线程使用一个时间间隔（时间片），就绪队列中的每个进程/线程轮流地运行一个时间片。当这个时间片耗尽时，强迫当前进程/线程让出处理器，让它排列到就绪队列的尾部，等候下一轮调度
```
//轮转调度算法
void RR(Event * eventHead)
{
	InitEvent(eventHead);

	int currentTime = 0;            //当前时间
	bool isCpuBusy = false;        //CPU忙碌状态
	list<Event*> readyQueue;
	Event *firstEvent;

	while (eventHead->next)
	{
		firstEvent = FetchFirstEvent(eventHead);
		if(firstEvent->happenTime>currentTime) currentTime = firstEvent->happenTime;

		if (firstEvent->type == ARRIVAL_EVENT)
		{
			if (isCpuBusy)
			{
				InsertTail(eventHead, readyQueue, firstEvent);
			}
			else
			{
				isCpuBusy = true;
				Event *nextEvent = new Event;
				nextEvent->jobBeginTime = firstEvent->jobBeginTime;
				CopyJob(&(nextEvent->job), &(firstEvent->job));
				
				if (firstEvent->remainTime <= TIMER)    //FinishEvent
				{
					nextEvent->type = FINISH_EVENT;
					nextEvent->happenTime = currentTime + firstEvent->remainTime;
					nextEvent->remainTime = 0;
					InsertByHappenTime(eventHead, nextEvent);
				}
				else    //TimerEvent
				{
					nextEvent->type = TIMER_EVENT;
					nextEvent->happenTime = currentTime + TIMER;
					nextEvent->remainTime = firstEvent->remainTime - TIMER;
					InsertByHappenTime(eventHead, nextEvent);
				}

				
			}
			continue;
		}
		if (firstEvent->type == TIMER_EVENT)
		{
			InsertTail(eventHead,readyQueue, firstEvent);
		}

		int isFinish = false;
		if (firstEvent->type == FINISH_EVENT)
		{
			ShowEvent(eventHead,firstEvent);

			isFinish = true;
		}
	
		while (!readyQueue.empty())
		{
			
			firstEvent = readyQueue.front();
			readyQueue.pop_front();
			if (isFinish)
				isCpuBusy = true, isFinish = false;
			Event *nextEvent = new Event;
			if (firstEvent->type == ARRIVAL_EVENT)
				nextEvent->jobBeginTime = currentTime;
			else if (firstEvent->type == TIMER_EVENT)
				nextEvent->jobBeginTime = firstEvent->jobBeginTime;
			CopyJob(&(nextEvent->job), &(firstEvent->job));
			if (firstEvent->remainTime <= TIMER)    //FinishEvent
			{
				nextEvent->type = FINISH_EVENT;
				nextEvent->happenTime = currentTime + firstEvent->remainTime;
				nextEvent->remainTime = 0;
				InsertByHappenTime(eventHead, nextEvent);
			}
			else    //TimerEvent
			{
				nextEvent->type = TIMER_EVENT;
				nextEvent->happenTime = currentTime + TIMER;
				nextEvent->remainTime = firstEvent->remainTime - TIMER;
				InsertByHappenTime(eventHead, nextEvent);
			}

			currentTime = nextEvent->happenTime;
			
		}
		
	}
}
```

##多级反馈队列调度
是将就绪进程分为两级或多级，系统相应建立两个或多个就绪进程队列，较高优先级的队列一般分配给较短的时间片。
处理器调度每次先从高级就绪队列中选取可占有处理器的进程，只有在选不到时，才从较低级的就绪进程队列中选取。
同一队列中的进程按先来先服务原则排队

实际上之前轮转调度算法的改进，从就绪队列选取优先级高的任务
```
//多级反馈队列调度
void MFQ(Event * eventHead)
{
	InitEvent(eventHead);

	int currentTime = 0;            //当前时间
	bool isCpuBusy = false;        //CPU忙碌状态
	list<Event*> readyQueue;
	Event *firstEvent;

	while (eventHead->next)
	{
		firstEvent = FetchFirstEvent(eventHead);
		if (firstEvent->happenTime>currentTime) currentTime = firstEvent->happenTime;

		if (firstEvent->type == ARRIVAL_EVENT)
		{
			if (isCpuBusy)
			{
				InsertByPriority(eventHead, readyQueue, firstEvent);
			}
			else
			{
				isCpuBusy = true;
				Event *nextEvent = new Event;
				nextEvent->jobBeginTime = firstEvent->jobBeginTime;
				CopyJob(&(nextEvent->job), &(firstEvent->job));

				if (firstEvent->remainTime <= TIMER)    //FinishEvent
				{
					nextEvent->type = FINISH_EVENT;
					nextEvent->happenTime = currentTime + firstEvent->remainTime;
					nextEvent->remainTime = 0;
					InsertByHappenTime(eventHead, nextEvent);
				}
				else    //TimerEvent
				{
					nextEvent->type = TIMER_EVENT;
					nextEvent->happenTime = currentTime + TIMER;
					nextEvent->remainTime = firstEvent->remainTime - TIMER;
					InsertByHappenTime(eventHead, nextEvent);
				}


			}
			continue;
		}
		if (firstEvent->type == TIMER_EVENT)
		{
			InsertByPriority(eventHead, readyQueue, firstEvent);
		}

		int isFinish = false;
		if (firstEvent->type == FINISH_EVENT)
		{
			ShowEvent(eventHead, firstEvent);

			isFinish = true;
		}

		while (!readyQueue.empty())
		{

			firstEvent = readyQueue.front();
			readyQueue.pop_front();
			if (isFinish)
				isCpuBusy = true, isFinish = false;
			Event *nextEvent = new Event;
			if (firstEvent->type == ARRIVAL_EVENT)
				nextEvent->jobBeginTime = currentTime;
			else if (firstEvent->type == TIMER_EVENT)
				nextEvent->jobBeginTime = firstEvent->jobBeginTime;
			CopyJob(&(nextEvent->job), &(firstEvent->job));
			if (firstEvent->remainTime <= TIMER)    //FinishEvent
			{
				nextEvent->type = FINISH_EVENT;
				nextEvent->happenTime = currentTime + firstEvent->remainTime;
				nextEvent->remainTime = 0;
				InsertByHappenTime(eventHead, nextEvent);
			}
			else    //TimerEvent
			{
				nextEvent->type = TIMER_EVENT;
				nextEvent->happenTime = currentTime + TIMER;
				nextEvent->remainTime = firstEvent->remainTime - TIMER;
				InsertByHappenTime(eventHead, nextEvent);
			}

			currentTime = nextEvent->happenTime;


		}

	}
}
```

完整代码地址
https://github.com/IceLanguage/OSAlgorithm
开心赏个star吧
![这里写图片描述](http://img.blog.csdn.net/20180122000942026?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQyNDQzMTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
