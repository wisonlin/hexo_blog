Keynote & Platforms State of Union

#### Core ML

ML (Machine Learning) 基本是这次大会的主题了。Core ML 在 iOS 系统上，被用在 翻译，输入预测，照片的人脸识别，物体检测，手写识别，音乐分类等很多地方。开发者可以使用这些高层的 API 直接受益，比如人脸识别，物体检测等，也可以直接使用 Core ML 的接口来做更多功能。

从产品功能上看，人脸识别，物体识别，都是 iOS 10 已经有的功能。直接使用 Core ML 可以让我们灵活的把机器学习用在其他场景。比如宠物聊天、相关推荐等场景。

Core ML 有自己的模型格式 mlmodel，并且是公开的，同时也提供了工具，可以将大部分现在流行的库的模型转成 mlmodel（比如 Caffe，Keras 等）。

Core ML 的计算完全是本地工作的，不需要跟服务器通信，优点是速度快，稳定，节省服务器成本。
当然也有缺点，mlmodel 会被编译到安装包里面，会占用包大小，比如演示中的 Model 数据大小是 5MB。因为是 JSON 数据，理论上有大部分可以分离出来放在后台，需要尝试。

#### ARkit

ARKit 算是整个 iOS 11 中最大的亮点了。目前可以知道的是支持 SLAM，而大家都知道，支持 SLAM 的 AR，才是真 AR。实际演示中，可以看到识别和追踪速度特别快。
> SLAM 的两个阶段：识别一个平面，在平面上创建模型；随着摄像头移动、转动，追踪平面的变化，来变换模型的角度，位置。

ARkit 的识别跟追踪原理是通过摄像头和陀螺仪，而且只需要单目摄像头，所以支持所有设备。所以说 ARkit 是最大的 AR 平台，这是目前能看到的 ARKit 的优势，因为从演示上看来，没有领先业界多少。注意到演示中，他们并没有快速的移动 iPad，也没有太靠近平面，所以不知追踪质量如何。而且在下午 Platforms State of Union 的演示中，在追踪上也出了一些问题。

> 在 iPhone 7 Plus 上估计会使用双摄像头，对识别和追踪会有很大提高，有待确认和研究。

**产品方面当然大有作为，比如我们可以通过 物体识别+LBS+AR 对同一个物体做一些虚拟标记。可以好好构思一些想法，但是 AR 方面的场景仅限平面，可以做一些组合。**

#### Photo

Photo 方面最大的亮点就是开放了景深信息，可以将人从背景中分离出来，可以做一些滤镜，换背景等等。**注意这里还包括视频。**

引入 HEIF，一种高效率的图片格式，号称相比 JPEG 减少一半的存储空间。可以对比评估一下。

还有其他特性，比如相机支持二维码，通过 Universal Link 可以直接跳到 App。

#### 其他

**Xcode**

Xcode 这次加了一些重构相关的功能，比如方法提取，重命名，等等，属于基础功能补全。特别的，**使用低版本不存在 API 会出警告！**我们以前经常出现调用低版本不支持的 API，并且没有警告，导致很多低版本系统找不到 API 而必崩的问题，并且低版本系统一般很难找到测试机，定位也很麻烦。现在只需要关注编译警告就可以了。开发和测试同学可以关注。

**Password AutoFill**

网页上的密码自动填充是早就有的功能，现在 iOS 提供了 API，可以直接使用这些数据填充到 Native 的 UI 上。我们的登录界面可以用得上。
	
**Drag & Drop** 

算是一个大特性，主要用在 iPad 上，不得不提应用间传递文件的方式还是通过拷贝文件，但是得益于 APS，这个拷贝底层实际上没有发生，确实不错。


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