# iOS之UITableView性能优化
## 前言

### tableView的执行顺序

1. 调用代理方法确定几个section：`numberOfSectionsInTableView:`
2. 设置表头表尾高度（如果设置了header和footerVIew）`heightForHeaderInSection:`和`tableView:heightForFooterSection:`
3. 设置section内cell数：`numberOfRowsInSection:`
4. 设置cell：`cellForRowAtIndexPath:`
5. 设置Cell高度：`heightForRowAtIndexPath:`
6. cell将要显示在屏幕上：`willDisplayCell:forRowAtIndexPath:`
7. 