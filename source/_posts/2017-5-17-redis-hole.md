---
layout: post
title: redis中遇到的坑
date: 2014-05-17 00:00
tags: [redis]
---

## 我们先列一下redis的几种数据类型
- String
- Hash
- List
- Set
- Sorted Set

## 那我们说一下这几种数据类型能做什么？
- String
  在redis中k-v的方式存储，查询O(1)，一般用于缓存，常用命令 GET SET
- Hash
  在redis中k-hash的方式，查询O(1)，常用命令 HGET HSET HSETALL
- List
  可用做队列、栈，常用命令：Lrange lpush lpop rpush rpop
- Set
  排重 求交集、并集、差集等操作 常用命令：sadd,spop,smembers,sunion
- Sorted Set
  是 Set的有序集合

## 那我们遇到哪些坑了呢？
首先 redis 是单线程的意味着任何操作只能使用一个cpu, 但是redis提供了一些操作是比较耗CPU的，例如我们求keys "aa*"的时候这时候是比较耗cpu的，假如key非常多直接回阻塞后面的所有操作，还有Set的交集 并集等操作都是非常耗cpu的，还有redis会支持一些事件例如过期删除添加等操作都会有对应的事件但是redis为了性能优化都是lasy懒惰算法的

## 总结
不要盲目的使用redis