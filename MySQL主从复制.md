## 何谓Mysql主从复制？

MySQL主从复制是一个异步的复制过程，底层是基于Mysql数据库自带的二进制日志功能。就是一台或多台MySQL数据库(slave, 即从库)从另一台MySQL数据库(master, 即主库)进行日志的复制然后再解析日志并应用到自身，最终实现从库的数据和主库的数据保持一致。 MySQL主从复制是MySQL数据库自带功能，无需借助第三方工具。

## 复制过程

MySQL复制过程分成三步:

- master将 改变记录到二进制日志(binary log)
- slave将master的binary log拷贝到它的中继日志(relay log )
- slave重做中继日志中的事件，将改变应用到自己的数据库中

## 主从复制的意义

面对日益增加的系统访问量，数据库的吞吐量面临着巨大瓶颈，对于同一时刻有大量并发读操作和较少写操作的应用系统来说，将数据库拆分成主库和从库，主库负责增删改等事务性操作，从库负责处理查询操作，能够有效避免由数据更新导致的行锁，使得整个系统的查询性能大为改善。