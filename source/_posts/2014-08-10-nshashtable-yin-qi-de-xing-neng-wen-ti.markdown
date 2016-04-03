---
layout: post
title: "NSHashTable 引起的性能问题"
date: 2014-08-10 01:46:40 +0800
comments: true
categories: iOS
---

  本文介绍了在 Core Text 排版中，往 NSAttributeString 增加一个属性时引起的性能问题。
  
<!-- more -->

  在 Feeds 的 Core Text 排版中，NSAttributeString 中的某一小段的点击跳转行为是存储在 NSAttributeString 的属性中的。
  
代码如下：

  ![image](/images/2014-08-10-1-1.png)

  QZTextLinkHelper 存储了点击跳转的相关信息，比如 url 跳转，昵称跳转等。
  
  setLinkHelper 做为 NSAttributeString 的扩展方法，通过 [NSAttributeString addAttribute:value:range] 方法存储 linkHelper 的属性。
  
  这里，linkHelper 的 linkAttributes 是一个 NSDictionary 对象。

  好，问题出来了。我们发现，Feeds 排版变得异常耗时，滑动卡顿，profile 一下之后发现一个奇怪的热点：

  ![image](/images/2014-08-10-1-2.jpeg)

  [NSAttributeString addAttribute:value:range] 里面耗时很多，仔细看调用栈发现，内部的 attribute 都是通过 NSHashTable 存储的。
  
  堆栈中出现的 [NSHashTable member:] 是取值操作，耗时主要在最后的比较两个 Dictionary 是否相等上面。
  
  我们先重温一下哈希表取值的原理，见下图，红色部分代表耗时很长的步骤。

  ![image](/images/2014-08-10-1-3.png)

  根据上图，我们可以看出，当哈希表频繁对比两个对象是否相等的时候，说明哈希表的键冲突已经非常严重了。
  
  于是转而把目光转向 NSDictionary 是怎么计算自己的哈希值的。
  
  写了一个 demo 试了一下，发现 NSDictionary 的哈希值等于其 key 的个数，非常简单的一个计算，也非常容易冲突。

  于是，解决方法就是在 addAttribute 的时候直接塞入 linkHelper 就可以了。
  因为NSObject的哈希值默认是指针地址。
  
总结：
  
  [NSAttributeString addAttribute:value:range] 这个方法如果传入一个 NSDictionary 会非常耗时，原因有两个：
  
  1. NSAttributeString 以 HashTable 的方式存储 attribute，这使 attribute 的存取变得很复杂，我们必须确保传入的自定义 attribute 的哈希值不易冲突，或者保证其 isEqual 方法的效率。
  
  2. NSDictionary 的哈希值计算太过于简单了，基本不能跟 NSHashTable 这种容器共存。

附：调用 [NSHashTable addObject:] 传入不同对象时的性能对比。
                                               
10000个NSNumber：2ms

10000个相同的NSDictionary：33000ms

10000个key数量各不相同的NSDictionary：400ms

