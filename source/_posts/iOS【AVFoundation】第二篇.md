---
title: iOS【AVFoundation】第二篇
date: 2023-08-04 21:18:00
tags:
- 图书
- 专业技术
- iOS
---

### 7 高级捕捉

- AVCaptureDevice 属性 videoZoomFactor，控制捕捉设备的缩放等级。最小1.0 最大由activeFormat属性确定，该属性是 AVCaptureDeviceFormat 类的实例，该类定义了属性 videoMaxZoomFactor。从哪个点放大，由 AVCaptureDeviceFormat 类的 videoZoomFactorUpscaleThreshold 属性决定。
- 修改缩放因子也需要锁定，rampToVideoZoomFactor:withRate:可实现随时间平滑缩放，rate设置1会每秒增加缩放因子一倍，通常设置为1.0到3.0之间。cancelVideoZoomRamp 方法取消缩放进程。
- 可以用kvo监听 videoZoomFactor 和 rampingVideoZoom 属性的变化，来调整滑块显示值。
- AVCaptureMetadataOutput 可实现对10个人脸进行实时检测，人脸检测时输出 AVMetadataFaceObject类型。其中定义了人脸边界bounds(设备坐标系，右下角是(1,1))，roll angle 头想肩倾斜角度，yaw angle y轴旋转的角度。
- CALayer 的 sublayerTransform 属性，CATransform3D类型，可以对所有子层应用视角转换。应用3D动画绕y轴旋转，即透视变化，可以将图片的二维平面映射到三维空间。
- CATransform3D 的 m34 属性，应用视角转换，让子层围绕Y轴旋转。CATransform3DConcat 函数可以将两个变换关联在一起。
- 条码扫描，一维条码8种（187页），二维码4种，QRCode是其中一种。
- 设置设备的 autoFocusRangeRestriction 属性为 AVCaptureAutoFocusRangeRestrictionNear，缩小扫描区域，自动对焦近处的内容，提高识别率。
- 扫描条码输出的实例 AVMetadataMachineReadableCodeObject，stringValue 属性，代表条码实际数据值，bounds 矩形边界，corners 数组，角点坐标，可以绘制与条码角落紧密对其的路径。
- 角点坐标可转化为CGPoint，使用函数 CGPointMakeWithDictionaryRepresentation
- CAShapeLayer 是 CALayer子类，用于绘制Bezier路径。
- 高帧率捕捉视频，可以在降低或提高播放速率时对音频播放设置算法(AVPlayerItem对象的audioTimePitchAlgorithm属性)，可以以固定帧率导出视频。
- AVCaptureDeviceFormat 的 videoSupportedFrameRateRanges属性，是AVFrameRateRange对象数组，其中有支持的最小帧率、最大帧率和时长信息。
- 可以获取一个摄像头设备所支持的最大format和最大帧率范围。(工程SlowKamera，199页)
- AVFrameRateRange 的 minFrameDuration 属性，代表最大帧率倒数。如最大60fps，则最小帧时长是1/60秒。
- AVCaptureVideoDataOutput 提供对捕捉到的视频数据更底层的控制，处理视频数据可用OpenGL ES 或 Core Image 。协议 AVCaptureVideoDataOutputSampleBufferDelegate，提供2个方法，一个输出视频帧，一个处理丢弃的视频帧。输出内容为 CMSampleBuffer 类型对象，包含一个 CVPixelBuffer，带有单帧原始像素数据。
- CVPixelBuffer 保存像素数据，可以操作图片内容，如添加灰度效果。
- CMSampleBuffer 还可以用 CMFormatDescription 对象形式来访问样本的细节信息。还有 CMVideoFormatDescription 和 CMAudioFormatDescription 获取音视频的细节。可用来区分音频和视频数据。
- CMSampleBuffer 还可以获取时间信息，2个函数可分别获取时间戳和解码时间戳。
- 使用 CMAttachment 协议，可从 CMSampleBuffer 中读取Exif(可交换图片文件格式标签，记录数码照片的属性和拍摄数据)元数据。
- EAGLContext 该对象提供了用于管理状态的渲染上下文，和用 OpenGL ES 绘制所需的资源。
- 摄像头二次抽样初始格式是420v，用OpenGL ES 会选用BGRA。
- CVOpenGLESTextureCacheRef 是Core Video像素buffer和OpenGL ES 贴图之间的桥梁。使用 CVOpenGLESTextureRef 可以从视频帧图像创建 OpenGL ES 贴图。
- 通读各工程源码(CubeKamera_Final工程有应用OpenGL的示例，显示一个旋转的方块，不懂OpenGL，看不懂)

### 8 读取和写入媒体

- AVAssetReader 用于从AVAsset 中读取媒体样本，用 AVAssetReaderOutput实例通过方法访问音频样本和视频帧，其是抽象类，定义了3个具体实例来读取解码的媒体样本。
例如 AVAssetReaderTrackOutput 类，使用 AVAssetTrack 实例创建，并添加读取输出设置，该类可被添加到 AVAssetReader 对象上。
- AVAssetWriter 对媒体资源编码并写入容器文件。由 AVAssetWriterInput 对象配置。当配置处理的视频的 AVAssetWriterInput 时，会用到适配器对象 AVAssetWriterInputPixelBufferAdaptor。还可以用 AVAssetWriterInputGroup 组成互斥参数，包含在播放时使用的字幕音轨。
- AVAssetWriter 支持交叉媒体样本，AVAssetWriterInput 有 readyForMoreMediaData 属性，表示保持交错模式下输入信息是否还可以附加更多数据。
- AVAssetWriter 可用于实时和离线操作2种情况，实时写入数据优化了写入器，视频和音频以大致相同的速率捕捉。离线操作，需要观察 readyForMoreMediaData 属性的状态，提供方法，控制数据的提供。
- AVAssetTrack 表示资源其中的一个轨道，例如视频轨道、音频轨道。
- AVAssetWriter 可添加 AVAssetWriterInput 类，该类实例带有媒体类型和配置参数。
- AVAssetWriter 和 AVAssetExportSession 相比，可以进行更多的设置控制，例如关键帧间隔、视频比特率、像素宽高比、纯净光圈、H.264配置文件等。
- AVAssetWriter 和 AVAssetReader 组合使用，从非实时资源读取并写入样本时可用。(215页代码)
- 图像化显示音频波形，第一步读取或解压音频数据。第二步缩减样本，分为小的样本块，找到块的最大样本、平均值、最大最小值。第三步绘制。
- CMSampleBufferGetImageBuffer 函数可检索到包含有帧的像素信息的 CVImageBufferRef。读取音频样本使用 CMSampleBufferGetDataBuffer 函数可以拿到 CMBlockBufferRef 的不保留引用，如果需要保留引用并对数据做处理，使用 CMSampleBufferGetAudioBufferListWithRetainedBlockBuffer 函数拿到 AudioBufferList 。
- 音频轨道输出的解压设置，读取音频样本数据(218、219页)
- 缩减音频样本。（221页）
- abs 绝对值函数。
- CFSwapInt16LittleToHost 小端转为主机内置字节序。
- 用 CoreGraphics 绘制波形。（工程 THWaveformView_Final）
- 除了用OpenGL ES 处理视频帧的显示，也可以用 Core Image 处理实时视频帧。
- AVCaptureVideoDataOutput 类的属性 alwaysDiscardsLateVideoFrames，可捕捉全部帧，会增加内存消耗。
- CIImage 的 imageWithCVPixelBuffer 方法可以从 CVPixelBufferRef 对象创建一个CIImage。
- AVAssetWriterInput 类的属性 expectsMediaDataInRealTime 设置为YES 意思为针对实时性进行优化。
- AVAssetWriterInputPixelBufferAdaptor 对象提供一个优化的 CVPixelBufferPool 可创建 CVPixelBuffer 对象来渲染视频帧。
- 使用 CMSampleBufferGetFormatDescription 函数从 CMSampleBufferRef 对象拿到 CMFormatDescriptionRef，再使用 CMFormatDescriptionGetMediaType 从 CMFormatDescriptionRef 拿到媒体类型信息 CMMediaType，即可判断类型是video 还是 audio 。
- 录制视频加筛选器(工程 KameraWriter_Final)
- 通读工程代码，其中有 NSRegularExpression 正则表达式和 NSTextCheckingResult类的用法。(工程 KameraWriter_Final)。切换几种调色器录制并保存。

### 9 媒体的组合和编辑

- 组合资源，组合 AVComposition(AVAsset子类)、组合轨道 AVCompositionTrack、AVCompositionTrackSegment 实际媒体区域。AVMutableComposition 、AVMutableCompositionTrack 可操作轨道和轨道分段。
- NSTimeInterval 是双精度类型，无法应用于高级时基媒体的开发，会导致丢帧或音频丢失。使用CMTime数据类型作为时间格式，可表示特定的时间点或持续时间。
- CMTime 结构中 CMTimeFlags 表示指定状态，如判断数据是否有效、是否出现舍入值等。创建 CMTime 实例，使用 CMTimeMake 函数。视频常见时间刻度是600，音频时间刻度是采样率，如44100（44.1kHz）。
- CMTimeAdd、CMTimeSubtract 时间相加或相减，CMTimeShow 打印到控制台。
- CMTimeRange 时间范围，由2个 CMTime 组成，一个表示起点，一个表示持续时间。构建函数 CMTimeRangeMake 通过起点和持续时间构建，或 CMTimeRangeFromTimeToTime 通过起点和终点时间构建。
- CMTimeRangeGetIntersection 函数获取两个时间范围的交集，CMTimeRangeGetUnion 获取并集。
- 创建资源用于组合时，使用 AVURLAsset，创建 AVURLAsset 实例可传入options参数，选项字典。AVURLAssetPreferpreciseDurationAndTimingKey 设置为@YES 代表可以计算出准确的时长和时间信息。
- 组合资源，分别组合视频轨道和音轨。AVMutableComposition 可添加轨道对象，指明媒体类型和一个标识符，kCMPersistentTrackID_Invalid 标识符常量表示轨道ID以1···n排列。
- 组合资源步骤，从资源提取视频轨道或音轨，从中提取给定时间范围和开始时间的片段，插入组合轨道中。其中给定的开始时间是指组合轨道的光标所在的时间。
- AVAssetExportSession 导出预设值还有 AVAssetExportPresetHighestQuality。AVAssetExportSession 对象还有 progress 属性，表示导出进度。
- 工程代码（TimeChallenge），CMTime 和 NSValue 可以互转。CMTimeRange 也可以和 NSValue 互转。CMTime 和 CMTimeRange 还可以和字典互转。
- 工程代码，可以静默 AVPlayerItem 的音轨，tracks属性拿到 AVPlayerItemTrack 设置该对象 enabled 为 NO。
- c语言的 static inline 内联函数。fminf函数返回2个浮点数中的较小者。

### 10 混合音频

- 画外音持续期间将音乐调小，开始时声音渐渐增大结束时渐渐减小。AVAudioMix 类可以处理这两种情景。
- AVAudoioMix 输入参数集，AVAudioMixInputParameters 类型，其关联单独音轨。创建自定义混合音频时使用 AVMutableAudioMix 和 AVMutableAudioMixInputParameters。AVAudoioMix 对象可被用于 AVPlayerItem 或 AVAssetExportSession 用于播放和导出。
- 可以用 AVMutableAudioMixInputParameters 修改音轨的音量，浮点数0到1，可以在某一时间点或一段时间内调节音量。

### 11 创建视频过渡效果

- 动态的视频过渡效果。AVVideoComposition，其对两个或多个视频组合在一起的方法给出一个总体描述，除了描述视频层组合信息之外，还包括配置视频组合的渲染尺寸，、缩放和帧时长等属性。AVVideoComposition 不是 AVComposition 子类，甚至没关联。
- AVVideoComposition 由 AVVideoCompositionInstruction 定义的指令组成的，提供时间范围信息。AVVideoCompositionLayerInstruction 应用模糊、变形、裁剪效果。可以用来在时间点或时间范围内对值进行修改，可以创建动态过渡，如溶解、渐淡效果。
- 两个视频轨道错开布局，过渡时有时间重叠部分，计算不重叠部分和重叠部分的时间范围，有工具类可以排查计算时间范围出现的问题，Apple Developer Center 中 AVCompositionDebugViewer 项目，可视化呈现。
- 不重叠部分添加一个 AVVideoCompositionLayerInstruction 指令，重叠部分也就是需要过渡的部分添加多个指令，设置为 AVVideoCompositionInstruction 对象的 layerInstructions 属性的值。
- AVMutableVideoComposition 属性有 instructions 数组表示指令集合，renderSize 表示渲染尺寸，对应视频原始尺寸。frameDuration 代表帧时长，renderScale 缩放，通常是1.0。
- 便捷方式创建 AVVideoComposition ，有初始化方法，通过参数 AVComposition 创建。这种方式创建不能解决全部问题，如需要对时间范围和指令进行更多的控制时。
- AVMutableVideoComposition 对象的 videoCompositionWithPropertiesOfAsset 方法可以从AVMutableComposition 对象中创建出来 AVVideoComposition 对象，会自动创建所需的组合对象和层指令。
- 3种过渡效果，溶解、推入、擦除。CGAffineTransform 可以修改层的转化、旋转、缩放。

### 12 动画图层内容

- Core Animation 创建高性能、基于GPU的动画效果。图层CALayer，一般是图片、Bezier路径，可设置背景颜色、透明度、角半径，几何属性如bounds、position.图层子类有CATextLayer、CAShapeLayer（渲染Bezier路径）。
- 动画 CAAnimation ，常用子类 CABasicAnimation、CAKeyframeAnimation 。CABasicAnimation 用于实现简单动画，CAKeyframeAnimation 用于实现更高级的功能，例如图层沿着Bezier路径动态显示，可以指定时间和节奏。
- 书上有一个用 CABasicAnimation 旋转一个图片的例子。(292页)
- AVSynchronizedLayer 用于与给定的 AVPlayerItem 对象同步时间。可以添加动画标题、水印、下沿字幕到播放视频中。
- AVVideoCompositionCoreAnimationTool 类，将动画图层整合到导出视频中。
- 注意点，removedOnCompletion 应设置为 NO，动画完成之后不删除。beginTime 设置为 AVCoreAnimationBeginTimeAtZero 常量，可在影片开头加入动画。
- CALayer 对象的 allowsEdgeAntialiasing 属性设置为YES，边缘抗锯齿效果。
- CALayer 图层对象的 addAnimation:forKey: 方法，其中的key参数，是自己定义的字符串，用于后面识别或取回该动画。如果不需要，则赋值为nil即可。
- CMTimeGetSeconds 函数用于将CMTime值转换为浮点值float.
- 创建视频动画时，不用 CAAnimationGroup 组合动画。
- CABasicAnimation 对象的 timingFunction 属性，设置一个时间函数，如淡入淡出曲线函数。属性 autoreverses 可以反向运行动画。
- CALayer 类属性 geometryFlipped，几何翻转。位置不会颠倒。
- 如何将动画整合和视频整合导出（306页）
