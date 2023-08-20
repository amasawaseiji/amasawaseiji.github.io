---
title: iOS【高性能iOS应用开发】
date: 2023-08-16 13:28:24
tags:
- 图书
- 专业技术
- iOS
---

### 1 移动应用的性能

- 应用间互操作和数据共享的机制，UIActivityViewController、MutipeerConnectivity 框架。
- 单点登录(single sign-on，SSO)
- Apple 提供了下载崩溃报告的服务，iTunes Connect中。用户可选择是否开启分享崩溃数据。设置里开启，针对所有应用生效。
- CocoaLumberjack 第三方库，有内置日期记录器。DDLogError、DDLogWarn、DDLogInfo、DDLogDebug、DDLogVerbose。可以向ASL(Apple System Log，NSLog方法的默认位置)记录日志，或向文件系统记录日志。（19页代码）

### 2 内存管理

- RAM 运行内存。线程有栈空间，栈帧消耗内存，方法中的变量占用栈内存，视图层级过多导致栈溢出。
- 1G RAM 分配给一个应用的堆大小不超过512M，操作系统管理堆。大量使用图片需要关注内存平均值和峰值。对象分配在堆中。栈变量可复制到堆，堆内容也可复制到栈。
- autorelease 用于延迟释放对象。@autoreleasepool 块结束时释放 autorelease 对象。事件循环迭代也在 autoreleasepool 块中。循环中嵌套使用自动释放池块降低内存峰值。
- 可以对某文件禁用ARC，编译器标记，-fno-objc-arc。
- ARC下不能使用retain、release、autorelease、retainCount 方法，不能调用dealloc方法，但可是实现，[super dealloc]也不行。但可以用CFRetain、CFRelease。id和void* 需要显示转换。
- __autoreleasing 使用于由id*传递的参数，预期 autorelease 会在方法内调用。常用于NSError， NSError * __autoreleasing * 。
- 属性限定符，retain 和 strong 一样（ARC下很少使用）。assign和 unsafe_untrtained 一样。copy暗指strong.
- 僵尸对象 NSZombieEnabled 环境变量。在scheme中开启。
- 代理用weak，如网络请求操作类，用完应将属性置nil，视图控制器释放时取消操作，在取消操作中将代理置nil.
- 块中用weakSelf 处理对self的捕获。
- NSTimer 对象导致的循环引用，可以在适当的时机将定时器 invalidate ，如视图控制器移出，或点击返回按钮时。也可以借由中间层使用弱引用解决循环引用的问题。(54页代码)
- 定时器的 invalidate 方法的意思是事件循环取消对定时器的引用。
- 观察者模式、通知中心，不会持有观察者、被观察者、上下文对象的强引用。
- 单例的使用场景，日志器、埋点服务、缓存。
- 跟踪内存使用情况的代码(66页)。

### 3 能耗

- tableView 控制数据载入规模，能展示数据的3到4倍最好，快速滚动时应推迟载入的时机，如滚动速度下降到某一阈值。
- NSOperationQueue 不会暂停或挂起正在执行的操作。完成后才会移出，挂起意味着不会启动新的操作，但执行中的操作不会停止。cancel 不会取消已经开始的执行。
- 定位的精度级别(75页)
- 后台运行被定位更新重启了应用，launchOptions里有键为 UIApplicationLaunchOptionsLocationKey 的值。
- UIApplication 对象的 idleTimerDisabled 属性设置为YES可以阻止屏幕休眠。
- UIWindow 对象和 UIScreen 对象 都可能不止一个，当连接外部显示器时，还可以切换屏幕。UIScreenDidConnectNotification 和 UIScreenDidDisConnectNotification 屏幕连接和屏幕断开的通知。(80页)
- UIDevice 的属性 batteryLevel(0到1浮点数)、batteryState(charging、Full) 可判断电池电量和是否在充电状态。batteryMonitoringEnabled 属性开启电池监控。
- 对cpu的使用率(84页代码)

### 4 并发编程

- NSThread 有栈空间大小的属性 stackSize .主线程的栈空间大小为1M。
- GCD的线程池上限64个，队列是先入先出
- 原子属性不能保证线程安全，只能阻止并行修改。两个线程同时修改，不能保证一个线程的操作是安全的。线程安全，应该是两个线程的操作是隔离开的。可以使用 @synchronized 锁包裹代码，保证一批操作的线程安全。加锁对象应为被修改属性等的对象。
- NSLock 锁，NSRecursiveLock 锁，可以多次加锁和解锁。NSCondition 锁，一个线程可以等待另一个线程释放锁。（98页代码）
- 读写锁，并行读取，但与写入互斥。用到GCD的屏障块。(101页代码)
- ReactiveCocoa库，oc的响应式编程库。RAC(代码112)
- Core Data 能保证数据一致性，但会导致性能损失。
- PromiseKit 框架，多层级调用，不会向右漂移代码。

### 5 应用的生命周期

- 应用状态流程图（120页）
- 推送通知，除了可能在 didReceiveRemoteNotification 中接受到，当应用未启动时，从通知中心打开APP，通知会出现在launchOptions参数中。
- iOS 8 didReceiveNotification:fetchCompletionHandler: 方法，如果实现会取代 didReceiveRemoteNotification 方法，只有应用运行时才会调，判断状态 inactive、background 。这个方法可以在应用非运行时调用，以此来启动应用，这个是静默推送。此方法可能会被调用两次，收到通知时(content-available)和点击通知进入APP时。
- 应用活动状态收到通知，要么不处理，要么给出恰当提示，如更新角标，应用内横幅。
- content-available 为1代表通知不应展示给用户，而是直接传递给应用，可唤醒应用。处理数据、远程拉取，创建本地通知。这种方法可加速应用响应。
- 后台拉取，设置间隔，和委托方法。使用 NSURLSession 配置后台会话，应用关闭，后台会话还可以继续。

### 6 用户界面

- 不要重写 loadView 方法
- 在viewDidAppear 中开始动画，在 viewWillDisAppear 中停止动画。
- 创建多个故事板或nib文件，避免一次性加载大文件。
- UITableView 避免对cell 调用 layoutIfNeeded 来布局。固定每个元素的尺寸。避免用透明的子视图，有性能损失。快速滚动时，拖动手势的速率可以获取。(160页代码)，避免渐变、图像缩放、离屏绘制
- 视图动画过程中，设置属性 shouldRasterize 为 YES，视图光栅化。
- 直接绘制比用xib复合视图，在性能上好一个数量级。应用稳定下来就可以换成直接绘制。
- 自动布局虽然比直接设定大小慢，但没有慢很多，是可以接受的。
- iOS 8 交互式通知，通知中心左滑通知或横幅下拉显示动作项，完成、稍后。使用 application:handleActionWithIdentifier:forLocalNotification 回调处理通知。
- iOS 扩展，今日窗口小部件，自定义键盘、分享、动作、照片编辑、文档提供者

### 7 网络

- CDN(content delivery network) 内容分发网络。提供静态内容，图像、字体、js、css等。
- HTTPS SSL握手验证服务器证书、共享密钥。请求头添加 Connection: keep-alive 下一次请求时可以复用。
- 移动网络，高速移动时 速度会比 在静止时速度慢。
- 判断网络状态采取不同策略，如低速网络不要对流媒体数据拉取，连续刷新设置阈值，检查是否已有一个网络请求。使用库发现网络状态的变化，给出提示。非wifi网络只展示视频预览图。对视频应用等，提供关闭自动下载或播放的功能。
- 流行的传输二进制格式，Protobuf.
- 设置里的开发者功能，可以设置网络调节器，模拟不同的网络条件。发布之前做测试。
- Charles 工具，抓包工具，可抓http、https请求和响应，发送自定义响应。

### 8 数据共享

- URL scheme，苹果保留的 scheme 有，http、https(Safari)、mailto(邮件) itms、itms-apps(App Store)、tel(电话)、app-settings(设置)。
应选择使用公司的标识符，保证别人不会跟你重复，你有解释的权利。出现重复的话结果不可预知。
- scheme 可以用 UIApplication canOpenURL: 方法来检测是否安装了应用。还可以用不同 scheme 来表示不同版本。openURL 打开应用。
- application:openURL:sourceApplication:annotation: 方法用来接收跳过来的链接，处理参数。
- 剪贴板 UIPasteBoard，可以存储多种类型的数据，还可以自定义剪贴板。可以存文本、NSData。有string属性和 strings 数组属性。(代码206页)
- 剪贴板在进程间通信，但不安全，应用进入后台清除剪贴板数据，设置 items 属性为 nil. 为防止复制粘贴，可以继承UITextView 做处理。
- UIDocumentInteractionController 可利用其他应用打开文档。支持预览、打印等，选择让别的应用打开等。（209页代码）
- 要想让应用支持打开特定的文档类型，需配置Info 里的 Documents Types ,配置一个UTI，文件类型标识符。
- UIActivityViewController 同一的服务接口，处理共享的数据操作。比使用UIDocumentInteractionController 更容易和灵活。可以共享 字符串、富文本、URL、图像、照片、UIActivityItemSource 符合协议的自定义对象。分享到的目标有内置应用和外部应用之分。（216页代码）可以共享一组数据，排除不想共享的应用。
- iOS 8 共享扩展，操作扩展，可取出共享数据中的图片（219页代码）
- 共享扩展，是共享活动里处理共享的数据操作的功能。（220页）
- 文档提供者，UIDocumentPickerViewController 文档选择器，需要 iCloud 授权。可以让文档阅读器从别的APP打开一个共享文档。（224页）文档提供者，UIDocumentPickerExtensionViewController 子类的使用（226页）。
- 应用群组有一个共享沙盒，可在多个应用之间共享数据。Capabilities 选项下 APP Groups 开启，设置群组id，与 NSUserDefaults 、NSFileManager 配合使用。(228页)

### 9 安全

- 关于应用安全的几本参考书或文章（230页）
- IDFV 供应商标识符，应用唯一标识符。用 device.identifierForVendor 获取，应用卸载重装会导致该值变化。
- IDFA 广告商标识符，是真正的唯一ID，可被重置。（232页获取代码）
- 应在钥匙串存储密码。应用密码PIN，应用的本地凭据。
- 使用 HTTPS 时，关闭TSL压缩。
- 仅信任白名单证书，NSURLConnection 验证。（238页）NSURLSession 也有相应方法。AFNetworking 库 也有处理 SSL 连接证书的方法。
- 数据保护功能，Data Protection , apple 账号上有针对应用id的数据保护功能，可以设置安全级别。（242页）访问数据需要设备解锁。
- 不要在非调试版本中使用 NSLog ,可以在设备导出日志中看到 NSLog 的输出。可以使用包装函数和宏或第三方库，如 CocoaLumberjack .（246页代码）
- 针对安全的测试清单（248页）

### 10 测试发布

- 单元测试，用测试宏 XCTAsset 断言是否为nil，是否相等。
- xcode 中可以开启 代码覆盖率选项，在Edit Scheme 中选择 Test 有选项 Gather coverage data .然后可以查看覆盖率报告
- 还可以生成 XML或HTML 格式的报告，需要开启一些设置以及安装工具包。（261）
- 异步方法测试，性能测试。
- OCMoke 框架 模拟依赖，类似造假数据以模拟网络数据获取方法。单元测试框架表（268页）
- UI测试自动化，Instruments 里的 Automation 工具，用于功能测试。用js编写测试用例脚本，还可以是命令行接口。
- 持续集成 Jenkins，Travis

### 11 工具

- accessibility Inspector 无障碍检查，针对残障人士
- Instruments 工具 各模板的介绍。
- 视图调试器，Debug View Hierachy 按钮激活。3D视角看视图层级。
- PonyDebugger 工具，无需连接开发机器，不需要暂停应用，就可以分析视图层级以及网络连接、查看日志等，输出到浏览器。（300页）
- Charles 工具，记录http和 https 请求的。

### 12 埋点与分析

- 第三方统计库，Flurry 以及一批。（312页）
- Aspects 库，跟踪事件，如跟踪 viewDidAppear: 方法。（316页）

### 13 iOS 9

- URL sheme 被限制，最多打开50个。
- openURL 可以打开http或https连接，应用能响应打开应用，不能响应打开 Safari. 用智能应用广告条引导用户下载应用。也就是 Capabilitiesl里的 Associated Domains 功能。创建 apple-app-site-association JSON 文件包含应用和关联路径。这样就能在应用和域名间建立信任。使用openssl 命令行工具对文件加密。（323页）
- 搜索功能，NSUserActivity 类新增方法及属性。Core Spotlight 框架，应用内容索引。（326页代码）
- UIStackView 可以定义布局是水平还是垂直，多个属性。（331页）
- SFSafariViewController 可以使用浏览器的 cookie 。（333页）
- 内容拦截扩展，与 Safari 集成，可过滤浏览网页的内容。（336页）
- Spotlight 索引扩展。
- 按需加载资源，Resource Tags 标签。主要是图片资源（342页）
- 开启bitcode 运行苹果对应用的二进制文件进行二次优化。

### 14 iOS 10

- SiriKit ，发送消息。需要申请权限。
- 通知，UNUserNotificationCenter 申请权限，通知触发器4种，时间间隔触发。携带附件，图片、音视频，但有大小限制，UNNotificationAttachment 。添加交互，UNNotificationAction，可以是按钮、文本。 （349页）
- 通知扩展，可以自定义通知显示的内容，仅能展示。UNNotificationExtensionGategory ，可以拦截通知。
- 通知服务扩展，通知消息里，mutable-content = 1,可以对通知消息更改，实现下载远程的附件，而不是只使用本地的，响应时间不能超过30秒。（353页）
- iMessage 扩展，使用表情包，图片音视频等。（354页）
- CallKit ，第三方APP可以做到和系统电话一致的体验，但是不提供通信服务。
