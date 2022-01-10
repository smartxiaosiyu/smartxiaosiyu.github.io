---
title: 'OpenGl ES 将图片解压缩成位图 载入到纹理去'
date: 2021-09-03 17:06:20
tags: [OpenGL]
published: true
hideInList: false
feature: 
isTop: false
---
 ### 用CPU解压图片
 + 1、将 UIImage 转换为 CGImageRef
    `CGImageRef cgImageRef = [image CGImage];`
   //判断图片是否获取成功
   ` if (!cgImageRef) {`
       ` NSLog(@"Failed to load image");`
       ` exit(1);`
   ` }`
 + 2、读取图片的大小，宽和高
    `GLuint width = (GLuint)CGImageGetWidth(cgImageRef);`
   ` GLuint height = (GLuint)CGImageGetHeight(cgImageRef);`
    //获取图片的rect
    `CGRect rect = CGRectMake(0, 0, width, height);`
    //获取图片的颜色空间
   ` CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();`
+ 3、获取图片字节数 宽*高*4（RGBA）
   ` void *imageData = malloc(width * height * 4);`
+ 4、创建上下文
    /*
     参数1：data,指向要渲染的绘制图像的内存地址
     参数2：width,bitmap的宽度，单位为像素
     参数3：height,bitmap的高度，单位为像素
     参数4：bitPerComponent,内存中像素的每个组件的位数，比如32位RGBA，就设置为8
     参数5：bytesPerRow,bitmap的没一行的内存所占的比特数
     参数6：colorSpace,bitmap上使用的颜色空间  kCGImageAlphaPremultipliedLast：RGBA
     */
    `CGContextRef context = CGBitmapContextCreate(imageData, width, height, 8, width * 4, colorSpace, kCGImageAlphaPremultipliedLast | kCGBitmapByteOrder32Big);`
    
    //将图片翻转过来(图片默认是倒置的)
    `CGContextTranslateCTM(context, 0, height);`
    `CGContextScaleCTM(context, 1.0f, -1.0f);`
    `CGColorSpaceRelease(colorSpace);`
   ` CGContextClearRect(context, rect);`
    
    **//对图片进行重新绘制，得到一张新的解压缩后的位图**
   ` CGContextDrawImage(context, rect, cgImageRef);`

 ### 设置图片纹理属性
  + 5.、获取纹理ID
   ` GLuint textureID;`创建一个纹理
    `glGenTextures(1, &textureID);`获取纹理id
    `glBindTexture(GL_TEXTURE_2D, textureID);`绑定纹理
 + 6、载入纹理2D数据
    /*
     参数1：纹理模式，GL_TEXTURE_1D、GL_TEXTURE_2D、GL_TEXTURE_3D
     参数2：加载的层次，一般设置为0
     参数3：纹理的颜色值GL_RGBA
     参数4：宽
     参数5：高
     参数6：border，边界宽度
     参数7：format
     参数8：type
     参数9：纹理数据
     */
    `glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, imageData);`
+ 7、设置纹理属性
    `glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);`
`glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);`
    `glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);`
   ` glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);`
+ 8、绑定纹理
    /*
     参数1：纹理维度
     参数2：纹理ID,因为只有一个纹理，给0就可以了。
     */
    `glBindTexture(GL_TEXTURE_2D, 0);`
    
+ 9、释放context,imageData
    `CGContextRelease(context);`
    `free(imageData);`
    
 + 10、返回纹理ID
    `return textureID;`

