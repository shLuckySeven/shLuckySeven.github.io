---
title: Github Pages配合Hexo搭建个人博客的过程
date: 2019-04-07 22:54:33
tags: 
	- Hexo
	- Node.js
	- Git
	- Next
categories: 博客搭建
---
# 开篇：（博客搬迁）
论技术博客的重要性

>- 作为一名程序员，我们不但要会敲代码，还要会总（chui）结（niu）学（bi）习（！），每一个过程都是心酸的，那么我们应该定期去把我们的学习过程、工作过程遇到的有趣的、难忘的一些技术经历记录下来，这将对我们以后的开发成长有很大的帮助，最常见的便是技术博客
>
>- 我最终选取了成本较低的专属域名 + Github Pages + Node.js + Hexo的技术方案来搬迁我的博客，查阅了好多博客和论坛，翻山越岭，将自己的博客搭建起来，比起成本最高的个人服务器+完全样式自定义，我少了服务器和纵深难度较大的样式完全自定义，后期有时间了，再往那个山头翻吧！

`看好多巨佬的博客真的挺酷炫的，就像修仙，我们不能一步到位，慢慢来吧！`

# 目录：

> #### 1.准备：环境选择和搭建
> #### 2.初长成：配置Hexo主题
> #### 4.总结：这一刻是喜悦的

## 一、准备
##### 磨刀不误砍柴工，开始搭建博客之前，我们必须先选择好方式和环境,那么我们开始前，有4个小可爱，需要了解、安装或者注册:
- Git:一款免费、开源的分布式版本控制系统
- Node.js:建立在Chrome上的JavaScript运行引擎
- Hexo:Hexo快速、简洁且高效的博客框架
- GitHub:国内一款面向开发者的云端开发平台，提供代码托管，运行空间，质量控制，项目管理等功能
- MacDown：MarkDown编辑神器

### 安装/注册/绑定：   
**Git** ----  [Git官网](https://git-scm.com)（安装环境）

找到Download,安装自己对应的系统版本，系统会自动判定你当前版本，推荐下载，如果没有推荐，那就自己选择吧。具体Git的安装步骤请自行百度
  
**Node.js** ----  [Node.js官网](https://nodejs.org/en/)（下载安装）

进入官网后系统依然会判定你的系统并给出稳定推荐版本和尝鲜版。这里我建议下载安装稳定版就可以，下载安装步骤同样省略

**Hexo** ---- [Hexo官方文档](https://hexo.io/zh-cn/docs/)（安装环境）

前面我们已经安装了Git、Node.js，然后我们开始使用npm，安装hexo，首先
cd到我们想要放置博客本地项目目录的文件夹下，依次执行以下命令：
>1.npm install hexo
>
>2.npm install
>
>3.npm install hexo-deployer-git
>
>注意：1.安装失败尝试切换到sudo下尝试；2.记得，不需要hexo init这条指令，因为init要求文件夹必须为空，后续我们讲解到博客搬迁的时候会讲到这个地方的坑

Hexo环境的安装过程，可能会遇到一些坑，到时候根据具体问题，自行百度，基本都可以找到答案，后续我会把环境配置过程中典型的易见的错误和解决办法也更新到博客中（因为我当时碰到的时候忘记了截图😭

**Github** ----  [Github官网](https://github.com)（注册绑定）

如果有github账号的，直接略过看就行，没有github账号的，老老实实按照步骤看：
>- 注册Gituhb账号
>
>- 创建仓库
>
>- 设置Github Pages(选择主题)
>
>- 注册域名并绑定到仓库中（如需要）

##### 1)注册Github账号
注册账号流程略……按照填写即可 

##### 2)注册Github账号 
新建仓库：点击头像左侧的+号, —>New repository，创建新仓库，如图：
![新建仓库.png](https://upload-images.jianshu.io/upload_images/1707644-ce02d14576627f30.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)   

仓库命名--博客讲的是博客搭建，所以我们创建仓库的规则是：   
> userName.github.io   

这里的userName就是你的github账号名，例如我的github账号名是shLuckySeven


![仓库命名.png](https://upload-images.jianshu.io/upload_images/1707644-74785eca40f24662.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 3)设置Github Pages(选择主题)
![主题选择和绑定域名](https://upload-images.jianshu.io/upload_images/1707644-7cf80471385e11c5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

选择一个主题，托管在GitHub上的博客就搞定了，例如我的个人博客：https://shLuckySeven.github.io/就可以访问了,注意：将其中的用户名换成你创建的仓库名   

##### 4)注册域名并绑定到仓库中（如需要）
这时候，你的确可以访问自己博客了，但是我们的博客访问地址是：userName.github.io   
如需专属域名，接下来，就需要我们去购买域名，配置成自己个性化的域名地址   

##### 购买域名：  
购买域名有很多途径，例如阿里云、腾讯云等，这里以阿里云为例，域名操作流程大概分为以下3步：
>- 选择域名   
>- 购买域名   
>- 域名解析   

1）选择域名   
选择你喜欢的域名，这里没事可说的，看自己，不差钱那就随意选，只要别人没注册   
***重点提示：***
但是一定要注意的是，选择的域名后缀一定是可以备案的，不然只能是注册保护，无法使用，在购买时，阿里云会有提示信息： 
![阿里云域名注意事项.png](https://upload-images.jianshu.io/upload_images/1707644-81498ca522be5a37.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2）购买域名   
购买就不多说了，实名认证、创建模板、购买，具体操作百度搜吧，很简单，或者在阿里云平台直接操作也可以   
3）域名解析 
  
- 添加记录  
 
![1-1.png](https://upload-images.jianshu.io/upload_images/1707644-d5f4e9ce6e95918b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 主机记录值填写
![1-2.png](https://upload-images.jianshu.io/upload_images/1707644-8474396b572ccd52.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里面主要区分两个概念：@ 和 www
   
> @:前缀省略   
> www:正常显示   
> 具体说明见截图，说的很清楚，我们解析域名就添加这两条记录值即可   
 
到这里，我们已经完成了域名解析工作，回到GitHub，根据我上面的那个配置Pages的截图完成域名和GitHub Pages 的绑定即可

## 二、初长成
**博客主题配置工作**
>找到我们之前创建好的本地存放博客项目的文件夹，找到_config.yml文件，打开 _config.yml,查看信息，参考Hexo官方文档，对文件里面的一些注释进行一些简单的修改先，主要配置以下地方：   

![config.png](https://upload-images.jianshu.io/upload_images/1707644-45cc4959aa7b5788.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![repo.png](https://upload-images.jianshu.io/upload_images/1707644-6c82f1856f632e2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里面介绍几个重要的即可：   
1.type:一定要配置成git
2.repo:写成你自己的博客仓库地址，格式按照我的，把username改成你的即可   

修改完毕之后，终端cd到你的存放博客项目文件夹下（如果之前cd过，应该一直在这）执行以下命令（按照顺序）：   
>hexo g   
>hexo s   

执行完 hexo s之后，控制台会给你输出一个地址
![local.png](https://upload-images.jianshu.io/upload_images/1707644-e37727849a0f0ade.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
把这个地址，贴到你的浏览器，进行查看，看到啥，你自己乐吧！

之后如果想发布到远程github上，根据上面我说的博客主题配置，_config.yml文件里的 repo，如果没有问题，执行   
> hexo g   
> hexo d   

即可部署到你的github Pages上，现在用你绑定的域名，或者userName.github.io再访问下试试！

## 三、总结：这一刻是开心的
不知不觉又深夜了，到了和大家说晚安的时间，我自己折腾这博客搭建的过程中，踩了好多坑，稍微晚几天，我找时间会把这块内容补进来，实在太困了！   
每天进步一点点，一个月后，两个月后，你会发现你已经提升了好几个台阶，回头再看，尽是美景！



