---
title: '日积月累'
date: 2021-09-14 19:40:03
tags: [日积月累]
published: true
hideInList: false
feature: 
isTop: false
---
### 异或
```
0^0=0,0^1=1 0异或任何数＝任何数

1^0=1,1^1=0 1异或任何数－任何数取反

任何数异或自己＝把自己置0
```

### 按位与
```
（1）&可以用来取出特定的位
1&1 = 1 除此之外都是0
0011 & 0001 可以取出最后一位 
掩码  xxMask  一般用来按位与值的 宏定义 1<<0 1<<1

（2）将某一位置为0 将掩码取反&这个二进制 ～掩码 = 掩码取反
```

```
加两个感叹号 可以强转成具体的布尔值 !!(5) 
```

### 按位或
```
“或” 有一个1 就是1
（1）将某一位置为1 将掩码｜这个二进制 00000010
```

### 位域
```
 struct {
     char tall : 1:
     char rich : 1;
     char handsome : 1;
 }_tallRichHandsome
 代表占一位
```

### lldb调试器
``` 
p/x 变量 

打印内存地址 p/x &(变量 )   address 

x address 查看内容
```

**布尔类型 8位**
```
把1位的值 扩展8位 则其他位会变成1 
2位的值 扩展8位 则正常 拉伸操作
```
### 判断视频方向
```
static func orientationFromTransform(_ transform: CGAffineTransform)
    -> (orientation: UIImage.Orientation, isPortrait: Bool) {
        var assetOrientation = UIImage.Orientation.up
        var isPortrait = false
        if transform.a == 0 && transform.b == 1.0 && transform.c == -1.0 && transform.d == 0 {
            assetOrientation = .right
            isPortrait = true
        } else if transform.a == 0 && transform.b == 1.0 && transform.c == 1.0 && transform.d == 0 {
            assetOrientation = .rightMirrored
            isPortrait = true
        } else if transform.a == 0 && transform.b == -1.0 && transform.c == 1.0 && transform.d == 0 {
            assetOrientation = .left
            isPortrait = true
        } else if transform.a == 0 && transform.b == -1.0 && transform.c == -1.0 && transform.d == 0 {
            assetOrientation = .leftMirrored
            isPortrait = true
        } else if transform.a == 1.0 && transform.b == 0 && transform.c == 0 && transform.d == 1.0 {
            assetOrientation = .up
        } else if transform.a == -1.0 && transform.b == 0 && transform.c == 0 && transform.d == -1.0 {
            assetOrientation = .down
        }
        return (assetOrientation, isPortrait)
}
```

0xFF 无符号 255 有符号-1

### 共用体
union {
    char wrb;
}_wrb

有些枚举 ｜ ｜ ｜  当枚举值很特殊 是2的整数次幂 则可以使用 + 但是不推荐用 

### 静态库和动态库
静态库：链接时候完整的拷贝到可执行文件，多次使用多次拷贝，造成冗余，使包的体积变大
动态库：链接时不复制，程序的运行时，由系统加载到内存中，供系统调用，省内存，和其他应用共用
**iOS的静态库？**
.a和.framework 样式
**OS的动态库？**
.dylib和.framework
**为什么framework既是静态又是动态？**
系统的framework是动态的，我们自己创建的是静态的。
**.a 和 .framework 的区别是什么？**
.a 是单纯的二进制文件，.framework是二进制问价+资源文件。
其中.a 不能直接使用，需要 .h文件配合，而.framework则可以直接使用。
.framework = .a + .h + sorrceFile(资源文件)

### CMTime
虽然很多通用的开发环境使用双精度类型无法应用于更多的高级时基媒体的开发中。比如，一个单一舍入错误就会导致丢帧或音频丢失。于是苹果在Core Media框架中定义了CMTime数据类型作为时间的格式
```
typedef struct
   
     CMTimeValue    value;      
     CMTimeScale    timescale;  
     CMTimeFlags    flags;      
     CMTimeEpoch    epoch;      
   } CMTime;
```
CMTime计算
CMTime t4 =CMTimeAdd(t1, t2); 相加
CMTime t5 =CMTimeSubtract(t3, t1); 相减

**CMTimeMake(a,b) a当前第几帧, b每秒钟多少帧，当前播放时间a/b。 CMTimeMakeWithSeconds(a,b) a当前时间,b每秒钟多少帧。**

### CMTimeRange
Core Media框架还为时间范围提供了一个数据类型，称为CMTimeRange
```
typedef struct
{
    CMTime          start;      
    CMTime          duration;   
} CMTimeRange;
```
创建如下：
```
 CMTimeRange timeRange1 = CMTimeRangeMake(t1, t2);
 CMTimeRange timeRange2 = CMTimeRangeFromTimeToTime(t4, t3);
 ```

 ### CMTimeRange的交集和并集
 ```
   //0-5s
    CMTimeRange range1 =CMTimeRangeMake(kCMTimeZero, CMTimeMake(5,1));
   //2-5s
    CMTimeRange range2 =CMTimeRangeMake(CMTimeMake(2, 1), CMTimeMake(5,1));
    //交叉时间范围 2-5s
    CMTimeRange intersectionRange =CMTimeRangeGetIntersection(range1, range2);
    CMTimeRangeShow(intersectionRange);
    
    //总和时间范围 7s
    CMTimeRange unionRange =CMTimeRangeGetUnion(range1, range2);
    CMTimeRangeShow(unionRange);
```
### 析构函数
C++的析构函数 == OC的 dealloc 释放 更快

### 有符号取值
5bits 0x00010000  第五位是符号为 为1的时候是负数  然后先取反码 再取 补码 （原码+1）

### Git版本控制忽略部分文件方法(.gitignore)
https://github.com/github/gitignore
vim 文件
```
            *.a       # 忽略所有 .a 结尾的文件            
            !lib.a    # 但 lib.a 除外            
            /TODO     # 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO           
            build/    # 忽略 build/ 目录下的所有文件            
            doc/*.txt # 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
```
**将.gitignore文件提交到仓库**
```
git add .gitignore
git commit -m "添加项目忽略文件"
git push
```

### 注释
https://www.cxyzjd.com/article/qq_14920635/89676810

### Swift Result<Success,Failure>
https://juejin.cn/post/6844903875971907598

### CABasicAnimation动画及其keypath值和作用
https://www.cnblogs.com/liuluoxing/p/5765089.html
