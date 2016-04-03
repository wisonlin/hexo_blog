---
layout: post
title: "Can't add self as subview"
date: 2014-06-01 11:37:34 +0800
comments: true
categories: iOS
---

iOS7刚发布的时候，总是出现这个 Can't add self as subview  的崩溃，团队内部没出现过，但是外部用户的crash频频上报这个崩溃。本文讲述发现这个bug，分析定位，到解决的过程。

<!-- more -->

异常描述和崩溃堆栈是这样的：

*** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: 'Can't add self as subview'

![image](/images/2014-06-01-1.jpg)

这里有两个线索，一个是从崩溃堆栈中看到了崩溃的时间点：导航栏对内部的控制器做切换动画的时候崩溃。

另一个线索则是addSubView的参数不能是对象本身。一开始怀疑是addSubView传入self引起，于是用类似 [self addSubView:self] 的代码试了一下，的确是会崩溃的。

![image](/images/2014-06-01-2.jpg)

但是堆栈跟外部用户上报的不一样，排除 [self addSubView:self] 直接导致崩溃的可能性。

也就是说，不是我们工程调用了[self addSubView:self] 引起崩溃，
而是我们工程里面的某一些代码会导致UIKit内部执行 addSubView 的时候，传入了 self。
为什么说是我们的代码引起呢？因为崩溃的时候，页面总是停留在某几个特定的页面，这个后面会分析。

再看看第二个线索，即导航栏在做动画的时候出了问题。

以上，我们可以得出一个中间结论，即我们的代码，让导航栏在做动画的时候，执行了一次 [self addSubView:self]。
再说说崩溃集中的几个页面，用户上报的崩溃中，并不是总在一个页面崩溃，但是固定出现在特定的某几个页面。

着重看了log里面崩溃前每一个页面切换的时间，果然比较短，有些甚至少于0.5秒，少于导航栏push和pop动画的时间。
接着，用代码模拟一下快速切换的场景，比如0.3秒只能做两次push操作：

```objc
[navigationController pushViewController:[[TMViewController alloc] init] animated:YES];
	dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(.3 * NSEC_PER_SEC)), 
	dispatch_get_main_queue(), ^{
	[navigationController pushViewController:[[TMViewController alloc] init] animated:YES];
}
```

果然崩溃了，得到的堆栈跟文章开头的一模一样。

接着，检查崩溃上报的其他页面，发现这些页面都在特定的场景下会出现同时做push或者pop操作的情况。
于是，模拟各种短时间push和pop页面的场景，都会出现这样的崩溃。
这时候几乎已经明确了就是动画被打断引起的。

原来，其实就是iOS5，6下的nested push 问题，只不过到iOS7上，这个问题的提前在做动画的时候崩溃了。

为了防止这种情况，我们在UINavigationController基类中加入防御，具体做法是在push，pop方法中设置一个标志位(animating=YES)，在动画结束之后，再重置这个标志位，然后，用这个标志位判断push和pop操作是否能够执行。

比如，push这样实现：

```objc
- (void)pushViewController:(UIViewController *)viewController animated:(BOOL)animated
{
	if (self.topViewController.animating) {
		NSLog(@"error push when animating.");
		return;
	}
	self.topViewController.animating = animated;
	viewController.animating = animated;
	[super pushViewController:viewController aniamted:animated];
}
```

其中有一个细节是，参与动画的两个控制器都需要设标志位。
结束动画的时候重置标志位，时机是在控制器的viewDidAppear和viewDidDisappear里面。

```objc
- (void)viewDidAppear:(BOOL)animated
{
	[super viewDidAppear:animated];
	self.animating = NO;
}

- (void)viewDidDisappear:(BOOL)animated
{
	[super viewDidDisappear:animated];
	self.animating = NO;
}

```
