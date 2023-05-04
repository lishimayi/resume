# iOS 属性属性修饰符作用

## 简述

iOS5之前是MRC，内存需要开发者管理；iOS5之后ARC，除去特殊情况（C框架，循环引用）不需要开发者进行手动内存管理。

## 主要修饰符种类

* copy
* retain
* assign
* strong
* weak
* readwrite/readonly（读写策略、访问权限）
* nonatomic/atomic（安全策略）

## 以MRC和ARC进行区分修饰符使用情况

修饰符        | MRC           | ARC     |
--------------------|------------------|-----------------------|
copy |✅|✅|
assign |✅|✅|
retain |✅||
strong | | ✅ | 
weak | | ✅ | 
readwrite/readonly  |✅|✅|
nonatomic/atomic |✅|✅|

## 属性修饰符对实例的引用计数影响

* retain 引用计数 + 1。
* strong 引用计数 + 1。
* copy 修饰对象是，对象被拷贝一份新的，新对象引用计数为1，原有对象引用计数受影响。