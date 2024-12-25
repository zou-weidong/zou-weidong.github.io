---
title: "集合Collection"
linkTitle: "集合Collection"
weight: 3
---

{{% pageinfo %}}
容器主要包括 Collection 和 Map 两种， Collection 存储着对象的集合，而 Map 存储着键值对的映射表。
{{% /pageinfo %}}


## Set

### Treeset
基于红黑树实现，支持有序操作，例如根据一个范围查找元素的操作，查找效率不如 HashSet。
HashSet 查找的时间复杂度为 O(1)，TreeSet 则为O(logN)。

### HashSet
基于哈希表的实现，支持快速查找，但是不支持有序性操作。遍历得到的结果顺序不确定。

## List

### ArrayList
基于动态数组实现，保证顺序支持随机访问。

有一个 capacity，表示底层数据的实际大小。当添加元素，容量不足时，容器会自动增大底层数组的大小。
没有实现同步（synchronized），如果需要多个线程并发访问，需要用户控制，也可以使用 Vector 替代。


### Vector
和 ArrayList 类似，是线程安全的。

### LinkedList

![list](/images/java/list.png)

基于双向链表实现，只能顺序访问，可以快速的在中间插入和删除元素。
不仅如此，LinkedList还可以当 栈 或者 队列（首选ArrayDeque）

解释一下删除remove(int index) 方法：
1. 根据index获取元素， index 与 sise/2 作比较，小于一半的时候从前向后遍历，大于一半的时候从后向前遍历
2. 删除获取的元素，修改前后指针的指向

插入add()方法默认是插入到最后，还可以根据index插入。

clear()方法是将内置的元素引用全部清空，如next,prev，item。


### Queue
          throw exception   return special value
Insert       add(e)                offer(e)
Remove       remove()              poll()
Examine      element()             peek()


### LinkdList
实现双向队列。

queue的几个方法也是常规用法。

> LinkedList list = new LinkedList<String>();  //只有这样写才能使用到queue的方法。


### Deque
双端队列，多一些方法，常规用法。



### PriorityQueue
基于堆结构实现，用来实现优先队列。
优先队列的作用是保证每次取出的元素都是队列中权值最小的，元素大小的评判可以通过元素本身的自然顺序，也可以通过构造时传入的比较器。


## Map

### TreeMap
基于红黑树实现。

### HashMap
基于数组+链表+红黑树实现，线程安全使用 ConcurrentHashMap。

![list](/images/java/hashmap.png)







### LinkedHashMap
使用双向链表来维护元素的顺序，顺序为插入顺序或者最近最少使用（LRU）顺序。








