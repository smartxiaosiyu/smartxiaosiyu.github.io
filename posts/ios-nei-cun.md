---
title: 'iOS 内存'
date: 2021-12-15 17:37:42
tags: [iOS]
published: true
hideInList: false
feature: 
isTop: false
---
### 内存布局
![](https://smartxiaosiyu.github.io/post-images/1639624670540.png)
代码段：编译之后的代码
数据段： 字符串常量  已初始化数据：已初始化的全局变量、静态变量  未初始化数据：未初始化的全局变
量、静态变量
堆：alloc malloc动态分配的内存 分配内存的空间地址越来越大
栈：函数调用开销，局部变量。分配的内存空间地址越来越小
堆空间越来越大 栈空间越来越小

### Tagged Point
+ 从64位开始，iOS引入了Tagged Point技术，用于优化NSNumber、NSDate、NSString等小对象的存储
+ 在没有使用Tagged Point之前 NSNumber、NSDate、NSString他们是对象 需要动态分配内存，维护引用技术等 NSNumber指针指向的是堆中NSNumber对象的地址值 指针8个字节 对象16个字节 一共24个字节
+ 使用 Tagged Point之后，NSNumber指针里存储的数据变成了：Tag+Data 数据直接存在指针中
+ 但是当指针不够存储数据的时候 才会使用动态内存的方式来存储数据
+ obj_mgsend 可以识别出来如果是Tagged Point，直接从指针中取出值 不需要找到isa指向的类对象 再从类对象找方法 节省了内存开销 查找开销
+ 如何判断一个指针是否为Tagged Pointer？
   iOS平台，最高有效位是1（第64bit）
   Mac平台，最低有效位是1
![](https://smartxiaosiyu.github.io/post-images/1639636465743.png)

![](https://smartxiaosiyu.github.io/post-images/1639735258084.png)


### Copy
拷贝的目的 产生一个新副本对象 修改了源对象 不会影响副本对象 反之亦然
copy 就相当于 retain操作
copy 不可变拷贝 产生不可变副本 浅拷贝 指针拷贝 计数器+1 的操作 没产生新对象
mutableCopy 可变拷贝 产生可变副本  深拷贝 内容拷贝 产生新对象

![](https://smartxiaosiyu.github.io/post-images/1640347715615.png)

![](https://smartxiaosiyu.github.io/post-images/1640416746636.jpeg)
![](https://smartxiaosiyu.github.io/post-images/1640416752678.jpeg)

属性不可以是mutableCopy
对象copy必须遵循Copying协议 是一个新的对象

### 引用计数存储
![](https://smartxiaosiyu.github.io/post-images/1640503801508.png)
![](https://smartxiaosiyu.github.io/post-images/1640503821772.png)
如果extra_rc 存不下引用计数的值 has_sidetable_rc变成1 就会 存储在SideTable类中
![](https://smartxiaosiyu.github.io/post-images/1640693158136.png)
获取 引用计数值
![](https://smartxiaosiyu.github.io/post-images/1640504815778.png)
![](https://smartxiaosiyu.github.io/post-images/1640504823834.png)

retain
![](https://smartxiaosiyu.github.io/post-images/1640504834582.png)
release
![](https://smartxiaosiyu.github.io/post-images/1640504842986.png)

### weak指针
weak指针指向的对象如果释放了，则这个指针会自动释放
将弱引用存到哈希里 当对象释放的时候 清空weak指针

### ARC
LLVM和runtime系统相互协作的一个结果
利用LLVM编译器帮我们自动release
弱引用是利用runtime

### Dealloc
![](https://smartxiaosiyu.github.io/post-images/1640693014995.png)
![](https://smartxiaosiyu.github.io/post-images/1640693452325.png)
![](https://smartxiaosiyu.github.io/post-images/1640693461628.png)
![](https://smartxiaosiyu.github.io/post-images/1640693467045.png)

### AutoReleasePool
![](https://smartxiaosiyu.github.io/post-images/1641282087999.png)
![](https://smartxiaosiyu.github.io/post-images/1641282100148.png)

每个AutoreleasePoolPage对象占用4096字节内存，除了用来存放它内部的成员变量，剩下的空间用来存放autorelease对象的地址
所有的AutoreleasePoolPage对象通过双向链表的形式连接在一起
![](https://smartxiaosiyu.github.io/post-images/1641282280295.png)
调用push方法会将一个POOL_BOUNDARY入栈，并且返回其存放的内存地址

调用pop方法时传入一个POOL_BOUNDARY的内存地址，会从最后一个入栈的对象开始发送release消息，直到遇到这个POOL_BOUNDARY

id *next指向了下一个能存放autorelease对象地址的区域  

### Runloop和Autorelease
+ autorelease对象在什么时机会被调用release？？
iOS在主线程的Runloop中注册了2个Observer
第1个Observer监听了kCFRunLoopEntry事件，会调用objc_autoreleasePoolPush()
第2个Observer
监听了kCFRunLoopBeforeWaiting事件，会调用objc_autoreleasePoolPop()、objc_autoreleasePoolPush()
监听了kCFRunLoopBeforeExit事件，会调用objc_autoreleasePoolPop()
+ 方法里有局部对象， 出了方法后会立即释放吗
  MRC的话 RunloopkCFRunLoopBeforeWaiting或者kCFRunLoopBeforeExit 时候释放
  ARC的话 默认函数内就释放 因为函数结束前调用release


