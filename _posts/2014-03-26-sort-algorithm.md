---
layout: post
title:  "sort algorithm"
date:   2014-03-26 16:07:49
categories: algorithm sort
---

排序算法最简单的算法是比较排序和冒泡排序，他们的实现比较简单，但是时间复杂度都是`O(n^2)`，因此在大多数情况下不是理想的排序算法，下面将介绍几种复杂度为`O(nlgn)`的排序算法。

##快速排序

快速排序在平均情况下是排序效率最好的一种算法，它的核心算法如下：

```cpp
int*  Partition(int* begin_ptr,int* end_ptr) 
{
	int* last_ptr = end_ptr-1;
	int* less_ptr = begin_ptr-1;
	int* greater_ptr = begin_ptr;
	for(;greater_ptr<last_ptr;++greater_ptr) {		//iterate from the begin element to the end 
		if(*greater_ptr<*last_ptr) {		//find a value less than the last value
			++less_ptr;
			std::iter_swap(less_ptr,greater_ptr);	//swap the less value
		}
	}
	std::iter_swap(last_ptr,less_ptr+1);
	return less_ptr+1;
}

void  QuickSort(int* begin_ptr,int* end_ptr)
{
	if(end_ptr <= begin_ptr)
		return ;
	int* sort_ptr = Partition(begin_ptr,end_ptr);
	QuickSort(begin_ptr,sort_ptr);
	QuickSort(sort_ptr+1,end_ptr);
}
```

上述代码的核心是Partition，它将`less_ptr+1`作为主元素将`[begin_ptr,end_ptr)`序列分成两部分，使得左边的元素都小于主元素，右边的元素都大于主元素。经过Partition之后，主元素就已经排好序了，然后对左边的序列和右边的序列进行递归排序，就能确保整个序列都有序。

Partition的主要思想可以用下图来表示：

![快速排序](/images/algorithm/sort/quick_sort.jpg)

上图中浅绿色(最后一个元素)就是主元素，序列中其他元素都跟他进行比较，浅红色序列是小于主元素的序列，青色背景序列式大于主元素的序列，greater指针一直向前探索，当遇到比主元素小的元素时，就会将less指针加一，然后交换greater和less的值，这样就能确保`[0,less]`为小于主元素的序列，`(less,greater]`为大于主元素的序列。

当比较完整个序列时，交换`less+1`和主元素，这样主元素之前的元素都是小于主元素，之后的元素都大于主元素，这样就成功将`[begin,end)`分成两个部分，并使得`less+1`指向的元素有序。

###快速排序的应用

* 输入一个整数数组，要求对数组进行重新排列，使得所有奇数位于数组的前半部分，所有偶数位于数组的后半部分

	这个问题可以用Partition一样的方法来求解，我们设定两个指针odd和even，令[0,odd]为奇数序列，(odd,even]为偶数序列，让even指针遍历整个数组，当遍历到奇数时，奇数序列加1，并且将奇数交换到odd指向的位置，使得依旧满足[0,odd]为奇数序列，(odd,even]为偶数序列。

* 在一个数组中，找出第k小的元素

	一般问题一般思路是将数组进行重新排序，然后直接获得第k小的元素，但是复杂度至少是`O(nlgn)`，有没有更好的办法呢？答案就是Partition，Partition每次能获得第`less+1`小的元素，我们可以根据k与less的大小在`[0,less]`或者`(less,end)`继续进行Partition。灿口源码如下：

	```cpp
	int  GetKElement(int* begin_ptr,int* end_ptr,int k)
	{
		int* sort_ptr = Partition(begin_ptr,end_ptr);
		int count = sort_ptr-begin_ptr+1;
		if( count == k) {		//find the number
			return *sort_ptr;
		} else if( count > k) {
			return GetKElement(begin_ptr,sort_ptr,k);
		} else {
			return GetKElement(sort_ptr+1,end_ptr,k-count);
		}
	}
	```


##归并排序

归并排序是用分而治之的方式来排序的。对一个大的数组(长度为n)排序，我们可以先将两个小的数组(长度为n/2)进行排序，然后将这两个部分数组合并，就得到最终的排序数组。它的算法复杂度很稳定，为`O(nlgn)`。参考源码如下：

```cpp
void  MergeSort(int* begin_ptr,int* end_ptr)
{
	if(begin_ptr<=end_ptr)
		return ;
	int* mid_ptr = begin_ptr+ (end_ptr-begin_ptr)/2;
	MergeSort(begin_ptr,mid_ptr);
	MergeSort(mid_ptr,end_ptr);

	int * copy_array = new int[end_ptr-begin_ptr];
	int * first_ptr = begin_ptr;
	int * second_ptr = mid_ptr;
	int * copy_ptr = copy_array;
	while(first_ptr<mid_ptr&&second_ptr<end_ptr) {
		if(*first_ptr<*second_ptr) {
			*copy_ptr = *first_ptr++;
		} else {
			*copy_ptr = *second_ptr++;
		}
		++copy_ptr;
	}
	while(first_ptr<mid_ptr) {
		*copy_ptr = *first_ptr++;
		++copy_ptr;
	}
	std::copy(copy_array,copy_ptr,begin_ptr);
	delete copy_array;
}
```

上述源码中，首先对两个子数组进行排序，然后合并这两个子数组。首先分配一个用来存放合并数组的区域`copy_array`，然后将两个子数组按照从小到大进行合并，然后将`copy_array`复制到原数组，完成合并。

###归并排序的应用

* 求逆序对
>在数组中的两个数字，如果前面的数字大于后面的数字，则这两个数字组成逆序对。输入一个数组，求数组中逆序对的总数。

这个问题的最简单方法就是让数组中每个数字与数组中的其他数字进行比较，找出其中的逆序对，但是时间复杂度为`O(n^2)`，算法效率较低。

这个时候我们可以借鉴归并排序的方法，将问题一分为二，求出每个子数组中的逆序对，然后对两个子数组进行合并，求出总的逆序对即可。如果只是单纯的对两个子数组求逆序对，那么时间复杂度依旧为`O(n^2)`，这不是我们想要的。但是如果我们将子数组进行排序，那么会怎样呢？

如果我们对子数组进行排序，那么当比较两个子数组时，我们不必每个元素都进行比较，因为它们是有序的。这样只要`O(n)`即可完成子数组的合并和逆序对的求取。具体求解过程与归并排序相似。
