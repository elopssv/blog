---
title: 全面解析-SQL事务+隔离级别+阻塞+死锁
date: 2016-08-27 18:15:18
tags: [sql,死锁,事务,隔离级别]
categories: sql
---
> 本篇主要是对SQL中事务和并发的详细讲解。

# 事务
## 什么是事务
为单个工作单元而执行的一系列操作。如查询、修改数据、修改数据定义。

## 语法
### 显示定义事务的开始、提交
````sql
BEGIN TRAN
INSERT INTO b(t1) VALUES(1)
INSERT INTO b(t1) VALUES(2)
COMMIT TRAN
````
### 隐式定义
如果不显示定义事务的边界，则SQL Server会默认把每个单独的语句作为一个事务，即在执行完每个语句之后就会自动提交事务。
<!-- more -->
## 事务的四个属性ACID
### 原子性Atomicity
[![](/images/sql-trasaction-isolation-deadlock-1.jpg)](/images/sql-trasaction-isolation-deadlock-1.jpg) 

1. 事务必须是原子工作单元。事务中进行的修改，要么全部执行，要么全都不执行；
2. 在事务完成之前（提交指令被记录到事务日志之前），系统出现故障或重新启动，SQL Server将会撤销在事务中进行的所有修改；
3. 事务在处理中遇到错误，SQL Server通常会自动回滚事务；
4. 少数不太严重的错误不会引发事务的自动回滚，如主键冲突、锁超时等；
5. 可以使用错误处理来捕获第4点提到的错误，并采取某种操作，如把错误记录在日志中，再回滚事务；
6. SELECT @@TRANCOUNT可用在代码的任何位置来判断当前使用SELECT @@TRANCOUNT的地方是否位于一个打开的事务当中，如果不在任何打开的事务范围内，则该函数返回0；如果在某个打开的事务返回范围内，则返回一个大于0的值。打开一个事务，@@TRANCOUNT=@@TRANCOUNT+1；提交一个事务，@@TRANCOUNT-1。

### 一致性Consiitency
[![](/images/sql-trasaction-isolation-deadlock-2.jpg)](/images/sql-trasaction-isolation-deadlock-2.jpg) 
1. 同时发生的事务在修改和查询数据时不发生冲突；
2. 一致性取决于应用程序的需要。后面会讲到一致性级别，以及如何对一致性进行控制。

### 隔离性Isolation
[![](/images/sql-trasaction-isolation-deadlock-3.jpg)](/images/sql-trasaction-isolation-deadlock-3.jpg) 
1. 用于控制数据访问，确保事务只访问处于期望的一致性级别下的数据；
2. 使用锁对各个事务之间正在修改和查询的数据进行隔离。

### 持久性Durability
[![](/images/sql-trasaction-isolation-deadlock-4.jpg)](/images/sql-trasaction-isolation-deadlock-4.jpg) 
1. 在将数据修改写入到磁盘上数据库的数据分区之前会把这些修改写入到磁盘上数据库的事务日志中，把提交指令记录到磁盘的事务日志中以后，及时数据修改还没有应用到磁盘的数据分区，也可以认为事务时持久化的。
2. 系统重新启动（正常启动或在发生系统故障之后启动），SQL Server会每个数据库的事务日志，进行回复处理。
3. 恢复处理包含两个阶段：重做阶段和撤销阶段。
4. 前滚：在重做阶段，对于提交指令已经写入到日志的事务，但数据修改还没有应用到数据分区的事务，数据库引擎会重做这些食物所做的所有修改。
5. 回滚：在撤销阶段，对于提交指令没有写入到日志中的事务，数据库引擎会撤销这些事务所做的修改。（这句话需要research,可能是不正确的。因为提交指令没有写入到数据分区，撤销修改是指撤销哪些修改呢？？？）

# 锁
## 事务中的锁
1. SQL Server使用锁来实现事务的隔离。
2. 事务获取锁这种控制资源，用于保护数据资源，防止其他事务对数据进行冲突的或不兼容的访问。

## 锁模式
1. 排他锁
    * 当试图修改数据时，事务只能为所依赖的数据资源请求排他锁。
    * 持有排他锁时间：一旦某个事务得到了排他锁，则这个事务将一直持有排他锁直到事务完成。
    * 排他锁和其他任何类型的锁在多事务中不能在同一阶段作用于同一个资源。如：当前事务获得了某个资源的排他锁，则其他事务不能获得该资源的任何其他类型的锁。其他事务获得了某个资源的任何其他类型的锁，则当前事务不能获得该资源的排他锁。
2. 共享锁
    * 当试图读取数据时，事务默认会为所依赖的数据资源请求共享锁。
    * 持有共享锁时间：从事务得到共享锁到读操作完成。
    * 多个事务可以在同一阶段用共享锁作用于同一数据资源。
    * 在读取数据时，可以对如何处理锁定进行控制。后面隔离级别会讲到如何对锁定进行控制。

## 排他锁和共享锁的兼容性
1. 如果数据正在由一个事务进行修改，则其他事务既不能修改该数据，也不能读取（至少默认不能）该数据，直到第一个事务完成。
2. 如果数据正在由一个事务读取，则其他事务不能修改该数据（至少默认不能）。

> to be continue..