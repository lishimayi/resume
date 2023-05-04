# OC block

## block类型

* GlobalBlock
	* 位于全局区
	* 在block内部未使用`静态变量`和`全局变量`以外的其他`外部变量`
* MallocBlock
	* 位于堆区
	* 在block内部使用了`局部变量`或者`OC属性`,并且赋值给强引用或者copy修饰的变量
* StackBlock
	* 位于栈区
	* 内部使用了`局部变量`或者`OC属性`,但是未赋值给强引用或者copy修饰的变量

## 什么情况下会触发block的copy

* 手动copy
	*  stackBlock, 会变成mallocBlock
	*  globalBlock, 还是globalBlock
* block作为返回值
	* 使用了局部变量就会被cpoy到堆上(因为被强引用了)
	* 未使用局部变量就还是globalBlock
* 被强引用或者Copy修饰
* 系统API包含usingBlock
	*  globalblock

## Block的循环引用是如何产生的如何解决

__weak

