# iOS webView

## 1、UIWebView

* （iOS2.0 ~ iOS12.0）
* 内存系统性泄漏
* 系统OOM较多
* 稳定性差
* WebCore 和 JSCore Crash较多
* 对HTML5和CSS3支持较少

## 2、WKWebView （WebKit）

* （iOS8 +）
* 独立的进程，内存空间
* crash不影响主app
* 对HTML和CSS 更好的支持
* 更友好的系统函数
* 采用JIT技术