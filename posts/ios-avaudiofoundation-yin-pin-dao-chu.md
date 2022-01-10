---
title: 'iOS AVAudioFoundation 音频导出'
date: 2021-09-16 11:34:33
tags: [音视频,AVFoundation]
published: true
hideInList: false
feature: 
isTop: false
---
### AVAssetReader
当我们要去操作音视频数据资源 asset可以用AVAssetReader， 比如读取 asset 中的音频轨道来展示波形等等
AVAssetReader 不适用于做实时处理。AVAssetReader 没法用来处理 HLS 之类的实时数据流

#### 读取 Asset
每一个 AVAssetReader 一次只能**与一个 asset 关联**，但是这个 asset 可以**包含多个轨道**。由于这个原因通常我们需要为 AVAssetReader 指定一个 AVAssetReaderOutput 的具体子类来具体操作 asset 的读取

#### 创建 Asset Reader
初始化 AVAssetReader 时需要传入相应读取的 asset
```
NSError *outError;
AVAsset *someAsset = <#AVAsset that you want to read#>;
AVAssetReader *assetReader = [AVAssetReader assetReaderWithAsset:someAsset error:&outError];
BOOL success = (assetReader != nil);
```
#### 创建 Asset Reader Outputs
在完成创建 asset reader 后，创建**至少一个 output 对象**来**接收读取的媒体数据**。当创建 output 对象时，要记得设置 alwaysCopiesSampleData 属性为 NO，这样会获得性能上的提升。

如果我们想从媒体资源中读取一个或多个轨道，并且可能会转换数据的格式，那么可以使用 AVAssetReaderTrackOutput 类，为每个 AVAssetTrack 轨道使用一个 track output 对象

下面的示例展示了使用 asset reader 来把一个 audio track 压缩为线性的 PCM
```
AVAsset *localAsset = assetReader.asset;
// Get the audio track to read.
AVAssetTrack *audioTrack = [[localAsset tracksWithMediaType:AVMediaTypeAudio] objectAtIndex:0];
// Decompression settings for Linear PCM.
NSDictionary *decompressionAudioSettings = @{AVFormatIDKey: [NSNumber numberWithUnsignedInt:kAudioFormatLinearPCM]};
// Create the output with the audio track and decompression settings.
AVAssetReaderOutput *trackOutput = [AVAssetReaderTrackOutput assetReaderTrackOutputWithTrack:audioTrack outputSettings:decompressionAudioSettings];
// Add the output to the reader if possible.
if ([assetReader canAddOutput:trackOutput]) {
    [assetReader addOutput:trackOutput];
}
```

下面的代码展示了如何基于一个 asset 来创建我们的 audio mix output 对象去处理所有的音频轨道，然后压缩这些音频轨道为线性 PCM 数据，并为 output 对象设置 audioMix 属性
```
AVAudioMix *audioMix = <#An AVAudioMix that specifies how the audio tracks from the AVAsset are mixed#>;
// Assumes that assetReader was initialized with an AVComposition object.
AVComposition *composition = (AVComposition *) assetReader.asset;

// Get the audio tracks to read.
NSArray *audioTracks = [composition tracksWithMediaType:AVMediaTypeAudio];

// Get the decompression settings for Linear PCM.
NSDictionary *decompressionAudioSettings = @{AVFormatIDKey: [NSNumber numberWithUnsignedInt:kAudioFormatLinearPCM]};

// Create the audio mix output with the audio tracks and decompression setttings.
AVAssetReaderOutput *audioMixOutput = [AVAssetReaderAudioMixOutput assetReaderAudioMixOutputWithAudioTracks:audioTracks audioSettings:decompressionAudioSettings];

// Associate the audio mix used to mix the audio tracks being read with the output.
audioMixOutput.audioMix = audioMix;

// Add the output to the reader if possible.
if ([assetReader canAddOutput:audioMixOutput]) {
    [assetReader addOutput:audioMixOutput];
}
```

下面的代码展示了，如果读取多个视频轨道的媒体数据并将他们压缩为 ARGB 格式。
```
AVVideoComposition *videoComposition = <#An AVVideoComposition that specifies how the video tracks from the AVAsset are composited#>;
// Assumes assetReader was initialized with an AVComposition.
AVComposition *composition = (AVComposition *) assetReader.asset;

// Get the video tracks to read.
NSArray *videoTracks = [composition tracksWithMediaType:AVMediaTypeVideo];

// Decompression settings for ARGB.
NSDictionary *decompressionVideoSettings = @{(id) kCVPixelBufferPixelFormatTypeKey: [NSNumber numberWithUnsignedInt:kCVPixelFormatType_32ARGB], (id) kCVPixelBufferIOSurfacePropertiesKey: [NSDictionary dictionary]};

// Create the video composition output with the video tracks and decompression setttings.
AVAssetReaderOutput *videoCompositionOutput = [AVAssetReaderVideoCompositionOutput assetReaderVideoCompositionOutputWithVideoTracks:videoTracks videoSettings:decompressionVideoSettings];

// Associate the video composition used to composite the video tracks being read with the output.
videoCompositionOutput.videoComposition = videoComposition;

// Add the output to the reader if possible.
if ([assetReader canAddOutput:videoCompositionOutput]) {
    [assetReader addOutput:videoCompositionOutput];
}
```

#### 读取 Asset 的媒体数据
在 output 对象创建完成后，接着就要开始读取数据了，这时候我们需要调用 asset reader 的 startReading 接口。接着，使用 copyNextSampleBuffer 接口来从各个 output 来获取媒体数据。
```
// Start the asset reader up.
[self.assetReader startReading];
BOOL done = NO;
while (!done) {
    // Copy the next sample buffer from the reader output.
    CMSampleBufferRef sampleBuffer = [self.assetReaderOutput copyNextSampleBuffer];
    if (sampleBuffer) {
        // Do something with sampleBuffer here.
        CFRelease(sampleBuffer);
        sampleBuffer = NULL;
    } else {
        // Find out why the asset reader output couldn't copy another sample buffer.
        if (self.assetReader.status == AVAssetReaderStatusFailed) {
            NSError *failureError = self.assetReader.error;
            // Handle the error here.
        } else {
            // The asset reader output has read all of its samples.
            done = YES;
        } 
    }
}
```