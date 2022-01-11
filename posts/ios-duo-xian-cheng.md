---
title: 'iOS 多线程'
date: 2021-10-22 11:46:39
tags: []
published: true
hideInList: false
feature: 
isTop: false
---
同步和异步主要影响：能不能开启新的线程
同步：在当前线程中执行任务，不具备开启新线程的能力
异步：在新的线程中执行任务，具备开启新线程的能力

并发和串行主要影响：任务的执行方式
并发：多个任务并发（同时）执行
串行：一个任务执行完毕后，再执行下一个任务


```
// 题目：写出NSLog的打印结果（来自美团 GCD 面试题）
__block int a = 0;
while (a < 5) {
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        a++;
    });
}
NSLog(@"输出: %d", a);

// 答案：
// 输出结果为：a >= 5
// 原因：while循环内部执行并发耗时任务，ARC环境下，一旦Block赋值就会触发copy，__block就会copy到堆上，所以当执行a++时，Block外部也能访问到改变后的值；当a不满足循环条件而跳出时，并发任务可能仍在只执行，此时仍然会改变a的值，鉴于不同机器的CPU和线程差异影响，所以最终输出结果会大于等于5
// 注意：a的输出值结果是a>=5，但是a的实际结果会远远大于5（NSLog输出完成后，并发耗时任务可能尚未完全结束）
```

### GCD（Grand Central Dispatch）
Grand Central Dispatch (GCD) 是 Apple 开发的一个多核编程的解决方法，基本概念就是**dispatch queue（调度队列）**，queue 是一个对象，它可以接受任务，并将任务以先到先执行的顺序来执行。dispatch queue 可以是并发的或串行的。GCD 的底层依然是用线程实现，不过我们可以不用关注实现的细节。GCD在后端管理着一个线程池，它不仅决定着你的代码块将在哪个线程被执行，还根据可用的系统资源对这些线程进行管理。这样通过GCD来管理线程，从而解决线程被创建的问题。

GCD 可用于多核的并行运算
GCD 会自动利用更多的CPU内核（比如双核、四核）
GCD 会自动管理线程的生命周期（创建线程、调度任务、销毁线程）
任务都是以block 的方式提交到对列上，然后 GCD 会自动的创建线程池去执行这些任务

### DispatchQueue
一个对象，用于在应用程序的主线程或后台线程上串行或并发地管理任务的执行。
DispatchQueue 是一个类似线程的概念，这里称作对列队列是一个 FIFO(先进先出，后进后出) 数据结构，意味着先提交到队列的任务会先开始执行。DispatchQueue 背后是一个由系统管理的线程池。
```
同步和异步的区别
    同步（`sync`）：只能在当前线程中执行任务，不具备开启新线程的能力
    异步 (`async`)：可以在新的线程中执行任务，具备开启新线程的能力
```
### DispatchQoS (quality of service) 服务质量
适用于任务的服务质量或执行优先级。

优先级由最低的 background 到最高的 userInteractive 共五个，还有一个为定义的 unspecified.

+ background：最低优先级，等同于 DISPATCH_QUEUE_PRIORITY_BACKGROUND. 用户不可见，比如：在后台存储大量数据
+ utility：优先级等同于 DISPATCH_QUEUE_PRIORITY_LOW，可以执行很长时间，再通知用户结果。比如：下载一个大文件，网络，计算
+ default：默认优先级,优先级等同于 DISPATCH_QUEUE_PRIORITY_DEFAULT，建议大多数情况下使用默认优先级
+ userInitiated：优先级等同于 DISPATCH_QUEUE_PRIORITY_HIGH,需要立刻的结果
+ userInteractive：用户交互相关，为了好的用户体验，任务需要立马执行。使用该优先级用于 UI 更新，事件处理和小工作量任务，在主线程执行
Qos指定了列队工作的优先级，系统会根据优先级来调度工作，越高的优先级能够越快被执行，但是也会消耗功能，所以准确的指定优先级能够保证app有效的使用资源。+ 

### DispatchWorkItem
想要执行的工作以某种方式进行封装，使您可以附加完成句柄或执行依赖项。通俗的说就是 DispatchWorkItem 把任务封装成一个对象。
```
let item = DispatchWorkItem {
        // 任务
    }
    DispatchQueue.global().async(execute: item)
```
### DispatchGroup
```
func enterLeaveGroup() {
        let group = DispatchGroup()
        let queue = DispatchQueue.global()
       
        // 把该任务添加到组队列中执行
        group.enter()
        queue.async(group: group, qos: .default, flags: []) {
            // 增加耗时
            DispatchQueue.main.asyncAfter(deadline: .now() + 1, execute: {
                 for _ in 0...4 {
                     print("(Thread.current) 耗时任务一")
                 }
                // 执行完之后从组队列中移除
                group.leave()
            })
           
        }
       
        // 把该任务添加到组队列中执行
        group.enter()
        queue.async(group: group, qos: .default, flags: []) {
            for _ in 0...4 {
                print("(Thread.current) 耗时任务二")
            }
            // 执行完之后从组队列中移除
            group.leave()
        }
       
        // 当上面所有的任务执行完之后通知
        group.notify(queue: queue) {
            print("(Thread.current) 所有的任务执行完了")
        }
    }
   
    /* 输出结果
<NSThread: 0x6000013dd980>{number = 6, name = (null)} 耗时任务二
<NSThread: 0x6000013dd980>{number = 6, name = (null)} 耗时任务二
<NSThread: 0x6000013dd980>{number = 6, name = (null)} 耗时任务二
<NSThread: 0x6000013dd980>{number = 6, name = (null)} 耗时任务二
<NSThread: 0x6000013dd980>{number = 6, name = (null)} 耗时任务二
<NSThread: 0x6000013862c0>{number = 1, name = main} 耗时任务一
<NSThread: 0x6000013862c0>{number = 1, name = main} 耗时任务一
<NSThread: 0x6000013862c0>{number = 1, name = main} 耗时任务一
<NSThread: 0x6000013862c0>{number = 1, name = main} 耗时任务一
<NSThread: 0x6000013862c0>{number = 1, name = main} 耗时任务一
<NSThread: 0x6000013ea2c0>{number = 7, name = (null)} 所有的任务执行完了
    */
```
### Suspend / Resume
Suspend 可以挂起一个线程，即暂停线程，但是仍然占用资源，只是不执行

Resume 恢复线程，即继续执行挂起的线程。

### GCD与NSOpration的区别
设置依赖关系
设置监听进度
设置优先级
还能继承
可以取消准备执行的任务
比GCD会带来一点额外的系统开销
比GCD更简单易用、代码可读性也更高

### NSOperation
NSOperation是一个和任务相关的抽象类，不具备封装操作的能力，必须使用其子类。

NSOperation⼦类的方式有3种：

系统实现的具体子类：NSInvocationOperation
系统实现的具体子类：NSBlockOperation
自定义子类，实现内部相应的⽅法。该类是线程安全的，不必管理线程生命周期和同步等问题。


开启操作有二种方式，一是通过start方法直接启动操作，该操作默认同步执行，二是将操作添加到NSOperationQueue中，然后由系统从队列中获取操作然后添加到一个新线程中执行，这些操作默认并发执行。

具体实现
方式一：直接由NSOperation子类对象启动。 首先将需要执行的操作封装到NSOperation子类对象中，然后该对象调用Start方法。
方式二：当添加到NSOperationQueue对象中，由该队列对象启动操作。
+ 1.将需要执行的操作封装到NSOperation子类对象中
+ 2.将该对象添加到NSOperationQueue中
+ 3.系统将NSOperation子类对象从NSOperationQueue中取出
+ 4.将取出的操作放到一个新线程中执行
使用队列来执行操作，分为2个阶段：第一阶段：添加到线程队列的过程，是上面的步骤1和2。第二阶段：系统自动从队列中取出线程，并且自动放到线程中执行，是上面的步骤3和4。


### NSInvocationOperation子类
NSInvocationOperation类是NSOperation的一个具体子类，管理作为调用指定的**单个封装任务执行**的操作。这个类实现了一个**非并发操作**。方法属性无论使用该子类的哪个在初始化的方法，都会在添加一个任务。 和NSBlockOperation子类不同的是，因为没有额外添加任务的方法，**使用NSInvocationOperation创建的对象只会有一个任务。**

创建操作对象的方式
使用initWithTarget:selector:object:创建sel参数是一个或0个的操作对象
使用initWithInvocation:方法，添加sel参数是0个或多个操作对象。
在未添加到队列的情况下，创建操作对象的过程中不会开辟线程，会在当前线程中执行同步操作。创建完成后，直接调用start方法，会启动操作对象来执行，或者添加到NSOperationQueue队列中。

**默认情况**下，调用start方法不会开辟一个新线程去执行操作，而是在当前线程**同步执行**任务。只有将其**放到一个NSOperationQueue中**，才会**异步执行操作**。

### NSBlockOperation子类
NSBlockOperation类是NSOperation的一个具体子类，它管理一个或多个块的并发执行。可以使用此对象一次执行多个块，而不必为每个块创建单独的操作对象。当执行多个块时，只有当所有块都完成执行时，才认为操作本身已经完成。

添加到操作中的块(block)将以默认优先级分配到适当的工作队列。

### NSBlockOperation子类
NSBlockOperation类是NSOperation的一个具体子类，它管理一个或多个块的并发执行。可以使用此对象一次执行多个块，而不必为每个块创建单独的操作对象。当执行多个块时，只有当所有块都完成执行时，才认为操作本身已经完成。

添加到操作中的块(block)将以默认优先级分配到适当的工作队列。

创建操作对象的方式
可以通过blockOperationWithBlock:创建NSBlockOperation对象，在创建的时候也添加一个任务。如果想添加更多的任务，可以使用addExecutionBlock:方法。
也可以通过init:创建NSBlockOperation对象。但是这种创建方式并不会在创建对象的时候添加任务，同样可以使用addExecutionBlock:方法添加任务。
对于启动操作和NSInvocationOperation类一样，都可以通过调用start方法和添加NSOperationQueue中来执行操作。

### 线程池
线程池同理，正是因为每次创建、销毁线程需要占用太多系统资源，所以我们建这么一个池子来统一管理线程。用的时候从池子里拿，不用了就放回来，也不用你销毁

**线程池的好处**
在多线程的第一篇文章中我们说过，进程会申请资源，拿来给线程用，所以线程是很占用系统资源的，那么我们用线程池来统一管理线程就能够很好的解决这种资源管理问题。

比如因为不需要创建、销毁线程，每次需要用的时候我就去拿，用完了之后再放回去，所以节省了很多资源开销，可以提高系统的运行速度。

而统一的管理和调度，可以合理分配内部资源，根据系统的当前情况调整线程的数量。

那总结来说有以下 3 个好处：

降低资源消耗：通过重复利用现有的线程来执行任务，避免多次创建和销毁线程。
提高相应速度：因为省去了创建线程这个步骤，所以在拿到任务时，可以立刻开始执行。
提供附加功能：线程池的可拓展性使得我们可以自己加入新的功能，比如说定时、延时来执行某些线程。

###线程同步、线程依赖、线程组
https://www.cnblogs.com/chglog/p/6782413.html

###线程队列同步异步情况
https://www.jianshu.com/p/745ef335e8cc