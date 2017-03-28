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


Data Sanitization

When providing a SQL statement to FMDB, you should not attempt to "sanitize" any values before insertion. Instead, you should use the standard SQLite binding syntax:

INSERT INTO myTable VALUES (?, ?, ?, ?)
The ? character is recognized by SQLite as a placeholder for a value to be inserted. The execution methods all accept a variable number of arguments (or a representation of those arguments, such as an NSArray, NSDictionary, or a va_list), which are properly escaped for you.


数据卫生处理

当给FMDB执行一条SQL语句时，在插入值之前，你不应该是尝试给值‘消毒’。而是你应该使用标准SQLite绑定语法。

INSERT INTO myTable VALUES (?, ?, ?, ?)

？字符是SQLite在有值插入操作时指定的占位符。所有接受可变蚕食的方法(或者一个表示可变参数的比如NSArray， NSDictionary，或者va_list)，会对这些作为正确的转义。

And, to use that SQL with the ? placeholders from Objective-C:

	NSInteger identifier = 42;
	NSString *name = @"Liam O'Flaherty (\"the famous Irish author\")";
	NSDate *date = [NSDate date];
	NSString *comment = nil;

	BOOL success = [db executeUpdate:@"INSERT INTO authors (identifier, name, date, comment) VALUES (?, ?, ?, ?)", @(identifier), name, date, comment ?: [NSNull null]];
	if (!success) {
	    NSLog(@"error = %@", [db lastErrorMessage]);
	}

就像，在Objective-C 中你可以这样使用SQL 通过？占位符：
	
	NSInteger identifier = 42;
	NSString *name = @"Liam O'Flaherty (\"the famous Irish author\")";
	NSDate *date = [NSDate date];
	NSString *comment = nil;

	BOOL success = [db executeUpdate:@"INSERT INTO authors (identifier, name, date, comment) VALUES (?, ?, ?, ?)", @(identifier), name, date, comment ?: [NSNull null]];
	if (!success) {
	    NSLog(@"error = %@", [db lastErrorMessage]);
	}


	Note: Fundamental data types, like the NSInteger variable identifier, should be as a NSNumber objects, achieved by using the @ syntax, shown above. Or you can use the [NSNumber numberWithInt:identifier] syntax, too.

	Likewise, SQL NULL values should be inserted as [NSNull null]. For example, in the case of comment which might be nil (and is in this example), you can use the comment ?: [NSNull null] syntax, which will insert the string if comment is not nil, but will insert [NSNull null] if it is nil.


	提示: 基本数据类型，像NSInterger的变量，应该用一个NSNumber对象代替，你可以像上面的例子那样，通过使用@语法完成。或者你也可以使用[NSNumber numberWithInt:identifier]语法。
	同样，在SQL中NULL值应当用[NSNull null]代替。例如，在comment也许为空的情况下(像之前那个例子)，你可以使用 comment ?: [NSNull null]语法，它将插入一条字符串如果comment不是空的话，否则将插入[NSNull null]。


In Swift, you would use executeUpdate(values:), which not only is a concise Swift syntax, but also throws errors for proper Swift 2 error handling:

	do {
	    let identifier = 42
	    let name = "Liam O'Flaherty (\"the famous Irish author\")"
	    let date = NSDate()
	    let comment: String? = nil

	    try db.executeUpdate("INSERT INTO authors (identifier, name, date, comment) VALUES (?, ?, ?, ?)", values: [identifier, name, date, comment ?? NSNull()])
	} catch {
	    print("error = \(error)")
	}
Note: In Swift, you don't have to wrap fundamental numeric types like you do in Objective-C. But if you are going to insert an optional string, you would probably use the comment ?? NSNull() syntax (i.e., if it is nil, use NSNull, otherwise use the string).


在Swift中，你可以使用executeUpdate(values:)， 这不仅是一条简介的Swift语句，并且捕获错误利用Swift2特有的错误处理:

	do {
	    let identifier = 42
	    let name = "Liam O'Flaherty (\"the famous Irish author\")"
	    let date = NSDate()
	    let comment: String? = nil

	    try db.executeUpdate("INSERT INTO authors (identifier, name, date, comment) VALUES (?, ?, ?, ?)", values: [identifier, name, date, comment ?? NSNull()])
	} catch {
    	print("error = \(error)")
	}

提示: 在Swift中，你不用像Objective—C中那样包裹一个基本数据类型。但是如果你要插入一个可选的字符串，你应该使用comment ?? NSNull()语法(也就是，如果为空，使用Null，反之使用字符串)。

Alternatively, you may use named parameters syntax:

	INSERT INTO authors (identifier, name, date, comment) VALUES (:identifier, :name, :date, :comment)

The parameters must start with a colon. SQLite itself supports other characters, but internally the dictionary keys are prefixed with a colon, do not include the colon in your dictionary keys.

	NSDictionary *arguments = @{@"identifier": @(identifier), @"name": name, @"date": date, @"comment": comment ?: [NSNull null]};
	BOOL success = [db executeUpdate:@"INSERT INTO authors (identifier, name, date, comment) VALUES (:identifier, :name, :date, :comment)" withParameterDictionary:arguments];
	if (!success) {
	    NSLog(@"error = %@", [db lastErrorMessage]);
	}
The key point is that one should not use NSString method stringWithFormat to manually insert values into the SQL statement, itself. Nor should one Swift string interpolation to insert values into the SQL. Use ? placeholders for values to be inserted into the database (or used in WHERE clauses in SELECT statements).


或者，你也可以使用指定参数的语法：
	
	NSERT INTO authors (identifier, name, date, comment) VALUES (:identifier, :name, :date, :comment)

指定的参数必须以冒号开头，虽然SQLite自身支持其它字符，但是在内部dictionary的key都有一个冒号前缀，因此不要包含冒号在你dictionary的keys中。

关键是我们不需要使用字符串方法stringWithFormat去手动插入值到SQL语句中，也不是，应该使用Swift插入值到SQL语句中。而是要给要插入数据库(或在SQL语句中使用WHRER字句)的值使用？占位符。

Using FMDatabaseQueue and Thread Safety.

Using a single instance of FMDatabase from multiple threads at once is a bad idea. It has always been OK to make a FMDatabase object per thread. Just don't share a single instance across threads, and definitely not across multiple threads at the same time. Bad things will eventually happen and you'll eventually get something to crash, or maybe get an exception, or maybe meteorites will fall out of the sky and hit your Mac Pro. This would suck.

使用FMDatabaseQueue 和 线程安全

在多线程中使用一个FMDatabase单例绝对是一个糟糕的主意。在每一个线程中都创建一个FMDatabase对象是可以的。千万不要在线程中共享一个实例对象， 除非保证在同时不存在于多个线程中。不然，可怕的情况下将发生然后你的应用将崩溃，或许获得一个异常，或许陨石从天上掉下来然后砸到你的Mac Pro。多么糟糕。


	So don't instantiate a single FMDatabase object and use it across multiple threads.

Instead, use FMDatabaseQueue. Instantiate a single FMDatabaseQueue and use it across multiple threads. The FMDatabaseQueue object will synchronize and coordinate access across the multiple threads. Here's how to use it:

First, make your queue.

FMDatabaseQueue *queue = [FMDatabaseQueue databaseQueueWithPath:aPath];
Then use it like so:

	[queue inDatabase:^(FMDatabase *db) {
	    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", @1];
	    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", @2];
	    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", @3];

	    FMResultSet *rs = [db executeQuery:@"select * from foo"];
	    while ([rs next]) {
	        …
	    }
	}];

	因此千万不要实例化一个FMDatabase单例对象在多个线程中访问它。

相反的，使用FMDatabaseQueue。实例化一个FMDatabaseQueue对象然后在多线程中访问它。FMDatabaseQueue对象将在多个线程中同步并协调去访问。下面告诉你如何使用：

首先，创建你的队列。
FMDatabaseQueue *queue = [FMDatabaseQueue databaseQueueWithPath:aPath];
然后像这样使用：

	[queue inDatabase:^(FMDatabase *db) {
	    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", @1];
	    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", @2];
	    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", @3];

	    FMResultSet *rs = [db executeQuery:@"select * from foo"];
	    while ([rs next]) {
	        …
	    }

An easy way to wrap things up in a transaction can be done like this:

	[queue inTransaction:^(FMDatabase *db, BOOL *rollback) {
	    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", @1];
	    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", @2];
	    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", @3];

	    if (whoopsSomethingWrongHappened) {
	        *rollback = YES;
	        return;
	    }

	    // etc ...
	}];

The Swift 3 equivalent would be:

	queue.inTransaction { db, rollback in
	    do {
	        try db?.executeUpdate("INSERT INTO myTable VALUES (?)", values: [1])
	        try db?.executeUpdate("INSERT INTO myTable VALUES (?)", values: [2])
	        try db?.executeUpdate("INSERT INTO myTable VALUES (?)", values: [3])

	        if whoopsSomethingWrongHappened {
	            rollback?.pointee = true
	            return
	        }

	        // etc ...
	    } catch {
	        rollback?.pointee = true
	        print(error)
	    }
	}
(Note, in Swift 3, use pointee. But in Swift 2.3, use memory rather than pointee.)

FMDatabaseQueue will run the blocks on a serialized queue (hence the name of the class). So if you call FMDatabaseQueue's methods from multiple threads at the same time, they will be executed in the order they are received. This way queries and updates won't step on each other's toes, and every one is happy.

Note: The calls to FMDatabaseQueue's methods are blocking. So even though you are passing along blocks, they will not be run on another thread.
	}];


像下面这样用一个简单的方式去包裹事务操作:

	[queue inTransaction:^(FMDatabase *db, BOOL *rollback) {
	    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", @1];
	    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", @2];
	    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", @3];

	    if (whoopsSomethingWrongHappened) {
	        *rollback = YES;
	        return;
	    }

	    // etc ...
	}];

Swift3中相同的用法:

	queue.inTransaction { db, rollback in
	    do {
	        try db?.executeUpdate("INSERT INTO myTable VALUES (?)", values: [1])
	        try db?.executeUpdate("INSERT INTO myTable VALUES (?)", values: [2])
	        try db?.executeUpdate("INSERT INTO myTable VALUES (?)", values: [3])

	        if whoopsSomethingWrongHappened {
	            rollback?.pointee = true
	            return
	        }

	        // etc ...
	    } catch {
	        rollback?.pointee = true
	        print(error)
	    }
	}

(提示， 在Swift3中使用指针，但是在Swift2.3中使用内存而不是指针)

FMDatabaseQueue 将会在一个串行队列执行这些Blocks(可能是Class为命名)，因此，如果你在多线程中同时调用FMDatabaseQueue的方法，他们将按照响应的顺序被处理，这样方式的查询和更新就不会出现踩到其他人脚趾，皆大欢喜。

注意，FMDatabaseQueue's methods的调用是一个block。因此尽管你和block一同调用，但是block也不会运行在另外一个线程中。


Making custom sqlite functions, based on blocks.

You can do this! For an example, look for -makeFunctionNamed: in main.m

制作自定义的sqlite放大，基于blocks

你可以这样做，下面就有一个一例子，详情在main.m 中的 -makeFunctionNamed:



Swift

You can use FMDB in Swift projects too.

To do this, you must:

1. Copy the relevant .m and .h files from the FMDB src folder into your project.
You can copy all of them (which is easiest), or only the ones you need. Likely you will need FMDatabase and FMResultSet at a minimum. FMDatabaseAdditions provides some very useful convenience methods, so you will likely want that, too. If you are doing multithreaded access to a database, FMDatabaseQueue is quite useful, too. If you choose to not copy all of the files from the src directory, though, you may want to update FMDB.h to only reference the files that you included in your project.

Note, if you're copying all of the files from the src folder into to your project (which is recommended), you may want to drag the individual files into your project, not the folder, itself, because if you drag the folder, you won't be prompted to add the bridging header (see next point).

2. If prompted to create a "bridging header", you should do so. If not prompted and if you don't already have a bridging header, add one.
For more information on bridging headers, see Swift and Objective-C in the Same Project.

3. In your bridging header, add a line that says:

 #import "FMDB.h"
4. Use the variations of executeQuery and executeUpdate with the sql and values parameters with try pattern, as shown below. These renditions of executeQuery and executeUpdate both throw errors in true Swift 2 fashion.


Swift

你也可以在Swift项目中使用FMDB。
想要这样做，你必须：

1. 复制相关的.m和.h文件从FMDB src文件夹中到你的项目中。
你能复制所有(这最容易)，或者仅仅是你需要的。比如，可能仅仅需要FMDatabase，FMResultSet。FMDatabaseAdditions提供了很多有用的便利方法，因此你可能喜欢使用它们。如果你需要在多线程中访问数据库，FMDatabaseQueue也是非常有用的。如果你不是在src目录中复制所有的文件，那么你需要修改FMDB.h成哪些导入到你项目中引用的文件。

注意，如果你从src文件中复制所有的文件到你的项目里(推荐这样做)，你可能会拖一个文件夹到你的项目中，不要整个文件夹，只有本身，因为如果你拖文件夹，你将不会提示你需要添加桥接文件（看下一步）

2. 如果提示创建'桥接文件'，你应该照做。如果没有提示或者你没有准备好一个桥接文件，在里面添加一个。关于桥接头文件更多信息，看Swift and Objective-C 命名的项目。

3. 在你的桥接头文件中，添加如下一行:
 #import "FMDB.h"
4. 像下面这样，执行executeQuery和executeUpdate方式时尝试用模式匹配执行Sql和数据参数。使用这些executeQuery和executeUpdate方法时异常处理方式使用Swift2的新特性。


If you do the above, you can then write Swift code that uses FMDatabase. For example, in Swift 3:

let fileURL = try! FileManager.default
    .url(for: .documentDirectory, in: .userDomainMask, appropriateFor: nil, create: false)
    .appendingPathComponent("test.sqlite")

guard let database = FMDatabase(path: fileURL.path) else {
    print("unable to create database")
    return
}

guard database.open() else {
    print("Unable to open database")
    return
}

do {
    try database.executeUpdate("create table test(x text, y text, z text)", values: nil)
    try database.executeUpdate("insert into test (x, y, z) values (?, ?, ?)", values: ["a", "b", "c"])
    try database.executeUpdate("insert into test (x, y, z) values (?, ?, ?)", values: ["e", "f", "g"])

    let rs = try database.executeQuery("select x, y, z from test", values: nil)
    while rs.next() {
        if let x = rs.string(forColumn: "x"), let y = rs.string(forColumn: "y"), let z = rs.string(forColumn: "z") {
            print("x = \(x); y = \(y); z = \(z)")
        }
    }
} catch {
    print("failed: \(error.localizedDescription)")
}

database.close()

如果你完成以上，你可以写Swift代码来使用FMDatabase， 例如，在Swift3中:

let fileURL = try! FileManager.default
    .url(for: .documentDirectory, in: .userDomainMask, appropriateFor: nil, create: false)
    .appendingPathComponent("test.sqlite")

guard let database = FMDatabase(path: fileURL.path) else {
    print("unable to create database")
    return
}

guard database.open() else {
    print("Unable to open database")
    return
}

do {
    try database.executeUpdate("create table test(x text, y text, z text)", values: nil)
    try database.executeUpdate("insert into test (x, y, z) values (?, ?, ?)", values: ["a", "b", "c"])
    try database.executeUpdate("insert into test (x, y, z) values (?, ?, ?)", values: ["e", "f", "g"])

    let rs = try database.executeQuery("select x, y, z from test", values: nil)
    while rs.next() {
        if let x = rs.string(forColumn: "x"), let y = rs.string(forColumn: "y"), let z = rs.string(forColumn: "z") {
            print("x = \(x); y = \(y); z = \(z)")
        }
    }
} catch {
    print("failed: \(error.localizedDescription)")
}

database.close()


Or in Swift 2:

let fileURL = try! NSFileManager.defaultManager()
    .URLForDirectory(.DocumentDirectory, inDomain: .UserDomainMask, appropriateForURL: nil, create: false)
    .URLByAppendingPathComponent("test.sqlite")

let database = FMDatabase(path: fileURL.path)

if !database.open() {
    print("Unable to open database")
    return
}

do {
    try database.executeUpdate("create table test(x text, y text, z text)", values: nil)
    try database.executeUpdate("insert into test (x, y, z) values (?, ?, ?)", values: ["a", "b", "c"])
    try database.executeUpdate("insert into test (x, y, z) values (?, ?, ?)", values: ["e", "f", "g"])

    let rs = try database.executeQuery("select x, y, z from test", values: nil)
    while rs.next() {
        let x = rs.stringForColumn("x")
        let y = rs.stringForColumn("y")
        let z = rs.stringForColumn("z")
        print("x = \(x); y = \(y); z = \(z)")
    }
} catch let error as NSError {
    print("failed: \(error.localizedDescription)")
}

database.close()

或着Swift2中:

let fileURL = try! NSFileManager.defaultManager()
    .URLForDirectory(.DocumentDirectory, inDomain: .UserDomainMask, appropriateForURL: nil, create: false)
    .URLByAppendingPathComponent("test.sqlite")

let database = FMDatabase(path: fileURL.path)

if !database.open() {
    print("Unable to open database")
    return
}

do {
    try database.executeUpdate("create table test(x text, y text, z text)", values: nil)
    try database.executeUpdate("insert into test (x, y, z) values (?, ?, ?)", values: ["a", "b", "c"])
    try database.executeUpdate("insert into test (x, y, z) values (?, ?, ?)", values: ["e", "f", "g"])

    let rs = try database.executeQuery("select x, y, z from test", values: nil)
    while rs.next() {
        let x = rs.stringForColumn("x")
        let y = rs.stringForColumn("y")
        let z = rs.stringForColumn("z")
        print("x = \(x); y = \(y); z = \(z)")
    }
} catch let error as NSError {
    print("failed: \(error.localizedDescription)")
}

database.close()


History

The history and changes are availbe on its GitHub page and are summarized in the "CHANGES_AND_TODO_LIST.txt" file.

Contributors

The contributors to FMDB are contained in the "Contributors.txt" file.

Additional projects using FMDB, which might be interesting to the discerning developer.

FMDBMigrationManager, A SQLite schema migration management system for FMDB: https://github.com/layerhq/FMDBMigrationManager
FCModel, An alternative to Core Data for people who like having direct SQL access: https://github.com/marcoarment/FCModel


历史记录

历史和更改记录可以在它的GitHub页面可以找到并汇总在CHANGES_AND_TODO_LIST.txt 文件中。

FMDB扩展项目,这些可能会让开发者感兴趣

FMDBMigrationManager 一个FMDB的SQLite 模式迁移管理系统 https://github.com/layerhq/FMDBMigrationManager
FCModel， 一个像CoreData(有些人或许喜欢)那样访问SQL访问 https://github.com/marcoarment/FCModel

Quick notes on FMDB's coding style

Spaces, not tabs. Square brackets, not dot notation. Look at what FMDB already does with curly brackets and such, and stick to that style.

FMDB编程风格快速笔记

空格，不要Tab。统一使用[符号，不要点符号。像FMDB那样对折[号等等，坚持风格。

Reporting bugs

Reduce your bug down to the smallest amount of code possible. You want to make it super easy for the developers to see and reproduce your bug. If it helps, pretend that the person who can fix your bug is active on shipping 3 major products, works on a handful of open source projects, has a newborn baby, and is generally very very busy.

And we've even added a template function to main.m (FMDBReportABugFunction) in the FMDB distribution to help you out:

Open up fmdb project in Xcode.
Open up main.m and modify the FMDBReportABugFunction to reproduce your bug.
Setup your table(s) in the code.
Make your query or update(s).
Add some assertions which demonstrate the bug.
Then you can bring it up on the FMDB mailing list by showing your nice and compact FMDBReportABugFunction, or you can report the bug via the github FMDB bug reporter.

Optional:

Figure out where the bug is, fix it, and send a patch in or bring that up on the mailing list. Make sure all the other tests run after your modifications.



报告问题



