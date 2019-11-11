---
title: 哪一个 List 实现了快插入
author: Juntech
top: false
cover: false
toc: true
mathjax: false
copyright: true
summary: 哪一个 List 实现了快插入
categories: java
tags: List
keywords: List
abbrlink: 7c8a1cbd
date: 2019-09-11 15:29:31
---

# 哪一个 List 实现了快插入?

## 1、哪一个 List 实现了快插入?

答：
LinkedList 和 ArrayList 是另个不同变量列表的实现。
ArrayList 的优势在于动态的增长数组，非常适合初始时总长度未知的情况下使用。
LinkedList 的优势在于在中间位置插入和删除操作，速度是快的。
LinkedList 实现了 List 接口，允许 null 元素。
此外 LinkedList 提供额外的 get，remove，insert 方法在 LinkedList 的首部或尾部。
这些操作使 LinkedList 可被用作堆栈 (stack)，队列 (queue) 或双向队列 (deque)。
ArrayList 实现了可变大小的数组。它允许所有元素，包括 null。
每个 ArrayList 实例都有一个容量(Capacity)，即用于存储元素的数组的大小。
这个容量可随着不断添加新元素而自动增加，但是增长算法并没有定义。
当需要插入大量元素时，在插入前可以调用 ensureCapacity 方法来增加 ArrayList 的容量以提高插入效率。

## 2、什么时候使用 ConcurrentHashMap?

答：
ConcurrentHashMap 被作为故障安全迭代器的一个实例，它允许完整的并发检索和更新。
当有大量的并发更新时，ConcurrentHashMap 此时可以被使用。
这非常类似于 Hashtable，但 ConcurrentHashMap 不锁定整个表来提供并发，所以从这点上 ConcurrentHashMap 的性能似乎更好一些。
所以当有大量更新时 ConcurrentHashMap 应该被使用。
Java企业面试题“哪一个 List 实现了快插入?”；“什么时候使用 ConcurrentHashMap?”希望对面试的小伙伴有所帮助
