---
title: iOS【iOS编程实战】
date: 2023-07-08 11:27:05
tags:
- 图书
- 专业技术
- iOS
---

### 1、2

- oc对象和cf对象，如果没所有权的变化，就无需用桥接转换。
- UIKit Dynamics。创建 UIDynamicAnimator 对象，添加到视图上，再给子视图添加行为。集合视图也有动力学行为，信息应用里有。
- UIMotionEffect。对视图做动画，是设备动作的函数。
- tintColor。给应用着色，子视图用父视图的 tintColor，用方法 tintColorDidChange 来更新变化。
- 添加模糊图层，先截屏再加模糊效果（书籍附带的工程里，有代码示例）

### 3

- +initialize 分类中不要实现，因为主类可能也有实现，会使用那个就无法确定了（会覆盖）。但是 + load方法 则是每个分类都可实现，都会运行，但不该手动调用。
- 分类里声明属性，用关联对象实现其存取方法（代码26页），static char 类型变量做键。
- 关联引用，关联里被附着的对象销毁，附着的对象也会释放。这个特点可以做调试用，通过的得知附着的对象销毁，推断出被附着的对象也销毁了。
- 指针容器类，NSPointerArray、NSHashTable、NSMapTable 类似于 数组、set和字典。可以是持有弱引用，非对象的指针，其数组可存储NULL值。
- NSCache 搭配 NSPurgeableData 使用。通常内存吃紧，iOS会杀掉后台应用，使用NSPurgeableData 则会自己释放内存。
- CFStringTransform 可以实现中文转拉丁字母。去除重音符号（声调）。（代码29页）
- 子类调用父类的方法，父类方法里是返回父类对象，实际得到的是子类对象。用instancetype，表示返回当前类。init开头的方法也这样用。
- 可以控制需要百分号编码的字符，有方法。可以互相转换base64和NSData。
- firstObject，如果数组为空，会返回nil

### 4

- 故事板的缺点，多人开发合并冲突，难以解决。可以使用多个故事板，每模块一个。故事板可以自定义控制器之间的切换动画。

### 5

- 直接从文件系统取出UIImage对象比较耗时，应放在并发队列里做。collectionView里返回cell的方法里也应该这么做，不然可能会卡顿。同时用一个字典做缓存，优化性能。用字典和用NSCache类似，有键值。(书籍工程代码里有示例)。
- 根据横竖屏使用不同识别符的cell，类可以是同一个（因为仅是image尺寸不一样）。根据宽高谁更大，决定是竖屏展示的图片还是横屏展示的图片，在图片全部加载时就拿到方向数据，存储在数组里。(书籍工程代码里有示例)
- 设置选中时的背景视图。
- 实现石工布局，自定义UICollectionViewLayout的子类，并计算每个item的位置宽高（书籍工程代码示例有MasonryLayoutDemo）
- 封面浏览布局。（书籍工程代码示例有CoverFlowDemo）

### 6
- 关于自动布局约束，滚动视图怎么设置（66页）。使用可视格式化语言添加约束，不建议用，重构难。（67页）

### 7

- OpenGL ES 是 OpenGL 绘图语言的子集，主要用来编写3d游戏。
- 除了自定义视图，一般不需要调用setNeedDisplay方法，setNeedDisplay会在下一次绘图周期中重绘。只需要更新布局，可以调用。变化的视图会依次调用layoutSubviews方法。
- 视图可以通过子视图、图层、drawRect:方法来表现内容。
- 用贝塞尔曲线绘图，圆弧。（paths工程）
- rintf函数（四舍五入），π表示弧度，代表180度。0指向右，π/2下，π或-π左，-π/2上。
- 坐标系或线宽不是整数或半整数的，线条可能会模糊，即使在视网膜屏上也是。
- UIViewContentModeRedraw，视图大小改变时，重新调用drawRect:方法
- floorf向下取整。CGAffineTransformMake(sx, 0.f, 0.f, sy, dx, dy)，sx表示水平方法的缩放，sy表示竖直方向的缩放，dx表示水平方向的偏移，dy表示竖直方向的偏移。第二和第三2个参数是表示旋转的。（transforms工程）
- floorl同是向下取整，只是返回的数字类型不同。
- 绘制类似滚动的心电图，用CGPATH，时间轮询并重绘所有点，平移缩放绘制。（graph工程）
- UIKit和CoreGraphics 使用的坐标系不同，UIKit左上角为原点，CoreGraphics左下角为原点。使用 UIGraphicsGetCurrentContext 返回的上下文是正常的。如果用的是CGBitmapContextCreate 是左下角为原点。需要翻转（85页代码）
- CGContextSaveGState 和 CGContextRestoreGState 保存和恢复上下文，上下文里包含画笔颜色等。UIGraphicsPushContext 意为切换上下文。CoreGraphics不需要切换上下文，因为其将上下文视为参数。
- 使用 UIGraphicsBeginImageContext 获取绘制的图片。（Drawing工程）
- 避免重绘，图片要谨慎缓存，其重绘比缓存好。因为内存会飞速增加
- 可以在后台线程绘制，需要在自己的CGContext中，而不是主视图图形上下文。
- CGRectIntegral()函数返回可以包住给定矩形的最小整形矩形大小
- alpha 为1，着色，为0不显示视图。opaque，意思为使用全不透明的颜色，可以改善性能，使用一个非透明的backgroundColor 可以确保绘制所有像素，可以搭配opaque使用。hidden代表不会绘制，等同于alpha=0。动画中隐藏视图，用alpha值到0，因为hidden不能产生动画效果。
- 隐藏和透明视图不接受触摸事件。可以用alpha 为1，opaque为NO且backgroundColor为nil或clearColor的视图来接受触摸事件。

### 8

- UIView管理绘制和事件触摸，CALayer只关乎绘制。每个UIView都有一个CALayer用于绘制。
- 图层的contents属性为CGImage，UIView会在可能需要绘制的时候绘制，CALayer会在要求绘制时绘制。
- 图层的 contentsGravity 类似视图的填充模式。
- 做翻转动画（layers工程）
- 给可变字符串用字典填充属性，例如NSTextAlignmentCenter，可以使用NSMutableParagraphStyle。（layers工程）
- 截取某视图，用UIView的UISnapshotting,drawViewHieraryAtRect:方法
- 隐式动画，CATransaction（原子事务）。显式动画，CABasicAnimation，先创建对象然后添加到图层上。还有关键帧动画，CAKeyframeAnimation。
- 动画是创建图层的副本并修改，属于表示层。动画完成后显示的状态由模型层决定。
- 显式和隐式动画冲突时，关闭隐式动画。

``` objective-c
[CATransaction begin];
[CATransaction setDisableActions:YES];
//做显式动画
[CATransaction commit];
```
- fillMode。kCAFillModeRemoved 动画执行前后不会对值有影响。kCAFillModeBackwards 动画前保持fromValue状态。kCAFillModeForwards 动画后保持toValue状态。kCAFillModeBoth 前两者结合。通常使用用kCAFillModeBackwards 或 kCAFillModeBoth。
- 图层的position 意为中心点。CATransform3D 的 m34 属性，是摄像机的视角，通常值为 -1/2000 就可以。
- CATransformLayer 可以设置 zPosition，画正方体也可以用这个。(Box和BoxTransform工程)
- CALayer可以设置圆角、边框、阴影（Decoraion工程）
- 用actions属性为图层添加一组动画，为图层子类的自定义属性添加动画。
- 图层的@dynamic属性，CALayer会自动生成存取方法，所以不要在图层中实现自定义的存取方法。

### 9

- 应用程序由一个阻塞状态的do/while循环驱动，事件发生就把事件分派给监听器，处理分派的对象叫运行循环(run loop).每个线程都拥有自己的运行循环，动画也在后台线程运行。
- main.m 文件中的 UIApplicationMain 会运行主运行循环。NSTimer觊觎运行循环进行消息派发。
- 操作队列和操作，支撑优先级、依赖关系和取消。
- 用sysctlbyname计算cpu核心数。一个集合视图，用操作队列先显示分辨率低的图片，后显示分辨率高的图片。准备重用时cell会调用prepareForReuse（工程juliaOp）
- 分派队列无法取消，无法设置优先级，先进先出。
- 可以暂停和恢复队列，dispatch_suspend、 dispatch_resume.也会暂停以其为目标队列的队列。
- @synchronized 互斥锁，竞争很少时成本高。GCD屏障比较平衡。
- dispatch_group_notify 在组中没有任何要执行的块时，会立即调用。dispatch_group_wait 会阻塞线程，直到组执行完毕。

### 10

- CoreFoundation中 只要含有Create、Copy 就需要调用CFRelease.
- 给CFAllocatorRef 类型的参数传入NULL就表示使用默认的分配器。
- 调用CFGetTypeID 来获取cf实例的类型。
- CFShow可以在控制台打印调试信息。
- 用c字符串和Pascal风格创建CFStringRef。(工程里)
- 将CF字符串转换回c字符串，多种方法。(工程里)
- 创建cf数组和字典。(工程里)
- 其他容器，如CFTree等
- CF字典能容纳NULL做为值或键，而CF数组不行。
- NS对象到CF类型 可以用__bridge 也可以什么都不加。(CFArrayRef)nsArray.
- 在自己的代码中，遵守CF命名约定，然后用CF_IMPLICIT_BRIDGING_ENABLED 和 CF_IMPLICIT_BRIDGING_DISABLED 将函数声明包裹起来。
- 将CF对象转到NS对象，如果有赋值或返回值的情况 __bridge 就不够用了，
- \__bridge_transfer cf对象转到ns对象，交换所有权。\__bridge_retained，ns对象转到cf对象，交换所有权。
- 互相转换过后，都应将原变量置为nil或NULL.
- 不显示支持桥接的类型也能桥接为NSObject，可以把CFTreeRef放在NSArray中。(工程里)
- CoreFoundation 更灵活，对内存管理控制权更多。其容器可以不保留成员，还能存储整数等非对象数据。

### 11

- NSPurgeableData 是NSData对象，可以标记为可清除，用 endContentAccess，遇到内存压力时就会丢弃，即使应用是挂起状态。
- 应用进入后台，imageName:方法加载的图片会自动丢弃，再次使用时重新从磁盘加载。位图缓存也会自动丢弃，唤醒应调用DrawRect方法。UIImageView 不会丢弃数据。
- 后台运行时，必须停止更新OpenGL视图。applicationWillTerminate:方法调用之后，应用还会运行一小段时间，其实是出于后台状态，这时不应进行openGL调用，否则会立即终止应用。
- NSURLSession 可以配置为不用蜂窝网络。（143页）
- 大量数据传输，用上传和下载任务，其可以后台传输。下载任务完成后存在Caches目录，需要把用 NSFileManager 把文件转移出来。NSURLSessionDownloadTask 创建后，执行resume方法开始下载。其回调处理块在后台队列执行，需要回到主线程更新UI. 会话不再重用需要取消。(代码144页)
- 一个下载后台下载图片的示例（PicDownLoader工程）
- 处理后台传输，可以设定更新数据的的最短时间。
- 远程通知推送给用户的频率由苹果决定。（不是每发一条就立刻收到一条，可能会有延迟）
- 状态恢复的测试流程，按下home键，xcode中终止应用，再重新运行应用。两个方法可以记录状态恢复的日志。（148页）
- AppDelegate 里2个方法用于开启状态恢复系统。（148页）
- 每个视图控制器可以设置恢复标识符。以及几个UIKit视图也支持状态恢复，例如collectionView、tableView、imageView、scrollView、textView等。
- coreData也有唯一标识符，在进入后台时可以保存上下文。可以对数据库对象编解码。
- 几种控制器内容的状态恢复方法。tableView和集合视图可以遵循UIDataSourceModelAssociation协议。（FavSpotsRestore工程）
- 故事板设置状态恢复标志符比较方便，纯代码可以调用 setRestorationIdentifier: 方法设置。

### 12

- REST（Representational State Transfer 表述性状态转移），W3C（万维网联盟），SOAP(简单对象访问协议)，WSDL(Web服务描述语言)
- REST式服务器，使客户端可根据URL进行缓存。数据格式常用XML(可扩展标记语言)和JSON（JavaScript 对象表示法）.
- 解析XML，SAX解析器 NSXMLParser，给定一个URL，用代理式回调。
- DOM解析器，可以用XPath随机访问数据，不用使用代理委托。iOS中的DOM式解析可以使用基于libxml2的第三方库，如KissXML、GDataXML.DOM可以使代码简洁易读，推荐用DOM解析器。
- JSON解析，有苹果自带的 NSJSONSerialization 以及一些第三方库，建议用苹果自带的，不用JSONKit。可以用github上的 json-benchmarks 工程来比较各类框架的性能。
- JSON格式的数据其实在服务端也是对象序列化的结果。
- 使用 MKNetworkKit 这个第三方框架，从服务端登录以及获取数据的用法。（其他类似框架如AFNetworking）
- 为什么离开视图控制器要取消请求，可以使得下个控制器中存在队列中的数据请求更快响应。
- KVC处理json转模型，应编写一个基类处理KVC工作，少数工作子类来做。对模型复制，需要基类实现NSCopying和 NSMutableCopy 方法，子类覆盖实现自己的深复制方法。KVC实现主要是 setValuesForKeysWithDictionary 这个方法，可以把字典值赋值给类的属性。如果遇到模型缺少匹配键的属性的情况，需要在基类覆写setValue: forUndefinedKey: 方法，以防应用报异常。在其中打印出未识别的键就行。（模型映射框架如 Mantle 11.3k star 基于KVC）。（工程iHotelApp）
- 可以将服务器错误相关信息放在plist文件中，以便于本地化显示错误信息。
- 队列并发数最大值设置6，是因为服务器可能不允许同一个ip出现大于6个并发请求，7个以上会导致超时。wifi下不要超过6个，运营商网络限制。
- 缓存策略：按需缓存和预缓存。按需缓存是对每次请求的一块数据缓存起来，当数据不存在时才下载。预缓存就是全下载，稍后再看。按需缓存可用CoreData.
- 缓存数据保存在 NSCachesDirectory 可以为其创建独立目录，存在 Library/caches 文件夹下。如果存在Documents下会被上传至iCloud。
- 预缓存可用SQLite 或 Core Data 实现。
- NSKeyedArchiver 归档 可以缓存数据模型。遵循NSCoding协议。NSKeyedUnarchiver 反归档。
- CoreData 优势是可以实现独立访问模型属性。SQLite 比不上 coreData, 速度慢，非线程安全。
- CoreData 版本之间的数据迁移简单。
- 归档数据是写入到闪存的，频繁读写闪存会降低性能。可以利用内存和闪存结合的方式改善性能。内存即是将数据存在 static字典 里的形式，闪存是指存到文件系统。可以用最近最少使用策略存到闪存。并在系统内存警告通知时，将内存数据写入闪存。
- nginx.conf 文件 设置过期时间。缓存控制头，把下列结尾的文件缓存7天。

``` objective-c
location ~ \.(jpg|gif|png|ico|jpeg|css|swf)$ {
  expires 7d;
}
```
- 可以用URL缓存来缓存图片。
