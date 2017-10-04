程序不当之处，欢迎指正

```
// HeapSort.cpp: 定义控制台应用程序的入口点。
#include "stdafx.h"
#include <vector>
#include <algorithm>  //swap;
#include <iostream>
using namespace std;

class Heap
{
	private:
		
		int size;
		void Max_heapify(vector<int> &A, int i,int s) {
			int l = Left(i);
			int r = Right(i);
			//找到比父节点大的子节点;
			int largest=i;
			if (l < s&&A[l] > A[i]) largest = l;
			if (r < s&&A[r] > A[largest]) largest = r;
			if (largest!= i)
			{
				swap(A[i],A[largest]);
				Max_heapify(A, largest,s);//防止子节点违反最大堆的性质;
			}

			
		}//维护堆的性质;
		
		void Build_Heap(){
			for (int i = size / 2 - 1; i >= 0; i--)
			{
				Max_heapify(heap, i,size);
			}
		}//构建堆;
		
	public:
		vector<int> heap;
		Heap(vector<int> array)
		{			
			heap.assign(array.begin(), array.end());
			size = array.size();
			Build_Heap();
		}
		int Parent(int index)//父节点;
		{
			if (index == 0) return 0;
			else return (index - 1) / 2;
		}
		int Left(int index)//左节点;
		{
			return 2 * index + 1;
		}
		int Right(int index)//右节点;
		{
			return Left(index) + 1;
		}
		int HeapSize()
		{
			return size;
		}

		//优先队列;
		int MaxIMum()//返回最大值;
		{
			return heap[0];
		}
		int Extract_Max()//去除最大值并返回最大值;
		{
			if (size< 1) printf("heap underflow");
			int max = heap[0];
			heap[0] = heap[size-1];
			
			size--;
			heap.pop_back();
			Max_heapify(heap,0,size);
			return max;
			
		}
		void InsertKey(int x,int key)//x索引值增加到key(key<x处值);
		{
			if (key < heap[x]) printf("new key is smaller than current key");
			else
			{
				heap[x] = key;
				while (x > 0 && heap[Parent(x)] < heap[x])
				{
					swap(heap[x],heap[Parent(x)]);
					x = Parent(x);
				}
			}
			
		}
		void Insert(int k)//插入值;
		{
			size++;
			heap.push_back(INT_MIN);
			InsertKey(size - 1, k);
		}

		vector<int> Heap_Sort()//堆排序;
		{
			vector<int> A;//存储排序的数组;
			A.assign(heap.begin(), heap.end());;
			int s = size;
			for (int i = s-1; i >0; i--)
			{
				swap(A[0], A[i]);
				s--;

				Max_heapify(A,0,s);
			}
			return A;//返回从小到大排序好的数组;
		}
		
};

int main()
{
	vector<int> Test{ 16,14,10,8,7,9,3,2,4,1 };
	Heap newHeap =Heap(Test);
	//测试节点的获取;
	cout << "Parent " << newHeap.Parent(3)<<endl;//2
	cout << "Left " << newHeap.Left(3) << endl;//7
	cout << "Right " << newHeap.Right(3) << endl;//8

	//测试堆性质成功维护;
	for each (int a in newHeap.heap)
	{
		cout << a << " " ;
	}
	cout << endl;

	//测试排序结果;
	vector<int> B = newHeap.Heap_Sort();
	for each (int a in B)
	{
		cout << a << " ";
	}
	cout << endl;

	//测试sort不影响原排序;
	for each (int a in newHeap.heap)
	{
		cout << a << " ";
	}
	cout << endl;

	//测试最大值;
	cout << "Max " << newHeap.MaxIMum() << endl;

	//测试Extract_Max();
	cout << newHeap.Extract_Max() << endl;
	for each (int a in newHeap.heap)
	{
		cout << a << " ";
	}
	cout << endl;

	//测试InsertKey;
	newHeap.InsertKey(5, 100);
	for each (int a in newHeap.heap)
	{
		cout << a << " ";
	}
	cout << endl;

	newHeap.Insert(34);
	for each (int a in newHeap.heap)
	{
		cout << a << " ";
	}
	cout << endl;
	while(true){}
    return 0;
}


```