---
title: 'OpenGl ES 灰度滤镜'
date: 2021-09-07 19:20:38
tags: []
published: true
hideInList: false
feature: 
isTop: false
---
### 灰度滤镜
+ 1.浮点算法：Gray=R*0.3+G*0.59+B*0.11
+ 2.整数⽅法：Gray=(R*30+G*59+B*11)/100
+ 3.移位⽅法：Gray =(R*76+G*151+B*28)>>8;
+ 4.平均值法：Gray=(R+G+B)/3; 
+ 5.仅取绿⾊：Gray=G；    
前三个是权值法

**片元着色器**
```
precision highp float;
uniform sampler2D Texture;
varying vec2 TextureCoordsVarying;
const highp vec3 W = vec3(0.2125, 0.7154, 0.0721);

void main (void) {
    
    vec4 mask = texture2D(Texture, TextureCoordsVarying);
    float luminance = dot(mask.rgb, W);
    gl_FragColor = vec4(vec3(luminance), 1.0);
}
```
