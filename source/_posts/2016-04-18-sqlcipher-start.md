---
title: SQLCipher数据库加解密
date: 2016-04-18 11:23
tags: [SQLCipher, SQLite]
categories: iOS
---

## 介绍

使用SQLite数据库的时候，有时候对于数据库要求比较高，特别是在iOS8.3之前，未越狱的系统也可以通过工具拿到应用程序沙盒里面的文件，这个时候我们就可以考虑对SQLite数据库进行加密，这样就不用担心sqlite文件泄露了

<!--more-->

通常数据库加密一般有两种方式
1. 对所有数据进行加密
2. 对数据库文件加密

> 第一种方式虽然加密了数据，但是并不完全，还是可以通过数据库查看到表结构等信息，并且对于数据库的数据，数据都是分散的，要对所有数据都进行加解密操作会严重影响性能，通常的做法是采取对文件加密的方式

iOS 免费版的sqlite库并不提供了加密的功能，SQLite只提供了加密的接口，但并没有实现，iOS上支持的加密库有下面几种

* [The SQLite Encryption Extension (SEE)](http://www.hwaci.com/sw/sqlite/see.html)
  * 收费，有以下几种加密方式
    > RC4  
    > AES-128 in OFB mode  
    > AES-128 in CCM mode  
    > AES-256 in OFB mode  


* [SQLiteEncrypt](http://www.sqlite-encrypt.com/)
  * 收费，使用AES加密
* [SQLiteCrypt](http://sqlite-crypt.com/index.htm)
  * 收费，使用256-bit AES加密
* [SQLCipher](http://sqlcipher.net/)
  * 开源，托管在[github](https://github.com/sqlcipher/sqlcipher)上，实现了SQLite官方的加密接口，也加了一些新的接口，详情参见[这里](https://www.zetetic.net/sqlcipher/sqlcipher-api/)

前三种都是收费的，SQLCipher是开源的，这里我们使用[SQLCipher](https://www.zetetic.net/sqlcipher/)

## 集成
如果你使用cocoapod的话就不需要自己配置了，为了方便，我们直接使用[FMDB](https://github.com/ccgus/fmdb)进行操作数据库，FMDB也支持SQLCipher

> pod 'FMDB/SQLCipher', '~> 2.6.2'


### 打开加密数据库
使用方式与原来的方式一样，只需要数据库open之后调用setKey设置一下秘钥即可  
下面摘了一段FMDatabase的open函数，在sqlite3_open成功后调用setKey方法设置秘钥

```objc
- (BOOL)open {
    if (_db) {
        return YES;
    }

    int err = sqlite3_open([self sqlitePath], &_db );
    if(err != SQLITE_OK) {
        NSLog(@"error opening!: %d", err);
        return NO;
    } else {
        //数据库open后设置加密key
        [self setKey:encryptKey_];
    }

    if (_maxBusyRetryTimeInterval > 0.0) {
        // set the handler
        [self setMaxBusyRetryTimeInterval:_maxBusyRetryTimeInterval];
    }

    return YES;
}
```
为了不修改FMDB的源代码，我们可以继承自FMDatabase类重写需要setKey的几个方法，这里我继承FMDatabase定义了一个 `FMEncryptDatabase` 类，提供打开加密文件的功能（具体定义见 [Demo](https://github.com/zhengbomo/sqlcipherDemo) ）

```objc
@interface FMEncryptDatabase : FMDatabase

+ (instancetype)databaseWithPath:(NSString*)aPath encryptKey:(NSString *)encryptKey;
- (instancetype)initWithPath:(NSString*)aPath encryptKey:(NSString *)encryptKey;

@end
```

用法与FMDatabase一样，只是需要传入secretKey

## SQLite数据库加解密
SQLCipher提供了几个命令用于加解密操作

#### 加密

```sqlite
$ ./sqlcipher plaintext.db  
sqlite> ATTACH DATABASE 'encrypted.db' AS encrypted KEY 'testkey';  
sqlite> SELECT sqlcipher_export('encrypted');  
sqlite> DETACH DATABASE encrypted;  
```

  1. 打开非加密数据库
  2. 创建一个新的加密的数据库附加到原数据库上
  3. 导出数据到新数据库上
  4. 卸载新数据库

#### 解密

```sqlite
$ ./sqlcipher encrypted.db  
sqlite> PRAGMA key = 'testkey';  
sqlite> ATTACH DATABASE 'plaintext.db' AS plaintext KEY '';  -- empty key will disable encryption
sqlite> SELECT sqlcipher_export('plaintext');  
sqlite> DETACH DATABASE plaintext;  
```

1. 打开加密数据库
2. 创建一个新的不加密的数据库附加到原数据库上
3. 导出数据到新数据库上
4. 卸载新数据库

### 代码操作
```objc

/** encrypt sqlite database to new file */
+ (BOOL)encryptDatabase:(NSString *)sourcePath targetPath:(NSString *)targetPath encryptKey:(NSString *)encryptKey
{
    const char* sqlQ = [[NSString stringWithFormat:@"ATTACH DATABASE '%@' AS encrypted KEY '%@';", targetPath, encryptKey] UTF8String];

    sqlite3 *unencrypted_DB;
    if (sqlite3_open([sourcePath UTF8String], &unencrypted_DB) == SQLITE_OK) {
        char *errmsg;
        // Attach empty encrypted database to unencrypted database
        sqlite3_exec(unencrypted_DB, sqlQ, NULL, NULL, &errmsg);
        if (errmsg) {
            NSLog(@"%@", [NSString stringWithUTF8String:errmsg]);
            sqlite3_close(unencrypted_DB);
            return NO;
        }

        // export database
        sqlite3_exec(unencrypted_DB, "SELECT sqlcipher_export('encrypted');", NULL, NULL, &errmsg);
        if (errmsg) {
            NSLog(@"%@", [NSString stringWithUTF8String:errmsg]);
            sqlite3_close(unencrypted_DB);
            return NO;
        }

        // Detach encrypted database
        sqlite3_exec(unencrypted_DB, "DETACH DATABASE encrypted;", NULL, NULL, &errmsg);
        if (errmsg) {
            NSLog(@"%@", [NSString stringWithUTF8String:errmsg]);
            sqlite3_close(unencrypted_DB);
            return NO;
        }

        sqlite3_close(unencrypted_DB);

        return YES;
    }
    else {
        sqlite3_close(unencrypted_DB);
        NSAssert1(NO, @"Failed to open database with message '%s'.", sqlite3_errmsg(unencrypted_DB));

        return NO;
    }
}

/** decrypt sqlite database to new file */
+ (BOOL)unEncryptDatabase:(NSString *)sourcePath targetPath:(NSString *)targetPath encryptKey:(NSString *)encryptKey
{
    const char* sqlQ = [[NSString stringWithFormat:@"ATTACH DATABASE '%@' AS plaintext KEY '';", targetPath] UTF8String];

    sqlite3 *encrypted_DB;
    if (sqlite3_open([sourcePath UTF8String], &encrypted_DB) == SQLITE_OK) {


        char* errmsg;

        sqlite3_exec(encrypted_DB, [[NSString stringWithFormat:@"PRAGMA key = '%@';", encryptKey] UTF8String], NULL, NULL, &errmsg);

        // Attach empty unencrypted database to encrypted database
        sqlite3_exec(encrypted_DB, sqlQ, NULL, NULL, &errmsg);

        if (errmsg) {
            NSLog(@"%@", [NSString stringWithUTF8String:errmsg]);
            sqlite3_close(encrypted_DB);
            return NO;
        }

        // export database
        sqlite3_exec(encrypted_DB, "SELECT sqlcipher_export('plaintext');", NULL, NULL, &errmsg);
        if (errmsg) {
            NSLog(@"%@", [NSString stringWithUTF8String:errmsg]);
            sqlite3_close(encrypted_DB);
            return NO;
        }

        // Detach unencrypted database
        sqlite3_exec(encrypted_DB, "DETACH DATABASE plaintext;", NULL, NULL, &errmsg);
        if (errmsg) {
            NSLog(@"%@", [NSString stringWithUTF8String:errmsg]);
            sqlite3_close(encrypted_DB);
            return NO;
        }

        sqlite3_close(encrypted_DB);

        return YES;
    }
    else {
        sqlite3_close(encrypted_DB);
        NSAssert1(NO, @"Failed to open database with message '%s'.", sqlite3_errmsg(encrypted_DB));

        return NO;
    }
}

/** change secretKey for sqlite database */
+ (BOOL)changeKey:(NSString *)dbPath originKey:(NSString *)originKey newKey:(NSString *)newKey
{
    sqlite3 *encrypted_DB;
    if (sqlite3_open([dbPath UTF8String], &encrypted_DB) == SQLITE_OK) {

        sqlite3_exec(encrypted_DB, [[NSString stringWithFormat:@"PRAGMA key = '%@';", originKey] UTF8String], NULL, NULL, NULL);

        sqlite3_exec(encrypted_DB, [[NSString stringWithFormat:@"PRAGMA rekey = '%@';", newKey] UTF8String], NULL, NULL, NULL);

        sqlite3_close(encrypted_DB);
        return YES;
    }
    else {
        sqlite3_close(encrypted_DB);
        NSAssert1(NO, @"Failed to open database with message '%s'.", sqlite3_errmsg(encrypted_DB));

        return NO;
    }
}
```  


## 总结
SQLCipher使用起来还是很方便的，基本上不需要怎么配置，需要注意的是，尽量不要在操作过程中修改secretKey，否则，可能导致读不了数据，在使用第三方库的时候尽量不去修改源代码，可以通过扩展或继承的方式修改原来的行为，这样第三方库代码可以与官方保持一致，可以跟随官方版本升级，具体代码可以到我的[github](https://github.com/zhengbomo/sqlcipherDemo)上下载咯

## 参考
* [http://www.cocoachina.com/industry/20140522/8517.html](http://www.cocoachina.com/industry/20140522/8517.html)
* [https://www.zetetic.net/sqlcipher/](https://www.zetetic.net/sqlcipher/)
