---
layout: page
title: hashtable源码解析
category: 
    - blogs


---
Hashtable也就是哈希表，是个非常重要的概率，在剖析hashtable源码前，我先简单介绍一下hashtable的原理

## 哈希表概念

**什么是哈希**（hash又称散列）？

将任意长度的消息压缩到某一固定长度的消息摘要的函数

***什么是哈希表***？

给定一张表，通过哈希函数F(key)能将键值转化成表中的一个地址，便是哈希表

**什么是哈希函数**？

能将键值转换为哈希表范围内的索引（0~M-1） 

**什么是哈希冲突**？

利用哈希函数计算地址时，不同的key值计算出了一样的结果，这种现象称为哈希冲突

**怎么解决哈希冲突**？

1.  开放定址法 ：

    ​	以发生哈希冲突的哈希地址为自变量，通过某种哈希函数得到一个新的空闲内存单元地址的方法 

2.  再哈希法 ：

    ​	发生冲突时，使用第二个、第三个、哈希函数计算地址，直到无冲突时 

3.  链地址法 ：

    ​	把产生冲突的数据元素另外存放在单链表中 



## HashHelpers

接下来我们开始读源码，尤其hashtable的源码经常使用HashHelpers这个辅助类，我先对HashHelpers类的几个关键方法进行介绍

```C#
public static readonly int[] primes = {
            3, 7, 11, 17, 23, 29, 37, 47, 59, 71, 89, 107, 131, 163, 197, 239, 293, 353, 431, 521, 631, 761, 919,
            1103, 1327, 1597, 1931, 2333, 2801, 3371, 4049, 4861, 5839, 7013, 8419, 10103, 12143, 14591,
            17519, 21023, 25229, 30293, 36353, 43627, 52361, 62851, 75431, 90523, 108631, 130363, 156437,
            187751, 225307, 270371, 324449, 389357, 467237, 560689, 672827, 807403, 968897, 1162687, 1395263,
            1674319, 2009191, 2411033, 2893249, 3471899, 4166287, 4999559, 5999471, 7199369};
```

这是一张素数表，共72个元素，所有元素都是符合f(n) =1 +2n(n=1,2,….)的素数



```C#
public static bool IsPrime(int candidate) 
       {
           if ((candidate & 1) != 0) 
           {
               int limit = (int)Math.Sqrt (candidate);
               for (int divisor = 3; divisor <= limit; divisor+=2)
               {
                   if ((candidate % divisor) == 0)
                       return false;
               }
               return true;
           }
           return (candidate == 2);
       }
```

IsPrime是用于判断candidate参数是否是素数



```C#
public static int GetPrime(int min) 
        {
            if (min < 0)
                throw new ArgumentException(Environment.GetResourceString("Arg_HTCapacityOverflow"));
 
            for (int i = 0; i < primes.Length; i++) 
            {
                int prime = primes[i];
                if (prime >= min) return prime;
            }

            for (int i = (min | 1); i < Int32.MaxValue;i+=2) 
            {
                if (IsPrime(i) && ((i - 1) % 101 != 0))
                    return i;
            }
            return min;
        }
```

GetPrime是用于获得一个素数，如果min小于预定表的最大数7199369就直接返回素数表中大于min数中最小的一个，如果超出预定表的最大数7199369，就按另一种复杂的方法判断。

不过这一种方法数学原理博主表示不能理解，不知道有哪位大神解释一下

## hashtable

接下来进入主题，我们来先从hashtable的构造函数开始

### 构造原理

hashtable的构造函数需要2个参数，一个是容量，一个是装载因子。

#### 装载因子是什么

装载因子哈希表是0.1 到 1.0 范围内的数字 ，是存储桶数（count）所占桶数组（buckets）桶数（hashsize）的最大比率 ，当桶数大于装载数（loadsize）时，桶数组就会扩容。

装载因子默认为1.0，但实际上微软将它乘以0.72,为什么是0.72呢，因为桶数组的过小 会导致空间的浪费，过大会导致哈希冲突的频繁发生，0.72是微软综合效率和内存损耗考虑，经过一番实验，取的一个比较均衡的结果

#### 桶和桶数组

至于什么是桶和桶数组，看下面这段代码你就懂了，正是bucket桶这个类型存储hashtable的hash信息。至于桶数组存储了所有hashtable

```c#
private struct bucket {
            public Object key;
            public Object val;
            public int hash_coll;  
        }
    
private bucket[] buckets;
```

#### 构造

```C#
public Hashtable(int capacity) : this(capacity, 1.0f) {
        }
public Hashtable(int capacity, float loadFactor) {
           if (capacity < 0)
               throw new ArgumentOutOfRangeException("capacity", Environment.GetResourceString("ArgumentOutOfRange_NeedNonNegNum"));
           if (!(loadFactor >= 0.1f && loadFactor <= 1.0f))
               throw new ArgumentOutOfRangeException("loadFactor", Environment.GetResourceString("ArgumentOutOfRange_HashtableLoadFactor", .1, 1.0));
    
           this.loadFactor = 0.72f * loadFactor;
 
           double rawsize = capacity / this.loadFactor;
           if (rawsize > Int32.MaxValue)
               throw new ArgumentException(Environment.GetResourceString("Arg_HTCapacityOverflow"));
 
           int hashsize = (rawsize > InitialSize) ? HashHelpers.GetPrime((int)rawsize) : InitialSize;
           buckets = new bucket[hashsize];
 
           loadsize = (int)(this.loadFactor * hashsize);
           isWriterInProgress = false;

       }
```

hashtable内部的构造函数内部主要做了什么呢

它利用 容量 /装载因子获取了一个桶数组大小的估值，借助HashHelpers.GetPrime获取了一个大于等于3的素数作为内部桶数组的初始长度，并以此长度初始桶数组

### 哈希算法函数

hashtable内部采用的是双重散列法，是开放地址法中最好的方法之一 

```C#
private uint InitHash(Object key, int hashsize, out uint seed, out uint incr) {      
       uint hashcode = (uint) GetHash(key) & 0x7FFFFFFF;//取整数
       seed = (uint) hashcode;
 
       incr = (uint)(1 + ((seed * HashPrime) % ((uint)hashsize - 1)));
       return hashcode;
   }
```
我们可以看到函数利用out 关键词引用传递了seek和incr2个变量，seek就是key本身的哈希值，incr则是第二个哈希函数的增量

#### 双重散列法简单介绍

就是有2个哈希函数，第一个哈希函数不行就用第二个，第二个就是之前计算的地址加上增量再mod m，再不行就继续用第二个哈希函数计算

#### 双重散列法算法的使用

关于算法解释得可能不是很清楚，直接用算法的使用让大家更深刻的理解这个算法

```C#
public virtual bool ContainsKey(Object key) {
            if (key == null) {
                throw new ArgumentNullException("key", Environment.GetResourceString("ArgumentNull_Key"));
            }
 
            uint seed;
            uint incr;

            bucket[] lbuckets = buckets;
            uint hashcode = InitHash(key, lbuckets.Length, out seed, out incr);
            int  ntry = 0;
            
            bucket b;
            int bucketNumber = (int) (seed % (uint)lbuckets.Length);            
            do {
                b = lbuckets[bucketNumber];
                if (b.key == null) {
                    return false;
                }
                if (((b.hash_coll &  0x7FFFFFFF) == hashcode) && 
                    KeyEquals (b.key, key))
                    return true;
                bucketNumber = (int) (((long)bucketNumber + incr)% (uint)lbuckets.Length);                              
            } while (b.hash_coll < 0 && ++ntry < lbuckets.Length);
            return false;//哈希冲突数超出桶数，循环终止
        }
```

这是hashtable的ContainsKey方法，它完全是双重散列法算法的使用

利用第一个哈希函数计算bucketNumber

如果bucketNumber发现是冲突的地址，就利用第二个哈希函数继续计算地址，如果一直失败就持续为上次的·计算结果加上增量，至于% (uint)lbuckets.Length)是为将计算后的地址限制在哈希表的地址范围

### 元素添加和哈希冲突

```C#
private void putEntry (bucket[] newBuckets, Object key, Object nvalue, int hashcode)
{
    uint seed = (uint) hashcode;
    uint incr = (uint)(1 + ((seed * HashPrime) % ((uint)newBuckets.Length - 1)));
    int bucketNumber = (int) (seed % (uint)newBuckets.Length);            
    do {
        if ((newBuckets[bucketNumber].key == null) || (newBuckets[bucketNumber].key == buckets)) {
            newBuckets[bucketNumber].val = nvalue;
            newBuckets[bucketNumber].key = key;
            newBuckets[bucketNumber].hash_coll |= hashcode;
            return;
        }

        if( newBuckets[bucketNumber].hash_coll >= 0 ) {
            newBuckets[bucketNumber].hash_coll |= unchecked((int)0x80000000);
            occupancy++;
        }
        bucketNumber = (int) (((long)bucketNumber + incr)% (uint)newBuckets.Length);                
    } while (true);
}
```
这段代码是哈希表内部添加元素的代码，在哈希表利用key值查找地址，如果冲突，则occupancy自增，且利用双重散列法寻找下一个地址，不冲突则为桶数组对于地址的空桶赋值

### 哈希表的扩容

下面这段代码是哈希表的扩容函数，建立新的的桶数组，我们可以看到这段扩容代码在拷贝旧数据时不是简单的复制粘贴,而是利用putEntry重新添加元素，这般操作固然能保证之后哈希存取的效率，但大规模的putEntry终究是高时耗的，程序员使用hashtable时最好选择一个比较大的容量，避免重复扩容

```C#
private void rehash( int newsize, bool forceNewHashCode ) {
 
            occupancy=0;
                  
            bucket[] newBuckets = new bucket[newsize];
    
            int nb;
            for (nb = 0; nb < buckets.Length; nb++){
                bucket oldb = buckets[nb];
                if ((oldb.key != null) && (oldb.key != buckets)) {
                    int hashcode = ((forceNewHashCode ? GetHash(oldb.key) : oldb.hash_coll) & 0x7FFFFFFF);                              
                    putEntry(newBuckets, oldb.key, oldb.val, hashcode);
                }
            }

            isWriterInProgress = true;
            buckets = newBuckets;
            loadsize = (int)(loadFactor * newsize);
            UpdateVersion();
            isWriterInProgress = false;
               
            return;
        }

```

### 插入条目

分析完hashtable的内部函数，我们再来看hashtable真正的插入元素操作

```C#
 private void Insert (Object key, Object nvalue, bool add) {

            if (key == null) {
                throw new ArgumentNullException("key", Environment.GetResourceString("ArgumentNull_Key"));
            }

            if (count >= loadsize) {
                expand();
            }
            else if(occupancy > loadsize && count > 100) {
                rehash();
            }
            
            uint seed;
            uint incr;

            uint hashcode = InitHash(key, buckets.Length, out seed, out incr);
            int  ntry = 0;
            int emptySlotNumber = -1;
     
            int bucketNumber = (int) (seed % (uint)buckets.Length);
            do {
 
              
                if (emptySlotNumber == -1 && (buckets[bucketNumber].key == buckets) && (buckets[bucketNumber].hash_coll < 0))
                    emptySlotNumber = bucketNumber;
 
               
                if ((buckets[bucketNumber].key == null) || 
                    (buckets[bucketNumber].key == buckets && ((buckets[bucketNumber].hash_coll & unchecked(0x80000000))==0))) {
 
                  
                    if (emptySlotNumber != -1)
                        bucketNumber = emptySlotNumber;

       
                    isWriterInProgress = true;                    
                    buckets[bucketNumber].val = nvalue;
                    buckets[bucketNumber].key  = key;
                    buckets[bucketNumber].hash_coll |= (int) hashcode;
                    count++;
                    UpdateVersion();
                    isWriterInProgress = false;   
                  
 

                    return;
                }
 

                if (((buckets[bucketNumber].hash_coll & 0x7FFFFFFF) == hashcode) && 
                    KeyEquals (buckets[bucketNumber].key, key)) {
                    if (add) {
                        throw new ArgumentException(Environment.GetResourceString("Argument_AddingDuplicate__", buckets[bucketNumber].key, key));
                    }
        
                    isWriterInProgress = true;                    
                    buckets[bucketNumber].val = nvalue;
                    UpdateVersion();                    
                    isWriterInProgress = false; 
               
                    return;
                }
 
                if (emptySlotNumber == -1) {
                    if( buckets[bucketNumber].hash_coll >= 0 ) {
                        buckets[bucketNumber].hash_coll |= unchecked((int)0x80000000);
                        occupancy++;
                    }
                }
 
                bucketNumber = (int) (((long)bucketNumber + incr)% (uint)buckets.Length);               
            } while (++ntry < buckets.Length);
 
            if (emptySlotNumber != -1)
            {
 
                isWriterInProgress = true;                    
                buckets[emptySlotNumber].val = nvalue;
                buckets[emptySlotNumber].key  = key;
                buckets[emptySlotNumber].hash_coll |= (int) hashcode;
                count++;
                UpdateVersion();                
                isWriterInProgress = false;     

                return;
            }
    
            throw new InvalidOperationException(Environment.GetResourceString("InvalidOperation_HashInsertFailed"));
        }
```

Insert的内部操作分三个步骤

1.  判断key是否为空，如果空，哈希函数是无法计算地址的

2.  判断条目数是否大于等于装载数，如果满足，按照之前对装载因子的介绍，哈希表需要扩容为原来的2倍以上；或者判断冲突数是否大于装载数，如果大于，则扩容

3.  接着在循环体内按照双重散列法寻找对应键值的桶并为对应键值的桶赋值

    

### 删除条目

最后是hashtable解析的最后一部分，条目的删除

代码都是千篇一律的，核心就是按照双重散列法寻找对应键值的桶后将桶变成空桶

```C#
 public virtual void Remove(Object key) {
            if (key == null) {
                throw new ArgumentNullException("key", Environment.GetResourceString("ArgumentNull_Key"));
            }

 
            uint seed;
            uint incr;
 
            uint hashcode = InitHash(key, buckets.Length, out seed, out incr);
            int ntry = 0;
            
            bucket b;
            int bn = (int) (seed % (uint)buckets.Length);  r
            do {
                b = buckets[bn];
                if (((b.hash_coll & 0x7FFFFFFF) == hashcode) && 
                    KeyEquals (b.key, key)) {
              
                    isWriterInProgress = true;

                    buckets[bn].hash_coll &= unchecked((int)0x80000000);
                    if (buckets[bn].hash_coll != 0) {
                        buckets[bn].key = buckets;
                    } 
                       else {
                        buckets[bn].key = null;
                    }
                    buckets[bn].val = null;  
                    count--;
                    UpdateVersion();
                    isWriterInProgress = false; 
               
                    return;
                }
                bn = (int) (((long)bn + incr)% (uint)buckets.Length);                               
            } while (b.hash_coll < 0 && ++ntry < buckets.Length);
 
        }
```