# OC KVO

## KVO坑

### 一、当在同一个类中存在多个kvo时，`- (void)oberveValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context;` 方法需要加判断：

```
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context {
	if (object == _tableView && [keyPath isEqualToString:@"contentOffset"]) {
		[self doSomethingWhenContentOffsetChanges];
	}else {// 父类也要调用
		[super observeValueForKeyPath:keyPath ofObject:object change:change context:context];
	}
}
```

### 二、dealloc中对KVO的注销。

KVO的一种缺陷(其实不能称为缺陷，应该称为特性)是，当对同一个 keypath进行两次removeObserver时会导致程序crash，这种情况常常出现在父类有一个kvo，父类在dealloc中remove 了一次，子类又remove了一次的情况下。不要以为这种情况很少出现！当你封装framework开源给别人用或者多人协作开发时是有可能出现的，而且 这种crash很难发现。不知道你发现没，目前的代码中context字段都是nil，那能否利用该字段来标识出到底kvo是superClass注册 的，还是self注册的？

　　回答是可以的。我们可以分别在父类以及本类中定义各自的context字符串，比如在本类中定义context为 @"ThisIsMyKVOContextNotSuper";然后在dealloc中remove observer时指定移除的自身添加的observer。这样iOS就能知道移除的是自己的kvo，而不是父类中的kvo，避免二次remove造成 crash。

