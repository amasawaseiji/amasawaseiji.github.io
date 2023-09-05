---
title: iOS【52个方法】第二篇
date: 2023-07-01 12:47:28
tags:
- 图书
- 专业技术
- iOS
---
### 14 类对象

- id类型能指代任意oc对象，对象类型并非在编译期就绑定好，而是在运行期查找。即使用了动态新增技术，编译器也认为能在某头文件中找到方法的定义，了解完整的方法签名，生成派发消息的正确代码。
- 运行期检查对象类型也叫类型信息查询，这个特性内置于NSObject协议里，凡是由NSObject与NSProxy继承来的对象都要遵从此协议。
- oc对象实例是指向某块内存数据的指针，不能把对象内存分配在栈上，id类型本身是指针。如果调用了对象没有的方法，编译器会知道，并给出警告信息。
- 对象的数据结构定义在运行期库的头文件里。isa指针定义了对象所属的类。Class对象也定义在其中。类对象的结构体存放类的元数据，例如实现方法，实例变量等。Class对象本身也是oc对象，其isa指向元类，表述类对象本身的元数据，定义了类方法。类方法可理解成类对象的实例方法。(57页类继承体系图)

``` objective-c
typedef struct objc_object {
  Class isa;
} *id;
```
``` objective-c
typedef struct objc_class *Class;
struct objc_class {
  Class isa;
  Class super_class;
  const char *name;
  long version;
  long info;
  long instance_size;
  struct objc_ivar_list *ivars;
  struct objc_method_list **methodLists;
  struct objc_cache *cache;
  struct objc_protocol_list *protocols;
}
```
- 经由类布局关系图，执行类型信息查询，能查到对象能否响应某选择子，是否遵从某协议。
- isMemberOfClass 判断对象是否为某类的实例，isKindOfClass 判断对象是否为某类或其派生类的实例。

``` objective-c
NSMutableDictionary *dic = [NSMutableDictionary new];
[dic isMemberOfClass:[NSDictionary class]]; //NO
[dic isMemberOfClass:[NSMutableDictionary class]]; //YES
[dic isKindOfClass:[NSDictionary class]]; //YES
```
- 还有个判断对象是否为某类实例的方法，类对象是单例，每个类的Class仅有一个实例。但应尽量有前一种，因为其可以处理使用了消息转发机制的对象。比如，代理proxy，其以NSProxy为根类。isKindOfClass传入代理对象可以返回接受代理的对象（因消息转发），class调用者是代理对象则返回的是发起代理的对象。

``` objective-c
if ([object class] == [SomeClass class]) {

}
```

### 15 前缀命名

- 命名冲突，链接过程会出错，出现重复符号，duplicate symbol.重复符号包括本身的类和元类。若是运行期载入含有重名类的程序库，可能导致程序崩溃。
- 前缀可与公司、程序关联的名字，自己用的前缀应该是3字母的，因苹果声明其保留使用2字母前缀的权利。
- 既有类增加分类，分类及分类中的方法也应加前缀。类的实现文件中所用的纯C函数和全局变量也应该加前缀。（61页代码示例）这么做在栈回溯信息中易判断问题源自哪块代码。
- 使用第三方库，自行修改后再发布供别人使用，应给自己的那份加上前缀。否则别人同时引用你这份和原始的那份，就会冲突。还可能有的问题是，版本不一样必须要引入2份，或者别的第三方库也引用了原始的库而没有改名。

### 16 指定初始化方法

- 指定一个初始化方法（需要在文档指明），别的初始化方法都来调用它。其中会存储内部数据，当数据存储机制改变时，只需改动此方法即可。调用非指定初始化方法，应该在其中调用指定初始化方法并给实例变量默认值，或者干脆抛出异常，告诉使用者不让调。

``` objective-c
- (id)init {
  @throw [NSException exceptionWithName:NSInternalInconsistencyException reason:@"Must use initWithWidth:andHeight: instead." userInfo:nil];
}
```
- 指定初始化方法的调用链一定要维系，子类的指定初始化方法要调用父类的指定初始化方法。如果子类的指定初始化方法和父类的不一致，子类应增加覆写父类的指定初始化方法，以符合子类的需求。覆写也不能符合子类的需求，应在该覆写的方法中抛出异常。(示例66页)
- 多个指定初始化方法，对象的实例有两种不同的创建方式，必须分开处理。NSCoding协议的初始化方法，下面代码。该方法和普通的初始化方法不同，要通过解码器将数据解压缩。这种情况下，出现了两个指定初始化方法。如果父类也实现了NSCoding，则子类的initWithCoder方法应调用父类的initWithCoder方法。（代码68页）
- NIB文件是XML格式，存放视图控制器和视图。加载NIB时解码。

``` objective-c
- (id)initWithCoder:(NSCoder *)decoder;
```
- 子类的指定初始化方法，应调用其父类的对应方法，逐层向上。

### 17 description

- 需要生成到控制台的字符串，对象会收到description消息。对象如果是自定义类，将只会打印类名和内存地址。内存地址只有想要判断两对象是否为同一指针才有用。
- NSObject 并不是唯一根类，许多方法定义在NSObject协议里，NSProxy也遵从NSObject协议。

``` objective-c
NSLog(@"object = %@", object);
```
- 要想输出有用信息，需要覆写description方法。

``` objective-c
- (NSString *)description {
  return [NSString stringWithFormat:@"<%@:%p, \"%@ %@\">", [self class], self, _firstName, _lastName];
}
```
- 一种简单的方法，借助字典的description方法，把待打印的信息放到字典里，然后返回的字符串中加入字典对象。

``` objective-c
- (NSString *)description {
  return [NSString stringWithFormat:@"<%@:%p, %@>", [self class], self, @{
    @"firstName":_firstName,
    @"lastName":_lastName
  }];
}
```
- debugDescription方法在调试器中以控制台命令打印对象时才调用。LLDB的po命令可以打印对象。类名和指针地址信息可以放debugDescription中输出，而普通的实例变量信息则可以放在description中输出。NSArray类就是这么做的。

### 18 尽量使用不可变对象

- 尽量把对外公布的属性设计成只读，在确有必要时才对外公布。改只读的属性值，编译就会报错。只读属性有必要写内存管理属性，以后想改为读写，会简单一些。
- 如果想要内部修改，外部只读，可以在内部将readonly重新声明为readwrite，在匿名分类（扩展）里重新声明，只改读写，其他的特质不改。但是外部依然可以通过kvc来改。
- nonatomic 会产生竞争条件，如想避免，可通过队列来做，将所有数据存取操作都设置为同步操作。
- 用类型信息查询功能，查出属性对应的实例变量在内存布局中的偏移量，来设置变量值。这样更不合规范。要尽量用不可变对象。
- 属性为集合，应对外提供一个只读属性，返回不可变的set,此set是内部可变set的拷贝。如果用mutableSet来做，容易出bug.在对象不知情时从底层修改set会令对象内各数据不一致。
- 不要在返回的对象上查询类型确定其是否可变，不要假定返回的set是可变的，因为可能是开发者漏写了拷贝，实际上并不想让你改。
- 不要把可变的集合类型公开，而应提供方法修改对象中的可变集合。

### 19 命名方式

- 驼峰命名变量、类名，类名首字母大写，变量首字母小写。方法名不宜太长，也不能太简略。
- NSString类的命名参考。

``` objective-c
+ string //创建空字符串 描述了返回值类型
+ stringWithString //创建与给定字符串内容相同的字符串 指明了返回类型
+ localizedStringWithFormat: //创建本地化字符串 第二个单词指明返回类型
- lowercaseString //包含返回值类型
- intValue //方法名凑足两个词，单词常表示属性
- length //实际上是属性
- lengthOfBytesUsingEncoding: //返回字节数组的长度 包含参数类型
- (void)getCharacters:(unichar*)buffer range:(NSRange)aRange; //获取字符串中给的范围内的字符
// 首个参数中传入数组 取到的字符放入其中，所以用get
// 经由参数来返回而不是返回值返回
// 提及参数类型 强调某参数，可以用介词in之类
- hasPrefix: //返回值是boolean类型，通常含有has一词
- isEqualToString: //返回值是boolean类型 方便述说用is.boolean类型的属性，例如属性enabled,其获取方法是isEnabled
```
- 命名规则：返回值是新建的，首词应指明返回值类型，除非还要修饰词。存取方法不尊法这一规则。
- 参数前要有参数类型
- 类型词不要用简称，如要用string 而不是str
- 布林属性前应加is，非属性，看情况用has和is
- get应用于输出参数当返回值的方法
- 超类前加修饰语是常用的命名习惯，UITableView
- UITableViewDelegate 命名时将定义协议的那个类放前面，后面加上delegate

### 20 私有方法名加前缀

- 原因在于，便于修改方法名或方法签名。容易分辨出哪些是私有方法可以修改，哪些是公共方法不能轻易修改。前缀最好包含下划线和字母p（p表示私有private）,下划线做区隔.私有方法只在实现的时候声明。

``` objective-c
- (void)p_privateMethod {

}
```
- oc没办法将方法标为私有，对象可响应任意消息，运行期会检查所能直接响应的消息，查出对应的实现方法。
- 苹果公司单用一个下划线表示私有方法，并不建议开发者这样用。因为可能继承时会造成覆写了父类(苹果定义的类)的方法。
- 不知道子类继承的类的私有方法所加的前缀是什么，应使用自己的类名前缀做私有方法的前缀。

### 21 错误模型

- 抛出异常，本应在作用域末尾自动释放对象也不能自动释放了。开启异常安全通过编译器的标志实现，-fobjc-arc-exceptions
- 抛出异常经常会导致内存泄露，应在不得已才抛出异常，是极其严重的错误，退出应用程序。例如，不能直接使用的父类，在子类必须覆写的方法里抛出异常。
- 一般错误，令方法返回nil/0，或使用NSError。调用者发现方法返回了nil，从而确定其中发生了错误。
- NSError包含的信息，Error domain错误范围，类型为字符串，通常用全局变量来定义。例如，NSURLErrorDomain。Error code 错误码，类型为整数，用enum定义，例如http状态码设为错误码。User info 用户信息，类型为字典。额外信息，可包含错误的本地化描述。
- 委托协议传递错误，调用者可自行决定是否处理此错误。另一种是经由方法的输出参数返回，参数是指针，指针指向NSError对象的指针，可以把NSError对象传递给调用者。这样的方法一般返回boolean值，表示该操作是否成功。若不关注错误，判断该boolean值就好。若不想知道错误，可以给error参数传入nil

``` objective-c
- (BOOL)doSomething:(NSError **)error
NSError *error = nil;
BOOL ret = [object doSomething:&error];
if (error) {
}
```
- arc时编译器会将NSError ** 转成 NSError * __autoreleasing * ,指针所指的对象会在方法执行后自动释放，因为doSomething方法不能保证其调用者可以将在方法内创建的NSError释放掉，所以需要加入autorelease。下列代码需要先判断error是否为nil，对空指针解引用会使程序崩溃

``` objective-c
- (BOOL)doSomething:(NSError **)error {
  if (/* 发生了错误 */) {
    if(error) {
      *error = [NSError errorWithDomain:domain code:code unserInfo:userInfo];
    }
    return NO;
  } else {
    return YES;
  }
}
```
- 错误信息定义

``` objective-c
//.h
extern NSString *const EOCErrorDomain;
//.m
NSString *const EOCErrorDomain = @"EOCErrorDomain";
typedef NS_ENUM(NSUInteger, EOCError) {
  EOCErrorUnknown = -1,
  EOCErrorBadInput = 500,
  //...
}
```

### 22 NSCopying协议

- 如果想让自己的类支持copy操作，就要实现NSCopying协议。zone是内存区，现在每个程序只有一个区，历史原因，现在可不用管。copy方法实际上会调用copyWithZone。

``` objective-c
- (id)copyWithZone:(NSZone*)zone
- (id)mutableCopyWithZone:(NSZone*)zone
```
- 遵从协议，实现copyWithZone方法。用全能初始化方法。如果对象中的数据并未在初始化方法中设置好，而需要另行设置。那么拷贝时也要多一些操作。->语法引用实例变量。如果_firends是不可变的则不用复制，复制了内存中将有两个一样的_firends造成浪费。可变的需要复制，不能引用同一个。
- 有些时候不能用全能初始化方法来做，例如初始化方法要设置一个复杂的内部结构，而拷贝后这个数据又要立马覆写。这样就没必要在复制的时候设置一遍。

``` objective-c
- (id)copyWithZone:(NSZone*)zone {
  EOCPerson *copy = [[[self class] allocWithZone:zone] initWithFirstName: _firstName andLastName: _lastName];
  copy->_firends = [_firends mutableCopy];
  return copy;
}

```
- NSMutableCopying协议，如果自己的类包含可变版本就应该实现该协议。若要获取可变版本的拷贝均应调用mutableCopy方法，若获取不可变版本，应调用copy方法。
- 深拷贝和浅拷贝，深拷贝意为拷贝对象本身也将其底层数据一并复制。集合类都执行浅拷贝，只拷贝容器本身，不复制其中数据。因为容器内对象未必都能拷贝。

``` objective-c
- (id)initWithSet:(NSArray *)array copyItems:(BOOL)copyItems
```

- 上面方法，若copyItems参数设置为YES，则会向其数组中每个对象发送copy消息，用拷贝好的元素创建新的set，返回给调用者。意思为执行深拷贝。
- 遵从NSCopying协议，大多都是执行浅拷贝，若要执行深拷贝，除非说明了用深拷贝来实现NSCopying协议的，否则要么找相关方法，要么自己写方法来做。

### 23 委托和数据源协议

- 委托协议名称，通常是在相关类名后面加上Delegate一词，用驼峰法来写。用weak，因为strong会造成循环引用。也可以是unsafe_unretained，相关对象销毁时不需要自动清空可用。weak是销毁时需要清空使用。委托协议通常在类内部使用，在扩展里声明遵从协议即可。若要公布则在@interface 后声明。用@optional 标注协议中的可选方法，委托协议里的方法通常是可选的。

``` objective-c
@property (nonatomic, weak) id <EOCNetworkFetcherDelegate> delegate;
```
- 因有可选方法，用respondsToSelector: 方法判断能否响应相关选择子，能响应再调用。这样就不用担心因方法没实现而导致的问题。即便没设置delegate，也不用担心错误，给nil发消息返回nil.
- 委托方法里应把发起委托的实例一并传入。因为发起委托的实例可能有多个，可以根据实例判断来执行不同的操作。
- 数据源模式旨在向类提供信息。可以把能否响应协议的某个方法这一信息缓存起来，优化程序效率。可以使用位段数据结构，这一c语言特性，把结构体中某个字段所占的二进制位个数设为特定值。值设为1表示1个二进制位。可以把实现缓存功能的代码放在delegate属性的设置方法中。调用方法时，检测结构体里的标志即可。如果某一方法需要多次调用，这项优化可提高程序效率。

``` objective-c
@interface EOCNetworkFetcher() {
  struct {
    unsigned int didUpdateProgressTo : 1;
  } _delegateFlags;
}
@end
```

``` objective-c
- (void)setDelegate:(id<EOCNetworkFetcherDelegate>)delegate {
  _delegate = delegate;
  _delegateFlags.didUpdateProgressTo = [delegate respondsToSelector:@selector(networkFetcher:didUpdateProgressTo:)];
}
```

``` objective-c
if (_delegateFlags.didUpdateProgressTo) {
  [_delegate networkFetcher:self didUpdateProgressTo:currentProgress];
}
```

### 24 分类

- 把代码按逻辑划分到几个分区中，都开发和调试有好处。可以把分类拆到各自的文件中去，EOCPerson+Friendship(.h/.m)，使用时要引入分类的头文件。
- 便于调试，分类中的方法，分类名称会出现在符号中，在调试器的回溯信息中也能看到。例如[EOCPerson(Friendship) addFriend:].
- 对私有方法，可以创建Private的分类，其方法只在框架内部使用。如果内部某个地方需要使用，就引入分类的头文件，分类的头文件不随程序库一并公布。

``` objective-c
@interface EOCPerson (Friendship)
//方法
@end
```
``` objective-c
#import "EOCPerson.h"

@interface EOCPerson (Friendship)
@end

//.m
#import "EOCPerson+Friendship.h"
@implementation EOCPerson (Friendship)
@end
```

### 25 第三方类的分类名加前缀

- 分类机制用来给既有类中新增功能。运行期系统加载分类时将分类方法加入类的方法列表中，如果类本有的方法，分类又实现了一次，将会覆盖原实现，还有可能会有多次覆盖，多次覆盖以最后一个为准。覆盖时机，谁先谁后不好说。
- 要解决这个问题，就是给相关名称加前缀，包括分类名和其中的方法名。分类重名不会出错，会给出警告。前缀为你专用的前缀。

``` objective-c
@interface NSString (ABC_HTTP)
- (NSString *)abc_urlEncodedString;
@end
```

### 26 分类中不要声明属性

- 分类上可以声明属性，但不要这么做。因为除了扩展，其他分类都无法向类中新增实例变量。属性所需的实例变量无法合成出来。这么做的话，编译器会给出警告信息。提示无法合成实例变量，需要开发者在分类中实现存取方法。如果用@dynamic在运行期提供，并用消息转发机制在运行期拦截方法调用，提供其实现，才可以这样做。
- 可以用关联对象解决分类里不能合成实例变量的问题。这样做可行但不建议这么做。因为在内存管理上容易出错，修改属性的内存管理语义，在设置方法里也要改。如果属性是可变的版本，设置时就要拷贝为可变版本。这样容易出错。
- 属性和实例变量都应定义在主接口里，分类的目标是扩展类功能，而不是封装数据。
- 只读属性可以在分类中使用，其不需要实例变量，只加一个获取方法。实现了其获取方法，实例变量也不会自动合成了。但也不建议这么做，属性表示有数据支撑，是用来封装数据的。这里应该只声明一个方法用来湖获取值就够了。

``` objective-c
#import <objc/runtime.h>
static const char* kFriendsPropertyKey = "kFriendsPropertyKey";
@implementation EOCPerson (Friendship)
- (NSArray *)friends {
  return objc_getAssociatedObject(self, kFriendsPropertyKey);
}
- (void)setFriends:(NSArray *)friends {
  objc_setAssociatedObject(self, kFriendsPropertyKey, friends, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
@end
```
