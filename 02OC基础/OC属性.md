## OC属性
> 在OC中用`@property`来声明一个属性，其实`@property`是一个语法糖，编译器会自动为实力变量生成`setter`和`getter`方法。

> 所以你在获取这个属性值和设置属性值得时候其实是调用了`getter`和`setter`方法获取和设置实例值
> 
> 一般编译器帮你生成的实例变量就是你的属性名前面加个下划线。当然你也可以直接使用`@synthesize`定义这个实例变量的名称，早起时候实例变量名是需要程序员手动定义的，编译器不会帮你做这件事。只有你自己需要修改一下变量名的时候才需要调用`@synthesize`，如下例子：
 
```
@interface ViewController ()
@property (nonatomic, copy)NSString *name;//声明属性

@end 

@implementation ViewController 
// 声明实例变量名称
@synthesize name = realName;

- (void)viewDidLoad {
	[super viewDidLoad];
	self.name = @"小明";
	NSLog(@"%@",self.name);
	NSLog(@"%@",_realName);// 只能使用这个“_realName”实例变量来操作了
}
// 输出值：
// 小明
// 小明
```
当然我们可以自己自定义`setter`和`getter` 方法，比如：

```
// getter
- (NSString *)name {
	return @"apple";
}
// setter
- (void)setName:(NSString *)name {
	 // 这里的实例变量是_realName，因为之前代码有改动
	_realName = @"banana";
}
```

那么我们根据`self.name`获取跟设置的值都不会变了

```
NSLog(@"%@",self.name);
NSLog(@"%@",_realName);
self.name = @"dasheng";
NSLog(@"%@",self.name);
NSLog(@"%@",_realName);

//输出
apple
(null)
apple
banana
```

## 属性存值
### Runtime下的实现
我们知道“类”在OC中是名为`objc_class`的结构体指针，这个结构体如下：

```
struct objc_class {
	Class isa OBJC_ISA_AVAILABILITY;
	
	#if !__OBJC2__
	
	Class super_class		OBJC2_UNAVAILABLE;
	
	const char *name		OBJC2_UNAVAILABLE;
	
	long version				OBJC2_UNAVAILABLE;
	
	long instance_size		OBJC2_UNAVAILABLE;
	
	struct objc_ivar_list *ivars		OBJC2_UNAVAILABLE;
	
	struct objc_method_list **methodLists		OBJC2_UNAVAILABLE;
	
	struct objc_cache *cache			OBJC2_UNAVAILABLE;
	
	struct objc_protocol_list *protocols		OBJC2_UNAVAILABLE;
	
	#endif
} OBJC2_UNAVAILABLE;
```

我们主要关注其中的

`struct objc_ivar_list *ivars OBJC2_UNAVAILABLE;`

`struct objc_method_list **methodLists OBJC2_UNAVAILABLE;`

`struct objc_protocol_list *protocols OBJC2_UNAVAILABLE;`

这几部分。

`objc_ivar_list`是该类的成员变量列表，我们上面说过的属性其实就是帮你生成了一个`setter`和`getter`方法，最后方法里操作的那个成员变量其实就是存储在这个成员变量链表里面的。而`setter`和`getter`方法也就存储在`objc_method_list`里面。

下面的`ivars`和`methodLists`存储的指针对应的结构体：

```
struct objc_ivar {
	char *ivar_name 			OBJC2_UNAVAILABLE;
	char *ivar_type			OBJC2_UNAVAILABLE;
	int ivar_offset			OBJC2_UNAVAILABLE;
#ifdef __LP64__
	int space				OBJC2_UNAVAILABLE;
#endif
}
```

```
struct objc_method {
	SEL method_name;
	char *method_types; /* a string representing argument / return tyoes */
	IMP method_imp;
}
```

所以说整个属性的生成过程就是 runtime中分为以下几步：

1. 创建该属性，设置其objc_ivar ,通过偏移量和内存占用就可以方便获取。
2. 生成其`setter`和`getter`。
3. 将属性的`ivar`添加到`ivar_list`中，作为类的成员变量存储。
4. 将`setter`和`getter`加入类的`method_list`中。之后可以通过直接调用或者点语法来使用。
5. 将属性的描述添加到类的属性描述列表中。

## 获取成员变量和属性

当然C/C++ 中也提供了对应的函数可以去到成员变量和属性。

```
// 获取整个成员变量的链表
Ivar * class_copyIvarList ( Class cls, unsigned int *count );

// 获取属性链表

objc_property_t * class_copyPropertyList ( Class cls, unsigned int *outCount );
```

`class_copyIvarList`函数，返回一个指向成员变量信息的数组，数组中的每个元素都是指向该成员变量信息的`objc_ivar`结构体指针 （只是class_copyPropertyList）。这个数组不包含父类中声明的变量，`outCount`指针返回数组的大小。需要注意的是，我们必须使用`free()`来释放这个数组。

举个例子：

```
@interface Person: NSObject {
	NSString *name;
}

@property (nonatomic, copy) NSString *age;

@end

unsigned int count = 0;
Ivar *members = class_copyIvarList([Person class], &count);
for (int i = 0; i < count; i++) {
	Ivar ivar - members[i];
	const char *memberName  = ivar_getName(ivar);
	NSLog(@"变量名 = %s", memberName);
}
free(members);

objc_property_t *properties = class_copyPropertyList([Person class], &count);
for (int i = 0 ; i < count; i++) {
	objc_property_t property = properties[i];
	const char * char_f = property_getName(property);
	NSString *propertyName = [NSString stringWithUTF8String:char_f];
	NSLog(@"属性名 = %@", propertyName);
}
free(propertires);
```

```
// 输出

变量名 = name
变量名 = _age
属性名 = age
```

给`atomic` 属性写`setter`和`getter`方法,要点就是需要加锁：

```
@property (atomic, copy) NSString *name;

- (NSString *)name {
	NSString *name;
	@synchronized (self) {
		name = _name;
	}
	return name;
}

- (void)setName:(NSString *)name {
	@synchronized(self) {// 加锁同步
		if (![_name isEqualToString:name]) {
			_name = name;
		}
	}
}
```

