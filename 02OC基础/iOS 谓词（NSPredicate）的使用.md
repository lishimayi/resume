# iOS 谓词（NSPredicate）的使用

## 谓词简介

> 官方解释：
> 
> The NSPredicate class is used tp define logical conditions uesed to constrain a search either for a fetch or for in-memory filtering.

NSPredicate类用于定义逻辑条件，这些条件用于限制对访存或内存中过滤的搜索。

意思就是：我是一个过滤器，不符合条件的多滚开。

## 一、NSPredicate的基本语法

只要我们使用谓词（Predicate）都需要为谓词定义表达式，而这个表达式必须返回BOOL值

谓词表达式由`表达式`、`运算符`和`值`构成·。

#### 1. 比较运算符

用`=`、`==`判断两表达式是否相等，在我们声明的谓词结构内“=”和“==”都是相同意思的判断，没有赋值的意思。下面是示例代码：

```
NSNumber *num = @123;
// 创建一个谓词
// 这里大写的SELF 就类似形参站位的存在
NSPredicate *predicate = [NSPreddicate predicateWithFormat:@"SELF = 123"];
// 开始对比
if ([predicate evaluateWithObject:num]) {
	NSLog(@"num is %@",num);
}

// 输出结果 
//2020-10-07 11:12:27.281 PredicteDemo[4130:80412] num is 123
```

下面是其他比较运算法符

`>=` `=>` 

`<=` `=<`

`>` 

`<` 

`!=` `<>`

`BETWEEN` 有固定标的格式 “ BETWEEN {100, 200 }” 下面是示例代码：

```
NSNumber *testNumber = @123;
NSPredicate *predicate = [NSPredicate predicateWithFormat:@"SELF BETWEEN {100, 200}"];
  if ([predicate evaluateWithObject:testNumber]) {
      NSLog(@"testString:%@", testNumber);
  } else {
      NSLog(@"不符合条件");
  }
  
  //输出结果
  //2016-01-07 11:20:39.921 PredicteDemo[4366:85408] testString:123
```

#### 2. 逻辑运算符

`AND`、 `&&` ：逻辑与。 示例代码

```
NSArray *testArray = @[@1, @2, @3, @4, @5, @6];
  NSPredicate *predicate = [NSPredicate predicateWithFormat:@"SELF > 2 && SELF < 5"];
  NSArray *filterArray = [testArray filteredArrayUsingPredicate:predicate];
  NSLog(@"filterArray:%@", filterArray);
  
  // 输出结果
  /*
   2016-01-07 11:27:01.885 PredicteDemo[4531:89537] filterArray:(
  3,
  4
)
*/
```

`OR` 、`||`：逻辑或

`NOT` 、`！`：逻辑非

#### 3. 字符串比较运算符

`BEGINWITH`：检查某个字符串是否以指定的字符串开头

`ENDSWITH`：检查某个字符串是否以指定的字符串结尾

`CONTAINS`：检查某个字符串是否包含指定的字符串

`LIKE`：检查某个字符串是否匹配指定的字符串模板。其之后可以跟?代表一个字符和 * 代表任意多个字符两个通配符。比如"name LIKE ' * ac * '"，这表示name的值中包含ac则返回YES；"name LIKE '?ac*'"，表示name的第2、3个字符为ac时返回YES。

`MATCHES`：检查某个字符串是否匹配指定的正则表达式。虽然正则表达式的执行效率是最低的，但其功能是最强大的，也是我们最常用的。

#### 3. 集合运算符

`ANY`、`SOME`：集合中任意一个元素满足条件，就返回YES。

`ALL`：集合中所有元素都满足条件，才返回YES。

`NONE`：集合中没有任何元素满足条件就返回YES。如:NONE person.age < 18，表示person集合中所有元素的age>=18时，才返回YES。

`IN`：等价于SQL语句中的IN运算符，只有当左边表达式或值出现在右边的集合中才会返回YES。我们通过一个例子来看一下

```
NSArray *filterArray = @[@"ab", @"abc"];
  NSArray *array = @[@"a", @"ab", @"abc", @"abcd"];
  NSPredicate *predicate = [NSPredicate predicateWithFormat:@"NOT (SELF IN %@)", filterArray];
  NSLog(@"%@", [array filteredArrayUsingPredicate:predicate]);
```

输出

```
2016-01-07 13:17:43.669 PredicteDemo[6701:136206] (
  a,
  abcd
)
```

`array[index]`：返回array数组中index索引处的元素

`array[FIRST]`：返回array数组中第一个元素

`array[LAST]`：返回array数组中最后一个元素

`array[SIZE]`：返回array数组中元素的个数

#### 直接量

在谓词表达式中可以直接使用如下直接量

`FALSE`、`NO`：代表逻辑假

`TRUE`、`YES`：代表逻辑真

`NULL`、`NIL`：代表空值

`SELF`：代表正在被判断的对象自身

` "string"` 或 `'string'`：代表字符串

数组：和c中的写法相同，如：{'one', 'two', 'three'}。

数值：包括证书、小数和科学计数法表示的形式

十六进制数：0x开头的数字

八进制：0o开头的数字

二进制：0b开头的数字

#### 保留字

下列单词都是保留字（不论大小写）

AND、OR、IN、NOT、ALL、ANY、SOME、NONE、LIKE、CASEINSENSITIVE、CI、MATCHES、CONTAINS、BEGINSWITH、ENDSWITH、BETWEEN、NULL、NIL、SELF、TRUE、YES、FALSE、NO、FIRST、LAST、SIZE、ANYKEY、SUBQUERY、CAST、TRUEPREDICATE、FALSEPREDICATE

注：虽然大小写都可以，但是更推荐使用大写来表示这些保留字

## 谓词用法

#### 1.定义谓词

`NSPredicate *predicate = [NSPredicate predicateWithFormat:];`

下面我们通过几个简单的例子来看看它该如何使用：

```
// .h
#import typedef NS_ENUM(NSInteger, ZLPersonSex) {
    ZLPersonSexMale = 0,
    ZLPersonSexFamale
};

@interface ZLPersonModel : NSObject
/** NSString 姓名 */
@property (nonatomic, copy) NSString *name;
/** NSUInteger 年龄 */
@property (nonatomic, assign) NSUInteger age;
/** ZLPersonSex 性别 */
@property (nonatomic, assign) ZLPersonSex sex;

+ (instancetype)personWithName:(NSString *)name age:(NSUInteger)age sex:(ZLPersonSex)sex;

@end
```

下面我们进入正题

```
ZLPersonModel *sunnyzl = [ZLPersonModel personWithName:@"sunnyzl" age:29 sex:ZLPersonSexMale];
    ZLPersonModel *jack = [ZLPersonModel personWithName:@"jack" age:22 sex:ZLPersonSexMale];
    //  首先我们来看一些简单的使用
    //  1.判断姓名是否是以s开头的
    NSPredicate *pred1 = [NSPredicate predicateWithFormat:@"name LIKE 's*'"];
    //  输出为：sunnyzl:1, jack:0
    NSLog(@"sunnyzl:%d, jack:%d", [pred1 evaluateWithObject:sunnyzl], [pred1 evaluateWithObject:jack]);
    //  2.判断年龄是否大于25
    NSPredicate *pred2 = [NSPredicate predicateWithFormat:@"age > 25"];
    //  输出为：sunnyzl的年龄是否大于25：1, jack的年龄是否大于25：0
    NSLog(@"sunnyzl的年龄是否大于25：%d, jack的年龄是否大于25：%d", [pred2 evaluateWithObject:sunnyzl], [pred2 evaluateWithObject:jack]);

```

看到这里我们会发现evaluateWithObject:方法返回的是一个BOOL值，如果符合条件就返回YES，不符合就返回NO。而即使是最简单的使用也有一些大用处，比如以前我们写判断手机号码、邮编等等，像我就喜欢用John Engelhart大神的RegexKitLite，然而由于年代久远需要导入libicucore.dylib库（xcode7为libicucore.tbd）且由于是mrc又需要添加-fno-objc-arc，至此我们才能使用。然而使用谓词让我们可以用同样简洁的代码实现相同的功能

列二 手机号是否正确

```
 - (BOOL)checkPhoneNumber:(NSString *)phoneNumber
{
    NSString *regex = @"^[1][3-8]\\d{9}$";
    NSPredicate *pred = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", regex];
    return [pred evaluateWithObject:phoneNumber];
}
```

看到这里是不是感觉好爽，感觉以前所有的正则都可以这么匹配，但是谓词匹配正则时也是有缺点的，下面通过一个例子来看一下这个致命的缺点

例三：谓词匹配正则的缺点

（本意：检测字符串中是否有特殊字符）

```
- (BOOL)checkSpecialCharacter:(NSString *)string
{
    NSString *regex = @"[`~!@#$^&*()=|{}':;',\\[\\].<>/?~！@#￥……&*（）——|{}【】‘；：”“'。，、？]";
    NSPredicate *pred = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", regex];
    return [pred evaluateWithObject:string];
}
```

我们想要的效果是字符串中有特殊字符时就返回YES，然而梦想是美好的，现实是残酷的

让我们看看这悲催的结局

```
NSString *testString = @"!";
NSLog(@"是否含有特殊字符：%d", [self checkSpecialCharacter:testString]);
//  当testString为一个特殊字符时，我们惊喜的发现输出为
//  是否含有特殊字符：1
```

看到这里我们心里猛然一喜，这tmd根本没问题嘛

让我们修改下testString的值

```
NSString *testString = @"!~";
NSLog(@"%d", [self checkSpecialCharacter:testString]);
//  我们会发现悲催的结局来了输出为
//  是否含有特殊字符：0
```

再次修改testString的值

```
NSString *testString = @"abc!~d";
NSLog(@"%d", [self checkSpecialCharacter:testString]);
//  我们会发现输出为
# //  是否含有特殊字符：0
```

这总与我们的想法事与愿违，看到这里我们会发现谓词对正则并不像我们使用NSRegularExpression时匹配的那么好，究其原因是为什么呢？我们用NSRegularExpression时会发现匹配到一个结果时就会存入数组，再从匹配到的位置继续向下匹配。

然而NSPredicate并不会做这样的自动操作，我们最终发现在NSPredicate输入[`~!@#$^&*()=|{}':;',\[\].<>/?~！@#￥……&*（）——|{}【】‘；：”“'。，、？]正则表达式时和写成^[`~!@#$^&*()=|{}':;',\[\].<>/?~！@#￥……&*（）——|{}【】‘；：”“'。，、？]$的效果是一样的。

所以通过这个例子我们总结出来，只有在正则表达式为^表达式$时才使用谓词，而不是所有情况都使用。

那么我们是不是因为这一点就摒弃它了呢，答案是否定的。因为虽然NSPredicate有这么一点瑕疵，但是它总体带给我们的便利其实除了正则表达式匹配时的这个问题外是更多的。

#### 2. 使用谓词过滤集合

此部分是我们需要掌握的重点，因为从这里我们就可以看到谓词的真正的强大之处

其实谓词本身就代表了一个逻辑条件，计算谓词之后返回的结果永远为BOOL类型的值。而谓词最常用的功能就是对集合进行过滤。当程序使用谓词对集合元素进行过滤时，程序会自动遍历其元素，并根据集合元素来计算谓词的值，当这个集合中的元素计算谓词并返回YES时，这个元素才会被保留下来。请注意程序会自动遍历其元素，它会将自动遍历过之后返回为YES的值重新组合成一个集合返回。

其实类似于我们使用tableView设置索引时使用的下段代码

```
- (NSArray *)sectionIndexTitlesForTableView:(UITableView *)tableView
{
    return [self.cityGroup valueForKey:@"title"];
}
```

中的[self.cityGroup valueForKey:@"title"]。它的作用是遍历所有title并将得到的值组成新的数组。

NSArray提供了如下方法使用谓词来过滤集合

- (NSArray*)filteredArrayUsingPredicate:(NSPredicate *)predicate:使用指定的谓词过滤NSArray集合，返回符合条件的元素组成的新集合

NSMutableArray提供了如下方法使用谓词来过滤集合

- (void)filterUsingPredicate:(NSPredicate *)predicate：使用指定的谓词过滤NSMutableArray，剔除集合中不符合条件的元素

NSSet提供了如下方法使用谓词来过滤集合

- (NSSet*)filteredSetUsingPredicate:(NSPredicate *)predicate NS_AVAILABLE(10_5, 3_0)：作用同NSArray中的方法

NSMutableSet提供了如下方法使用谓词来过滤集合

- (void)filterUsingPredicate:(NSPredicate *)predicate NS_AVAILABLE(10_5, 3_0)：作用同NSMutableArray中的方法。

通过上面的描述可以看出，使用谓词过滤不可变集合和可变集合的区别是：过滤不可变集合时，会返回符合条件的集合元素组成的新集合；过滤可变集合时，没有返回值，会直接剔除不符合条件的集合元素

下面让我们来看几个例子：

例一：

```
NSMutableArray *arrayM = [@[@20, @40, @50, @30, @60, @70] mutableCopy];
    //  过滤大于50的值
    NSPredicate *pred1 = [NSPredicate predicateWithFormat:@"SELF > 50"];
    [arrayM filterUsingPredicate:pred1];
    NSLog(@"arrayM:%@", arrayM);
    NSArray *array = @[[ZLPersonModel personWithName:@"Jack" age:20 sex:ZLPersonSexMale],
                       [ZLPersonModel personWithName:@"Rose" age:22 sex:ZLPersonSexFamale],
                       [ZLPersonModel personWithName:@"Jackson" age:30 sex:ZLPersonSexMale],
                       [ZLPersonModel personWithName:@"Johnson" age:35 sex:ZLPersonSexMale]];
    //  要求取出包含‘son’的元素
    NSPredicate *pred2 = [NSPredicate predicateWithFormat:@"name CONTAINS 'son'"];
    NSArray *newArray = [array filteredArrayUsingPredicate:pred2];
    NSLog(@"%@", newArray);
```

输出：

```
2016-01-07 16:50:09.510 PredicteDemo[13660:293822] arrayM:(
    60,
    70
)
2016-01-07 16:50:09.511 PredicteDemo[13660:293822] (
    "[name = Jackson, age = 30, sex = 0]",
    "[name = Johnson, age = 35, sex = 0]"
)
```

从这个例子我们就可以看到NSPredicate有多么强大，如果让我们用其他的方法来实现又是一大堆if...else。

让我们来回顾一下上面的从第二个数组中去除第一个数组中相同的元素

例二：

```
NSArray *filterArray = @[@"ab", @"abc"];
    NSArray *array = @[@"a", @"ab", @"abc", @"abcd"];
    NSPredicate *predicate = [NSPredicate predicateWithFormat:@"NOT (SELF IN %@)", filterArray];
    NSLog(@"%@", [array filteredArrayUsingPredicate:predicate]);
```

输出为：

```
2016-01-07 13:17:43.669 PredicteDemo[6701:136206] (
    a,
    abcd
)
```

如果我们不用NSPredicate的话，肯定又是各种if...else，for循环等等。可以看出NSPredicate的出现为我们节省了大量的时间和精力。

#### 3.在谓词中使用占位符参数

我们上面所有的例子中谓词总是固定的，然而我们在现实中处理变量时决定了谓词应该是可变的。下面我们来看看如果让谓词变化起来。

首先如果我们想在谓词表达式中使用变量，那么我们需要了解下列两种占位符：

%K：用于动态传入属性名

%@：用于动态设置属性值

其实相当于变量名与变量值

除此之外，还可以在谓词表达式中使用动态改变的属性值，就像环境变量一样

```
NSPredicate *pred = [NSPredicate predicateWithFormat:@"SELF CONTAINS $VALUE"];
```

上述表达式中，$VALUE是一个可以动态变化的值，它其实最后是在字典中的一个key，所以可以根据你的需要写不同的值，但是必须有$开头，随着程序改变$VALUE这个谓词表达式的比较条件就可以动态改变。

下面我们通过一个例子来看看这三个重要的占位符应该如何使用

例一：

```
NSArray *array = @[[ZLPersonModel personWithName:@"Jack" age:20 sex:ZLPersonSexMale],
                     [ZLPersonModel personWithName:@"Rose" age:22 sex:ZLPersonSexFamale],
                     [ZLPersonModel personWithName:@"Jackson" age:30 sex:ZLPersonSexMale],
                     [ZLPersonModel personWithName:@"Johnson" age:35 sex:ZLPersonSexMale]];
  //  定义一个property来存放属性名，定义一个value来存放值
  NSString *property = @"name";
  NSString *value = @"Jack";
  //  该谓词的作用是如果元素中property属性含有值value时就取出放入新的数组内，这里是name包含Jack
  NSPredicate *pred = [NSPredicate predicateWithFormat:@"%K CONTAINS %@", property, value];
  NSArray *newArray = [array filteredArrayUsingPredicate:pred];
  NSLog(@"newArray:%@", newArray);
  
  //  创建谓词，属性名改为age，要求这个age包含$VALUE字符串
  NSPredicate *predTemp = [NSPredicate predicateWithFormat:@"%K > $VALUE", @"age"];
  // 指定$SUBSTR的值为 25    这里注释中的$SUBSTR改为$VALUE
  NSPredicate *pred1 = [predTemp predicateWithSubstitutionVariables:@{@"VALUE" : @25}];
  NSArray *newArray1 = [array filteredArrayUsingPredicate:pred1];
  NSLog(@"newArray1:%@", newArray1);
  
  //  修改 $SUBSTR的值为32，  这里注释中的SUBSTR改为$VALUE
  NSPredicate *pred2 = [predTemp predicateWithSubstitutionVariables:@{@"VALUE" : @32}];
  NSArray *newArray2 = [array filteredArrayUsingPredicate:pred2];
  NSLog(@"newArray2:%@", newArray2);
```

输出为：

```
2016-01-07 17:28:02.062 PredicteDemo[14542:309494] newArray:(
  "[name = Jack, age = 20, sex = 0]",
  "[name = Jackson, age = 30, sex = 0]"
)
2016-01-07 17:28:02.063 PredicteDemo[14542:309494] newArray1:(
  "[name = Jackson, age = 30, sex = 0]",
  "[name = Johnson, age = 35, sex = 0]"
)
2016-01-07 17:28:02.063 PredicteDemo[14542:309494] newArray2:(
  "[name = Johnson, age = 35, sex = 0]"
)
```

从上例中我们主要可以看出来%K和$VALUE的含义。

那么至此NSPredicate就到到此介绍完毕。

文章转自：http://www.cocoachina.com/articles/14926