# iOS Runtime机制的详解

## 前要

将源代码转换为可执行程序需要3步：编译·链接·运行。不同的编译语言在这个三个步骤中锁进行的操作有所不同。

## 1. 什么是Runtime

Runtime是用C和汇编编写的用于实现OC动态语言机制的开源库。Runtime简称运行时，就是系统在运行的时候一些机制。为我们提供了在程序在运行时动态创建和检查对象，修改类和对象的方法。

## 2. OC与Runtime的交互层级

OC与Runtime系统在三个层级上进行不同的交互。

* Runtime与OC的源代码交互。
* Runtime与Foundation框架的NSObject类定义的方法。
* Runtime的函数直接被调用。

大部分时间开发者只需要专注于OC代码就可以，Runtime系统自动在幕后运作。

## 3. 静态类型语言与动态类型语言

* 静态类型语言：变量的数据类型在编译时就可以确定的语言，多数静态类型的语言要求在使用变量之前必须声明类型。C，C++，Java，C# 都属于静态类型语言。
* 动态类型语言：变量的数据类型在运行时确定的语言，变量在使用之前不需要变量类型声明，通常的变量类型是被赋值的那个变量的类型。Python，ruby，OC，js这些都是动态类型的语言。

## 4. C 和OC的函数调用对比

* C函数的调用在编译的时候会决定会调用哪个函数，编译完之后直接顺序执行，无任何二义性。
* OC函数的调用通过消息发送，编译时并不能决定真正调用哪个函数（在编译阶段OC可以调用任何函数，即使这个函数并未实现，只要声明过就不会报错，而C语言会报错），只有在真正运行的时候才会根据函数的名称找到具体对应函数来调用。

## 5. Runtime的具体实现

我们写得OC代码，在运行时候也是转换成了Runtime方式运行的。更好的去了解Runtime能够帮我们更深入的掌握OC语言。每一个OC方法，底层必然有一个与之对应的Runtime方法。

```
// 当我们写下这样的代码
[tableView cellForRowAtIndexPath:indexPath];
// 在编译时，Runtime会将上述代码转换成【发送消息】
objc_msgSend(tableView, @selector(cellForRowAtIndexPath:),indexPath);
```

## 6. 常见Runtime方法

### 获取属性列表

```
objc_property_t *propertyList = class_copyPropertyList([self class], &count) {
	for (unsigned int i = 0; i < count; i++) {
		const char *propertyName = property_getName(propertyList[i]);
		NSLog(@"property ----->%@",[NSString stringWithUTF8String:propertyName]);
	}
}
```

### 获取方法列表

```
Method *methodList = class_copyMethodList([slef class], &count) ;
for (unsigned int i = 0; i < count; i++ ) {
	Method method = methodList[i];
	NSLog(@"method ----->%@",NSStringFromSelector(method_getName(method)));
}
```

### 获取成员变量列表 

```
Ivar *ivarList = class_copyIvarList([self class], &count);
for (unsigned int i = 0; i < count; i++ ) {
	Ivar myIvar = ivarList[i];
	const char *ivarName = ivar_getName(myIvar);
	NSLog(@"Ivar -----> %@",[NSString stringWithUFT8String:ivarName]);
}
```

### 获取协议列表

```
__unsafe_unretained Protocol **protocolList = class_copyProtocolList([self class], &count);
for (unsigned int i; i<count; i++)  {
	Protocol *myProtocol = protocolList[i];
	const char *protocolName = protocol_getName(myProtocol);
	NSLog(@"protocol -----> %@", [NSString stringWithUTF8String:protocolName]);
}
```

> 现在有一个Person类，和Person类创建的xiaoming对象，和test1 和test2方法。

#### 获得类方法

```
Class PersonClass = object_getClass([Person class]);

SEL oriSEL = @selectot(test1);
Method oriMethod = class_getInstanceMethod([xiaoming Class], oriSEL);
```

#### 获得实例方法

```
Class PersonClass = object_getClass([xiaoming class]);
SEL oriSEL = @selectot(test2);
Method cusMethod = class_getInstanceMethod([xiaoming class], oriSEL);
```

#### 添加方法

```
BOOL addsucc = class_addMethod(xiaomingClass, oriSEL, method_getImplementation(cusMethod), method_getTypeEncoding(cusMethod));
```

#### 替换原方法实现

```
class_replaceMethod(toolClaa, cusSEL, method_getImplementation(oriMethod), method_getTypeEncoding(oriMetod));
```

#### 交换方法

```
method_exchangeImplementations(oriMethod, cusMethod);
```

###  常规作用
* 动态添加对象的成员变量和方法
* 动态的交换两个方法的实现
* 拦截替换方法
* 在方法上增加额外功能
* 实现NSCoding的自动归档和解档
* 实现字典模型的自动转换

### 代码实现

若要使用Runtime，需要先引入头文件`import <objc/Runtime.h>`

#### 动态变量控制

在程序中xiaoming的age是10，后来被Runtime修改成了20，看下怎么做到的。

1. 动态获取xiaoming 类中的所有属性包括私有属性。
	
	`Ivar *ivar = class_copyIvarList([self.xiaoming class], &count);`
2. 遍历属性找到对应的name

	`const char *varName = ivar_getName(var);`
	
3. 修改对应字段值为20

	`object_setIvar(self.xiaoMing, var, @"20");`
4. 参考代码

```
- (void)answer {
	unsigned int count = 0;
	Ivar *ivar = class_copyIvarList([self.xiaoMing class], &count);
	for (int i = 0; i < count; i++) {
		Ivar var = ivar[i];
		const char *varName = ivar_getName(var);
		if ([name isEqualToString:@"_age"]) {
			objc_setIvar(self.xiaoMing, var , @"20");
			break;
		}
	}
	NSLog(@"xiao ming's age is %@", self.xiaoMing.age);
}
```
#### 动态添加方法

在程序中假设XiaoMing没有guess方法，后来被Runtime添加了一个叫guess的方法，最终在调用guess方法做出响应。那么Runtime如何做到的呢？

* 动态给XiaoMing类中添加guess方法：
```
/*
 * (IMP)guessAnswer 意思是guessAnswer的地址指针
 * "v@:" v:代表返回值void，如果是i代表int，@：代表id sel，“:”代表SEL_cmd
 * “v@:@@” 意思是，两个参数的没有返回值。****
 */
class_addMethod([self.xiaoMing class], @selector(guess), (IMP)guessAnswer, "v@:");
```

* 调用guess方法的响应时间：

```
[self.xiaoMing performSelector:@selector(guess)];
```
* 编写guessAnswer的实现：

```
// void 前面没有 + - 号，因为是C代码
// 必须有两个指定参数 id self , SEL_cmd
void guessAnswer(id self, SEL_cmd) {
	NSLog(@"i am from beijing");
}
```
* 参考代码

```
- (void)answer {
	class_addMethod([self.xiaoMing class], @selector(guess), (IMP)guessAnswer, "v@:");
	if ([self.xiaoMing respondsToSelector:@selector(guess)]) {
		[self.xiaoming performSelector:@selecttor:(guess)];
	} else {
		NSLog(@"there is no guess func");
	}
}

void guessAnswer(id self, SEL_cmd) {
	NSLog(@"i am from beijing");
}
```

#### 动态交换两个方法的实现
在程序中，假设XiaoMing类中有test1 和test2 这两个方法，如何使用Runtime对2个方法的调用和实现相互调换？

* 获取这个类中的两个方法并互换

```
Method m1 = class_getInstanceMethod([self.xiaoMing class], @selector(test1));
Method m2 = class_getInstanceMethod([self.xiaoMing class], @selector(test2));
method_exchangeImplementations(m1, m2);// 交换完成，
```

#### 拦截并替换方法

在程序中，假设XiaoMing类有test1方法。但是出于某种原因我们改变这个方法的实现，但又不能去动它的源码，这个时候Runtime就出现了。

* 我们先新增一个Tool类，然后自己实现一个 change方法。通过Runtime吧tes1 替换成change。

```
Class PersonClass = object_getClass([Person class]);
Class ToolClass = object_getClass([Tool class]);

// 原方法的SEL和Method
SEL oriSEL = @selector(test1);
Method oriMethod = class_getInstanceMethod(PersonClass, oriSEL);
//交换SEL和Method
SEL cusSEL = @selector(change);
Method cusMethod = class_getInstanceMehtod(ToolClass, cusSEL);
// 先尝试给原方法添加实现，这里为了避免原方法未实现的情况
BOOL addSucc = class_addMethod(PersonClass, oriSEL, method_getImplementation(cusMethod), method_getTypeEncoding(cusMethod));

if (addSucc) {
	// 添加成功：将原方法的实现替换到交换方法的实现
	class_replaceMethod(ToolClass, cusSEL, method_getImplementation(oriMethod), method_getTypeEncoding(oriMethod));
}else {
// 添加失败：说明原方法已经实现，之间替换两个方法即可。
method_exchangeImplementations(oriMethod, cusMethod);
}
```

#### 在现有方法上增加额外功能

有这样一个场景，出于某些需求，我们需要跟踪记录app中按钮的点击次数和频率，如何解决？当然通过集成按钮类或者通过类别实现是一个方法，但是会带来其他问题比如，别人不一定实例化你的子类，或者其他类别也实现了点击方法导致不确定会调用哪一个，Runtime这样解决。

```
@implementation UIButton (Hook)

+ (void)load {
	static dispatch_once_t onceToken;
	dispatch_once(&onceToken, ^{
		Class selfClass = [self class];
		
		SEL oriSEL = @selector(sendAction:to:forEvent:);
		Method oriMethod = class_getInstanceMethod(selfClass, oriSEL);
		
		SEL cusSEL = @selector(mySendAction:for:Event:);
		Method cusMethod = class_getinstanceMethod(selfClass, cusSEL);
		
		BOOL addSucc = class_addMethod(selfClass, oriSEL, method_getImplementation(cusMethod), method_getTypeEncoding(cusMethod));
		if (addSucc) {
			class_replaceMehtod(selfClass, cusSEL, method_getImplementation(oriMethod), method_getTypeEncoding(oriMethod));
		}else {
			method_exchangeImplementations(oriMethod, cusMethod);
		}
	})
}

- (void)mySendAction:(SEL)action to:(id)target forEvent:(UIEvent *)event {
	[CountTool addClickCount];
	[self mySendAction:action to:target forEvent:event];
}

@end
```

load方法会在类第一次加载的时候调用，调用的时间比较靠前，适合在这里做方法交换，在程序中只会执行一次。

#### 实现NSCoding的自动归档和解档

如果你实现过自定义模型数据持久化过程，那么你肯定明白，如果一个数据模型有很多属性，那么我们需要对每个属性实现一边encodeObject和decodeObjectForKey方法，如果这样的模型又多了很多个，这还真是一个十分麻烦的事情。接下来看下简单的实现。

```
// 假设现在有一个Movie类，有3个属性
// .h
#import <Foundation/Foundation.h>

// 1. 如果想要当前类可以实现归档和反归档，需要遵守NSCoding协议。
@interface Movie : NSObject<NSCoding>

@property (nonatomic, copy) NSString *movieId;
@property (nonatomic, copy) NSString *movieName;
@property (nonatomic, copy) NSString *pic_url;

@end

// 如果是正常写法。.m文件应该是这样的：
//.m 


#import "Movie.h"
@implementation Movie
 
- (void)encodeWithCoder:(NSCoder *)aCoder
{
    [aCoder encodeObject:_movieId forKey:@"id"];
    [aCoder encodeObject:_movieName forKey:@"name"];
    [aCoder encodeObject:_pic_url forKey:@"url"];
    
}
 
- (id)initWithCoder:(NSCoder *)aDecoder
{
    if (self = [super init]) {
        self.movieId = [aDecoder decodeObjectForKey:@"id"];
        self.movieName = [aDecoder decodeObjectForKey:@"name"];
        self.pic_url = [aDecoder decodeObjectForKey:@"url"];
    }
    return self;
}
@end

// 如果你有100 个属性每个都写一遍岂不是很烦
// 有了Runtime 我们可以简单实现
//.m
#import "Movie.h"
#import <objc/Runtime.h>

@implementation Movie 

- (void)encodeWithCoder:(NSCoder *)encoder {
	unsigned int count = 0;
	Ivar *ivars =  class_copyIvarList([Movie class], &count);
	
	for (int i = 0 ; i < count ; i++) {
		// 取出i位置的成员变量，
		Ivar ivar = ivars[i];
		// 查看成员变量
		const cahr *name = ivar_getName(ivar);
		// 归档
		NSString *key = [NSString stringWithUTF8String:name];
		id value = [self valueForKey:key];
		[encoder encdeObject:value forKey:key];
	}
	free(ivars);
}

- (id)initWithCoder:(NSCoder *)decoder {
	if (self = [super init]) {
        unsigned int count = 0;
        Ivar *ivars = class_copyIvarList([Movie class], &count);
        for (int i = 0; i<count; i++) {
        // 取出i位置对应的成员变量
        Ivar ivar = ivars[i];
        // 查看成员变量
        const char *name = ivar_getName(ivar);
       // 归档
       NSString *key = [NSString stringWithUTF8String:name];
      id value = [decoder decodeObjectForKey:key];
       // 设置到成员变量身上
        [self setValue:value forKey:key];
            
        }
        free(ivars);
    } 
    return self;

}
@end
```

这样的方式实现，不管有多少个属性，几行代码就搞定了。怎么，还嫌麻烦，下面是更简单的方法:

```
// 我们把encodeWithCoder和initWithCoder 这两个方法抽成宏
#import "Movie.h"
#import "objc/Runtime.h"

#define encodeRuntime(A)\
\
unsigned int count = 0;\
Ivar *ivars = class_copyIvarList([A class], &count);\
for (int i = 0; i < count; i++) {\
Ivar ivar = ivars[i];\
const char *name = ivar_getName(ivar);\
NSString *key = [NSString stringWithUTF8String:name];\
id value = [self valueForKey:key];\
[encoder encodeObject:value forKey:key];\
}\
free(ivars);\
\
#define initCoderRuntime(A)\
\
if (self = [super init]) {\
unsigned int count = 0;\
Ivar *iars = class_copyIvarList([A class], &count) ;\
for (int i = 0; i < count; i++ ) {\
Ivar *iavr = ivars[i];\
const char *name = ivar_getName(ivar);\
NSString *key = [NSString stringWithUTF8String:name];\
id value = [decoder decodeObjectForKey:key];\
[self setValue:value ForKey:key];\
}\
free(ivars);
}\
return self;\
\

@implementation Movie

- (void)encodeWithCoder:(NScoder *)encoder {
	encoderRuntime(Movie);
}

- (id)initWithCoder:(NSCoder *)decoder {
	initCoderRuntime(Movie);
}
@end
// 这样我们吧两个单独放到文件里面，以后需要持久化数据模型就只调用这两个宏
```

#### 实现字典和模型的自动转换

字典转模型的应用可以说是每个APP都需要使用的场景，虽然方式策略各有不同，但是原理都是一致的，遍历模型中的所有属性，根据模型的属性名去字典中查找key，取出对应的值给模型属性赋值。例如：JSONModel，MJExtension都是通过这种方式。

* 先实现最外层的属性转换

```
// 创建对应模型对象
id objc = [[self alloc] init];
unsigned int count = 0;
// 1 获取成员变量属性组
Ivar *ivarList = class_copyIvarList(self, &count);
// 2 遍历所有成员属性名，逐个去字典中取出相应value给模型属性赋值
for (int i = 0; i < count; i++) {
	Ivar ivar = ivarList[i];
	const char *name = ivar_getName(ivar);
	NSString *ivarName = [NSString stringWithUTF8String:name];
	// _成员属性名 转换成字典的key
	NSString *key = [ivarName subStringFromIndex:1];
	// 字典取值
	id value = dict[key];
	// 获取成员属性类型
	NSString *ivarType = [NSString stringWithUTF8String:ivar_getTypeEncoding(ivar)];
}
```

如果模型比较简单，只有NSString和NSNumber等，这样就可以搞定了。但是如果模型含有NSArray，或者NSDictionary等。我们需要进行二步转换。

* 内层数组字典转换

```
if ([value isKindeClass:[NSDictionary class]] && ![ivarType containsString:@"NS"])  {
	// 是字典对象，并且属性名对应的类型是自定义类型
	// 处理字符串@\"User\" - > User
	invarType = [ivarType stringByReplaceingOccurrencesOfString:@"@" withString:@""];
	ivarType = [ivarType stringByReplaceingOccurrencesOfString:@"\" withString:@""];
	// 自定义对象并且值是字典
	// value : user字典-> User模型
	// 获取模型（user）类对象
	Class modelClass = NSClassFromString(ivarType);
	
	// 字典转模型
	if (modelClass) {
		value = [modelClass objectWithDict:value];
	}
}

if ([value isKindOfClass:[NSArray class]]) {
	// 判断对应类有没有实现字典数组转模型数组的协议
	if ([self respondsToSelector:@selector(arrayContainModelClass)]) {
		// 转换成id类型，就能调用任何对象方法
		id idSelf = self;
		// 获取数组中字典对应的模型
		NSString *type = [idSelf arrContainModelClass][key];
		// 生成模型
		Class classModel = NSClassFromString(type);
		NSMutableArray *arrM = [NSMutableArray array];
		// 遍历字典数组， 生成模型数组
		for (NSDictionary *dict in value) {
			// 字典转模型
			id model = [classModel objectWithDict:dict];
			[arrM addObject:model];
		}
		value = arrM;
	}
}
```

我觉得系统自带的KVC模式字典转模型就挺好，假设Movie就是一个模型对象，dict是一个需要转化的[movie setValuesForKeysWithDictionary:dict]; 这个是系统自带的字典转模型的方法。不过市使用这个方法的时候需要在模型里再实现一个方法才行：
`- (void)setValue:(id)value forUndefinedKey:(NSString *)key`,重写这个方法是为了实现两个目的：

1. 模型中的属性和字典中的key不一致的情况，比如字典中的id，我们需要把他赋值给uid属性。
2. 字典中属性比模型中属性还多的情况。

如果出现上面两种情况而没有实现下面这个方法，程序会崩溃：

```
- (void)setValue:(id)value forUndefinedKey:(NSString *)key {
	if ([key isEqualToString:@"id"]) {
		self.uid = value;
	}
}
```

## 7.集中参数概念

以上集中方法应该算是Runtime中在实际场景中应用的大部分情况了，平常编码差不多足够。
如果从头到尾仔细阅读，相信你用法应该会了，虽然用是主要目的，有几个基本的参数概念还是要了解一下的，

#### 1.objec_msgSend

```
/* Basic Messaging Primitives
 * On some architectures, use objc_msgSend_stret for some struct return types.
 * On some architectures, use objc_msgSend_fpret for some float return types.
 * On some architectures, use objc_msgSend_fp2ret for some float return types.
 * 
 * These functions must be cast to an appropriate fucntion pointer type
 * before being called
 */
```

这是官方声明，从这个函数可以看出来，这是个最基本的用于发消息的函数。另外，这个函数并不能发送所有的消息类型，只能发送基本的消息。比如，在一些处理器上，我们必须使用`objc_msgSend_stret`来发送返回值类型为结构体的消息，使用`objec_msgSend_fpret`来发送返回值类型是浮点型的消息，而又在一些处理器上，还得使用`objc_msgSend_fp2ret`来发送返回值类型为浮点类型的消息。

关键一点：无论何时，要调用`objc_msgSend`函数，必须要将函数强制转换成何时的函数指针类型才能调用。

从objc_msgSend函数的声明来看，他应该是不带返回值的，但是我们在使用中可以强制转换类型。一边接收返回值，另外，它的参数是可以任意多个的，前提是也要强制函数指针类型。

其实，编译器会根据objc_msgSend, objc_msgSend_stret, objc_msgSendSuper或obc_msgSendSuper_strect 四个方法中选择一个调用，如果消息是传递超类，那么会调“super”函数，如果消息返回值是结构体而不是简单值，那么会调用名字带有"stret"的函数

#### 2. SEL
objec_msgSend函数第二个参数是SEL它是selector在Objc中的表示类型（Swift中是Selector类）。selector是方法选择器，可以理解为区分方法的 ID，而这个 ID 的数据结构是SEL:
typedef struct objc_selector *SEL;
其实它就是个映射到方法的C字符串，你可以用 Objc 编译器命令@selector()或者 Runtime 系统的sel_registerName函数来获得一个SEL类型的方法选择器。
不同类中相同名字的方法所对应的方法选择器是相同的，即使方法名字相同而变量类型不同也会导致它们具有相同的方法选择器，于是 Objc 中方法命名有时会带上参数类型(NSNumber一堆抽象工厂方法)，Cocoa 中有好多长长的方法哦。

#### 3. id

objc_msgSend第一个参数类型为id，大家对它都不陌生，它是一个指向类实例的指针：
typedef struct objc_object *id;
那objc_object又是啥呢：
struct objc_object { Class isa; };
objc_object结构体包含一个isa指针，根据isa指针就可以顺藤摸瓜找到对象所属的类。
PS:isa指针不总是指向实例对象所属的类，不能依靠它来确定类型，而是应该用class方法来确定实例对象的类。因为KVO的实现机理就是将被观察对象的isa指针指向一个中间类而不是真实的类，这是一种叫做 isa-swizzling 的技术，详见官方文档.

#### 4.class 

之所以说isa是指针是因为Class其实是一个指向objc_class结构体的指针：
typedef struct objc_class *Class;
objc_class里面的东西多着呢：

```
struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;
#if  !__OBJC2__
    Class super_class                                        OBJC2_UNAVAILABLE;
    const char *name                                         OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE;
    struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache *cache                                 OBJC2_UNAVAILABLE;
    struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE;
#endif
 
} OBJC2_UNAVAILABLE;

```

可以看到运行时一个类还关联了它的超类指针，类名，成员变量，方法，缓存，还有附属的协议。
在objc_class结构体中：ivars是objc_ivar_list指针；methodLists是指向objc_method_list指针的指针。也就是说可以动态修改 *methodLists的值来添加成员方法，这也是Category实现的原理.

## 8.消息机制的基本原理

### 前要
OC中方法的调用：`[object selector];`,其本质是让对象在运行时发送消息的过程。

### 我们来看看[object selector]; 在`编译阶段`和`运行阶段`分别做了什么？

#### 1. 编译阶段：`[object selector];`方法被编译器转换为：

1. `objc_msgSend(object. selector)`(不带参数)
2. `objc_msgSend(object. selector, org1, org2, ...)`(带参数)

#### 2. 运行时阶段：消息接受者`object` 寻找对应的`selector`

1. 通过`object`的`isa 指针`找到`object`的`Class （类）`
2. 在`Class （类）`的`cache （方法缓存）`的散列表中寻找对应的`IMP(方法实现）`
3. 如果在`cache(方法缓存）`中没有找到对应的`IMP(方法实现)`的话，就继续在`Class(类)`的`method list`中找对应的`selector`,如果找到了，填充的`cache（方法缓存）`中，并返回`selector`;
4. 如果在Class类中没有找到Selector，就继续在它的superClass中寻找；
5. 一旦找到对应的selector，直接执行receiver对应的selector方法实现的IMP。
6. 若找不到对应的selector，消息被转发或者临时向recevier添加这个selector对应的实现方法，否则就会发生crash。


### 消息重定向

```
/*
*  如果经过消息动态解析、消息接受者重定向，
*  Runtime系统还是找不到相应的方法实现而无法响应消息，
*  Runtime会利用 - methodSignatureForSelector: 
*  或者 + methodSignatureForSelector: 方法回去函数的参数和返回值类型。
*/

//  如果 methodSignatureForSelector: 返回了一个NSMethodSignature对象（函数签名），
// Runtime系统就会创建一个NSInvocation对象，并通过forwardInvocation:消息通知当前对象
// 给与此次消息发送最后一次寻找IMP 的机会。

// 如果methodSignatureForSelector: 返回nil。则Runtime系统会发出
// doesNotRecognizeSelector:消息，程序也就会崩溃。

// 所以我们可以在forwardInvocation:方法中对消息进行转发。

// 类方法调用
// 获取类方法函数的参数和返回值类型，返回值是签名
+ (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector;

// 类方法消息重定向
+ (void)forwardInvocation:(NSInvocation *)anInvocation;
+ doesNotRecognizeSelector:

// 对象方法调用
// 获取类方法函数的参数和返回值类型，返回值是签名
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector;

// 对象方法消息重定向
- forwardInvocation:
- doesNotRecognizeSelector:
```

例子：

```
#import "ViewController.h"
#include "objc/Runtime.h"

@interface Person : NSObject

- (void)fun;

@end

@implementation Person

- (void)fun {
	NSLog(@"is fun");
}

@end

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidload {
	[super viewDidLoad];
	// 执行fun函数
	[self performSelector:@selector(fun)];
}

+ (BOOL)resovleInstanceMethod:(SEL)sel {
	return YES;// 为了进行下一步接受消息重定向
}

- (id)forwardingTargetForSelector:(SEL)aSelector {
	return nil;// 为了进行下一步，消息重定向
}

// 获取函数的参数和返回值类型，返回签名。
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector {
	if ([NSStringFromSelector(aSelector) isEqualToString:@"fun"]) {
		return [NSMethodSignature signatureWithObjCTypes:@"v@:"];
	}
	return [super methodSignatureForSelector:aSelector];
}

// 消息重定向
- (void)forwardInvocation:(NSInvocation *)anInvocation {
	SEL sel = anInvocation.selector;// 从anInvocation 中获取消息
	Person *p = [[Person alloc] init];
	
	if ([p respondsToSelector:sel]) {// 判断person 对象方法是否可以响应sel
		[anInvocation invokeWithTarget:p];// 若可以响应，则将消息转发给其他对象处理。
	}else {
		[self doeNotRecognizeSeletor:sel];// 若仍然无法响应则报错；找不到响应方法。
	}
}
```

> 打印结果：“is fun”

可看到，我们在 - forwardInvocation: 方法里让Person对象去执行了fun函数。 


## 9. isa详解

* 要想学习Runtime，首先要了解它底层的一些常用数据结构，比如isa指针
* 在arm64架构之前，isa就是一个普通的指针 ，存储着Class、Meta-Class对象的内存地址
* 从arm64架构开始 ，对isa进行了优化，变成了一个共用体 (union）结构，还使用位域来存储更多的信息

## 10. 面试题：
 
* `isKindOfClass:` 和 `isMemberOfClass:` 这两个方法有什么区别？

