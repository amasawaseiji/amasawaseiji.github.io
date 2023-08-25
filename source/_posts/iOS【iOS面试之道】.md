---
title: iOS【iOS面试之道】
date: 2023-08-20 15:32:24
tags:
- 图书
- 专业技术
- iOS
---

### 面试

- 程序员面试金典一书，LeetCode 刷题。
- 设计模式一书。

### 算法

- 给定整型数组和目标值，判断数组中是否有两数之和等于目标值。
- 给定整型数组中仅有两数之和等于目标值，求两数在数组中的序号。（31页）
- 给定字符串，按单词顺序反转。（33页）
- 给定链表和值x，小于x的值放左边，大于等于x的值放右边，原链表的节点顺序不变。（37页）
- 如何检测链表中是否有环，用快行指针，两个指针，一个移动快一个慢，如果两个相等就说明有环。
- 删除链表中第n个节点，用快行指针解决，速度相同，但两个指针相差n个节点，前一个移动到尾节点，后一个指针的下一节点就是要删除的节点。（39页）
- 栈，后进先出。队列，先进先出。
- 给出一个文件路径，要求将其简化。. 代表当前目录，..代表上级目录。（46页）
- 二叉树，二叉查找树，由递归定义的。二叉树的遍历，前中后序遍历、层级遍历（广度优先遍历）
- 设计一个展示二叉树的APP。（52页）
- 排序，归并排序、快速排序、桶排序。（54页）
- 排序中稳定的意思，即排序后的次序与未排序的次序相同。
- Swift 的sort函数，是内省算法，堆、插入、快速排序3种，依据深度选择最佳算法。
- 二分搜索，复杂度O(logn)。（58页）
- 面试题：会议时间重叠，怎么合并？排序后合并。（60页）
- 面试题：某个版本崩溃，后续所有版本都崩溃，找出第一个崩溃的版本。（62页）
- 搜索旋转有序数组（64页）
- tableView 插值排序，按时间排序。（65页）
- 深度优先搜索DFS 用递归，广度优先搜索BFS 用循环，配合队列。
- 前缀树，一种有序树。面试题：查找单词APP，给定字母矩阵，查出其中任意方向任意长度的单词。（73页）
- 动态规划，实现斐波那契数列。（75页）
- 书籍：算法导论、编程珠玑、剑指Offer

### 语言工具

- swift，class 是引用类型，在堆上。struct 是值类型，在栈上。
- 是函数式和面向对象编程语言。
- 泛型，代码满足任意类型的变量或方法。
- Open，在Module中被访问和重写。Public，Module中被访问。Internal，当前Module中被访问和重写。File-private，当前文件中使用。Private，作用域内使用。
- weak 和 Unowned ，后者被释放后，有无效引用，继续访问会崩溃。
- 值类型struct复制时，内存中是同一个对象，直到修改复制后的对象时，才会重新创建一个新对象。
- willSet 可获取newValue，didSet可获取oldValue.
- 结构体中修改成员变量的方法，加 mutating 关键字。
- @autoclosure 推迟计算到需要用时。（89页）
- 柯里化 Currying 特性，函数式编程思想。（90页）
- 求 0-100中为偶数且是其他数字平方的数字，函数式编程，map、fliter的使用。

```
(0...10).map {$0 * $0}.fliter {$0 % 2 == 0 }
```
- ARC在编译时管理内存，垃圾收集器在运行时管理内存。
- xcode中检查内存泄露的工具（93页）
- copy 修饰 NSString NSArray NSDictionary，是复制一份对象，指针指向不同地址。
- NSMutableString 不能用copy，这样修改时会崩溃。
- assign 修饰对象 会造成野指针，释放后指针地址仍然存在，堆上容易造成崩溃，栈上会自动处理。所以应修饰基本数据类型。
- atomic 速度慢，比 nonatomic 安全，但不是绝对的线程安全。需要用@synchronized 保证线程安全。
- runloop 每个线程仅有一个，一直运行，响应事件和消息。主线程的runloop默认启动，其他线程默认不启动。用 [NSRunLoop currentRunLoop] 来启动。
- __weak 用于修饰变量，防止block中的循环引用。
- __block 修饰block内部要修改的外部变量。
- block 适用于轻便简单的回调，如网络传输。代理适用于公共接口较多的情况，有利于解耦。block 运行成本高，栈复制到堆或是对象就加计数。代理只是回调对象指针，多个查表的动作。
- 用 NSInteger 代替 int, NSInteger 可以在32位和64位计算机中，代表不同的整型数据。int 只表示 32位整型数据。同理 用 NSUInteger 和 CGFloat 代替 unsigned、float.
- 属性顺序 原子性、读写顺序、内存管理语义。
- NSString 字面值相等，指针也相等，指向同一个内存。== 可以判定为真，这是编译器优化。
- NSOperationQueue 对象的 addOperationWithBlock 方法，不是在主线程中。
- 滑动列表时，timer 会暂停。是因为 runLoop 的 mode 被切换了，从默认模式切换到事件跟踪模式，而timer在默认模式，自然就停止了。5种模式。
- NSDefaultRunLoopMode 默认设置。NSConnectionReplyMode 用于处理NSConnection事件，一般用不到。NSModalPanelRunLoopMode 处理modal panels事件。NSEventTrackingRunLoopMode 拖拽和用户交互时的模式。NSRunLoopCommonModes 模式合集，包括默认、Modal、Event Tracking 3大模式，可以处理所有事件。
- 2种方法解决，一是将timer加入到 CommonModes ，二是加入到新开启的线程，然后开启该线程runLoop。（101页）
- swift 有函数式编程、面向对象编程、面向协议编程，注重值类型的数据结构，是静态性语言。oc只有面向对象编程，注重指针和索引，是动态性语言。
- swift的数组、字典、字符串都是值类型，在栈上操作，高效使用内存。也是为了线程安全。
- swift中将协议设计成可选（102页）
- 协议可以被class、struct、enum 实现，weak 关键字不能修饰值类型，protocol 前加 @objc 或协议名后加 class （104页）
- oc swift 混编，ProjectName-Bridging-Header.h 或 ProjectName-Swift.h 文件 会自动生成。
- swift初始化方法，convenience 提供便捷初始化方法，必须调用designated初始化方法来完成。 required 关键字强制子类重写父类中所修饰的初始化方法。
- swift中的协议（105）动态特性（107）
- 方法交换，应保证原子性和唯一性，尽可能在+load方法中实现，用dispatch_once 实现，保证只运行一次。
- isKindOfClass 和 isMemberOfClass ，前者判断包括子类，后者只包括该类本身的实例。
- 给分类添加属性，会报错，提示找不到getter和setter，因为分类不会自动生成这2方法，可以用关联对象实现。
- xcode Buildtime issues 展示警告、错误、静态分析错误（未初始化变量、未使用数据、API使用错误）， Runtime issues 线程问题（数据竞争，多个线程同时写）、UI布局和渲染问题、内存问题（泄露）
- APP启动时长优化，开启环境变量以及方法。（114页）
- 检测循环引用（115页）
- EXC_BAD_ACCESS 的处理定位，开启僵尸对象检测（117页）
- playground 执行异步操作、展示tableView (118页)

### 系统框架

- storyboard/xib 和纯代码相比的优缺点（120页）
- Auto Layout 和 Frame 区别，框架ComponentKit、Texture、LayoutKit（121页）
- UIView 对比 CALayer (122页)
- frame、bounds、center
- layoutIfNeeded、layoutSubviews、setNeedsLayout 比较(123页)
- safeArea safeAreaLayoutGuide safeAreaInsets 比较（124页）
- UIView Animation 、 CALayer Animation 、 UIViewPropertyAnimator 比较。最后者是iOS 10引进，处理交互式动画
- 多屏UI适用，Regular、Compact, 包括iPad上（128页）
- iOS 11 Drag and Drop，app内部拖动图片或跨APP拖动（iPad）（129页）
- collectionView 的 supplenmentary views 和 decoration views ，前者是补充视图、后者是装饰视图。补充视图用于section的header和footer ，装饰视图用于可设置背景。
- 列表视图滑动慢的优化（138页）
- 预加载问题，ASDK框架（139页）
- 实现瀑布流（140页）
- GET POST 对比，session 和 cookie ，4种网络通信协议（143页）
- HTTPS连接和响应的过程（144页）
- URLSession 4个类的比较（144）
- swift做一个网络请求（146）
- APNs推送机制
- swift 编码解码
- 数据持久化方案4种，Realm（152）
- 3种并发操作，串行并发、同步异步，以及示例
- 并发编程的3大问题，竞态条件、优先倒置、死锁（158）
- 代码中竞态条件检测，xcode中scheme中 Thread Sanitizer
- 竞态条件，解决方案3种（160页）
- GCD 中 dispatch_async、dispatch_after、dispatch_once、dispatch_group_t 4个方法说明
- GCD 中 global 队列 6种优先级
- 队列Operations，Operation, BlockOperation, OperationQueue ，操作有4种状态，BlockOperation 可以像 dispatch_group 一样管理多个任务。队列可以实现，暂停、继续、中止、优先顺序、依赖等复杂操作。可以设置最大并行数量。
- Operation 可以取消cancel，但是要根据其 isCancelled 状态，来暂停操作。
- 队列有对所有操作都取消的方法。（168页）
- 设计模式，7种3类。（169页）
- swift中单例创建
- 装饰模式，oc和swift中的应用，分类、代理、swift中的扩展。
- 观察者模式，注册时通知中心不会对观察者进行引用计数+1的操作。kvo，swift实现
- 备忘录模式，UserDefaults

### 5 经验之谈

- 5种架构，MVC架构，MVCS，MVP，MVVM
- VIPER 视图驱动的框架，有路由，层级间交互代码多，不适宜用在小型APP
- POP(Protocol Oriented Programming) 面向协议编程。OOP(Object Oriented Programming)面向对象编程
- swift实现二分搜索
- APP崩溃原因分析，模拟器和真机测试，单元测试、性能测试、UI测试、测试覆盖率
- appid codeSigning 应用瘦身，3种类型。审核被拒原因
