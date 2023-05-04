# CocoaPods安装2020/11/01

## 简介

**CocoaPods是一个用Ruby写的、负责管理iOS项目中第三方开源库的工具，CocoaPods能让我们集中的、统一管理第三方开源库，为我们节省设置和更新第三方开源库的时间。**

## 安装

在安装之前，我们需要关注一下Ruby的版本。因为安装Cocoapods 需要2.2.2版本级以上。

### 1、查看Ruby版本。

```

ruby -v
 
```
如果版本低了，需要升级ruby

### 2、升级Ruby环境，首先需要安装rvm。(rvm：ruby version manager)

```
curl -L get.rvm.io | bash -s stable 
source ~/.bashrc
source ~/.bash_profile

```

### 3、查看rvm版本

```

rvm -v
```

### 4、使用rvm列出可以安装的ruby版本

```

rvm list known
```

结果如下：

```
# MRI Rubies
[ruby-]1.8.6[-p420]
[ruby-]1.8.7[-head] # security released on head
[ruby-]1.9.1[-p431]
[ruby-]1.9.2[-p330]
[ruby-]1.9.3[-p551]
[ruby-]2.0.0[-p648]
[ruby-]2.1[.10]
[ruby-]2.2[.10]
[ruby-]2.3[.7]
[ruby-]2.4[.4]
[ruby-]2.5[.1] 
.....
[ruby-]2.6[.3]  // 重点在这里 重点在这里 重点在这里
[ruby-]2.7[.0-preview1]   // 测试版
ruby-head
```

5、安装ruby

```
rvm install 2.6.3
```

这里很多小伙伴会遇到错误，大部分是因为没有安装Homebrew造成，所以所以所以要提前安装比较好

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
// ruby -e "$(curl -fsSL https://raw.github.com/Homebrew/homebrew/go/install)"
```

### 6、设置为默认版本

```
rvm use 2.6.3 --default

```

### 7、更换源

```
sudo gem update --system

gem sources --remove https://rubygems.org/

gem sources --add https://gems.ruby-china.com/
```

8、为了验证你的Ruby镜像是并且仅是ruby-china，执行以下命令查看

```
gem sources -l
```

如果是以下结果说明正确，如果有其他的请自行百度解决

```
*** CURRENT SOURCES ***

https://gems.ruby-china.com/
```

### 9、这时候才正式开始安装CocoaPods

```
//sudo gem install -n /usr/local/bin cocoapods
sudo gem install cocoapods// 官方只给了这句话，
```

### 10、如果安装了多个Xcode使用下面的命令选择（一般需要选择最近的Xcode版本）

```
sudo xcode-select -switch /Applications/Xcode.app/Contents/Developer

```