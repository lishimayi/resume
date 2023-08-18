# iOS UITableView如何在多线程下修改或者同步数据

## 场景

主线程删除tableView的数据源的一条数据时候，又触发了子线程加载，这样的话`删除`和`加载更多`会同事访问和修改数据，这时如何处理？

### 1 并发访问，数据拷贝

假设，删除数据在主线程，加载更多在子线程A；

1. 在删除数据之前做数据copy到加载更多的A线程中去，并记录需要删除的数据；
2. 主线程删除需要删除的数据并刷新
3. A线程中，完成加载更多后，删除需要删除的数据后转到主线程刷新UI。

缺点：需要copy，消耗资源。

### 2 串行访问

假设删除数据在主线程，加载更多在子线程A，串行同步队列B；

1. 将加载更多拿到的数据和需要删除的数据都放到队列B的任务中；
2. 串行队列中，同步执行；加载更多和删除数都完成后，转到主线程刷新UI。

缺点：串行队列同步执行，如果删除或者加载更多都很耗时的话，整个过程就会很慢。

## 重用机制
* 重用池 ViewReusePool 
## 数据源同步
* 如何在tableview解决多线程情况下，数据处理
    * 并发访问，数据拷贝
    * 串行访问 

## tableView的执行顺序

1. 调用代理方法确定几个section：`numberOfSectionsInTableView:`
2. 设置表头表尾高度（如果设置了header和footerVIew）`heightForHeaderInSection:`和`tableView:heightForFooterSection:`
3. 设置section内cell数：`numberOfRowsInSection:`
4. 设置cell：`cellForRowAtIndexPath:`
5. 设置Cell高度：`heightForRowAtIndexPath:`
6. cell将要显示在屏幕上：`willDisplayCell:forRowAtIndexPath:`