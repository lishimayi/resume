# iOS appDelegate 内都有哪些函数
## 第一个
 `- (BOOL)application:(UIApplication *)application didFinishLanchingWithOprations:(NSDictionary *)oprations;` 
 
 app 基础加载完成,可以进入开发人员自己的一些配置
##  第二个
 `- (void)applicationWillResignActive:(UIApplication *)application;`  
 
 app从活跃将转向非活跃状态, 例如来了电话, 消息. 或者换气任务管理,要清掉app的状态 
 
 这个时候我们需要暂停一些正在进行的任务: timers/ 降低OpenGL ES帧速率
 
## 第三个

`- (void)applicationDidEnterBackground:(UIApplication *)application;`

// Use this method to release shared resources, save user data, invalidate timers, and store enough application state information to restore your application to its current state in case it is terminated later.

* 使用这个方法释放被共享的资源
* 保存用户信息 
* 停止计时器
* 保存一些状态信息

// If your application supports background execution, this method is called instead of applicationWillTerminate: when the user quits.

* 如果你的app支持后台运行, 这个方法用来代替 `applicationWillTerminate:` 方法,当用户退出的时候

## 第四个

`- (void)applicationWillEnterForeground:(UIApplication *)application;`

app从后台进入非活跃的状态

在这里可以解开很多在进入后台的一些暂停的任务

## 第五个

`- (void)applicationDidBecomeActive:(UIApplication *)application;`

Restart any tasks that were paused (or not yet started) while the application was inactive. If the application was previously in the background, optionally refresh the user interface.

## 第六个

`- (void)applicationWillTerminate:(UIApplication *)application;`

Called when the application is about to terminate. Save data if appropriate. See also applicationDidEnterBackground:.


