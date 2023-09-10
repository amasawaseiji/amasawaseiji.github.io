---
title: iOS【Swift官方文档】第二篇
date: 2023-09-05 22:21:56
tags:
- 图书
- 专业技术
- iOS
---

### 1.10 属性

- 计算属性用于类、结构体、枚举，存储属性用于类和结构体
- 结构体实例被赋值给常量，其变量属性也就无法修改了
- 延时加载存储属性，lazy 关键字，必须是变量，被多个线程访问，可能会初始化多次
- 计算属性，提供get和可选的set，set没有定义新值参数的话默认使用newValue，get可以省略return 返回单一表达式。计算属性只能用 var 声明
- 只读计算属性，可以省去get关键字和一对花括号
- 属性观察器，存储属性、继承存储属性、继承计算属性。willSet使用newValue，didSet可以用oldValue，in-out 属性也会调。

```swift
class StepCounter {
    var totalSteps: Int = 0 {
        willSet(newTotalSteps) {}
        didSet {
            if totalSteps > oldValue  {}
        }
    }
}
```
- 属性包装器，通过wrappedValue访问number，通过@TwelveOrLess把包装器应用到属性。属性包装器也可以通过构造器来指定初始状态。

```swift
@propertyWrapper
struct TwelveOrLess {
    private var number = 0
    var wrappedValue: Int {
        get { return number }
        set { number = min(newValue, 12) }
    }
}
struct SmallRectangle {
    @TwelveOrLess var height: Int
    @TwelveOrLess var width: Int
}
var rectangle = SmallRectangle()
rectangle.height = 24
print(rectangle.height)
// 打印 "12"
// 另一个版本的写法
struct SmallRectangle {
    private var _height = TwelveOrLess()
    private var _width = TwelveOrLess()
    var height: Int {
        get { return _height.wrappedValue }
        set { _height.wrappedValue = newValue }
    }
    var width: Int {
        get { return _width.wrappedValue }
        set { _width.wrappedValue = newValue }
    }
}
```
```swift
@propertyWrapper
struct SmallNumber {
    private var maximum: Int
    private var number: Int
    var wrappedValue: Int {
        get { return number }
        set { number = min(newValue, maximum) }
    }
    init() {
        maximum = 12
        number = 0
    }
    init(wrappedValue: Int) {
        maximum = 12
        number = min(wrappedValue, maximum)
    }
    init(wrappedValue: Int, maximum: Int) {
        self.maximum = maximum
        number = min(wrappedValue, maximum)
    }
}
struct UnitRectangle {
    @SmallNumber var height: Int = 1
    @SmallNumber var width: Int = 1
}
var unitRectangle = UnitRectangle()
print(unitRectangle.height, unitRectangle.width)
// 打印 "1 1"

struct NarrowRectangle {
    @SmallNumber(wrappedValue: 2, maximum: 5) var height: Int
    @SmallNumber(wrappedValue: 3, maximum: 4) var width: Int
}
var narrowRectangle = NarrowRectangle()
print(narrowRectangle.height, narrowRectangle.width)
// 打印 "2 3"
narrowRectangle.height = 100
narrowRectangle.width = 100
print(narrowRectangle.height, narrowRectangle.width)
// 打印 "5 4"

struct MixedRectangle {
    @SmallNumber var height: Int = 1
    //这样也可以
    @SmallNumber(maximum: 9) var width: Int = 2
}
```
- 通过.$someNumber 访问包装器的被呈现值，在包装器中 private(set) var projectedValue: Bool 这样写

```swift
enum Size {
    case small, large
}
struct SizedRectangle {
    @SmallNumber var height: Int
    @SmallNumber var width: Int

    mutating func resize(to size: Size) -> Bool {
        switch size {
        case .small:
            height = 10
            width = 20
        case .large:
            height = 100
            width = 100
        }
        return $height || $width
    }
}
```
- 函数内的局部变量也可以用属性观察器，全局变量或计算型变量不能用
- 类型属性，只有一份，是类型实例共享的数据。存储型类型属性必须指定默认值，会延迟初始化并只初始化一次。
- 使用关键字 static 定义类型属性，关键字 class 支持子类对父类的实现进行重写。通过类型本身来访问

```swift
class SomeClass {
    static var storedTypeProperty = "Some value."
    static var computedTypeProperty: Int {
        return 27
    }
    class var overrideableComputedTypeProperty: Int {
        return 107
    }
}
print(SomeClass.computedTypeProperty)
// 打印“27”
```

### 1.11 方法

- 类、结构体、枚举都可定义实例方法、类型方法
- 方法参数名称和实例属性相同时，需要指明self
- 方法前加 mutating ，代表是可以修改结构体、枚举的属性的方法。结构体若被赋值给常量，不能调用可变方法
- 结构体可变方法中可以给slef赋值，创建新的实例。枚举可以给self赋值，改变枚举成员的类型
- 类型方法中可直接访问类型属性，不需要使用类型名称调用
- @discardableResult 特性，表明允许方法调用时忽略返回值

### 1.12 下标

- 下标定义在类、结构体、枚举中，使用subscript关键字
- 有参数、返回值，可以设定读写、只读，通过getter、setter 实现。只读可简写，省略大括号和get 关键字
- 下标访问通过中括号 [6]
- swift 字典下标取的值是可选的，给值赋值nil可删除该值
- 下标可以定义多参数，例如定义一个矩阵结构体，可以像下面这样给其内容赋值

```swift
matrix[0, 1] = 1.5
matrix[1, 0] = 3.2
```
- 类型下标，通过在 subscript 前加上 static 定义

### 1.13 继承

- 只有类有继承，子类继承父类的所有属性和方法
- 子类中重写，用 override 关键字，用 super 调用超类的方法等
- 重写属性，只读可重写为读写，读写不可重写为可读

```swift
class Car: Vehicle {
    var gear = 1
    override var description: String {
        return super.description + " in gear \(gear)"
    }
}
```
- 可以为重写的属性添加属性观察器，继承来的常量属性和只读计算属性不能添加属性观察器
- 标记为 final 可防止被重写，final class 表明类不可被继承

### 1.14 构造过程

- 构造器执行初始化工作，析构器执行清理工作。
- 方法标签，如果没提供，用参数名生成默认标签
- 常量属性只能在定义它的类的构造过程中修改，不能在子类中改
- 所有属性都被赋值，构造器会默认实现，不需要自己写
- 结构体有逐一成员构造器，属性有默认值，创建实例时可省略部分或全部参数
- 结构体、枚举，有自定义构造器，就无法访问默认构造器和逐一成员构造器，还希望保留的话，把自定义构造器写到扩展中
- convenience 表示是便利构造器，便利构造器必须调同级的指定构造器，子类的指定构造器必须调父类的
- 两段式构造过程，子类构造器可以带 override 表明是重写父类的构造器。便利构造器实际上不能被子类重写，子类加上 override 和不加没影响。并不需要加
- 类也有默认构造器，是指定构造器
- 先调用 super.init() 再修改继承来的属性，不修改的话可以不调用。await super.init() 调用来处理父类构造器是异步的情况
- 子类仅可修改继承来的变量属性，常量属性不能修改
- 子类没定义指定构造器会继承父类的，实现了父类的所有指定构造器（包括重写为便利构造器），将继承父类的便利构造器
- override convenience 可以将父类的指定构造器重写为便利构造器
- 可失败构造器，init? ，例如 Int(exactly:)，构建失败会返回nil

```swift
struct Animal {
    let species: String
    init?(species: String) {
        if species.isEmpty {
        	return nil
        }
        self.species = species
    }
}
```
- 有原始值的枚举类型自带可失败构造器，init?(rawValue:)
- 子类可用非可失败构造器重写父类的可失败构造器，反过来则不行
- init! 也可定义可失败构造器
- required 修饰符表明子类必须实现该构造器，子类中也需要加，这种方式重写父类的指定构造器时，不用加 override 修饰符
- 使用闭包为属性提供默认值，闭包里不能访问属性或方法以及使用self

```swift
class SomeClass {
    let someProperty: SomeType = {
        // 在这个闭包中给 someProperty 创建一个默认值
        // someValue 必须和 SomeType 类型相同
        return someValue
    }()
}
```

### 1.15 析构过程

- deinit 关键字，不能主动调用，会被自动调用，例如将类实例指向nil，会造成实例被释放，析构被调用。在析构时做一些清理

### 1.16 可选链

- 给通过可选链式调用的属性赋值，等号右侧的代码不会执行。也就是说左侧可选链式调用是nil了，右侧代码不执行。
- 访问下标写法 john.residence?[0].name
- 字典下标访问得到的是可选值，testScores["Dave"]?

### 1.17 错误处理

- 抛出错误用 throw 关键字，throws 表示函数、方法、构造器可以抛出错误
- guard 语句的 else 分支中可以用throw，提前退出方法
- 错误可以通过 try 关键字继续向上层调用的函数抛出，上层调用的函数也需要加 throws 关键字
- do-catch 语句，使用 error 常量
- 不会抛出错误的函数中，需要用 do-catch 语句处理错误
- 捕获多个错误，catch语句可以用逗号分隔

```swift
func eat(item: String) throws {
    do {
        try vendingMachine.vend(itemNamed: item)
    } catch VendingMachineError.invalidSelection, VendingMachineError.insufficientFunds, VendingMachineError.outOfStock {
        print("Invalid selection, out of stock, or not enough money.")
    }
}
let x = try? someThrowingFunction()
let photo = try! loadImage(atPath: "./Resources/John Appleseed.jpg")
```
- try? 将错误转为可选值，如果抛出错误，返回 nil
- try! 禁用错误传递，实际不会抛出错误时使用
- defer 语句将代码的执行延迟到当前的作用域退出之前，其中不能包含 return、break、抛出错误，defer 语句中的代码是后序执行的

### 1.18 并发

- 异步方法，用 async 关键字跟在参数列表后面，同时支持异步的抛出错误，async 写在 throws 关键字前边
- 调用时使用 await 关键字标记在方法之前，意思为可以挂起这段代码，执行其他的代码
- 使用 sleep(until:clock:) 方法模拟一个延时操作

```swift
func listPhotos(inGallery name: String) async -> [String] {
    let result = // 省略一些异步网络请求代码
    return result
}

func listPhotos(inGallery name: String) async throws -> [String] {
	try await Task.sleep(until: .now + .seconds(2), clock: .continuous)
	return ["IMG001", "IMG99", "IMG0404"]
}
```
- 异步序列，for-await-in 循环，每次循环开始时可能挂起当前执行
- 异步并发执行，在使用时添加 await 标记

```swift
async let firstPhoto = downloadPhoto(named: photoNames[0])
async let secondPhoto = downloadPhoto(named: photoNames[1])
async let thirdPhoto = downloadPhoto(named: photoNames[2])

let photos = await [firstPhoto, secondPhoto, thirdPhoto]
show(photos)
```
- 任务组、游离任务。任务取消，使用 Task.checkCancellation() Task.isCancelled 检查是否取消，使用 Task.cancel() 手动执行扩散取消

```swift
await withTaskGroup(of: Data.self) { taskGroup in
    let photoNames = await listPhotos(inGallery: "Summer Vacation")
    for name in photoNames {
        taskGroup.addTask { await downloadPhoto(named: name) }
    }
}

let newPhoto = // ... 图片数据 ...
let handle = Task {
	return await add(newPhoto, toGalleryNamed: "Spring Adventures")
}
let result = await handle.value
```
- actor 是引用类型，用来在并发代码间分享信息，同一时间只允许一个任务访问其可变状态。访问其属性需要用 await 关键字，不加会报错
- 可发送类型，遵从Sendable协议，意为发送时不可变的类型

### 1.19 类型转换

- is as ，也可以检验一个类型是否遵从某协议
- 数组能根据元素的相同基类，推断出是基类类型
- as? as! 向下转换为子类型

```swift
if item is Movie {}

for item in library {
  if let movie = item as? Movie {
      print("Movie: \(movie.name), dir. \(movie.director)")
  }
}
var things: [Any] = []
let optionalNumber: Int? = 3
things.append(optionalNumber)        // 警告
things.append(optionalNumber as Any) // 没有警告
```
- Any 表示任何类型，包括函数类型以及闭包表达式、可选类型。AnyObject 表示类类型的实例，可以用as is 识别为具体的类型
