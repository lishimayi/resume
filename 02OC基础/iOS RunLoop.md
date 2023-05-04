# iOS-RunLoop

## 一、RunLoop简介

RunLoop字面意思运行循环（跑圈），在我们的项目也就是一个死循环（充满灵性的）。为什么说他是充满灵性的死循环呢？因为他可以在我们需要的时候自己跑起来运行，在我们没有操作的时候就停下来休息，这样就能够充分节省CPU资源提高程序的性能。

## 二、RunLoop基本作用：

 1. **保持程序持续运行**，程序一启动就会开一个主线程，主线程一开起来就会跑一个主线程对应的`RunLoop`,`RunLoop`保证主线程不被销毁，也就保证了程序的持续运行。
 2. **处理APP中的各种事件**，（如：触摸事件，定时事件，Selector事件等）。
 3. **节省CPU资源，提高程序性能**，程序运行起来时，当什么操作都没有的时候，`RunLoop`就会告诉CPU，现在没事情做，我要去休息。这时候CPU就会将其资源释放出来去做其他事情，当有事情要做`RunLoop`就会立马去做事情。**我们通过下图了解RunLoop内部运行原理**
 
![RunLoop原理图.png](https://upload-images.jianshu.io/upload_images/4034746-5567380746852892.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过图片可以看出，RunLoop在跑圈过程中，当收到Input sources或者Timer sources的时候就会交个对应的方法去处理。当没有事件消息传入的时候，RunLoop就会休息了，这里只是简单的理解这张图。接下来我们来了解RunLoop对象和一些相关类，来更深入的了解RunLoop运行流程。

## 三、RunLoop在哪里开启

我们知道主线程一开启，就会跑一个和主线程对应的RunLoop，那么RunLoop一定是在程序的入口main函数中开启。

```
int main(int argc, char *argv[]) {
	@autoreleaspool {
		return UIApplicationMain(argc, argv, nil ,NSStringFromClass([AppDelegate class]));
	}
}
```

进入UIApplication

	UIKIT_EXTERN int UIApplicationMain(int argc, char *argv[], NSString *__nullable principalClassName, NSString *__nullable delegateClassName);
	
我们发现它返回的是一个int，那么我们对main函数做了一些修改

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

运行程序，我们发现只会打印“开始”，并不会打印结束，说明**UIApplicationMain函数中，开启了一个和主程序相关的RunLoop，导致UIApplicationMain不会返回，一直在运行中，也就保证了程序的持续运行。**

我们看下RunLoop的源码

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

我们发现RunLoop确实do while 通过判断result来实现的。因此，我们可以把RunLoop看成是一个死循环。如果没有RunLoop，UIApplicationMain函数执行完毕之后将直接返回，无法支持程序的持续运行。

## 四、RunLoop对象

两个框架支持RunLoop：

1. Foundation框架的NSRunLoop对象（基于CFRunLoopRef封装）
2. CoreFoundation框架的CFRunLoopRef（重点学习这一个）

### 获得Runloop对象

```
// Foundation
[NSRunLoop currentRunLoop];
[NSRunLoop mainRunLoop];

// Core Foundation
CFRunLoopCurrent();
CFRunLoopGetMain();
```

## RunLoop和线程的关系

1. 每线程都有唯一一个与之对应的RunLoop对象。
2. 主线程的RunLoop已经自动创建好了，子线程的RunLoop需要主动创建。
3. RunLoop在第一次获取时创建，在线程结束时销毁。

### 1. 主线程相关联的RunLoop创建

CFRunLoopRef源码

```
// 创建字典
CFMutableDictionaryRef dict = CFDictionaryCreatMutable(kCFAllocatorSystemDefault, 0, NULL, &kCFTypeDictionaryValueCallBacks);
// 创建主线程，根据传入的主线程创建主线程对应的RunLoop
CFRunLoopRef mainLoop = __CFRunLoopCreate(pthread_main_thread_np());
// 保存主线程 将主线程的 -key 和RunLoop-value保存到字典中
CFDictionarySetValue(dict, pthreadPointer(pthread_main_thread_np()), mainLoop);
```

### 2. 创建与子线程相关联的RunLoop

CFRunLoopRef源码

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

**从上面的代码可以看出，线程和RunLoop之间是一一对应的，其关系是保存在一个字典中，所以我们创建子线程RunLoop时，需要在子子线程中获取当前线程的RunLoop对象即可`[NSRunLoop currentRunLoop];`如果不获取，子线程就不会创建与之相关联的RunLoop，并且只能在一个线程的内部获取其RunLoop**

**`[NSRunLoop currentRunLoop];`方法调用时，会去先查看一下字典里有没有存子线程相对应的RunLoop，有则直接返回，否则就创建一个存在字典中**

略
原文：
https://www.jianshu.com/p/b9426458fcf6

## RunLoop实现的部分功能（部分）：

* autoreleasePool
* 监听和响应事件（如：事件响应，手势识别，网络事件）
* UI更新
* 定时器
* PerformSelector


# 监听事件

## 添加RunLoop 观察者

```
- (void)addRunloopObserver {
    // 获取当前RunLoop
    CFRunLoopRef runloop = CFRunLoopGetCurrent();
    // 添加观察者
    static CFRunLoopObserverRef defaultModeObserver;
    

}
```
