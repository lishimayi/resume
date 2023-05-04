#iOS.架构的演变

## 路线

* MVC 
* MVVM
* MVP
* VIPER

## 架构的初衷

* 随着版本不断的迭代，业务的增加，代码维护成本不断增加，最根本的问题要解决

##MCV

### 过重的C层，需要减重

viewModel(MVVM: 数据绑定RAC)/ presentor(MVP：protocol通讯)
* API调用
* 业务逻辑
* 数据转换
* 数据资源

## coordinator 

* create and setup VCs
* manage VCs‘ flow
* manage child coordinators

## Redux 管理状态的框架

* 一个状态数据管理的库
* 只有一个人可以修改数据
* 只有一个地方可以修改数据
* 单向数据流
