# iOS事件传递和响应机制

## 前言：

#### 按照事件顺序和事件的生命周期是这样的：

1. 事件的产生和传递：
	
	 事件如何从父控件传递到子控件并寻找最合适的View。寻找合适view的底层实现，拦截事件的处理
2. 找到最合适的view后事件的处理：
	
	touches方法的重写，也就是事件的响应

#### 其中重点和难点：

* 寻找合适的view
* 寻找最合适的view的底层实现（hitTest:withEvent: 的底层实现）

## 一、iOS中的事件

#### 学习触摸事件首先要了解一个比较重要的概念：响应者对象（UIResponder）。

iOS中，只有继承了UIResponder的对象才能接收并处理事件。此对象我们称之为："响应者对象"。以下是继承了UIResponder的类：

* UIApplication
* UIViewController
* UIView

### 那，为什么继承自UIResponder类就接收并处理事件呢：

因为UIResponder中提供了以下对象方法来处理事件。

```
// Generally, all responders which do custom touch handling should override all four of these methods.
// Your responder will receive either touchesEnded:withEvent: or touchesCancelled:withEvent: for each
// touch it is handling (those touches it received in touchesBegan:withEvent:).
// *** You must handle cancelled touches to ensure correct behavior in your application.  Failure to
// do so is very likely to lead to incorrect behavior or crashes.
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;
- (void)touchesMoved:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;
- (void)touchesEnded:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;
- (void)touchesCancelled:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;
- (void)touchesEstimatedPropertiesUpdated:(NSSet<UITouch *> *)touches API_AVAILABLE(ios(9.1));

// Generally, all responders which do custom press handling should override all four of these methods.
// Your responder will receive either pressesEnded:withEvent or pressesCancelled:withEvent: for each
// press it is handling (those presses it received in pressesBegan:withEvent:).
// pressesChanged:withEvent: will be invoked for presses that provide an analog value
// (like thumbsticks or analog push buttons)
// *** You must handle cancelled presses to ensure correct behavior in your application.  Failure to
// do so is very likely to lead to incorrect behavior or crashes.
- (void)pressesBegan:(NSSet<UIPress *> *)presses withEvent:(nullable UIPressesEvent *)event API_AVAILABLE(ios(9.0));
- (void)pressesChanged:(NSSet<UIPress *> *)presses withEvent:(nullable UIPressesEvent *)event API_AVAILABLE(ios(9.0));
- (void)pressesEnded:(NSSet<UIPress *> *)presses withEvent:(nullable UIPressesEvent *)event API_AVAILABLE(ios(9.0));
- (void)pressesCancelled:(NSSet<UIPress *> *)presses withEvent:(nullable UIPressesEvent *)event API_AVAILABLE(ios(9.0));

- (void)motionBegan:(UIEventSubtype)motion withEvent:(nullable UIEvent *)event API_AVAILABLE(ios(3.0));
- (void)motionEnded:(UIEventSubtype)motion withEvent:(nullable UIEvent *)event API_AVAILABLE(ios(3.0));
- (void)motionCancelled:(UIEventSubtype)motion withEvent:(nullable UIEvent *)event API_AVAILABLE(ios(3.0));

- (void)remoteControlReceivedWithEvent:(nullable UIEvent *)event API_AVAILABLE(ios(4.0));

```

## 二、事件处理

#### 用UIView为例来说明触摸事件的处理。

```
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;
- (void)touchesMoved:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;
- (void)touchesEnded:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;
- (void)touchesCancelled:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;
```

以上方法系统会自动调用。我们可以通过重写来处理自定义事件。

* 如果两根手指同事触摸一个view：那么View只会调用一次touchesBegan:withEvent:方法，taches参数中装2个UITouch对象。
* 如果这两根手指一前一后触摸VIew：那么View会调用两次touchesBegan:withEvent:方法，每次传一个UITouch对象。

### UITouch对象

* 用户单点触摸一次屏幕时，会创建一个UITouch对象。
* 用户多点触摸一次屏幕时，会创建对应触摸点数的UITouch对象。

### UITouch的作用

* 保存触点的信息：位置，事件，阶段。
* 当点移动时，这个UITouch对象的相应信息会更新。
* 当点离开时，对应UITouch对象被销毁。

`提示：`iOS开发中，避免使用双击事件。

### UITouch属性

```
@property (nonatomic, readonly, retain) UIWindow *window;// touch of window
@property (nonatomic, readonly, retain) UIView *view;// touch of view

@property (nonatomic, readonly) NSInteger tapCount;// tap counts in a shot time
@property (nonatomic, readonly) NSTimeInterval timestamp; // time record 

@property (nonatomic, readonly) UITouchPhase phase;// touch state

```

### UITouch方法

```
// touch point 
- (CGPoint)locationInView:(UIView *)view;
// last touch point 
- (CGPoint)previousLocationInView:(UIView *)view;
```

### code imp

```
- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event{ 
    // 想让控件随着手指移动而移动,监听手指移动 
    // 获取UITouch对象 
    UITouch *touch = [touches anyObject]; 
    // 获取当前点的位置 
    CGPoint curP = [touch locationInView:self]; 
    // 获取上一个点的位置 
    CGPoint preP = [touch previousLocationInView:self]; 
    // 获取它们x轴的偏移量,每次都是相对上一次 
    CGFloat offsetX = curP.x - preP.x; 
    // 获取y轴的偏移量 
    CGFloat offsetY = curP.y - preP.y; 
    // 修改控件的形变或者frame,center,就可以控制控件的位置 
    // 形变也是相对上一次形变(平移) 
    // CGAffineTransformMakeTranslation:会把之前形变给清空,重新开始设置形变参数 
    // make:相对于最原始的位置形变 
    // CGAffineTransform t:相对这个t的形变的基础上再去形变 
    // 如果相对哪个形变再次形变,就传入它的形变 
    self.transform = CGAffineTransformTranslate(self.transform, offsetX, offsetY);}
```

## 三、iOS中事件的产生和传递

### 事件的产生

* 发生触摸事件后，系统会将事件加入到有UIApplication管理的队列中（FIFO：先进就先出）。
* UIApplication 从队列中取出一个事件，下发处理，通常是程序的主窗口（keyWindow）
* keyWindow会在视图层次结构中寻找一个最合适的视图来处理触摸事件，**这也是整个事件处理的第一步**

### 事件的传递

* 触摸事件的传递是从父控件传递到子控件。
* UIApplication - > window - > 。。。- > someView（最合适的）
* 注意：如果父控件不能接收事件，子控件就不能接收到事件。

### 如何找到最合适的控件来处理事件？

1. 判断keWindow（主窗口）自己能否接收触摸事件。（此处设定能接收事件）
2. 然后判断触摸点在不在自己身上（keywindow，肯定在自己身上）
3. 然后从后向前遍历自己的子控件，并重复(1. )和(2.)操作。
4. 最后知道遍历子控件时未找到合适的view，那么这个事件就确认适合自己来处理。

#### 判断不合适的条件（不能接收并处理触摸事件）：

* 不允许交互：userInterfaceEnable = NO;
* 隐藏：.hidden  = YES;
* 透明度：透明度< 0.01 认为控件时透明的，不能接收、处理事件。

## 寻找合适view的底层实现

两个重要的方法：

` hitTest:withEvent:`方法

`pointInside:withEvent:`方法

### hitTest：withEvent：方法

#### 什么时候调用？

* 只要事件传递给一个控件，这个控件就会调用它自己的hitTest：withEvent：方法。

#### 作用

* 寻找并返回最合适的view（能够响应事件的那个最合适的view）

`注意`：不管这个控件能不能处理事件，也不管触摸点在不在这个控件上，事件都会先传递给这个控件，随后再调用hitTest：withEvent：方法。

#### 拦截事件的处理

* 正因为hitTest：withEvent：方法可以返回最合适的View，所以可以通过重写这个方法，返回指定的View作为最合适的View。
* 不管点击哪里，最合适的View都是hitTest：withEvent：方法返回的这个View。
* 通过重写hitTest：withEvent：，就可以拦截事件传递的过程，想让谁处理事件就让谁处理事件。

事件传递给谁，谁就会调用hitTest：withEvent：方法。

`注意：`如果hitTest：withEvent：方法返回nil，那么调用该方法的控件本身和其子控件都不是最合适的view，也就是在自己身上没有找到最合适的view，那么最合适的就是父控件。

***所以最合适的传递顺序是这样的：***

产生触摸事件 →UIApplication事件队列→[UIWindow hitTest:WithEvent:]→返回**更合适**的view→[子控件 hitTest：withEvent：]→返回**最合适**的view

```
#import "WSWindow.h"

@implementation WSWindow
// 什么时候调用：只要事件一传递给一个控件，那么这个控件就会调用自己的方法
// 作用：返回最合适的View
// UIApplication → [UIWindow hitTest:withEvent:]; 寻找最合适额view告诉系统
// point:当前触摸的点
// point: 是方法调用坐着坐标系上的点

- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event {
	// 1. 判断下窗口能否接收事件
	if (self.userInterfaceEnabled == NO || self.hidden == YES || self.alpha <= 0.01) {
		return nil;
	}
	// 2. 不在窗口上
	if ([self pointInside:point withEvent:event] == NO) {
		return nil;
	}
	// 3. 从后往前遍历子控件
	int count = (int)self.subviews.count;
	for (int i = 0 ; i < count ; i++) {
		// 获取子控件
		UIView *childView = self.subviews[count - 1 - i];
		// 坐标系转换，把自己点转换为子控件的点
		CGPoint childP = [self convertPoint:point toView:childView];
		UIView *fitView = [childView hitTest:childP withEvent:event];
		if (fitView) {
			// 如果能找到最合适的view
			return fitView;
		}
	}
	// 4.没找到更合适的View ，也就是没有比自己更合适的了。
	return self;
}
// 作用；判断传入的点在不在调用者的坐标系上，
// point：是方法调用者坐标系上的点。
/*
- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event {
	return NO;
}
*/ 

- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event {
	NSLog(@"%s",__func__);
}

@end
```

**hitTest:withEvent:方法底层会调用pointInside：withEvent：方法判断点在不在方法调用者的范围内。**

#### pointInside：withEvent：方法

pointInside：withEvent：方法判断点在不在当前view上。是就返回YES，如果不是就返回NO，那么方法的调用者也不能够处理事件。

## 四、事件的响应

### 触摸事件的处理整个过程

1. 用户点击屏幕后产生一个触摸事件，经过一些列的传递过程后，会找到最合适的视图控件来处理这个事件。
2. 找打最合适的视图控件之后，会调用控件的touches方法来做具体的时间处理，（如：touchesBegan，touchesMoved，touchedEnded）
3. 这些touches方法默认做法是将事件顺着响应者链向上传递（也就是touch方法默认不处理事件，只传递事件），将事件交给上一个响应者做处理。

### 响应者链示意图

**响应者链：**在iOS中程序中无论是最后面的UIWindow还是最前面的某个按钮，他们的摆放是有前后关系的，一个控件可以放在另一个控件的上面或者下面，那么用户点击某个控件时候，那么用户点击某个控件时是触发上面的控件还是下面的空间呢?这种先后关系构成了一条链条叫“响应者链”。也可以说，响应者链是有多个响应者对象连起来的链条。在iOS中响应者链的关系可用下图表示：

![响应者链示意图](https://upload-images.jianshu.io/upload_images/1055199-2a49a16e1e483b5c.png)

**响应者对象**：能处理事件的对象，也就是继承自UIResponder的对象。

**作用：** 能很清楚的看见每个响应者之间的联系，并且可以让一个事件多个对象处理。

**如何判断上一个响应者**

* 如果当前这个view是控制器的view，那么控制器就是上一个响应者。
* 如果当前这个view不是控制器的view，那么父控件就是上一个响应者。

#### 响应者链的事件传递过程

1. 如果当前view是控制器的view，那么控制器及时上一个响应者，事件传递给控制器。如果当前view不是控制器的view，那么父视图就是当前view的上一个响应者，事件传递给它的父视图。
2. 在视图层次结构的最顶级视图，如果也不能处理接收到的事件或消息，则其将事件或者消息传递给window对象进行处理。
3. 如果window对象也不处理，则其将事件或者消息传递给UIApplication对象
4. 如果UIApplication也不能处理改时间或消息，则将其丢弃。
