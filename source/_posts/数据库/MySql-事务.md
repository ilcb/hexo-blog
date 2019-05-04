---
layout: _post
title: "MySql事务"
date: 2019-04-30
tags: "mysql"
categories: "mysql"
---
### MySql中怎么实现事务
#### mysql中事务相关的语句：
##### autocommit
autocommit设置事务是否自动提交(mysql默认为自动提交)
```sql
## 查看事务是否自动提交
show global variables like 'autocommit%';
show session variables like 'autocommit%';
## 设置是否自动开启事务提交
set autocommit = 0; -- 手动提交
set autocommit = 1; --自动提交
```
##### start transaction或begin
开始一个事务，需要注意虽然start transaction和begin都表示开启事务，但是在存储过程中将begin识别为和end搭配使用，所以存储过程中只能使用start transaction;
##### commit或commit work
提交一个事务，从begin/start transaction到commit之间的sql语句都会被执行
##### rollback或者rollback work
表示回滚事务，将会撤销所有未提交的修改并结束当前事务
##### savepoint 标识符
savepoint XX，表示创建一个事务的保存点XX，以便回滚到当前保存点，而不是回滚整个事务
##### rollback to 标识符
rollback to XX，把事务回滚到标记点，而不回滚在此标记点之前的任何工作。需要和SAVEPOINT命令一起使用，例如可以发出两条UPDATE语句，后面跟一个SAVEPOINT，然后又是两条DELETE语句。如果执行DELETE语句期间出现了某种异常情况，而且你捕获到这个异常，并发出ROLLBACK TO SAVEPOINT命令，事务就会回滚到指定的SAVEPOINT，撤销DELETE完成的所有工作，而UPDATE语句完成的工作不受影响
##### release savepoint 标识符
release savepoint XX，表示删除事务保存点XX
#### 实例
使用的表sql如下：
```sql
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(12) DEFAULT '',
  `age` int(11) DEFAULT '0',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```
##### 正常事务提交
![normal](normal.png)
开启了一个事务，然后向user表插入了2条数据，提交，数据持久化到表中；
##### 回滚
![rollback](rollback.png)
开启了一个新的事务，删除了插入的2条数据，然后rollback了，最终数据仍然在表中；
##### 回滚到保存点
![savepoint](https://ws4.sinaimg.cn/large/006tNc79gy1g2msqffqvpj30i011042p.jpg)
1. 删除了id为1的记录，创建了保存点delete1
2. 插入了id为3的记录，创建了保存点insert1
3. 删除了id为2的记录
回滚到保存点delete1，步骤2、3中的操作回滚，delete1之前的操作保留
### MySql事务实现原理
在MySql中，InnoDB存储引擎是支持事务的，一个事务必须满足4大特性：
原子性：整个事务的所有操作要么全部成功，要么全部失败，在MySql中通过redo log日志实现
一致性：数据库总是从一个一执行状态变为另一个一致性状态，在MySql中通过undo log实现
隔离型：一个事务在提交之前所做出的操作是否能为其他事务可见，有不同的隔离级别，在MySql中通过锁及MVCC来实现
持久性：事务一旦提交，事务所做的修改会被永久保存，即使数据库崩溃，修改的数据也不会丢失，在MySql中通过redo log日志实现
#### mysql事务日志参数
```sql
show global variables like '%innodb%log%';
```
![log](log.png)
**innodb_log_file_size**：表示每个redo log file文件的大小，单位为byte
**innodb_log_files_in_group**：表示每个redo log group中有几个redo log file
**innodb_log_group_home_dir**：表示redo log group的文件所在路径，默认./ 表示在数据库的数据目录下（默认情况下，对应的物理文件位于数据库的data目录下的ib_logfile1&ib_logfile2）
**Innodb_flush_log_at_trx_commit**：表示当事务提交之后是否立即将redo log从log buffer刷新到redo log file中，此配置可以配置3个值
默认值为1，表示事务提交时必须将redo log从log buffer中写入到redo log file中（事务提交 -> redo log buffer -> os buffer -> log file）
如果值设置为0，事务提交时并不会将redo log从redo log buffer刷新到redo log file中，但是会每秒钟自动将redo log从redo log buffer刷新到redo log file中，可以理解为当事务提交时，redo log存在于redo log buffer，每秒钟redo log从redo log buffer中经过os buffer刷新到redo log file中一次，这种情况下，如果数据库崩溃，最多丢失1s的redo log。
如果值设置为2，表示在事务提交时，只会将redo log写入到系统缓存（os buffer）中，但是不会立即写入到redo log file中，而是每秒中从os buffer中刷新到redo log file中，可以理解为当日志提交时，redo log存在于redo log buffer和os buffer中，每秒钟redo log从os buffer中刷新到redo log file中1次，这种情况下，如果只是数据库宕机，操作系统未宕机，数据不会丢失，如果此时操作习题哦那个宕机，重启数据库后会丢失还未从os buffer中刷新到redo log file中的事务操作；
#### redo log
mysql会将事务中的sql语句涉及的所有数据操作先记录到redo log中，然后再将操作从redo log同步到相应数据文件中，即在事务提交成功，修改数据文件中的记录之前，要保证对应的所有修改操作都已经记录到了redo log中，所以，即使数据文件中的数据被修改到一半时被打断，也能依靠redo log中的日志将剩余部分的操作再次同步到相应的数据文件中。redo log是物理日志，记录的是数据库对页的操作，而不是逻辑上的增删改查，具有幂等性。
使用redo log，能够确保持久性和原子性，即事务中的所有sql被当成一个执行单元。
redo log由2部分组成：redo log buffer(重做日志缓冲)和redo log file(重做日志文件)，redo log buffer存于内存之中，redo log file永久存储于磁盘。
事务操作执行：
1. 事务修改操作记录到redo log buffer
2. 事务修改操作记录到redo log file
3. 事务修改操作更新到数据库数据文件中
步骤1的速度很快，但是因为内存中的数据是易失的，无法满足持久性的要求，为了满足持久性，需要进行第2步，将redo log buffer中的日志写入到redo log file中，相当于从内存中同步到磁盘上，再执行第3步操作将日志记录的操作同步到数据文件中；
#### undo log
undo log理解为数据被修改前的备份，如果事务进行了一半，有1条sql没有执行成功，数据库可以依据undo log进行撤销，undo并不能将数据库物理地恢复到执行语句或者事务之前的样子，它是逻辑日志，当回滚日志被使用时，它只会按照日志逻辑地将数据库中的修改撤销掉，可以理解为，我们在事务中使用的每一条 INSERT 都对应了一条 DELETE，每一条 UPDATE 也都对应一条相反的 UPDATE 语句，MySql使用redo log实现事务的持久性。
#### log group
Log  group为重做日志组，其中有多个重做日志文件（redo log file）,当日志组中的第1个logfile被写满，会写下一个重做日志文件，当日志组中所有redo log file被写满，将redo log覆盖写入第一个redo log file.。