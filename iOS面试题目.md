# iOS面试题目集锦

### 1. Swift和OC的区别

```
一，区别

1，最明显的区别：OC一个类由.h和.m两个文件组成，而swift只有.swift一个文件，所以整体的文件数量比OC有一定减少。

2，不像C语言和OC语言一样都必须有一个主函数main()作为程序的入口，swift程序从第一句开始向下顺序执行，一直到最后。



OC的main函数：

 int main(int argh, char * argh[]) {

 @autoreasepool {

   return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));

   }

 }

3，swift数据类型都会自动判断，只区分变量var和常量let。

4，关于BOOL类型更加严格，swift不再是OC的非0就是真，而是true才是真false才是假。

5，swift的switch语句后面可以跟各种数据类型了，如Int，字符串，元组，区间都行，并且里面不用写break。

6，swift 中可以定义不继承于其它类的类，称之为基类(base calss)。

7，final关键字

使用final关键修饰的类，不能被其他类继承。

继承final修饰的类会报错：Inheritance from a final class '…..'

8，类方法修饰符：static

9，guard关键词

注意事项：

1.guard关键字必须使用在函数中。

2.guard关键字必须和else同时出现。

3.guard关键字只有条件为false的时候才能走else语句 相反执行后边语句。

用处：

判断某个参数是否符合要求，不符合直接返回。省去过多的if-else语句。

10，in out关键词

in-out是修饰函数参数类型,表示该参数在函数内修改后(即函数返回后),其值为修改后的值.

1,适用类型为变量

2,in-out修饰后的参数,在传参时需&修饰

作者：liang1030
链接：https://www.jianshu.com/p/ab543aaecf50
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

### 2. 编译连接

```
程序要运行起来，必须要经历四个步骤： 
	1 预处理
	2 编译
	3 汇编
	4 连接
hello.c -(-E 预处理)-> hello.i -(-s 编译)->hello.s-(-c 汇编)->hello.o-(连接)->可执行文件（exe）
```
### 3. OC语言（面向对象）三大特性：

```
// 1封装
即隐藏对象的属性和实现细节，仅对外公开接口，控制在程序中属性的读和修改的访问级别；
将抽象得到的数据和行为（或功能）相结合，形成一个有机的整体，也就是将数据与操作数据的源代码进行有机的结合，形成“类”，其中数据和函数都是类的成员。
// 2继承

// 3多态
1. 多态在代码中的体现，即为多种形态，必须要有继承，没有继承就没有多态。
2. 在使用多态时，会进行动态检测，以调用真实的对象方法。
3. 多态在代码中的体现即父类指针指向子类对象。
```

### 4.@synthesize & @dynamic

```
@synthesize 
// 会自动生成setter 和getter方法
@dynamic 
// 需要手动生成setter 和getter方法
```

### 5. UITableView & UICollection

### 6. NSProxy & NSObject

https://www.jianshu.com/p/20c441f19126

### 7. 多继承的实现和区别

https://www.jianshu.com/p/9601e84177a3

### 8. 传值通知和推送通知（本地和远程）

### 9. `NSCache 和 NSDictionary`

## Category

### 1. Category的使用场景是什么？
### 2. Category的实现原理？
 * category编译过后的底层结构是 struct category_t ,里面存储着分类的对象方法，类方法，属性和协议信息
 * 在程序运行的时候，runtime会将Category的数据合并到类信息中（类对象，元类对象中）

### 3. Category和Class Extension的区别是什么？
 * Class Extension在编译的时候，它的数据就已经包含在类信息中
 * Category是在运行的时候，数据才会合并到类信息中（类对象，元类对象中）

### 4. Category中有load方法吗，load方法什么时候调用，load方法能继承吗？
* Category中有load方法
* load方法是在runtime加载类和分类时候，先调用所有类的load，父类先于子类；然后调用所有分类的load，分类load顺序取决于编译顺序。
* load方法可以被继承，但是一般不主动调用load方法，都是让系统自动取调用

### 5. load方法和initialize方法有什么区别，他们在Category中的调用顺序是什么，以及出现继承的时候他们之间的调用过程？
 * 调用方式：load是直接根据函数地址调用，initialize是消息机制
 * 调用时机：load在加载时候调用，initialize会在类第一次接收到消息的时候调用（父类的initialize可能会被调用多次）、
 * initialize会走消息机制，有分类就，会被分类的initialize覆盖，有父类，会优先调用父类

### 6. Category能否添加成员变量？如果可以，如何给Category添加成员变量？
 * 不能直接给Category能添加成员变量，但是可以间接添加
 * Category底层实现的结构体中包含属性列表，说明我们可以在Category中声明属性，不过在声明属性的时候，只会有set和get方法的声明，不会生成成员变量和实现。
 * 可以使用runtime的关联对象，给Category间接添加成员变量：‘objc_setAssociateObject(id _Nonnull object, const void * _Nonnull key, id _Nonnull value, objc_AssociationPolicy policy)’

### 7.关联对象原理

* 实现关联对象技术的核心对象有
	* AssociationsManager
	* AssociationsHashMap
	* ObjectAssociationMap
	* ObjcAssosiation

## Block

* block的原理？本质是什么？
	* block的原理
	* block本质是OC对象，有ISA指针，封装了函数调用以及函数调用环境。（函数调用环境就是，参数，外面访问了值）
* __block的作用是什么？有什么使用注意点？
	* __block会将auto变量封装成对象
	* 
* block的属性修饰符为什么是copy？使用block有哪些注意事项？
* block在修改NSMutableArray，是否需要添加__block?

### 1. block的变量捕获机制

变量类型         | 捕获到block内部          | 访问方式    |
--------------------|------------------|-----------------------|
局部变量：auto |√   | 值传递  
局部变量：static| √| 指针传递 

### 2. block类型

* 没有访问auto变量的block是全局block
* 访问了auto变量的block是stack block
* stack block 进行了copy操作就是malloc block
* arc下会根据情况将block自动copy，导致访问auto的block自动copy就变成了mallocblock

### 3. 每种block调用copy的结果

block类型         | 副本源的配置存储域        | 复制效果
--------------------|------------------|-----------------------|
_NSConcreteStackBlock |栈  | 从栈复制到堆  
_NSConcreteGlobalBlock  | 程序的数据区| 什么也不做
_NSConcreteMallocBlock |堆 |引用计数器加一

### 4. block内部访问了对象类型的auto变量

* 如果block是在栈上的，不管指针是MRC还是ARC，不会对auto变量产生强引用。
* 如果block在堆上
	* 会调用block内部的copy函数
	* copy函数内部会调用`_Block_object_assign`函数
	* `_Block_object_assign`会根据auto变量的修饰符（__strong, __weak , __unsafe_unretained）做出相应的操作，类似于retain操作（弱引用，强引用）。
* 如果block从堆上移除
	* 会调用block内部的`dispose`函数
	* `dispose`函数内部会调用`_Block_object_dispose`函数
	* `_Block_object_dispose`函数会自动释放引用的auto变量，类似于release

函数         | 调用时机
--------------------|------------------------|
copy函数 | 栈上的函数复制到堆上
dispose函数  |堆上的block被废弃的时候

### 5. __block修饰符
* __block可以解决block内部无法修改auto变量值的问题
* __block不能修饰全局变量和静态变量（static）
* 编译器会将

