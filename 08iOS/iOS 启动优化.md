# iOS 启动优化

## 前言

iOS的启动优化的用户直观感受就是用户点击手机屏幕的app图标到首页加载完毕这个过程事件优化。

### 这个过程分为3个阶段

1. main()执行之前
2. main()执行之后
3. 首屏渲染完成后

## main()函数执行前

在main()函数执行之前，系统主要做下面几件事：

* 加载可执行文件（APP的.o文件集合）；
* 加载动态链接库，进行rebase指针调整和bind符号绑定；
* Objc运行时的初始化处理，包括Objc相关类的注册，category注册，Selector唯一性检查等；
* 初始化，包括了执行`+load()`方法、attribute((constructor))修饰的函数的调用创建C++静态全局变量。

### 相应地，这个阶段对于启动速度优化来说，可以做的事情包括：

* 减少动态库加载。每个库本身都是依赖关系，苹果公司建议使用更少的动态库，并且建议在使用动态库较多的时候，尽量将多个动态库进行合并。数量上，苹果建议最多使用6个非系统动态库。
* 减少加载启动后不会去使用的类或者方法。
* +load() 方法里的内容可以放到首屏渲染完成之后再去执行，或者使用+initialize（）方法进行替换掉。因为在一个+load（）方法里，进行运行时方法替换操作会带来4毫秒的消耗，极少成多的+laod（）方法会带来大影响。
* 控制C++ 全局变量