---
layout: post
title:  "Iterator Invalidation"
date:   2014-03-26 10:28:49
categories: Cpp STL iterator
---

# **迭代器失效规则**
##Insertion
###Sequence containers

* `vector`: all iterators and references before the point of insertion are unaffected, unless the new container size is greater than the previous capacity (in which case all iterators and references are invalidated) 

* `deque`: all iterators and references are invalidated, unless the inserted member is at an end (front or back) of the deque (in which case all iterators are invalidated, but references to elements are unaffected) 
		
* `list`: all iterators and references unaffected 
		
###Associative containers

* `[multi]{set,map}`: all iterators and references unaffected 

###Unsorted associative containers

* `unordered_[multi]{set,map}`: all iterators invalidated when rehashing occurs, but references unaffected . Rehashing does not occur if the insertion does not cause the container's size to exceed z * B where z is the maximum load factor and B the current number of buckets. 

##Erasure
###Sequence containers

* `vector`: every iterator and reference after the point of erase is invalidated 
* `deque`: erasing the last element invalidates only iterators and references to the erased elements and the past-the-end iterator; erasing the first element invalidates only iterators and references to the erased elements; erasing any other elements invalidates all iterators and references (including the past-the-end iterator)
* `list`: only the iterators and references to the erased element is invalidated 

###Associative containers

* `[multi]{set,map}`: only iterators and references to the erased elements are invalidated 

###Unordered associative containers

* `unordered_[multi]{set,map}`: only iterators and references to the erased elements are invalidated

# **失效的原因**
###Sequence Containers
* `vector`: 对于vector来说，它就是一个动态数组，当插入和删除元素时，它会移动后面的元素，因此插入或者删除元素后面的迭代器和引用都会失效。我们知道vector会预先分配比较大的空间来存放元素，但是当插入元素充满空间时，它会引起vector进行重新分配空间，此时所有的迭代器和引用都会失效。

* `deque`：deque是一个双端队列，它的实现一般来说是由一个映射表和一些连续的空间组成(具体请参考《STL源码剖析》)，当插入元素时，可能会引起映射表的重新分配和空间内的元素的移动，因此会导致迭代器失效，当将元素插入到两端(push_front,push_back)且映射表不需要重新分配的情况下，元素不需要移动，因此元素的引用不受影响。

	当删除两端元素时，其他的元素不会移动，因此除了被删除元素，其他元素的迭代器和引用皆不受影响。但是当删除飞两端元素时，会引起元素向前或者向后移动，因此迭代器和引用都会失效。

* `list`：list是一个双链表，对于它的删除和插入元素，只需要改变元素的prev和next指针即可，现有的迭代器和引用并没有改变，因此list在删除或者插入元素之后，除了删除元素之外的迭代器和引用都是有效的。

* `[multi]{set,map}`：它们的本质是一颗红黑树，这种数据结构和双链表一样，对插入和删除不受影响。

* `unordered_[multi]{set,map}`：无序映射表是一个哈希映射表，当插入元素且不会导致重新hash或者删除元素时，迭代器和引用都不会失效。当重新进行hash时，哈希数组会进行重新非配，因此所有迭代器都会失效，但是元素不会重新分配，因此所有引用仍然有效。