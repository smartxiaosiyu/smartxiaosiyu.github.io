---
title: 'iOS AVFoundation 音视频剪辑和预览（合并、混音、插入、裁剪）'
date: 2021-09-28 11:47:13
tags: [音视频,AVFoundation]
published: true
hideInList: false
feature: 
isTop: false
---
### 音频合并
https://www.jianshu.com/p/fb063d592a3c  

###音频剪切
https://juejin.cn/post/6844903781662982158

###iOS视音频实用Demo
https://www.twblogs.net/a/5b8ce6c72b7177188336bec1/?lang=zh-cn

### 混合音频 修改音量
https://www.shangmayuan.com/a/d6810fb117b24bef846b4ed7.html

### 1、音视频资源组装
在 AVFoundation 中，视频和音频数据可以用 AVAsset 表示，AVAsset 里面包含了 AVAssetTrack 数据。

比如：一个视频文件里面包含了一个视频 track 和两个音频 track。
图：AVAsset 及其子类结构
![](https://smartxiaosiyu.github.io/post-images/1632813359808.jpeg)

可以使用 AVComposition 对 track 进行裁剪和变速等操作，也可以把多段 track 拼接到 AVComposition 里面，在处理完 track 的拼接和修改后，得到最终的 AVComposition，它是 AVAsset 的子类，也就是说可以把它传递到 AVPlayer、AVAssetImageGenerator、AVExportSession 和 AVAssetReader 里面作为数据源，把 AVComposition 当成是一个视频数据进行处理。

### 2、视频画面拼接/混音处理
 + AVFoundation 提供了 AVVideoComposition 对象和 AVVideoCompositing 协议用于处理视频的画面帧。**使用 AVMutableComposition 类可以增删 AVAsset 来将单个或者多个 AVAsset 集合到一起，用来合成新视频**
AVFoundation 类 API 中最核心的类是 AVVideoComposition / AVMutableVideoComposition 。

**AVVideoComposition / AVMutableVideoComposition 对两个或多个视频轨道组合在一起的方法给出了一个总体描述**。它由一组时间范围和描述组合行为的介绍内容组成。这些信息出现在组合资源内的任意时间点。

AVVideoComposition / AVMutableVideoComposition 管理所有视频轨道，可以决定最终视频的尺寸，裁剪需要在这里进行；

**AVMutableCompositionTrack 将多个 AVAsset 集合到一起合成新视频中轨道信息，有音频轨、视频轨等，里面可以插入各种对应的素材（画中画，水印等）；**

![](https://smartxiaosiyu.github.io/post-images/1632813569403.jpeg)

 + 2.2 AVFoundation 中的音频处理
AVFoundation 提供了 AVAudioMix 用于处理音频数据。 AVAudioMix 这个类很简单，只有一个 inputParameters 属性，它是一个 AVAudioMixInputParameters 数组。具体的音频处理都在 AVAudioMixInputParameters 里进行配置。AVAudioMixInputParameters 只能绑定单个 AVAssetTrack 的音频数据。

AVAudioMixInputParameters 内可以设置音量，支持分段设置音量，以及设置两个时间点的音量变化，比如 0 - 1 秒，音量大小从 0 - 1.0 线性递增。

AVAudioMixInputParameters 内还有个 audioTapProcessor 属性，他是一个 MTAudioProcessingTap 类。这个属性提供了接口用于实时处理音频数据。


> AVAsset 所有媒体资源的承载类。可以是音频，视频，图像。
AVMutableComposition 用于组合AVAsset的音视频轨道。
AVMutableCompositionTrack 分为视频轨道，音频轨道，它由AVAsset内部解码生成。
AVMutableVideoComposition 视频编辑指令管理类，它是整个视频编辑里动画指令组合。
AVMutableVideoCompositionInstruction 视频视图编辑指令，可以用它来设置视频的背景颜色等。它可以包含一组AVVideoCompositionLayerInstruction。
AVMutableVideoCompositionLayerInstruction 视频层编辑指令，可以用它设置视频的transform,opacity等。

https://zhuanlan.zhihu.com/p/369109995
https://www.codersrc.com/archives/10539.html

### 总结总体流程
AVFoundation 视频剪辑的整体工作流程：
![](https://smartxiaosiyu.github.io/post-images/1632815232101.png)

拆解下步骤：
 + 1.创建一个或多个 AVAsset。
 + 2.创建 AVComposition、AVVideoComposition 及 AVAudioMix。其中 AVComposition 指定了音视频轨道的时间对齐，AVVideoComposition 指定了视频轨道在任何给定时间点的几何变换与混合，AVAudioMix 管理音频轨道的混合参数。
 + 我们可以使用这三个对象来创建 AVPlayerItem，并从中创建一个 AVPlayer 来播放编辑效果。
 + 我们也可以使用这三个对象来创建 AVAssetExportSession，用来将编辑结果写入文件。

## AVComposition
**AVComposition** 是一个或多个 AVCompositionTrack **音视频轨道的集合**。其中 **AVCompositionTrack** 又可以**包含来自多个 AVAsset 的 AVAssetTrack**。
![](https://smartxiaosiyu.github.io/post-images/1632815522710.png)

## AVVideoComposition
AVVideoComposition 可以用来指定渲染大小和渲染缩放，以及帧率。此外，还存储了实现 AVVideoCompositionInstructionProtocol 协议的 Instruction（指令）数组，这些 Instruction 存储了混合的参数。有了这些混合参数之后，AVVideoComposition 可以通过一个实现 AVVideoCompositing 协议的 Compositor（混合器） 来混合对应的图像帧。

整体工作流如下图所示：
![](https://smartxiaosiyu.github.io/post-images/1632815956180.png) 

## AVAudioMix
使用 AVAudioMix，你可以在 AVComposition 的音频轨道上处理音频。AVAudioMix 包含一组的 AVAudioMixInputParameters，每个 AVAudioMixInputParameters 对应一个音频的 AVCompositionTrack。如下图所示
![](https://smartxiaosiyu.github.io/post-images/1632816124491.png)

https://xie.infoq.cn/article/9735e9fb133b5d294b175b4bd
