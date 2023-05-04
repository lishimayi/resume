# OC中的类与元类

## OC 类是一个结构体指针，指向名为objc_class的结构体。

### objc_class的构造为：

```
//结构体指针的定义
typedef struct objc_class *Class;
// 类结构体的构造：
struct objc_class {
	Class isa OBJC_ISA_AVAILABILITY;// isa 是一个class修饰的结构体指针; (此处isa 指针指向元类)
	#if !__OBJC2__
	CLass super_class// 指向父类
	const char *name
	long version
	long info
	long instance_size
	struct objc_ivar_list *ivars
	struct objc_method_list **methodLists
	struct objc_cache *cahe
	struct objc_protocol_list *protocols
	#endif
} OBJC_UNAVAILABLE
```

OC中对象的定义是这样的：

```
typedef struct objc_object {
	Class isa;
} *id;
```

每个对象都属于一个类，在OC中，对象所属的类是isa指针决定的，即isa指针指向对象所属的类

OC对象有一个大家都熟悉的特性：消息发送机制。

```
[@"stringTest", stringByAppendString:@"text"];
```

原理是OC对象在发送消息时候，Runtime库会追寻着对象的isa指针得到对象所属的类（在这里是NSString类）。这个类包含了能能应用于这个类的所有实力方法以及指向父类的指针，以便找到父类的实例方法。Runtime库检查这个类 和其父类的方法列表。找到消息对应的方法。编译器会将消息转换成消息函数objc_msgSend进行调用。

我们平时在代码时候也会对类发消息：

	NSString *testString = [NSString stringWithFormat:@"%d,%s",3,@"test"];
	
可以得知，OC类其实也是个对象，一个对象就要有一个它属于的类，就意味着也要有一个isa指针，指向它所属的类。从消息机制层面来说：

> 当你给对象发消息时，消息是在寻找这个对象的类的方法列表。
> 当你给类发消息时，消息是在寻找这个类的元类的方法列表。

既然元类是个类，和类一样也是个对象。那么元类具体是什么呢？
所有的元类都使用根元类作为他们的类，