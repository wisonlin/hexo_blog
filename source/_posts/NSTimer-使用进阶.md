---
title: NSTimer 使用进阶
tags:
- iOS
- NSTimer
---

NSTimer 是 iOS 上的一种计时器，通过 NSTimer 对象，可以指定时间间隔，向一个对象发送消息。NSTimer 是比较常用的工具，比如用来定时更新界面，定时发送请求等等。但是在使用过程中，有很多需要注意的地方，稍微不注意就会产生 bug，crash，内存泄漏。本文讲解了使用 NSTimer 时需要注意的问题。
<!--more-->
#### 1. NSTimer 容易泄漏
比如以下代码创建了一个计时器：

```
self.timer =
  [NSTimer scheduledTimerWithTimeInterval:1
                                   target:self
                                 selector:@selector(update)
                                 userInfo:nil
                                  repeats:YES];
```
上述代码，将创建一个无限循环的 timer，并投入当前线程的 Runloop 中开始执行。此时，Runloop 会引用住 timer，timer 会引用住 self，self 则保存了 timer。如下图所示：

![image](/images/20161012-1.png)

需要注意的是，这种无限循环的 timer，会一直执行，需要调用`[timer invalidate]`显式停止。否则 runloop 会一直引用着 timer，timer 又引用了 self，导致 self 整个对象泄漏，实际情况中，这个 self 有可能是一个 view，甚至是一个 controller。

那，`[timer invalidate]` 要什么时候调用？
有些人会在 self 的 dealloc 里面调用，这几乎可以确定是错误的。因为 timer 会引用住 self，在 timer 停止之前，是不会释放 self 的，self 的 dealloc 也不可能会被调用。

正确的做法应该是根据业务需要，在适当的地方启动 timer 和 停止 timer。比如 timer 是页面用来更新页面内部的 view 的，那可以选择在页面显示的时候启动 timer，页面不可见的时候停止 timer。比如：

```
- (void)viewWillAppear
{
  [super viewWillAppear];
  self.timer =
    [NSTimer scheduledTimerWithTimeInterval:1
                                     target:self
                                   selector:@selector(update)
                                   userInfo:nil
                                    repeats:YES];
}

- (void)viewDidDisappear
{
  [super viewDidDisappear];
  [self.timer invalidate];
}

```

#### 2. 错误特征
  实际开发中，或者 Code Review 的时候，可以通过一些特征初步判定可能会有问题。

##### 错误特征 1:

```
- (void)dealloc
{
  [self.timer invalidate];
}
```
  以上代码是有问题的。当 timer 没有停止的时候，self 会被引用，也就没有机会走到 dealloc。同时，代码作者应该对 timer 没有正确的认识，所以需要 review 整个 timer 的使用情况。
##### 错误特征 2：

```
[NSTimer scheduledTimerWithTimeInterval:1
                                 target:self
                               selector:@selector(update)
                               userInfo:nil
                                repeats:YES];

```
以上代码创建了一个 timer，但是没有保存起来，后续自然也没有机会停止这个 timer。所以会导致 timer 泄漏。

##### 错误特征 3:

```
- (void)viewDidAppear:(BOOL)animated
{
  [super viewDidAppear:animated];
  self.timer =
    [NSTimer scheduledTimerWithTimeInterval:1
                                     target:self
                                   selector:@selector(update)
                                   userInfo:nil
                                    repeats:YES];
}
```

以上代码也是有问题的。因为我们要确保 timer 的创建和销毁必须是成对调用，否则会发生泄漏。而对于 viewDidAppear 其实很难找到一个准确的与之成对的方法（跟 viewWillDisappear 和 viewDidDisappear 都不是成对调用的），这里就需要检查 Timer 有没有被重复创建和有没有在适当的时机销毁。

#### 3. 停止 timer 可能会导致 self 对象销毁
值得注意的是，调用 `[timer invalidate]` 停止 timer，此时 timer 会释放 target，如果 timer 是最后一个持有 target 的对象，那么此次释放会直接触发 target 的  。比如：

```
- (void)onEnterBackground:(id)sender
{
	[self.timer invalidate];
	[self.view stopAnimation]; // dangerous!
} 
```

以上代码，加入第一行的 invalidate 之后，self 被销毁了，那么第二行访问 self.view 时候，就会触发野指针 crash。因为 Objective-C 的方法里面，self 是没有被 retain 的。这种情况，有个临时的解决方案如下：

```
- (void)onEnterBackground:(id)sender
{
	__weak id weakSelf = self;
	[self.timer invalidate];
	[weakSelf.view stopAnimation]; // dangerous!
} 
```
将 self 改为弱引用。但是也是一个临时解决方案。正确解决方法是，查出其它对象没有引用 self 的时候，为什么 timer 还没停止。这个案例告诉大家，当见到 invalidate 被调用之后很神奇地出现了 self 野指针 crash 的时候，不要惊讶，就是 timer 没处理好。

#### 4. Perform Delay

`[NSObject performSelector:withObject:afterDelay:]` 和 `[NSObject performSelector:withObject:afterDelay:inMode:]` 我们简称为 Perform Delay，他们的实现原理就是一个不循环（repeat 为 NO）的 timer。所以使用这两个接口的注意事项跟使用 timer 类似。需要在适当的地方调用 `[NSObject 
cancelPreviousPerformRequestsWithTarget:selector:object:]`


#### 5. Runloop Mode

注意创建 NSTimer 或者调用 Perform Delay 方法，都是往当前线程的 Runloop 中投递消息，大部分接口的默认投递模式是 CFRunloopDefaultMode。也就是说，Runloop 不在 DefaultMode 下运行的时候（比如滚动列表的时候主线程的 runloop mode 是 CFRunloopTrackingMode），消息将被暂时阻塞，不能及时处理。

#### 6. Weak Timer

NSTimer 之所以比较难用对，比较重要的原因主要是 NSTimer 对 target 是强引用的。这导致了 target 泄漏，或者生命周期超出开发者的预期。timer 如果对 target 是弱引用的话，这些问题就不存在了，这就是 Weak Timer。
Weak Timer 的实现方式分为两种，第一种是在 NSTimer 和 target 中间加多一层代理(Proxy)，代理作为 target 被 NSTimer 强引用，同时弱引用真正的 target，并对它转发消息。示例图如下：

![image](/images/20161012-2.png)

```
+ (NSTimer *)qz_scheduledWeakTimerWithTimeInterval:(NSTimeInterval)ti target:(id)target selector:(SEL)selector userInfo:(id)userInfo repeats:(BOOL)repeats
{
    QzoneWeakProxy *proxy = [[QzoneWeakProxy weakProxyForObject:target];
    return [self scheduledTimerWithTimeInterval:ti target:proxy selector:aSelector userInfo:userInfo repeats:repeats];
}
```

第二种方案是用 dispatch timer 自己实现一遍 timer，具体实现里面，弱引用 target。
比如这个：https://github.com/mindsnacks/MSWeakTimer。