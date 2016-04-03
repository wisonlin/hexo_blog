---
layout: post
title: "iOS8 导航控制器横屏页面弹出竖屏页面的实现"
date: 2014-10-19 20:07:21 +0800
comments: true
categories: iOS
---

我们都知道，同一个导航控制器中，我们可以做到两个不同的页面有不同的方向。 
例如一个页面支持竖屏，我们 push 一个新的页面，支持横屏。
本文介绍了这种情况下，iOS 8 做 pop 动画的实现。

<!-- more -->

例子如下：

<img src="/images/2014-10-19-1.jpg" width="320" height="568" />
<img src="/images/2014-10-19-2.png" width="320" height="568" />

好，问题来了，第二个页面横屏，点击第二个页面的返回按钮，触发导航栏的 pop 逻辑，此时的动画应该是怎么样的？

iOS 8 会有这样的一个动画：

<img src="/images/2014-10-19-3.png" width="434" height="593" />

横屏页面会原封不动的，做一个类似pop的动画，箭头为横屏页面的动画方向。
iOS 8 为了做到这个效果，选择对导航控制器的 pop 操作动刀。具体是这么做的：
做 pop 动作的时候，导航控制器发现当前控制器是横屏，上一个控制器是竖屏的时候。
那么会临时 present 一个当前页面的截图，由这个截图来模拟 pop 动画，而真正的 pop 操作不包含动画。
流程如下：

![image](/images/2014-10-19-4.png) 

简而言之，iOS 8 通过截图的方式，实现了横屏到竖屏的 pop 动画。
所以，-[UINavigationController popViewControllerAnimated:] 这个接口的实现就会复杂很多。
此时的 pop 操作，包含了 present一个页面，dismiss一个页面，以及真正的pop操作。
代码大概应该是这样的：

```objc
- (UIViewController *)popViewControllerAnimated:(BOOL)animatd
{
    if (animated) {
        if (self.topController.orientation == landscape && self.preViewController.orientation == portrait) {
            UISnapshotModalViewController *snapShot = [[UISnapshotModalViewController alloc] initWithViewController:self.topViewController];
            [self presentViewController:snapShot animated:NO];
            [self popViewControllerAnimated:NO];
            [self dismissViewControllerWithPopAnimate];
        }
        [self doPopViewControllerAnimated];
    } else {
        [self doPopViewController];
    }
}
```

以上，便是 iOS 8 在 横屏页面 pop 出竖屏页面的动画的实现。
注意第七行代码，如果调用 popViewControllerAnimated 时如果传入的参数是 YES，
那么 popViewControllerAnimated 会被调用两次！
第一次是 popViewControllerAnimated:YES，第二次是 popViewControllerAnimated:NO 。
重写或者替换 UINavigationController 的 popViewControllerAnimated: 时需要注意这里。
