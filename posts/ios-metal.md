---
title: 'iOS Metal'
date: 2021-10-15 17:51:54
tags: [Metal]
published: false
hideInList: false
feature: 
isTop: false
---
OpenGL ES 可以模拟器执行 借助CPU来完成GPU的计算
Metal 5S以上设备才可以真机执行 至少A7处理器 必须真机执行 不支持模拟器

.Apple 建议: Respond to View Events
响应视图事件
- (void)mtkView:(nonnull MTKView *)view drawableSizeWillChange:(CGSiz
e)size;
- (void)drawInMTKView:(nonnull MTKView *)view;
- 
MTKViewDelegate 有两个方法
1. drawableSizeWillChange 
调用条件：窗口大小发生变化(Mac OS), ᯿重新布局(设备方向更改)时
2. drawInMTKView ; 渲染循环 60FPS

Apple 建议: Metal Command Objects
**//MTLDevice 对象表示GPU.**
 _view.device = MTLCreateSystemDefaultDevice();

**//所有应用程序需要与GPU交互的第一个对象时`MTLCommandQueue`对象**
**//MTLCommandQueue这个队列可以将我们的命令 按照正常顺序 一帧一帧发送到GPU中间 GPU执行的命令**
_commandQueue = [_device newCommandQueue];

**//ֵ使用MTLCommandQueue 创建象并且加入到MTCommandBuffer对象**
**//为当前渲染的每个渲染传递创建一个新的命令缓冲区܄**
 id<MTLCommandBuffer> commandBuffer = [_commandQueue commandBuffer];
 commandBuffer.label = @"MyCommand";

 ### Metal 命令对象之间的关系
+  命令缓存区(command buffer) 是从命令队列(command queue) 创建的
+  命令编码器(command encoders) 将命令编码到命令缓存区中
+  提交命令缓存区并将其发送到GPU 
+  GPU执⾏命令并将结果呈现为可绘制

### 渲染管线的三大阶段
![](https://smartxiaosiyu.github.io/post-images/1634816268639.png)

### Metal和OpenGL ES 对比
```
_view = (MTKView *)self.view;
_view.device = MTLCreateSystemDefaultDevice();
```

Metal
```
//1.获取GPU 设备
_device = mtkView.device;

//2.在项目中加载所有的(.metal)着色器文件
// 从bundle中获取.metal文件
id<MTLLibrary> defaultLibrary = [_device newDefaultLibrary];
//从库中加载顶点函数
id<MTLFunction> vertexFunction = [defaultLibrary newFunctionWithName:@"vertexShader"];
//从库中加载片元函数
id<MTLFunction> fragmentFunction = [defaultLibrary newFunctionWithName:@"fragmentShader"];
//3.配置用于创建管道状态的管道描述
MTLRenderPipelineDescriptor *pipelineStateDescriptor = [[MTLRenderPipelineDescriptor alloc] init];
//管道名称
pipelineStateDescriptor.label = @"Simple Pipeline";
//可编程函数,用于处理渲染过程中的各个顶点
pipelineStateDescriptor.vertexFunction = vertexFunction;
//可编程函数,用于处理渲染过程中各个片段/片元
pipelineStateDescriptor.fragmentFunction = fragmentFunction;
//一组存储颜色数据的组件
pipelineStateDescriptor.colorAttachments[0].pixelFormat = mtkView.colorPixelFormat;
//4.同步创建并返回渲染管线状态对象
_pipelineState = [_device newRenderPipelineStateWithDescriptor:pipelineStateDescriptor error:&error];
```

OpenGL ES
```
//1. 编译顶点着色器/片元着色器
    GLuint vertexShader = [self compileShaderWithName:shaderName type:GL_VERTEX_SHADER];
    GLuint fragmentShader = [self compileShaderWithName:shaderName type:GL_FRAGMENT_SHADER];
    
//2. 将顶点/片元附着到program
GLuint program = glCreateProgram();
glAttachShader(program, vertexShader);
glAttachShader(program, fragmentShader);
    
//3.linkProgram
glLinkProgram(program);
//2. use Program
glUseProgram(program);
```
```
//5.创建命令队列
_commandQueue = [_device newCommandQueue];
```

Metal
//每当视图需要渲染帧时调用
```
   //2.为当前渲染的每个渲染传递创建一个新的命令缓冲区
   //缓冲区会有好多好多队列
    id<MTLCommandBuffer> commandBuffer = [_commandQueue commandBuffer];
    //指定缓存区名称
    commandBuffer.label = @"MyCommand";
    
    //3.
    // MTLRenderPassDescriptor:一组渲染目标，用作渲染通道生成的像素的输出目标。渲染描述符
    MTLRenderPassDescriptor *renderPassDescriptor = view.currentRenderPassDescriptor;
    //判断渲染目标是否为空
    if(renderPassDescriptor != nil)
    {
        //4.创建渲染命令编码器,这样我们才可以渲染到something
        id<MTLRenderCommandEncoder> renderEncoder =[commandBuffer renderCommandEncoderWithDescriptor:renderPassDescriptor];
        //渲染器名称
        renderEncoder.label = @"MyRenderEncoder";

        //5.设置我们绘制的可绘制区域
        /*
        typedef struct {
            double originX, originY, width, height, znear, zfar;
        } MTLViewport;
         */
        //视口指定Metal渲染内容的drawable区域。 视口是具有x和y偏移，宽度和高度以及近和远平面的3D区域
        //为管道分配自定义视口需要通过调用setViewport：方法将MTLViewport结构编码为渲染命令编码器。 如果未指定视口，Metal会设置一个默认视口，其大小与用于创建渲染命令编码器的drawable相同。
        MTLViewport viewPort = {
            0.0,0.0,_viewportSize.x,_viewportSize.y,-1.0,1.0
        };
        [renderEncoder setViewport:viewPort];
        //[renderEncoder setViewport:(MTLViewport){0.0, 0.0, _viewportSize.x, _viewportSize.y, -1.0, 1.0 }];
        
        //6.设置当前渲染管道状态对象
        [renderEncoder setRenderPipelineState:_pipelineState]
    }
    ```

    ```
        //7.从应用程序OC 代码 中发送数据给Metal 顶点着色器 函数
        //顶点数据+颜色数据
        //   1) 指向要传递给着色器的内存的指针
        //   2) 我们想要传递的数据的内存大小
        //   3)一个整数索引，它对应于我们的“vertexShader”函数中的缓冲区属性限定符的索引。

        [renderEncoder setVertexBytes:triangleVertices
                               length:sizeof(triangleVertices)
                              atIndex:CCVertexInputIndexVertices];

        //viewPortSize 数据
        //1) 发送到顶点着色函数中,视图大小
        //2) 视图大小内存空间大小
        //3) 对应的索引
        [renderEncoder setVertexBytes:&_viewportSize
                               length:sizeof(_viewportSize)
                              atIndex:CCVertexInputIndexViewportSize];

       
        
        //8.画出三角形的3个顶点
        // @method drawPrimitives:vertexStart:vertexCount:
        //@brief 在不使用索引列表的情况下,绘制图元
        //@param 绘制图形组装的基元类型
        //@param 从哪个位置数据开始绘制,一般为0
        //@param 每个图元的顶点个数,绘制的图型顶点数量
        /*
         MTLPrimitiveTypePoint = 0, 点
         MTLPrimitiveTypeLine = 1, 线段
         MTLPrimitiveTypeLineStrip = 2, 线环
         MTLPrimitiveTypeTriangle = 3,  三角形
         MTLPrimitiveTypeTriangleStrip = 4, 三角型扇
         */
        [renderEncoder drawPrimitives:MTLPrimitiveTypeTriangle
                          vertexStart:0
                          vertexCount:3];

        //9.表示已该编码器生成的命令都已完成,并且从NTLCommandBuffer中分离
        [renderEncoder endEncoding];

        //10.一旦框架缓冲区完成，使用当前可绘制的进度表
        [commandBuffer presentDrawable:view.currentDrawable];
    ```

    OpenGL ES
    ```
    //3. 获取Position,Texture,TextureCoords 的索引位置
    GLuint positionSlot = glGetAttribLocation(program, "Position");
    GLuint textureSlot = glGetUniformLocation(program, "Texture");
    GLuint textureCoordsSlot = glGetAttribLocation(program, "TextureCoords");
    
    //4.激活纹理,绑定纹理ID
    glActiveTexture(GL_TEXTURE0);
    glBindTexture(GL_TEXTURE_2D, self.textureID);
    
    //5.纹理sample
    glUniform1i(textureSlot, 0);
    
    //6.打开positionSlot 属性并且传递数据到positionSlot中(顶点坐标)
    glEnableVertexAttribArray(positionSlot);
    glVertexAttribPointer(positionSlot, 3, GL_FLOAT, GL_FALSE, sizeof(SenceVertex), NULL + offsetof(SenceVertex, positionCoord));
    
    //7.打开textureCoordsSlot 属性并传递数据到textureCoordsSlot(纹理坐标)
    glEnableVertexAttribArray(textureCoordsSlot);
    glVertexAttribPointer(textureCoordsSlot, 2, GL_FLOAT, GL_FALSE, sizeof(SenceVertex), NULL + offsetof(SenceVertex, textureCoord));
```
Metal
```
//11.最后,在这里完成渲染并将命令缓冲区推送到GPU
 [commandBuffer commit];
```
OpenGL ES
```
    // 重绘
    glDrawArrays(GL_TRIANGLE_STRIP, 0, 4);
    //渲染到屏幕上
    [self.context presentRenderbuffer:GL_RENDERBUFFER];
```

### 函数修饰符
Kernel 并行计算函数 让Metal帮助高效计算 返回值必须是void
Vertex 顶点函数
Fragment 片元函数
### 变量/函数参数地址修饰符
device 描述在设备中开辟一段空间存储变量
threadground  描述在线程组中开辟一段空间存储变量 该线程组都可以访问该变量
constant 不可变常量修饰符 在设备中开辟一段空间存储变量
thread 在该线程中间只能被该线程调用
