---
title: 'OpenGL ES 封装顶点数据和绘制代码'
date: 2021-09-01 15:00:10
tags: [OpenGL]
published: true
hideInList: false
feature: 
isTop: false
---
**VBO**
>顶点缓冲对象VBO是在显卡存储空间中开辟出的一块内存缓存区，用于存储顶点的各类属性信息，如顶点坐标，顶点法向量，顶点颜色数据等。 在渲染时，可以直接从VBO中取出顶点的各类属性数据，由于VBO在显存而不是在内存中，不需要从CPU传输数据，处理效率更高。

**创建一个顶点属性数组缓冲区**
>//创建缓存对象并返回缓存对象的标识符
`1.glGenBuffers(1, &name) `

>//将缓存对象对应到相应的缓存上
`2.glBindBuffer(GL_ARRAY_BUFFER,self.name); `

>//将数据拷贝到缓存对象
`3.glBufferData(`
   ` GL_ARRAY_BUFFER,  // Initialize buffer contents`
     `bufferSizeBytes, // Number of bytes to copy`
   ` dataPtr,          // Address of bytes to copy`
     `usage);           // Hint: cache in GPU memory`

**准备顶点绘制**
>//将缓存对象对应到相应的缓存上
`1.glBindBuffer(GL_ARRAY_BUFFER,self.name); `

>//启用指定属性  1.出于性能考虑，所有顶点着色器的属性 （Attribute）变量都是关闭的，意味着数据在着色器端是不可见的，哪怕数据已经上传到GPU，由glEnableVertexAttribArray启用指定属性，才可在顶点着色器中访问逐顶点的属性数据. 2.VBO只是建立CPU和GPU之间的逻辑连接，从而实现了CPU数据上传至GPU。但是，数据在GPU端是否可见，即，着色器能否读取到数据，由是否启用了对应的属性决定，这就是glEnableVertexAttribArray的功能，允许顶点着色器读取GPU（服务器端）数据。
`2.glEnableVertexAttribArray(属性); `

>//顶点数据传入GPU之后，还需要通知OpenGL如何解释这些顶点数据，这个工作由函数glVertexAttribPointer完成
`3.glVertexAttribPointer(`
                          `index,//参数指定顶点属性位置`
                          `count,//指定顶点属性大小`
                          `GL_FLOAT,//指定数据类型`
                          `GL_FALSE,//数据被标准化`
                          `(int)self.stride,//步长`
                          `NULL + offset);//偏移量 NULL+offset`


**绘制**
>//绘制
/*
     glDrawArrays (GLenum mode, GLint first, GLsizei count);提供绘制功能。当采用顶点数组方式绘制图形时，使用该函数。该函数根据顶点数组中的坐标数据和指定的模式，进行绘制。
     参数列表:
     mode，绘制方式，OpenGL2.0以后提供以下参数：GL_POINTS、GL_LINES、GL_LINE_LOOP、GL_LINE_STRIP、GL_TRIANGLES、GL_TRIANGLE_STRIP、GL_TRIANGLE_FAN。
     first，从数组缓存中的哪一位开始绘制，一般为0。
     count，数组中顶点的数量。
     */
`glDrawArrays(mode, first, count);`
