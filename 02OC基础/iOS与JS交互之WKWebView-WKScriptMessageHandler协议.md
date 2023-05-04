# iOS与JS交互之WKWebView-WKScriptMessageHandler协议

## 前言

“iOS原生与JS交互”是指iOS原生代码（本文是OC）调用JS代码，JS代码调用OC代码。

>本文介绍如果使用wkwebview的WKScriptMessageHandler实现OC与JS交互。
>WKWebView是苹果在iOS8（2014年）推出的Wekit框架中负责网页的渲染和展示的类，相比UIWebView速度更快，占用内存更少，支持更多的html特性，`WKScriptMessageHandler`是Webkit提供的一种在WKWebView上进行JS消息控制的协议。
>

## 一、js调用iOS

### 代码示例

JS 代码: 假设这里有一个登录功能

```
//! 登录按钮
<button onclick = "login()" style = "font-size: 18px;">登录</button>
```

```
//! 登录
function login() {
  var token = "js_tokenString";
  loginSucceed(token);
}

//! 登录成功
function loginSucceed(token) {
  var action = "loginSucceed";
  window.webkit.messageHandlers.jsToOc.postMessage(action, token);
}
```

iOS代码

```
//! 导入WebKit框架头文件
#import <WebKit/WebKit.h>

//! WKWebViewWKScriptMessageHandlerController遵守WKScriptMessageHandler协议
@interface WKWebViewWKScriptMessageHandlerController () <WKScriptMessageHandler>
```

```
//! 为userContentController添加ScriptMessageHandler，并指明name
WKUserContentController *userContentController = [[WKUserContentController alloc] init];
[userContentController addScriptMessageHandler:self name:@"jsToOc"];

//! 使用添加了ScriptMessageHandler的userContentController配置configuration
WKWebViewConfiguration *configuration = [[WKWebViewConfiguration alloc] init];
configuration.userContentController = userContentController;

//! 使用configuration对象初始化webView
_webView = [[WKWebView alloc] initWithFrame:self.view.bounds configuration:configuration];

```

```
#pragma mark - WKScriptMessageHandler

//! WKWebView收到ScriptMessage时回调此方法
- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message {
    
    if ([message.name caseInsensitiveCompare:@"jsToOc"] == NSOrderedSame) {
        [WKWebViewWKScriptMessageHandlerController showAlertWithTitle:message.name message:message.body cancelHandler:nil];
    }
}
```

## 实现原理

1. JS与iOS约定好jsToOc方法，用作JS在调用iOS时的方法；
2. iOS使用WKUserContentController的-addScriptMessageHandler:name:方法监听name为jsToOc的消息；
3. JS通过window.webkit.messageHandlers.jsToOc.postMessage()的方式对jsToOc方法发送消息；
4. iOS在-userContentController:didReceiveScriptMessage:方法中读取name为jsToOc的消息数据message.body。

> PS：[userContentController addScriptMessageHandler:self name:@"jsToOc"]会引起循环引用问题。一般来说，在合适的时机removeScriptMessageHandler可以解决此问题。比如：在-viewWillAppear:方法中执行add操作，在-viewWillDisappear:方法中执行remove操作。如下：

```
- (void)viewWillAppear:(BOOL)animated {
    
    [super viewWillAppear:animated];
    
    [_webView.configuration.userContentController addScriptMessageHandler:self name:@"jsToOc"];
}

- (void)viewWillDisappear:(BOOL)animated {
    
    [super viewWillDisappear:animated];
    
    [_webView.configuration.userContentController removeScriptMessageHandlerForName:@"jsToOc"];
}
```
>本文转载于：
作者：QiShare
链接：https://www.jianshu.com/p/905b40e609e2

> 另一篇ios调用JS https://www.jianshu.com/p/e23aa25d7514


