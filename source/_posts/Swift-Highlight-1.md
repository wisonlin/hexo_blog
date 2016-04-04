---
title: Swift Highlight 1
date: 2016-04-04 00:32:27
tags:
---

> Unlike Objective-C, Swift enables you to set sub-properties of a structure property directly.

> 跟 Objective-C 不一样，Swift 中的一个属性是结构体的话，这个结构体的变量是可以被直接改变的。

<!-- more -->

比如在 Objective-C 中，以下代码会变异错误：
```
  view.frame.size.width = 2.0;  // compile error
```

必须这样写：
```
  CGRect newFrame = view.frame;
  newFrame.size.width = 2.0;
  view.frame = newFrame;
```

在 Swift 中，则可以直接更改：
```
  view.frame.size.width = 2.0;  // compile success
```

上述这个改变视图宽高的例子，出现的场景比较常见，所以在 Objective-C 中其实有一种简单的方法，就是创建一个 Category，收归这些代码：
```
// in Category of UIView
- (void)setWidth:(CGFloat)width
{
  CGRect newFrame = view.frame;
  newFrame.size.width = 2.0;
  view.frame = newFrame;
}
```
这样，使用的时候就不会太罗嗦：
```
  view.width = 2.0;
```
