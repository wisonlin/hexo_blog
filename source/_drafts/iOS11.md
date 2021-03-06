## Keynote & Platforms State of Union

#### Core ML

ML (Machine Learning) 基本是这次大会的主题了。Core ML 在 iOS 系统上，被用在 翻译，输入预测，照片的人脸识别，物体检测，手写识别，音乐分类等很多地方。开发者可以使用这些高层的 API 直接受益，比如人脸识别，物体检测等，也可以直接使用 Core ML 的接口来做更多功能。

从产品功能上看，人脸识别，物体识别，都是 iOS 10 已经有的功能。直接使用 Core ML 可以让我们灵活的把机器学习用在其他场景。比如宠物聊天、相关推荐等场景。

Core ML 有自己的模型格式 mlmodel，并且是公开的，同时也提供了工具，可以将大部分现在流行的库的模型转成 mlmodel（比如 Caffe，Keras 等）。

Core ML 的计算完全是本地工作的，不需要跟服务器通信，优点是速度快，稳定，节省服务器成本。
当然也有缺点，mlmodel 会被编译到安装包里面，会占用包大小，比如演示中的 Model 数据大小是 5MB。因为是 JSON 数据，理论上有大部分可以分离出来放在后台，需要尝试。

Core ML 异常火爆，后续还有更多技术细节。

#### ARkit

ARKit 算是整个 iOS 11 中最大的亮点了。目前可以知道的是支持 SLAM，而大家都知道，支持 SLAM 的 AR，才是真 AR。实际演示中，可以看到识别和追踪速度特别快。
> SLAM 的两个阶段：识别一个平面，在平面上创建模型；随着摄像头移动、转动，追踪平面的变化，来变换模型的角度，位置。

ARkit 的识别跟追踪原理是通过摄像头和陀螺仪，而且只需要单目摄像头，所以支持所有设备。所以说 ARkit 是最大的 AR 平台，这是目前能看到的 ARKit 的优势，因为从演示上看来，没有领先业界多少。注意到演示中，他们并没有快速的移动 iPad，也没有太靠近桌面，所以不知追踪质量如何。也暂时没有看到支持多平面的能力。而且在下午 Platforms State of Union 的演示中，在追踪上也出了一些问题。但是相比业界的其他产品，确实非常惊艳。

> 在 iPhone 7 Plus 上估计会使用双摄像头，对识别和追踪会有很大提高，有待确认和研究。

**ARKit 在追踪上的质量，需要以下几点来保证：**

1、相机不能中断图像数据，必须是连续的。

2、环境中的平面，必须是带有纹理的，不能是光滑的平面。

3、环境中的平面，必须是静止的。

**另外，跟苹果的内部人员交流，建议场景仅限于平面。**

**产品方面有很大的想象空间，比如我们可以通过 物体识别+AR 对同一个物体添加一些虚拟物体。（比如一起下棋）**


#### Photo

* Photo 方面最大的亮点就是开放了景深信息，可以将人从背景中分离出来，可以做一些滤镜，换背景等等。**注意这里还包括视频，所以直播也可以用。**

* 还有其他特性，比如相机支持二维码，通过 Universal Link 可以直接跳到 App。

#### 其他

* **Xcode**

Xcode 这次加了一些重构相关的功能，比如方法提取，重命名，等等，属于基础功能补全。特别的，**使用低版本不存在 API 会出警告！**我们以前经常出现调用低版本不支持的 API，并且没有警告，导致很多低版本系统找不到 API 而必崩的问题，并且低版本系统一般很难找到测试机，定位也很麻烦。现在只需要关注编译警告就可以了。开发和测试同学可以关注。

* **Password AutoFill**

网页上的密码自动填充是早就有的功能，现在 iOS 提供了 API，可以直接使用这些数据填充到 Native 的 UI 上。我们的登录界面可以用得上。
	
* **Drag & Drop** 

算是一个大特性，主要用在 iPad 上，不得不提应用间传递文件的方式还是通过拷贝文件，但是得益于 APS，这个拷贝底层实际上没有发生，确实不错。

## Other

Swift Playground 2
	Swift Playground 是一款编程教学应用，使用真实的代码教学，这次新版本新增了支持分享，社区，简单说，就是支持交作业，也包括抄作业了。
	
Refactoring
协议补全




What's New in Cocoa Touch

1. Drag & Drop
	* 拖拽功能自定义动画，自定义内容，

2. File Management

	* 

3. UI RefineMent
	* Large Title
	* Cell 的右划菜单

4. Swift 4 & Foundation
	
 
5. Differing System Gesture

6. 现在可以挡住边缘检测。

7. AutoLyaout & ScrollView
	ScrollView Size vs SrcorllView Content Size
	
8. Dynamic (Font) Type
	Labe

Password AutoFill
Asset Catalog

Variable Refresh Rate
	可控制？

Localization


ARKit
A9

TestFlight 人数会增加，但是还不确定。

NLP
时态
分词 API，词性，标签。

Named Identify includes chinese ？
when does Named Entity supports chinese

Debuging whth Xcode 9
Vision Framework: Building on Core ML
Customized Loading in WKWebView
What's New in iTunes Connect
Capturing Depth in iPhone Photography

Core ML in Depth
does mlmodel support oc ?
bundle spport download ?


## ARKit Lab

Hi all，
昨天和今天都去了 ARKit Lab 问问题，这两天 ARKit Lab 前面都是大排长队，简直是明星 Kit。
相信大家已经迫不及待地想要了解里面的一些技术细节。
以下是我在那里问的问题，同时也有其他同事让我问的问题（更多是产品方面），也一并梳理在后面。

#### 1Q：是否可以同时识别多个平面？
1A：多平面是支持的，可以在同一个世界模型下，同时识别多个平面，这些平面会在本次探测中有效，他们会被记住。（然后这位热心的开发人员直接跟我说要怎么做才能做到，非常 nice。之前设想的宠物可以在多个平面上跳跃就可以实现了）

#### 2Q：iPhone 7 Plus 有双摄像头，ARKit 有没有利用这个摄像头，让平面识别更加精准？
2A：没有，为了统一设备体验，没有使用。

#### 3Q：是否可以识别平面的形状？
3A：不行，识别的平面都是方形的。
（实际上我用 demo 试了一下，也发现确实只能是方形，比如一个虚拟物体放在地上，移动摄像头让圆形的桌子遮住它，会发现圆形的桌子，被识别成一个方形的平面，虚拟物体提前消失。）

#### 4Q：支持竖直平面，比如能否识别墙壁？
4A：暂时不支持。

#### 4Q2:那开发者能否通过接口加强来做到竖直平面的识别？我们都知道，ARKit 是通过 Core ML 支持的，开发者能否通过替换底层的训练集，让 ARKit 支持？
4A2:不行，ARKit 从上到下都是不支持定制的，如果要定制，只能从 Core ML 做起，相当于自己开发一个 ARKit。

以下是帮别人问题的问题，也一并梳理了：

#### 1Q：有没有特征提取的接口，能否进行特征匹配，识别同一种物体？

1A：不支持，这不是 ARKit 需要提供的功能。（我也觉得，可以用物体识别的库）

#### 2Q：低端机兼容。

2A：不支持，也不考虑。（太正常了，只是循例问一下）

#### 3Q、是否可以识别刚体的轮廓，比如一个可乐瓶。

3A、不支持，以后会考虑。（不过我们自己可以通过景深功能做到轮廓效果）

#### 4Q：观测场景的信息共享，跟 LBS 结合。举个例子，两台手机在拍同一张桌子，有没有可能通过一些方式识别到这是同一张桌子？

4A：不支持，很难做到，因为手机上建立的一个世界，跟其他手机建立的世界是分开的。但是以后会考虑。
（这个我们也想想方法，比如 LBS+蓝牙识别手机的距离，物体识别检测是同一种物体，陀螺仪识别方向，就大体上可以知道是同一个物体了。）


## Kernel & Runtime Lab

以下是跟大家收集到的跟系统核心相关问题的解答，有疑问的我都跟他们互留了联系方式。

#### 应用的线程数限制
Q：一个 App 的线程数有没有上限？如果有，每种设备的上限分别是多少？达到上限之后，再创建线程会有什么事情发生？

A：**单个 App 的线程数限制是 500**，准确的说，应该是单个进程的线程数。所有种类的设备都统一 限制500个，但 Mac Pro 上可以达到 2000。如果达到上限，那么当你创建线程的时候，你不会得到一个线程，比如使用 pthread_create，那么会返回 EAGAIN，并且不会创建线程。（后面省略各种提醒，叫我们不要使用太多线程，耗电大，发热，应该多用线程池。。）

#### 应用的内存限制
Q：单个应用能够使用的最大内存是多少？
A：暂时没有答案，只知道在达到最大限制的 80% 的时候，会收到内存警告，他查到会联系我。

#### 占用空间限制
Q：单个应用的占用空间是否有限制？
A：目前来说没有限制。怀疑是句柄达到上限？当然，如果有发现这种问题，可以 bug report。





景深可以拿来做动画



## Keynote Agenda

ARKit

功能

工作原理
World Tracking
Visual Inertial Odometry
No external setup

Plan Detection
Hit-testing
Light estimation

Easy integration
AR Views
Custom rendering

流程
平面
Simultaneous localization and mapping
demo


Core ML

工作原理
架构
功能描述
流程
自定义 Model

## Work Shop

day 1 上午
sesion：
	重新讲解了一遍 Introducing ARKit
	
Q&A：
Technical Inside(SLAM)
Marker (Core ML + Vision)
Feature Point 计算
Shadow (SceneKit)
Vertival Plane & angle (not support (yet), Feature Point)
zhiwei_wang@apple.com



NEW SURFACE DETECTED AT (-0.09, -0.97, -0.81)
NEW SURFACE DETECTED AT (0.27, -1.11, -0.98)
NEW SURFACE DETECTED AT (-0.60, -0.28, -0.20)NEW SURFACE DETECTED AT (0.32, -1.00, -1.05)

1,2,4 是同一个平面，第三个值是 z，误差有点大


day 2

1.RGB presantation, downside to metal
2.CUBE
3.barrier area
4.marker 的识别和 arkit 合并？(主要是考虑性能问题)（NO）
5. depth map (NO), 可以通过 feature point 产生
6. 光的方向（NO）
7.忽略
8. 前置摄像头（主要限制在 moving tracking）
9. 精准的边界检测
10. camera， flashlighgt 参数（可以）（待定）
10.1 captureSesion 和 arsession 不能同时工作。
11. opengl
12. shadow solution(scenekit session)
13. ARKit 的工作原理（看session）
14. 支持 FBX ？
15. size estimation accuration
16. workd tracking 的合理范围（很难讲 feature point 够多就行）
17. relative position of 2 phones(u may try it upside the arkit; or make the phone togather while init)
18. A9 UP
19. vetical plane(cant provide robust solution)
20. how to reduce power consumption of caturesession & cmmotion(10-15%, 25-30%, do as...as we can)
21. 
22. feature point 的更多信息（NO）
23. ARKit 提供 GPU 接口（NO）
24. 多人游戏（可以，但是 arsence 不行）
25. when world created?
26. area learning?
27. 性能问题（10%-15% iphone 7 and newer device， wrorld tracking using cpu）
28. 忽略
29. featrue plane (no plan (yet))
30. 

总结：

1. 关于 marker
	很多人问到了 marker，他们还是建议在 ARKit 的基础上去做这个特性，可以尝试使用 Core ML+Vision。（其实他们很不理解，SLAM 是比 marker 更先进的技术，为啥你们要用 marker）
2. 为什么不支持前置摄像头
	像直播等场景，主要是使用前置摄像头，然而 ARKit 并不支持。他们给的理由是，ARKit 是很依赖运动信息的，直播的时候手机一般是固定在一个地方，很难检测。
3. 平面的边界
	大家的需求普遍需要一个精准的平面边界，不过目前还是做不到。他们给的回答是随着平面识别的进行，识别的平面会越来越大，基本可以把整个现实中的平面识别出来（当然这里指的是方形平面），精度不够，但是大部分情况下是够用的。
4. 性能问题
	平面识别和追踪是用 CPU 跑的，渲染则是用 GPU。CPU 方面，iPhone 7 及以上的设备占用 10%-15%，较低端的设备则占用 25%-30%。