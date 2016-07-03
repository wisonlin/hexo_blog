---
layout: post
title: "[iOS] ViewController的生命周期及其加载View的步骤"
date: 2012-03-03 12:17:08 +0800
comments: true
categories: iOS
---


首先先阐明ViewController的职责：对内管理与之关联的View，对外跟其他ViewController通信和协调。对于与之关联的View，ViewController总是在需要的时候才加载视图，并在不需要的时候卸载视图，所以也同时担当了管理应用资源的责任。理解ViewController的LifeCycle（生命周期），能够有效地管理应用资源。

<!-- more -->

#### **ViewController的初始化：**

从Storyboards中加载的时候，会调用initWithCode，如果不存在则调用init。之后对里面的每个对象调用awakeFromNib方法。
从内存中alloc出来的情况下，调init方法。
ViewController查找与其关联的view，其顺序是：

1. 先判断子类是否重写了loadView，如果有直接调用。之后调viewDidLoad完成View的加载。

2. 如果是外部通过调用initWithNibName:bundle指定nib文件名的话，ViewController记载此nib来创建View。  

3. 如果initWithNibName:bundle的name参数为nil，则ViewController会通过以下两个步骤找到与其关联的nib。
A 如果类名包含Controller，例如ViewController的类名是MyViewController，则查找是否存在MyView.nib；
B 找跟ViewController类名一样的文件，例如MyViewController，则查找是否存在MyViewController.nib。

4. 如果子类没有重写的loadView，则ViewController会从StroyBoards中找或者调用其默认的loadView，默认的loadView返回一个空白的UIView对象。
注意第一步，ViewController是判断子类是否重写了loadView，而不是判断调用子类的loadView之后ViewController的View是否为空。就是说，如果子类重写了loadView的话，不管子类在loadView里面能否获取到View，ViewController都会直接调viewDidLoad完成View的加载。

#### ** ViewController的卸载View的步骤：**

1. 系统发出内存警告或者ViewController本身调用导致didReceiveMemoryWarning被调用
2. 如果此时view没有被加入到任何视图树上，则调用viewWillUnload之后释放View
3. 调用viewDidUnload

注意view的Load和Unload不是成对调用的。
因为viewWillUnload和viewDidUnload这两个函数只在内存警告时被调用。
就算强制将viewController的view设为nil也不会触发。
如果viewController从创建到销毁期间都没有内存警告，那么这两个函数将始终不会被调用。

更新：
iOS6已将viewWillUnload和viewDidUnload废弃，原因是UIKit在内存警告的时候已经不会自动释放无用的视图。
详见 viewDidUnload 和 viewWillUnload 被废弃

注意：
由于Controller加载View时，会自动将一些View对象指向其对应的IBOutlet变量。
所以当view被卸载时我们必须在viewDidUnload将这些变量release掉，ViewController不会自动做这件事。
具体做法是将变量设置为空，（注意和dealloc中将变量release的区别）注意此时Controller的view属性是空的。
在ViewController的生命周期的各个阶段，我们都有责任去适当的创建和销毁对象，具体各个阶段要做的事情，见官方文档的表Managing Memory Efficiently

注：本文中的ViewController即视图控制器，根类是UIViewController。View是视图，根类是UIView。
