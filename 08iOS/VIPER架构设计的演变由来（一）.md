# VIPER架构设计的演变由来（一）

MVP

* view负责界面展示和布局管理，想presentor暴露视图更新和数据获取的接口
* Presentor负责接收来自View的事件，通过View的接口更新视图，并管理Model
* Model和MVC中的一样，提供数据模型和数据操作。

在iOS里UIView和UIViewController共同组成了MVP中的View。UIView负责元素的展示，Controller负责界面的布局和组合，并把事件转发给Presentor。

因此，在iOS 中使用MVP很简单，在View和Presentor之间用Protocol做好事件传递就可以。缺点是多了一层隔离接口，胶水代码变多。

但是随着页面越来越复杂，Presentor中的业务也会越来越庞大，总有一天会遇到新的问题，如何细分Presentor。

MVVM

`Model-View-ViewModel` 模式，他和MVP一样，目的是解决View和model的耦合 + Controller过重。部分分工如下：

### 最普遍的MVVM

* model提供数据模型
* View负责视图展示
* ViewModel用于描述View的状态，例如View的颜色，显示的文字等属性类的信息，将View抽象成了一个特殊的模型，并且持有和管理Model，维护业务逻辑

在MVP中，View通过接口的方式描述自己，在MVVM中，则通过ViewModel来描述自己的特征，那么ViewModel如何能将自己的变化更新到View上呢？MVVM经常和数据绑定一起出现，在UIViewController中，将View和ViewModel的属性用类似KVO的方式进行绑定，这样ViewModel的变化就能立刻传输到View上。

### 数据的绑定

利用ReactiveCocoa和RxSwift这些函数式响应编程框架实现数据的绑定，可以用很少的代码完成复杂业务逻辑，熟练时能够提升开发速度。但是数据绑定的缺点很明显：调试困难，数据来源难以回溯，在线上出现bug的时候就很难追踪，所以从这方面来讲又增加了维护的成本。

其实数据绑定只是一种为了减少胶水代码的技术实现方式，MVVM的设计并没有要求必须使用数据绑定，你也完全可以使用Protocol的方式来将ViewModel的变换传递给View，让数据流向清晰。MVVM的关键是将View进行抽象，从而实现View和Model进行解耦。

### ViewModel的职责

但是除了数据绑定，MVVM还有另外一个问题，把业务逻辑放到了ViewModel中，虽然能够为Controller减重，但是只是把问题转移了，最终的ViewModel也还是称为另外一个Massive ViewModel。

而且当ViewModel维护Model和业务逻辑时，可复用性就会大大降低。例如，把同一个登录界面复用到另一个app中时，login model中的属性名或者类型可能完全改变，从而数据处理的方式也会改变，到时ViewModel无法重用。而当View由多个子View组成的时候，ViewModel里也会引入多个子Model。这就又导致了View的实现影响了ViewModel的实现。奇怪的是，国内iOS圈对这个问题探讨比较少。

ViewModel到时是什么？ 从它的命名和最初设计来看，他只是View的抽象，目的是方便和Model进行数据转换。而默认把业务也放到ViewModel里，大概是由于objc.io上那篇文章的影响。其实在微软的WPF和前端里，MVVM的业务逻辑大部分放在Model层，先关讨论可以参考：

[MVVM: ViewModel and Business Logic Connection](https://stackoverflow.com/questions/16338536/mvvm-viewmodel-and-business-logic-connection)

[Where does business logic sit in MVVM?](https://stackoverflow.com/questions/10964003/where-does-business-logic-sit-in-mvvm)

[The Problems with MVVM on iOS](https://www.danielhall.io/the-problems-with-mvvm-on-ios)

针对这个问题，有人又提出了MVVMP架构（`Model-View-ViewModel-Presentor`）,把业务逻辑放到Presentor里。Presentor的引入让ViewModel专注于View的抽象，和model分离开来，只负责管理View相关的状态，传递View的事件，因此ViewModel中的代码可以得到很好的复用。而Presentor负责大部分业务逻辑，如果模块需要重用，则把业务逻辑中的数据操作（domain logic）单独分离出来作为重用代码，其他的无法重用的应用逻辑（application logic）则依旧放在Presentor里面。

和MVP相比，MVVM使用了一种更优雅的方式来抽象View，但是他和MVP其实类似，只是做了View和Model的解耦，仍然没有对Controller进行进一步细分。

## VIPER

VIPER全称`View-Interactor-Presentor-Entity-Router`。示意图如下：

![VIPER](https://upload-images.jianshu.io/upload_images/1865432-580872920986b640.png)

相比之前的MVX框架，VIPER多出了两个东西：Interactor（交互器）Router（路由）

各部分职责如下：

### View 

* 提供完整的视图，负责视图的组合，布局，更新。
* 向Presentor提供更新视图接口
* 将View相关事件发送给Presentor

### Presentor

* 接收并处理View的事件。
* 向Interactor请求调用业务逻辑
* 向interactor提供View中的数据
* 接收并处理来自Interactor的数据回调事件
* 通知View进行更新操作
* 通过Router跳转到其他的View

### Router

* 提供View之间的跳转功能，减少了模块之间的耦合
* 初始化VIPER的各个模块

### Interactor

* 维护主要的业务逻辑功能，向Presentor提供现有的业务用例
* 维护，获取，更新entity
* 当有业务相关的事件发生时候，处理事件，并通知Presentor

### Entity

* 和model一样的数据模型

## 和MV(X)的区别

VIPER把MVC中的Controller进一步拆分成了Presenter、Router和Interactor。和MVP中负责业务逻辑的Presenter不同，VIPER的Presenter的主要工作是在View和Interactor之间传递事件，并管理一些View的展示逻辑，主要的业务逻辑实现代码都放在了Interactor里。Interactor的设计里提出了"用例"的概念，也就是把每一个会出现的业务流程封装好，这样可测试性会大大提高。而Router则进一步解决了不同模块之间的耦合。所以，VIPER和上面几个MVX相比，多总结出了几个需要维护的东西：

* View事件管理
* 数据事件管理
* 事件和业务的转化
* 总结每个业务用例
* 模块内分层隔离
* 模块间通信

而这里面，还可以进一步细分一些职责。VIPER实际上已经把Controller的概念淡化了，这拆分出来的几个部分，都有很明确的单一职责，有些部分之间是完全隔绝的，在开发时就应该清晰地区分它们各自的职责，而不是将它们视为一个Controller。

## 优点

VIPER的特色就是职责明确，粒度细，隔离关系明确，这样能带来很多优点：

* 可测试性好。UI测试和业务逻辑测试可以各自单独进行。
* 易于迭代。各部分遵循单一职责，可以很明确地知道新的代码应该放在哪里。
* 隔离程度高，耦合程度低。一个模块的代码不容易影响到另一个模块。
* 易于团队合作。各部分分工明确，团队合作时易于统一代码风格，可以快速接手别人的代码。

## 缺点

* 一个模块内的类数量增大，代码量增大，在层与层之间需要花更多时间设计接口。
	* 使用代码模板来自动生成文件和模板代码可以减少很多重复劳动，而花费时间设计和编写接口是减少耦合的路上不可避免的，你也可以使用数据绑定这样的技术来减少一些传递的层次。	
* 模块的初始化较为复杂，打开一个新的界面需要生成View、Presenter、Interactor，并且设置互相之间的依赖关系。而iOS中缺少这种设置复杂初始化的原生方式。

简单来说，就是Cocoa框架缺少一个强大的自定义依赖注入工具。这个问题影响不是特别大，可以选用一些第三方工具来实现，也可以在Router的界面跳转方法里，对模块进行初始化，只不过总是不够完美。针对这个问题，我实现了一个基于protocol声明依赖的界面跳转Router，将会在之后的文章中进行详解。

## 总结

有人可能会觉得，一个界面模块真的有必要使用这么复杂的架构吗？这样是不是过度设计？

我反对这种观点。不要被VIPER的组织图吓到，VIPER并不复杂，它是将原来MVC中的Controller中的各种任务进行了清晰的分解，在写代码时，你会很清楚你正在做什么。事实上，它比使用了数据绑定技术的MVVM更加简单，就是因为它职责明确。从MVC转到VIPER的过程同样是很清晰的，它甚至把重构的思路都体现出来了。而MVVM则留下了许多尚未明确的责任，导致不同的人会在某些地方有不同的实现。即便你还在使用MVC，你也应该在Controller中分离出VIPER总结出的那些专项职责，既然如此，为何不彻底地明确这些职责，把它们分散到不同的文件中呢？一旦开始这样的工作，你就已经向VIPER靠拢了。

有人可能会觉得，VIPER适合大型app，中小型app没必要过早使用。

我反对这种观点。VIPER是单个界面模块内的架构设计，并不是整个app架构层面的设计，和app的整体架构没有多大的关系，也不存在过早使用VIPER的情况。所以，严格来说，是复杂界面更适合VIPER，而不是大型app更适合VIPER。

至此，我的结论就是，快点拥抱VIPER的怀抱吧。


## 文章转自 

https://www.jianshu.com/p/8418fba98e1a

