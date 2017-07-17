# FMDB
iOS 中原生的 SQLite API 在进行数据存储的时候，需要使用 C 语言中的函数，操作比较麻烦。于是，就出现了一系列将 SQLite API 进行封装的库，例如 FMDB、PlausibleDatabase 和 qlitepersistentobjects 等。

### FMDB 简介

#### 什么是 FMDB 
FMDB 是一款简洁、易用的封装库。因此，在这里推荐使用第三方框架 FMDB，它是对 libsqlite3 框架的封装，用起来的步骤与 SQLite 使用类似，并且它对于多线程的并发操作进行了处理，所以也是线程安全的。

#### FMDB 的优缺点
优点：
- 对多线程的并发操作进行处理，所以是线程安全的（重要特性之一）。
- 以 OC 的方式封装了 SQLite 的 C 语言 API，使用起来简洁、高效，没有原来的一大堆晦涩难懂、影响开发效率的 C 语句，更加面向对象。
- FMDB 是轻量级的框架，灵活易用。

缺点
- 因为它是 OC 的封装的，只能在 iOS 开发的时候使用，所以在实现跨平台操作的时候存在局限性。


### FMDB 框架中的重要类
- FMDatabase：一个 FMDatabase 对象就代表一个单独的 SQLite 数据库（注意并不是表），用来执行 SQL 语句。
- FMResultSet：使用 FMDatabase 执行查询后的结果集。
- FMDatabaseQueue：用于在多线程中执行多个查询或更新，线程安全的。


### FMDB 的详细使用步骤
#### 准备步骤
- 在工程中导入 FMDB，可以选择手动下载导入 [GitHub-FMDB](https://github.com/ccgus/fmdb) 或使用 [CocoaPods 导入](https://www.lewanny.com/2016/iOS-CocoaPods-usage-and-problem/)。
- 导入 libsqlite3.0 框架或 导入libsqlite3 框架（而这本质一致，libsqlite3.0 指向 libsqlite3）。
- 在需要用到 FMDB 的控制器（或模型）地方导入头文件 import FMDatabase.h 。
- 本地的 .sqlite 的查看，非常推荐火狐浏览器中的插件 [SQLite Manager](https://sqlitemanager.en.softonic.com/mac) 。

 <img src="http://omkug1guu.bkt.clouddn.com/iOS-thirdlib-FMDB-detailed-annotation/SqliteManager.png" width = 400 height=200>
 
 <img src="http://omkug1guu.bkt.clouddn.com/iOS-thirdlib-FMDB-detailed-annotation/SqliteManagerDetail.png" width = 400 height=200>
 
- 创建一个继承于 NSObject 的 Student 类，用来进行数据的增删改查。

```objc
#import <Foundation/Foundation.h>

@interface Student : NSObject

@property (nonatomic, assign) int num;       // 学号
@property (nonatomic, copy) NSString *name;  // 名字
@property (nonatomic, copy) NSString *sex;   // 性别
@property (nonatomic, assign) int age;       // 年龄

@end

```

#### 数据库的创建
```objc
@interface MainViewController () {
    FMDatabase *_db;     // FMDB 对象
    NSString *_docPath;  // 沙盒（数据库）路径
}
@end

// 1. 获取文件路径
_docPath = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject];
    NSLog(@"文件路径：%@", _docPath);
    
// 2. 数据文件名称
NSString *fileName = [_docPath stringByAppendingPathComponent:@"student.sqlite"];
    
// 3. 获取数据库
_db = [FMDatabase databaseWithPath:fileName];
if ([_db open]) {
	NSLog(@"打开数据库成功");
} else {
	NSLog(@"打开数据库失败");
}
```
然后可以打开 _docPath 的路径，可以看到名为 student.sqlite 的数据库已经创建好了。

#### 数据表的创建
```objc
// 创建表格
BOOL result = [_db executeUpdate:@"CREATE TABLE IF NOT EXISTS table_student (id integer PRIMARY KEY AUTOINCREMENT, name text NOT NULL, age integer NOT NULL, sex text NOT NULL);"];
if (result) {
	NSLog(@"创建数据表成功");
} else { 
	NSLog(@"创建数据表失败");
}
```
使用火狐浏览器的 SQLite Manager 插件打开 student.sqlite 可以看到table_student 表已经被创建好了。

#### 增加数据
```objc
// 插入数据
NSString *name = [NSString stringWithFormat:@"王胖子"];
int age = 36;
NSString *sex = @"男";
BOOL resurtInsert = [_db executeUpdate:@"INSERT INTO table_student (name, age, sex) VALUES (?,?,?)",name,@(age),sex];
if (resurtInsert) {
	NSLog(@"插入成功");
} else {
	NSLog(@"插入失败");
}
```

#### 删除数据
```objc
// 1.不确定的参数用？来占位 （后面参数必须是oc对象,需要将int包装成OC对象）
// int idNum = 1;
// BOOL result = [_db executeUpdate:@"DELETE FROM t_student WHERE id = ?",@(idNum)];
BOOL result = [_db executeUpdate:@"DELETE FROM table_student WHERE name = ?",@"王胖子"];
if (result) {
	NSLog(@"删除成功");
}
```

#### 修改数据
```objc
NSString *oldName = @"王胖子";
NSString *newNAme = @"无邪";
BOOL result = [_db executeUpdate:@"UPDATE table_student set name = ? WHERE name = ?", newNAme, oldName];
if (result) {
	NSLog(@"修改成功");
}
```

#### 查询数据
```objc
// FMResultSet *resultSet = [_db executeQuery:@"SELECT * FROM table_student"];
FMResultSet *resultSet = [_db executeQuery:@"SELECT * FROM table_student WHERE id < ?", @(4)];
while ([resultSet next]) {
	int idNum = [resultSet intForColumn:@"id"];
	NSString *name = [resultSet objectForColumn:@"name"];
	int age = [resultSet intForColumn:@"age"];
	NSString *sex = [resultSet objectForColumn:@"sex"];
	NSLog(@"学号：%@ 姓名：%@ 年龄：%@ 性别：%@",@(idNum),name,@(age),sex);
}
```

#### 表的删除
```objc
BOOL result = [_db executeUpdate:@"DROP TABLE IF EXISTS table_student"];
if (result) {
	NSLog(@"删除成功");
}
```
执行之后，刷新 SQLite，可以看到 table_student 表已经被删除了

 




 



