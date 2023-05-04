#iOS分类（category）和拓展（extension）

##背景：

有讲：在大型项目，企业级开发中多人同时维护同一个类，这样势必会导致当前类随着项目开阵，变得臃肿。iOS中的分类（category）就很好的解决了这个问题。

##分类（category）：

### 概念

分类（category）是OC中特有语法，它是表示一个指向分类的结构体指针。原则上只能添加方法，不能添加成员（实例）变量。具体原因看源码组成：

```
// category源码:
// Category是表示一个指向分类的结构体指针，其定义如下：
typedef struct objc_category *Category;
struct objc_category {
	char *category_name								OBJC2_UNAVAILABLE; // 分类名
	char *class_name								OBJC2_UNAVAILABLE; // 分类所属类名
	struct objc_method_list *instance_methods		OBJC2_UNAVAILABLE; // 实例方法列表
	struct objc_method_list *class_methods			OBJC2_UNAVAILABLE; // 类方法列表
	struct objc_protocol_list *protocols			OBJC2_UNAVAILABLE; // 分类所实现的协议列表
}

// 通过上面我们可以发现：这个结构体主要包含了分类定义的实例方法和类方法。没有属性相关列表。
```
##### 注意：

##### 1. 所以：原则上讲分类只能给原有类添加方法，不能添加属性（成员变量），但实际上因为OC是动态类型语言，我们可以用使用runtime库的动态绑定给原类添加。

##### 2. 分类中可以写`@property`,但是系统不会给生成相应的`setter`和`getter`方法，也不会生成成员变量（编译的时候会报警告）

##### 3. 可以在分类中访问原有类中.h的属性；

##### 4. 如果分类中有和原有类同名的方法，会有限调用分类中的方法，就是说会忽略原有类的方法。所以同名方法调用的优先级为：`分类>本类>父类`。因此在开发中尽量不要覆盖原有类。

##### 5. 如果多个分类中都有和原有类中同名的方法，那么调用该方法的时候执行谁由编译器决定；编译会执行最后一个参与编译的分类中的方法。

### 分类格式：

```
@inteface 待拓展的类 (分类的名称)
@end

@implementation 待拓展的类 (分类的名称)
@end

```

### 实际代码：

```
// Programmer+Category.h 文件
@interface Programmer (Category)

@property (nonatomic, copy) NSString *nameWithSetterGetter;//已设置setter和getter方法的属性

@property (nonatomic, copy) NSString *nameWithoutSetterGetter;// 未设置setter和getter方法的属性（注意，是可以写在这，而且编译智慧报警告，运行部报错）

- (void)programmerCategoryMethod;// 分类的方法

@end

// Programmer+Category.h 文件

@implementation Programmer (Category)


- (void)ProgramCategoryMethod {
	NSLog(@"do something.");
}
@end
```
