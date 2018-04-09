---
title: 道长Swift30Projects
date: 2017-09-04 22:49:35
tags:
- 专业技术
---

这是阅读故胤道长那30个项目时的记录要点，需要配合着[代码](https://github.com/soapyigu/Swift30Projects)看。

- cocoapods 引入模块之后，需要先编译再 import
- 父 VC 添加子 VC ，子 VC 需要调用 didMove 方法。
- 视频截取。
- unwind segue 作用，怎么操作。
- 弹性动画，第一个参数是阻尼，第二个参数是初始动力。阻尼越大，减速越快。
- 转场动画使用时机和方法。第17个例子有参考意义。
- CoreImage 做图片滤镜，第19个例子。
- 带图Cell加载策略，字典存储索引和操作，滚动、停止，滚动结束加载取消不可见的，开始新出现的可见的。
- iOS 10 UICollectionView 新特性，主要的一点是，9是一下加载一整行，10是一个个来，比9滑动更流畅了。
- UICollectionView 9新增的 InteractiveMovement 手势移动重排。
- 图片按宽缩放，当高大于最大高，应该按高反算宽。
- 神器函数AVMakeRect，图片按比例缩放尺寸。需导入AV框架。
- @IBDesignable @IBInspectable 在IB上给 view 子类添加自定义属性。
- 打开地图导航到某地。第22。
- 初步了解了 SwiftyJSON 的机制，利用枚举。第22。
- iOS 9.0 通讯录。第23。
- 一点 CoreData 第24。
- TodayExtension widget的简单使用。第25.
- 3D Touch 启动和获取按压力值。第26.
- iOS 10 的本地通知。第27.
- @convention(block) 表示兼容 OC的闭包，^表示按位异或。
- embed segue 嵌入 segue 的使用，Alamofire 的使用，map flatMap 区别，flatMap能解包可选值去nil，还可以合并子数组。
