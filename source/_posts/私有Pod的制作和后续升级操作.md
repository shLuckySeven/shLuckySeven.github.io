---
title: 私有Pod的集成和后续升级操作（手把手版本）
date: 2019-04-08 22:54:33
tags: 
	- cocoaPods
categories: 组件化
---
## 开篇：
入戏：
>有一天，亘亘深夜做了个梦，梦到自己成为了一个牛逼的巨佬，坐拥上千万的开发者迷弟迷妹粉丝，开源框架上千款……
一次粉丝见面会结束，观众散去，亘亘看见会议的角落坐着一位年纪尚小的开发者独自发闷，走近聊了几句，小弟弟告诉他：原来是一个刚入行的开发者，对私有Pod的房集成不是太明白，因为这个问题，还丢失了一个很好的就业机会。于是，亘亘告诉他，等他回去找时间出一篇关于私有Pod集成的博客，帮助这类人搞懂这里面的弯弯绕绕。于是，这篇博客诞生了！

好的故事未完，美好即将待续……

私有pod，我们都不陌生，那么我们如果想让别的开发者肆无忌惮的几个命令行就嵌入我们的自己做的私有库使用，这也是件自豪而牛逼的事，那么，如何制作私有Pod呢？   
其实制作私有pod，把主要的几个步骤掌握住，就很简单了，下面我简要讲解下私有pod的制作步骤以及后续升级的步骤
# 目录
- 第一步：创建私有库工程
- 第二步：私有库本地的配置和集成
- 第三步：Specs 文件仓库管理
- 第四步：关于后续私有库升级

## 第一步：创建私有库工程
cd到你想放项目的本地目录下，比如桌面上新建一个叫“podKit”的文件夹，那么，我们依次执行以下命令：    

> - cd desktop   
> - pod lib create podKit   

成功后会出现以下代码，按需选择填写即可
```
# 选择编程语言
What language do you want to use?? [ Swift / ObjC ]
> Objc  

# 在你的项目中是否创建一个demo工程，为了方便测试，我选择了Yes
Would you like to include a demo application with your library? [ Yes / No ]
 > Yes  

# 测试框架选择哪一个
Which testing frameworks will you use? [ Specta / Kiwi / None ]
 > None

#要不要做视图测试
Would you like to do view based testing? [ Yes / No ]
 > Yes

# 类前缀名
What is your class prefix?
 > SH
```

![1-1.png](https://upload-images.jianshu.io/upload_images/1707644-1c2226c277ca713d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
Pod私有库创建成功后会自动打开项目~

## 第二步：私有库本地的配置和集成
1. 找到刚才创建的podKit文件夹，点击入去，里面的目录如下：

![1-2.png](https://upload-images.jianshu.io/upload_images/1707644-5109d53c2a59b2d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2. 把你想要做成私有库的文件，整理好，目录结构都调整好，一股脑拷贝到podKit这个目录下面，这个文件夹下面默认有个Classes和Assets文件夹，我个人的习惯，都会把它们删掉，直接把我将要做成私有库的文件拷贝进来，类似这样：
![1-3.png](https://upload-images.jianshu.io/upload_images/1707644-ecad1550464ee28e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. 接下来，打开项目，打开那个Example文件下下面的xxx.xcworkspace，这个不用我多说了吧，这个时候你展开的项目，展开Pods -- 啥也没有，哈哈哈，别急，往下继续看：
4. 编辑podspec文件
编辑这个文件之前，需要去github账号上，新建一个远程仓库，把地址拷过来洗干净备用，新建远程仓库，我在这不做说明了
> 它的位置就在根目录下，就是那个podKit. podspec，在xcode里面打开直接编辑它就行，上个图吧，这个玩意其实也简单：   

![1-5.png](https://upload-images.jianshu.io/upload_images/1707644-f3be2fbf3110d68f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`这个玩意编辑的时候一定要小心，最容易出错的一个环节，验证spec文件的时候如果这个文件有问题会报错`
5. cd到我们的Example目录下，执行：pod update --no-repo-update 命令行
这个时候再看下刚才的目录，上图我已经标注好了，ClassA 就添加进来了，因为ClassB文件夹为空，所以没有添加进来
>注意：你要不听我的话，不cd到我说的目录，那你就等着看红吧(因为在那个目录下才有PodFile文件呀)，还有别问为啥update后面为啥跟那些小可爱，因为你这个时候还没有远程仓库   

6.  然后这时候在你本地测试一下私有库是否可以使用
> 在你的工程下面的Example下，有个Example for podKit 文件夹，展开，便是我们熟悉的正常xcode项目目录了，在那做个#import导入试试就行，不报错，编译能通过，就成功了

7.  验证spec文件
> cd到项目根目录下执行： pod lib lint --allow-warnings
![1-6.png](https://upload-images.jianshu.io/upload_images/1707644-8fc237619d07bee5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 执行了3遍，第三次成功了，截图我有说明，有两次是反面教材，哈哈
> 这一步，切记：命令行的执行目录是不同的：
pod lib lint --allow-warnings是在项目根目录下
pod update --no-repo-update 是在Example目录下

8. 项目发布，git提交操作
***依次执行以下命令，一定要按照顺序执行：***
> - git git remote add origin https://github.com/shLuckySeven/podKit.git
> - git status 
> - git add .
> - git status
> - git commit -m "提交信息"
> - git pull origin master --allow-unrelated-histories
> - 这一步，注意：上面执行完pull origin master后，由于你的仓库是新建的，建的时候我们可能创建了README和ignore文件，所以pull的时候，和本地的文件会有冲突，所以，这个时候我们继续执行：
> - git status 查看冲突
> - git add .
> - git commit -m "解决冲突
> - git push origin master
到这，应该就成功了，如果还是报错，那你就自行百度吧，或者对着我的步骤再仔细核对一遍   

8-1. 打tag
> - git tag 0.0.1
> - git push origin 0.0.1   

到这，我们的私有库代码部分就结束了，登录你的github查看远程仓库，应该就有你的代码了，下面，我们开始整 spec文件管理仓库   
## 第三步：Specs 文件仓库管理
specs 的作用：
> Specs，叫做私有库文件管理仓库，也就是说我们后续对私有库升级、版本的操作，全靠这个东西，它的制作流程和之前制作代码仓库的流程一样，假如我们创建的Specs叫 "SHspec"

1. 添加Specs到本地
> 执行命令行：pod repo add SHSpec https://github.com/shLuckySeven/SHSpec.git   
这里给大家说个小窍门：快捷键Command+Shift+.查看隐藏文件
![1-7.png](https://upload-images.jianshu.io/upload_images/1707644-5afa689457627864.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如图：我们的~/.cocoapods/repos/SHSpec出现了，它里面就替我们管理了我们的私有pod的spec版本文件

2. 将私有库的.podspec文件添加到本地的Specs中，并同步到远端仓库
> cd到我们的私有库项目根目录下，一定要cd到这
> 执行： pod repo push SHSpec podKit.podspec 

![1-8.png](https://upload-images.jianshu.io/upload_images/1707644-9477116f6636f61c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
再去刚才说的~/.cocoapods/repos/SHSpec目录下面，看看：
![1-9.png](https://upload-images.jianshu.io/upload_images/1707644-3ec49deb20331380.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
podKit出现了，而且，版本号是0.0.1，登录github远端查看

![1-10.png](https://upload-images.jianshu.io/upload_images/1707644-de579afdafced94f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

到这，其实可以说已经非常nice了，但是我们还要用我们的私有pod库，命令行：
> pod search podKit

此处，应该有掌声，我觉得！

## 第四步：关于后续私有库升级
> 1.修改代码，需改podspec文件里面的版本号
> 2.pod update --no-repo-update 
> 3.代码编译验证通过之后，将代码push到远端
> 4.打新的tag 
git tag -a '0.0.1'  -m '描述' //添加tag
git tag //查看tag
> 5.podspec文件验证：pod lib lint --allow-warnings 
> 6.将podspec文件push到Specs仓库中：
pod repo push SHSpec podKit.podspec

又到了深夜说再见的时候，观众朋友们，下次再见！


