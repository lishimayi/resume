# block

### block 本质

- OC 对象
  - 带有 isa 指针
- 这个对象封装了函数调用地址以及函数调用环境(函数参数、返回值、捕获的外部变量等)
- 下面我们来看一个例子，定义了一个 block，并在 block 里面访问量 block 外面的变量 age，它底层存储结构如下图所示，block 底层就是一个结构体\_\_main_block_impl_0。
  ![图1](/11Source/block.jpg)

### block 类型

- GlobalBlock
  - 位于全局区
  - 在 block 内部未使用`静态变量`和`全局变量`以外的其他`外部变量`
- MallocBlock
  - 位于堆区
  - 在 block 内部使用了`局部变量`或者`OC属性`,并且赋值给强引用或者 copy 修饰的变量
- StackBlock
  - 位于栈区
  - 内部使用了`局部变量`或者`OC属性`,但是未赋值给强引用或者 copy 修饰的变量

### 什么情况下会触发 block 的 copy

- 手动 copy
  - stackBlock, 会变成 mallocBlock
  - globalBlock, 还是 globalBlock
- block 作为返回值
  - 使用了局部变量就会被 cpoy 到堆上(因为被强引用了)
  - 未使用局部变量就还是 globalBlock
- 被强引用或者 Copy 修饰
- 系统 API 包含 usingBlock
  - globalblock

### block 的变量捕获机制

```
int c = 1000; // 全局变量
static int d = 10000; // 静态全局变量

int main(int argc, const char * argv[]) {
    @autoreleasepool {

        int a = 10; // 局部变量
        static int b = 100; // 静态局部变量
        void (^block)(void) = ^{
             NSLog(@"a = %d",a);
             NSLog(@"b = %d",b);
             NSLog(@"c = %d",c);
             NSLog(@"d = %d",d);
         };
         a = 20;
         b = 200;
         c = 2000;
         d = 20000;
         block();
    }
    return 0;
}

// ***************打印结果***************
2020-01-07 15:08:37.541840+0800 CommandLine[70672:7611766] a = 10
2020-01-07 15:08:37.542168+0800 CommandLine[70672:7611766] b = 200
2020-01-07 15:08:37.542201+0800 CommandLine[70672:7611766] c = 2000
2020-01-07 15:08:37.542222+0800 CommandLine[70672:7611766] d = 20000
```

- 全局变量 c 不捕获
- 静态全局变量 d 不捕获
- 局部变量 a 值捕获，修改不影响外部这里的 int a 书写完整：auto int a
- 静态局部变量 b 地址捕获，修改时会影响外部

### block 对象类型变量捕获

- 栈上 block 捕获对象类型变量
  - 对象变量的指针是弱引用，block 不强引用它
  - 对象变量的指针是强引用，block 不强引用它

### \_\_block

- `__block`修饰的变量

  ```
  struct __main_block_impl_0 {
  	struct __block_impl impl;
  	struct __main_block_desc_0* Desc;
  	__Block_byref_age_0 *age; // by ref
  };

  struct __Block_byref_age_0 {
  	void *__isa; // isa指针
  	__Block_byref_age_0 *__forwarding; // 如果这block是在堆上那么这个指针就是指向它自己，如果这个block是在栈上，那这个指针是指向它拷贝到堆上后的那个block
  	int __flags;
  	int __size; // 结构体大小
  	int age; // 真正捕获到的age
  };
  ```

- `__block`不管是修饰基础数据类型还是修饰对象数据类型，底层都是将它包装成一个对象
- 当 block 在栈上的时候，并不会对`__block` 修饰的变量产生强引用
- 当 block 从站上 copy 到堆上的时候，
  - 会调用 block 内部的 copy 函数
  - copy 函数会内部调用`_Block_object_assign` 函数
  - `_Block_object_assign` 会对`__block` 变量形成强引用（retain）
- 当 block 从堆中移除时
  - 会调用 block 内部的 dispose 函数
  - `dispose` 函数内部会调用 `_Block_object_dispose` 函数
  - `_Block_object_dispose` 函数会对`__block` 变量释放操作（release）
