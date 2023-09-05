---
title: iOS【52个方法】第四篇
date: 2023-07-05 21:54:57
tags:
- 图书
- 专业技术
- iOS
---

### 41 派发队列与同步锁

- 内置的同步锁，根据给定的对象创建锁，等待块中的代码执行完毕。滥用会降低代码效率，若在self上频繁加锁，程序必须按顺序执行，可能要等一段其他代码执行完毕，才能执行当前代码。不能保证完全的线程安全，同一线程多次调用获取值，可能有其他线程改写值，而导致获取的结果不同。

``` objective-c
@synchronized(self) {

}
```

- 第二种是NSLock，或递归锁NSRecursiveLock，线程能多次持有该锁，而不会死锁。同步块会导致死锁，效率不高。用锁对象一旦遇到死锁会很麻烦。

``` objective-c
_lock = [[NSLock alloc] init];
[_lock lock];
[_lock unlock];
```
- 使用串行同步队列，将读取与写入操作都放在一个块里，保证数据同步。设置方法不一定非得是同步的，改为异步派发后，性能会变慢，因为异步派发需要拷贝块。如果异步执行任务复杂，需要更多时间，这种方法值得考虑。

``` objective-c
_syncQueue = dispatch_queue_create("syncQueue", NULL);

- (NSString *)someString {
  __block NSString *localSomeString;
  dispatch_sync(_syncQueue, ^{
    localSomeString = _someString;
  });
  return localSomeString;
}
- (void)setSomeString:(NSString *)someString {
  dispatch_async(_syncQueue, ^{
    _someString = someString;
  }); //异步派发 可以提升速度
  // dispatch_sync(_syncQueue, ^{
  //   _someString = someString;
  // }); 同步派发
}
```

- 多个获取方法可并发执行，设置方法与获取方法之间不能并发执行。改用并发队列，还能继续提升速度。并发队列+栅栏，可实现同步。栅栏块必须单独执行，不能与其他块并行，只对并发队列有意义。并发队列发现栅栏块，先将当前所有块执行完毕，然后单独执行栅栏块。可使用栅栏块实现设置方法。(时序图169页)

``` objective-c
_syncQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

- (NSString *)someString {
  __block NSString *localSomeString;
  dispatch_sync(_syncQueue, ^{
    localSomeString = _someString;
  });
  return localSomeString;
}
- (void)setSomeString:(NSString *)someString {
  dispatch_barrier_async(_syncQueue, ^{
    _someString = someString;
  });
  //或用dispatch_barrier_sync 效率可能更高，少一个拷贝操作
}
```

### 42 少用performSelector

- 若选择子是运行期决定，这个方法较有用。如下在ARC下编译代码，编译器会给出警告信息。此种情况，编译器不知道方法名，不能运用内存管理机制判断返回值是否应释放。因此ARC不添加释放操作，可能导致内存泄露，并且难以侦测。

``` objective-c
- (id)performSelector:(SEL)selector;
SEL selector;
if (/* 某条件 */) {
  selector = @selector(foo);
} else {
  selector = @selector(bar);
}
[object performSelector:selector];
```
- performSelector返回值是id类型，只能用于选择子返回类型为void或对象类型。如果想返回整数或浮点数等类型的值，需要执行复杂的转换。若返回值类型为C结构体则不可用performSelector方法。
- 传递参数的版本，所传参数必须是id类型，最多只能接受2个参数。延后执行的无法带2个参数，如果要用这些方法，需要将许多参数打包到字典中，这会增加开销且容易出bug.

``` objective-c
- (id)performSelector:(SEL)selector withObject:(id)object;
- (id)performSelector:(SEL)selector withObject:(id)objectA withObject:(id)objectB;
- (id)performSelector:(SEL)selector withObject:(id)object afterDelay:(NSTimeInterval)delay;
- (id)performSelector:(SEL)selector onThread:(NSThread*)thread withObject:(id)object waitUntilDone:(BOOL)wait;
- (id)performSelectorOnMainThread:(SEL)selector withObject:(id)object waitUntilDone:(BOOL)wait;
```
- 用块来代替，延后执行用dispatch_after实现，另一个线程执行任务通过 dispatch_sync 和 dispatch_async 来实现。

``` objective-c
dispatch_time_t time = dispatch_time(DISPATCH_TIME_NOW,(int64_t)(5.0 * NSEC_PER_SEC));
dispatch_after(time, dispatch_get_main_queue(), ^(void){
  [self doSomething];
});
dispatch_async(dispatch_get_main_queue(), ^{
  [self doSomething];
});
```

### 43 GCD与操作队列的使用时机

- 操作队列NSOperationQueue执行后台任务比较好，用操作NSOperation子类的方式放在队列中，也能并发执行。操作队列在底层是用GCD实现的。
- GCD是纯C，操作队列是oc对象，操作队列更为重量级，有时候操作队列的开销比较小，但带来的好处可以超过其缺点。
- 用NSOperationQueue的addOperationWithBlock: 方法搭配 NSBlockOperation 类来使用操作队列。下列为使用操作队列的好处。
- 操作队列可取消某个操作，NSOperation 的cancel方法，不过已经启动的任务无法取消。GCD无法取消。
- 操作队列可指定操作间的依赖关系，使某个操作需要依赖于其他操作完成。
- 通过KVO监控NSOperation对象的属性，例如监控isCancelled属性判断是否取消,isFinished属性判断任务是否完成。
- 指定操作的优先级，优先级高的操作先执行。GCD也有优先级，但是是针对整个队列来说的，而不是具体每个块。NSOperation对象也有线程优先级，GCD也有此功能，但是操作队列使用起来更简单，只需设置一个属性。
- 可以重用NSOperation对象，NSBlockOperation即是其子类，也可以自己创建子类。
- NSNotificationCenter 使用的是操作队列。

``` objective-c
- (id)addObserverForName:(NSString *)name object:(id)object queue:(NSOperationQueue *)queue usingBlock:(void(^)(NSNotification*))block;
```

### 44 Dispatch Group

- 将并发任务合为一组

``` objective-c
dispatch_group_t dispatch_group_create();
void dispatch_group_async(dispatch_group_t group, dispatch_queue_t queue, dispatch_block_t block);
//第一种将任务分组的方法
void dispatch_group_enter(dispatch_group_t group);
void dispatch_group_leave(dispatch_group_t group);
//第二种，这一对函数要成对使用
long dispatch_group_wait(dispatch_group_t group, dispatch_time_t timeout);
//timeout表示在等待group执行完毕时应阻塞多久，如其时间小于timeout返回0，否则返回非0值，参数可以是常量DISPATCH_TIME_FOREVER，表示会一直等待group执行完而不超时
void dispatch_group_notify(dispatch_group_t group, dispatch_queue_t queue, dispatch_block_t block);
// 另一种等group执行完毕的方法，之后块可以在指定线程上执行
```
- 数组中每个对象都执行某项任务

``` objective-c
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_group_t group = dispatch_group_create();
for (id obj in collection) {
  dispatch_group_async(group, queue, ^{[obj performTask];});
}
dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
//等待所有任务执行完毕，会阻塞当前线程
dispatch_group_notify(group, dispatch_get_main_queue(), ^{
  //任务完成后继续的操作
});
//不会阻塞当前线程
```
- 可以把任务放到优先级高的线程执行，再把所有任务归入一个group

``` objective-c
dispatch_queue_t lowQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_LOW, 0);
dispatch_queue_t highQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0);
dispatch_group_t group = dispatch_group_create();
for (id obj in lowCollection) {
  dispatch_group_async(group, lowQueue, ^{[obj performTask];});
}
for (id obj in highCollection) {
  dispatch_group_async(group, highQueue, ^{[obj performTask];});
}
dispatch_group_notify(group, dispatch_get_main_queue(), ^{
  //任务完成后继续的操作
});
```
- 可以用串行队列，这样group作用不大，因为是逐个执行，在提交完全部任务后再提交块即可。

``` objective-c
dispatch_queue_t queue = dispatch_queue_create("queue", NULL);
for (id obj in collection) {
  dispatch_async(queue, ^{[obj performTask];});
}
dispatch_async(queue, ^{
  //继续执行任务
});
```
- 并发队列中可能会创建和复用多个线程，并发线程数量根据系统资源状况判断，cpu有多个核心，gcd会给该队列配备多个线程。
- 还有个多次执行的块，每次传给块的值递增，从0开始，直至iterations-1。dispatch_apply所用的队列可以是并发队列。

``` objective-c
void dispatch_apply(size_t iterations, dispatch_queue_t queue, void(^block)(size_t));
dispatch_queue_t queue = dispatch_queue_create("queue", NULL);
dispatch_apply(10, queue, ^(size_t i){
  //执行任务
});

dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_apply(array.count, queue, ^(size_t i){
  id obj = array[i];
  [obj performTask];
});
```
- dispatch_apply会持续阻塞，直到所有任务都执行完毕。如果把块派给了当前队列或高于当前队列的某个串行队列，将导致死锁。

### 45 dispatch_once

- 单例，即某类只有一个实例，不会每次使用都创建新的实例。

``` objective-c
+ (id)sharedInstance {
  static EOCClass *sharedInstance = nil;
  @synchronized(self) {
    if (!sharedInstance) {
      sharedInstance = [[self alloc] init];
    }
  }
  return sharedInstance;
}
//同步块这种写法很常见
```
- GCD的实现方式，dispatch_once，该函数保证块只执行一次，首次调用时执行，线程安全。每次调用token参数必须相同，常声明在static或global作用域里。
- 该实现方式可以简化代码，保证线程安全。把token定义在static作用域，保证编译器每次都复用该变量，而不创建新变量。这种方法更快。

``` objective-c
void dispatch_once(dispatch_once_t *token, dispatch_block_t block);

+ (id)sharedInstance {
  static EOCClass *sharedInstance = nil;
  static dispatch_once_t onceToken;
  dispatch_once(&onceToken, ^{
    sharedInstance = [[self alloc] init];
  });
  return sharedInstance;
}
```

### 46 不使用dispatch_get_current_queue

- iOS已废弃该方法，macOS未废弃。主线程相当于GCD的主队列。下面代码获取方法可能会死锁，如果执行块的队列恰好是当前执行获取方法的队列，dispatch_sync就会一直不返回，直到块执行完毕才行。而执行块的队列是当前队列，其dispatch_sync一直在阻塞，在等待块执行完。块会没机会执行。(意思就是当前队列调dispatch_sync，实际就已经阻塞了，阻塞之后还想在当前队列执行块，这个块是不会执行的)

``` objective-c
- (NSString *)someString {
  __block NSString *localSomeString;
  dispatch_sync(_syncQueue, ^{
    localSomeString = _someString;
  });
  return localSomeString;
}
```
- 下例这种情况有可能死锁

``` objective-c
- (NSString *)someString {
  __block NSString *localSomeString;
  dispatch_block_t accessorBlock = ^{
    localSomeString = _someString;
  };
  if (dispatch_get_current_queue() == _syncQueue) {
      accessorBlock();
  } else {
    dispatch_sync(_syncQueue, accessorBlock);
  }
  return localSomeString;
}
```
- 下列代码肯定死锁，外层的queueA想执行完毕，要等最内层的dispatch_sync执行完毕，而内层是不可能执行完毕的，于是死锁。

``` objective-c
dispatch_queue_t queueA = dispatch_queue_create("queueA", NULL);
dispatch_queue_t queueB = dispatch_queue_create("queueB", NULL);
dispatch_sync(queueA, ^{
  dispatch_sync(queueB, ^{
    // dispatch_sync(queueA, ^{
    //   //死锁
    // });
    dispatch_block_t block = ^{};
    if (dispatch_get_current_queue() == queueA) {
        block();
    } else {
      dispatch_sync(queueA, block);
    }
    //这样依旧会死锁，dispatch_get_current_queue 返回是queueB
  });
});
```
- 应该确保同步操作所用的队列绝不访问属性，其队列只应该用来同步属性。每项属性可以有专用的同步队列，可以创建多个队列。
- 队列之间有层级体系，某个队列中的块可以在其上级队列中执行，层级里地位最高的队列是全局并发队列。(182页层级图)
- 检查当前队列是否为执行同步派发所用的队列，不一定奏效。如果将操作放在上级队列中执行，而上级队列和当前队列都是串行队列，依然会死锁。
- 使用GCD提供的队列特定数据，可以将数据关联到队列里，根据队列找关联数据时，会沿着体系向上找，直到找到数据或到达根队列为止。这个方法可解决获取当前队列却不是认为会取到的队列的问题。

``` objective-c
dispatch_queue_t queueA = dispatch_queue_create("queueA", NULL);
dispatch_queue_t queueB = dispatch_queue_create("queueB", NULL);
dispatch_set_target_queue(queueB, queueA);
//将B的目标设为A,A的目标是默认优先级的全局并发队列。A是B的上级
static int kQueueSpecific;
CFStringRef queueSpecificValue = CFSTR("queueA");
dispatch_queue_set_specific(queueA, &kQueueSpecific, (void*)queueSpecificValue, (dispatch_function_t)CFRelease);
//第二个参数类型 const void *key 第三个参数 void *context 作为值 第四个类型是dispatch_function_t 参数是destructor
//函数是按指针来比较键的 NSDictionary 比较的是键的对象等同性
//ARC不会管理CoreFoundation对象的内存 此例需要管理对象内存
//最后的参数是析构函数 typedef void (*dispatch_function_t)(void *)
dispatch_sync(queueB, ^{
  dispatch_block_t block = ^{};
  CFStringRef retrievedValue = dispatch_get_specific(&kQueueSpecific);
  if (retrievedValue) {
    block();
  } else {
    dispatch_async(queueA, block);
  }
});
```
### 47 系统框架

- iOS平台的系统框架使用动态库。Foundation框架使用NS前缀(NeXTSTEP，macOS的前身)
- CoreFoundation，Foundation里的许多功能，都能在其中找到对应的C语言API，用桥接可相互转换。
- CFNetwork C语言级别的网络通信能力，NSURLConnection 是 Foundation 对其内容封装的OC接口
- CoreAudio C语言API操作音频硬件
- AVFoundation 录制音视频 oc
- CoreText 文字排版渲染 c
- 使用c语言api可提升速度，但要注意内存管理问题
- CoreAnimation oc 渲染图形，播放动画，其本身不是框架，而是QuartzCore框架的一部分
- CoreGraphics c 2d渲染数据结构与函数，有CGPoint CGSize CGRect
- Social 社交网络

### 48 块枚举

- for循环方式遍历比其他方式简单
- NSEnumerator 抽象基类，定义了2个方法。nextObject返回枚举里的下个对象，全部对象返回之后，就会返回nil。遍历字典和set也可用一样的写法

``` objective-c
- (NSArray *)allObjects
- (id)nextObject

NSEnumerator *enumerator = [anArray objectEnumerator]; //或[aSet objectEnumerator];
//[anArray reverseObjectEnumerator] 反向枚举器
id obj;
while ((obj = [enumerator nextObject]) != nil) {
  /* code */
}
NSEnumerator *enumerator = [aDictionary keyEnumerator];
id key;
while ((key = [enumerator nextObject]) != nil) {
  id value = aDictionary[key];
  /* code */
}
```
- for in 遍历，也可以反向遍历数组

``` objective-c
for (id key in aDictionary) {
  id value = aDictionary[key];
}
for (id obj in aSet) {
}
for (id obj in [anArray reverseObjectEnumerator]) {
}
```
- 块的遍历，数组、字典、NSSet的遍历方法。可以对元素或键值改参数类型，因为id可以被其他类型覆写，如果确定知道元素是什么类型，就应该在方法参数里指明。

``` objective-c
- (void)enumerateObjectsUsingBlock:(void(^)(id obj, NSUInteger idx, BOOL *stop))block
- (void)enumerateObjectsWithOptions:(NSEnumerationOptions)options UsingBlock:(void(^)(id obj, NSUInteger idx, BOOL *stop))block
//可以反向遍历NSEnumerationReverse，只有遍历块数组或有序set这样设置才有意义
//NSEnumerationOptions 取值可用按位或。也可以并发执行块NSEnumerationConcurrent
- (void)enumerateKeysAndObjectsUsingBlock:(void(^)(id key, id obj, BOOL *stop))block

[anArray enumerateObjectsUsingBlock:^(id obj, NSUInteger idx, BOOL *stop){
    if (shouldStop) {
      *stop = YES;
    }
}];
```
### 49 桥接

- 转换oc对象和cf数据结构，\__bridge表明ARC仍保留对象的所有权，而__bridge_retained表明ARC将交出对象的所有权
- __bridge_transfer 将cf数据结构转为oc对象，可令ARC获得所有权

``` objective-c
CFArrayRef aCFArray = (__bridge CFArrayRef)anNSArray;
CFArrayGetCount(aCFArray);
//如果用__bridge_retained来做，则需要释放aCFArray
```
- NSDictionary 键内存管理语义是拷贝，值是保留。桥接可以改变其语义。

``` objective-c
CFMutableDictionaryRef CFDictionaryCreateMutable(CFAllocatorRef allocator, CFIndex capacity, const CFDictionaryKeyCallBacks *keyCallBacks, const CFDictionaryValueCallBacks *valueCallBacks)
// 首参 内存分配器 负责分配和回收内存 通常传入NULL 表示用默认的分配器
// 2参 定义字典初始大小 向分配器提示一开始应分配多少内存
// 3、4 参数 定义回调函数 表示字典中的键和值在遇到各种事件时应执行何种操作 为指向结构体的指针

struct CFDictionaryKeyCallBacks {
  CFIndex version;
  CFDictionaryRetainCallBack retain;
  CFDictionaryReleaseCallBack release;
  CFDictionaryCopyDescriptionCallBack copyDescription;
  CFDictionaryEqualCallBack equal;
  CFDictionaryHashCallBack hash;
};
struct CFDictionaryValueCallBacks {
  CFIndex version;
  CFDictionaryRetainCallBack retain;
  CFDictionaryReleaseCallBack release;
  CFDictionaryCopyDescriptionCallBack copyDescription;
  CFDictionaryEqualCallBack equal;
};
//version 参数应设为0 表示版本号 检测新版与旧版数据结构之间是否兼容
// 其余参数是函数指针，定义了各种事件发生时应执行何种函数。字典中加入新的键值，会调用retain函数
typedef const void* (*CFDictionaryRetainCallBack) (
  CFAllocatorRef allocator,
  const void *value
);
//retain 是个函数指针 参数value表示即将加入字典的键或值，返回的void *表示加到字典中的值
```

- 将其与桥接搭配使用，可以创建出与oc创建出的字典不同的特殊字典。(代码196页)
- 如果加入NSMutableDictionary 中的键不支持拷贝，会报错 copyWithZone 方法未实现。用 CoreFoundation 创建字典，可以对键执行保留而非拷贝，因为可修改其内存管理语义

### 50 缓存NSCache

- NSCache 在系统资源将耗尽时，可以自动删减缓存，先删减最久未使用的对象
- NSCache 不会拷贝键，而是保留它。不支持拷贝键是因为很多时候键所用的对象不支持拷贝
- NSCache是线程安全的，多个线程可同时访问NSCache，而不用加锁
- 可以控制删减缓存的时机，根据缓存中对象总数或所有对象的总开销。在将对象加入缓存时可以指定其开销值，当对象总数或开销值超上限，就会自动删减了，可用系统资源紧张时也会这么做。
- 如果开销值计算复杂耗时，例如需要访问磁盘或数据库才能计算出来，就不太好了。因为使用缓存的目的是为了提升速度的。如果要加入的是NSData可以指定开销值，其开销值就是数据大小，这样仅是读取属性而已。

``` objective-c
@implementation EOCClass {
  NSCache *_cache;
}
- (id)init {
  if ((self = [super init])) {
    _cache = [NSCache new];
    _cache.countLimit = 100;
    _cache.totalCostLimit = 5 * 1024 * 1024;
    //5M数据最大
  }
  return self;
}
- (void)downloadDataForURL:(NSURL*)url {
  NSData *cachedData = [_cache objectForKey:url];
  if (cachedData) {
    // 使用数据
  } else {
    //下载数据并缓存
    [_cache setObject:data forKey:url cost:data.length];
  }
}
@end
```
- NSPurgeableData 是 NSMutableData子类，实现了NSDiscardableContent协议，如果某对象所占内存能随时丢弃，就可实现该协议的方法。协议里有isContentDiscarded方法，可查询相关内存是否已释放。
- NSPurgeableData 对象有beginContentAccess和endContentAccess方法，告诉系统不可丢弃和可丢弃的状态变化。与cache搭配使用，当其被系统丢弃时，也会自动从缓存中移除。

``` objective-c
- (void)downloadDataForURL:(NSURL*)url {
  NSPurgeableData *cachedData = [_cache objectForKey:url];
  if (cachedData) {
    [cachedData beginContentAccess];
    // 使用数据
    [cachedData endContentAccess];
  } else {
    //下载数据并缓存
    NSPurgeableData *purgeableData = [NSPurgeableData dataWithData:data];
    //创建之后自动引用为1，不需要调beginContentAccess
    [_cache setObject:purgeableData forKey:url cost:purgeableData.length];
    // 使用数据
    [purgeableData endContentAccess];
  }
}
```
### 51 精简initialize 和 load 的代码

- load方法，每个类及其分类必定会调用，仅调用一次。当类或分类载入系统时，就会执行，通常是程序启动的时候。如果类与分类都定义了load方法，则先调用类里再调用分类里的。由于无法判断出类的载入顺序，在load方法中使用其他类是不安全的。如果使用了其他类，而该类没有载入并完成初始化，其可能无法正常使用。
- 如果load没有实现，就不会调用。务必精简，因为整个程序在执行load时都会阻塞。不要在里面使用锁。其真正用途在于调试程序。

``` objective-c
+ (void)load
+ (void)initialize {
  if (self == [EOCBaseClass class]) {
    //当期望的类载入时，才执行相关操作
  }
}
```

- initialize 在类首次使用前调用，是惰性调用，只调用一次。不能通过代码调用。如果该类没有使用，就不会调用。而load是每个类都会调，不管使用与否。
- 在initialize可以调用任意类的任意方法，是线程安全的。只有执行initialize的线程可以操作类或实例，其他线程都要先阻塞。其有超类实现，如果某类没实现，会调用其超类的方法，并且其超类方法先执行。
- 也不要在其中执行复杂的代码，加锁之类，会阻塞初始化，如果是主线程就不好了。第二开发者无法控制类的初始化时机。第三，如果其中使用其他类，而其他类被迫使初始化，其中又用到了本类的数据，这些数据还未初始化好。
- initialize 只应该用来设置内部数据，也最好不要调用本类的方法。
- 全局状态可以在initialize 中初始化。整数可以在编译器确定，可变数组不行，会报错。它是oc对象，必须先激活运行期系统。NSString可以在编译期创建。

``` objective-c
static const int kInterval = 10;
static NSMutableArray *kSomeObjects;
@implementation
+ (void)initialize {
  if (self == [EOCClass class]) {
      kSomeObjects = [NSMutableArray new];
  }
}
@end
```

### 52 NSTimer

- 计时器和 run loop关联，运行循环会触发任务。可以创建并安排在当前运行循环中，也可以创建后，自己调度。计时器会保留目标对象，等到自身失效时再释放。invalidate方法可令计时器失效。一次性的计时器会自动失效，重复执行模式需要自己调用invalidate才行。

``` objective-c
+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)seconds target:(id)target selector:(SEL)selector userInfo:(id)userInfo repeats:(BOOL)repeats
//
```

- 重复执行模式容易导致循环引用(代码206页)，这种内存泄露很严重，因为计时器还在反复执行任务。

``` objective-c
//这个加的分类提供可以打破循序的方式
// block通过copy拷贝到堆
//target 是NSTimer类对象，是个单例。保留类对象，但类对象无需回收，不用担心
//计时器有效时会一直保留userInfo参数，应该是内部属性做的保留，其是个id类型
@interface NSTimer (EOCBlocksSupport)
+ (NSTimer *)eoc_scheduledTimerWithTimeInterval:(NSTimeInterval)seconds block:(void(^)())block repeats:(BOOL)repeats;
@end
@implementation NSTimer (EOCBlocksSupport)
+ (NSTimer *)eoc_scheduledTimerWithTimeInterval:(NSTimeInterval)seconds block:(void(^)())block repeats:(BOOL)repeats {
  return [self scheduledTimerWithTimeInterval:seconds target:self selector:@selector(eoc_blockInvoke:) userInfo:[block copy] repeats:repeats];
}
+ (void)eoc_blockInvoke:(NSTimer *)timer {
  void (^block)() = timer.userInfo;
  if (block) {
    block();
  }
}
@end
```
``` objective-c
- (void)startPolling {
  _pollTimer = [NSTimer eoc_scheduledTimerWithTimeInterval:5.0 block:^{
    [self p_doPoll];
  } repeats:YES];
  //这里仍有保留环 块捕获self self引用计时器 计时器通过userInfo保留了块
}

- (void)startPolling {
  __weak EOCClass *weakSelf = self;
  _pollTimer = [NSTimer eoc_scheduledTimerWithTimeInterval:5.0 block:^{
    EOCClass *strongSelf = weakSelf;
    [strongSelf p_doPoll];
  } repeats:YES];
  //这样self 不会被计时器保留。当块执行时，生成strong引用，保证实例在执行期间存活
  //如果编写dealloc时忘了调用计时器的invalidate方法，会导致计时器再次运行，但块里的weakSelf会是nil，也就是实际调了方法也没效果
}
```
