---
title: Swift Highlight 3 ：容器是结构体
date: 2016-04-06 23:37:26
tags:
- iOS
- Swift
- Best Practice
categories: Swift Highlight
---

Swift's `String`, `Array`, and `Dictionary` types are implemented as structures. This means that strings, arrays, and dictionaries are copied when they are assigned to a new constant or variable, or when they are passed to a function or method.

Swift 的 `String`, `Array`, 和 `Dictionary` 类型是用结构体实现的，而结构体是一种值类型，在赋值，参数传递过程中，会使用直接拷贝内存的方式。所以，这些字符串，数组，字典等对象在这些操作里面，将会被拷贝，跟原来的对象互不影响。

<!-- more -->

这是一个很方便的特性，可以避免一些麻烦，比如容器在传递过程中被意外更改等。对比 Objective-C 就知道这样做的好处了。
在 Objective-C 里面，这些容器是用类来实现的（继承自 NSObject），赋值，或者传递参数等过程中，只会增加对象的引用计数。所以一个对象不能随意暴露内部 Mutable 的容器，比如：

```
// in CustomClass.h
// warning, visible mutable array.
@property (strong, nonatomic) NSMutableArray *mutableDataList;
```
这样的代码存在一种隐患，外面可以随意修改这个容器里面的元素，导致类内部处理这个数组的时候，会出现不一致性。比如例子中的 `CustomClass` 在 `dataList` 被改变的时候，会同步修改界面，而外部对象可能随意修改了 `dataList`，但是并没有触发 `CustomClass` 刷新界面。所以正确做法是，对外只暴露一个不可更改的属性。

```
// in CustomClass.h
@property (strong, nonatomic, readonly) NSArray *dataList;
```
```
// in CustomClass.m
@property (strong, nonatomic, readonly) NSMutableArray *mutableDataList;

- (NSArray *)dataList
{
  return [self.mutableDataList copy];
}
```
或者直接把属性定位为 copy，但是是个次优的方案。
```
// in CustomClass.h
@property (copy, nonatomic) NSMutableArray *mutableDataList;
```

用 Swift 就不会有这样的困扰了，但是注意以上的例子的拷贝只是浅拷贝，只拷贝了容器本身，容器里面的元素，如果是个对象的话，那么在传递过程中，还是同一个对象。
