---
title: 'iOS 分类'
date: 2021-12-13 15:58:22
tags: [iOS]
published: true
hideInList: false
feature: 
isTop: false
---
编译出来的结果
![](https://smartxiaosiyu.github.io/post-images/1639382328780.png)

运行时
将 rw->methods 指向的数组扩容，将原来的方法列表移到最后，add分类到数组里，优先调用最后面编译的分类，再调用原来的方法 

### Category的加载处理过程
通过Runtime加载某个类的所有Category数据

把所有Category的方法、属性、协议数据，合并到一个大数组中
后面参与编译的Category数据，会在数组的前面

将合并后的分类数据（方法、属性、协议），插入到类原来数据的前面

### 类扩展和分类的区别
类扩展 编译的时候 就已经合并在类中了 无非将.h里公开声明里的东西变成.m里私有的
![](https://smartxiaosiyu.github.io/post-images/1639383526599.png)
分类是运行时机制 再将分类的东西合并到类里面

![](https://smartxiaosiyu.github.io/post-images/1639387605285.png)
![](https://smartxiaosiyu.github.io/post-images/1639387680149.png)

### memmove和memcpy
1234 -> 0124
memmove  直接可以
memcpy 变成0114