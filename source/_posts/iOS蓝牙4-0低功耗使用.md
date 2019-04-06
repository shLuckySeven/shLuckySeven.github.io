---
title: iOS蓝牙4.0低功耗使用
date: 2017-07-06 22:23:51
tags: 蓝牙
categories: 开发积累
---

iOS 蓝牙4.0开发使用

嗨大家好，我是iOS开发一枚大帅比，过去的几年项目涉及到蓝牙比较多，抽空之余，把使用的小知识点归纳起来，一是方便自己对知识做很好的总结和复习，二是希望能帮助到琅琊开发的朋友们。

一、简介（什么是BLE4.0？哪里使用它？）

现在的互联网时代，智能硬件设备越来越多样化，这些设备中，有多是通过手机来控制硬件设备，来达到控制的效果，这中间少不了要使用到蓝牙功能，通过蓝牙来通信来控制设备。也就是我们说的“设备中心和外设的故事”。

蓝牙版本介绍：

每个人对于蓝牙都不陌生，近距离数据传输，方便；可是当你的业务需求需要你第一次接触蓝牙开发的时候，却会发现你对它并不了解；首先，蓝牙发展至今经历了8个版本的更新。1.1、1.2、2.0、2.1、3.0、4.0、4.1、4.2。那么在1.x~3.0之间的我们称之为传统蓝牙，4.x开始的蓝牙我们称之为低功耗蓝牙也就是蓝牙ble，当然4.x版本的蓝牙也是向下兼容的。android手机必须系统版本4.3及以上才支持BLE API。低功耗蓝牙较传统蓝牙，传输速度更快，覆盖范围更广，安全性更高，延迟更短，耗电极低等等优点。（现在的穿戴设备都是使用BLE蓝牙技术的）

传统蓝牙与低功耗蓝牙通信方式也有所不同，传统的一般通过socket方式，而低功耗蓝牙是通过Gatt协议来实现。

本文章目的便是介绍BLE 4.0的使用以及相关问题的解决，本文采用简要模式介绍BLE4.0的核心类的使用以及蓝牙开发的简介，如需了解蓝牙知识的详细知识点，百度可搜索其他大神的文章进行学习，下文通用BLE代为蓝牙4.0。

二、BLE的核心模式

BLE的两种模式分为CBCentralMannager 中心模式 和CBPeripheralManager 外设模式，在这里主要和大家分享CBCentralMannager 中心模式的开发和使用。

CBCentralMannager 中心模式

以手机（app）作为设备中心，连接其他外设（硬件）的场景。详细流程如下：

1、建立中心角色

2、扫描外设

3、发现外设

4、连接外设：

    1.连接失败

    2. 连接断开

    3.连接成功

5、扫描外设中的服务

    1.发现并获取外设中的服务Service

6、扫描外设对应服务的特征

    1.发现并获取外设对应服务的特征Characteristic

    2.给对应特征写数据

    3.订阅特征的通知 7.1 根据特征读取数据
说明：Service，Characteristic是每个硬件设备出厂设定的设备服务和特征值，用UUID作为唯一标识符。UUID为这种格式：0000ffe1-0000-1000-8000-00805f9b34fb。比如有3个Service，那么就有三个不同的UUID与Service对应。这些UUID都写在硬件里，我们通过BLE提供的API可以读取到，一个BLE终端可以包含多个Service， 一个Service可以包含多个Characteristic，一个Characteristic包含一个value和多个Descriptor，一个Descriptor包含一个Value。Characteristic是比较重要的，是手机与BLE终端交换数据的关键，读取设置数据等操作都是操作Characteristic的相关属性。说白了，我们可以把每个服务特征值看成是我们网络请求的api接口，我们和蓝牙的交互，是要连接到对应的service和对应的Characteristic才可以正确读取到和硬件文档以及出厂定制好的服务和数据。

三、CBPeriphera

暂且把这个小可爱理解为你所能发现搜索到的外设（硬件）对象，蓝牙硬件的信息都包含在它里面

下面我们看下这个类所包含的内容：

CB_EXTERN_CLASS @interface CBPeripheral : CBPeer

//代理

@property(weak, nonatomic, nullable) id<CBPeripheralDelegate> delegate;

//我们所能搜索到的外设名称

@property(retain, readonly, nullable) NSString *name;

//信号强度

@property(retain, readonly, nullable) NSNumber *RSSI;

//设备状态，包含以下4种状态

@property(readonly) CBPeripheralState state;

typedef NS_ENUM(NSInteger, CBPeripheralState) {

CBPeripheralStateDisconnected = 0,

CBPeripheralStateConnecting,

CBPeripheralStateConnected,

CBPeripheralStateDisconnecting

}

//服务群组

@property(retain, readonly, nullable) NSArray<CBService *> *services;

由于函数和属性太多，我只列举主要的几个，其他的可在代码中点击查看即可
四、使用步骤概要

1、导入：#import <CoreBluetooth/CoreBluetooth.h>

2、遵守CBCentralManagerDelegate,CBPeripheralDelegate协议

3、初始化中心Manager，创建中心角色

为了方便引用，一般把manager声明为一个属性对象：

@property (nonatomic, strong) CBCentralManager * centralManager;

 self.centralManager = [[CBCentralManager alloc] initWithDelegate:self queue:nil];

注：queue为你想把蓝牙初始化放到哪个队列，一般默认主队列

        options传nil即可，具体使用可查看策略含义
4、检查当前设备中心的蓝牙状态

// 状态更新时调用

- (void)centralManagerDidUpdateState:(CBCentralManager *)central

{

    switch (central.state) {

        case CBManagerStateUnknown:{

            NSLog(@"为知状态");

            self.peripheralState = central.state;

        }

            break;

        case CBManagerStateResetting:

        {

            NSLog(@"重置状态");

            self.peripheralState = central.state;

        }

            break;

        case CBManagerStateUnsupported:

        {

            NSLog(@"不支持的状态");

            self.peripheralState = central.state;

        }

            break;

        case CBManagerStateUnauthorized:

        {

            NSLog(@"未授权的状态");

            self.peripheralState = central.state;

        }

            break;

        case CBManagerStatePoweredOff:

        {

            NSLog(@"关闭状态");

            self.peripheralState = central.state;

        }

            break;

        case CBManagerStatePoweredOn:

        {

            NSLog(@"开启状态－可用状态");

            self.peripheralState = central.state;

            NSLog(@"%ld",(long)self.peripheralState);

        }

            break;

        default:

            break;

    }

}
5、开始扫描

if (self.peripheralState == CBManagerStatePoweredOn){

[self.centralManager scanForPeripheralsWithServices:nil options:nil];

}
6、发现外设

/**

扫描到设备

@param central 中心管理者

@param peripheral 扫描到的设备

@param advertisementData 广告信息

@param RSSI 信号强度

*/

- (void)centralManager:(CBCentralManager *)central didDiscoverPeripheral:(CBPeripheral *)peripheral advertisementData:(NSDictionary<NSString *,id> *)advertisementData RSSI:(NSNumber *)RSSI

{

    NSLog(@"%@",[NSString stringWithFormat:@"发现设备,设备名:%@",peripheral.name]);

}
7、连接外设

[self.centralManager connectPeripheral:peripheral options:nil];
      连接状态

/**

@param central 中心管理者

@param peripheral 连接失败的设备

@param error 错误信息

*/

** 连接失败 **

- (void)centralManager:(CBCentralManager *)central didFailToConnectPeripheral:(CBPeripheral *)peripheral error:(NSError *)error

{

    NSLog(@"%@",@"连接失败");

}

** 连接断开 **

- (void)centralManager:(CBCentralManager *)central didDisconnectPeripheral:(CBPeripheral *)peripheral error:(NSError *)error { 

NSLog(@"%@",@"断开连接");

 }

** 连接成功 ** 

- (void)centralManager:(CBCentralManager *)central didConnectPeripheral:(CBPeripheral *)peripheral { 

NSLog(@"连接设备:%@成功",peripheral.name);

 [self.centralManager stopScan]; 

}
8、扫描外设服务Service

// 设置设备的代理

peripheral.delegate = self;

// services:传入nil  代表扫描所有服务

[peripheral discoverServices:nil];
/**

扫描到服务

@param peripheral 服务对应的设备

@param error 扫描错误信息

*/

- (void)peripheral:(CBPeripheral *)peripheral didDiscoverServices:(NSError *)error

{

    // 遍历所有的服务

    for (CBService *service in peripheral.services)

    {

        NSLog(@"服务:%@",service.UUID.UUIDString);

    }

}
9、扫描service特征值

kWriteServerUUID 为和厂家协定好的，

例：读取电量的UUID：08B07DCB-13DD-435C-8BDA-04DA65BDC6AA

// 获取对应的服务

        if (![service.UUID.UUIDString isEqualToString:kWriteServerUUID])

        {

            return;

        }

        // 根据服务去扫描特征

        [peripheral discoverCharacteristics:nil forService:service];
10、给特征值写数据（就是发送指令）

[peripheral writeValue:data forCharacteristic:characteristic type:CBCharacteristicWriteWithResponse];
11、订阅特征值通知（订阅后可收到回应的数据）

if ([characteristic.UUID.UUIDString isEqualToString:kNotifyCharacteristicUUID]){

  [peripheral setNotifyValue:YES forCharacteristic:characteristic];

}
12、根据特征值读取数据（注：应把之前发送数据和订阅通知的peripheral和characteristic定义为全局属性）

*该处即为接收到外设发送的数据的方法

/**

根据特征读到数据

@param peripheral 读取到数据对应的设备

@param characteristic 特征

@param error 错误信息

*/

- (void)peripheral:(CBPeripheral *)peripheral didUpdateValueForCharacteristic:(nonnull CBCharacteristic *)characteristic error:(nullable NSError *)error

{

    if ([characteristic.UUID.UUIDString isEqualToString:kNotifyCharacteristicUUID])

    {

        NSData *data = characteristic.value;

        NSLog(@"%@",data);

    }

}

到这里，整个蓝牙使用的核心流程就基本介绍完了，鉴于写博客文章太累，太耗时，我决定先睡觉去，回头把文章慢慢细化一下，查漏补缺，格式可能也不是很完美，请读者们见谅，多提宝贵意见，喜欢的老铁们，可以点下喜欢或者收藏！

打个广告

本人github地址：shLuckySeven

欢迎去点star

