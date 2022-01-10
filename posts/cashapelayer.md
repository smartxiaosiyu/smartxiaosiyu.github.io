---
title: 'CAShapeLayer'
date: 2021-11-24 15:39:49
tags: [UI]
published: true
hideInList: false
feature: 
isTop: false
---

关于CAShapeLayer和DrawRect的比较

DrawRect：DrawRect属于CoreGraphic框架，占用CPU，消耗性能大
CAShapeLayer：CAShapeLayer属于CoreAnimation框架，通过GPU来渲染图形，节省性能。动画渲染直接提交给手机GPU，不消耗内存

### 布局
要分析CALayer的anchorPoint和position属性,首先要讨论一下CALayer的布局.
我们所熟悉的UIView有三个重要的布局属性:frame,bounds和center,CALayer对应的叫做 frame,bounds和position.

frame代表了图层的外部坐标(在父图层上占据的空间)
bounds为内部坐标
position代表了相对父图层anchorPoint的位置

### 锚点
+ 和position共同决定图层相对父图层的位置,即frame的x,y
+ 在图层旋转时的固定点
锚点使用单位坐标来描述,范围为左上角{0, 0}到右下角{1, 1},默认坐标是{0.5, 0.5}.

### 锚点和position的关系
position是图层的anchorPoint在父图层中的位置坐标.
anchorPoint和position共同决定图层相对父图层的位置,即frame属性的frame.origin.
单方面修改anchorPoint或者position并不会对彼此产生影响,修改其中一个值,受影响的只会是frame.origin.
anchorPoint如果是0.5，0。5 就相当于 中心点 那可以理解成 子view的中心点在父view的位置坐标

position.x =  frame.origin.x - anchorPoint.x * bounds.size.width；

position.y =  frame.origin.y - anchorPoint.y * bounds.size.height

### 总结
单方面修改anchorPoint或者position并不会对彼此产生影响,修改其中一个值,受影响的只会是frame.origin.

anchorPoint和position共同决定了frame
frame.origin.x = position.x - anchorPoint.x * bounds.size.width；
frame.origin.y = position.y - anchorPoint.y * bounds.size.height

anchorPoint是图层在旋转时的固定点

https://www.jianshu.com/p/998a6119a275

