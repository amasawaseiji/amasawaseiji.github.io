---
title: iOS【52个方法】第三篇
date: 2023-07-02 18:04:46
tags:
- 图书
- 专业技术
- iOS
---

### 27 扩展（匿名分类）

- oc动态消息系统决定了不可能实现真正的私有方法或实例变量。
- 扩展定义在实现文件里，是唯一能声明实例变量的分类。其方法也应该定义在类的主实现文件里。该分类没有名字。其中可以定义方法和实例变量。
- 这样定义的好处是可以将变量隐藏起来(也可以定义在实现块里，和定义在扩展里等效)，在公共接口里用@private，还是会泄露信息，外界会知道有个叫xx的类。
- 定义在扩展里也不是真的私有，只是隐藏程度更好。

``` objective-c
@implementation EOCPerson {
  int _anotherInstanceVariable;
  /* 和定义在扩展里等效，但还是建议放扩展里，因为能将所有数据放一起 */
}
@end
```

- 游戏后端用C++来写，.mm扩展名表示按oc++来编译。扩展在编写oc++代码时也有用。如果在公共接口里引入c++类，这就要求oc类也会被编译为oc++才行，引入该oc类的类也要按oc++来编译，最终导致所有类都要编译成oc++。用向前声明，将实例变量做成指针也不行。因为class关键字是c++的关键字，仍然需要按oc++来编译。

``` objective-c
class SomeCppClass;
@interface EOCClass: NSObject {
@private
  SomeCppClass * _cppClass;
}
@end
```
- 用扩展，可以让头文件里没有c++代码，WebKit以及CoreAnimation都是内部用c++编写，但对外公布的接口是纯oc的。
- 扩展还有个用处，就是属性外部只读，内部改为可读写。以便内部设置值。相当于内部重新声明一遍，只不过将读写特质改为可读写。这样内部可调用设置方法或点语法。
- 只在实现文件中使用的私有方法也可定义在扩展中。方法名用p_前缀开头。现在可以不声明直接实现，但定义一下还是有好处的，可以使代码易读。
- 扩展可声明遵从了私有协议，如果写在公共接口里，用向前声明 @protocol 以不引入协议所在的头文件。这样编译器还是会给出警告，看不到协议的定义，无法得知其中所含有的方法。

### 28 用协议提供匿名对象

- 协议定义了方法，遵从协议就要实现他们，非可选就必须实现。返回id类型遵从协议的对象，这样类名就可隐藏。下面delegate就是匿名的。

``` objective-c
@property (nonatomic, weak) id <EOCDelegate> delegate;
```
- 字典对象也有例子，键是设置时拷贝，值是设置时保留。key参数是匿名对象，确定能给其发送拷贝消息就行了。

``` objective-c
- (void)setObject:(id)object forKey:(id<NSCopying>)key;
```
- 没有办法令所有类都继承同一个基类，可以创建匿名对象将类包裹一下，使匿名对象成为其子类，遵从协议。

``` objective-c
@protocol EOCDatabaseConnection
- (void)connect;
@end

@protocol EOCDatabaseConnection;
@interface EOCDatabaseManager : NSObject
+ (id)sharedInstance;
- (id<EOCDatabaseConnection>)connectionWithIdentifier:(NSString*)identifier;
@end
```
- 协议表示对象不重要，对象遵从的方法重要。

``` objective-c
id <NSFetchedResultsSectionInfo> sectionInfo = sections[sections];
NSUInteger num = sectionInfo.numberOfObjects;
```

### 29 引用计数

- retain 递增引用计数，release 递减引用计数，autorelease 稍后清理自动释放池时，再递减。当引用计数归零，就被回收了。引用对象回溯的最终对象，iOS上是UIApplication对象，是应用程序启动时创建的单例。
- alloc方法返回的对象由调用者持有，其引用计数至少为1. 数组的addObject: 方法会在待添加的对象上增加引用，即数组引用着元素。
- 释放之后再使用不一定会导致崩溃，其内存解除分配，只是放回可用内存池。如果使用时该内存还未被覆写，那么该对象依然有效，此时不会崩溃。过早释放导致的bug难以排查。
- release之后，将对象置为nil，可以避免出现指向无效对象，成为悬挂指针。
- 属性特质为strong，在设置时会先保留新值，释放旧值，然后更新实例变量，顺序不能变。
- 自动释放池通常在下一次事件循环释放。方法返回对象时常用，内部用autorelease。使对象超出方法边界而存在，除非自己写自动释放池，否则就是下一次事件循环时清空释放池。但是如果要持有此对象的话，就需要保留。

``` objective-c
- (NSString *)stringValue {
  NSString *str = [[NSString alloc] initWithFormat:@"xxx: %@",self];
  return [str autorelease];
}

_instanceVariable = [[self stringValue] retain];
[_instanceVariable release];
```

- 循环引用会导致内存泄露，解决方法通常是用弱引用，或外界命令循环中的某对象不再引用另一个对象。

### 30 ARC

- Clang编译器，自带一个静态分析器，可以分析出代码里引用计数出问题的地方。ARC的意思的引用和释放操作，它帮你做。ARC下不能调用retain、release、autorelease、dealloc方法。会产生编译错误。ARC调用的这些方法底层c语言版本，这样性能更好。例如调的是objc_retain.
- alloc、new、copy、mutableCopy 开头的方法名，其返回的对象归调用者持有。其他的则表示返回的对象不归调用者持有，会自动释放。(124页代码演示)
- ARC执行的优化，对同一个对象的多次保留与释放操作可能会被成对的移除。
- ARC在运行期组件的优化，ARC检测到autorelease和紧随其后的retain这一对多余的操作，会设置和检测全局数据结构中的一个标志位，来执行objc_autoreleaseReturnValue和objc_retainAutoreleasedReturnValue这一对函数，这比调用autorelease和retain更快。（126页代码演示）
- 将内存管理交给编译器和运行期组件来做，可使代码得到多项优化。
- 局部变量和实例变量是指向对象的强引用。ARC下的设置方法，先保留新值，释放旧值，最后设置实例变量。
- 局部变量与实例变量的修饰符，\__strong 保留，\__unsafe_unretained 不保留，不安全，可能会使用对象已经回收的变量，\__weak 不保留，安全，对象回收，变量自动清空。__autoreleasing 对象传递给方法做参数时用，方法返回时自动释放。
- 用__weak 打破block引入的循环引用，因为block会保留捕获的对象，如果对象又引用了block，就会造成循环引用。

``` objective-c
EOCNetworkFetcher * __weak weakFetcher = fetcher;
[fetcher startWithCompletion:^(BOOL success) {
  NSLog(@"Finished fetching from %@", weakFetcher.url);
}];
```

- ARC在dealloc中插入释放实例变量的方法，生成清理代码。ARC下dealloc方法不能调用超类的dealloc方法，需要释放CoreFoundation对象或是有malloc()分配在堆上的内存。

``` objective-c
- (void)dealloc {
  CFRelease(_coreFoundationObject);
  free(_heapAllocateMemoryBlob);
}
```
- ARC能够优化retain\release\autorelease操作，使其不经过oc的消息派发机制，直接调用隐藏在运行期程序库的C函数。

### 31 dealloc方法中只释放引用和解除监听

- CoreFoundation对象为什么要手动释放，因为他们是纯C的api生成的，不是oc对象。
- 应该移除监听，例如通知、kvo观察者，发送通知给回收后的对象，会导致崩溃。

``` objective-c
- (void)dealloc {
  [[NSNotificationCenter defaultCenter] removeObserver: self];
}
```
- 开销大或系统稀缺的资源，例如文件描述符、套接字、大块内存，应在用完资源就释放，不必等到dealloc时再释放。减少稀缺资源的保留时间。可使用自定义关闭方法，在这里清理资源。关闭需要在系统把对象回收之前调用。还有个原因是并不是每个对象都会执行dealloc，而是在应用程序终止时才将资源还给操作系统。系统有个程序终止时调用的方法。

``` objective-c
- (void)close;

- (void)applicationWillTerminate:(UIApplication *)application
```
- close方法和dealloc配合使用，防止忘记调用close。通常不应该在dealloc方法里调用其他方法，如果所调方法需要异步执行或调用自己的某些方法，等到任务执行完毕时，执行回调，当前对象可能已被销毁了，会使程序崩溃。
- 还有个原因是调用的方法可能和对象回收操作，不在一个线程里。对象处于正在回收的状态，系统已改动了对象的内部数据结构。
- 也不应该调用属性的存取方法，因为其内部可能在做在回收阶段无法安全执行的代码。若是属性处于kvo观测之下，观察者可能会在属性值改变时保留或使用这个即将回收的对象。这会导致错误。

``` objective-c
- (void)close {
  _close = YES;
}
- dealloc {
  if (_close) {
    NSLog(@"ERROR: close was not called before dealloc");
    [self close];
  }
}
```

### 32 异常安全代码的内存管理问题

- c++与oc都支持异常，且相互兼容。什么时候捕获异常，oc++编码、第三方库抛出的异常、系统库kvo注销尚未注册的观察者会抛出异常。
- try块中保留的对象，在释放之前抛出异常，会导致内存泄露。C++的析构函数由oc的异常处理例程来运行，发生异常时，必须析构，不然会泄露。文件句柄没正确清理，更易泄露。
- 异常会令执行终止，并跳至catch块。可以使用@finally块，其中的代码一定会执行，且只运行一次。

``` objective-c
@try {
  //创建某对象
  [object doSomethingThatMayThrow];
}
@catch (...) {
  NSLog(@"Whoops, there was an error. Oh well...");
}
@finally {
  [object release];
}
```
- 假如try块逻辑复杂，容易忘记某个对象导致泄露。
- ARC下不会自动处理，将释放操作移动到@finally块中，如果这样做会严重影响性能。
- -fobjc-arc-exceptions 这个编译器标志是默认关闭的，因为oc中只有当程序必须终止才抛出异常，这时会否发生内存泄露已经不重要了。还去添加异常安全代码没有意义。
- oc++模式，会把该标志打开。因为c++处理异常与ARC实现的附加代码类似，所以开启性能损失不大，且c++频繁使用异常。
- ARC要捕获异常，要打开编译器标志。如果有大量异常，应该用NSError取代异常。

### 33 弱引用避免循环引用

- 避免循环引用的最佳方法是弱引用，unsafe_unretained表示非拥有关系，但不安全，若其所指的对象被回收了，再用其调方法可能会使程序崩溃。
- unsafe_unretained 与 assign 等价，只是assign通常用于(int\float\结构体等)，unsafe_unretained则多用于对象类型。
- weak与unsafe_unretained作用完全相同，只是其所指对象被回收，属性值会自动置nil.
- weak比unsafe_unretained更安全，不至于使程序崩溃。

### 34 自动释放池块降低内存峰值

- 自动释放池存储在稍后某时释放的对象。清空自动释放池，会向其中的对象发送release方法。
- iOS系统主线程或GCD机制中的线程，都有默认的自动释放池。每次执行事件循环都会将其清空。main函数里创建自动释放池，包裹程序的主入口点。这个作为最外围捕捉全部自动释放对象所用的池。
- 自动释放池嵌套，可以控制程序的内存峰值。for循环创建对象，可能会放自动释放池里，而自动释放池要等下一次事件循环才清空，这样会造成内存量持续上涨后又突然下降。
- 内存峰值是指程序在某个时间段内的最大内存用量，新增自动释放池可以减少峰值。每次执行循环都会创建并清空自动释放池。
- 自动释放池有一定的开销，尽量不要建立额外的自动释放池。

``` objective-c
for (NSDictionary *record in databaseRecords) {
  @autoreleasepool {
    //创建对象加到数组里
  }
}
```

### 35 僵尸对象调试内存管理问题

- 向已回收的对象发消息不安全，取决于对象所占内存有无被其他内容覆写，若内存恰好存活的对象占据，运行期系统会把消息发给这个对象，这个对象也许能应答消息。
- 僵尸对象调试功能开启，把已回收的对象转为僵尸对象，其内存无法使用不会被覆写，僵尸对象收到消息后会抛出异常，说明发送的消息以及回收之前的对象。NSZombieEnabled设为YES。给僵尸对方发生消息，程序会终止。
- xcode里scheme的 Enable Zombie Objects 选项。开启的话，系统在回收对象时会将对象转化为僵尸对象，而不彻底回收。（代码示例142页）
- 运行期生成僵尸类，首次碰到某类的对象要变成僵尸对象时，会创建僵尸类，用的是运行期程序库的函数，可以操作类列表。僵尸类从名为_NSZombie的模板里创建出来，充当标记。
- 运行期如果发现开启检测，用方法调配技术将dealloc方法替换成将待回收的对象转化为僵尸对象的方法。执行完之后，其类名已经变成僵尸类了。（_NSZombie_加原类名）创建新类名原因在于这样可以知道对象原来所属的类(用objc_duplicateClass()函数创建新类)。
- _NSZombie_类没有超类，也没实现方法，是个根类，只有一个实例变量_isa(所有oc类都有)，给该类发消息会经过完整的消息转发机制。
- 转发中，\_\_\_forwarding___函数会检查接受消息的对象所属的类名，若发现是僵尸对象，就需要特殊处理，将打印消息指明僵尸对象收到的消息和原来所属的类，然后程序终止。(代码145页)

### 36 不用retainCount

- ARC中已废弃该方法，这个方法不该调用。
- 原因在于其返回的数量只是某个时间点的值，未考虑稍后的自动释放池清空操作。其未必能反应实际的计数。retainCount 数量不可能返回0，只有系统不优化时，该数值才可能返回0.
- 字符串字面量和数字字面量创建的字符串和Number，都是单例对象，其引用计数都很大。编译器会把NSString对象表示的数据放到应用程序二进制文件里，而没有创建NSString对象。
- NSNumber则用标签指针标注特定类型的数值。其不使用NSNumber对象，而是把数值有关的全部消息都放到指针值里面。运行期消息派发时检测到标签指针，对它执行操作。浮点数类型不用这种方法，其引用计数为1.
- 单例对象引用值不会变，单例之间的值也不相同。
- 只是调试，也不该用retainCount方法，其值不准确。

### 37 块

- 多线程编程和核心是块与GCD，块可在C、C++、OC、OC++中使用，可将其想对象一样传递。GCD可以适时创建、复用、摧毁后台线程。
- Clang是开发iOS程序所用编译器，块是C语言层面的特性。块有其类型，实际是个值，可将其赋值给变量，语法和函数指针类似。
- 下例中someBlock是变量名，void是返回类型，后面小括号内是参数。

``` objective-c
void (^someBlock)() = ^{
  //块内容
};

int (^addBlock)(int a, int b) = ^(int a, int b){
  return a + b;
};
int add = addBlock(2, 5); //7

__block NSUInteger count = 0;
```
- 块可以捕获声明其范围内的所有变量。默认情况下，块捕获的变量，在块里不能修改。不然编译器会报错。声明变量时使用__block 修饰符，就可以在块内修改了。
- 内联块，在函数里传入块作为参数。如果不用块，需要传入函数指针或选择子，代码松散，使用块可以将业务逻辑都放一起。
- 块如果捕获对象，会自动保留。释放块时一并释放对象。块可视为对象，有引用计数。
- 块总能修改实例变量，无需加__block，如果捕获对象的实例变量会把self一并捕获，因为实例变量与self所指代的实例是关联在一起的。直接访问实例变量和通过self访问是等效的。块捕获self会保留它，如果self也保留了块，会导致循环引用。

``` objective-c
//@interface 某class
- (void)anInstanceMethod {
  void (^someBlock)() = ^{
    _anInstanceVariable = @"something";
    //self->_anInstanceVariable = @"something"; 两种写法等效
  };
}
//@end
```
- 块本身是对象，有isa指针，指向Class对象。其中invoke是函数指针，指向块的实现代码。接受一个void*类型的变量，代表块。descriptor变量是结构体指针，其中有块的大小，还有copy和dispose两个辅助函数对应的函数指针。copy要保留捕获的对象，dispose释放。
- 块会将捕获的变量拷贝一份，拷贝的不是对象本身，而是指向对象的指针变量。
- 定义块的时候，内存分配在栈上。块只在定义它的范围内有效，下面的代码就可能出错。超出if的范围，块的内存可能被覆写掉。

``` objective-c
void (^block)();
if (/* 某条件 */) {
  block = ^{
    NSLog(@"Block A");
  };
} else {
  block = ^{
    NSLog(@"Block B");
  };
  /* block = [^{
    NSLog(@"Block B");
  } copy];
  复制后就不会出错了
  */
}
```
- 给块对象发送copy消息，可以将块从栈复制到堆。拷贝后的块，可以在定义其的范围外使用。复制到堆上后，块就有引用计数了。后续的复制，只会递增其引用计数。ARC下块对象也会自动释放。copy见上面代码。
- 全局块，不捕捉任何变量，其使用的内存区域在编译期确定。全局块声明在全局内存里。其拷贝是个空操作，实际上是单例。

### 38 块的typedef

- 给块类型起别名，使代码易读。修改块类型比较方便，例如加个参数，改后所有用到该类型的地方都会报同一种错误，可逐个修复。
- 给同一个块起多个别名，用在不同地方。

``` objective-c
typedef int(^EOCSomeBlock)(BOOL flag, int value);
EOCSomeBlock block = ^(BOOL flag, int value){};
//块作为参数
- (void)startWithCompletionHandler:(void(^)(NSData *data, NSError *error))completion;
typedef void(^EOCCompletionHandler)(NSData *data, NSError *error);
- (void)startWithCompletionHandler:(EOCCompletionHandler)completion;
```

### 39 handler块

- iOS系统发现主线程阻塞一段时间后，会令其终止。用块来代替代理，代码较清晰易懂整洁。
- 代理模式的缺点，如果有多个代理，要在回调方法里根据传入的代理切换不同的处理。这种做法，需要将多个代理保存为实例变量。而块可以将回调与创建对象放在一起。
- 可以用两个块分别处理成功和失败的情况，也可以将成功和失败放一个块里。放一个块里更灵活，代码也更长。还有个好处是处理数据过程中发现错误，可以将其视为失败，这样可以方便的转化为错误情况。建议用同一个块处理成功与失败的情况。(代码159页)
- 块可以指定在某队列执行，若没有安排队列，则由调用api的线程执行。

``` objective-c
- (id)addObserverForName:(NSString *)name object:(id)object queue:(NSOperationQueue *)queue usingBlock:(void(^)(NSNotification*))block;
```

### 40 块与循环引用

- 一个三方导致的循环引用，打破保留环的办法是将其中某个变量的引用置为nil。（代码163页）
- 也可以在块执行后，将引用置为nil（代码165页）
- 块作为参数而不是公共属性使用，因为公共属性是可以自由使用的，不能在内部将其清理掉，会破坏封装语义。
