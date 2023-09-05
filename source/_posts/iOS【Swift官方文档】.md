---
title: iOS【Swift官方文档】
date: 2023-09-03 11:53:49
tags:
- 图书
- 专业技术
- iOS
---

### 0 swift 初见

- print() 打印
- 数据类型：Double、可选值(String?)、元组用()
- 转换类型：\\(apples)、String(width)
- 三引号包含多行字符串，"""
- []方括号表示数组和字典，数组append()加元素
```swift
let emptyArray: [String] = []
let emptyDictionary: [String: Float] = [:]
```
- 条件语句、循环体的小括号可省略，语句体的大括号必须
- if、switch、for-in、while、repeat-while，if 条件必须是布尔表达式，if let处理值缺失情况，?? 也可以处理，nickName ?? fullName
- if let nickname {} 解包代码可以简短，与被解包值用相同名称
- switch case 可匹配字符串，匹配到会退出，不用写break。多情况匹配 case "cucumber", "watercress":，条件匹配
case let x where x.hasSuffix("pepper"):
- for-in 遍历字典，可省略key用_替代。
```swift
for (_, numbers) in interestingNumbers {}
```
- 循环中用 ..< 表示下标范围，不包含上界，...包含
```swift
for i in 0..<4 {}
```
- 函数参数标签，或用_表示不使用标签
```swift
func greet(_ person: String, on day: String) -> String {
    return ""
}
```
- 元组可以用名称和下标取元素
- 函数嵌套，做返回值、参数
- 闭包，用in分开声明和函数体，简写可忽略参数、返回值。作为唯一参数可忽略小括号，用参数位置引用参数
```swift
numbers.map({ number in 3 * number })
numbers.sorted { $0 > $1 }
```
- 类，构造器 init，构造器中用self区别实例变量和参数，每个属性都需要赋值。deinit析构函数，override覆写父类方法。getter 和 setter 计算属性，setter 新值名字newValue，willSet、didSet 用来在值改变时做一些处理，不含构造器中值改变的情况。
- 可选值解包后操作，optionalSquare?.sideLength，为nil整个表达式都返回nil否则解包
- 枚举可包含方法，默认从0开始为原始值赋值，可显示赋值改变。rawValue访问原始值，原始值可以是字符串、浮点数。可以用原始值构建枚举实例，该值是可选型。不是一定要提供原始值。
```swift
if let convertedRank = Rank(rawValue: 3) {}
```
- 枚举成员可以关联值，
```swift
enum ServerResponse {
    case result(String, String)
    case failure(String)
}
```
- 结构体和类相似，结构体穿值，类传引用
- 并发性：async、await，async let 异步并行运行，Task {}同步中调异步函数，不等待返回
- 类、枚举、结构体都可遵从协议，结构体中 mutating 关键字表明方法会修改结构体，而类中不需标明，类可以修改其属性
- 用扩展让某类型遵从某协议。用协议名做类型时，不可调协议外的方法或属性
- throw 抛出错误，throws表示可抛出错误的函数，do-catch try error，可以多catch，try? 可以返回nil抛弃错误或返回可选值
- defer 代码块 表示函数返回前最后执行的代码，不管是否抛出错误
- 泛型函数或类型，方法、类、结构体、枚举都可用泛型。用where指定一系列需求。
```swift
func anyCommonElements<T: Sequence, U: Sequence>(_ lhs: T, _ rhs: U) -> Bool
    where T.Element: Equatable, T.Element == U.Element
{
    for lhsItem in lhs {
        for rhsItem in rhs {
            if lhsItem == rhsItem {
                return true
            }
        }
    }
    return false
}
anyCommonElements([1, 2, 3], [3])
```

### 1.1 基础部分

- 类型：Int Double Float Bool String Array Set Dictionary 元组 可选类型
```swift
var red, green, blue: Double
```
- 常量变量名，不能以数字开头
- print(someValue, terminator:"") 输出不换行
- 多行注释可嵌套
- 同行代码，多条语句可用分号;
- 整数 UInt8 Int32，UInt8.min、UInt8.max。Int 根据平台决定是长度等于 Int32 还是 Int64，UInt也一样
- Double 64位浮点数，15 位小数。Float 32位浮点数，只有 6 位小数
- 浮点数字面量赋值会被推断为Double
- 进制，0b二进制、0o八进制、0x十六进制
- 十进制指数，1.25e2 等于 1.25 × 10^2，1.25e-2 表示 1.25 × 10^-2
- 十六进制指数，0xFp2 表示 15 × 2^2，0xFp-2 表示 15 × 2^-2
```swift
// 额外格式增加可读性，不影响字面量值
let paddedDouble = 000123.456
let oneMillion = 1_000_000
let justOverOneMillion = 1_000_000.000_000_1
```
- 类型转换：UInt16(one)、Double(three)、Int(pi)浮点数会被截断
- typealias 定义类型别名
- 元组，分解的时候可省略部分值，使用_
```swift
let (justTheStatusCode, _) = http404Error
```
- 可选类型，Int(possibleNumber) 构造器，返回的Int?，可选类型声明时没赋值，会自动置nil，确定有值用强制解析 convertedNumber!，隐式解包可使代码简短，也可以解包为变量 if var，一个if语句可包含多个可选绑定或布尔条件，用逗号隔开，其中之一为假，整个表达式为假
```swift
if let firstNumber = Int("4"), let secondNumber = Int("42"), firstNumber < secondNumber && secondNumber < 100 {
}
```
- 隐式解析可选类型
```swift
let assumedString: String! = "An implicitly unwrapped optional string."
let implicitString: String = assumedString  // 不需要感叹号
let optionalString = assumedString
// optionalString 的类型是 "String?"，assumedString 也没有被强制解析。
```
- 断言仅在调试用，先决条件在调试和生产环境用。断言和先决条件都可关闭的。fatalError(_:file:line:) 不会被关闭，总是会中断，早期开发阶段可用。
```swift
let age = -3
assert(age >= 0, "A person's age cannot be less than zero")
// 因为 age < 0，所以断言会触发
assert(age >= 0)
assertionFailure("A person's age can't be less than zero.")
precondition(index > 0, "Index must be greater than zero.")
preconditionFailure(_:file:line:)
```

### 1.2 基本运算符

- 赋值，元组 let (x, y) = (1, 2)
- 求余，对负数b求余，符号会被忽略，a % b 和 a % -b 结果一样
- 一元操作符，-负号，+正号
- 比较运算符，== != > < >= <=，=== 和 !== 比较对象引用是否相同，元组也可比较，元素类型可被比较时，如数字、字符串，布尔值不能比较。只能比较7个以内元素的元组
- 单侧区间，包含2到数据结尾或数组开头到2，包含2.
```swift
for name in names[2...] {}
for name in names[...2] {}
for name in names[..<2] {}
let range = ...5
range.contains(-1)  // true
```

### 1.3 字符串和字符

- 多行字符串，反斜杠（\）作为续行符.多行字符串中使用 """，需要至少一个转义符号\\
- 字符串中使用引号 \\" ，使用Unicode 标量 \u{24}
- 使用 #""# 包裹起来的字符串，能打印出转义字符，#"Line 1 \#nLine 2"# 可实现换行。##也可以应用在多行字符串中，或打印出非插值处理的结果
- String() 或 "" 初始化，isEmpty 判空
- 字符串是值类型，会拷贝。for-in可遍历字符串
- Character 字符类型
```swift
let catCharacters: [Character] = ["C", "a", "t", "!", "🐱"]
let catString = String(catCharacters)
```
- 支持 += 和 append 字符，多行字符串拼接的换行问题
- Unicode 标量，可扩展的字形群集可以由多个 Unicode 标量组成
```swift
let eAcute: Character = "\u{E9}"                         // é
let combinedEAcute: Character = "\u{65}\u{301}"          // e 后面加上  ́
// eAcute 是 é, combinedEAcute 是 é
let enclosedEAcute: Character = "\u{E9}\u{20DD}"
// enclosedEAcute 是 é⃝
let regionalIndicatorForUS: Character = "\u{1F1FA}\u{1F1F8}"
// regionalIndicatorForUS 是 🇺🇸
```
- count 属性，字符数量，和 NSString 的 length 不一定相同
- startIndex endIndex 属性，endIndex 是最后一个字符的后一个位置。空串startIndex 和 endIndex 是相等的
- index(before:) index(after:) index(_:offsetBy:) 方法，也可用在数组、字典、集合中
- indices 包含全部索引的范围
```swift
for index in greeting.indices {
   print("\(greeting[index]) ", terminator: "")
}
```
- insert(_:at:)、insert(contentsOf:at:) 插入一个字符或一段字符串  
- remove(at:)、removeSubrange(_:) 方法删除一个字符或删除一个范围的子串，有返回值
- 上面两类方法都可用在数组、字典、集合中
- greeting.firstIndex(of: ",") 可将Substring转为String以便长期存储
- 字符串比较，只要字符集群表达的是同一个语义，就认为相等
- hasPrefix(_:) 和 hasSuffix(_:) 方法
- utf8、utf16、unicodeScalars(UInt32，访问其value得到数字，直接访问得到字符) 属性，访问字符串的3种表示方式。

### 1.4 集合类型

- 数组，可用+相加或+=
```swift
var threeDoubles = Array(repeating: 0.0, count: 3)
var shoppingList = ["Eggs", "Milk"]
shoppingList[4...6] = ["Bananas", "Apples"]
```
- 数组方法有 count、isEmpty、append(_:)、removeLast() 有返回值
- 遍历，有for-in 或 enumerated()方法，使用元组作为返回值
```swift
for (index, value) in shoppingList.enumerated() {
    print("Item \(String(index + 1)): \(value)")
}
```
- 可哈希化，hashValue，基本数据类型都可，String、Int、Double 和 Bool 和 无关联值的枚举成员
- 集合，没有便利化的创建方法，也可以用[]表示
```swift
var letters = Set<Character>()
var favoriteGenres: Set<String> = ["Rock", "Classical", "Hip hop"]
var favoriteGenres: Set = ["Rock", "Classical", "Hip hop"]
```
- 集合方法 count、isEmpty insert(_:) remove(_:) 包含则返回，不包含返回nil、removeAll()、contains(_:)、sorted()返回排序的数组
- intersection(_:) 交集、symmetricDifference(_:) 不想交集、union(_:) 并集、subtracting(_:) 不在另一集合中的值创建集合
- == 是否相等，isSubset(of:) 是否子集，isSuperset(of:) 是否父集，isStrictSubset(of:) 真子集，isStrictSuperset(of:) 真父集，isDisjoint(with:) 没交集
- 字典键遵从Hashable协议，
```swift
var namesOfIntegers: [Int: String] = [:]
var airports: [String: String] = ["YYZ": "Toronto Pearson", "DUB": "Dublin"]
var airports = ["YYZ": "Toronto Pearson", "DUB": "Dublin"]
airports["APL"] = nil //移除键值对
[String](airports.keys) //构造新数组
```
- 字典方法 count isEmpty updateValue(_:forKey:) 会返回更新前的值，是可选值，新建值也可。下标访问也返回可选类型，removeValue(forKey:) 也返回可选值
- 字典遍历，返回元组形式。keys 或者 values 属性

### 1.5 控制流

- for-in区间遍历数组，也可以忽略变量名。stride(from:to:by:) 函数，设置间隔跳跃。stride(from:through:by:) 同样效果，包含结尾临界值。
```swift
for _ in 1...power {}
```
- switch case default，没有贯穿，所以不用break。匹配多值，用逗号隔开。区间匹配，元组匹配也可以是区间，下划线（_）来匹配所有可能的值，可以匹配多个分支的情况，只匹配第一个。也可以对值绑定，或用where做额外判断。复合匹配也可以用值绑定，但需要是相同的绑定值。
```swift
let somePoint = (1, 1)
switch somePoint {
case (_, 0):
    print("\(somePoint) is on the x-axis")
case (-2...2, -2...2):
    print("\(somePoint) is inside the box")
}
case (let x, 0):
    print("on the x-axis with an x value of \(x)")
case let (x, y) where x == y:
    print("(\(x), \(y)) is on the line x == y")
case (let distance, 0), (0, let distance):
    print("On an axis, \(distance) from the origin")
```
- switch 中用break 来忽略分支，fallthrough 关键字可以贯穿，连接到下一个case中代码
- 循环加标签 gameLoop: ，例如while循环中使用switch语句，可以指明break 或 continue 的是循环。
- guard else 语句，必须转移控制以退出代码块，例如 return、break、continue、throw 以及无返回的函数，例如 fatalError()
- 检测API可用性。@avaliable(macOS 10.12, *) 指明需要更高的版本。
```swift
if #available(iOS 10, macOS 10.12, *) {
    // 在 iOS 使用 iOS 10 的 API, 在 macOS 使用 macOS 10.12 的 API
} else {
    // 使用先前版本的 iOS 和 macOS 的 API
}
guard #avaliable(macOS 10.12, *) else{}
if #unavailable(iOS 10) {
	//回滚代码
}
```

### 1.6 函数

- 无返回值的函数返回 Void ，一个空元组()
- 可选元组类型，(Int, Int)? 元组可能为nil
- 一行 return 语句可以忽略return，fatalError("Oh no!") 可以做隐式返回值
- 参数名需要不同，参数标签可相同，但不同更好。有标签，调用时必须使用标签
- 默认参数值
```swfit
func someFunction(parameterWithoutDefault: Int, parameterWithDefault: Int = 12) {
    // 如果你在调用时候不传第二个参数，parameterWithDefault 会值为 12 传入到函数体中。
}
```
- 可变参数，函数内部转变成数组了
```swfit
func arithmeticMean(_ numbers: Double...) -> Double {
  for number in numbers {

    }
}
arithmeticMean(1, 2, 3, 4, 5)
```
- 输入输出参数 inout 关键字，不能有默认值。参数默认是常量，函数内不可修改。
```swfit
func swapTwoInts(_ a: inout Int, _ b: inout Int) {
    let temporaryA = a
    a = b
    b = temporaryA
}
var someInt = 3
var anotherInt = 107
swapTwoInts(&someInt, &anotherInt)
```
- 函数类型，() -> Void 没参没返回值，函数类型可以当其他类型一样用，赋值给变量，也可以应用类型推断而不必写出函数类型

### 1.7 闭包

- 参数和返回值定义，in 分割开函数体。$0 $1 表示第一个和第二个参数。排序简写过程
```swfit
reversedNames = names.sorted(by: { (s1: String, s2: String) -> Bool in return s1 > s2 } )
reversedNames = names.sorted(by: { s1, s2 in return s1 > s2 } )
reversedNames = names.sorted(by: { s1, s2 in s1 > s2 } )
reversedNames = names.sorted(by: { $0 > $1 } )
reversedNames = names.sorted(by: >)
//尾随闭包
reversedNames = names.sorted() { $0 > $1 }
//唯一参数，省略()
reversedNames = names.sorted { $0 > $1 }
```
- 尾随闭包，最后一个参数是闭包可用。多个闭包，调用时第一个闭包的参数标签可省略。
```swfit
let strings = numbers.map {
    (number) -> String in
    var output = ""
    return output
}
func loadPicture(from server: Server, completion:(Picture) -> Void,
		onFailure: () -> Void) {
}
loadPicture(from: someServer){	picture in
	someView.currentPicture = picture
} onFailure: {
	print("")
}
```
- 闭包捕获变量的引用，捕获的值不改变，可能捕获的是值拷贝。函数和闭包都是引用类型，将闭包赋值给常量，引用是常量。再赋值给其他常量，两个常用将引用同一个闭包。
- 逃逸闭包，函数返回之后才执行。逃逸闭包中需要显示使用self，非逃逸闭包则不用
```swfit
var completionHandlers: [() -> Void] = []
func someFunctionWithEscapingClosure(completionHandler: @escaping () -> Void) {
    completionHandlers.append(completionHandler)
}
class SomeClass {
    var x = 10
    func doSomething() {
        someFunctionWithEscapingClosure { self.x = 100 }
        someFunctionWithNonescapingClosure { x = 200 }
    }
}
```
- 自动闭包，能延迟求值，也可作为函数参数传递，@autoclosure可以将参数自动转化为闭包，过度使用不太好代码会难以理解。自动闭包也可以逃逸
```swift
let customerProvider = { customersInLine.remove(at: 0) }
print("Now serving \(customerProvider())!")
// 打印出“Now serving Chris!”
print(customersInLine.count)
// 打印出“4”
// customersInLine is ["Ewa", "Barry", "Daniella"]
func serve(customer customerProvider: @autoclosure () -> String) {
    print("Now serving \(customerProvider())!")
}
serve(customer: customersInLine.remove(at: 0))
// 打印“Now serving Ewa!”
// customersInLine i= ["Barry", "Daniella"]
var customerProviders: [() -> String] = []
func collectCustomerProviders(_ customerProvider: @autoclosure @escaping () -> String) {
    customerProviders.append(customerProvider)
}
collectCustomerProviders(customersInLine.remove(at: 0))
collectCustomerProviders(customersInLine.remove(at: 0))
for customerProvider in customerProviders {
    print("Now serving \(customerProvider())!")
}
// 打印“Now serving Barry!”
// 打印“Now serving Daniella!”
```

### 1.8 枚举

- 枚举原始值可以是字符串、字符、整型、浮点数
- 枚举有关联值、计算属性、实例方法、构造函数，支持扩展、协议
- 枚举成员不定义默认不会被赋值数字
- 枚举类型名大写字母开头，枚举类型被再次赋值时可用点语法简写
```swift
directionToHead = .east
```
- 枚举成员的遍历，遵循CaseIterable，调用 allCases 得到所有成员的集合
```swift
enum Beverage: CaseIterable {
    case coffee, tea, juice
}
let numberOfChoices = Beverage.allCases.count
```
- 枚举关联值，关联值可以在switch的case分支中提取出来
```swift
enum Barcode {
	case upc(Int, Int, Int, Int)
	case qrCode(String)
}
var productBarcode = Barcode.upc(8, 85909, 51226, 3)
productBarcode = .qrCode("ABCDEFGHIJKLMNOP")
switch productBarcode {
case .upc(let numberSystem, let manufacturer, let product, let check):
case .qrCode(let productCode):
// 可以简写一个let或var
case let .upc(numberSystem, manufacturer, product, check):
case let .qrCode(productCode):
}
```
- 原始值，类型必须相同，值必须唯一。原始值不能变，关联值可变。隐式赋值，值为整型时，默认从0开始，为第一个成员赋值，后面的依次+1.字符串隐式赋值为成员名称
```swift
enum ASCIIControlCharacter: Character {
    case tab = "\t"
    case lineFeed = "\n"
}
//用rawValue访问原始值
Planet.earth.rawValue
```
- 原始值构造器，定义用了原始值就有构造器，构造器可能失败返回nil
```swift
let possiblePlanet = Planet(rawValue: 7)
// possiblePlanet 类型为 Planet? 值为 Planet.uranus
```
- 递归枚举 indirect 关键字，在成员前加或枚举类型前加
```swift
indirect enum ArithmeticExpression {
    case number(Int)
    case addition(ArithmeticExpression, ArithmeticExpression)
    case multiplication(ArithmeticExpression, ArithmeticExpression)
}
let five = ArithmeticExpression.number(5)
let four = ArithmeticExpression.number(4)
let sum = ArithmeticExpression.addition(five, four)
let product = ArithmeticExpression.multiplication(sum, ArithmeticExpression.number(2))
func evaluate(_ expression: ArithmeticExpression) -> Int {
    switch expression {
    case let .number(value):
        return value
    case let .addition(left, right):
        return evaluate(left) + evaluate(right)
    case let .multiplication(left, right):
        return evaluate(left) * evaluate(right)
    }
}
print(evaluate(product))
// 打印“18”
```

### 1.9 类和结构体

- 类与结构体都支持 属性、方法、下标、构造器、扩展、协议。类另外支持继承、类型转换、析构、引用计数
- 类型名大写开头，属性和方法小写开头。结构体有成员逐一构造器，类没有
```swift
let vga = Resolution(width: 640, height: 480)
```
- 结构体和枚举是值类型，值会拷贝，基本类型，整数、浮点数、布尔值、字符串数组字典都是值类型，底层用结构体实现
- 类是引用类型，类实例用let常量引用，仍然可以改类实例的属性。引用没变，实例变了
- 恒等运算符 === 和 !== 判断是不是对同一个实例的引用
