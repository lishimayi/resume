# NSDictionary的底层实现原理

### 前言

* NSDictionary（字典）是使用 hash表来实现key和value之间的映射和存储的
* hash函数设计的好坏影响着数据的查找效率。
* 数据在hash表中分布越均匀，其访问效率越高
* OC语言中，通常都是利用NSString 来作为键值，其内部使用的hash函数也是通过使用 NSString对象作为键值来保证数据的各个节点在hash表中均匀分布。

### NSDictionary内部结构