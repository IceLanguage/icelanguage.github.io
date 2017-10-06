---
layout: page
title: 深入理解不同的二分查找C++
category: 
    - blogs
---

**二分查找（版本A）**
---------
以任一元素S[mid] = x为界，都可将区间分为三部分，且根据此时的有序性必有：
S[low, mid) <= S[mid]<=  S(mid, high)
![这里写图片描述](http://img.blog.csdn.net/20171006182815612?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQyNDQzMTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
~~~cpp
// 二分查找算法（版本A）：在有序向量区间[low, high)内查找元素e，0 <= low<high<size

while (low < high) { //每步迭代可能要做两次比较判断，有三个分支
	int mid = (low + high) >> 1; //以中点为轴点
	if (e < A[mid]) high = mid; //深入前半段[low, mid)继续查找
	else if (A[mid] < e) low = mid + 1; //深入后半段(mid, high)继续查找
	else return mid; //在mid处命中
	} //成功查找可以提前终止
	return -1; //查找失败
}
~~~
只需将目标元素e与x做一比较，即可视比较结果分三种情况做进一步处理：
（1）若e 《x，则目标元素如果存在，必属于左侧子区间S[low, mid)，故可深入其中继续查找；
（2）若x < e，则目标元素如果存在，必属于右侧子区间S(mid, high)，故也可深入其中继续查找；
（3）若e = x，则意味着已经在此处命中，故查找随即终止

**Fibonacci 查找**
------------
![这里写图片描述](http://img.blog.csdn.net/20171006182903794?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQyNDQzMTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
这个方法就是在之前方法基础上把取中点换成了取黄金分割点，算法效率比之前的更高，不详细介绍。

**二分查找（版本B ）**
----------

与二分查找算法的版本A基本类似。不同之处是，在每个切分点A[mid]
处，仅做一次元素比较。具体地，若目标元素小于A[mid]，则深入前端子向量A[low, mid)继续查
找；否则，深入后端子向量A[mid, high)继续查找
![这里写图片描述](http://img.blog.csdn.net/20171006183035065?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQyNDQzMTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

~~~lua
while (1 < high - low) { //迭代仅需做一次比较刞断，有两个分支；成功查找到能提前终止
	int mid = (low + high) >> 1; //以中点为轴点
	(e < A[mid]) ? high = mid : low = mid; //经比较后确定深入[low, mid)或[mid, high)
	} //出口时high = low + 1，查找区间仅含一个元素A[low]
	return (e == A[low]) ? low : -1 ; //查找成功时迒回对应秩；否则统一返回-1
}
~~~
**优点：从三分支到两分支，进一步提高了算法效率**

**二分查找（版本C ）**
----------
![这里写图片描述](http://img.blog.csdn.net/20171006184026918?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQyNDQzMTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
每次转入后端分支时，子向量的左边界取作mid + 1而不是mid
循环结束之后，无论成功与否，只需返回low - 1即可
效率进一步提高
~~~java
while (low < high) { //迭代仅需做一次比较判断，有两个分支
	int mid = (low + high) >> 1; //以中点为轴点
	(e < A[mid]) ? high = mid : low = mid + 1; //经比较后确定深入[low, mid)或(mid, high)
	} //成功查找能提前终止
	return --low; //循环结束时，low为大于e的元素的最小秩，故low - 1即大于e元素的最大
} 
~~~