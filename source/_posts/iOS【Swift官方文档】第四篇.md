---
title: iOS【Swift官方文档】第四篇
date: 2023-09-15 21:00:08
tags:
- 图书
- 专业技术
- iOS
---

### 2.2 词法结构
- 二进制字面量加 0b，八进制字面量加 0o，十六进制字面量加 0x
- 1.25e2 表示 1.25 x 10²，1.25e－2 表示 1.25 x 10¯²。0xFp2 表示 15 x 2²，0xFp-2 表示 15 x 2¯²
- 浮点数默认 Double 即64位浮点数，也有Float 表示32位浮点数
- 正则表达式字面量，/ 正则表达式 /，/\(/ 匹配左括号，/\d/ 匹配单个数字

### 2.3 类型
- 可变参数，Int... 等同于 [Int]
- Void 是空元组类型 () 的别名
- 多个 -> 从右向左结合
- 元类型，SomeClass.Type、SomeProtocal.Protocol
- 对类型的实例使用 type(of:) 表达式来获取该实例动态的、在运行阶段的类型。
type(of: someInstance).printClassName()
- Self 引用当前类型

### 2.4 表达式

- #function 作为方法的默认参数值，将返回调用函数的名字
- Key-path 表达式，所有类型都可以通过传递 key-path 参数到下标方法 subscript(keyPath:) 来访问它的值，例如

```Swift
struct SomeStructure {
    var someValue: Int
}
let s = SomeStructure(someValue: 12)
let pathToProperty = \SomeStructure.someValue
let value = s[keyPath: pathToProperty]
// 值为 12
```
- 数组有filter方法，例如

```Swift
toDoList.filter { $0.completed }.map { $0.description }
```
- 有 ++ 自增符

### 2.5 语句

- guard语句的退出，return、break、continue、throw
- @unknown default 匹配未来加入的枚举
- 标签可以标注，循环和switch语句，break 和 continue 后都可以跟标签
- 多个defer语句，最后面的先执行
- 条件编译 #if 开始，#endif 结束。#if compiler(>=5)， #if swift(>=4.2)， #if compiler(>=5) && swift(<5)
- 编译时诊断语句，可用性条件语句

```Swift
#error("error message")
#warning("warning message")
if #available(platform name version, ..., *) {}
```

### 2.6 声明

- rethrows 参数类型为抛出函数，仅能传递，自身不能抛出
- 枚举支持递归，用 indirect 修饰符标记

```Swift
import 模块
func someFunction(callback: () throws -> Void) rethrows {
    try callback()
}
enum Tree<T> {
	case empty
	indirect case node(value: T, left: Tree, right:Tree)
}
```
- dynamic 修饰符修饰任何兼容 Objective-C 的类的成员
- optional 只能将 optional 修饰符用于被 objc 特性标记的协议

### 2.7 特性

- available、unavailable

```Swift
// 引入
introduced: 版本号
// 弃用
deprecated: 版本号
// 废弃
obsoleted: 版本号
// 重命名
renamed: 新名字
class ExampleClass: NSObject {
    @objc var enabled: Bool {
        @objc(isEnabled) get {
            // 返回适当的值
        }
    }
}
```
- discardableResult 返回值没使用不会警告
- dynamicCallable 实例作为可调用的函数
- dynamicMemberLookup 运行时查找成员，配合 subscript 下标使用
- frozen 限制对类型修改
- objc 告诉编译器其可以在OC代码中使用
- propertyWrapper 属性包装器
- resultBuilder 结果构造器
