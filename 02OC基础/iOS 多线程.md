# 一、iOS多线程（三种方式，以及优缺点）

## NSThread

### 简介

> NSThread是苹果官方提供面向对象操作线程的最轻量级技术，简单方便，可以直接操作线程对象。
> 
> 不过需要自己控制线程的生命周期；手动管理线程同步；线程加锁等。
> 
> 开销较大
> 
> 最常用到的就是 `[NSThread currentThread]`获取当前线程。

### NSThread提供的接口

* `@property (class, readonly, strong) NSThread *currentThread;` : 当前线程的属性
* `+ (void)detachNewThreadWithBlock:(void (^)(void))block;` : block形式创建一个线程，并在block中设置这个线程中需要执行的任务。
* `+ (void)detachNewThreadSelector:(SEL)selector toTarget:(id)target withObject:(nullable id)argument;`：选择器的形式实现开启线程执行任务
* `BOOL isMulti = [NSThread isMultiThreaded];`  ： 当前app是否是多线程，（暂时不明确，官方文档没没看懂）
* `@property(readonly, retain) NSMutableDictionary *threadDictionary;`

```
Summary 
The thread object's dictionary.

Declaration
@property(readonly, retain) NSMutableDictionary *threadDictionary;

Discussion
You can use the returned dictionary to store thread-specific data.
The thread dictionary is not used during any manipulations of the NSThread object—it is simply a place where you can store any interesting data. 
For example, Foundation uses it to store the thread’s default NSConnection and NSAssertionHandler instances. 
You may define your own keys for the dictionary.
```

*  `+ (void)sleepUntilDate:(NSDate *)date;` ：阻塞线程直到指定时间点

```
Summary
Blocks the current thread until the time specified.
Declaration
+ (void)sleepUntilDate:(NSDate *)date;
Discussion
No run loop processing occurs while the thread is blocked.
Parameters

aDate	
The time at which to resume processing.

```

* `+ (void)sleepForTimeInterval:(NSTimeInterval)ti;`：阻塞线程一段时间
* `+ (void)exit;`：退出当前线程
* 后面的接口

```
+ (double)threadPriority;
+ (BOOL)setThreadPriority:(double)p;

@property double threadPriority API_AVAILABLE(macos(10.6), ios(4.0), watchos(2.0), tvos(9.0)); // To be deprecated; use qualityOfService below

@property NSQualityOfService qualityOfService API_AVAILABLE(macos(10.10), ios(8.0), watchos(2.0), tvos(9.0)); // read-only after the thread is started

@property (class, readonly, copy) NSArray<NSNumber *> *callStackReturnAddresses API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));
@property (class, readonly, copy) NSArray<NSString *> *callStackSymbols API_AVAILABLE(macos(10.6), ios(4.0), watchos(2.0), tvos(9.0));

@property (nullable, copy) NSString *name API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));

@property NSUInteger stackSize API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));

@property (readonly) BOOL isMainThread API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));
@property (class, readonly) BOOL isMainThread API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0)); // reports whether current thread is main
@property (class, readonly, strong) NSThread *mainThread API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));

- (instancetype)init API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0)) NS_DESIGNATED_INITIALIZER;
- (instancetype)initWithTarget:(id)target selector:(SEL)selector object:(nullable id)argument API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));
- (instancetype)initWithBlock:(void (^)(void))block API_AVAILABLE(macosx(10.12), ios(10.0), watchos(3.0), tvos(10.0));

@property (readonly, getter=isExecuting) BOOL executing API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));
@property (readonly, getter=isFinished) BOOL finished API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));
@property (readonly, getter=isCancelled) BOOL cancelled API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));

- (void)cancel API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));

- (void)start API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));

- (void)main API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));	// thread body method

@end

FOUNDATION_EXPORT NSNotificationName const NSWillBecomeMultiThreadedNotification;
FOUNDATION_EXPORT NSNotificationName const NSDidBecomeSingleThreadedNotification;
FOUNDATION_EXPORT NSNotificationName const NSThreadWillExitNotification;

@interface NSObject (NSThreadPerformAdditions)

- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(nullable id)arg waitUntilDone:(BOOL)wait modes:(nullable NSArray<NSString *> *)array;
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(nullable id)arg waitUntilDone:(BOOL)wait;
	// equivalent to the first method with kCFRunLoopCommonModes

- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(nullable id)arg waitUntilDone:(BOOL)wait modes:(nullable NSArray<NSString *> *)array API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));
- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(nullable id)arg waitUntilDone:(BOOL)wait API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));
	// equivalent to the first method with kCFRunLoopCommonModes
- (void)performSelectorInBackground:(SEL)aSelector withObject:(nullable id)arg API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));

@end
```

* 优点：比其他两个更轻量级。
* 缺点：需要自己管理线程的生命周期，线程同步。线程同步对数据的加锁会有一定的开销。

## Coca operation

* 优点：不需要关心线程管理，数据同步的事情，可以把精力放在自己需要的执行操作上。Cocoa operation 相关的类：NSOperation，NSOperationQueue。NSOperation是个抽象类，使用它必须使用它的子类，可以实现它或者使用它定义好的两个子类：NSInvocationOperation和NSBlockOperation。创建NSOperation子类对象，把对象添加到NSOperationQueue队列里执行。

## GCD

* 优点：Apple开发多核编程解决方法。

# 二、iOS与多线程（八）

https://www.jianshu.com/p/580050bbfc91

### 前言

> 信号量机制是多线程通信中比较重要的一部分，对于NSOperation可以设置并发数，但是对于GCD就不能设置并发数，那么就只能靠信号量机制了。

### 线程与进程

### 1. 进程

进程是指在系统中正在运行的一个程序，每个进程之间是独立的，每个进程就均运行在其专用的且受保护的没存空间

### 2. 线程

1个进程想要执行任务，必须得有线程，每1个进程至少有一条线程，这个线程称为主线程。一个进程（程序）的所有任务都在线程中执行。

也可以这样理解：线程就是进程的一个执行单元，是进程内科调度实体。比进程更小的独立运行的基本单位。线程也被称为轻量级的进程

一个程序至少一个进程，一个进程至少一个线程。

线程的串行：1个线程中任务的执行是串行的。如果要在一个线程中执行多个任务，那么只能一个一个地按顺序执行这些任务。也就是说，在同一时间内，一个线程只能执行一个任务。

### 3.线程与进程的区别

* 地址空间：统一进程的线程共享本进程的地址空间，而进程之间则是独立的地址空间。
* 资源拥有：同一进程内的线程共享本进程的资源如内存，I/O,CPU等。但是进程之间的资源是独立的。一个进程崩溃后，在保护模式下不会对其他进程产生影响
