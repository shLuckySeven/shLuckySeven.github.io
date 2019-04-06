---
title: iOS- 性能优化（启动、电量、包体等）
date: 2019-03-06 22:13:29
tags: iOS
category: 性能优化
---

 随着我们移动互联网的兴起到火爆，成千上万款app营运而生，电商、出行、音视频、教育等等，五花八门，那么每一款APP都会有对应的人群去下载使用，那么用户对一款app的钟爱程度除了这款app很受喜爱和直接用途之外，受大家喜爱并且留存在手机上长久不卸载的一个重要原因，便是我们的app的性能，包括：启动速度、启动后的流畅度、app的安装包体积、app运行对手机的电量消耗、app是否有闪退等，这几个条件，决定了一款app是否优秀。
下面我针对这几个方面做下项目性能优化的经验和知识点分享
### 目录
- 项目启动
- 项目运行
- 安装包体积
- app的电量消耗

## 1.项目启动
>项目启动环节，我们大致分为2种启动：即**冷启动（Cold Launch
）**和**热启动（Warm Launch）**，针对优化，我们主要针对冷启动

#####  知识点：打印启动时间
> 通过添加环境变量可以打印出APP的启动时间分析（Edit scheme -> Run -> Arguments）
DYLD_PRINT_STATISTICS设置为1
如果需要更详细的信息，那就将DYLD_PRINT_STATISTICS_DETAILS设置为1
##### 冷启动大概又分为三个阶段
> 1.dyld
> 2.Runtime
> 3.main函数

![launch.png](https://upload-images.jianshu.io/upload_images/1707644-8ac5cc2cdf286e22.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
####  **dyld**
dyld（dynamic link editor），Apple的动态链接器，可以用来装载Mach-O文件（可执行文件、动态库等）
启动APP时，dyld所做的事情有:
>装载APP的可执行文件，同时会递归加载所有依赖的动态库
当dyld把可执行文件、动态库都装载完毕后，会通知Runtime进行下一步的处理

具体关于dyld的讲解，我推荐一篇文章[dyld专题讲解](https://www.jianshu.com/p/5f337da8fbef)，写的不错，后续我也会对dyld写篇自己的理解的博客
#### **Runtime**
这里说的Runtime并不是要对Runtime进行底层讲解，是针对程序的启动流程的Runtime阶段进行分析，推荐一篇关于Runtime的底层讲解[iOS底层原理总结 - 探寻Runtime本质（一）](https://www.jianshu.com/p/d949b51d5de7)
启动APP时，runtime所做的事情有
>- 调用map_images进行可执行文件内容的解析和处理
>- 在load_images中调用call_load_methods，调用所有Class和Category的+load方法
>- 进行各种objc结构的初始化（注册Objc类 、初始化类对象等等）
>- 调用C++静态初始化器和__attribute__((constructor))修饰的函数

到此为止，可执行文件和动态库中所有的符号(Class，Protocol，Selector，IMP，…)都已经按格式成功加载到内存中，被runtime 所管理

#### **Main函数**
main函数的调用，便是我们熟知的项目中Main入口和AppDelegate类里面的那一系列的调用了
### 总结一下优化方案，按照阶段：
dyld
>减少动态库、合并一些动态库（定期清理不必要的动态库）
减少Objc类、分类的数量、减少Selector数量（定期清理不必要的类、分类）
减少C++虚函数数量
Swift尽量使用struct

Runtime
>用+initialize方法和dispatch_once取代所有的__attribute__((constructor))、C++静态构造器、ObjC的+load

Main函数
>- 在不影响用户体验的前提下，尽可能将一些操作延迟，不要全部都放在 finishLaunching方法中,例如三方注册、启动图等
>- 按需加载

## 2.项目运行
其实项目运行优化，就涉及到我们的代码优化了，最常见也是影响最大的，就是 
>**卡顿现象**

`针对卡顿现象的原因和优化我在这里大概列举一下，我会在另一篇关于CPU和GPU，以及离屏渲染等方向进行讲解`
#### CPU优化策略：
1.尽量用轻量级的对象，比如用不到事件处理的地方，可以考虑使用CALayer取代UIView
2.不要频繁地调用UIView的相关属性，比如frame、bounds、transform等属性，尽量减少不必要的修改
3.尽量提前计算好布局，在有需要时一次性调整对应的属性，不要多次修改属性
4.Autolayout会比直接设置frame消耗更多的CPU资源
5.图片的size最好刚好跟UIImageView的size保持一致
6.控制一下线程的最大并发数量
7.尽量把耗时的操作放到子线程
8.文本处理（尺寸计算、绘制）
9.图片处理（解码、绘制）
#### GPU优化策略：
1.尽量避免短时间内大量图片的显示，尽可能将多张图片合成一张进行显示
2.GPU能处理的最大纹理尺寸是4096x4096，一旦超过这个尺寸，就会占用3.CPU资源进行处理，所以纹理尽量不要超过这个尺寸
4.尽量减少视图数量和层次
5.减少透明的视图（alpha<1），不透明的就设置opaque为YES
6.尽量避免出现离屏渲染
#### 离屏渲染
**在OpenGL中，GPU有2种渲染方式**
On-Screen Rendering：当前屏幕渲染，在当前用于显示的屏幕缓冲区进行渲染操作
Off-Screen Rendering：离屏渲染，在当前屏幕缓冲区以外新开辟一个缓冲区进行渲染操作

**离屏渲染消耗性能的原因:**
1.需要创建新的缓冲区
2.离屏渲染的整个过程，需要多次切换上下文环境，先是从当前屏幕（On-Screen）切换到离屏（Off-Screen）；等到离屏渲染结束以后，将离屏缓冲区的渲染结果显示到屏幕上，又需要将上下文环境从离屏切换到当前屏幕

**哪些操作会触发离屏渲染？**
- 光栅化，layer.shouldRasterize = YES
- 遮罩，layer.mask
- 圆角，同时设置layer.masksToBounds = YES、layer.cornerRadius大于0
考虑通过CoreGraphics绘制裁剪圆角，或者叫美工提供圆角图片
- 阴影，layer.shadowXXX
如果设置了layer.shadowPath就不会产生离屏渲染
#### 卡顿检测
我们平时所说的“卡顿”主要是因为在主线程执行了比较耗时的操作
##### 如何检测？
可以添加Observer到主线程RunLoop中，通过监听RunLoop状态切换的耗时，以达到监控卡顿的目的
当然网上还有更多的别的方案，后续我会针对这一块进行讲解，读者可以留言别的方式交流
## 3.安装包体积瘦身
首先我们要知道安装包的组成成分
>- 可执行文件
>- 资源（图片、音频、视频等）

##### 资源瘦身方案：
- 采取无损压缩，进行资源压缩
- 定期删除无用资源，这里推荐一个工具
[检测无用资源]（https://github.com/tinymind/LSUnusedResources）
##### 可执行文件瘦身方案：
- 编译器优化
Strip Linked Product、Make Strings Read-Only、Symbols Hidden by Default设置为YES
去掉异常支持，Enable C++ Exceptions、Enable Objective-C Exceptions设置为NO， Other C Flags添加-fno-exceptions
- 利用AppCode（https://www.jetbrains.com/objc/）检测未使用的代码：菜单栏 -> Code -> Inspect Code
- 编写LLVM插件检测出重复代码、未被调用的代码

## 4.App的电量消耗优化
首先我们要知道App好点的几个主要来源
![energy.png](https://upload-images.jianshu.io/upload_images/1707644-a838a0fca65acba8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>- CPU
>- 网络请求任务
>- 定位
>- 图片处理

我们大概从以下入手进行优化，（视自己项目情况而定）：

1.尽可能降低CPU、GPU功耗
2.尽可能少用定时器
3.优化I/O操作
```
1.尽量不要频繁写入小数据，最好批量一次性写入
2.读写大量重要数据时，考虑用dispatch_io，其提供了基于GCD的异步操作文件I/O的API。用dispatch_io系统会优化磁盘访问
3.数据量比较大的，建议使用数据库（比如SQLite、CoreData）
```

4.网络优化
```
1.减少、压缩网络数据
2.如果多次请求的结果是相同的，尽量使用缓存
3.使用断点续传，否则网络不稳定时可能多次传输相同的内容
4.网络不可用时，不要尝试执行网络请求
5.让用户可以取消长时间运行或者速度很慢的网络操作，设置合适的超时时间
批量传输，比如，下载视频流时，不要传输很小的数据包，直接下载整个文件或者一大块一大块地下载。如果下载广告，一次性多下载一些，然后再慢慢展示。如果下载电子邮件，一次下载多封，不要一封一封地下载
```
5.定位优化
```
1.如果只是需要快速确定用户位置，最好用CLLocationManagerrequestLocation方法。定位完成后，会自动让定位硬件断电
2.如果不是导航应用，尽量不要实时更新位置，定位完毕就关掉定位服务
3.尽量降低定位精度，比如尽量不要使用精度最高的kCLLocationAccuracyBest
需要后台定位时，尽量设置pausesLocationUpdatesAutomatically为YES，如果用户不太可能移动的时候系统会自动暂停位置更新
4.尽量不要使用startMonitoringSignificantLocationChanges，优先考虑startMonitoringForRegion:
```

6.硬件检测优化
```
用户移动、摇晃、倾斜设备时，会产生动作(motion)事件，这些事件由加速度计、陀螺仪、磁力计等硬件检测。在不需要检测的场合，应该及时关闭这些硬件
```
## 结尾
到这里，对app的优化就基本说完了，大家可以留言交流更好的优化方案，文章如有错误或者理解不到位之处，希望大家指点，有帮助的老铁们，点个喜欢，谢谢
Github:  [https://github.com/shLuckySeven](https://github.com/shLuckySeven)


