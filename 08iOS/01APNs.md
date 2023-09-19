
# APNs 苹果推送服务

### 什么是APNs？

APNs 全称Apple Push Notification service翻译过来就是苹果推送通知服务，它是一个系统级别的推送服务即无论App是在前端，后台，还是被杀死状态都是可以接受到消息。

## 本我不讲具体的实现，只讲一些关键知识点

### 注册过程
* app --(UDID+bundleID)--> APNs
* app <---(deviceToken)---- APNs
* app --(deviceToken+uid)--> server

### 推送过程
* server --(deviceToken+contentD)--> APNs
* APNs --(deviceToken+content)--> app
