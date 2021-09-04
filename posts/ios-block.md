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