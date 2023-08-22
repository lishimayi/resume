# iOS属性问题

## 属性修饰符的作用
iOS5之前是MRC, 内存需要程序员管理，iOS之后是ARC，除非特殊情况（例如：使用C框架或者循环引用）不需要程序员手动管理内存。

iOS中当我们定义属性`@property`的时候，就需要属性修饰符，

### 主要属性修饰符
1. copy
2. assign
3. retain
4. strong
5. weak
6. readwrite/readonly（读写策略，访问权限）
7. nonatomic/atomic（安全策略）

### 根据MRC和ARC划分属性修饰符的使用范围
```
//MRC：nonatomic,atomic,retain,assign,copy,readwrite,readonly
//ARC：nonatomic,atomic,strong,weak,assign,copy,readwrite,readonly
```
### 什么影响retainCount计数
```
1. alloc方法是为了对象分配内存，retaincount 为1
2. retainCount ：引用计数，下面简称计数
3. release 对象计数 -1
4. retain 计数 +1。
5. copy 一个对象会变成一个新的对象，这个对象的计数为1，原有的对象计数不变。 
```
不管MRC还是ARC，对象的释放都依据`reference count`是否为0,
### 修饰符详述

* copy：
	1.  一般用于修饰不可变容易的属性（NSArray,NSDictionary,NSString,block）
	2. MRC和ARC均可用
	3. 其setter方法与retain处理流程一样，先旧值release再copy出新的对象
	4. copy修饰block是在MRC和ARC的区别：
		* MRC环境下
			* block访问外部局部变量，block存放在栈里面。
			* 只要block访问整个app都存在的变量，那么肯定是全局区
			* 不能使用retain引用block，因为block不在堆里面，只有使用copy才会把block放在堆里面
		* ARC环境下
			* 只要block访问外部局部变量，block就会存放到堆里
			* 可以用strong去引用，因为本身已经存放到堆区了
			* 也可以使用copy进行修饰，但是strong性能更好。
	5. 使用block时候注意循环引用，造成无法释放，内存泄漏。
	6. 在NSString属性中
	  	* 如果外部赋值是NSString，那么用strong和copy都没有问题
	  	* 但是如果外部赋值的是NSMutableString，NSString指针可以持有NSMutableString对象。如果用strong修饰，那么外部的值变化了，里面的值也会变化，这是因为指向的是同一个内存地址
    如果用copy修饰，那么外部的值变化了，里面的值也不会变化，因为对对象的内存做了深度拷贝，复制了一份内存，指针的指向已经变化了
   

* assign：
	1. MRC和ARC均可用。
	2. 一般用来修饰基础数据类型（NSInteger, CGFloat）和C数据类型（int,float,double）等，他们的setter方法直接赋值，不进行任何retain操作。
* retain：
	1. 在MRC下使用，被其修饰的对象，retainCount要+1
	2. retain只能修饰OC对象啊，不能修饰非OC对象，
	3. 一般修饰非NSString的NSObject类及其子类。
* strong：
	1. strong表示对对象的强引用。计数会+1
	2. ARC环境下也可以用来修饰block，（strong和weak两个修饰符默认是strong）。
	3. 用于指针变量，setter方法对参数进行release旧值再retain新值。
	4. 两个对象之间相互强引用会造成循环引用，内存泄漏。
* weak
	1. weak表示对象的弱引用，被其修饰的对象随时可以被系统销毁回收。不会使传入的对象计数+1
	2. weak比较常用的地方是delegate属性的设置和Xib拖线
	3. weak和assign的区别：当他们指向的对象被释放以后，weak会被自动设置为nil，assign不会，所以会导致野指针的出现，可能会导致crash
* readwrite/readonly
	1. readwrite表示该属性可读可写，readonly表示只可读，不可写。
	2. readwrite程序自动创建setter和getter方法，readonly程序创建getter方法。此外还可以自定义setter和getter方法。
	3. 系统默认就是readwrite
* nonatomic和atomic
	1. nonatomic非原子属性。它的特点是多线程并发访问性能高，但是访问不安全；与之相对的atomic特点是安全但是以耗费系统资源为代价，所以在工程开发中用nonatomic时候较多。
	2. 系统默认的atomic，为setter方法加锁，而nonatomic不为setter方法加锁。
	3. 使用nonatomic要注意多线程间通讯的线程安全。
	4. 为什么nonatomic比atomic快，原因是它直接访问内存中的地址，不关心其他线程是否在改变这个值，并且中间没有死锁保护，
	5. 不要误认为多线程下家atomic是安全的。

```
1、nonatomic、atomiac

nonatomic：非原子的， atomiac 原子的 。属性默认是 atomiac ， 也就是原子性的。nonatomic执行效率高。

atomiac：读写安全，但效率低，不是绝对的安全，比如操作数组，增加或移除，这种情况可以使用互斥锁来保证线程安全

nonatomic：多线程执行效率高。

2、readwrite、readonly

readwrite 读写，readonly 只读。 属性默认是 readwrite ， 支持读写。

readwirte: 属性同时具有 set 和 get 方法。
readonly: 属性只具有 get 方法。

3、strong、retain、weak、assign、copy、unsafe_unretained

retain 、assign 是 MRC 时的关键字，到 ARC 时，换成了 strong 和 weak 。 属性默认是 MRC -- assign ;ARC -- 对象是 strong，基本数据类型还是 assign 。

4、strong、weak、assign、unsafe_unretained

strong 是每对这个属性引用一次，retainCount 就会+1，只能修饰 NSObject 对象，不能修饰基本数据类型。是 id 和 对象 的默认修饰符。
weak 对属性引用时，retainCount 不变，只能修饰 NSObject 对象，不能修饰基本数据类型。 主要用于避免循环引用。weak的原理：runtime维护了一个weak表，用于存储指向某个对象的所有weak指针。weak表其实是一个hash（哈希）表，key是所指对象的地址，value是weak指针的地址（这个地址的值是所指对象指针的地址）数组。

assign是默认关键字，用来修饰基本数据类型。
对这个关键字声明的属性操作时，retainCount 是一直不变的。

为什么我们不用assign去声明对象呢？

因为 assign 修饰的对象，在释放之后，指针的地址还是存在的，也就是说指针并没有被置为nil，造成野指针。访问野指针，会导致程序 crash。

为什么可以用assign修饰基本数据类型？

因为基本数据类型是分配在栈上，栈的内存会由系统自己自动处理回收，不会造成野指针。

5、unsafe_unretained

unsafe_unretained和_weak一样，表示的是对象的一种弱引用关系，唯一的区别是：weak修饰的对象被释放后，指向对象的指针会置空，也就是指向nil,不会产生野指针；而unsafe_unretained修饰的对象被释放后，指针不会置空，而是变成一个野指针，那么此时如果访问这个对象的话，程序就会Crash，抛出BAD_ACCESS的异常。

6、copy

copy分深copy和浅copy

浅copy，对象指针的复制，目标对象指针和原对象指针指向同一块内存空间，引用计数增加

深copy，对象内容的复制，开辟一块新的内存空间

可变的对象的copy和mutableCopy都是深拷贝

不可变对象的copy是浅拷贝，mutable是深拷贝

copy方法返回的都是不可变对象

NSString使用copy修饰不用strong修饰，用strong修饰一个name属性，如果赋值的是一个可变对象，当可变对象的值发生改变的时候，name的值也会改变，这不是我们期望的，是因为name使用strong修饰后，指向跟可变对象相同的一块内存地址，如果使用copy的话，则是深拷贝，会开辟一块新的内存空间，因此可变对象值变化时，也不会影响name的值。

7、@synthesize 和 @dynamic 分别有什么作用？

@property 有两个对应的词，一个是 @synthesize，一个是 @dynamic。如果 @synthesize 和 @dynamic 都没写，那么默认的就是 @syntheszie var = _var;

@synthesize 的语义是如果你没有手动实现 setter 方法和 getter 方法，那么编译器会自动为你加上这两个方法。

@dynamic 告诉编译器：属性的 setter 与 getter 方法由用户自己实现，不自动生成。（当然对于 readonly 的属性只需提供 getter 即可）。假如一个属性被声明为 @dynamic var，然后你没有提供 @setter 方法和 @getter 方法，编译的时候没问题，但是当程序运行到 instance.var = someVar，由于缺 setter 方法会导致程序崩溃；或者当运行到 someVar = var 时，由于缺 getter 方法同样会导致崩溃。编译时没问题，运行时才执行相应的方法，这就是所谓的动态绑定。

```
