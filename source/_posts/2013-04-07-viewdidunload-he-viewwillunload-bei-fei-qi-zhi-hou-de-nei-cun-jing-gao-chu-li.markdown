---
layout: post
title: "viewDidUnload 和 viewWillUnload 被废弃之后的内存警告处理"
date: 2013-04-07 21:31:24 +0800
comments: true
categories: iOS
---

由于iOS6以上的UIKit不会在内存警告时自动释放视图，所以viewWillUnload和viewDidUnload将不再触发，因此，在iOS6上，开发者需要负责内存警告时将不用到的视图释放。
<!-- more -->
WWDC2012的视频有提到，具体代码如下：

```objc
- (void)didReceiveMemoryWarning {
     if ([self.view window] == nil) {
          self.view = nil;
          self.otherSubView = nil;
     }
}
```

由于[self view]会引发视图的加载所以上述代码还是有潜在风险的，假如视图控制器在创建之后，在还没有加载视图时收到内存警告，那上面的代码就会触发视图的加载（调用了[self view]引起），反而加大了内存占用。所以应该先判断一下视图是否已被加载。

```objc
- (void)didReceiveMemoryWarning {
     if ([self isViewLoaded] && [self.view window] == nil) {
          self.view = nil;
          self.otherSubView = nil;
     }
}
```

Notification 的注册和反注册以及Delegate的设置和置空

如果注册的通知跟界面相关，可以考虑将注册放入viewWillAppear并在viewWillDisappear中反注册。
如果需要在视图加载时就注册，那就在viewDidLoad注册，dealloc和didReceiveMemoryWarning中根据视图是否加载过来进行反注册。
注意viewDidUnload和viewDidLoad不是成对调用的，所以即使是iOS5或者以下的版本也不能在viewDidUnload里面反注册。参见[iOS] ViewController的生命周期及其加载View的步骤。

综上所述，最佳实践的代码如下：

```objc
- (void)viewDidLoad
{
     self.subView.delegate = self;
     [[NSNotificationCenter defaultCenter] addObserver:self];
     self.viewCreatedByCode = [[UIView alloc] init];
}

// 自定义函数viewUnloaded，其操作与viewDidLoad对称。
- (void)viewUnloaded
{
     self.subView.delegate = nil;
     [[NSNotificationCenter defaultCenter] removeObserver:self];
     self.viewCreatedByCode = nil;
}

- (void)didReceiveMemoryWarning {
     if ([self isViewLoaded] && [self.view window] == nil) {
          self.view = nil; // 需要开发者手动释放控制器的视图。
          self.viewCreatedByNib = nil;  // 在xib中创建的视图也要手动清空。
          [self viewUnloaded]; // 视图已被卸载，调用viewDIdLoad的反操作。
     }
}
 
- (void)dealloc
{
     if ([self isViewLoaded]) {
          [self viewUnloaded]; // 如果视图已被加载，说明viewDidLoad被调用过，所以调用viewDIdLoad的反操作。
     }
}


- (void)viewDidLoad
{
     self.subView.delegate = self;
     [[NSNotificationCenter defaultCenter] addObserver:self];
     self.viewCreatedByCode = [[UIView alloc] init];
}

- (void)viewDidUnload
{
     self.subView.delegate = nil;
     [[NSNotificationCenter defaultCenter] removeObserver:self];
     self.viewCreatedByCode = nil; 
}


- (void)didReceiveMemoryWarning {
     if ([self isViewLoaded] && [self.view window] == nil) {
         [self.photoCache removeAllObjects];
         if (>=iOS6) {
              self.view = nil;
              [self viewDidUnload];
         }
     }
}
 
- (void)dealloc
{
     if ([self isViewLoaded]) {
         [[NSNotificationCenter defaultCenter] removeObserver:self];
     }
}
```