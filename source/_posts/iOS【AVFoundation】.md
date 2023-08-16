---
title: iOS【AVFoundation】
date: 2023-03-09 21:13:39
tags:
- 图书
- 专业技术
- iOS
---

### 1 入门

- AV Foundation 基于64位架构处理器设计。
- UIWebView中支持 \<audio> 和 \<video> 这样的H5标签，用于播放音频和视频。
- 基于底层框架，CoreAudio、Core Video、Core Media、Core Animation。
- 核心功能，音频播放和记录、媒体文件检查、视频播放、媒体捕捉、媒体编辑、媒体处理。
- 将模拟信号转换为数字信号，就是采样。模拟信号是连续的，数字信号是离散的。
- 采样包括时间采样，用于音频捕捉，一段时间内的音高、声调变化。空间采样，用于可视化的媒体内容数字化的过程中，如对图片捕捉亮度和色度。视频则是两种采样都有。
- 声波通过振幅和频率传入人耳，频率决定音调，振幅决定音量。人耳将其转换成电信号传递给大脑。
- 麦克风将声波转换成电压信号。
- 使用音频生成器将连续的信号转换成离散形式。音频正弦波里振幅代表信号的强度，频率代表一定周期内震动完成循环的次数，单位是赫兹Hz，人类可以听到频率范围是20Hz-20kHz。
- 音频数字化的编码方法，称为线性脉冲编码调制（Linear PCM 或LPCM）。其采样一个固定的音频信号，过程的周期称为采样率。不断提高采样率可以更好地表现原始信号的信息。
- 尼奎斯特频率，采样频率可用于生成足够好的数字呈现效果。该频率为需要采样对象最高频率的两倍。
- 音频的位元深度，表示音频的精度，即振幅。过少的位结果会导致信号产生噪声和扭曲，cd音质位元深度为16，专业的位元深度可以达到24或更高。
- 不经过压缩的数字呈现效果，需要大量存储空间。
- 视频的帧即一张图片。视频帧率，即1秒内展现的帧数，单位是FPS。
- 8位的RGB色彩空间，意味着红色、绿色、蓝色各占8位。
- 视频如果未压缩，其存储需求非常大的空间。
- RGB颜色模式，每个像素都由红绿蓝三个颜色组成。YUV颜色模式使用色彩通道UV替换了像素的亮度通道Y。图片的细节保存在亮度通道中，人眼对亮度的敏感度大于颜色。所以可以减少图片的颜色信息来进行压缩，这被称为色彩的二次抽样。
- 二次抽样参数，将亮度比例表示为色度值。每个像素点需要亮度值却不一定需要色度值。常见的亮度比色度的比例有1比1，2比1，4比1。
- 取样一般是摄像机做的，专业相机是1比1，大部分图片拍摄是2比1，iPhone上则是4比1。后期校正颜色时，会有失真和噪声点问题。
- 编解码器使用算法对音频或视频数据进行压缩和编码，同时也可以对编码文件解码。
- 压缩分为无损和有损压缩，常用的zip和gzip是无损压缩。有损压缩，例如提取音频中人类比较敏感的频率段1kHz-5kHz的内容，减少和消除特定频率。有损压缩目的是减少冗余数据，使质量损耗达到最小。
- H.264视频文件格式，广泛用于消费者摄像头捕捉到的资源，也称为网络流媒体视频使用的主要格式。是MPEG-4的一部分。
- H.264压缩包括，帧内压缩、帧间压缩。帧内压缩通过消除独立视频帧里的色彩和结构中的冗余信息来压缩，在不降低图片质量的情况下尽可能缩小尺寸，同JEPG压缩的原理类似。这一过程创建的帧称为I-frames。
- 帧间压缩，消除一组图片在时间维度上的冗余，例如场景中一段时间内背景都相同，这就是冗余背景。
- I-frames 关键帧，包含完整图片所需要的所有数据。P-frames 预测帧。B-frams 双向帧，依赖于周围其他的帧。
- H.264 可以编码视图。BaseLine 低效压缩，压缩后文件仍较大，iPhone3GS可能用到。Main 较高的压缩率，High 高质量的压缩效果。
- ProRes 是中间层编解码器，独立于帧，只有I-frames可以被使用，适合用于专业编辑。具有最高的编解码质量，只在macOS上可用。
- AV Foundation 还支持如 MPEG-1、MPEG-2、MPEG-4、H.263、DV 编解码器。
- AAC，高级音频编码，是H.264标准相应的音频处理方式，目前主流编码方式。比MP3格式有显著提升，是web上最理想的音频格式。
- AV Foundation 支持对MP3解码，但不支持编码。
- .mov .m4v .mpg .m4a 是文件的容器格式。包含不同的媒体类型，如视频、音频、字幕、章节信息。每个格式都有规范，规范确定文件结构，文件结构包含如电影标题、歌曲作者信息等。
- QuickTime 容器格式，被广泛使用，拥有非常高的可靠性。
- MPEG-4(MP4) 容器格式，从QuickTime规范中派生出来，行业标准格式。扩展名为 .mp4，其变化扩展名，如m4a音频文件，m4v视频文件，都使用相同的MP4容器格式。
- AVSpeechSynthesizer和AVSpeechUtterance类可以实现，文本转语音播放功能。AVSpeechUtterance实例属性，voice播放的语音，可用 AVSpeechSynthesisVoice 初始化。rate 速率，介于0到1.0。pitchMultiplier 音调高低，介于0.5-2.0。postUtteranceDelay 下一句之前的暂停时间。（工程HelloAVF_Final）

### 2 播放和录制音频

- iOS平台，音乐和电话，扬声器和耳机，播放声音不共存。
- 音频会话在应用程序和操作系统之间扮演中间人角色。系统可以对用户使用音频的行为进行管理。
- 默认音频会话，来自于预配置，激活音频播放、开启静音播放的音频会消失、锁屏音频静音、播放音频，后台播放静音。
- 7种音频会话分类，有些可以允许混音，就是和背景声音相混合。还可以通过options和modes进一步自定义开发。（22页）
- 设置分类一般在应用程序启动处，也就是didFinishLanunchingWithOptions方法中，AVAudioSession单例可以通过 setCategory: 和 setActive: 配置分类以及激活配置。
- AVAudioPlayer 实现音频播放，可以播放文本或内存中的音频，常为播放音频的最佳选择。基于Core Audio ，可实现播放、循环、音频计量。除非需要播放网络流中的音频、访问原始音频样本，或低时延，否则其都可胜任。
- 可以基于bundle中的音频文件创建URL，从URL加载音频。prepareToPlay 方法是可选的，作用是降低调用play时的延迟。
- pause 和 stop 方法，暂停和停止播放。继续调用 play ，暂停和停止的音频都会继续播放。区别在stop会撤销 prepareToPlay 所做的设置，pause 则不会。
- volume 修改音量，播放器音量独立于系统音量，可以实现声音渐隐效果。其值在0.0-1.0之间。
- pan 立体声，其值为-1.0到1.0 默认0.0。
- rate 调整倍速，范围从0.5倍速到2.0倍速。
- numberOfLoops 实现循环，大于0即为n次循环，-1是无限循环。循环可以是AAC格式、AppleLossless格式，MP3格式也可以，但不推荐用，需要特殊工具处理。
- 音频计量，读取音量力度平均值和峰值，提供可视化反馈效果。
- 设置 enableRate 属性为YES，可以对播放率控制。
- deviceCurrentTime 捕捉当前设备时间，playAtTime: 方法可以延时播放。
- currentTime 设置为0.0f 让播放进度回到原点。
- 播放音频时静音和锁屏问题，默认的音频会话分类名称叫 Solo Ambient ，只可以播放音频。设置分类为 AVAudioSessionCategoryPlayback 可以让应用程序在后台播放音频。设置Info.plist文件 Required background modes 数组中的 item 为 App plays audio or streams audio/video using AirPlay 可以实现锁屏时的后台播放。
- 中断事件，如电话呼入、闹钟响起、弹出 FaceTime 。中断事件会导致播放中的音频慢慢消失和暂停，这时默认行为。当中断事件结束时，音频播放无法自动恢复。
- AVAudioSessionInterruptionNotification 对中断事件的处理（见工程AudioLooper_Final）
- 线路改变，插入耳机、断开USB麦克风。插入和拔出耳机时，音频会从扬声器转到耳机，再转回扬声器，按设计此时应该保持静音状态。
- 通知名称 AVAudioSessionRouteChangeNotification ，通知 userInfo 里的 AVAudioSessionRouteChangeReasonKey 表示原因，AVAudioSessionRouteChangeReasonOldDeviceUnavailable 表示耳机断开。获取前一个线路的描述信息，其中有描述不同 I/O 接口的属性。需要判断是否为耳机接口，如果是停止播放。（见工程AudioLooper_Final）
- 通读工程代码（AudioLooper_Final），其中有根据颜色获取HSB模式颜色的方法，B表示亮度，值范围0到1。给矩形内设置渐变色的方法，绘制圆角矩形、三角形等方法。在矩形内画圆，设置线条色和填充色以及阴影等。给视图做旋转动画，追踪触摸手势移动和结束。

##### AVAudioRecorder

- AVAudioRecorder 创建需要一个写入地址URL，一个设置字典。其 prepareToRecord 方法，可以将录制启动的延时降到最小。
- 设置字典，AVFormatIDKey 键定义写入内容的音频格式，其值有 kAudioFormatLinearPCM(未压缩写入，文件最大)、MPEG4AAC和AppleIMA4(显著缩小文件，还能保证质量)
- 如果写入的URL指定文件格式是.wav，则只能指定格式为LinearPCM，其余格式都会导致错误。示例这里写入的.m4a 使用 MPEG4AAC 格式。
- AVSampleRateKey 定义采样率。使用低采样率，如8kHz导致粗粒度的录制效果，文件会较小。用44.1kHz(CD质量采样率)，得到高质量文件较大的结果。标准采样率有8000、16000、22050、44100，示例使用的是 @22050.0f 。
- AVNumberOfChannelsKey 定义通道数，1即单声道录制，2即立体声录制。通常应创建单声道录制。外部硬件录制可以用立体声。
- 录制可以在未来某一时间点开始或指定录制时长，还可以暂停并在之后从暂停的地方重启录制。
- 录制并播放，应使用分类 AVAudioSessionCategoryPlayAndRecord。
- 使用麦克风，需要申请许可，试图访问麦克风时，系统会自动弹出对话框。
- .caf (Core Audio Format)格式，可以保存Core Audio 支持的任何音频格式，录音通常用它。
- AVEncoderBitDepthHintKey 定义位深度，示例是16位。
- NSInteger 数字类型用 %i。%02i 输出整数，左侧补零。long用%ld。NSUInteger用%lu。
- Audio Metering 对音频测量，读取平均分贝和峰值分贝。averagePowerForChannel 和 peakPowerForChannel 方法返回表示分贝等级的浮点数(dB)。范围从最小-160dB到0dB。通过设置meteringEnabled 属性为YES支持测量。调用 updateMeters 获取最新的值。
- CADisplayLink 需要频繁更新时可代替NSTimer，其可用与显示刷新率同步。frameInterval 帧间隔，比如设置5就是显示屏每刷新5帧调用一次。
- 计量功能会有额外计算，短时间录制没问题，长时间录制应考虑关闭该功能或采用比Quarta更高效的绘制方法，如OpenGL ES.
- 通读工程代码（VoiceMemo_Final），其中有用文件系统从一个地址向另一个地址拷贝内容的代码。UIImage 的 imageWithRenderingMode:方法使用 UIImageRenderingModeAlwaysOriginal 即可不被 Tint Color 所影响，始终使用图片原始颜色。

### 3 资源和元数据

- AVAsset 抽象类，包含媒体资源的标题、时长、元数据、创建日期。通过URL创建一个资源，URL可以是bundle里、文件系统里的、远程服务器上的音频流或视频流。资源还隐藏了位置信息。
- AVAsset 作为时基媒体的容器，用AVAssetTrack类代表统一媒体类型，常见的形态有音频视频流、文本、副标题、隐藏字幕。其tracks属性返回数组，元素是曲目。还可以用AVAsset 标识符、媒体类型、特征来找曲目。
- 使用 assetWithURL: 创建实例，实际上是创建了AVURLAsset子类实例，其可以传入选项字典。AVURLAssetPreferpreciseDurationAndTimingKey:@YES 用此选项意为可以提供更精确的时长和时间信息。
- 用ALAssetLibrary 从照片图库获取一个视频资源。（51页）
- 用MediaPlayer框架 从iPod库中查找内容，并用得到的URL创建资源。(51页)
- 用 iTunesLibrary 框架从Mac iTunes 库中查找内容，并创建资源。(52页)
- AVAsset 会延迟载入资源的属性，请求时才载入。访问属性是同步的，会阻塞线程，所以需要异步查询属性。
- statusOfValueForKey:error: 查询给定属性的状态，返回AVKeyValueStatus类型枚举值，如果不是AVKeyValueStatusLoaded 就可能导致卡顿。
- loadValuesAsynchronouslyForKeys:completionHandler: 方法可以异步载入一个属性或多个属性。查询多个属性时，回调块也只执行一次，需要为每个查询的属性调用查询其状态的方法。
- 元数据被嵌入到媒体格式中，用来对媒体描述。Apple环境的媒体类型主要有4种，QuickTime(mov)、MPEG-4 video(mp4和m4a)、MPEG-4 audio(m4a)、MPEG-Layer  Ⅲ audio(mp3)。
- QuickTime 架构定义了.mov文件的内部结构。由atoms数据结构组成。atom包含ftyp(描述文件类型和兼容类型)、mdat(音视频媒体)、moov(细节描述)。moov/meta/ilst/ 包含标准元数据，键都是com.apple.quicktime前缀，例如标题、年份等。moov/udta/ 包含用户数据，如演唱者、版权信息。
- MPEG-4 MP4派生于QuickTime格式，也由atom数据结构组成。元数据在moov/udta/meta/ilst/ 中。.mp4是标准扩展，变体如.m4v .m4a .m4p .m4b 都是MPEG-4容器格式，只不过包含附加扩展。.m4v包含FairPlay加密和AC3-audio扩展，.m4a针对音频，m4p是iTunes音频格式，使用FairPlay扩展，m4b针对有声读物。
- MP3，不用容器格式，使用编码音频数据，元数据结构块位于文件开头。用ID3v2格式保存描述信息，含有歌曲演唱者、唱片、音乐风格等。前10个字节表示ID3块头部，剩下的数据都是描述元数据键值的帧。AV Foundation 支持读取ID3v2标签的所有版本，不支持写入。ID3v2.2 和 ID3v2.3版本的布局不同。
- AVMetadataItem类是访问元数据的接口类。键空间可以组合相关键，实现对 AVMetadataItem 实例的筛选。common键空间，定义如曲名、歌手和插图信息等元素，使用commonMetadata属性查询。metadataForFormat:方法访问指定键查询元数据，availableMetadataFormats查询所有支持的键，可以遍历其结果为每个键调用metadataForFormat:方法来查询所有元数据。(59页)
- 使用键空间值和键值获取一个资源的演奏者和唱片的元数据，用方法 metadataItemsFromArray:withKey:keySpace:拿到AVMetadataItem实例。(60页)
- AVMetadataItem 包含键值对，有key属性、value属性。value可以是字符串、数字、NSData、字典。还有commonKey属性。直接查询key属性可能只会得到一串数字(61页)，可以使用分类将其转化为字符串。(61页)
- \>> 右移运算符，右边丢弃，左边补0或1。CFSwapInt32BigToHost() 方法，用于从大端转为小端模式。iOS中默认小端模式。
- iOS 8已有用标识符(identifier)获取元数据的方法，可以不用键和键空间获取资源了。
- 从 AVMetadataItem 获取Artwork(插图)，其value属性是NSData或字典。获取comment(注释)、track(音轨)，其中number表示第几首，count表示共多少首，或者叫音轨编号和计数值。disc表示唱片数据，和音轨类似，number表示第几张，count表示共多少张唱片，或者是唱片编号和计数值。genre表示风格，MP3有126个风格，iTunes除此之外还定义了对电视、电影、有声读物的风格集。
- NSImage的TIFFRepresentation 属性，返回对应的NSData类数据。
- AVAssetExportSession 可以将 AVAsset 资源转码导出到磁盘中。可以用来写入新的元数据，需要导出预设确定内容的质量、大小，outputURL 确定导出的地址，outputFileType 确定导出格式，exportAsynchronouslyWithCompletionHandler: 方法开始导出。
- AVAssetExportPresetPassthrough 只修改元数据，不重新编码，可以用这个预设，过程时间很短。不过其不可以添加元数据，也不能修改ID3标签(即MP3文件不支持写入)。

### 4 视频播放

- AVPlayer 控制器用来播放时基媒体资源，支持本地、分步下载、HTTP Live Streaming 协议流媒体。其不可见，可以播放MP3、AAC音频文件，需要使用AVPlayerLayer类来展示界面。
- AVPlayer 管理单独资源的播放，如果需要多个资源播放或设置循环播放，可用AVQueuePlayer。
- AVPlayerLayer 基于 Core Animation，Core Animation 是负责图形渲染和动画的框架，其基于 OpenGL，性能较好。AVPlayerLayer 和 CALayer一样，可作为UIView的备用层。
- AVPlayerLayer 中videoGravity属性，用来设置视频拉伸缩放。resizeAspect 缩放视频大小保持原始宽高比，是默认值。resizeAspectFill 保持宽高比填满播放区域，会导致部分内容被裁剪。resize 会拉伸填满，会导致变形，不常用。
- AVPlayerItem 用来建立媒体资源的动态数据模型，有seekToTime: 方法，currentTime、presentationSize 属性。AVPlayerItemTrack实例表示类型统一的媒体流，AVPlayerItem 与 AVAssetTrack 实例对应。
- 用 AVAsset 创建 AVPlayerItem，用AVPlayerItem创建 AVPlayer，用AVPlayer创建AVPlayerLayer，将AVPlayerLayer添加到视图layer上。
- AVPlayerItem的AVPlayerItemStatus属性，创建时是AVPlayerItemStatusUnknown，表示还没载入，当关联AVPlayer后才可播放，状态会变为AVPlayerItemStatusReadyToPlay，可用kvo监测status属性变化。(89页)
-  使用NSTimeInterval(就是double)，即浮点值会导致运算不精确的情况，AV Foundation 采用更可靠的CMTime数据结构来展示时间。CMTime定义(90页)，关键值是value和timescale，都是整数，分别做分子和分母。CMTimeMake(1,2)表示0.5秒，kCMTimeZero表示0。
- 可以用UIView的layerClass方法返回AVPlayerLayer类。
- 定期时间监听，AVPlayer的 addPeriodicTimeObserverForInterval:queue:usingBlock:。边界时间监听方法，如25%、50%、75%。
- AVAssetImageGenerator 类用来从视频中提取图片，生成缩略图，可以生成一个或一组图片。属性 maximumSize 设置宽度100高度0意为生成固定宽度，高度视频宽高比计算的缩略图。(102页)
- 字幕，AVMediaSelectionGroup和AVMediaSelectionOption，可用的媒体特征有视频、音频、字幕。一个资源可包含多个音频轨道和字幕轨道。AVMediaCharacteristicLegible代表字幕特性。
- AirPlay 音视频在Apple TV上播放，音频在第三方系统播放。AVPlayer 的 allowExternalPlayback 启用或禁用，默认启用。内部启用该功能用MPVolumeView类，包含音量滑动条和线路选择按钮。可以禁用音量滑动条。WiFi启动且有可用线路时才会显示线路选择按钮。
- 通读源码，UINavigationBar的topItem表示当前栈顶的UINavigationItem，可以设置title。backItem，表示上一个。

### 5 AV Kit

- 在iOS和macos上都有，iOS 8 引入iOS。
- MPMoviePlayerController (media player框架)，定义了播放控件，支持AirPlay，无法使用更高级的功能。
- AVPlayerViewController 是视图控制器类，可控制AVPlayer实例，可展示播放控件，有videoGravity 设置播放显示模式，有动态播放控件。iOS 8推出，支持更高级的功能。
- AVPlayerView 是NSView子类，属于mac平台的AV Kit 框架。提供搓擦条、音量控制器、动态章节字幕菜单、分享菜单。可以修改控制栏的显示样式，有内嵌类型、浮动类型、最小化类型、None类型。只有内嵌和浮动类型支持设置分享菜单。
- 可以用代码获取章节数据，可以用NSMenu实现跳转上一章和下一章的功能。
- 数组的遍历方法 enumerateObjectsWithOptions:usingBlock: 第一个参数 NSEnumerationOptions 如果传入 NSEnumerationReverse，代表数组从后向前循环。
- Core Media 定义了大量函数和宏，其中常用的 CMTIME_COMPARE_INLINE(time1, comparator, time2) 是比较两个时间的。
- 资源可以修剪，但是需要导出，源资源是不可改变的。
- 导出用 AVAssetExportSession，需要一个预设和资源。设置timeRange属性，导出修剪后的视频，如果没修剪会导出全部视频。
- QTKit框架的类QTMovieModernizer类，可以解码过时的文件格式的视频并播放。

### 6 媒体捕捉

- AVCaptureSession 捕捉会话，连接输入和输出的资源，捕捉从物理设备得到的数据流，如摄像头、麦克风。会话可设置预设值，控制格式和质量，默认 AVCaptureSessionPresetHigh.
- AVCaptureDevice 为物理设备定义接口，如摄像头的对焦、曝光、白平衡和闪光。defaultDeviceWithMediaType: 方法为给定类型返回设备，传入AVMediaTypeVideo iOS系统会返回后置摄像头。
- AVCaptureDeviceInput为device和会话连线，用device创建DeviceInput。
- AVCaptureOutput是抽象基类，扩展类如AVCaptureStillImageOutput(静态图片)、AVCaptureMovieFileOutput(视频)。底层扩展类AVCaptureAudioDataOutput、AVCaptureVideoDataOutput，可以访问数字样本，提供更强大的功能，如对音频、视频流实时处理。
- AVCaptureConnection 连接输入和输出，对音频、视频分别建立连接，可以禁用某些连接或单独访问音频连接里的音轨。
- AVCaptureVideoPreviewLayer 实时预览捕捉到的视频，支持缩放拉伸，resizeAspect、resizeAspectFill、resize 3种模式。
- AVCaptureVideoPreviewLayer 有session属性，captureDevicePointOfInterestForPoint方法将屏幕触坐标系转换为摄像头设备坐标系的点，两者可以互转。
- 设备坐标系左上角(0,0)，右下角(1,1)。
- AVCaptureSession 的startRunning 和 stopRunning 方法都需要在非主线程异步调用，因为其为同步调用会消耗一定时间。
- 隐私提升对话框，创建AVCaptureDeviceInput时会触发，触发时系统会返回设备，例如静音的音频设备和黑白帧的相机设备，用户同意开始捕捉，否则不会记录内容，不同意时下次创建时会返回nil并收到Error。这时应弹出提示让用户在设置中开启隐私。
- 有前后置摄像头时可以切换摄像头，会话要动态重新配置，调用会话的 beginConfiguration和 commitConfiguration 方法，进行单独的原子性的修改。
- 对设备进行配置，对焦、曝光、白平衡、闪光灯、手电筒。前置摄像头不支持对焦。配置时需要设置锁，lockForConfiguration: 和 unlockForConfiguration，设置完需要释放锁。
- 支持兴趣点对焦、设定自动对焦模式，兴趣点曝光、自动曝光模式、锁定曝光模式(需要kvo来观察设备的adjustingExposure属性，知道何时曝光调整完成后设置为锁定曝光模式，注意这里的异步回主队列问题)，重设对焦和曝光兴趣点及模式。
- 手电筒和闪光灯模式调整，拍照时用到闪光灯，录视频时用到手电筒。可以切换不同的模式，分别支持关闭、开启、自动3种模式。
- AVCaptureConnection 连接是自动建立的，CMSampleBuffer 用来保存捕捉到的图片。可以将捕捉到的图片取出来，设置方向，存储到照片库中。
- Assets Library 框架，ALAssetLibrary 类访问照片库，可以写入，第一次调用时会弹出隐私提示。拒绝会导致写入失败，并返回错误信息。ALAuthorizationStatus 可获取现在的授权状态。
- UIImage 有 imageOrientation属性，表示图片方向，共有8个枚举值，分别为上下左右，镜像的上下左右。
- AVCaptureMovieFileOutput 父类 AVCaptureFileOutput 有录制时长限制、大小限制、保留最小可用磁盘空间。捕捉的QuickTime影片，影片头在末尾，如果遇到崩溃和电话拨入，影片头就不能正确写入。AVCaptureMovieFileOutput 用分段捕捉方法解决该问题。movieFragmentInterval 属性可以修改分段间隔，分段写入片段可以完整创建影片头。
- 录制视频，可以设置视频方向、稳定性，设备设置平滑对焦。
- 从视频中生成缩略图，用 AVAssetImageGenerator 类，设置maximumSize 和 appliesPreferredTrackTransform(捕捉时考虑视频变化)属性。调用 copyCGImageAtTime:actualTime:error: 获取单张图，这是个同步方法，需要在非主线程的异步线程调用。
- 通读工程代码，其中有自定义 UIControl 子类的方法，使用 CABasicAnimation 和 CAAnimationGroup 的方法以及用CALayer画圆的方法。使用CATextLayer的方法，使用 UISwipeGestureRecognizer 左滑右滑的手势，双击手势，双击双指触摸手势，手势依赖。
