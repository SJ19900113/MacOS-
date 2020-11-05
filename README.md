# MacOS-

## mac ox 操作系统


### I/O Kit

作用：编写驱动程序，内核扩展



对系统硬件进行抽象，预定义了多种类型的硬件基类。

iokit.framework (用户态，c语言) + 内核层框架kernel.framework（嵌入式c++，c++一个子集），driverkit



iokit包含两个数据库

1 i/o catalog  所有可用iokit类的注册表，不加载也有

2 i/o registry 跟踪iocatalog里面类对象实例，加载过的		ioreg命令查看



三个主要概念

1 家族 

 iousbfamily 处理usb相关设备支持的许多技术实现细节 ，定义了一类设备中通用功能的抽象。是动态加载的，系统需要的时候加载，不需要的时候卸载

2 驱动程序

驱动程序管理特定：设备或者总线

一个驱动程序可以与多个家族关联，比如usb存储设备，关联iousbfamily，iostroagefamily

当一个驱动加载时，与其相关的Families也将加载，这个关联关系在驱动的属性清单里定义

3 块nub

可以理解为线，一个驱动的连接点，一个Nub作为Family的一个部分被实例化，Nub就象是电视机与有线服务接口中间的连线，一个驱动为其控制的每个设备或服务发布一个Nub

http://www.yekki.me/iokit-fundamentals/ 有图

驱动一般至少会与两个Family发生关系，一个服务提供者，即：为设备提供服务的上游对象，以nub形式表示；一个是父类，即：此驱动类的父类。



kernel.famewrok 用于内核空间驱动程序开发，两个重要部分：

1 libkern库

 在kernel.framework/libkern文件夹

运行时 + osobject   osdictionary,osarray,osstring,osinteger, 内核扩展中的info.plist用，用户态plist用cf类型

2 iokit文件夹

包含内核空间驱动程序开发的头文件



iokit.framework 两个作用：

1 用户应用程序提供函数，检测硬件设备，为设备查找驱动程序，为驱动发送控制请求状态

2 支持用户空间应用程序直接和某些硬件通信。



platform expert

处理系统总线的设备枚举和检测，理解为主板的驱动程序 ，负责系统启动之后io设备树即ioregistry的初始化

ioplatformexportdevice为该树的根结点（上面有根类ioregistryentry）



info.plist 属性列表，包含匹配字典

/system/library/extentions 里面的kext，包里面plist里有

驱动匹配

1. usb 连接到电脑

2. 创建一个iousbdevice实例，表示这个设备
3. iokit 便利所有有匹配字典的驱动程序，条件是ioproviderclass 是iousbdevice
4. 如果配合字典中的内容和当前设备一致，就加入潜在驱动列表
5. 根据probe ，找出一个最合适的
6. 加载的时候会是实例化一个ioclass的驱动程序类的实例 （下文：如何写一个用户态驱动程序）

设备匹配

？



如何写一个用户态驱动程序

继承自要用的类，或者基类ioservice

1 xcode创建基于iokit driver模版的工程

1 可以选择性重写，init，start，stop，probe，free

2 在info。plist里面加匹配字典和库依赖





ioresouce块，可以用作无硬件设备驱动程序的提供者类

iomatchcategory，iokit允许一个块对每个匹配类别装载一个驱动程序





二. 应用程序与驱动交互

驱动程序 （串行设备的驱动，需要创建ioserialstreamsync的实例）

通过ioregistry 中的驱动程序路径找到



api：iokit

迭代硬件设备

1.创建匹配字典

2 匹配，创建出一个迭代器

3 迭代，拿到一些数据



监控状态，添加，移除事件

1 匹配字典

2 创建一个port  ，通过该渠道向用户空间传送通知消息

3 根据port创建一个runloopsource

4 把source加入到ioserviceaddmatchingnotification，为驱动程序设置好回调函数，轮询



移除要注意，回调函数之前设备已经移除，需要一个结构体来保持对象



***还是基于自定义的驱动程序。 驱动与硬件交互的两种方式

1 。修改设备驱动程序属性

iokit驱动程序都有一个属性表，任何用户态应用程序都可以访问，可以添加新的keyvalue，也可以修改

IORegistryEntryCreateCFProperty 提供一个驱动程序属性表的cfdic

ioresgistryentrysetcfproperty() 修改健值。 需要驱动程序重写setproperties方法

2 连接

iouserclient ，每个连接，都会实例化一个iouserclient对象，

需要在驱动程序info.plist中  iouserclientclass添加一个字符串值，表示用户客户端类的类名

a 建立连接 ioserviceopen() , 返回一个连接对象（本身也是继承ioservice），初始化usercline的时候会保持应用程序task

b 定义一个接口，将驱动程序的功能公开给用户空间应用程序。（一些函数库，用户空间重写来编码参数，userclient重写来解码，之后传给驱动程序）

c iokit提供一些函数来给应用程序调用userclient 

​		ioconnectcallscalarmethod（）

​        ioconnectcallstructmethod （）可以传结构体

​        ioconnectcallmethod （）上面两者 

d 用户程序调用之后，userclient externalmethod（）接收，然后路由到userclient里面重写的一个调度表



e 异步操作，iconnectcallasyncmethod ，可以在用户空间创建一个port，当参数传给userclient

ioserviceclose，应用程序与驱动程序断开连接
