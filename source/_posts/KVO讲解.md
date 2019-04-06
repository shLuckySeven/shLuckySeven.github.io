---
title: 最不花里胡哨的KVO博客，一看就懂
date: 2019-04-06 22:54:33
tags: 底层原理
categories: iOS
---

**hi，I’m shuhuan，我的技术博客风格：简约、易理解，不花哨，花最少的时间掌握理解更多的知识点!** 

## 文章开篇，先介绍下本篇博客主要介绍的内容：
### 目录
>- 什么是kvo
>- kvo的使用场景
>- kvo的使用方法
>- kvo的底层原理
>- kvo如何简单添加对多个属性的监听
>- kvo使用需要注意的坑
>- …… 持续更新

1. 什么是KVO？
kvo其实是我们iOS开发中，苹果提供给我们的一个特别重要而且常用的核心技术，只是大多数简单业务开发的时候可能并不常用，但是但凡涉及到代码库的封装，或者工具类的封装，稍微高层次点的开发，就会涉及到kvo，那么kvo是什么？
> kvo 其实就是： k - v - o 

看到这有木有想打我？
别着急，往下看，老铁
##### K
> k就是Key，我们开发再常用不过的小可爱，字典、hash、通知、CAAnimation等，到处都用，我们把它理解成一个“唯一标识”，用一个标识，去替代它的身份，方便在它藏身的地方找到它

举个栗子：
我们熟知的 “工号9527”的那个录音笑话，您好，我是9527，很高兴为您服务……
也就是说这个9527，在客服的系统里，它是代表一个人，这个人比如叫：扫地僧,扫地僧是他的真实姓名，每个客服人员都有自己对应的被投诉的记录record，每被投诉一次，record就+1，以下要用到，但是他们对外服务并不使用自己的名字，多方面原因：方便客服系统管理、方便对外简易称呼（客户才不会动脑筋去记住那么多客服的名字）、客服人员的隐私等等，那么这个 9527 就是 这个扫地僧的“ Key”，这个key要保证在整个客服系统里面是唯一的，不然客户投诉，客服系统有两个9527，那该找谁呢，往下就不用我继续说了，key就是一个别名，但唯一，方便容器管理和查找。
##### V
> v就是Value，就是值的意思，每个key对应一个Value，Value可以根据需求和场景选择类型：int、NSNumber、NSString、id等等
kvo模式里value就是我们某个类的某个属性
以上的例子中，扫地僧的record，便是Value了，所以有如下对应关系：
key: 9527 -- Value: record

##### O
>o就是Observing（观察者），好了，到这，基本就有kvo的概念的雏形了
那么这个O，便是我上面例子中说的这个投诉的客户，他要监管或者能实时知晓这个工号（Key）为9527的员工扫地僧的record（Value），看看自己的投诉到底生效没，实时知晓这个词用的简直太灵性了

KVO概念总结：客户投诉Key为：9527这位客服，他要“实时知晓”9527的value：record是否真的改变，record的改变，客服系统会实时通知到这位客户系统对9527的record的判罚情况，这就是kvo的概念，一句话：observer对一个Key的Value的改变的实时监听，
也就是说：kvo是“键值监听”，可以用于监听某个对象属性值的改变

`注：我以上举的例子可能不太完全恰当，只是当时脑子第一时间想到9527的笑话，想着让读者更加容易理解，所以，较真的宝贝们不要和我较真就好了，谢谢理解！`

2. KVO的使用场景
说起使用场景，这个味道简直太浓太上头了，那么既然kvo这么神奇，那么我们在开发中，什么时候才会使用kvo呢？
这个主要看自己开发中的业务场景，比如我们要在某个类A，去监听引用进来的类B的某个属性值的变化，这个属性的变化改变后我们需要A类做出一些反应。例如Model类B的age属性，我们初始化在类A的lable上显示了这个age为10，结果这个age在某些时候改变了，那么用KVO监听这个age的变化，A作为监听者observer，就可以随时知道age的改变从而让A类的那个lable快速直接做出变化，而不用去A.lable.text =xxx重新赋值
比如：监听scrollView的contentOffset属性，来完成用户滚动时动态改变某些控件的属性实现效果，包括渐变导航栏、下拉刷新控件等效果。
除此之外KVO还可以监听更多业务场景，在这里就不多说

3. KVO的使用

```
- (void)viewDidLoad {
    [super viewDidLoad];
    Person *p1 = [[Person alloc] init];
    p1.age = 2;
    // self 监听 p1的 age属性，这里的self我们就可以理解为上述例子中的“客户”
    NSKeyValueObservingOptions options = NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld;

    [p1 addObserver:self forKeyPath:@"age" options:options context:nil];
    p1.age = 10;
    [p1 removeObserver:self forKeyPath:@"age"];
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context
{
    NSLog(@"监听到%@的%@改变了%@", object, keyPath,change);
}

// 打印内容
监听到<Person: 0x604000205460>的age改变了{
    kind = 1;
    new = 10;
    old = 2;
}
```
上面代码中可以看到，在给p1对象添加监听之后，age属性的值在发生改变时，就会通知到监听者，执行监听者的observeValueForKeyPath方法，当然这里我们是在同一个类里做的改变值演示，节目效果目的，真正使用kvo多数情况是跨类改变和监听的

参数和方法概念介绍：
> * NSKeyValueObservingOptions：监听策略，以上代码为新值和旧值都监听
> * removeObserver: 记得监听完事一定要移除
> *  (void)observeValueForKeyPath: 值改变的回调方法
> * change:监听对象的具体值情况

4.KVO的底层原理

- 简述原理：上述p1对象添加addObserver操作之后，p1对象的isa指针由之前的指向类对象Person变为指向一个通过Runtime动态创建的子类NSKVONotifyin_Person类对象，该子类拥有（重写）自己的set方法实现，set方法实现内部会顺序调用willChangeValueForKey方法、然后调用原来（父类）的setter方法实现、didChangeValueForKey方法，而didChangeValueForKey方法内部又会调用监听器的observeValueForKeyPath:ofObject:change:context:监听方法。也就是说一旦p1对象添加了KVO监听以后，其isa指针就会发生变化，因此set方法的执行效果就不一样了。

- NSKVONotifyin_Person内部调用逻辑
NSKVONotifyin_Person中的setage方法中其实调用了 Fundation框架中C语言函数 _NSsetIntValueAndNotify，_NSsetIntValueAndNotify内部做的操作相当于，首先调用willChangeValueForKey 将要改变方法，之后调用父类的setage方法对成员变量赋值，最后调用didChangeValueForKey已经改变方法。didChangeValueForKey中会调用监听器的监听方法，最终来到监听者的observeValueForKeyPath方法中。

- NSKVONotifyin_Person类重写方法的目的和实现
> 重写的方法有：
> +(Class)class
> -(void)setAge:(int)age:

这里NSKVONotifyin_Person重写class方法是为了隐藏NSKVONotifyin_Person。不被外界所看到。我们在p1添加过KVO监听之后，打印p1对象的class可以发现返回Person

> 如何手动触发KVO?
 被监听的属性的值被修改时，就会自动触发KVO。如果想要手动触发KVO，则需要我们自己调用willChangeValueForKey和didChangeValueForKey方法即可在不改变属性值的情况下手动触发KVO，并且这两个方法缺一不可。

5.kvo如何添加多个属性的监听
 思考下，如果我们要同时对多个属性进行监听，那么我们就需要添加N多个addObserver么？
例如我们涉及到属性嵌套，Person类里面有个Dog类属性，Dog类有age和level属性需要监听
当然不用！这就是我这节要讲的内容，往下看：
> 场景：Person类继承NSObject，Dog类集成Person类，有name、age、level等属性，在VC中添加对person.dog的属性kvo观察，如果dog的属性太多，无需多次添加addobserver进行重复代码，此时可调用一个函数：
```
-  (NSSet<NSString  *>  *)keyPathForvaluesAffectingValueForKey:(NSString  *)key{
    NSSet * keyPaths =[super keyPathForvaluesAffectingValueForKey:key];
    if ([key isEqualToString:@"dog"]) {
        keyPaths =[[NSSet alloc]initWithObjects:@"_dog.age",@"_dog.level", nil];
    }
    return keyPaths;
}
```
在这个函数里面做属性的判断和处理，返回一个NSSet集合，在监听的地方，直接对“dog”监听就可以了

6.KVO使用需要注意什么

> 1.使用完了记得要移除，不然会引起crash
2.key的容错处理
  如果用“key”的形式写key很容易出错，这里给大家说一个小技巧：
我们在写keypath的时候使用下面的方法去写，能规避一些错误
NSStringFromSelector(@selector(age))

### 个人宣传广告位：
Github：[shLuckySeven](https://github.com/shLuckySeven)