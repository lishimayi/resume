# iOS 容器类

## 概览

1. NSArray、NSMutableArray
2. NSNumber
3. NSDIctionary、NSMutableDictionary
4. NSSet、NSMutableSet
5. NSCache （不确定是不是容器类）

## 一 、数组

### ① 不可以变数组

#### 创建数组

```
// 初始化方法
NSArray *arr = [[NSArray alloc] initWithObjects:@"1", @"2", nil];
// 类方法
 NSArray *arr2 = [NSArray arrayWithObjects:@"2", @"3"];

NSArray *arr3 =  [NSArray arrayWithArray:arr2];
```
#### 获取数组长度

```
- (NSInteger)count;
```
#### 根据索引值获取对象

```
- (id)objectAtIndex:(NSInteger)index
```

#### 获取某对象在数组中的索引值

```
- (NSInteger)indexOfObject:(id)anObject
```

#### 获取对象arr

```
- (NSArray *)objectsAtIndexes:(NSIndexSet *)indexes
```

### ① 可以变数组

#### 创建数组

```
// 初始化方法
NSArray *arr = [[NSArray alloc] initWithObjects:@"1", @"2", nil];
// 类方法
 NSArray *arr2 = [NSArray arrayWithObjects:@"2", @"3"];

NSArray *arr3 =  [NSArray arrayWithArray:arr2];
```


