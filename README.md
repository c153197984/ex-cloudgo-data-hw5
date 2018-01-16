# HW5：ex-cloudgo-data

---

## 准备工作
1. 配置MySQL服务器

   找到MySQL的安装包`mysql-5.7.17.msi`并下载安装。
   
   创建表格代码：

   ```SQL
   create database HW5;
   USE HW5;

   CREATE TABLE IF NOT EXISTS`userinfo` (
       `uid` INT(10) NOT NULL AUTO_INCREMENT,
       `username` VARCHAR(64) NULL DEFAULT NULL,
       `department` VARCHAR(64) NULL DEFAULT NULL,
       `createtime` DATE NULL DEFAULT NULL,
       PRIMARY KEY (`uid`)
   );

   CREATE TABLE IF NOT EXISTS `userdetail` (
       `uid` INT(10) NOT NULL DEFAULT '0',
       `intro` TEXT NULL,
       `profile` TEXT NULL,
       PRIMARY KEY (`uid`)
   );
   ```



## 操作指令

1.运行

    $ go run main.go


2.传递一个新用户

    $ curl -v -d "username=Alice&department=sdvx" localhost:8080/service/user

3.查询一个用户（通过ID）

    $ curl -v localhost:8080/service/user?uid=1

4.查询所有用户

    $ curl -v localhost:8080/service/user?uid=
    
## 工作原理

### 使用 `database/sql`

主要实现思想和Java数据库连接的思路相似。

对于实体User，我们有三个模块，分别是`entities /`中的`user_entity.go`，`user_dao.go`和`user_service.go`。

`user_entity`提供用户的定义。

`user_dao`封装了实体和数据库之间的操作，如插入和查询。 这些功能不能直接使用。

`user_service`导出在数据库中与用户操作相关的服务。 它取得`user_dao`中的函数，并在必要时提供。

要使用连接，只需调用`user_service`提供的服务。

### 使用 `xorm`

`xorm`是一个ORM框架，它丰富地支持各种数据库。

使用该框架，我们不需要自己创建任何模块。 我一需要做的就是调用`xorm`的功能。

如添加一个用户:

    _, err := mySQLEngine.Table("users").Insert(user)
    if err != nil {
        panic(err)
    }

ORM不只是实现一个自动DAO。 DAO只是一个或多个SQL语句对特定数据的封装。 ORM更像是通用SQL语句本身的封装。 它使我们可以轻松地对任何想要使用的数据执行SQL操作。


## 比较`xorm`和`database/sql`

    GO的`xorm`库将数据库的基本操作（增删查改）封装成了一个`engine`对象，操作数据不用编写DAO服务，全藉由`engine`对象所提供的方法来完成。不过由于`xorm`相当是对SQL的进一步封装，事实上其效率是不如SQL的。
    
    使用ab测试性能。
   
   测试语句为`ab -n 10000 -c 100 " $ ab -n 10000 -c 100 http://localhost:8080/service/user?uid="`。测试结果如下：

   对于`database/sql`实现版本：

   ```
   This is ApacheBench, Version 2.3 <$Revision: 1706008 $>
        Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
        Licensed to The Apache Software Foundation, http://www.apache.org/

        Benchmarking localhost (be patient)
        Completed 1000 requests
        Completed 2000 requests
        Completed 3000 requests
        Completed 4000 requests
        Completed 5000 requests
        Completed 6000 requests
        Completed 7000 requests
        Completed 8000 requests
        Completed 9000 requests
        Completed 10000 requests
        Finished 10000 requests

        Server Software:
        Server Hostname:        localhost
        Server Port:            8080

        Document Path:          /service/user?uid=
        Document Length:        461 bytes

        Concurrency Level:      100
        Time taken for tests:   2.330 seconds
        Complete requests:      10000
        Failed requests:        0
        Total transferred:      5850000 bytes
        HTML transferred:       4610000 bytes
        Requests per second:    4292.45 [#/sec] (mean)
        Time per request:       23.297 [ms] (mean)
        Time per request:       0.233 [ms] (mean, across all concurrent requests)
        Transfer rate:          2452.23 [Kbytes/sec] received

        Connection Times (ms)
                        min  mean[+/-sd] median   max
        Connect:        0    1   1.0      0       8
        Processing:     0   22  15.9     22     150
        Waiting:        0   22  15.9     21     149
        Total:          0   23  16.0     23     152

        Percentage of the requests served within a certain time (ms)
        50%     23
        66%     27
        75%     30
        80%     31
        90%     36
        95%     42
        98%     54
        99%     84
        100%    152 (longest request)
   ```

   对于`xorm`实现版本：

   ```
   This is ApacheBench, Version 2.3 <$Revision: 1706008 $>
        Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
        Licensed to The Apache Software Foundation, http://www.apache.org/

        Benchmarking localhost (be patient)
        Completed 1000 requests
        Completed 2000 requests
        Completed 3000 requests
        Completed 4000 requests
        Completed 5000 requests
        Completed 6000 requests
        Completed 7000 requests
        Completed 8000 requests
        Completed 9000 requests
        Completed 10000 requests
        Finished 10000 requests


        Server Software:
        Server Hostname:        localhost
        Server Port:            8080

        Document Path:          /service/user?uid=
        Document Length:        481 bytes

        Concurrency Level:      100
        Time taken for tests:   2.594 seconds
        Complete requests:      10000
        Failed requests:        0
        Total transferred:      6050000 bytes
        HTML transferred:       4810000 bytes
        Requests per second:    3854.58 [#/sec] (mean)
        Time per request:       25.943 [ms] (mean)
        Time per request:       0.259 [ms] (mean, across all concurrent requests)
        Transfer rate:          2277.36 [Kbytes/sec] received

        Connection Times (ms)
                      min  mean[+/-sd] median   max
        Connect:        0    1   1.0      0      10
        Processing:     1   25  11.7     25     219
        Waiting:        1   24  11.7     24     219
        Total:          1   26  11.8     26     219
        WARNING: The median and mean for the initial connection time are not within a normal deviation
                These results are probably not that reliable.

        Percentage of the requests served within a certain time (ms)
          50%     26
          66%     30
          75%     34
          80%     36
          90%     42
          95%     46
          98%     51
          99%     54
         100%    219 (longest request)
   ```

