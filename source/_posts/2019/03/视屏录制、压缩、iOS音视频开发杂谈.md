---
title: 视屏录制、压缩、iOS音视频开发杂谈
date: 2019-03-06 15:03:09
tags: 
---



# 视频录制
先看一张iOS录制方式的总结UML类图：
![](https://ws4.sinaimg.cn/large/006tKfTcgy1g0t3dd5gwyj31gd0irgpb.jpg)


## UIImagePickerController
* **UIImagePickerController**：系统的相机控制器，只能设置一些简单参数(如视频的画质等)，自定义程度不高，只能自定义界面上的操作按钮。

## AVFoundation
- **AVFoundation**：是一个可以用来使用和创建基于媒体的框架，它提供了一个能使用基于视频监听数据的接口。类图中的**AVCaptureMovieFileOutput**和**AVAssetWriter**两种方式都是AVFoundation中的。

- **AVCaptureSession(捕捉会话)**：从捕捉设备(如摄像头、麦克风)得到数据流，输出到一个或多个目的地。AVCaptureSession可以动态配置输入输出(如前后摄像头)的线路，在会话进行中按需重新配置捕捉环境。 AVCaptureSession 可以额外配置一个会话预设值(preset，默认是高质量AVCaptureSessionPresetHigh),用来控制捕捉数据的质量。

-  **AVCaptureDeviceInput(捕捉设备的输入)**：在使用捕捉设备进行处理之前，需要将它添加到捕捉会话的输入。不过一个设备不能直接添加到AVCaptureSession 中，需要利用 AVCaptureDeviceInput 的一个实例封装起来添加。

- **AVCaptureDevice(捕捉设备)**：针对物理硬件设备(如控制摄像头的对焦、曝光、白平衡和闪光灯等)定义了大量的控制方法。 

- **AVCaptureOutput** 捕捉设备的输出 - 如上文所提，AVCaptureSession 会从AVCaptureDevice拿数据流，并输出到一个或者多个目的地，这个目的地就是 AVCaptureOutput。 
  AVCaptureOutput是一个基类，AVFoundation为我们提供了 四个 扩展类:  
  - AVCaptureStillImageOutput 捕捉静态照片（拍照） 
  - AVCaptureMovieFileOutput  捕捉视频（视频 + 音频） 
  - AVCaptureVideoDataOutput  视频录制数据流 
  - AVCaptureAudioDataOutput  音频录制数据流 
  
其中AVCaptureVideoDataOutput和AVCaptureAudioDataOutput不停地往AVAssetWriter传递VideoSampleBuffer和AudioSampleBuffer，AVAssetWriter对VideoSampleBuffer做分辨率压缩，以及对AudioSampleBuffer做码率压缩，以便可以更好的进行音频视频实时处理。对于以上四者的关系，类似于 AVCaptureSession 是过滤器，AVCaptureDevice 是“原始”材料，AVCaptureDeviceInput 是 AVCaptureDevice 的收集器，AVCaptureOutput 就是产物了。
 
- **AVCaptureConnection** 捕捉连接 - 那么问题来了，上面四者之间的“导管”是什么呢？那就是 AVCaptureConnection。利用AVCaptureConnection 可以很好的将这几个独立的功能件很好的连接起来。 
- **AVCaptureVideoPreviewLayer** 捕捉预览，以上所有的数据处理，都是在代码中执行的，用户无法看到AVCaptureSession到底在做什么事情。所以AVFoundation 为我们提供了一个叫做 AVCaptureVideoPreviewLayer 的东西，提供实时预览。AVCaptureVideoPreviewLayer 是 CoreAnimation 的 CALayer 的子类。 
 关于预览层的填充模式有 AVLayerVideoGravityResizeAspect、AVLayerVideoGravityResizeAspectFill、AVLayerVideoGravityResize三种

## AVCaptureMovieFileOutput和AVAssetWriter的异同：
**相同点**：两者的输入基本是一样的，数据采集都在AVCaptureSession中进行，视频和音频的输入都一样，画面的预览等也一致。

**不同点**：两者最大的不同就是输出方式不一致。
- **AVCaptureMovieFileOutput**只需要一个输出即可，指定一个文件路后，视频和音频会写入到指定路径，不需要其他复杂的操作。这是它的使用优点，但同时也带了音视频不能在录制的时候就进行处理，只能先读文件再进行处理，这样也造成了它耗时的缺点等等。
- **AVAssetWriter**需要AVCaptureVideoDataOutput和AVCaptureAudioDataOutput两个单独的输出，拿到各自的输出数据后，然后自己进行相应的处理，这种方式处理非常灵活。

附上[在Objc中国:【iOS上捕获视频】](https://objccn.io/issue-23-1/)中的图和建议：
![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0t3el933wj30hc0e6myl.jpg)
**AVCaptureDataOutput 和 AVAssetWriter
如果你想要对影音输出有更多的操作，你可以使用 AVCaptureVideoDataOutput 和 AVCaptureAudioDataOutput 而不是我们上节讨论的 AVCaptureMovieFileOutput。
这些输出将会各自捕获视频和音频的样本缓存，接着发送到它们的代理。代理要么对采样缓冲进行处理 (比如给视频加滤镜)，要么保持原样传送。使用 AVAssetWriter 对象可以将样本缓存写入文件。**

想要深入研究自定义相机并做更多复杂处理的，这里推荐一个参考库[PBJVision](https://github.com/piemonte/PBJVision) ，或许能给你一些启发🤔。
<br>
<br>

# 二、视频压缩
iOS视频压缩是用AVFoundation中的**AVAssetExportSession**这个类来进行处理的，AVAssetExportSession是对AVAsset对象内容进行转码, 并输出到制定的路径;
```
默认初始化方法
- (nullable instancetype)initWithAsset:(AVAsset *)asset presetName:(NSString *)presetName NS_DESIGNATED_INITIALIZER;
```
其中的的参数分别是：
asset, 参数为要转码的asset对象;
presetName, 该参数为要进行转码的方式名称, 为字符串类型, 系统有给定的类型值:
```
AVF_EXPORT NSString *const AVAssetExportPresetLowQuality         NS_AVAILABLE(10_祝你：生日快乐！幸福、平安！ 杨辉 , 4_0);
AVF_EXPORT NSString *const AVAssetExportPresetMediumQuality      NS_AVAILABLE(10_祝你：生日快乐！幸福、平安！ 杨辉 , 4_0);
AVF_EXPORT NSString *const AVAssetExportPresetHighestQuality     NS_AVAILABLE(10_祝你：生日快乐！幸福、平安！ 杨辉 , 4_0);
AVF_EXPORT NSString *const AVAssetExportPreset640x480           NS_AVAILABLE(10_7, 4_0);
AVF_EXPORT NSString *const AVAssetExportPreset960x540           NS_AVAILABLE(10_7, 4_0);
AVF_EXPORT NSString *const AVAssetExportPreset1280x720          NS_AVAILABLE(10_7, 4_0);
AVF_EXPORT NSString *const AVAssetExportPreset1920x1080         NS_AVAILABLE(10_7, 5_0);
AVF_EXPORT NSString *const AVAssetExportPreset3840x2160         NS_AVAILABLE(10_10, 9_0);
```
**注：AVAssetExportSession默认使用的音频编码格式是AAC，视频编码格式是H.264。**

此外还需要设置两个属性：
```
// 转码后视频的输出类型
exportSession.outputFileType = AVFileTypeMPEG4;
// 转码后视频的输出路径
exportSession.outputURL = [NSURL fileURLWithPath:outPath];
```

<br>

AVAssetExportSession是苹果在AVFoundation框架中提供的，使用起来也比较简单，当然同时也比较单一，只能使用一套转码预设，如果不满足你的需求，可以看看这个[SDAVAssetExportSession](https://github.com/rs/SDAVAssetExportSession)🤔，或许会对你有所帮助。

<br>
<br>


# 三、客户端音视频开发
既然说到了视频的录制和压缩，那么索性再理论性的探讨下客户端音视频开发的流程。😄
### 1、什么是客户端音视频开发呢？
音视频开发就是将采集到的音频和视频数据流持续的输出到显示器进行显示。我一般通俗的将它理解成：画画和放歌。
#### 画什么画❓
> 解码采集到的视频数据流，将它画成一帧帧画面 

#### 放什么歌❓
> 解码采集到的音频数据流，进行播放

<br>

### 2、音视频数据从哪里来❓
“画画”和“放歌”首先的有数据吧，那么数据从哪里来呢？对于客户端来说，并不关心数据到底是从硬件设备(智能手机摄像头采集)还是从服务器(远程主播设备的摄像头采集)到的，这些都有现成的许多框架，它都帮我们封装了很多功能。
音视频开发有三大步骤：
* 传输层
* 编解码
* 渲染输出

**传输层：**解决数据哪来的问题。音视频开发讲究的是实时性，而且也需要持续输出，所以音视频开发都是进行长链接的，
**编解码：**负责数据压缩传输和解压输出问题 (编解码又分硬解和软解，手机硬件解码称为硬解，利用算法消耗CPU资源解码称为软解如ffmpeg)
**渲染输出：**负责渲染视频数据和将音频数据传给手机麦克风或者耳机进行播放(画面渲染就是OpenGL, 音频输出就是Audio Unit)

<br>

### 3、什么是数据❓
![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0t3f94816j30pq04smxo.jpg)

通过与服务器进行长链接，接收服务器传过来的持续不断的数据流。那么问题又来了，这些数据流是以什么样的方式进行传输呢？
[网络传输TCP传输一个包大小](https://blog.csdn.net/caoshangpa/article/details/51530685)最大是有限制的，音视频的数据比较大，需要很多个包进行传输，也就形成了数据流，而且这些传输数据中还分音频和视频数据，一下子传过来我们该怎么识别呢？这里提出两个问题：
> 1、传给我的是什么数据？
> 2、传给我的数据到底有多大？

带着这两个问题，可能很容易就想到：客户端是不是应该和服务器之前商定一个传输规则？
是的，没错，服务器过来的数据一般都是遵循了这么一个协议的数据**TLV**，看上面那张图：
* **type**：传输的是音频/视频？

* **length**：传输的音频/ 视频多长?

* **value**：音频/视频数据

type和length这两个很好理解，那么作为音频或者是视频数据的value又是什么样的类型呢？
相信大家都**AAC/H.264**这些名词都早有耳闻了，没错，value传输的就是这种类型，当type类型是音频的时候传输的是AAC格式的音频数据，当type类型是视频的时候传输的是H.264格式的音频数据(当然还有其他的音频格式如WAV/MP3/Ogg，视频格式有H.261/H.262/H.263等等)。

**但这里要说明一点**：服务器传输的value类型虽然是AAC/H.264，但这并不是说最初我们采集到的数据就是AAC/H.264了。最开始摄像头采集到的视频数据是YUV格式的，麦克风采集到的音频数据是PCM格式的；但是“一帧画面”的YUV格式数据非常大，不便于传输，所以将其压缩成H.264格式的便于传输，这个过程就是我们常说的编码。音频数据PCM格式压缩成AAC格式也是同样的道理。

这个时候我们客户端接收到数据就和容易识别了：收到的数据前面多少个字节是表示数据类型，多少个字节是表示数据大小，后面剩余的就是该类型的实际数据了。

<br>

### 4、编码解码
前面说过了YUV和PCM格式的数据太大，不便于传输，将其压缩成H.264和AAC格式的进行传输，这个压缩过程就是编码。

此时客户端收到了服务器的数据，虽然知道了是什么类型的数据，也知道了数据的大小，但是依然显示不出来，这是是为什么呢？
因为手机只认识RGB和PCM格式，什么H.264/AAC它根本就不知道是什么鬼东西。

what？RGB又出来了，那刚刚说的采集的YUV又是什么鬼？
![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0t3fmwibgj3064064t8v.jpg)

**RGB**：这个就不解释了吧，不知道的面壁去～。在屏幕上看到的图片呀文字呀, 都是许多由一个一个的像素拼接起来的, 每一个像素点的数据就是RGB, 所以你看到的图片文字,你在你手机电脑上看到的任何东西都是许多红色、绿色、蓝色组合成的。

**YUV**：视频帧的裸数据表示形式，主要用于优化彩色视频信号的传输，使其向后兼容老式的黑白电视机。
这样解释YUV你可能还是有点懵逼，好吧，那通俗点说就是：
![](https://ws1.sinaimg.cn/large/006tKfTcgy1g0t3g6exdsj30j60693yw.jpg)

**Y：**表示明亮度(Luminance或Luma)，也称灰阶值，简单来说就是表示黑白的；

**UV：**表示色度(Chrominance或Chroma)，他的作用是描述影像的色彩及饱和度的，用于指定像素的颜色。
老式的黑白电视机就是只有Y信号分量而没有U、V信号分量，表示的就是黑白灰度图像。

那RGB和YUV的关系是什么呢? RGB和YUV都可以通过对方,进行一个矩阵计算的得到, RGB通过一个矩阵计算可以生成YUV,  YUV可以通过一个矩阵计算生成RGB.
![](https://ws3.sinaimg.cn/large/006tKfTcgy1g0t3gi7vpcj307w04t0sy.jpg)

[想了解更多RGB和YUV的可以点击这里](https://blog.csdn.net/asahinokawa/article/details/80596655)

<br>

所以手机说手机要YUV或者RGB都没问题.可以计算得到的.那怎么得到YUV了? 
将接收到的value中通过压缩得到的H.264解码不久得到YUV了嘛。这个过程就是常说的解码了。

解码又分硬解和软解；
- 硬解：手机通过自身的硬解模块进行解码
- 软解：就是利用设计的算法程序通过，消耗CPU的资源来进行解码，如FFmpeg。

<br>

### 5、渲染输出
通过上面的解码，我们得到了YUV和AAC。

音频播放：将AAC数据给audio unit，由audio unit解码得到PCM之后拿给硬件，硬件就可以播放了。

视频播放：如需要RGB可以通过计算YUV得到RGB，其实RGB/YUV就是一帧画面的一个像素点，视频的本质其实就是一帧帧画面组成的，一般视频每秒大概25～30帧，我们能看到视频是连续的而不是一张张图片播放是因为人眼一般识别帧数是24/s左右。

既然知道了每个像素点，那就开始“画画”(渲染)吧～
 
怎么画？ 用OpenGL画。
至于怎么用OpenGL画？那这个就有点大了，2D的平面还好说，3D的立体效果那就得靠各位自己去研究了。

渲染大致过程这里以FFmpeg软解为例：利用CPU完成渲染所需的计算，并将计算结果提交到GPU进行合成、变换、渲染，渲染完成后把渲染结果提交到帧缓冲区去，等待下一次 VSync 信号到来时显示到屏幕上，这样我们就能看到画面了。

<br>

**总结**：以上就是视频录制、压缩和客户端音视频开发理论流程，当然实际开发中有许多的细节，远不止这东西，如果大家有兴趣的话可以深入研究。

<br>

**参考文档：**
[在Objc中国:【iOS上捕获视频】](https://objccn.io/issue-23-1/)
[iOS三种录制视频方式详细对比](http://gcblog.github.io/2017/03/22/iOS%E4%B8%89%E7%A7%8D%E5%BD%95%E5%88%B6%E8%A7%86%E9%A2%91%E6%96%B9%E5%BC%8F%E8%AF%A6%E7%BB%86%E5%AF%B9%E6%AF%94/) 
[iOS小视频优化心得](https://mp.weixin.qq.com/s?__biz=MzAwNDY1ODY2OQ==&mid=207686973&idx=1&sn=1883a6c9fa0462dd5596b8890b6fccf6&3rd=MzA3MDU4NTYzMw==&scene=6#rd)
[iOS小视频功能开发优化记录](https://juejin.im/entry/5787814179bc44005fbd61c6) 
[对颜色空间YUV、RGB的理解](https://blog.csdn.net/asahinokawa/article/details/80596655)
[iOS音视频开发闲谈](https://www.jianshu.com/p/1fdcae98191c)







