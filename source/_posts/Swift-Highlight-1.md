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

> Swift 刚出的时候学过一次，但是中间又好久没碰所以忘得差不多了，趁着 Swift 3.0 即将到来，赶紧重温了一下 Swift 2.2，这一次我想通过一系列的文章来沉淀，帮助记忆。不过现在网上关于 Swift 的技术文章实在太多，优秀的也非常多，所以我想从另一个角度去讲解 Swift，取名 `Swift Highlight`，写一些 Swift 相比老旧的 Objective-C 来说，令人眼前一亮的地方，或者它解决了 Objective-C 一些令人诟病的地方。希望能够让大家在学习 Swift 特性的同时也能够知道 Swift 为什么要这么做，好在哪里，也能够对 Objective-C 有更深层次的认识。
