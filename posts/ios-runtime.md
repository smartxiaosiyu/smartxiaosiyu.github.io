---
title: 'iOS Runtime'
date: 2021-09-14 19:49:51
tags: [iOS]
published: false
hideInList: false
feature: 
isTop: false
---
 ###  instance对象
instance对象是通过类alloc出来的对象，每个instance对象是不同的对象，占据着两块不同的内存
![](https://smartxiaosiyu.github.io/post-images/1634715225272.png)

### Class对象
![](https://smartxiaosiyu.github.io/post-images/1634715332044.png)
![](https://smartxiaosiyu.github.io/post-images/1634715341334.png)
objectClass1 ~ objectClass5都是NSObject的class对象（类对象）
它们是同一个对象。**每个类在内存中有且只有一个class对象**

### 元类对象
![](https://smartxiaosiyu.github.io/post-images/1634715483854.png)
objectMetaClass是NSObject的meta-class对象（元类对象）
**每个类在内存中有且只有一个meta-class对象**
![](https://smartxiaosiyu.github.io/post-images/1634715522926.png)
查看MetaClass对象
![](https://smartxiaosiyu.github.io/post-images/1634715559338.png)
 + Objective-C是一门动态性比较强的编程语言，跟C、C++等语言有着很大的不同
 + Objective-C的动态性是由Runtime API来支撑的
 + Runtime API提供的接口基本都是C语言的，源码由C\C++\汇编语言编写
类对象、元类对象的内存地址 后三位都是0

![](https://smartxiaosiyu.github.io/post-images/1634715730857.png)
instance的isa指向class
当调用对象方法时，通过instance的isa找到class，最后找到对象方法的实现进行调用

class的isa指向meta-class
当调用类方法时，通过class的isa找到meta-class，最后找到类方法的实现进行调用

![](https://smartxiaosiyu.github.io/post-images/1634715840415.png)
```
instance的isa指向class

class的isa指向meta-class

meta-class的isa指向基类的meta-class

class的superclass指向父类的class
如果没有父类，superclass指针为nil

meta-class的superclass指向父类的meta-class
基类的meta-class的superclass指向基类的class

instance调用对象方法的轨迹
isa找到class，方法不存在，就通过superclass找父类

class调用类方法的轨迹
isa找meta-class，方法不存在，就通过superclass找父类
```

### isa
arm64前 实例对象的isa指向类对象地址 类对象指向元类对象
arm64后 isa使用共用体做了优化 将64位的数据 其中33位才用才存储内存地址  需要isa&一个掩码得到内存地址 
```
nonpointer
0，代表普通的指针，存储着Class、Meta-Class对象的内存地址
1，代表优化过，使用位域存储更多的信息

has_assoc
是否有设置过关联对象，如果没有，释放时会更快

has_cxx_dtor
是否有C++的析构函数（.cxx_destruct），如果没有，释放时会更快

shiftcls
存储着Class、Meta-Class对象的内存地址信息

magic
用于在调试时分辨对象是否未完成初始化

weakly_referenced
是否有被弱引用指向过，如果没有，释放时会更快

deallocating
对象是否正在释放

extra_rc
里面存储的值是引用计数器减1

has_sidetable_rc
引用计数器是否过大无法存储在isa中
如果为1，那么引用计数会存储在一个叫SideTable的类的属性中

```
### Class的结构
![](https://smartxiaosiyu.github.io/post-images/1634721542720.png)

class_rw_t里面的methods、properties、protocols是二维数组，是可读可写的，包含了类的初始内容、分类的内容

![](https://smartxiaosiyu.github.io/post-images/1634721677224.png)

### 销毁一个实例对象
![](https://smartxiaosiyu.github.io/post-images/1633693179449.png)

### method_t是对方法\函数的封装
![](https://smartxiaosiyu.github.io/post-images/1633953700358.png)

**SEL**
SEL代表方法\函数名，一般叫做选择器，底层结构跟char *类似
可以通过@selector()和sel_registerName()获得
可以通过sel_getName()和NSStringFromSelector()转成字符串
不同类中相同名字的方法，所对应的方法选择器是相同的
**IMP**
指向函数的指针，存储的就是函数的实现
**Types**
i 24 @ 0 ： 8 i16 f 20
返回Int类型 总共24个字节  @ 代表id （self）：代表SEL参数 下一个是Int参数 最后一个是Float参数
数字代表在哪个位置 占多少字节数

### 方法缓存
运用散列表存储 Key：SEL Value：IMP
用 Key & Mask 取出 在散列表里的索引 如果不是当前key 则索引-1 散裂碰撞
mask = 散列表长度-1
如果散列表长度不够用 则扩容 mask变化 清除缓存
无论哈希还是散列表 本质都是给一个值通过算法得到一个索引

### objc_msgSend执行流程
OC中的方法调用，其实都是转换为objc_msgSend函数的调用
**objc_msgSend的执行流程可以分为3大阶段**
+ 消息发送
![](https://smartxiaosiyu.github.io/post-images/1634721850187.png)

+ 动态方法解析
**如果当时没有这个方法就动态添创建一个方法**

消息发送 找不到test方法
```
+ (BOOL)resolveInstanceMethod:(SEL)sel
{
    if (sel == @selector(test)) {
        // 获取其他方法
        Method method = class_getInstanceMethod(self, @selector(other));

        // 动态添加test方法的实现
        class_addMethod(self, sel,
                        method_getImplementation(method),
                        method_getTypeEncoding(method));

        // 返回YES代表有动态添加方法
        return YES;
    }
    return [super resolveInstanceMethod:sel];
}
```
![](https://smartxiaosiyu.github.io/post-images/1635164774221.png)

+ 消息转发
  我自己处理不了，交给别人处理
  ![](https://smartxiaosiyu.github.io/post-images/1635409522559.png)
  ![](https://smartxiaosiyu.github.io/post-images/1635409577096.png)

  forwardInvocation哪怕不做什么都可以 不会崩溃 
  NSInvocation 包装了一个方法的调用

  forwardingTargetForSelector 
  类对象也有消息转发 而且 可以转成实例对象调用方法 因为 本质是 objc_msgSend（对象，@selector(test)）

### [super messsage]底层实现
  方法调用者也就是消息接受者
  1.消息接受者仍然是子类对象
  2.从父类开始查找方法的实现  

![](https://smartxiaosiyu.github.io/post-images/1635682771655.png)
![](https://smartxiaosiyu.github.io/post-images/1635682800680.png)

### 监控找不到的方法 可输出Log 
![](https://smartxiaosiyu.github.io/post-images/1635683410053.png)

### isKindOfClass isMemberOfClass
![](https://smartxiaosiyu.github.io/post-images/1636282412947.png)
![](https://smartxiaosiyu.github.io/post-images/1636282421258.jpg)

### 栈空间分布
![](https://smartxiaosiyu.github.io/post-images/1635854968046.png)

![](https://smartxiaosiyu.github.io/post-images/1636293981407.png)
![](https://smartxiaosiyu.github.io/post-images/1636293991588.JPG)