https://github.com/ccgus/fmdb/blob/master/README.markdown

# FMDB

This is an Objective-C wrapper around SQLite: http://sqlite.org/

FMDB是用Objc对SQLite的封装。

Read the SQLite FAQ:

http://www.sqlite.org/faq.html

Since FMDB is built on top of SQLite, you're going to want to read this page top to bottom at least once. And while you're there, make sure to bookmark the SQLite Documentation page: http://www.sqlite.org/docs.html

尽管FMDB是基于SQLite实现，你也需要将本文从上到下读一遍。在你那最好为SQLite文档http://www.sqlite.org/docs.html做一个书签。

Contributing

Do you have an awesome idea that deserves to be in FMDB? You might consider pinging ccgus first to make sure he hasn't already ruled it out for some reason. Otherwise pull requests are great, and make sure you stick to the local coding conventions. However, please be patient and if you haven't heard anything from ccgus for a week or more, you might want to send a note asking what's up.


贡献

你是否有一个绝妙的点子在FMDB上？首先你确保他真的没有处理好这些原因然后你在考虑叮ccgus。另外pull是很好的，但是确保你坚持本地编码的约定。无论如何，如果你没有如果持续一周收到ccgus的回复请持有耐心，你可以发一个报告问一下什么状况



CocoaPods

FMDB can be installed using CocoaPods.

If using FMDB with SQLCipher you must use the FMDB/SQLCipher subspec. The FMDB/SQLCipher subspec declares SQLCipher as a dependency, allowing FMDB to be compiled with the -DSQLITE_HAS_CODEC flag.

CocoaPods

FMDB 可以使用CocoaPods工具进行安装。

如果你想在FMDB使用SQLCipher，你确保使用FMDB/SQLCipher subspec. FMDB/SQLCipher subspec 声明 SQLCipher 作为一个依赖。允许FMDB通过编译使用 -DSQLITE_HAS_CODEC 标示。

FMDB Class Reference:

http://ccgus.github.io/fmdb/html/index.html

Automatic Reference Counting (ARC) or Manual Memory Management?

You can use either style in your Cocoa project. FMDB will figure out which you are using at compile time and do the right thing.

FMDB 类 参考

http://ccgus.github.io/fmdb/html/index.html

自动引用计数还是手动内存管理？

你可以在你的项目中选择任意的方式。FMDB将在编译期知道你选择的方式，并作出正确的处理。


Usage

There are three main classes in FMDB:

FMDatabase - Represents a single SQLite database. Used for executing SQL statements.
FMResultSet - Represents the results of executing a query on an FMDatabase.
FMDatabaseQueue - If you're want to perform queries and updates on multiple threads, you'll want to use this class. It's described in the "Thread Safety" section below.


使用指南

下面是在FMDB中主要的类：

FMDatabase - 代表一个单独的数据库文件。用于执行SQL语句。
FMResultSet - 代表在FMDatabase里一个执行一次查询的结果集
FMDatabaseQueue - 如果你想要在多线程中解决问题或更新数据。你将会使用这个类。具体描述在 线程安全 这一章节下。


Database Creation

An FMDatabase is created with a path to a SQLite database file. This path can be one of these three:

A file system path. The file does not have to exist on disk. If it does not exist, it is created for you.
An empty string (@""). An empty database is created at a temporary location. This database is deleted with the FMDatabase connection is closed.
NULL. An in-memory database is created. This database will be destroyed with the FMDatabase connection is closed.
(For more information on temporary and in-memory databases, read the sqlite documentation on the subject: http://www.sqlite.org/inmemorydb.html)

	NSString *path = [NSTemporaryDirectory() stringByAppendingPathComponent:@"tmp.db"];
	FMDatabase *db = [FMDatabase databaseWithPath:path];

数据库创建

一个FMDatabase对象 通过一个指向SQLite数据文件的路径创建。这个路径可以是一下方式中的一个：

一个文件系统路径。在磁盘上这个文件不能存在。如果没有存在，它将为你创建一个。
一个空的字符串(@"")。在临时存储位置一个空数据库将被创建，当FMDatabase连接关闭时这个数据库将被删除。
NULL。 一个内存中的数据库被创建。当FMDatabase连接关闭时，这个数据库将被销毁。
（关于在临时位置和内存中的数据库的更多信息，可以阅读sqlite文档的这一章节http://www.sqlite.org/inmemorydb.html)



Opening

Before you can interact with the database, it must be opened. Opening fails if there are insufficient resources or permissions to open and/or create the database.

	if (![db open]) {
	    // [db release];   // uncomment this line in manual referencing code; in ARC, this is not necessary/permitted
	    db = nil;
	    return;
	}

打开数据库

在你使用数据库前，你确保它是打开的。打开失败可能是资源不足或者权限不足无法打开或者创建数据库。

	if (![db open]) {
	    // [db release];   // 在手动内存管理下取消注释。在ARC中，这样是不必要也是不被允许的。
	    db = nil;
	    return;
	}


Executing Updates

Any sort of SQL statement which is not a SELECT statement qualifies as an update. This includes CREATE, UPDATE, INSERT, ALTER, COMMIT, BEGIN, DETACH, DELETE, DROP, END, EXPLAIN, VACUUM, and REPLACE statements (plus many more). Basically, if your SQL statement does not begin with SELECT, it is an update statement.

Executing updates returns a single value, a BOOL. A return value of YES means the update was successfully executed, and a return value of NO means that some error was encountered. You may invoke the -lastErrorMessage and -lastErrorCode methods to retrieve more information.

执行更新操作

任何形式的SQL语句不是一条SELECT子句都看作一次更新。包括CREATE，UPDATE，INSERT，ALTER，COMMIT，BEGIN，DETACH，DROP，END，EXPLAIN，VACUUM，和REPLACE子句(或许还有更多)。基本上，如果你的SQL语句不是已SELECT开头，它都看作一条更新语句。

执行更新操作会返回一个BOOL类型的值。返回值YES表示这次更新成功执行，而返回值为NO表示遇到了一些错误。你可以调用-lastErrorMessage或-lastErrorCode 方法来获取更多的错误信息。

Executing Queries

A SELECT statement is a query and is executed via one of the -executeQuery... methods.

Executing queries returns an FMResultSet object if successful, and nil upon failure. You should use the -lastErrorMessage and -lastErrorCode methods to determine why a query failed.

In order to iterate through the results of your query, you use a while() loop. You also need to "step" from one record to the other. With FMDB, the easiest way to do that is like this:

	FMResultSet *s = [db executeQuery:@"SELECT * FROM myTable"];
	while ([s next]) {
	    //retrieve values for each record
	}

执行查询操作

一个SELECT语句表示一次查询，通过执行-executeQuery...方法。

执行查询操作如果成功会返回一个FMResultSet对象，失败的话返回nil。你应该使用-lastErrorMessage或-lastErrorCode来确定失败的原因。


为了彻底迭代你查询的结果集，你可以使用while()循环。你还会需要一些"步骤"从一条记录到另一条记录。在FDMB中，你可以用以下方式轻易的去实现。

	FMResultSet *s = [db executeQuery:@"SELECT * FROM myTable"];
	while ([s next]) {
	    //每条记录中的检索值
	}

You must always invoke -[FMResultSet next] before attempting to access the values returned in a query, even if you're only expecting one:

	FMResultSet *s = [db executeQuery:@"SELECT COUNT(*) FROM myTable"];
	if ([s next]) {
	    int totalCount = [s intForColumnIndex:0];
	}

当你尝试在一次查询中去访问结果集时你必须始终调用-[FMResultSet next]，即使你只执行一次：

	FMResultSet *s = [db executeQuery:@"SELECT COUNT(*) FROM myTable"];
	if ([s next]) {
	    int totalCount = [s intForColumnIndex:0];
	}

FMResultSet has many methods to retrieve data in an appropriate format:

	intForColumn:
	longForColumn:
	longLongIntForColumn:
	boolForColumn:
	doubleForColumn:
	stringForColumn:
	dateForColumn:
	dataForColumn:
	dataNoCopyForColumn:
	UTF8StringForColumnName:
	objectForColumnName:

Each of these methods also has a {type}ForColumnIndex: variant that is used to retrieve the data based on the position of the column in the results, as opposed to the column's name.

Typically, there's no need to -close an FMResultSet yourself, since that happens when either the result set is deallocated, or the parent database is closed.

FMResultSet 提供多种方法用内置的格式来获取数据

	intForColumn:
	longForColumn:
	longLongIntForColumn:
	boolForColumn:
	doubleForColumn:
	stringForColumn:
	dateForColumn:
	dataForColumn:
	dataNoCopyForColumn:
	UTF8StringForColumnName:
	objectForColumnName:

这些方法每一个都是{type}ForColumnIndex:的变体，获取数据通过在结果集中列的位置而不是列的名字。

通常情况下，一个FMResultSet不需要你自己来调用-close，因为这发生在结果集被销毁或者父数据库被关闭。

Closing

When you have finished executing queries and updates on the database, you should -close the FMDatabase connection so that SQLite will relinquish any resources it has acquired during the course of its operation.

	[db close];


关闭操作

当你在数据卡上完成执行查询操作活着更新操作时，你应当调用-close 来处理FMDatabase的连接，因为这样SQLite才会释放它在操作中使用的资源。


Transactions

FMDatabase can begin and commit a transaction by invoking one of the appropriate methods or executing a begin/end transaction statement.

事务操作

FMDatabase可以开始或提交一个事务通过调用一个内置的方法或执行开始／结束事务语句。


Multiple Statements and Batch Stuff

You can use FMDatabase's executeStatements:withResultBlock: to do multiple statements in a string:

	NSString *sql = @"create table bulktest1 (id integer primary key autoincrement, x text);"
	                 "create table bulktest2 (id integer primary key autoincrement, y text);"
	                 "create table bulktest3 (id integer primary key autoincrement, z text);"
	                 "insert into bulktest1 (x) values ('XXX');"
	                 "insert into bulktest2 (y) values ('YYY');"
	                 "insert into bulktest3 (z) values ('ZZZ');";

	success = [db executeStatements:sql];

	sql = @"select count(*) as count from bulktest1;"
	       "select count(*) as count from bulktest2;"
	       "select count(*) as count from bulktest3;";

	success = [self.db executeStatements:sql withResultBlock:^int(NSDictionary *dictionary) {
	    NSInteger count = [dictionary[@"count"] integerValue];
	    XCTAssertEqual(count, 1, @"expected one record for dictionary %@", dictionary);
	    return 0;
	}];


多条语句和批处理操作

你可以像这样在一行字符串使用FMDatabase的executeStatements:withResultBlock:方法来处理多条语句:

	NSString *sql = @"create table bulktest1 (id integer primary key autoincrement, x text);"
	                 "create table bulktest2 (id integer primary key autoincrement, y text);"
	                 "create table bulktest3 (id integer primary key autoincrement, z text);"
	                 "insert into bulktest1 (x) values ('XXX');"
	                 "insert into bulktest2 (y) values ('YYY');"
	                 "insert into bulktest3 (z) values ('ZZZ');";

	success = [db executeStatements:sql];

	sql = @"select count(*) as count from bulktest1;"
	       "select count(*) as count from bulktest2;"
	       "select count(*) as count from bulktest3;";

	success = [self.db executeStatements:sql withResultBlock:^int(NSDictionary *dictionary) {
	    NSInteger count = [dictionary[@"count"] integerValue];
	    XCTAssertEqual(count, 1, @"expected one record for dictionary %@", dictionary);
	    return 0;
	}];






