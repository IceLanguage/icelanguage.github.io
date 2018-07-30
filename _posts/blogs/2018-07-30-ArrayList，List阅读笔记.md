---
layout: page
title: ArrayList和List源码阅读笔记
category: 
    - blogs


---
阅读ArrayList和List源码，在这里做个笔记

## ArrayList

ArrayList是C#早期的动态数组，但因为装箱拆箱带来的性能损耗，不推荐使用，不过他的源码还是值得一读的

ArrayList的源码看起来很长，但其核心实现代码非常简单，内部维护一个Object[]的数组和_size变量，内部实现基本都基于Array类实现。其实没啥干货，都是频繁调用Array.Copy之类的

不过有一个函数要着重注意，那就是EnsureCapacity

```C#
 private void EnsureCapacity(int min) {
            if (_items.Length < min) {
                int newCapacity = _items.Length == 0? _defaultCapacity: _items.Length * 2;
                if ((uint)newCapacity > Array.MaxArrayLength) newCapacity = Array.MaxArrayLength;
                if (newCapacity < min) newCapacity = min;
                Capacity = newCapacity;
            }
 }
public ArrayList(int capacity) {
             if (capacity < 0) throw new ArgumentOutOfRangeException("capacity", Environment.GetResourceString("ArgumentOutOfRange_MustBeNonNegNum", "capacity"));
             if (capacity == 0)
                 _items = emptyArray;
             else
                 _items = new Object[capacity];
        }
```

EnsureCapacity是ArrayList的扩容函数，当Capacity(也就是内部数组_items.Length)不够大时，Capacity会扩容为原来的2倍，而Capacity被修改时不可避免要重新new一个数组并拷贝旧数组的数据，所以ArrayList在插入大规模数据前最好在构造函数里预设ArrayList的容量，避免频繁开辟内存导致性能损耗以及过长的程序执行时间，如果构造函数没有提供容量信息，内部Object[]数组会是空数组new Object[0] （不是new Object[_defaultCapacity]）



## List

List估计是使用频率最高的集合了

List的实现和ArrayLis实现思想基本，不过采用泛型避免了装箱拆箱的问题