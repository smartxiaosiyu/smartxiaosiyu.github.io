---
title: 'iOS Block'
date: 2021-09-02 19:09:23
tags: [iOS]
published: true
hideInList: false
feature: 
isTop: false
---
### Block的数据结构
![](https://smartxiaosiyu.github.io/post-images/1630582433987.png)
> +  block本质上也是一个OC对象，它内部也有个isa指针
> + block是封装了函数调用以及函数调用环境的OC对象

### Block的变量捕获

为了保证block内部能够正常访问外部的变量，block有个变量捕获机制

|  变量类型   | 是否捕获到block内部  | 访问方式|
|  :----:  | :----:  | :----:  |
| 局部变量：auto  离开作用域就会销毁 | ⭕️ |值传递 |
| 局部变量：static  | ⭕️ |指针传递（地址传递） |
| 全局变量  | ❌ |直接访问 |

`当问会不会捕获，只要分析清楚是局部变量还是全局变量 局部变量就会捕获 全局变量不用捕获`
`OC里所有方法 前面两个参数都是调用者本身`*self* `调用者本身的函数名`*_cmd*
`如果block访问属性 （name），其实就是访问 self.name，self是局部变量，所以会捕获`


### Block类型
block有3种类型，可以通过调用class方法或者isa指针查看具体类型，最终都是继承自NSBlock类型
> + GlobalBlock
> + StackBlock
> + MallocBlock
![](https://smartxiaosiyu.github.io/post-images/1630583404563.png)
text 代码段 内存地址比较小
data段 存储着全局变量
堆段 一般放alloc出来的内存 动态分配内存 需要自己申请、清理内存（管理内存）
栈段 存放局部变量 系统会自动分配内存 函数调用完毕 栈里的数据可能是垃圾数据 内存地址比较大

|  block类型   | 环境|
|  :----:  | :----:  | 
| GlobalBlock | 不访问auto变量 |
| StackBlock | 访问了auto变量 |
| MallocBlock  | stackBlock调用copy |
 
![](https://smartxiaosiyu.github.io/post-images/1630584618950.png)
> 如上图所示 由于访问了auto变量 所以block是一个stack类型 test函数调用完毕 栈内存里的数据可能就被销毁 变成垃圾数据 所以在访问 block里的age变量 获取不到真正的值 只需要copy block  就是MallocBlock 存放到堆上 就可以正确使用

每一种类型的block调用copy后的结果如下所示
![](https://smartxiaosiyu.github.io/post-images/1630584991343.png)

### Block的copy
**在ARC的情况下 会自动将栈上的block放到堆上去**
 + block作为函数返回值时
 + 将block赋值给__strong指针时
 + _block作为Cocoa API中方法名含有usingBlock的方法参数时
 + block作为GCD API的方法参数时
  
> 函数指针 保存的是函数地址

### 当block内部访问了对象类型的auto变量时
 + **如果block是在栈上，将不会对auto变量产生强引用**
 + **如果Block被拷贝到堆上**
    +  会调用block内部的copy函数
    +  copy函数会调用_block_object_assign函数
    +  函数会根据auto的变量的修饰符做出相应的操作 类似于retain （形成强引用弱引用）
 + ** 如果block从堆上移除**
    + 会调用block内部的dispose函数
    + dispose函数内部会调用_Block_object_dispose函数
    + _Block_object_dispose函数会自动释放引用的auto变量（release） 当然auto变量的对象到底释不释放 取决于引用计数是否为0

一个对象什么时候释放 就看bloc访问对象的强引用什么时候销毁 释放掉

### 修改block内部变量__block
 + 变量是全局变量或者static修饰的变量
 + 变量是auto修饰的变量 __block 修饰符修饰 
 + __block可以用于解决block内部无法修改auto变量值的问题
 + __block不能修饰全局变量、静态变量（static）
 + 编译器会将__block变量包装成一个对象


![](https://smartxiaosiyu.github.io/post-images/1630980710798.png)
![](https://smartxiaosiyu.github.io/post-images/1630980719497.png)
![](https://smartxiaosiyu.github.io/post-images/1630980726397.png)
![](https://smartxiaosiyu.github.io/post-images/1630980733270.png)

block内部修改age 实则是修改__block_byref_age_0.forwading.age 的值

```
NSMutableArray *array = [NSMutableArray array]
XSYBlock block = {
    [array addOIbject:@(1)];
}
是可以的 无需__block 因为 这个是在使用指针 不是赋值（修改变量里的值）
```
### __block的内存管理
 + 当block在栈上时，并不会对__block变量产生强引用
 + 当block被copy到堆时
   + 会调用block内部的copy函数
   + copy函数内部会调用_Block_object_assign函数
   + _Block_object_assign函数会对__block变量形成**强引用（retain）**

![](https://smartxiaosiyu.github.io/post-images/1630996361399.png)

+ 当block从堆中移除时
   + 会调用block内部的dispose函数
   + dispose函数内部会调用_Block_object_dispose函数
   + _Block_object_dispose函数会自动释放引用的__block变量（release）
![](https://smartxiaosiyu.github.io/post-images/1630996445157.png)

### forwarding指针
![](https://smartxiaosiyu.github.io/post-images/1631006099981.png)
目的就是 无论谁堆上的age 还是栈上的age 通过forwading指向堆上的age 

>  int a = 0 局部变量存在栈上的  p/x &a  打印a的内存地址
  
MRC 环境下  __block 修饰 copy函数内部会调用_Block_object_assign函数 但是不会retain 所以不存在强引用  （视频083 14min）

> 修饰block 是strong或者copy都会拷贝到堆上 但是最好copy 因为MRC ARC统一

### 循环引用
**ARC**
+ __weak：不会产生强引用，指向的对象销毁时，会自动让指针置为nil
+  __unsafe_unretained：不会产生强引用，不安全，指向的对象销毁时，指针存储的地址值不变
+  __block
  ![](https://smartxiaosiyu.github.io/post-images/1631102517377.png)
**MRC**
+  __unsafe_unretained
+  __block 不会产生强引用
  ![](https://smartxiaosiyu.github.io/post-images/1631103110274.png)