---
title: iOS【iOS编程实战】第二篇
date: 2023-07-16 14:43:03
tags:
- 图书
- 专业技术
- iOS
---

### 13

- 蓝牙LE，点对点通信，产生数据的叫服务端，消费数据的叫客户端。服务端常指蓝牙设备，客户端指iPhone。蓝牙设备广播自己，客户端扫描探测附近的外围设备。
- 扫描设备、连接设备、发现服务、发现特性、写入或读取特性。
- mac终端 uuidgen 命令可以生成一个uuid标识符
- 也可以写一个应用作为外围设备来广播消息。
- 扫描可以后台运行。电量考虑，外围设备的广播频率应随时间减少。
- 使用状态保存和恢复特性，蓝牙可以在应用被终止时，自动启动应用。

### 14

- 沙盒导致应用程序间通信很难。
- 沙盒目录，有Documnets、Library、temp、.app bundle. 推荐将内容保存在 Library 下创建的子目录。
- Documnets 存储文档、绘图文件。这类文件有命名，可以共享出来。
- Library 存储不可见数据，放在 Library/ApplicationSupport 目录下。会被自动备份。
- Library/Caches 不会被备份，但应用升级时会保留。
- temp 既不会备份，升级时也不保留。程序不运行时可能删除。
- data写入文件系统可以开启保护配置。(212页)
- SGKeychain 框架，可以设置密码和读取密码。存到钥匙串中。
- 对称加密AES可以用 RNCryptor，推荐用AES-128 即128位密钥长度。
- PBKDF2能将密码转为密钥。
- AES在iOS设备上，模式CBC（密码分组链模式），PKCS#7填充。
- 本章讲的密码学的相关知识并没有看懂。

### 15

- 避免根据设备型号字符串来检测硬件兼容性。
- 一般兼容两个版本的SDK就够了，因为大部分人更新系统都很快。
- 弱链接框架，可以选择性的加载框架。修改link binary with library 中的库，从required 改为 optional.
- 检查类可用性，判断类是否存在，if ([xxx class]){}
- 检查方法可用性，respondsToSelector:
- 检查C函数可用性，if(cfuntion != NULL) {}
- 检查麦克风可用性（229页代码）
- 检测摄像头可用性，是否存在前置摄像头，是否支持视频录像。检测闪光灯是否存在。（230页）
- 检测陀螺仪是否存在，指南针可用性。
- 检测是否视网膜屏幕，看scale 是否是2.（232页）
- 检测震动或发出提醒
- 检测远程控制（外接耳机按钮触发）（232页）
- 检测拨打电话功能，先检测是否有该功能，再决定是否显示拨打按钮。（UIApplication的 canOpenURL:）
- 发送短信和邮件的功能，两个对应的控制器，及是否可发送的属性，用来检查。没有配置邮箱账号和sim卡，是不能发送邮件和短信的。
- iPhone 5S 开始支持arm64指令集。64位，最低部署目标是 iOS 7 以上。
- 应用图标不用做圆角处理，系统会自动处理。
- tintColor 被多个系统控件使用，用来标记要突出的内容。
- 64位架构，避免用 int float ，应该用 NSInteger 和 CGFloat.
- #if \_\_LP64\_\_  宏 判断是否是64位架构。
- Info.plist 中的 UIRequiredDeviceCapabilities 键可以设置设备必须具备的功能，没有该功能的设备不能安装。

### 16

- 本地化可以在开发后期做，找好本地化服务商，他们可提供测试。
- NSLocalizedString 本地化字符串。genstrings 工具 可以自动创建本地化字符串文件。(243页)
- 一个检查没有本地化字符串的脚本（本章附件中）
- NSNumberFormatter 有些国家不是以小数点来分割整数和小数，有用逗号和撇号的。还有处理百分号。
- 货币转换（工程代码Money）
- nib文件本地化，用Base Internationalization ，从右向左读的语言需要特殊处理

### 17

- LLDB (Lower Lever DeBugger, 底层调试器)，可以编写LLDB脚本，使调试更容易。
- dSYM (调试信息文件) ，是编译过的代码和源代码之间的映射。在构建工程时自动创建。
- PHP和Python是脚本语言，HTML是标记语言。
- 将崩溃报告符号化，不能删除已提交归档文件xcarchive文件，它可以用来符号化崩溃文件。
- 可以将dSYM纳入版本控制，发布时提交。
- 异常断点，cmd+Y 禁用所有断点。
- 符号断点，可以输入方法名。有用的是 malloc_error_break 和 [NSObject doesNotRecognizeSelector:] 这两个符号，可以应对抛出 EXC_BAD_ACCESS 崩溃。
- 条件断点，满足特定的条件才会暂停。还可以设置动作，如播放声音。
- 控制台中打印，非对象变量，如 CGRect、CGPoint , 用 p (CGPoint) xx.
- register read 打印寄存器。
- 模拟器上 ecx 寄存器保存的是崩溃时调用的选择器名称。可以打印指定寄存器， register read ecx. 打印多个寄存器，用空格分隔。设备上是 r 开头的。
- eax 或 r0（设备上） 保存的是接收者的地址。ecx 或r4 保存最后调用的选择器，r1-r3 保存方法参数。sp、lr、pc 是r13-r15的别名。（261页）
- otool 配合 grep 命令 可以找出崩溃时执行到哪个方法 (261页)
- LLDB 可以导入 python 脚本，实现查找数组中指定元素。（262代码示例）
- NSZombieEnabled 僵尸对象检测，在edit scheme 中启用。调试内存问题。ARC出现之后，用到几率减小。
- EXC_BAD_ACCESS 访问已释放的对象或向他发送消息时出现崩溃。常出现的原因是用错了所有权修饰符。例如该用strong的用了assign
- SIGSEGV 不常见，导致原因常为不正确的类型转换。其他崩溃（264页）
- SIGABRT c函数abort，控制台会输出大量信息，输入bt命名打印回溯信息。
- 0x8badf00d 通常出现在同步网络调用阻塞了主线程。
- 可以自定义错误处理的c函数.(265)
- 可以用断言预测错误，结束程序执行。NSAssert(表达式, @"描述"); 可以禁用NSAssert。发布版本中建议禁用断言。现在发布版已默认禁用断言。
- 断言的使用，可以定义为宏。可以写在else分支和switch的default分支里。（266页）
- 异常用来处理不该发生的错误，应该结束运行。不建议用@throw和@catch块，使用异常不如使用断言。
- OC中ARC默认不是异常安全的，OC++中ARC是异常安全的。使用异常安全的ARC会导致性能下降。应避免大量使用OC++. -fobjc-arc-exceptions 编译标志可以使用异常安全的ARC.
- iTunes Connect 的崩溃报告，可以用dSYM文件进行符号化。上架的archive文件不要轻易删除，里面有dSYM文件。可以将归档文件纳入版本控制。可以在控制台中，将崩溃文件符号化。（269页）
- 可以选取第三方崩溃报告服务，有收费的，和免费的。需要集成，他们会将崩溃报告上传到服务器。

### 18

- Instruments 工具，product 中 profile 可开启。Allocations 工具。
- 应尽量避免频繁创建和释放内存，重用会比较好。
- Timer Profiler 工具，调优cpu性能。
- 将一段代码用 #define 宏处理，可以将代码内联处理，可提升性能。
- Accelerate 框架，苹果自带的，处理数学函数库集合。
- GLKit 集成了OpenGL.
- 编译器优化，默认-Os , -Ofast 和 -O3 比默认的快。非科学计算类应用-Ofast最快，科学计算类-O3更好。（283页）
- -Ofast 采取了一些对浮点数计算精度降低的优化，可能造成浮点数计算有轻微误差。所以苹果默认未打开。对于大多数应用，-Ofast 是个不错的选择。
- LLVM的LTO(链接阶段优化)，位于构建设置，Apple LLVM 5.0 -Code Generation 部分，中等体积的应用可开启，大应用会明显增加构建时间。
- Core Animation 工具可以 看帧率，帧率在绘图时应该保持稳定，不更新UI时应该比较低。其选项在debug菜单栏的viewdebugging里的rendering里。
- 网络使用成本跟时间相关，将网络活动集中在短时间内，这样设计比较好。

### 19

- UISnapBehavior 快速移动，damping 属性越高，移动越慢。默认0.5。移动到新的点需要移除并创建新的行为。
- UIDynamicAnimator 可以添加和移除行为，其被添加在视图上。
- UIAttachmentBehavior 附着行为。一个视图可以附着到另一视图上。
- 推力UIPushBehavior，使用 CGVector 向量产生推力。
- UIGravityBehavior 重力，一个动力学动画只能有一个重力行为。
- UICollisionBehavior 碰撞。
- 通过拖动手势和快速移动行为，实现拖拽移动视图。自定义 UIDynamicBehavior行为子类，通过addChildBehavior 为其添加行为。通过action观察其行为，持续调用。创建一个爆炸动画。（TearOff）
- UIDynamicAnimator 可以添加到集合视图的layout上。UICollectionViewLayoutAttributes 遵从UIDynamicItem协议，也就是说属性可以添加动力项。可以通过 layoutAttributesForElementsInRect 方法中替换掉属性。（CollectionDrag）
- 动力项可以是含有中心边界和变换的对象，可以是视图、集合视图布局属性。

### 20

- 视图控制器的 transitionCoordinator 属性，过渡协调器。
- 交互式过渡（滑动手势操作）过程中取消过渡，第二个视图的 viewDidAppear不会调用，notifyWhenInteractionEndsUsingBlock 方法无论过渡是成功还是取消，都会调用，可以在其block中做其他工作。
- 集合视图向集合视图过渡，设置属性 useLayoutToLayoutNavigationTransitions 为YES，有默认过渡动画。(工程UIViewControllerAnimatedTransitioning)
- 可以自定义过渡动画，遵从协议 UIViewControllerTransitioningDelegate、UIViewControllerAnimatedTransitioning，在协议方法中可以自定义动画。(工程CustomTransitionsDemo)
- 自定义交互式过渡动画，遵从协议 UIViewControllerInteractiveTransitioning ，继承 UIPercentDrivenInteractiveTransition 类，实现自定义动画类，可以更新百分比，完成和取消动画。（工程InteractiveCustomTransitionsDemo）

### 21

- 属性字符串，可以更改字体、颜色。也可以从RTF文件中读入属性化字符串。
- 用 enumerateAttribute 可以遍历属性字符串的属性，用 UIFontDescriptor 字体描述符更改为斜体等。（工程BeBold）
- 段落样式 NSMutableParagraphStyle 应用于 NSParagraphStyleAttributeName 键。（319页）
- 可以用URL来初始化属性字符串，读取HTML文件。避免在主线程中调用。也可以属性字符串中生成HTML，但这种转换丢掉附件，精度不好。（320页）
- 一些第三方库可以简化创建属性字符串，例如 MTStringParser 和 DTCoreText.
- 动态字体，在设备的设置里可调整。字体应该用 preferredFontForTextStyle 而不是指定字体大小。用户改变字体，有通知告知。（322页）
- Text Kit 的组件有，NSTextStorage 存储文本，NSTextContainer 表示区域，NSLayoutManager 应用布局。文本可以有多种方式布局，每种布局可以有多个区域。可以给布局设置多个区域。（工程DuplicateLayout和DoubleLayout）
- 可以用 UIBezierPath 给区域内挖洞，就是排除某一部分，使文本跨过该区域显示，用区域的 exclusionPaths 属性实现。（工程Exclusion）
- 通过继承 NSTextContainer 实现 lineFragmentRectForProposedRect 方法可以决定给定区域内，哪些地方能显示文字。（工程CircleLayout）
- 通过继承 NSTextStorage 可以批量给每个特定的子文本应用属性，如改变文字颜色。（工程ScribbleLayout）
- 绘制阶段，先绘制背景，然后是字形，然后是下划线和删除线。
- 通过继承 NSLayoutManager 可以在绘制阶段，批量对给定的每个子文本更改字形和背景等。（工程ScribbleLayout）
- 可以沿着贝塞尔曲线绘制文本（CurvyText工程）
- 使用 Core Text 绘制属性化字符串（SimpleLayout工程）。以及绘制多列展示、中间挖洞展示、形状内展示。（Columns工程）

### 22

- 观察者机制中，有委托、通知。使用kvo，性能较好。
- 使用kvc，通过 valueForKeyPath 传入字符串可以获取任何对象的任何属性。如果获取到的属性值类型是数字类型(int、float)，会自动转为NSNumber对象，如果是其他的非对象类型（结构体、指针）会转为NSValue对象。
- 设置属性，通过 setValue:forKey: 方法。
- valueForKeyPath 比 valueForKey 更灵活，前者可以包含嵌套关系，例如参数可以是 @"xxx.xx".
- valueForKey 寻找对应项时（numbers），会搜索getNumbers、numbers、isNumbers 找到这种方法就用其返回值。也会搜索 countOfNumbers、objectInNumbersAtIndex，kvc会生成代理数组（看起来numbers像是个数组）。其次会查找 countOfNumbers、enumeratorOfNumbers、memberOfNumbers 返回代理集合。最后查找 _numbers、_isNumbers、numbers、isNumbers 实例变量。
- 如果查找到实现了类似数组的实现方法，会生成并返回一个代理数组。将对数组的调用转移到kvc对象上。（工程KVC-Collection）
- setValue:forKey: 如果传入的值是nil 会抛出异常。
- 对数组调用 valueForKey: 会被传递给数组中的每一个元素，并将结果作为数组返回回来。多个消息可以用键路径传递 @"xxx.xx" 使用valueForKeyPath 方法。并将对每个元素进行多次消息调用的结果作为数组返回。（347页）
- 对数组元素属性应用操作符，如 @sum.length 就是对每个元素的length属性求和。这种用法在处理 CoreData 时有用，可以优化数据库查询操作，比等价的循环写法快。
- kvo 只要用存取方法来修改属性，观察机制就会自动生效，非常简洁。
- 添加观察 addObserver:forKeyPath:options:context: ，context通常传入 (void*)self 。移除观察 removeObserver:forKeyPath: 观察更新方法。（工程KVO）

``` objective-c
- (void)observeValueForKeyPath:(NSString *)keyPath
                      ofObject:(id)object
                        change:(NSDictionary *)change
                       context:(void *)context
```
- kvo如何实现的？依赖于两个方法 willChangeValueForKey 和 didChangeValueForKey ,这些方法调用会在中间插入。通过方法混写实现。这种方法注入在运行时实现。
- kvo的缺点，没有提供一个清除所有观察的方法。删除观察写错keyPath 会崩溃。出现bug难以解决。建议保守使用。对性能有要求的对象，可以用kvo代替委托和通知。

### 23

- 信号量用来限制并行任务的上限。小于0就阻塞了。可以在任何线程中使用。dispatch_semaphore_create 创建、dispatch_semaphore_wait 等待、dispatch_semaphore_signal 释放信号量。
- 可以将异步操作转为同步，创建信号量为0，执行等待，在回调中释放信号量。（工程SyncSemaphore）
- 分派源 dispatch_source_t ，可以向源分派事件，指定事件处理块。可以用作进度汇报。（工程ProgressReport）
- 可以用 dispatch_source_create 创建定时器源。（358页）
- dispatch_once 应用于创建单例，也可以初始化重要的常量，例如NSDateFormatter . 其块只会执行一次。初始化变量声明为 static ,表明这个变量只存在一个实例，本质是类变量。 dispatch_once 线程安全。如果已经执行过了，会被迅速跳过。
- 队列关联数据，dispatch_queue_set_specific ，支持队列层次结构，找不到值会沿着队列链向上找目标队列。（360页）
- dispatch_get_current_queue 需要避免使用，很危险。用 dispatch_queue_set_specific 关联数据，可以解决由此引发的问题，判断出指定的队列是否是当前的。（361页）
- 分派 I/O 当对需要对数据进行更多的控制时，就用到这些技术。
- 网络套接字编程一般使用流式的通道读取方式。通过分派数据、分派I/O 和底层的socket 和 connect 调用，可以实现HTTP下载数据。这些内容大部分都已经被包含在高级对象中，如 NSURLConnection ，如果需要非常严密的控制内存，其是正确的选择。
- 请求操作包括一个服务通道、一个文件通道、请求数据、返回数据的header和body分离等。

### 24

- OC对象其实是C结构体。ivar（实例变量）。Class结构体包含一个元类指针，元类存储类方法列表，元类是Class的isa指针。方法、属性、协议存储在类定义的可写段中，这些信息运行时可改变，分类基于此实现。ivar 存储在只读段，不能被修改，所以分类不能添加ivar.
- 运行时函数，class_copyMethodList、method_getName、sel_getName 。有copy 就要有free.
- objc_msgSend 性能并不差，没必要去替换它。
- NSInvocation 包含方法签名，其封装了一个方法的返回类型和参数类型。"@@:*"，第一个@表明返回值是id类型，@:表明该方法接受一个id和一个SEL，例如self和_cmd，_cmd表示选择器。最后的 * 表明参数为字符串 char * .
- 可以用@encode来获取表示该类型的字符串。 @encode(id) 结果是 "@"
- 创建NSInvocation对象（374页）
- 使用 methodSignatureForSelector 和 forwardInvocation 可以将消息转发给能处理消息的对象。（工程ObserverTrampoline）
- 消息转发流程（378页）
- resolveClassMethod 和 resolveInstanceMethod 在运行时提供实现，这是@dynamic 合成属性的方式。声明为@dynamic，编译器就不会自动合成一个ivar了。使用 class_addMethod 在运行时向类中添加方法即可实现。（工程Person）
- NSProxy 是轻量级的根类，遵循 NSObject 协议，NSProxy 避开了大部分难以代理的方法，例如需要本地运行循环的方法。
- 实现 NSProxy 子类，必须覆盖 methodSignatureForSelector 和 forwardInvocation 方法，否则会抛出异常。（工程Person）
- NSInvocation 类把目标、选择器、方法签名和参数打包在一起。forwardInvocation 是比较慢的转发方式，是转发链上的最后一环。
- 转发失败，会调用 doesNotRecognizeSelector 抛出异常。
- swizzling 方法混写。用分类实现混写，不要在+load方法中做。（工程MethodSwizzle）。调试可以用，不建议在产品中用。
- ISA混写，可以修改对象的类。（工程ISASwizzle）。kvo就是混写ISA实现的。
