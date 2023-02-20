---
title: 数据结构-链表
date: 2019-07-17 15:34:02.0
updated: 2022-05-03 13:53:34.826
url: https://maoxian.fun/archives/shu-ju-jie-gou-lian-biao
categories: 
- 程序
- 数据结构-算法
tags: 
- 程序
- 代码
- 数据结构
---

链表是线性表的一种，是一种基础的数据结构，相对于数组更加的灵活。本文以**单链表**为例、用 **C++** 语言描述介绍链表的原理与实现。

## 数组

在讨论链表之前，先来看一下另一种线性表——数组。数组是储存在一块连续分配的内存中的，通过对元素下标、元素类型和数组首地址的运算，我们可以很方便的得到元素在内存中的地址。以一维数组为例，数组中的每个元素都是按照顺序依次储存在一块连续分配的内存中。

![数组在内存中的分配](./assets/img/Memory.jpg?x-oss-process=style/mxcompress)

数组由于是分配在连续的内存空间上，在随机读取方面具有极大的优势，但是在程序中数组的大小是固定的，一旦定义了一个数组，就不能对其大小进行调整。所以我们在设定时往往以最大的需求来定义数组，而有些时候这会造成一些空间上的浪费。

## 链表

链表是一种灵活的数据结构，在内存上是不连续的。

### 结构

链表中的每个元素称为节点Node。节点在内存中是随机分布的，每个节点分为两个部分，存放数据的数据域和保存地址的指针域。

![img](./assets/img/Node-cf7df3e42588b5b990f72608ca0fd257-cd4fc8-1610089027.jpeg?x-oss-process=style/mxcompress)

单链表由一个头节点Node *head（指向第一个节点的地址）和各节点组成，从头节点开始，通过指针将每个节点依次连接起来，最后一个指针置空（指向nullptr）。

![img](./assets/img/delete-d9814b52ffbb9a97cf928c54c7395c43-ca62e2-1610089087.jpeg?x-oss-process=style/mxcompress)

在单链表中，每个节点除了本身的数据之外保存的还有下一个节点的地址，对链表的访问也只能从头节点出发，直到找到目标。节点的储存是分散的，只是通过指针将每个节点串了起来，形成一个完整的链，从而提高了数据储存的灵活性。缺点则是当要访问某一元素的时候，必须从头开始遍历，时间复杂度为O(n)。

### 节点的插入与删除

链表中每个节点通过指针连接，因此，插入与删除节点的时候只要修改对应指针的指向。

![Insert](./assets/img/Insert.jpg?x-oss-process=style/mxcompress)

插入节点时，首先将要插入的野生节点的指针指向要插入的后一个节点（图中节点2），再将前一个节点的指针指向野生节点。

![delete](./assets/img/delete.jpg?x-oss-process=style/mxcompress)

删除节点与插入类似，只要将前一节点（head）的指针指向被删除后一节点（节点2），同时别忘了将节点1的内存空间回收。

### 分类

单链表是链表中最简单的一个类型，链表还有双链表、循环链表。

![doubleNode](./assets/img/doubleNode.jpg?x-oss-process=style/mxcompress)

双链表是在单链表的基础上，在节点的指针域中增加一个指针*prevNode(previous node)指向前一个节点，实现对链表的双向访问。

![cycleList](./assets/img/cycleList.jpg?x-oss-process=style/mxcompress)

循环链表是在单/双链表的基础上，将尾节点的*nextNode指针指向头节点，使链表形成一个环状结构。