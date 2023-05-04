# OC 消息机制
## 1、给nil对象发消息程序会不会crash（崩溃）？

答：不会崩溃。OC调用一个方法，在编译阶段会将方法转换成objc_msgSend()（调用这个函数就是消息发送）。objc_msgSend函数的参数1、消息接受者，也就是调用方法的实例对象。2、处理消息的选择器。如果这实例对象为nil，Selector也为空，函数直接返回了，不会crash。但是对NSNul的实例对象发送消息会crash。因为NSNull类只有一个null方法。若对象已被释放，引用计数为0，去调用方法肯定也会crash，访问了野指针。那么，安全的做法就是将释放的对象置为nil，变为空指针。

## 2、objc向一个对象发送消息[obj foo]和objc_msgSend()函数之间有什么关系？

objc_msgSend()是[obj foo]的具体实现。在动态编译时，[obj foo]会被转意为：objc_msgSend(obj, @selector(foo))

先去obj对应的类中找方法；找缓存，找不到时去找方法列表；再找父类，以此向上传递；最后找不到则转发。



#### 上代码

```
/*
* 前提，有一个Person类，
* 类有个实例方法 - (void)sendMessage:(NSString *)msg;
*/

// 调用方法
Person *person =  [Person new];
[person sendMessage:@"helllo"];
// 在编译过程中这段代码会转换成一个C 函数：
//objc_msgSend(person, @selector(sendMessage:),@"hello");

// 对象，类

// 消息转发流程
resolveInstanceMethod// 动态添加实现
forwardingTargetForSelector// 交给其它类有imp的是调用
forwardInvocation// 消息中专中心

```