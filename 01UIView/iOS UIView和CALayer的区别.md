# iOS UIView和CALayer的区别

## 基于框架不同

* CALayer基于QuartzCore框架
* UIView基于UIKit框架

## 是否可以响应时间

* UIView可以响应事件 （UIView继承自UIResponder，只有继承自UIREsponder的类可以响应事件）
* CALayer不可以响应时间（CALayer继承自NSObject）

## UIView与CALayer的关系

* UIView 遵守了CALayer的 `CALayeDelegate`协议 ,所以，UIView是CALayer的代理。
* UIVIew 主要负责事件处理，CALayer主要负责绘制。
* 每个UIView内部都有一个CALayer在背后提供内容的绘制和显示。
* UIView的尺寸和样式都是由内部的Layer提供。两者都有树状层级结构，layer内部有Sublayers，view内部有subViews，但是layer比view多一个anchorPoint

## 总结

* 创建UIView对象时，UIView内部会自动创建一个层（CALayer对象），通过UIView的layer属性可以访问这个层。当UIView需要显示到屏幕上时，会调用drawRect：方法进行绘制渲染，并且会将所有内容绘制在自己的层上，绘制完毕后，系统会将层拷贝到屏幕上，于是就完成了UIView的显示。
* UIView相比CALayer最大的区别就在UIView继承自UIResponder，可以响应用户事件，而CALayer不可以。UIView重点在内容的管理，CALayer 重点在内容的绘制。
* UIView本身，更像是一个CALayer的管理器，访问它的和绘图、坐标相关的属性，如frame，bounds等，实际上内部都是访问它所在CALayer的相关属性
* UIView和CALayer是相互依赖的关系。UIView依赖CALayer提供的内容，CALayer依赖UIView提供的容器来显示绘制的内容。归根到底CALayer是这一切的基础，如果没有CALayer，UIView自身也不会存在，UIView是一个特殊的CALayer实现，添加了响应事件的能力。