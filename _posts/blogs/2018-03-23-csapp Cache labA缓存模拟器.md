---
layout: page
title: 计算机基础篇-Cache labA缓存模拟器
category: 
    - blogs
---

#csapp Cache labA缓存模拟器


开始吧，Bash on Unbuntu on Windows10能运行Cache lab
（Attack lab因为不能运行被我放弃了，我会说？反正没人会看这篇博文）

##预热

首先按要求输入命令
可以把访问内存的日志输出到终端
```
valgrind --log-fd=1 --tool=lackey -v --trace-mem=yes ls -l 
```
最后得到一大串输出日志
下面只列出一部分在这篇博文
```
 S fff000228,8
I  04f18710,3
I  04f18713,7
 L 0520fe78,8
I  04f1871a,6
I  04f18720,5
I  04f18725,2
I  04f18740,3
I  04f18743,3
I  04f18746,2
==380==
==380== Counted 0 calls to main()
==380==
==380== Jccs:
==380==   total:         73,642
==380==   taken:         31,043 (42%)
==380==
==380== Executed:
==380==   SBs entered:   76,455
==380==   SBs completed: 49,975
==380==   guest instrs:  414,411
==380==   IRStmts:       2,441,773
==380==
==380== Ratios:
==380==   guest instrs : SB entered  = 54 : 10
==380==        IRStmts : SB entered  = 319 : 10
==380==        IRStmts : guest instr = 58 : 10
==380==
==380== Exit code:       0
```
每一条对内存访问的记录格式是 

[空格]操作符 地址,大小，

以 I 开头的是载入指令的记录，不算在内存访问中。

M 表示数据修改，需要一次载入 + 一次存储，也就是相当于两次访问

S 表示数据存储

L 表示数据载入

地址指的是一个 64 位的 16 进制内存地址

大小表示该操作内存访问的字节数

这次lab的任务是读入 valgrind 生成的 trace 日志，并统计出 hit, miss 和 eviction 的次数。

其实是做出参考程序 csim-ref一样的功能

需要修改的是 csim.c 和 trans.c。编译的时候只需要简单 make clean 和 make，然后就可以进行测试了。

##需要的头文件
```
#include "cachelab.h"
#include <stdio.h> /* fopen freopen perror */
#include <string.h>
#include <strings.h>
#include <unistd.h>/* getopt */
#include <getopt.h> /* getopt -std=c99 POSIX macros defined in <features.h> prevents <unistd.h> from including <getopt.h>*/
#include <stdlib.h>/* atol exit*/
#include <errno.h>  /* errno */
```
##宏定义
```
#define MAGIC_LRU_NUM 999
#define false 0
#define true 1
```
##定义数据结构
首先根据官方ppt(https://github.com/IceLanguage/csapp/blob/master/cachelab-handout/cachelab-ppt.pptx)16页给出的信息，在 csim.c 构建数据结构
```
A cache is just 2D array of cache lines:
    struct cache_line cache[S][E];
    S = 2^s,  is the number of sets
    E is associativity
    
Each cache_line has:
    Valid bit
    Tag
    LRU counter ( only if you are not using a queue ) 
```
```
struct CacheLine
{
	int validBit;//有效位
	int tag;//标识位
	int LRU_count;//LRU算法需要位
};

```
```
typedef struct{
    CacheLine* lines;    //cache set包含的行
} Set;

typedef struct {
    int set_num;    //组数
    int line_num;   //行数
    Set* sets;      //模拟cache
} MyCache;
```
##主函数和宏定义
main需要写成如下形式，参考ppt20页
```
int main(int argc,char **argv)
{
}
```
##程序逻辑

1.首先从命令中拿到所需的参数

2.再从文件中读入内容

3.进行 cache 的存储

4.printSummary
##LRU算法原理

找到最近最久未使用的那个页面调出内存
我的程序维护一个LRU_num达成LRU算法的实现

##完整代码

```
#include "cachelab.h"
#include <stdio.h> /* fopen freopen perror */
#include <string.h>
#include <strings.h>
#include <unistd.h>/* getopt */
#include <getopt.h> /* getopt -std=c99 POSIX macros defined in <features.h> prevents <unistd.h> from including <getopt.h>*/
#include <stdlib.h>/* atol exit*/
#include <errno.h>  /* errno */
#define MAGIC_LRU_NUM 999
#define false 0
#define true 1

typedef struct
{
	int valid;//有效位
	int tag;//标识位
	int LRU_num;//LRU算法需要位
}CacheLine;

typedef struct{
    CacheLine* lines;    //cache set包含的行
} CacheSet;

typedef struct {
    int set_num;    //组数
    int line_num;   //行数
    CacheSet* sets;      //模拟cache
} MyCache;

//全局变量
int misses;
int hits;
int evictions;
const char *help_message = "Usage: \"Your complied program\" [-hv] -s <s> -E <E> -b <b> -t <tracefile>\n" \
	                     "<s> <E> <b> should all above zero and below 64.\n" \
                         "Complied with std=c99\n";
/*检查参数合法性*/
void checkOptarg(char *curOptarg){
    if(curOptarg[0]=='-'){
        printf("./csim :Missing required command line argument\n");
        exit(0);
    }
}

/*初始化模拟内存*/
void init_Cache(int s,int E,int b,MyCache *cache){
    if(s < 0){
        printf("invaild cache sets number\n!");
        exit(0);
    }
    cache->set_num = 2 << s; //索引位数2^s 组

    cache->line_num = E;//（关联性）每行组数

    cache->sets = (CacheSet *)malloc(cache->set_num * sizeof(CacheSet));
    if(!cache->sets){
        printf("Set Memory error\n");
        exit(0);
    }
    int i ,j;
    for(i=0; i< cache->set_num; i++)
    {
        cache->sets[i].lines = (CacheLine *)malloc(cache->line_num*sizeof(CacheLine));
        if(!cache->sets){
            printf("Line Memory error\n");
            exit(0);
        }
        for(j=0; j < E; j++){
            cache->sets[i].lines[j].valid = false;
            cache->sets[i].lines[j].LRU_num = 0;
        }
    }
    return ;
}

/*获取地址中的组号*/
int getSet(int addr,int s,int b){
    addr = addr >> b;
    int mask =  (1<<s)-1;
    return addr &mask;
}

/*获取地址中的标识号*/
int getTag(int addr,int s,int b){
    int mask = s+b;
    return addr >> mask;
}
/*查找某组中当前最小的LRU_num行，作为牺牲行 */
int findMinLRUnum(MyCache *cache,int setBits){
    int i;
    int minIndex=0;
    int minLru = MAGIC_LRU_NUM;
    for(i=0;i<cache->line_num;i++){
        if(cache->sets[setBits].lines[i].LRU_num < minLru){
            minIndex = i;
            minLru = cache->sets[setBits].lines[i].LRU_num;
        }
    }
    return minIndex;
}
/*更新Lru，hit的为最大的MAGIC_LRU_NUM,其他LRU均减一*/
void updateLRUnum(MyCache *cache,int setBits,int hitIndex){
        cache->sets[setBits].lines[hitIndex].LRU_num = MAGIC_LRU_NUM;
        int j;
        for(j=0;j<cache->line_num;j++){//更新其他行的LRU_num
            if(j!=hitIndex) cache->sets[setBits].lines[j].LRU_num--;
        }
}
/*判断是否命中*/
int isMiss(MyCache *cache,int setBits,int tagBits){
    int i;
    int isMiss = true;
    for(i=0;i<cache->line_num;i++){
        if(cache->sets[setBits].lines[i].valid == true&& cache->sets[setBits].lines[i].tag == tagBits){
            isMiss = false;
            updateLRUnum(cache,setBits,i);
        }
    }
    return isMiss;
}

/*更新内存数据*/
int updateCache(MyCache *cache,int setBits,int tagBits){
    int i;
    int isfull = true;
    for(i=0;i<cache->line_num;i++){
        if(cache->sets[setBits].lines[i].valid ==false){
            isfull = false; //该组未满
            break;
        }
    }
    if(isfull == false){
        cache->sets[setBits].lines[i].valid = true;
        cache->sets[setBits].lines[i].tag = tagBits;
        updateLRUnum(cache,setBits,i);
    }else{
        //组已经满，需要牺牲行
        int evictionIndex = findMinLRUnum(cache,setBits);
        cache->sets[setBits].lines[evictionIndex].valid = true;
        cache->sets[setBits].lines[evictionIndex].tag = tagBits;
        updateLRUnum(cache,setBits,evictionIndex);
    }
    return isfull;
}

//M 表示数据修改，需要一次载入 + 一次存储，也就是相当于两次访问
void modifyData(MyCache *cache,int addr,int size,int setBits,int tagBits ,int isVerbose){
    loadData(cache,addr,size,setBits,tagBits,isVerbose);
    storeData(cache,addr,size,setBits,tagBits,isVerbose);
}

//S 表示数据存储
void storeData(MyCache *cache,int addr,int size,int setBits,int tagBits ,int isVerbose){
    loadData(cache,addr,size,setBits,tagBits,isVerbose);
}

//L 表示数据载入
void loadData(MyCache *cache,int addr,int size,int setBits,int tagBits ,int isVerbose){

    if(isMiss(cache,setBits,tagBits)==true){ //没有命中
        misses++;
        if(isVerbose ==true) printf("miss ");
        if(updateCache(cache,setBits,tagBits) == true){//该组已满，需要牺牲行
            evictions++;
            if(isVerbose==true) printf("eviction ");
        }
    }else{ //命中
       hits++;
       if(isVerbose == true) printf("hit ");
    }
}


int main(int argc,char **argv)
{
	char ch;    /* command options */
	int s,E,b;
	int isVerbose=false;
	char tracefileName[100],opt[100];
	int addr,size;
    misses = hits = evictions =false;
	MyCache cache;
	//读取参数
	while((ch = getopt(argc, argv, "hvs:E:b:t:")) != -1)
	{
		switch(ch)
		{
			case 'v':
	            isVerbose = true;
	            break;
	        case 's':
	            checkOptarg(optarg);
	            s = atoi(optarg);
	            break;
	        case 'E':
	            checkOptarg(optarg);
	            E = atoi(optarg);
	            break;
	        case 'b':
	            checkOptarg(optarg);
	            b = atoi(optarg);
	            break;
	        case 't':
	            checkOptarg(optarg);
	            strcpy(tracefileName,optarg);
	            break;
		}
	}

	init_Cache(s,E,b,&cache);

	FILE *tracefile = fopen(tracefileName,"r");

    while(fscanf(tracefile,"%s %x,%d",opt,&addr,&size) != EOF){

    	//忽略所有指令高速缓存访问
        if(strcmp(opt,"I")==0)continue;

        int setBits = getSet(addr,s,b);
        int tagBits = getTag(addr,s,b);
        if(isVerbose==true) printf("%s %x,%d ",opt,addr,size);
     	if(strcmp(opt,"M")==0) {
            modifyData(&cache,addr,size,setBits,tagBits,isVerbose);
        }
        if(strcmp(opt,"S")==0) {
            storeData(&cache,addr,size,setBits,tagBits,isVerbose);
        }      
        if(strcmp(opt,"L")==0) {
            loadData(&cache,addr,size,setBits,tagBits,isVerbose);
        }
        if(isVerbose ==true) printf("\n");
    }
    printSummary(hits,misses,evictions);
    return 0;
}


```
##结果
```
Admin@DESKTOP-4CJQTHR:/mnt/c/Windows/System32/ubuntu/csapp/cachelab-handout$ ./test-csim
                        Your simulator     Reference simulator
Points (s,E,b)    Hits  Misses  Evicts    Hits  Misses  Evicts
     3 (1,1,1)       9       8       6       9       8       6  traces/yi2.trace
     3 (4,2,4)       4       5       2       4       5       2  traces/yi.trace
     3 (2,1,4)       2       3       1       2       3       1  traces/dave.trace
     3 (2,1,3)     167      71      67     167      71      67  traces/trans.trace
     3 (2,2,3)     201      37      29     201      37      29  traces/trans.trace
     3 (2,4,3)     212      26      10     212      26      10  traces/trans.trace
     3 (5,1,5)     231       7       0     231       7       0  traces/trans.trace
     6 (5,1,5)  265189   21775   21743  265189   21775   21743  traces/long.trace
    27

TEST_CSIM_RESULTS=27
```
##参考博客

https://blog.csdn.net/u012336567/article/details/51899136

http://wdxtub.com/2016/04/16/thin-csapp-3/

http://wdxtub.com/2016/04/16/thick-csapp-lab-4/

https://www.cnblogs.com/liqiuhao/p/8026100.html?utm_source=debugrun&utm_medium=referral