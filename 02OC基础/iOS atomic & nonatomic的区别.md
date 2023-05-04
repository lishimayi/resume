# iOS atomic & nonatomic的区别

## 前言

> atomic 和 nonatomic 用来决定编译器生成的setter和getter是否原原子操作

### atomic: 
* 为默认修饰符
* 读写安全,其他操作不安全
* 有一定性能开销

当某个属性用`atomic`修饰时, setter函数实现如下:

```
{lock}
if (property != newValue) {
	[property release];
	property  = [newValue retain];
}
{unlock}
```

### nonatomic: 

* 读写操作和其他操作都不安全
* 相对atomic性能消耗较少



