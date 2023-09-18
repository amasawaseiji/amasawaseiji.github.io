---
title: iOS【Swift官方文档】第三篇
date: 2023-09-10 21:00:08
tags:
- 图书
- 专业技术
- iOS
---

### 1.20 嵌套类型

- 例如在结构体里定义枚举，在枚举中又定义结结构体
- 外部引用时用类型名来引用

```swift
let heartsSymbol = BlackjackCard.Suit.hearts.rawValue
```

### 1.21 扩展

- 可以用于枚举、结构体、类、协议，扩展没有名字，用于增加新功能，但不能重写已有的功能
- 可以添加实例和类的计算属性、定义实例和类方法、新构造器、下标、嵌套类型、遵从协议
- extension 关键字，不能添加存储属性，也不能为现有属性添加观察者

```swift
extension SomeType: SomeProtocol, AnotherProtocol {
  // 协议所需要的实现写在这里
}

extension Double {
    var km: Double { return self * 1_000.0 }
    var m: Double { return self }
    var cm: Double { return self / 100.0 }
    var mm: Double { return self / 1_000.0 }
    var ft: Double { return self / 3.28084 }
}
let oneInch = 25.4.mm
print("One inch is \(oneInch) meters")
// 打印“One inch is 0.0254 meters”
let aMarathon = 42.km + 195.m
```
- 可以给类添加便利构造器，不能添加指定构造器和析构器
- mutating 表明方法可以修改结构体、枚举的属性或自身self

### 1.22 协议

- 类、结构体、枚举都可使用协议，协议可被扩展
- 协议里的属性，需提供名称和类型以及读写状态。协议里可读写，不能是常量属性或可读计算属性。协议里可读，该属性可以可读也可以读写。
- 协议中的方法不需要方法体和大括号，不支持提供默认参数。

```swift
// 有父类这样定义
class SomeClass: SomeSuperClass, FirstProtocol, AnotherProtocol {
	// 这里是类的定义部分
}
protocol SomeProtocol {
	var mustBeSettable: Int { get set }
	var doesNotNeedToBeSettable: Int { get }
  static var someTypeProperty: Int { get set }
  //类类型可以用class关键字
}
protocol SomeProtocol {
    static func someTypeMethod()
}
```
- 线性同余伪随机数生成 .truncatingRemainder(dividingBy)
- 协议中也可定义mutating，类类型不用加此前缀，结构体和枚举需要
- 协议中定义构造器，指定和便利都行，需要加上 required 修饰符，确保子类也实现该构造器并遵从协议。如果是 final 类，不需要加 required
- 同时遵从协议和继承自父类需要加 required override
- 协议中定义可失败构造器，可实现为init?和init，协议中非可失败可实现为init和init!，隐式解包可失败构造器
- 协议作为类型使用，如作为属性的类型，任何遵从协议的实例都可以被赋值给该属性
- 委托，将协议声明为弱引用。通过 is 操作符，判断遵循协议的类型是否是具体的某个类型实例
- 协议里可读变量属性，实现可使用常量属性
- 可通过扩展让类型遵从某协议
- 有条件的遵循协议，where 关键字判断

```swift
extension Array: TextRepresentable where Element: TextRepresentable {}
```
- 类型已实现协议中的所有内容，可通过空的扩展遵循协议
- 几种遵循 Equatable 协议的类型有合成实现，只有存储属性的结构体、只有关联类型的枚举、没有任何关联类型的枚举（不需遵从协议）。实现了 == 和 != 操作符
- 几种遵从 Hashable 协议的类型有合成实现，同上。实现了 hash(into:)
- 遵从 Comparable 协议的类似有合成实现的有，没原始值的枚举类型。实现了 < <= > >=
- 协议可以继承协议
- 添加 AnyObject 关键字，限制协议只能使用于类类型

```swift
protocol SomeClassOnlyProtocol: AnyObject, SomeInheritedProtocol {
    // 这里是类专属协议的定义部分
}
```
- 组合协议，Named & Aged ，例如参数是遵从两协议的类型。类类型也可以和协议组合
- is 和 as? as! 检查是否遵从某协议，以及转换到协议类型
- 协议中 optional 关键字定义可选的要求，需要带上 @objc 属性，只能用于 Objective-C 类，其他类和结构体枚举都不能遵循这种协议。可选要求，类型是可选的，即函数是可选的

```swift
@objc protocol CounterDataSource {
    @objc optional func increment(forCount count: Int) -> Int
    @objc optional var fixedIncrement: Int { get }
}

extension Collection where Element: Equatable {
    func allEqual() -> Bool {
        for element in self {
            if element != self.first {
                return false
            }
        }
        return true
    }
}
```
- 可以为协议使用扩展，增加协议中的方法，但扩展不能声明协议遵从别的协议，那只能定义在协议声明中。协议的扩展可以提供默认实现，也可以添加限制条件 用 where 关键字

### 1.23 泛型

- Swift 标准库有 swap(_ :_:) 函数
- 用大驼峰命名法，命名类型参数，如 T、U、V
- 用泛型实现栈，对泛型类型扩展时，可以直接使用类型参数

```swift
struct Stack<Element> {
    var items: [Element] = []
    mutating func push(_ item: Element) {
        items.append(item)
    }
    mutating func pop() -> Element {
        return items.removeLast()
    }
}
var stackOfStrings = Stack<String>()

func someFunction<T: SomeClass, U: SomeProtocol>(someT: T, someU: U) {
    // 这里是泛型函数的函数体部分
}
```
- 类型约束，类型参数必须继承自指定类、遵循特定协议或协议组合
- Swift 的基本类型（例如 String、Int、Double 和 Bool）默认都是可哈希的
- 协议的关联类型，通过 associatedtype 关键字指定，为某个类型提供占位符。associatedtype Item，typealias Item = Int，也能被推断出类型，而不用指定 Item 是 Int 类型。还可以给关联类型添加约束

```swift
protocol Container {
    associatedtype Item: Equatable
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }
}
```
- 关联类型约束里，可遵从协议

```swift
protocol SuffixableContainer: Container {
    associatedtype Suffix: SuffixableContainer where Suffix.Item == Item
    func suffix(_ size: Int) -> Suffix
}
func allItemsMatch<C1: Container, C2: Container>
    (_ someContainer: C1, _ anotherContainer: C2) -> Bool
    where C1.Item == C2.Item, C1.Item: Equatable {

    }

// 只有其中元素符合Equatable协议时，才会实现扩展里的内容
extension Stack where Element: Equatable
// 扩展协议时
extension Container where Item: Equatable
// 要求Item为特定类型
extension Container where Item == Double
```
- 泛型 where 语句，扩展也可以添加 where 语句，也就是条件扩展
- 也可以对扩展中的方法添加 where 语句，也就是条件方法。只有满足此约束，该方法才会被添加

```swift
func average() -> Double where Item == Int {}
func endsWith(_ item: Item) -> Bool where Item: Equatable {}

protocol Container {
  associatedtype Iterator: IteratorProtocol where Iterator.Element == Item
}
protocol ComparableContainer: Container where Item: Comparable { }
subscript<Indices: Sequence>(indices: Indices) -> [Item]
        where Indices.Iterator.Element == Int {}
```
- 协议中关联类型，添加where语句。协议继承声明时也可以添加
- 下标也可以添加where约束

### 1.24 不透明类型

- 字符串有下面方法，String(repeating: "*", count: length)
- 数组组合成字符串 result.joined(separator: "\n")
- 字符串分割 .split(separator: "\n")
- 数组反转 lines.reversed()
- 不透明类型。有关联类型的协议不能作为返回类型

```swift
func flip<T: Shape>(_ shape: T) -> some Shape {}

// 错误：有关联类型的协议不能作为返回类型。
func makeProtocolContainer<T>(item: T) -> Container {
    return [item]
}

// 错误：没有足够多的信息来推断 C 的类型。
func makeProtocolContainer<T, C: Container>(item: T) -> C {
    return [item]
}

func makeOpaqueContainer<T>(item: T) -> some Container {
    return [item]
}
```

### 1.25 自动引用计数

- 仅用于类的实例，循环引用问题
- 弱引用和无主引用，弱引用也有自动置nil，应定义为可选类型变量，weak var tenant: Person?
- unowned 无主引用，在其他实例有更长的生命周期时使用，非可选值，不会自动置nil
- UInt64 能容纳16位，unowned(unsafe) 来声明不安全无主引用
- 无主可选引用，unowned var nextCourse: Course?
- 无主引用搭配隐式解包可选值属性使用，避免循环引用的同时，capitalCity 能像非可选值一样存取和使用

```swift
class Country {
    let name: String
    var capitalCity: City!
    init(name: String, capitalName: String) {
        self.name = name
        self.capitalCity = City(name: capitalName, country: self)
    }
}
class City {
    let name: String
    unowned let country: Country
    init(name: String, country: Country) {
        self.name = name
        self.country = country
    }
}
```
- 闭包引用 self 导致的循环引用，通过捕获列表解决。闭包和捕获的实例总是互相引用并且总是同时销毁时，将闭包内的捕获定义为 无主引用。在被捕获的引用可能会变为 nil 时，将闭包内的捕获定义为 弱引用。

```swift
lazy var someClosure = {
    [unowned self, weak delegate = self.delegate]
    (index: Int, stringToProcess: String) -> String in
    // 这里是闭包的函数体
}
lazy var someClosure = {
    [unowned self, weak delegate = self.delegate] in
    // 这里是闭包的函数体
}
```

### 1.26 内存安全

- 访问冲突指内存访问期间，还有别的访问进来。in-out 参数冲突，函数内外同时访问
- 同一个函数的多个 in-out 参数里传入同一个变量也会产生冲突

```swift
var stepSize = 1
func increment(_ number: inout Int) {
    number += stepSize
}
increment(&stepSize)
// 错误：stepSize 访问冲突

// 显式拷贝
var copyOfStepSize = stepSize
increment(&copyOfStepSize)
// 更新原来的值
stepSize = copyOfStepSize
// stepSize 现在的值是 2

func balance(_ x: inout Int, _ y: inout Int) {
    let sum = x + y
    x = sum / 2
    y = sum - x
}
var playerOneScore = 42
var playerTwoScore = 30
balance(&playerOneScore, &playerTwoScore)  // 正常
balance(&playerOneScore, &playerOneScore)
// 错误：playerOneScore 访问冲突
```
- 结构体 mutating 方法的访问冲突。对同一块内存地址同时进行两个写访问也会造成冲突。
- 元组元素的写访问冲突，因为修改其一个元素会访问值类型的整体。对全局变量的结构体的属性访问也是一样，局部变量可以。

```swift
var playerInformation = (health: 10, energy: 20)
balance(&playerInformation.health, &playerInformation.energy)
// 错误：playerInformation 的属性访问冲突
```

### 1.27 访问控制

- 5种访问级别，open 和 public，指定框架的外部接口。internal 框架使用，fileprivate源文件内使用，private当前作用域使用，或同一文件内通过 extension 访问。
- open作用于类，能在模块外被继承和重写
- 默认internal级别
- 测试 target 访问，需要用 @testable 特性
- 可以修饰类，变量、常量、函数
- public 类型的成员默认是 internal
- 函数类型根据其返回值、参数的最高级别来确定
- 枚举类型和其成员具有同样的级别
- setter和getter，set可以级别更低，定义为只在结构体内部访问。外部表现为只读

```swift
struct TrackedString {
    private(set) var numberOfEdits = 0
    var value: String = "" {
        didSet {
            numberOfEdits += 1
        }
    }
}
```
- 类型为public 默认构造器是 internal
- 协议及其方法或属性，级别必须相同，包括 public 也是这样

### 1.28 高级运算符

- 默认不会溢出，想允许溢出可以使用 &+ 溢出加法运算符
- 支持C中全部位运算符。 ~ 按位取反。 & 按位与，全为1才是1。 | 按位或，任一为1就是1。^ 按位异或，相同为0不同为1。<< >> 左移和右移

```swift
let pink: UInt32 = 0xCC6699
let redComponent = (pink & 0xFF0000) >> 16  // redComponent 是 0xCC，即 204
let greenComponent = (pink & 0x00FF00) >> 8 // greenComponent 是 0x66， 即 102
let blueComponent = pink & 0x0000FF         // blueComponent 是 0x99，即 153
```
- 有符号整数，第一位1表示负数，0表示正数，其余位表示数值。负数数值存储2的数值位位数次方减去实际值的绝对值，例如8位整数的-4，是2的7次方128-4=124，数值位表示为124。这就叫补码，相加就执行标准相加，多出的位丢弃。右移空出的位补充符号位
- 溢出运算，&+ &- &*
- 上溢会变成0，下溢会变成最大数
- 有符号整数下溢出，会变成正的最大值
- 乘法、求余优先于加法
- 运算符重载，重载+，在结构体实例上实现加法
- 前缀、后缀运算符，func之前加上 prefix、postfix 修饰符
- 复合赋值运算符，例如 +=，左参数是 inout
- = 和3元运算符不可重载
- 等价运算符，== 和 !=，需要遵循Equatable协议，实现相等，系统会帮你实现不等
- 自定义运算符，operator关键字，指定 prefix、infix 、postfix 修饰符。例如 prefix operator +++
- 自定义中缀运算符，默认优先级组 AdditionPrecedence
- 同时使用后缀和前缀运算符，后缀运算符先进行运算
- 字符串方法 .uppercased()
- 结果构造器，@resultBuilder特性，支持if-else转换、for循环转换，将声明描述转换为方法调用
