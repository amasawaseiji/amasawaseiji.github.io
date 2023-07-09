---
title: iOS【多线程和内存管理】
date: 2023-02-26 18:28:02
tags:
- 图书
- 专业技术
- iOS
---
### ARC
- ARC(automatic reference counting)，自动引用计数，编译器进行内存管理，降低程序崩溃，内存泄露。
- NSObject (+alloc -retain -release -dealloc)
- 自己生成对象，自己持有。(alloc、new、copy、mutableCopy)，这些方法以及用这些方法开头的方法。非这几个或开头创建的对象使用autorelease，可以创建但不持有
- copy、mutableCopy:利用NSCopying方法copyWithZone:方法和NSMutableCopying方法mutableCopyWithZone:方法。也生成并持有对象，区别在于生成的对象可变更和不可变更。
- retain、release、autorelease.
- autorelease,不立即释放，注册到autoreleasepool中，pool结束时释放。
- 释放非自己持有的对象，会崩溃
- -retainCount 获取引用计数，release之后计数为0会执行dealloc方法
- oc实现，引用计数和对象内存地址存储在散列表里，通过散列表能找到各引用计数和对象的地址。
- 废弃NSAutoreleasePool时，调用过autorelease方法的对象都将调用release方法。[pool drain]
- NSRunLoop程序主循环，对NSAutoreleasePool对象生成、持有、释放
- for循环可能会产生大量autorelease对象，可适当生成和释放NSAutoreleasePool对象
- IMP Caching:取类名，方法名，函数指针。运行效率快
- pool嵌套，使用的是最内层的pool执行autorelease
- autorelease对象 实际是被加到pool对象的数组里，drain时会对数组中所有对象执行release
- NSAutoreleasePool类方法showPools检查对象是否已经被autorelease
- 不能对NSAutoreleasePool对象执行autorelease，会抛出异常，因为NSAutoreleasePool已经重载了autorelease实例方法
- id相当于c语言中的void *
- `__strong、__weak、__unsafe_unretained、__autoreleasing` 4种修饰符
- `__strong`ARC有效时默认修饰符，不写就默认有,id __strong obj,:(id __strong)obj（做参数时）
- `id __strong obj;`等同于`id __strong obj = nil; `，`__weak、__autoreleasing` 也同样
- `__weak`是解决循环引用问题的，循环引用会导致内存泄露，内存泄露就是对象超出生存周期依然存在
- 自引用，持有自身。
- 使用`__weak`不会持有对象。直接用`__weak`引用编译器会告警，应该先用`__strong`引用，再赋值给`__weak`变量
- `__weak`引用的对象被释放，则其修饰的变量会自动置为nil
- `__unsafe_unretained`是`__weak`在iOS5以下版本的代替
- `__unsafe_unretained`也不持有对象，直接使用会告警。其引用的对象被释放，其修饰的变量不会自动置为nil。所以访问`__unsafe_unretained`修饰的变量，有可能会崩溃，需要确保其引用的对象存在，才没问题。
- ARC有效时，不能使用autorelease方法，也不能用NSAutoreleasePool类。
- ARC有效时，用@autoreleasepool {} 块替代NSAutoreleasePool类生成持有废弃。用`__autoreleasing`修饰变量代替调用autorelease方法。显示使用`__autoreleasing`同显式使用`__strong`罕见
- 编译器检查方法名是否以（alloc、new、copy、mutableCopy）开头，决定是否将返回值的对象注册到自动释放池。
- 在访问`__weak`修饰的变量时，实际会将要访问的对象注册到自动释放池，以防因为是弱引用，而对象又被废弃了
- id的指针或对象的指针，`id *obj` 和 `NSObject **obj`,实际会被附上`__autoreleasing`修饰符变成`id __autoreleasing *obj`和`NSObject* __autoreleasing *obj`
- `NSError *error = nil; ` `&error` 等同于 `NSError **`类型
- 使用`__autoreleasing`修饰符的变量作为参数，也会注册到自动释放池
- 赋值给对象指针时，所有权修饰符必须一致。`NSError *error = nil;
NSError * __strong *pError = &error; `不加`__strong`编译器会报错。同理，`__weak`和`__unsafe_unretained`也一样`NSError __weak *error = nil;NSError * __weak *pError = &error; `
- 非显式声明的`__strong`变量作为方法参数中的对象指针类型，会被编译器自动转换为`__autoreleasing`
- 对象指针作为参数虽然可以声明为`__strong`类型，但最好不要这样做，因为这不符合内存管理的思考方式。还是以`__autoreleasing`好
- `__autoreleasing`修饰变量，必须为自动变量（局部变量、函数、方法参数）
- @autoreleasepool {} 也能嵌套使用，ARC无效其也能使用，推荐用@autoreleasepool
- NSRunLoop实现，随时能释放自动释放池中的对象
- _objc_autoreleasePoolPrint()，调试用的非公开函数
- ARC下，不能用retain/release/retainCount/autorelease.使用会出现编译错误，dealloc也是
- ARC下，除(alloc、new、copy、mutableCopy)还有init需返回自己持有的对象。特例 - (void)initialize;
- dealloc适用于释放c语言库分配的内存、删除代理、观察者对象。
- ARC下，dealloc里不能调用 [super dealloc]
- oc对象不能作为c语言结构体成员，会引起编译报错，如果要用，可转为void *或加`__unsafe_unretained`修饰，因为其不属于编译器的内存管理对象，但用这个有隐患，可能造成程序崩溃或内存泄露
- ARC下，void * 和id转换，需要用`__bridge`,`void *p = (__bridge void*)obj;id o = (__bridge id)p`,这样转成void*安全性也很低，例如空指针导致崩溃
- `__bridge_retained`赋值时增加持有对象，与retain相似
- `__bridge_transfer`赋值时释放持有对象，与release相似
- 这些转换多数用于oc对象与core foundation（c语言编写框架）对象转换时
- CFRetain/CFRelease,CFBridgingRetain()和`__bridge_retained`一样的作用,CFBridgingRelease和`__bridge_transfer`一样的作用，做转换时必须恰当使用
- 属性修饰中，copy、retain、strong 相当于给使用`__strong`修饰符的变量赋值。assign、unsafe_unretained 相当于`__unsafe_unretained`，weak相当于`__weak`
- 属性中带copy不是简单的赋值，是复制对象
- C语言中使用动态数组，`id __strong *array = nil;` `__strong`必要显示声明，不显示默认为`__autoreleasing`,初始化必须为nil，指针类型不会自动置nil。分配内存用calloc函数，可使分配区域初始化为0(malloc函数还需用memset函数将内存填充为0，malloc不可将数组元素置nil，可能导致释放不存在的对象，推荐用calloc函数)，释放时还必须释放每个数组元素，不能仅释放数组内存。动态数组不归编译器管，静态数组归编译器管。需要将数组元素置nil再释放数组。memcpy和realloc也不要用，会引起内存泄露和重复废弃。`__weak`可修饰动态数组，`__autoreleasing`不可，`__unsafe_unretained`可用于C语言的指针
- objc_retainAutorelesedReturnValue和objc_autoreleaseReturnValue函数不将对象注册到自动释放池直接传递，最优化
- `__weak`修饰的变量引用的对象如果被释放，则变量自动置nil.使用`__weak`修饰的变量，会自动注册到自动释放池
- weak散列表，objc_storeWeak函数
- 如大量使用`__weak`修饰符的变量，会消耗cpu资源，所有只在要避免循环引用时使用
- `__weak`不能修饰刚创建的对象，因为会立即被释放，并将变量置nil
- `__weak`修饰的变量被使用时，实际会先retain对象，并注册到自动释放池。多次使用多次注册，最好先赋值给`__strong`修饰的变量再用
- NSMachPort类不支持`__weak`，有自己的引用计数机制，使用的话编译器会报错
- allowsWeakReference返回NO程序将异常终止、retainWeakReference返回NO，将无法获取`__weak`引用的对象，将返回nil
- _objc_rootRetainCount函数获取计数数值，但有时是不准确的，多线程竞争状态、对象被释放、不正确的地址等

### Blocks

- 带局部变量值的匿名函数
- 范式：^ 返回值类型 参数列表 表达式
- 返回值类型可省略，要么是表达式return语句返回值的类型，要么是void类型
- 参数列表可省略，使用参数时，^{}（2个都省略）
- 函数指针：
`int func(int count)
{
  return count + 1;
}
int (*funcptr)(int) = &func;
  `
  函数指针调用并赋值：`int result = (*funcptr)(10);`
- blockl类型变量：`int (^blk)(int) = ^(int count){return count+1;};` `int (^blk1)(int) = blk;`
- 可做函数参数、返回值使用，记述方式复复杂，可用typedef解决
- `typedef int (^blk_t)(int);` `void func(int (^blk)(int))`变为`void func(blk_t blk)` `int (^func())(int)` 变为`blk_t func()`
- block类型变量指针：`blk_t *blkptr = &blk; (*blkptr)(10);`
- block中使用局部变量，截获的是局部变量的瞬间值。即使这些值后面被改也不影响block截获的值
- block表达式中改写外部变量的值，会产生编译错误。若想改，必须给外部变量声明`__block`说明符`__block int val = 0;`
- 向截获的外部可变数组中插值，不会有问题，但向该数组赋值会有问题
- block表达式中使用c语言数组，例如`cosnt char text[] = "hello";`使用text[2]有问题，使用指针可以,`cosnt char *text = "hello";`再使用text[2]可以
- objc_msgSend函数，从对象持有类的结构体中检索函数指针并调用
- block实质是oc对象，block调用实质是用函数指针调用函数
- id、Class都是结构体指针
- oc类与对象的实质：类的结构体指针,class_t包含成员变量、方法名、方法实现、属性、父类指针
- block的局部变量截获，只针对表达式中使用了的变量，没使用的不截获
- 截获的实质是，执行block语法时，表达式使用的变量被保存到block的结构体实例
- c语言中的静态变量、静态全局变量、全局变量，允许block改写值 `static`
- `static`表示作为静态变量存储在数据区中 `auto`表示作为自动变量存储在栈中
- 加`__block`，会使变量转为结构体实例。多个block中可使用同一个__block变量，同一个block中使用多次__block变量
- block可存储在栈上、堆上、数据区上(.data区)。在记述全局变量的地方使用block语法，就是存储在数据区上。另一个，block不截获自动变量时，也可以存储在数据区。
- 除去上两种之外，存储在栈上。栈上的block，变量作用域结束，block就该被废弃。但可以通过复制到堆上，即使变量作用域结束，堆上的block和__block变量还可以继续存在。
- `__block`变量的结构体成员变量`__forwarding`可以实现`__block`变量配置在堆上和栈上都能正确访问`__block`变量
- ARC下，将block作为函数返回值返回时编译器会自动生成复制到堆上的代码
- 将block作为方法参数使用，编译器不能自动判断生成代码。
- cocoa框架的usingBlock和gcd中的api不需要手动复制
- 需要手动复制的时候，给block 调 copy 方法就行。不能将所有栈上的block都复制到堆上，因为在栈上也能使用的block复制到堆上，会浪费cpu资源
- 存在数据区的block调copy什么也不做，堆上的block调copy方法，引用计数加1
- 多次调copy也没有问题，不确定是否要copy时，copy一下不会有任何问题
- `__block`变量跟随block被从栈复制到堆，并被block持有，被多个block持有，增加`__block`变量的引用计数
- 栈上和堆上同时有`__block`变量，在block内部和外部访问`__block`变量，访问的是同一个，即堆上的`__block`变量，栈上的`__forwarding`指向堆上的结构体实例地址
- __main_block_copy_0 和 __main_block_dispose_0 函数，是block中的相当于retain和release作用的函数，block从栈复制到堆时调copy，堆上的block被废弃时调dispose
- block从栈复制到堆的几种情况：调用block的copy方法，block作为函数返回值，赋值给类的__strong的id类型或block类型变量，方法名带usingBlock的方法里传递block时。这4种实际上都是调用_Block_copy函数，从栈复制到堆
- `__block`和持有__strong修饰的对象不同之处，参数不同，BLOCK_FIELD_IS_OBJECT和BLOCK_FIELD_IS_BYREF
- 调用copy才会持有截获的对象，不然对象无法超出变量作用域而存在
- `__block`和__strong一起使用，作用的`__block`一样。`__block`和__weak一起使用，作用域结束时，__weak修饰的对象会被置为nil。`__unsafe_unretained`和`__block`一起使用，注意空指针，`__block`不可与`__autoreleasing`一起使用，会报编译错误
- block的循环引用，类成员变量是block类型，block中持有类的对象，可以通过`__weak`和`__unsafe_unretained`避免循环引用，注意引用了类的其他成员变量，也会造成循环引用
- 使用__block也可避免循环引用，但要在block语句中执行置nil操作，且要执行block

### GCD

- serial queue 和 concurrent queue，串行和并行队列
- 多个serial queue可并行执行
- 线程过多会降低系统的响应性能
- 仅在多个线程更新相同资源导致数据竞争时使用serial queue
- serial queue比concurrent queue能生成更多的线程
- 生成concurrent queue`dispatch_queue_create("", DISPATCH_QUEUE_CONCURRENT)`
- 生成serial queue `dispatch_queue_create("", NULL)`
- dispatch_queue_t 类型
- 队列必须自己释放，dispatch_release(),dispatch_retain()
- dispatch_async()追加的block，即使执行dispatch_release()，也不会立即废弃队列，因为block也持有队列，执行完block才会废弃
- 含有create的api,有必要释放
- main dispatch queue 是 serial queue，在主线程的runLoop中执行
- global dispatch queue 4个优先级，高high、默认default、低low、后台background
- `dispatch_get_main_queue()` `dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH,0)`
`dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT,0)`
`dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_LOW,0)`
`dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND,0)`
- 系统级的main dispatch queue和global dispatch queue 不需要retain\release
- create生成的queue使用global queue默认优先级相同优先级的线程，要改变优先级用`dispatch_set_target_queue(queue1,queue2)` queue1是欲设置优先级的队列，queue2是同优先级的目标队列，queue1需要为create生成的
- dispatch_set_target_queue还会改执行层次，多个serial queue指定同一个serial queue，并行执行的多个serial queue会在目标queue上同时只执行一个
- `dispatch_time_t time = dispatch_time(DISPATCH_TIME_NOW,3ull * NSEC_PER_SEC);
dispatch_after(time,dispatch_get_main_queue(),^{});
  `
- runloop每隔1/60秒执行，这个时间延迟只是大致，不精确
- ull是c语言数值字面量，表示unsigned long long
- NSEC_PER_SEC单位是秒，NSEC_PER_MSEC单位是毫秒
- dispatch_time_t 类型的值也可由NSDate对象转化来，使用struct timespec类型以及dispatch_walltime()函数，见155页
- `dispatch_group_t group = dispatch_group_create();
dispatch_group_async(group, queue, ^{});
  dispatch_group_nofity(group,dispatch_get_main_queue(),^{});
    dispatch_release(group);`
- 用group可以处理多个并行执行，监测结束做结束处理
- group带create结束需要释放，追加的block也持有group,所以group使用结束立即释放没问题
- dispatch_group_wait函数，`dispatch_group_wait(group,DISPATCH_TIME_FOREVER);`第二参数为等待时间属于dispatch_time_t类型，这里为永久等待。
- long result = dispatch_group_wait(group,time);返回值不为0，意味着未结束,二参为DISPATCH_TIME_FOREVER时，结果恒为0。wait函数会阻塞线程，参数为NOW时，不等待
- `dispatch_barrier_async(queue, blk);`
- 使用concurrent queue和barrier函数，可实现数据库访问和文件访问，barrier相当于插入到并行队列中的单个处理，会等待插入之前的处理全部结束，然后单独执行barrier处理，之后再继续插入之后的并行处理
- dispatch_sync函数，也会阻塞线程
- 死锁问题：`dispatch_sync(mainQueue, ^{});
  dispatch_async(mainQueue,^{
    dispatch_sync(mainQueue, ^{});
    });`

`dispatch_async(serialQueue,^{
  dispatch_sync(serialQueue, ^{});});
`
- `dispatch_apply(10,queue,^(size_t index){})`执行多次，并等待全部执行结束。在dispatch_async中使用使用apply,162页
- dispatch_suspend(queue); dispatch_resume(queue); 挂起对已执行的处理没影响，未执行的处理会停止，恢复会继续执行
- `dispatch_semaphore_t semaphore = dispatch_semaphore_create(1);`计数为0时等待，大于0减1不等待。
- `dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER)`等待计数大于等于1，减1并从函数返回。返回不为0等待，为0继续。
- dispatch_semaphore_signal(semaphore);给计数加1，处理前等待，处理后+1
- `static dispatch_once_t pred; dispatch_once(&pred,^{});` 生成单例时使用
- 178页代码
