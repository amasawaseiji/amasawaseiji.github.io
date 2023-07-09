---
title: iOS【52个方法】
date: 2022-07-02 10:46:30
tags:
- 图书
- 专业技术
- iOS
---
### 1

- 消息结构和函数调用。消息结构：运行时应执行的代码由运行环境决定，运行时查找应执行的方法。**接收消息的对象也在运行时处理，如检查对象类型，叫做动态绑定**。函数调用：由编译器决定。若函数是多态，运行时按照virtual table(虚方法表)查出应执行的函数实现。

- 运行期组件本质是动态库，含有全部内存管理方法。oc运行期环境抽象出内存管理架构，叫做**引用计数**

- oc中指针用来指示对象。对象内存分配在堆(heap)中，不是栈(stack)上。指针变量指向堆中内存，两个指针变量可指向同一对象。指针变量在栈里，32位架构4字节，64位8字节，内容为指向对象的内存地址。堆内存需直接管理，栈内存栈帧弹出时自动清理。

- c结构体，如CGRect，使用栈空间。如只需保存int、float、double、char等非对象类型，用结构体就行，用OC对象会影响性能，有额外开销如分配和释放内存。

### 2

- @class 向前声明(forward declaring)，使用某类，不需知道其全部细节，只需知道类名。.h用@class，.m文件引入某类头文件。减少需引入的头文件数量，.h直接引入，会增加编译时间。

- 向前声明解决了两个类互相引用的问题。两个类在头文件里互相引用，会造成循环引用。#include循环引用导致死循环。#import会导致其中一个类无法正确编译。

- 声明类遵从协议(protocol)，@interface classA : classB<protocol>，需要知道协议的完整定义，不能使用向前声明。这种情况，最好把协议单独放一个头文件中。若是放其他类的头文件里，因为不能用向前声明，必须引入此头文件。1可能产生互相引用问题，2增加编译时间。

- 代理协议不用单独写一个头文件，因为协议只有与定义代理的类放一起才有意义。此时的做法应该在.m文件里声明代理协议，这部分代码放在class-continuation(私有扩展)里。这样只需在.m文件中引入包含代理协议的头文件即可。

- 若有属性、实例变量或遵从协议需要引入头文件，应尽量放在私有扩展里。作用：**降低类之间耦合，即降低类之间依赖程度，减少编译时间**

### 3

- 字面量语法，能缩减代码长度，易读，整洁，精简，安全。@"string",@1(int),@2.5f(float),@3.14159(double),@YES(bool),@'a'(char)。表达式：@(x*y)。字面量创建数组@[@"1"]，取下标arr[1]。

- 字面量语法创建数组，有nil会抛异常，因为其实质上是语法糖，效果等于先创建数组再将方括号内对象加入数组中。arrayWithObjects:不会报错，但是会舍弃遇到nil之后的元素。

- 字面量语法创建字典，也是一旦有值为nil，就会抛出异常。dic[@"key"]。

- 可变数组、字典可用取下标赋值方式直接修改。mArr[1] = @"xx"，mDic[@"key"] = @"xx"。

- 字面量语法创建的字符串、数组、字典都是不可变的。若要变成可变，需要mutableCopy一份。

### 4

- #define预处理指令定义常量，等效替换，会替换所有遇到的，无类型信息。
``` objective-c
static const NSTimeInterval kAnimationDuration = 0.3;
```
这种方式包含类型信息，清楚描述常量含义。

- 常量命名方法：若常量局限于某.m文件内使用，加字母k作为前缀。若类外部也可见，应以类名为前缀。oc没有命名空间(namespace)。

- 常量声明位置：若不打算公开，应定义在.m文件内，#import下面。

- const代表不可修改，若修改编译器会报错。static修饰符意为变量仅在定义它的.m文件(编译单元)可见，作用域为.m文件生成的目标文件内(object file)。如果不加static，编译器会将之视为外部符号，若别的.m文件也声明了同名变量，编译会报duplicate symbol 重复符号错误。实际上 static const 效果等同于#define指令，只不过前者带有类型信息。

- 全局常量：如通知名称，放在全局符号表(global symbol table)中。可以在定义该常量的编译单元之外使用。
``` objective-c
// .h文件
extern NSString *const EOCStringConstant;
// .m文件
NSString *const EOCStringConstant = @"value";
```
const表明指针常量不应改变。extern关键字告知编译器，全局符号表将会有该符号，无需查看定义，就允许使用该常量。生成二进制文件后，肯定能找到此常量。这类常量必须定义，只能定义一次。定义在声明该变量头文件相关的.m文件内。编译时存储在数据段(data section)。名称应以相关类名做前缀，如UIKit中的UIApplicationEnterBackgroundNotification.
``` objective-c
// .h
extern const NSTimeInterval EOCAnimatedViewAnimationDuration;
// .m
const NSTimeInterval EOCAnimatedViewAnimationDuration = 0.3;
```

- #define 定义常量可能会被修改，例如重复定义，从而引起错误。

### 5

- 枚举enum，是一种常量命名方式，状态码或可组合选项适宜用枚举。定义枚举集，易读懂。编号从0开始递增1.
``` objective-c
enum EOCConnectionState {
    EOCConnectionState1,
    EOCConnectionState2,
    EOCConnectionState3,
};
enum EOCConnectionState state = EOCConnectionState1;
typedef enum EOCConnectionState EOCConnectionState;
EOCConnectionState state1 = EOCConnectionState1;
```
- C++11新特性，可以用底层数据类型保存枚举类型变量，这样可以向前声明枚举变量了。不指定底层数据类型，编译器不知道数据类型大小，不知道该给变量分配多少空间，就无法使用向前声明。还可以手动指定枚举成员的值。
``` objective-c
enum EOCConnectionState : NSInteger;
enum EOCConnectionState {
  EOCConnectionState1 = 1,
  ···
}
```

- 定义选项，选项可组合。用按位或操作来组合。按位与操作判断是否开启某项。
``` objective-c
enum EOCConnectionState {
  EOCConnectionState1 = 0,
  EOCConnectionState2 = 1 << 0,
  EOCConnectionState3 = 1 << 1,
  EOCConnectionState3 = 1 << 2,
  ···
}
enum EOCConnectionState state = EOCConnectionState2 | EOCConnectionState3;
if (state & EOCConnectionState3) {
  // EOCConnectionState3 is set
}
```

- Foundation框架里辅助的宏，具备向后兼容能力，编译器支持新标准，就用新语法，否则用旧语法。
``` objective-c
typedef NS_ENUM(NSUInteger, EOCConnectionState) {
  EOCConnectionState1,
  ···
};
typedef NS_OPTIONS(NSUInteger, EOCConnectionState) {
  EOCConnectionState1 = 1 << 0,
  EOCConnectionState1 = 1 << 1,
  EOCConnectionState1 = 1 << 2,
  ···
};
```
NS_OPTIONS宏在C++模式下可省去类型转换操作，所以需要按位或操作来组合的枚举都应使用NS_OPTIONS宏定义，不需要组合用NS_ENUM定义。

- switch语句里，用枚举定义状态机，不用default分支。这样新加入枚举，编译器会有警告信息。

### 6

- 在类定义下使用  @public @private (即在@interface xx:NSObject {} 大括号中)定义变量的作用域存在问题，对象布局在编译期就已固定，使用偏移量硬编码，表示变量距离对象内存区域多远。修改类定义之后必须重新编译，不然就会出错。oc应对此问题的办法是把实例变量当做存储偏移量所用的特殊变量，由类对象保管。这样偏移量在运行期查找，定义变了偏移量也变了，就总能使用正确的偏移量。还可以在运行期向类中新增实例变量。可在分类或实现文件中定义实例变量。

- @property 属性，编译器会自动写出一套存取方法，用于访问变量。这个过程由编译器在编译期执行。除此之外，编译器还会向类中添加实例变量，并在属性名前加下划线作为变量名。访问属性使用点语法，相当于调用存取方法。

- 在.m文件里指定实例变量的名字，@synthesize firstName = _myFirstName;一般无需修改默认变量名。

- 存取方法可以自己实现一个，另一个还是由编译器生成。@dynamic 意思是不让编译器创建实例变量也不创建存取方法。而是在运行期动态创建存取方法。NSManagedObject子类的某些属性不是实例变量，数据来源于数据库。
``` objective-c
// .h
@property NSString *firstName;
// .m
@dynamic firstName;
```

- 属性特质（attribute）分四类。
- 其一原子性：某操作具备整体性，其中间步骤生成的临时结果，其他部分看不到，只能看到操作前后的结果。编译器合成的方法默认是原子的，否则写成nonatomic。自定义存取方法应遵从与其特质相符的原子性。
- 其二读写权限，readwrite，拥有获取方法和设置方法。如果属性由@synthesize实现，则编译器自动生成这两个方法。（？存疑）。readonly 仅有获取方法，仅当属性由@synthesize实现，编译器为其合成获取方法。
- 内存管理语义。assign,设置方法执行纯量类型的赋值(CGFloat或NSInteger)，strong,拥有关系，设置新值时会先保留新值，释放旧值，再将新值设置上去。weak,不保留新值，不释放旧值，同assign类似，属性对象摧毁时，值会清空，nil out. unsafe_untrtained,同assign相同，适用于对象类型，属性对象摧毁时，值不会清空。copy,与strong类似，设置方法不保留新值，而是拷贝，例如NSString*,用此特质保护其封装性。其有NSMutableString子类，不为copy则属性值可被改动，copy确保对象值不会无意间改动。
- 方法名，getter=name,指定获取方法的方法名，例@property (getter=isOn) Bool on;setter=name,不常见。
- 自己实现存取方法，应保证具备属性所声明的特质。copy特质的属性，其设置方法里应用copy复制。
- atomic原子性是什么意思？两个线程读写同一属性，总能看到有效的属性值。不然，线程读取到的属性值可能不对。
- 为什么所有属性都是nonatomic,有历史原因，同步锁的开销太大，性能问题。其次，atomic也不能保证线程安全。iOS一般用nonatomic，macos永atomic不会有性能瓶颈。

### 7

- 读取实例变量时用直接访问的形式，设置实例变量时通过属性。为什么？设置时使用属性，能确保内存管理语义得以贯彻。其二能提高读取的速度。
- 直接访问速度快，直接访问实例变量的那块内存。绕过了属性定义的内存管理语义。不会触发键值观测KVO.
- 通过属性访问，有助于排查错误，可在存取方法里打断点。
- 初始化方法中，应直接访问实例变量，为什么？因为子类可能会覆写设置方法。在父类初始化方法中，将属性设置一个值，若是用设置方法来做，调的是子类的设置方法。（子类没实现初始化方法而使用子类的情况）
- 初始化方法中必须调用设置方法，实例变量声明在父类，子类中无法直接访问的情况。(?存疑，代码中是哪种情形)
- 懒加载，必须用获取方法。为什么？懒加载的意思是初始化的时候不给属性赋值，使用获取方法的时候才赋值。直接访问，可能会得到未设置值的实例变量。

### 8

- ==操作符比较的是两个指针本身，而不是其所指的对象。应该用 isEqual方法判断对象的等同性。例如NSString类，==判断不等同，isEqual 以及 isEqualToString 判断等同。而isEqualToString比isEqual速度快，后者还要执行额外的步骤，因为它不知道对象类型。
- NSObject有2个判断等同性的方法，isEqual 和hash.其实现都是判断指针值相等，两对象才相等。isEqual相等，hash同值，hash同值，isEqual不一定相等。
- isEqual 覆写的情况，先判断指针相等(即==号)即相等，然后比较是否同一个类，（考虑父子类是否相等的情况，有时认为相等），最后检查每个属性是否相等。
- 集合是数组、字典和set的总称。集合会使用对象的hash码做索引，如果对象的hash码都相同，将产生性能问题。set会根据hash码把对象分装到不同的数组，向其中添加新对象时，会先找到与之相关的数组，再依次检索每个元素，查看是否有等值的对象。所以如果哈希码相同，将会检索所有的元素，产生性能问题。

``` objective-c
- (NSUInteger)hash {
  return 1337;
}
```
- 另一种实现，将对象的属性都塞入另一个字符串中，返回该字符串的hash码。这样在添加到集合也会有性能问题，因想要添加必须计算其哈希码。

``` objective-c
- (NSUInteger)hash {
  NSString *str = [NSString stringWithFormat:@"%@:%@:%i",_firstName, _lastName, _age];
  return [str hash];
}
```
- 最后一种实现，如下代码。既能保持高效，又能使哈希码位于一定范围内，不会频繁重复。编写哈希方法，应在减少碰撞频度和降低运算复杂度之间做取舍。

``` objective-c
- (NSUInteger)hash {
  NSUInteger firstNameHash = [_firstName hash];
  NSUInteger lastNameHash = [_lastName hash];
  NSUInteger ageHash = _age;
  return firstNameHash ^ lastNameHash ^ ageHash;
}
```

- isEqualToArray 和 isEqualToDictionary。编写判断等同性的方法，应覆写isEqual方法，isEqual方法常见的写法为先判断调用者和参数是否是同一类，如是同类调自己编写的等同性方法，不同则调用父类的。

``` objective-c
- (Bool)isEqual:(id)obj {
  if ([self class] == [object class]) {
    return [self isEqualToPerson:(EOCPerson *)obj];
  } else {
    return [super isEqual:obj];
  }
}
```

- 等同深度判定，数组判断等同，先看元素个数，个数相同，再看每个位置的元素是否相同，都相等才认为是数组相等。是否要检测每个属性都相等才判断等同，看情况。例如数据库数据创建的对象，其等同性判断仅需要看其唯一标识符，其标识符是做主键的。
- 集合中加入可变类型的对象，加入之后不应该修改。例如set中装入2个数组，改变其中一个和另一个完全相同，此时set中会有两个相同的数组。再拷贝，又会变为只剩一个。加入之后再修改是有隐患的，后面的行为难预料。(35页示例代码)

### 9

- 类族，一个基类，多个子类，各子类代表不同类型。基类接口中没有名为init的方法，暗示该类实例不该直接创建。isMemberOfClass: 调用者是子类，参数是父类，会返回NO.
- 可变与不可变数组可相互转换。下面代码的if条件不可能为真，NSArray初始化方法返回的实例类型是某种内部类型。下面第二段代码，可判断实例是否在类族中。
- 创建类族的子类，需要继承自基类，定义自己的数据存储方式，覆写基类中的需要覆写的方法。

``` objective-c
id maybeAnArray = /* ... */;
if ([maybeAnArray class] == [NSArray class]) {
}
```

``` objective-c
if ([maybeAnArray isKindOfClass:[NSArray class]]) {
}
```

### 10

- 关联对象，设置关联对象通过键来区分，可指明存储策略，以维护内存管理语义。存储策略是个枚举。
|枚举类型|等效于|
|:-:|:-:|
|OBJC_ASSOCIATION_ASSIGN|assign|
|OBJC_ASSOCIATION_RETAIN_NONATOMIC|retain,nonatomic|
|OBJC_ASSOCIATION_COPY_NONATOMIC|copy,nonatomic|
|OBJC_ASSOCIATION_RETAIN|retain|
|OBJC_ASSOCIATION_COPY|copy|

- 方法，分别为设置值，取值，移除所有关联对象。key是不透明指针，同指针才能匹配到同一个值，用静态全局变量做键。例子，给UIAlertView关联block,将按钮操作放入block中。（代码示例42页）
``` objective-c
void objc_setAssociatedObject(id object, void *key,id value,objc_AssociationPolicy policy)
id objc_getAssociatedObject(id object, void*key)
void objc_removeAssociatedObjects(id object)
```
``` objective-c
static void* key = "key";
```
- 关联对象，不能滥用，关联对象之间的关系没有正式定义，内存管理语义是在关联的时候才确定，出现循环引用难查。
- 如果多次使用到类似UIAlertView这种视图，用block代替代理要比使用关联对象好。

### 11 objc_msgSend

- OC是C的超集，C语言在编译期就能决定运行时所调用的函数，函数地址硬编码在指令中。如果调用的函数在运行期才确定，就要使用动态绑定，待调用的函数地址无法硬编码在指令中，要在运行期读取出来。(C的代码示例43页)
- OC传递消息是动态绑定，该调用哪个方法运行期决定，甚至可以在运行时改变。方法调用者叫做接收者，方法名叫选择子，选择子和参数合起来叫消息。方法调用背后是c语言函数objc_msgSend。
``` objective-c
void objc_msgSend(id self, SEL cmd, ...)
```
- 此函数能接受2个或以上的参数，第一个参数是接收者，第二个是选择子，后面是消息中的参数
``` objective-c
objc_msgSend(obj, @selector(messageName:), parameter);
```
- 该函数的执行原理，在接受者所属的类中搜寻方法列表中与选择子名称相符的方法，跳至其实现代码，找不到会沿着继承体系向上查找。最终还是找不到，就执行消息转发。
- 效率如何？该函数会将匹配结果缓存在快速映射表里，每个类都有这块缓存。后续再调同样的方法，执行就较快了。虽然效率还可以，还是不如C语言的静态绑定的函数调用。
- objc_msgSend_stret 处理返回结构体类型的。objc_msgSend_fpret 处理返回浮点数类型的。objc_msgSendSuper 处理给超类发消息的。[super message:para];
- oc对象的方法可视为c函数。每个类里有一张表格，其指针指向如下这种函数，方法名是查表用的键。objc_msgSend即通过这张表查找并跳至其实现的。
``` objective-c
<return_type> Class_selector(id self, SEL _cmd, ...);
```
- 尾调用优化，如果某函数的最后一个操作是调用另一个函数，就可以用尾调用优化技术。编译器会生成跳转至另一函数所需的指令码，而不会向堆栈中推入新的栈帧。只有当某函数的最后一个操作仅仅是调用其他函数，而不将其返回值另做它用。才能执行尾调用优化。如果不这么做的话，每个函数都会推入栈帧，过早造成栈溢出。

### 12 消息转发机制

- 向类发送了无法解读的消息不会报错，因为运行期还可以向类中添加方法。无法解读时，启动消息转发机制。
- unrecognized selector sent to instance ,这种提示信息就表明启动了转发机制，转发给NSObject的默认实现。这种异常是由NSObject的 doesNotRecognizeSelector 方法抛出的。__NSFCNumber 是实现无缝桥接的内部类，配置NSNumber对象时会一并创建。
- 消息转发，第一先看接收者是否能动态添加方法来处理未知的选择子，这叫动态方法解析。第二看有没有其他对象能处理消息，有就转发给该对象，结束。没有则启动完整的消息转发机制，把与消息有关的全部细节封装到NSInvocation对象中，给接收者最后一次机会，令其解决未处理的消息。
- 动态方法解析，首先调用类的下列方法。表示该类能否新增一个实例方法处理此选择子。如果未实现的方法是类方法，那么会调用resolveClassMethod。前提是相关方法实现已写好，运行的时候动态插入。

``` objective-c
+ (BOOL)resolveInstanceMethod:(SEL)selector
```
- CoreData里的NSManagedObjects对象的@dynamic属性，通常用动态方法解析来做。添加的方法是用纯C函数实现，操作相关的数据结构，类中的属性就是就存放在这些数据结构里。下例，autoDictionaryGetter和autoDictionarySetter为函数指针，class_addMethod向类中添加方法，其最后一个参数代表待添加方法的类型编码，类型编码开头表示返回值类型，后续表示各个参数。

``` objective-c
id autoDictionaryGetter(id self, SEL _cmd);
void autoDictionarySetter(id self, SEL _cmd, id value);
+ (BOOL)resolveInstanceMethod:(SEL)selector {
  NSString *selectorString = NSStringFromSelector(selector);
  if (/* selector id from a @dynamic property */) {
    if ([selectorString hasPrefix:@"set"]) {
      class_addMethod(self, selector, (IMP)autoDictionarySetter, "v@:@");
    } else {
      class_addMethod(self, selector, (IMP)autoDictionaryGetter, "@@:");
    }
    return YES;
  }
  return [super resolveInstanceMethod:selector];
}
```

- 转给其他接收者处理，能找到，该方法返回该接收者，否则返回nil.若一个对象内部，有一系列其他对象，可以经由此方法将能处理选择子的某内部对象返回。这一步所转发的消息无法操作。

``` objective-c
- (id)forwardingTargetForSelector:(SEL)selector
```

- 完整的消息转发，创建NSInvocation对象，把那条消息的全部细节封装起来，包括选择子，目标和参数。下列方法可以实现为，在触发消息前，改变消息内容，比如追加参数，更换选择子等。若发现调用操作不应由本类处理，则调用超类的同名方法，继承体系的每个类都有机会处理，直至NSObject.最后会调用doesNotRecognizeSelector抛出异常。表明选择子最终未能处理。（49页流程图）

``` objective-c
- (void)forwardInvocation:(NSInvocation*)invocation
```
- 最好在第一步就处理掉，这样运行期系统就可以缓存该方法，稍后再调用同名选择子，就无需启动消息转发流程了。
- 完整例子演示实现@dynamic属性。(50、51、52页代码)
- coreAnimation框架里的CALayer类，就是类似本例。CALayer是兼容于kvc的容器类。可以向其中添加属性，然后以键值对的方式访问。属性值的存储由基类负责，我们只需在CALayer子类中定义新属性。

### 13 方法调配method swizzling
- 类的方法列表会把选择子的名称映射到相关的方法实现之上，这些方法以函数指针的形式表现，这种指针叫IMP。运行期系统可以向映射表中新增选择子，改变选择子对应的实现，交换选择子映射到的指针。

``` objective-c
id (*IMP)(id, SEL, ...)
```
- 交换方法实现，方法实现可由函数获得(下列第二个)，直接交换意义不大。

``` objective-c
void method_exchangeImplementations(Method m1, Method m2)
```
``` objective-c
Method class_getInstanceMethod(Class aClass, SEL aSelector)
```
``` objective-c
Method originalMethod = class_getInstance([NSString class], @selector(lowercaseString));
Method swappedMethod = class_getInstance([NSString class], @selector(uppercaseString));
method_exchangeImplementations(originalMethod, swappedMethod);
```
- 为既有方法添加新功能，有意义的，可写在分类中。下列方法看上去会陷入死循环，实际上不会，因为在运行期eoc_myLowercaseString将和lowercaseString交换。实现两个方法交换，即可实现调用lowercaseString而额外输出一些内容。

``` objective-c
@interface NSString (EOCMyAdditions)
- (NSString *)eoc_myLowercaseString;
@end

@implementation NSString (EOCMyAdditions)
- (NSString *)eoc_myLowercaseString {
  NSString *lowercase = [self eoc_myLowercaseString];
  NSLog(@"%@ => %@", self, lowercase);
  return lowercase;
}
@end
```
- 借由此功能，可以为不透明方法添加日志记录功能（埋点），仅调试可以用，不宜滥用。
