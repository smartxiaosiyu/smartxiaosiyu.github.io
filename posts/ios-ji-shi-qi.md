---
title: 'iOS 定时器'
date: 2021-11-25 11:21:58
tags: []
published: true
hideInList: false
feature: 
isTop: false
---
![](https://smartxiaosiyu.github.io/post-images/1637810663803.png)
self 对Timer是强引用 Timer对self是弱引用

### NSTimer 循环引用
```
self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(timerTest) userInfo:nil repeats:YES];
```
self 对Timer是强引用 Timer对self是强引用

更改为以下代码：
```
self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0 target:[MJProxy proxyWithTarget:self] selector:@selector(timerTest) userInfo:nil repeats:YES];
```
```
@interface MJProxy : NSObject
+ (instancetype)proxyWithTarget:(id)target;
@property (weak, nonatomic) id target;
@end
```
```
+ (instancetype)proxyWithTarget:(id)target
{
    MJProxy *proxy = [[MJProxy alloc] init];
    proxy.target = target;
    return proxy;
}

- (id)forwardingTargetForSelector:(SEL)aSelector
{
    return self.target;
}

@end
```
![](https://smartxiaosiyu.github.io/post-images/1637824068622.png)
Timer对OtherObject是强引用 OtherObject对self是弱引用 self 对Timer是强引用 
self 释放了 timer释放了 OtherObject释放了

### NSProxy
继承NSProxy类 不需要init 专门用来做消息转发
直接做消息转发 不会先从父类找方法 因为没有方法
```
+ (instancetype)proxyWithTarget:(id)target
{
    // NSProxy对象不需要调用init，因为它本来就没有init方法
    MJProxy *proxy = [MJProxy alloc];
    proxy.target = target;
    return proxy;
}

- (NSMethodSignature *)methodSignatureForSelector:(SEL)sel
{
    return [self.target methodSignatureForSelector:sel];
}

- (void)forwardInvocation:(NSInvocation *)invocation
{
    [invocation invokeWithTarget:self.target];
}
@end
```
```
@interface MJProxy : NSProxy
+ (instancetype)proxyWithTarget:(id)target;
@property (weak, nonatomic) id target;
@end
```
如果MJProxy MJProxy1有test方法则直接实现

### GCD定时器
NSTimer依赖于RunLoop，如果RunLoop的任务过于繁重，可能会导致NSTimer不准时

而GCD的定时器会更加准时
GCD和内核挂钩


