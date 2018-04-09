---
title: 【iOS编程】
date: 2017-08-27 21:27:23
tags:
- 图书
- 专业技术
---

非常基础的一本书，给初学者看的，比较厚，有500页，虽然是基于 iOS 7 的，但也不过时。

虽然是基础的东西，但是也有一些内容值得记录下来。

- #import 相比 #include 不会重复导入同一个文件。

- 初始化方法中应该直接访问实例变量，而不是调用存取方法。

- 头文件的排列顺序，实例变量、类方法、初始化方法、其它方法。

- @import 表示编译器会优化预编译头文件和缓存编译结果的过程，文件中不再明确引用框架，编译器会根据@import 自动导入相应的框架，只有 Apple 提供的框架可以使用@import

- 集合类保存的是指向对象的指针，而不是对象本身。

- nonatomic 不是默认的类型，所以要主动写出。读写的默认是 readwrite

- unsafe_unretained 用于纯量类型，表示会直接为实例变量赋值。在 ARC 之前，这种类型用的是 assign。unsafe_unretained 也用于对象类型，但是对象被销毁时，指针不会自动置 nil，而是成为空指针，而 weak 会。非对象类型，默认是unsafe_unretained，对象类型默认是 strong。copy 用于有可变子类的对象。使用 copy 具有防御性，防御其他程序员的修改。

- 存取方法都被覆盖，就不会自动生成实例变量了。如果要自定义属性的合成方式，使用@synthesize，使用@synthesize age; 这样生成的实例变量和方法名相同。

- view的 superView 属性使用弱引用避免循环引用

- hpyot 函数，求勾股的弦长

- 带有 ref 的也是对象，也是在堆上分配内存。使用带有 create、copy 的需要 release。

- 重绘 setNeedDisplay 后调用 drawRect，每个事件处理周期只调用一次 drawRect

- 子类无法访问父类在类扩展中的属性和方法

- 视图控制器创建的时候，view 属性为 nil，当需要展示的时候，才会调用 loadView 方法。

- 插座变量用 weak 是因为，内存不够的时候，视图控制器会释放其 view，如果不是 weak 的话，这些子视图释放不了。

- initWithNibName:bundle:是视图控制器的指定初始化方法。

- 委托方法的第一个参数应该是对象自身，是因为可能会有多个对象使用同一个委托，传入自身是给委托判断用的。

- 不能为协议创建对象、添加实例变量

- 静态变量和全局变量一样，不是保存在栈中的，所以不会再方法返回时释放

- @class 不需要知道细节，节省编译时间。

- removeObjectIdenticalTo: 和 removeObject: 区别，前者会枚举数组，向每一个对象发送 isEqual: 消息，后者只会比较指向对象的指针。

- UIViewController 对象默认有一个 editButtonItem 属性

- 摄像类型 kUTTypeMovie 加入 mediaTypes ，可以将视频存入相册，一个函数。kUTTypeMovie 是 ref 类型的，需要桥接成 NSString

- 长按手势默认识别持续0.5s 以上的触摸

- 基准线，大部分视图的基准线与底边相同。例外是 UITextField，其基准线是文本区域的底边

- UIImageView 固有内容的大小是显示图片的尺寸

- dismiss 应该是present 的那个控制器负责 dismiss，需要后面的控制器调用 presentingViewController 属性来获取。

- 模态时负责模态的是该控制器的 root 控制器

- 取沙盒目录是为什么返回的是 array 对象，因为 macOS 的缘故，可能会匹配到多个

- 锁屏进入的是未激活状态 后来运行10s 会进入挂起状态，内存过低时，会终止挂起状态

- 集合类中元素可写入的对象包括 NSString NSNumber NSDate NSData NSArray NSDictionary

- cell里面包含了 UIScrollView 因为可以左划删除

- Block对象是在栈中创建的，为了使方法返回时 block不被销毁，使用 copy 可以将 block拷贝到堆中，这样就不会在方法返回时被释放了。block 对捕获的 OC对象有强引用

- JSON(JavaScript Object Notation)

- URL 中不许出现空格和双引号，需要转义 stringByAddingPercentEscapesUsingEncodeing:

- NSURLRequest 默认使用 GET

- 归档的缺点是整存整取，coredata 可以增量读取，更新，删除，插入

- 进入后台会生成快照，启动时代替启动图片

- 状态恢复，没见过的，用到了再翻翻

- 本地化，国际化可以参考

- NSUserDefault 还跟设置里面对应的应用下的设置有关，没什么卵用

- 弹簧动画，一个是阻尼，从0到1，表示阻力大小，越大越不会弹。另一个是初始速度，通常是0，表示动力大小

- 可以从 UIColor 中取 rgb 的数值，有这个方法

- 仍有一些没写到的，但也可能记不住的，因为这个书多而不深。
