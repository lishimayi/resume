# UITableView

## 重用机制

- 重用池（ViewReusePool）原理图：

  ![图1](/11Source/table_reuse_pool.png)

- 上图的解释：
  - 渲染 cell 的时候，首先去池子里面查看是否有可重用 cell，有就拿出来用，没有就创建。
  - 当边缘 cell 滚动出屏幕显示区域是，会被放到重用，待用。

## 多线程下数据同步问题

### 场景

- 主线程删除 tableView 的数据源的一条数据时候，又触发了子线程加载，这样的话`删除`和`加载更多`会同事访问和修改数据，这时如何处理？

### 解决办法：

- 并发访问，数据拷贝
- 串行访问

### 两种方案详细介绍

- 并发访问，数据拷贝

  - 假设，删除数据在主线程，加载更多在子线程 A；
  - 在删除数据之前做数据 copy 到加载更多的 A 线程中去，并记录需要删除的数据
  - 主线程删除需要删除的数据并刷新
  - A 线程中，完成加载更多后，删除需要删除的数据后转到主线程刷新 UI。
  - 缺点：需要 copy，消耗资源。

- 串行访问
  - 假设删除数据在主线程，加载更多在子线程 A，串行同步队列 B；
  - 将加载更多拿到的数据和需要删除的数据都放到队列 B 的任务中；
  - 串行队列中，同步执行；加载更多和删除数都完成后，转到主线程刷新 UI。
  - 缺点：串行队列同步执行，如果删除或者加载更多都很耗时的话，整个过程就会很慢。

## tableView 的执行顺序

1. 调用代理方法确定几个 section：`numberOfSectionsInTableView:`
2. 设置表头表尾高度（如果设置了 header 和 footerVIew）`heightForHeaderInSection:`和`tableView:heightForFooterSection:`
3. 设置 section 内 cell 数：`numberOfRowsInSection:`
4. 设置 cell：`cellForRowAtIndexPath:`
5. 设置 Cell 高度：`heightForRowAtIndexPath:`
6. cell 将要显示在屏幕上：`willDisplayCell:forRowAtIndexPath:`
