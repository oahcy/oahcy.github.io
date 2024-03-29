---
title: skiplist-intro
tags: 
- skiplist
- leveldb
date: 2023-08-31 10:32:30
---


skiplist，即跳表是由William Pugh在1989年发明的，允许快速查询一个有序连续元素的数据链表，搜索、插入、删除的平均时间复杂度均为O(lgn)。

本文介绍下对于skiplist的理解，包括背景、推导过程、伪代码以及复杂度的证明。

# 1.背景

有序数组的好处是可以通过二分实现O(lgn)的高效查找，然而插入元素时，为了保证有序性，时间复杂度是O(n)的。链表则刚好相反，插入数据是O(1)，查找元素则是O(n)的。即使链表数据是有序的，查找元素仍然是O(n)的，因为本质上，链表不支持random access.

那么，是否存在一种链表，既支持高效的数据插入，又可以实现高效的查找？

介绍skiplist前，我们先举一个坐火车的例子。

假设公司组织出去bui，我们居住在城市A，想要到城市H去，其中城市A到H有两趟车：

一趟慢车：

```A -> B -> C -> D -> E -> F -> G -> H -> I -> J -> K -> L -> M -> N```

此外还有一趟快车，经过H的上个城市G

```A -> C -> E -> G -> I -> K -> M```

bui自然要尽快去吃喝玩乐，那么怎么能尽快的到达城市H呢？很直观的，我们会这么选择

- 先乘坐快车 A -> C -> E -> G
- 再乘坐慢车 G -> H

假设把火车的路线想象为一个链表，那么这个链表明显是有序的，每个城市都是链表上一个节点，去到城市H相当于找到H这个节点，可以看到：

**通过乘坐快车，我们能够更快的开始bui**

即

**通过使用数据量更少（子集）的辅助有序链表，我们能够实现更快速的查找**

这就是跳表最朴素的想法(really pretty simple👏 so young so naive.)

{% asset_img analysis_of_two_linked_lists.png  %}

![2023-08-31T105857](2023-08-31T105857.png)