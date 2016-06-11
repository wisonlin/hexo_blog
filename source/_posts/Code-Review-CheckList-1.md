---
title: Code Review CheckList 1
date: 2016-04-09 19:30:54
tags:
- iOS
- Best Practice
categories: Code Review CheckList
---

本文记录了一些 Objective-C 上的一些最佳实践，作用是每个人都可以通过本文描述的规则，对其它团队成员进行 Code Review。

另外，每个规则后面，会有一个规则代号，用中括号括起来，发现代码不符合某一条规则的时候，可以直接使用代号取代问题描述。比如看到某些代码不符合下面的第一条规则，可以在所在的代码行增加评论:`[Be Modern]`。

<!-- more -->
以下是规则：

### 1. 充分利用 Modern Objective-C [Be Modern]
1. `@property` 不用写 `@synthesize`，除非自己同时定义了 setter 和 getter。

2. 使用 literal 来定义常量对象。
```
[NSNumber numberWithInteger:2]
    => @(2)
[NSArray arrayWithObjects:a, b, c]
    => @[a, b, c]
[NSDictionary dictionaryWithObjectsAndKeys:oa, ka, ob, kb]
    => @{ka : oa, kb : ob}
```

3. 使用 NS_ENUM 来声明枚举。
```
typedef NS_ENUM(NSUInteger, POPAnimationEventType)
{
	...
}
```

 *详细可参考 WWDC 2012 session 405 modern objective-c*


### 2. ARC 带来的变化 [ARC]

 1. `dealloc` 方法里面不需要释放成员变量的引用。

 2. `delegate`(委托)作为属性要用 weak 修饰，避免循环引用和野指针。一些系统的老接口，需要确保 `dealloc` 的时候将 `delegate` 置 nil。

### 3. 封装

 1. 利用匿名 category 来定义私有属性，和声明遵循的没必要公开的协议。

### 4. 越界
 4. 迭代方式尽量用 for in，可以避免操作越界。



### 5. 多线程问题

1. `synchronize` 和 `dispatch_once`

    尽量用 `dispatch_once`，性能不错，某些场合语义也会更加清晰。
    在 Qzone 里面，定义单件可直接使用宏 QZONE_SINGLETON_INTERFACE。

2. property 的 atomic 属性

    多线程中，容器的 property 加了 atomic 只能保证获取和设置容器变量是原子操作，
    但是容器本身不是线程安全的，需要另外加锁。
    具体实现中，应当尽量避免多线程访问同一个容器的情况，减少锁的使用。

### 6. 对对象生命周期的把控

1. `NSTimer` 和 `performSelector:afterDelay:` 都会持有对象，需要在适当的时机打破循环引用。

2. 以上这两个 API，也需要在适当的场景停止运行，否则会增加耗电量，挂后台的时候会更快地被系统 kill 掉等等。

### 7. 调用了基类提供的只能用来重写的方法

  比如 `layoutSubviews`，`drawRect`，`touchBegan` 等等，具体需要熟悉文档。

### 8. 重写基类方法时，没有在实现中调用基类方法

  比如 `viewDidLoad`, `viewWillAppear`, `prepareForReuse` 等等，具体需要熟悉文档。

### 9. 关注性能

  1. 尽量不要触发 CPU 渲染

    尽量不使用 `[UIImage drawInRect]` 等 API。

  2. 尽量不要触发离屏渲染
```
    layer.cornerRadius = .5;
    layer.shouldRasterize = YES;
    layer.rasterizationScale = [UIScreen mainScreen].scale;
```

### 10. 关注可维护性

  1. 尽量不调用 `[UITableView reloadData]` 刷新整个列表
    除非整个列表的数据都发生变化，否则不要刷新整个列表，
    这样会让整个列表中的某些状态丢失，难以维护。推荐使用类似
    `[UITableView reloadCell:]`, `[UITableViewCell reset]`
    这样局部的刷新方式。

### 11. 范型

  过去：
```
    NSArray *viewList;
    ((UIView *)viewList[0]).hidden = YES;
```
  现在：
```
    NSArray<UIView *> *viewList;
    viewList[0].hidden = YES;
```

### 12. 拷贝

  1. 赋值
```
    aList = bList;
```
  2. 浅拷贝
```
aList = [bList copy];
```
  3. 深拷贝
```
aList = [[NSArray alloc] initWithArray:bList copyItem:YES];
// * 需要容器内的元素实现 `NSCopying` 协议。*
```


|revision|date|author|comment|
|:----:|:-----:|:----:|:----:|
|1.0|2014-7-5|wisonlin| Init Commit |
|2.0|2015-11-26|wisonlin| Add deep copy; Add generic type. |
|3.0|2016-5-12|wisonlin| 增加问题标记，和使用说明。增加封装[] |
