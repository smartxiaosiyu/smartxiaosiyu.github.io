---
title: 'OpenGL ES 分屏滤镜'
date: 2021-09-03 14:36:29
tags: [OpenGL]
published: true
hideInList: false
feature: 
isTop: false
---

## 原始图片渲染到屏幕上去

 <!-- + 1.创建上下文
    self.context = [[EAGLContext alloc] initWithAPI:kEAGLRenderingAPIOpenGLES2];
    [EAGLContext setCurrentContext:self.context];

 + 2.开辟顶点数组内存空间
  ```
   typedef struct {
    GLKVector3 positionCoord; // (X, Y, Z)
    GLKVector2 textureCoord; // (U, V)
    } SenceVertex;
    self.vertices = malloc(sizeof(SenceVertex) * 4);
    ```

 + 3.初始化顶点(0,1,2,3)的顶点坐标以及纹理坐标
   ```
   self.vertices[0] = (SenceVertex){{-1, 1, 0}, {0, 1}};
    self.vertices[1] = (SenceVertex){{-1, -1, 0}, {0, 0}};
    self.vertices[2] = (SenceVertex){{1, 1, 0}, {1, 1}};
    self.vertices[3] = (SenceVertex){{1, -1, 0}, {1, 0}};
 ```

 + 4.创建图层
    ```CAEAGLLayer *layer = [[CAEAGLLayer alloc] init];```

 + 5.绑定渲染缓存区
    渲染缓存区、帧缓存区
    
        1.渲染缓存区,帧缓存区对象
            GLuint renderBuffer;
            GLuint frameBuffer;
        
        2.获取帧渲染缓存区名称,绑定渲染缓存区以及将渲染缓存区与layer建立连接
        glGenRenderbuffers(1, &renderBuffer);
        glBindRenderbuffer(GL_RENDERBUFFER, renderBuffer);
     [self.context renderbufferStorage:GL_RENDERBUFFER fromDrawable:layer];
        
        3.获取帧缓存区名称,绑定帧缓存区以及将渲染缓存区附着到帧缓存区上
        glGenFramebuffers(1, &frameBuffer);
        glBindFramebuffer(GL_FRAMEBUFFER, frameBuffer);
        glFramebufferRenderbuffer(GL_FRAMEBUFFER,
                                 GL_COLOR_ATTACHMENT0,
                                `GL_RENDERBUFFER,
                                renderBuffer); -->
                                
 + 6.读取图片，把图片载入到纹理中去
    ```将图片解压缩成位图 载入到纹理去```

 + 7.设置视口
   ``` glViewport(0, 0, self.drawableWidth, self.drawableHeight);```
 + 8.设置顶点缓存区
   将我们的顶点数据拷贝到缓存中去

   ```
   GLuint vertexBuffer;
    glGenBuffers(1, &vertexBuffer);
    glBindBuffer(GL_ARRAY_BUFFER, vertexBuffer);
    GLsizeiptr bufferSizeBytes = sizeof(SenceVertex) * 4;
    glBufferData(GL_ARRAY_BUFFER, bufferSizeBytes, self.vertices, GL_STATIC_DRAW);
   ```
 + 9.设置默认着色器


### 片元着色器代码(关于分屏的着色器代码)
``` 
precision highp float;
uniform sampler2D Texture;
varying highp vec2 TextureCoordsVarying;

void main() {
    vec2 uv = TextureCoordsVarying.xy;//纹理坐标
    float y;
    if (uv.y >= 0.0 && uv.y <= 0.5) {
        y = uv.y + 0.25;
    } else {
        y = uv.y - 0.25;
    }
    gl_FragColor = texture2D(Texture, vec2(uv.x, y));//取纹素
} 
```

### 渲染
```
-(void)render{
    
    // 清除画布
    glClear(GL_COLOR_BUFFER_BIT);
    glClearColor(1, 1, 1, 1);
    
    //使用program
    glUseProgram(self.program);
    //绑定buffer
    glBindBuffer(GL_ARRAY_BUFFER, self.vertexBuffer);
    
    // 重绘
    glDrawArrays(GL_TRIANGLE_STRIP, 0, 4);
    //渲染到屏幕上
    [self.context presentRenderbuffer:GL_RENDERBUFFER];
    
}
```
