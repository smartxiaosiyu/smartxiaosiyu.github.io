---
title: 'FFmpeg 命令行'
date: 2021-09-09 15:46:34
tags: [FFmpeg]
published: true
hideInList: false
feature: 
isTop: false
---
```
[1]volume=%.02f[a1]; [0][a1][2]
表示三个输入 但是只对第二个输入 使用流处理 
[1]volume=%.02f[a1]; [0][1][2]
表示三个输入 但是只对第二个输入做了流处理 但是不使用 
```

**FFmpeg 命令的典型语法是：**
```
ffmpeg [全局选项] {[输入文件选项] -i 输入_url_地址} ...
 {[输出文件选项] 输出_url_地址} ...
```
 + 1、获取音频/视频文件信息
  ```
  $ ffmpeg -i video.mp4
  不想看 FFmpeg 标语和其它细节
  $ ffmpeg -i video.mp4 -hide_banner
  ```
  + 2、转换视频文件到不同的格式
  ```
  $ ffmpeg -i video.mp4 video.avi

  如果你想维持你的源视频文件的质量，使用 -qscale 0 参数：

  $ ffmpeg -i input.webm -qscale 0 output.mp4
  ```
  + 3、转换视频文件到音频文件
   ```
   $ ffmpeg -i input.mp4 -vn output.mp3
   $ ffmpeg -i input.mp4 -vn -ar 44100 -ac 2 -ab 320 -f mp3 output.mp3

-vn – 表明我们已经在输出文件中禁用视频录制。
-ar – 设置输出文件的音频频率。通常使用的值是22050 Hz、44100 Hz、48000 Hz。
-ac – 设置音频通道的数目。
-ab – 表明音频比特率。
-f – 输出文件格式。在我们的实例中，它是 mp3 格式
   ```
+ 4、更改视频文件的分辨率
 ```
$ ffmpeg -i input.mp4 -filter:v scale=1280:720 -c:a copy output.mp4
$ ffmpeg -i input.mp4 -s 640x480 -c:a copy output.mp4
```
+ 5、压缩视频文件
 ```
  $ ffmpeg -i input.mp4 -vf scale=1280:-1 -c:v libx264 -preset veryslow -crf 24 output.mp4
```
+ 6、压缩音频文件
```
  $ ffmpeg -i input.mp3 -ab 128 output.mp3
 ```
+ 7、从一个视频文件移除音频流
```
  $ ffmpeg -i input.mp4 -an output.mp4

  -an 表示没有音频录制
```
+ 8、从一个媒体文件移除视频流
```
  $ ffmpeg -i input.mp4 -vn output.mp3
  
  -vn 代表没有视频录制
  也可以使用 -ab 标志来指出输出文件的比特率
  $ ffmpeg -i input.mp4 -vn -ab 320 output.mp3
```
+ 9、从视频中提取图像
 ```
    $ ffmpeg -i input.mp4 -r 1 -f image2 image-%2d.png
    在这里，

    -r – 设置帧速度。即，每秒提取帧到图像的数字。默认值是 25。
    -f – 表示输出格式，即，在我们的实例中是图像。
    image-%2d.png – 表明我们如何想命名提取的图像。在这个实例中，命名应该像这样image-01.png、image-02.png、image-03.png 等等开始。如果你使用 %3d，那么图像的命名像 image-001.png、image-002.png 等等开始。
```
+ 10、裁剪视频
```
ffmpeg -i input.mp4 -filter:v "crop=w:h:x:y" output.mp4

input.mp4 – 源视频文件。
-filter:v – 表示视频过滤器。
crop – 表示裁剪过滤器。
w – 我们想自源视频中裁剪的矩形的宽度。
h – 矩形的高度。
x – 我们想自源视频中裁剪的矩形的 x 坐标 。
y – 矩形的 y 坐标
```
+ 11、转换一个视频的具体的部分
```
你可能想仅转换视频文件的一个具体的部分到不同的格式。以示例说明，下面的命令将转换所给定视频input.mp4 文件的开始 10 秒到视频 .avi 格式。

$ ffmpeg -i input.mp4 -t 10 output.avi
```
+ 12、设置视频的屏幕高宽比
```
你可以使用 -aspect 标志设置一个视频文件的屏幕高宽比，像下面。

$ ffmpeg -i input.mp4 -aspect 16:9 output.mp4
```
+ 13、添加海报图像到音频文件
```
可以添加海报图像到你的文件，以便图像将在播放音频文件时显示。这对托管在视频托管主机或共享网站中的音频文件是有用的

$ ffmpeg -loop 1 -i inputimage.jpg -i inputaudio.mp3 -c:v libx264 -c:a aac -strict experimental -b:a 192k -shortest output.mp4
```
+ 14、使用开始和停止时间剪下一段媒体文件
```
$ ffmpeg -i input.mp4 -ss 00:00:50 -codec copy -t 50 output.mp4

-s – 表示视频剪辑的开始时间。在我们的示例中，开始时间是第 50 秒。
-t – 表示总的持续时间
我们可以像下面剪下音频。

$ ffmpeg -i audio.mp3 -ss 00:01:54 -to 00:06:53 -c copy output.mp3
```
+ 15、切分视频文件为多个部分
```
一些网站将仅允许你上传具体指定大小的视频。在这样的情况下，你可以切分大的视频文件到多个较小的部分

$ ffmpeg -i input.mp4 -t 00:00:30 -c copy part1.mp4 -ss 00:00:30 -codec copy part2.mp4
-t 00:00:30 表示从视频的开始到视频的第 30 秒创建一部分视频。
-ss 00:00:30 为视频的下一部分显示开始时间戳。它意味着第 2 部分将从第 30 秒开始，并将持续到原始视频文件的结尾
```
+ 16、接合或合并多个视频部分到一个
```
FFmpeg 也可以接合多个视频部分，并创建一个单个视频文件。

创建包含你想接合文件的准确的路径的 join.txt。所有的文件都应该是相同的格式（相同的编码格式）。所有文件的路径应该逐个列出，像下面。

file /home/sk/myvideos/part1.mp4
file /home/sk/myvideos/part2.mp4
file /home/sk/myvideos/part3.mp4
file /home/sk/myvideos/part4.mp4
$ ffmpeg -f concat -safe 0 -i join.txt -c copy output.mp4
上面的命令将接合 part1.mp4、part2.mp4、part3.mp4 和 part4.mp4 文件到一个称为 output.mp4 的单个文件中
```
+ 17、添加字幕到一个视频文件
```
$ fmpeg -i input.mp4 -i subtitle.srt -map 0 -map 1 -c copy -c:v libx264 -crf 23 -preset veryfast output.mp4
```
+ 18、预览或测试视频或音频文件
```
你可能希望通过预览来验证或测试输出的文件是否已经被恰当地转码编码。为完成预览，你可以从你的终端播放它，用命令：

$ ffplay video.mp4
类似地，你可以测试音频文件，像下面所示。

$ ffplay audio.mp3
```
+ 19、增加/减少视频播放速度
```
FFmpeg 允许你调整视频播放速度。

为增加视频播放速度，运行：

$ ffmpeg -i input.mp4 -vf "setpts=0.5*PTS" output.mp4
该命令将双倍视频的速度。

为降低你的视频速度，你需要使用一个大于 1 的倍数。为减少播放速度，运行：

$ ffmpeg -i input.mp4 -vf "setpts=4.0*PTS" output.mp4
```

FFmpeg命令列語法之-filter_complex: https://www.itread01.com/p/15577.html

+ 20、获取视频的信息
```
ffmpeg -i video.avi
```

+ 21、将图片序列合成视频
```
ffmpeg -f image2 -i image%d.jpg video.mpg
上面的命令会把当前目录下的图片（名字如：image1.jpg. image2.jpg. 等…）合并成video.mpg
```

+ 22、将视频分解成图片序列
```
ffmpeg -i video.mpg image%d.jpg
上面的命令会生成image1.jpg. image2.jpg. …
支持的图片格式有：PGM. PPM. PAM. PGMYUV. JPEG. GIF. PNG. TIFF. SGI
```

### 遇到的问题
```
FATAL: Automatic encoder selection failed for output stream #0:1. Default encoder for format mp3 (codec png) is probably disabled. Please choose an encoder manually.
2021-10-11 15:31:36.714322+0800 AudioClipDemo[18260:3684337] FATAL: Error selecting an encoder for stream 0:1
```
这是由于mp3有封面图 编码的时候按照封面图从mp3中编码 所以有问题

./configure --list-encoders 在ffmpeg目录下 查找支持的编码列表
