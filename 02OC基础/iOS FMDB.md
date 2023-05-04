# FMDB

## 什么是FMDB

FMDB是iOS平台的SQLite数据库框架，以OC的方式封装了SQLite的C语言API。

## FMDB的优点

* 使用起来更加面向对象，避免了C的复杂语法
* 对比官方的coreData ，更加轻量级和灵活
* 提供了多线程安全的数据库操作方法，有效防止数据混乱

## FMDB的使用

#### 通过cocopods

`pod 'FMDB'`

在项目引入头文件

`#import "FMDataBase.h"`

首先将数据存入沙盒下

```
NSString *docPath = @"";
docPath = [NSSearchPathDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject];
```

### 设置数据库名

```
// 设置数据库名称
NSString *fileName = [docPath stringByAppendingPathComponent:@"student.sqlite"];
// 创建并获取数据库信息
FMDataBase *fmdb = [FMDataBase dataWithPath:fileName];
// 尝试打开数据库
if ([fmdb open]) {
	NSLog(@"打开数据库成功！");
}else {
	NSLog(@"打开数据库失败！");
}

```

### 创建表

```
// PRIMARY KEY AUTOINCREMENT 基键 并且自增
BOOL executeUpdate = [fmdb executeUpdate:@"CREAT TABLE IF NOT EXISTS t_student (id interger PRIMARY KEY AUTOINCREMENT , name text NOT NULL, age integer NOT NULL, sex text NOT NULL);"];

if (excuteUpdate) {
	NSLog(@"创建表成功！");
}else {
	NSLog(@"创建表失败！");
}
```

### 数据添加（插入数据）

注意：

* executeUpdate：不确定的参数用`?`来站位（后面参数必须是OC对象），`;`代表语句结束。
* executeUpdateWithFormat：不确定的参数用`%@`、`%d`等来占位（参数为原始数据类型，执行语句不区分大小写）。

```
int mark_student = 1;
NSString *name = [NSString stringWithFormat:@"马冬梅%@",@(mark_student)];
int age = mark_student;
NSString *sex = @"女";
mark_student ++;
BOOL results = [fmdb executeUpdate:@"INSERT INTO t_student (name, age, sex) VALUES (?, ?, ?)", name, @(age), sex];
// BOOL result = [fmdb executeUpdateWithFormat:@"insert into t_student (name,age, sex) values (%@,%i,%@)",name,age,sex];

// 3 参数是数组的使用方式
//  BOOL result = [fmdb executeUpdate:@"INSERT INTO t_student(name,age,sex) VALUES  (?,?,?);" withArgumentsInArray:@[name,@(age),sex]];

if (results) {
	NSLog(@"插入成功！");
}else {
	NSLog(@"插入失败！");
}
```

### 删除数据

注意： （同上）

```
int idNum = 2;
// 按照id进行删除
BOOL delete = [fmdb executeUpdate:@"DELETE FROM t_tudent WHERE id = ?",@(idNum)];

//按照名字进行删除
//    BOOL deleate=[fmdb executeUpdateWithFormat:@"delete from t_student where name = %@",@"王子涵1"];
if (delete) {
    NSLog(@"删除成功");
}else{
    NSLog(@"删除失败");
}
```

### 修改数据

```
//修改学生的名字
NSString *newName = @"王小毛";
NSString *oldName = @"马冬梅3";
BOOL update=[fmdb executeUpdate:@"update t_student set name = ? where name =?",newName,oldName];
if (update) {
    NSLog(@"修改成功");
} else {
    NSLog(@"修改失败");
}
```

### 查询数据

```
// 查询整个表
FMResultSet *resultSet = [fmdb executeQuery:@"select * from t_student"];
// 按照条件查询
// FMResultSet *resultSet = [fmdb executeQuery:@"select * from t_student where id < ?", @4];

// 遍历结果集合

while ([resultSet next]) {
	int idNum  = [resultSet intForColumn:@"id"];
	NSString *name = [resultSet objectForColumnName:@"name"];
	int age = [resultSet intForColumn:@"age"];
	NSString *sex = [resultSet objectForColumnName:@"sex"];
	NSLog(@"学号：%@ 姓名：%@ 年龄：%@ 性别：%@",@(idNum),name,@(age),sex);
}

```

### 删除表

```
// 如果表存在就删除

BOOL result  = [fmdb executeUpdate:@"drop table if exists t_student"];
if (result) {
	NSLog(@"删除表成功！");
}else {
	NSLog(@"删除表失败！");
}
```
