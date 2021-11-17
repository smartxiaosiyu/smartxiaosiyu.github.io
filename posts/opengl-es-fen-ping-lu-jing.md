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
