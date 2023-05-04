# iOS Notification实现原理

## 一、基本使用

### 基本概念：

`NSNotification`是ios中一个调度消息通知的类，采用单利模式，在程序中实现传值，回调等地方，应用很广。iOS中`NSNotification`和`NSNotificationCenter`是使用`观察者模式`来实现的，用于跨层的消息传递。

### 在使用通知的时候注意的一些细节：

* 通知一定要移除，在 dealloc 方法中移除，但是 iOS9 之后不部分情况大再需要手动移除了。

	* 不需要手动移除的原因：
		* 通知`NSNotification`在注册者被回收时需要手动移除，是一直以来的使用准则。原因是在MRC时代，通知中心持有的是注册者的`unsafe_unretained`指针，在注册者被回收时若不对通知进行手动移除，则指针指向被回收的内存区域，成为野指针。这时再发送通知，便会造成crash。而在iOS 9以后，通知中心持有的是注册者的weak指针，这时即使不对通知进行手动移除，指针也会在注册者被回收后自动置空。我们知道，向空指针发送消息是不会有问题的。
但是有一个例外。如果用`- (id <NSObject>)addObserverForName:(nullable NSNotificationName)name object:(nullable id)obj queue:(nullable NSOperationQueue *)queue usingBlock:(void (^)(NSNotification *note))block API_AVAILABLE(macos(10.6), ios(4.0), watchos(2.0), tvos(9.0));`这个API来注册通知，可以直接传入block类型参数。使用这个API会导致注册者被系统retain，因此仍然需要像以前一样手动移除通知，同时这个block类型参数也需注意避免循环引用。

* 通知有分同步通知和异步通知，只是我们同步通知用的比较多。
* 不能用 `- (instancetype)init`初始化一个通知。

## 通知实现的原理

### 概述


