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



