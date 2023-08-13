# iOS-RunLoop

## RunLoop 是什么？

- RunLoop 字面意思运行循环（跑圈），在我们的项目也就是一个死循环（充满灵性的）。为什么说他是充满灵性的死循环呢？因为他可以在我们需要的时候自己跑起来运行，在我们没有操作的时候就停下来休息，这样就能够充分节省 CPU 资源提高程序的性能。

## RunLoop 基本作用：

- **保持程序持续运行**，程序一启动就会开一个主线程，主线程一开起来就会跑一个主线程对应的`RunLoop`,`RunLoop`保证主线程不被销毁，也就保证了程序的持续运行。
- **处理 APP 中的各种事件**，（如：触摸事件，定时事件，Selector 事件等）。
- **节省 CPU 资源，提高程序性能**，程序运行起来时，当什么操作都没有的时候，`RunLoop`就会告诉 CPU，现在没事情做，我要去休息。这时候 CPU 就会将其资源释放出来去做其他事情，当有事情要做`RunLoop`就会立马去做事情。**我们通过下图了解 RunLoop 内部运行原理**

![RunLoop原理图.png](https://upload-images.jianshu.io/upload_images/4034746-5567380746852892.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过图片可以看出，RunLoop 在跑圈过程中，当收到 Input sources 或者 Timer sources 的时候就会交个对应的方法去处理。当没有事件消息传入的时候，RunLoop 就会休息了，这里只是简单的理解这张图。接下来我们来了解 RunLoop 对象和一些相关类，来更深入的了解 RunLoop 运行流程。

## RunLoop 运行逻辑/流程

1. 通知 observers：进入 Loop
2. 通知 observers：即将处理 Timers
3. 通知 observers：即将处理 Sources
4. 处理 blocks
5. 处理 Source0（可能会再次处理 blocks）
6. 如果存在 Source1 则跳转到第 8 步
7. 通知 observers：开始休眠（等待被唤醒）
8. 通知 observers：结束休眠（被某个消息唤醒）

   1. 处理 Timers
   2. 处理 GCD Async To main queue
   3. 处理 Source1

9. 处理 blocks
10. 根据前面执行的结果，决定如何操作
    1. 回到第 2 步
    2. 退出 Loop
11. 通知 observers：退出 Loop

## 关于 Runloop 的几个类

- Core Foundation 中关于 RunLoop 的 5 个类
  - CFRunLoopRef
  - CFRunLoopModeRef
  - CFRunLoopSourceRef
  - CFRunLoopTimerRef
  - CFRunLoopObserverRef

## CFRunLoopModeRef

- 代表 RunLoop 的运行模式
- 一个 RunLoop 包含若干个 Mode，每个 Mode 又包含若干个 Source0、Source1、Timer、Observer
- RunLoop 启动时只能选择其中一个 Mode 作为 currentMode
- CFRunLoopModeRef 的 5 种：
  - kCFRunLoopDefaultMode：APP 默认 Mode，通常主线程在这个 mode 下运行
  - UITrackingRunbLoopMode：界面跟踪 Mode，用于 ScrollView 追踪触摸滑动，保证界面滑动时不受其他 Mode 影响。
  - UIInitializationRunLoopMode：在刚启动时进入的第一个 Mode，启动完成后就不再使用
  - GSEventReceiveRunLoopMode：接受系统事件的内部 Mode，通常用不到。
  - kCFRunLoopCommonModes：这是一个占位用 Mode，不是一种真正的 Mode
- 关于 Mode 的几点知识:
  - 如果当前 Mode 内没有事件要处理，RunLoop 将会直接退出。

## Source0

- 触摸事件
- perform Selectors

## Source1

- 基于 Port 的线程间通

## Timer

- 定时器，NSTimer

## Observers

- 监听器
- 用来监听 RunLoop 的状态。

## RunLoop 对象

iOS 中有两套 API 来访问和使用 Runloop：

1. Foundation 框架的 NSRunLoop 对象
2. CoreFoundation 框架的 CFRunLoopRef（重点学习这一个）

注意：

- NSRunloop 和 CFRunLoopRef 都代表着 RunLoop 对象
- NSRunLoop 是基于 CFRunLoopRef 的一层 OC 封装

关于 CFRunLoopRef：

- 是开源的
- 下载链接：https://opensource.apple.com/tarballs/CF/
- 下载链接 2：https://github.com/apple-oss-distributions/CF/tags

## RunLoop 和线程的关系

- 每线程都有唯一一个与之对应的 RunLoop 对象。
- 主线程的 RunLoop 已经自动创建好了。
- 子线程 RunLoop 在第一次获取时创建，
- 在线程结束时销毁。
- Runloop 保存在一个全局的 Dictionary 里，线程作为 key，Runloop 作为 Value

## RunLoop 实现的部分功能（部分）：

- autoreleasePool
- 监听和响应事件（如：事件响应，手势识别，网络事件）
- UI 更新
- 定时器
- PerformSelector

## 应用范围

- 定时器（timer）、PerformSelector
- GCD Async Main Queue
- 事件响应、手势识别、界面刷新
- 网络请求
- AutoreleasePool

## 面试题：

1.  讲讲 Runloop 项目中有用到吗？
2.  Runloop 内部实现逻辑?
3.  Runloop 和线程的关系？
4.  timer 和 Runloop 的关系？
5.  程序中添加每 3 秒响应一次 NSTimer，当拖动 tableview 是 timer 可能无法响应要怎么解决？
6.  Runloop 是怎么响应用户操作的，具体流程是什么样的？
7.  说说 Runloop 的几种状态？
8.  Runloop 的 Mode 作用是什么？

# 附加信息

## 监听事件，添加 RunLoop 观察者

```
- (void)addRunloopObserver {
    // 获取当前RunLoop
    CFRunLoopRef runloop = CFRunLoopGetCurrent();
    // 添加观察者
    static CFRunLoopObserverRef defaultModeObserver;


}
```

## 主线程相关联的 RunLoop 创建

CFRunLoopRef 源码

```
// 创建字典
CFMutableDictionaryRef dict = CFDictionaryCreatMutable(kCFAllocatorSystemDefault, 0, NULL, &kCFTypeDictionaryValueCallBacks);
// 创建主线程，根据传入的主线程创建主线程对应的RunLoop
CFRunLoopRef mainLoop = __CFRunLoopCreate(pthread_main_thread_np());
// 保存主线程 将主线程的 -key 和RunLoop-value保存到字典中
CFDictionarySetValue(dict, pthreadPointer(pthread_main_thread_np()), mainLoop);
```

## 创建与子线程相关联的 RunLoop

CFRunLoopRef 源码

```
    // 从字典中获取子线程的runloop
    CFRunLoopRef loop = (CFRunLoopRef)CFDictionaryGetValue(__CFRunLoops, pthreadPointer(t));
    __CFUnlock(&loopsLock);
    if (!loop) {
        // 如果子线程的runloop不存在,那么就为该线程创建一个对应的runloop
    CFRunLoopRef newLoop = __CFRunLoopCreate(t);
        __CFLock(&loopsLock);
    loop = (CFRunLoopRef)CFDictionaryGetValue(__CFRunLoops, pthreadPointer(t));
        // 把当前子线程和对应的runloop保存到字典中
    if (!loop) {
        CFDictionarySetValue(__CFRunLoops, pthreadPointer(t), newLoop);
        loop = newLoop;
    }
        // don't release run loops inside the loopsLock, because CFRunLoopDeallocate may end up taking it
        __CFUnlock(&loopsLock);
    CFRelease(newLoop);
    }
```

**从上面的代码可以看出，线程和 RunLoop 之间是一一对应的，其关系是保存在一个字典中，所以我们创建子线程 RunLoop 时，需要在子子线程中获取当前线程的 RunLoop 对象即可`[NSRunLoop currentRunLoop];`如果不获取，子线程就不会创建与之相关联的 RunLoop，并且只能在一个线程的内部获取其 RunLoop**

**`[NSRunLoop currentRunLoop];`方法调用时，会去先查看一下字典里有没有存子线程相对应的 RunLoop，有则直接返回，否则就创建一个存在字典中**

略
原文：
https://www.jianshu.com/p/b9426458fcf6

## 获得 Runloop 对象

```
// Foundation
[NSRunLoop currentRunLoop];
[NSRunLoop mainRunLoop];

// Core Foundation
CFRunLoopCurrent();
CFRunLoopGetMain();
```

## RunLoop 在哪里开启

- 我们知道主线程一开启，就会跑一个和主线程对应的 RunLoop，那么 RunLoop 一定是在程序的入口 main 函数中开启。

```
int main(int argc, char *argv[]) {
	@autoreleaspool {
		return UIApplicationMain(argc, argv, nil ,NSStringFromClass([AppDelegate class]));
	}
}
```

进入 UIApplication

    UIKIT_EXTERN int UIApplicationMain(int argc, char *argv[], NSString *__nullable principalClassName, NSString *__nullable delegateClassName);

我们发现它返回的是一个 int，那么我们对 main 函数做了一些修改

```
int main(int argc, char * argc[]) {
	@autoreleasepool {
		NSLog(@"开始");
		int re = UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
		NSLog(@"结束");
		return re;
	}
}
```

运行程序，我们发现只会打印“开始”，并不会打印结束，说明**UIApplicationMain 函数中，开启了一个和主程序相关的 RunLoop，导致 UIApplicationMain 不会返回，一直在运行中，也就保证了程序的持续运行。**

我们看下 RunLoop 的源码

```
// 用Default Model启动
void CFRunLoopRun(void) { /* DOES CALLOUT */
	int32_t result;
	do {
		result = CFRunLoopRunSpecific(CFRunLoopGetCurrent(), kCFRunLoopDefaultMode, 1.0e10, false);
		CHECK_FOR_FORK();
	} while (kCFRunLoopRunStoped != result && kCFRunLoopRunFinished != result);
}
```

我们发现 RunLoop 确实 do while 通过判断 result 来实现的。因此，我们可以把 RunLoop 看成是一个死循环。如果没有 RunLoop，UIApplicationMain 函数执行完毕之后将直接返回，无法支持程序的持续运行。
