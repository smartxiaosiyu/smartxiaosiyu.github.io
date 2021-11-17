---
title: 'OpenGl ES 缩放 灵魂出窍滤镜'
date: 2021-09-11 21:58:09
tags: [OpenGL]
published: true
hideInList: false
feature: 
isTop: false
---
### 缩放滤镜
修改顶点坐标和纹理坐标的映射关系
放大过程顶点着色器完成
```
attribute vec4 Position;
attribute vec2 TextureCoords;
varying vec2 TextureCoordsVarying;

uniform float Time;

const float PI = 3.1415926;

void main (void) {
    float duration = 0.6;
    float maxAmplitude = 0.3;
    
    float time = mod(Time, duration);
    float amplitude = 1.0 + maxAmplitude * abs(sin(time * (PI / duration)));
    
    gl_Position = vec4(Position.x * amplitude, Position.y * amplitude, Position.zw);
    TextureCoordsVarying = TextureCoords;
}
```

### 灵魂出窍滤镜
```
precision highp float;

uniform sampler2D Texture;
varying vec2 TextureCoordsVarying;

uniform float Time;

void main (void) {
    float duration = 0.7;
    float maxAlpha = 0.4;
    float maxScale = 1.8;
    
    float progress = mod(Time, duration) / duration; // 0~1
    float alpha = maxAlpha * (1.0 - progress);
    float scale = 1.0 + (maxScale - 1.0) * progress;
    
    float weakX = 0.5 + (TextureCoordsVarying.x - 0.5) / scale;
    float weakY = 0.5 + (TextureCoordsVarying.y - 0.5) / scale;
    vec2 weakTextureCoords = vec2(weakX, weakY);
    
    vec4 weakMask = texture2D(Texture, weakTextureCoords);
    
    vec4 mask = texture2D(Texture, TextureCoordsVarying);
    
    gl_FragColor = mask * (1.0 - alpha) + weakMask * alpha;
}
```