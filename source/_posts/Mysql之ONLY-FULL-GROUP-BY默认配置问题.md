title: Mysql之ONLY_FULL_GROUP_BY默认配置问题
tags:
  - 数据库
categories:
  - 技术
date: 2020-03-11 10:58:00
cover: true

---

![](http://q6pznk9ej.bkt.clouddn.com/haishanggangqinshijpg.jpg)
<!-- more -->
## 错误
```
### Cause: 
com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Expression #3 of SELECT list is 
not in GROUP BY clause and contains nonaggregated column 'fhshgl.ii.COMMODITYDETAILPRICE_ID' 
which is not functionally dependent on columns in GROUP BY clause; 
this is incompatible with sql_mode=only_full_group_by
```
## 原因
在把`MySQL`升级到`5.7`或者更高的版本，一些以前看上去不会出错的`group by` 操作在这个版本以后就会出现语法报错的情况：

在这个模式下，我们使用分组查询时，出现在`select`字段后面的只能是`group by`后面的分组字段，或使用聚合函数包裹着的字段。

`Oracled`等数据库都不支持`select target list`中出现语义不明确的列，这样的语句在这些数据库中是会被报错的，所以从`MySQL 5.7`版本开始修正了这个语义，就是所说的`ONLY_FULL_GROUP_BY`语义。

因为有`only_full_group_by`，所以我们要在`MySQL`中正确的使用`group by`语句的话，只能是`select column1 from tb1 group by column1`(即只能展示group by的字段，其他均都要报1055的错)

## 解决
### 暂时性关闭
可以通过`select @@sql_mode`查出`sql_mode`以后去掉`ONLY_FULL_GROUP_BY`

查看当前连接会话的sql模式：`select @@session.sql_mode;`
查看全局sql_mode设置：`select @@global.sql_mode;`

设置全局sql_mode可以在不重启MySQL的情况下生效
```
set @@global.sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';

set @@SESSION.sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';
```
### 更改配置文件
 linux系统更改/etc/my.cnf文件，使用vi命令打开，如果有sql_mode=...的注释就把注释打开，如果没有就加上sql_mode=...（可以通过select @@sql_mode查出sql_mode以后去掉ONLY_FULL_GROUP_BY后复制过来）

 windows下配置文件是安装目录下的my.ini文件，其余同上

## 相关参数说明
| 参数| 说明| 
|-----|-----|
| ONLY_FULL_GROUP_BY| 对于GROUP BY聚合操作，如果在SELECT中的列，没有在GROUP BY中出现，那么这个SQL是不合法的，因为列不在GROUP BY从句中|
| NO_AUTO_VALUE_ON_ZERO | 该值影响自增长列的插入。默认设置下，插入0或NULL代表生成下一个自增长值。如果用户 希望插入的值为0，而该列又是自增长的，那么这个选项就有用了。|
| STRICT_TRANS_TABLES | 在该模式下，如果一个值不能插入到一个事务表中，则中断当前的操作，对非事务表不做限制。|
| NO_ZERO_IN_DATE| 在严格模式下，不允许日期和月份为零|
| NO_ZERO_DATE | 设置该值，mysql数据库不允许插入零日期，插入零日期会抛出错误而不是警告。|
| ERROR_FOR_DIVISION_BY_ZERO| 在INSERT或UPDATE过程中，如果数据被零除，则产生错误而非警告。如 果未给出该模式，那么数据被零除时MySQL返回NULL|
| NO_AUTO_CREATE_USER | 禁止GRANT创建密码为空的用户|
| NO_ENGINE_SUBSTITUTION| 如果需要的存储引擎被禁用或未编译，那么抛出错误。不设置此值时，用默认的存储引擎替代，并抛出一个异常|
| PIPES_AS_CONCAT | 将\|\|视为字符串的连接操作符而非或运算符，这和Oracle数据库是一样的，也和字符串的拼接函数Concat相类似|
| ANSI_QUOTES | 启用ANSI_QUOTES后，不能用双引号来引用字符串，因为它被解释为识别符|
