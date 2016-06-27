# MySql教学视频-燕十八
## 18-sql的查询模型
Mysql里把列看做变量，Where子句是变量的逻辑判断，如：

	#查询变量shop_price的值大于5000的goods对象的goods_id变量的值
	select goods_id form goods where shop_price > 5000;
	
这样来看，既然是变量，那就可以做各种运算

	select (market_price - shop_price) as discount from goods where (market_price - shop_price) > 200;
	注意这里为啥where子句中又对market_price和shop_price做了一次运算呢，因为where子句是对表本身筛选的，所以对于discount是不起作用的，如果要对查询的子句进行筛选则应使用having子句
	
**Notes：**
<br>	
为啥要在所有查询子句中加入*where 1 = 1*?<br>
在Java开发中，尤其是使用Mybatis等ORM框架开发时，复杂SQL语句经常需要拼接and、or等条件，如果是where子句，那么在每次拼接时就需要对子句进行判断，这样会增加开销，所以在写SQL时统一增加where 1 = 1子句
	

## 19-sql之group分组及统计函数

- max 求最大值

		select max(shop_price) from goods;
- min 求最小值

		select min(shop_price) from goods;
- sum 求总和

		select sum(goods_num) from goods;
- avg 平均值

		select avg(shop_price) from goods;
- count 总行数

		#此时goods_id为空的行不会计算在内
		select count(goods_id) from goods;
		#这样就会统计表内所有行数，包括空值行		
		select count(*) from goods;
		#对于myisam引擎，有计数器维护行数，和count（*）没有区别，然而对于innodb引擎，count(*)会统计行数，开销较大，建议使用下面的语句		
		select count(1) from goods;
		
### group by分组
计算每个分组的商品数量
	
	select count(goods_num) from goods group by cat_id;

group by子句是将表按照字段分组之后，再应用查询子句

	#注意这不是一条符合SQL标准的查询语句，但是mysql中可以执行这条语句，mysql会把第一次出现的goods_id取出来
	select goods_id, sum(goods_num) from goods;
	#配合group by子句则可以取出每个分组对应的值，注意如果select的字段为非聚合函数，则必须在group by子句中，这一限制在PostgreSQL中则没有
	select cat_id, sum(goods_num) from goods group by cat_id;
	
	


## 20-having子句

配合group by分组使用，在分组结果里筛选数据

## 21 having子句续

两个错误实例：

	select name, avg(score) from result where score <60 group by name having count(*) >= 2;
	#where子句会在having筛选之前对数据进行过滤，所以注意这里的having处理的数据其实已经是被过滤了，这里是一个trap！
	
	select name, avg(score), count(score < 60) as gks from result group by name having gks >= 2;
	#count()函数的含义是数据的行数，对count(0)/count(1)都是一样的结果
	
正确的语句：
	
	#1.计算每个人的平均成绩
	select name, avg(score) from result group by name;
	#2.列可以进行投影机算，包括逻辑运算，true-1，false-0
	select name, subject, score, score < 60 as gk from result;
	#3.最终的语句,这是先计算平均成绩
	select name, avg(score), sum(score < 60) as gk from result group by name having gk > 2;
	
	
## 22 order by与limit

order by是对最终结果集的排序,要在筛选条件的最后位置， 默认asc
order by column desc 降序
order by column  asc 升序
	
	#商品按照价格降序排列
	select * from goods order by shop_price desc;
	
	#多个排序用逗号分隔，按照声明顺序优先级降低
	select * from goods order by cat_id desc, shop_price;
	
	
limit 在最终结果集上限制结果返回数量

	select * from goods order by shop_price desc limit 15;
	limit [offset] N, 
	offset为偏移量，从offset的位置开始选择数据
	N为条目数量
	
	#取商品价格最高的那个，but这样会先查出所有的然后只返回一条，效率呢？
	select * from goods order by shop_price desc limit 1;
	#取出价格2-3高的商品
	select * from goods order by shop_price desc limit 1 2;				
	
## 23 查询子句陷阱 **trappppp**
如果group by 子句，那么select的字段只能是group by的字段和聚合函数， 虽然MySQL会自动匹配第一行，这不符合SQL语法，而且MySQL查出的结果也不对。

	#这样也是会取max的第一个结果然后匹配多个行，查出的数据不对
	select max(goods_id), cat_id from goods group by cat_id;

order by子句是对最终数据集的排序，不可能在其他子句之前
where group by having order by limit严格按照顺序


## 24 where型子查询（嵌套查询）
	
where型子查询就是将内查询的结果作为筛选数据的条件
	#查询goods_id最大的商品，不适用排序
	select * from goods where goods_id = (	select max(goods_id) from goods);
	

## 25 from型子查询

将查询结果看作一个临时表，然后再从这个临时表中查询

	select * from (select goods_id, cat_id, goods_name from goods order by cat_id asc, goods desc) group by cat_id;


## 26 exist型子查询
和where型子查询类似，都是用嵌套查询的结果去筛选数据,
exists : 强调的是是否返回结果集，不要求知道返回什么

	#查询有商品的分类
	select cat_id, cat_name from category where exists (select * from goods where goods.cat_id = category.cat_id);
		
		
## 27 MySQL中的NULL

not null default '';
is null, is not null
奇怪的是NULL在数据库中是占用空间的，相比字符串的空值''不占空间

## 28 新手1+N模式查询订单报价

用php写一个产品报价单页面，查询语句很简单

	#了解一下php的用法
	<?php
	
	header('Content-type:text/html; chartset=utf-8');
	
	 $conn = mysql_connect();
	 $sql = 'use test';
	 mysql_query($sql, $conn);
	 
	 $sql = 'set names utf8';
	 mysql_query($sql, $conn);
	 
	 $sql = 'select goods_name, cat_id, goods_number, shop_price from goods';
	 $rs = mysql_query($sql, $conn);
	 
	 $list = array();
	 while($row = mysql_fetch_assoc($rs)){
	 	$list[] = $row;
	 }
	
	>
	
所谓1+N模式，就是先查询一次，然后对查询结果集N的每条数据再查一次，这种查询非常低效，**要使用连接查询来替代这种方法**


## 29 全连接查询

全连接是对数据集合的笛卡尔积操作
MySQL的结果集每一个都有一rowid

表的全相乘操作（这种操作很低效，而且实用意义不大）：
select * from test1, test2;

这种操作从行的角度来看，就是2表的每一行两两组合，从列的角度来看，结果集中的列是两表列的总和

注意这种全相乘操作的结果集中如果有两列列名相同，则必须为其作区分，加上表名或者设置别名

	select * from test1, test2 where test1.id = test2.id;
	

## 30 左连接（left Join）查询

以左表为基准，关联右表，最终结果集的效果是包含左表所有数据，部分符合条件的右表数据

	select * from test1 left join test2 on test1.id = test2.id;
	
连接操作是在内存中对表进行笛卡尔积的操作，产生新的表，在此基础上还可以进行数据筛选

同理右连接只是以右表为基准，其他和左连接没有本质区别

## 31 左右链接的比较

左右连接的一个区别在于关联的基准不同，左连接会保持左表的全部数据，右表没有数据的，NULL填充，右连接则保持右表的全部数据，左表没有对应数据的，NULL填充

左右连接只是在语句的顺序使用上有区别，可以互换，只需同时调换两表的顺序

还有一种外连接，也就是左右连接的结合，但是这种语法MySQL不支持！！！！

## 32 左右连接例题

	select * from m left join h on m.team_id = h.team_id;
	
	select * from m left join h as h1 on m.team_id = h1.team_id left join h on tem.g_id = h.g_id
	
**注意多次连接时重复列名的处理和表的别名设置**

## 33 留言板查询

	#留言数据来源于两张表，feedback 留言表，comment 评论表
	select * from feedback;
	select * from comment;
	
这样两次查询效率很低，可以使用union查询

## 34 union查询

union 用于合并两个或多个 select 语句的结果集。
请注意，union 内部的 select 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。同时，每条 select 语句中的列的顺序必须相同。

	select goods_id, goods_name, shop_price from goods where shop_price < 30
	union
	select goods_id, goods_name, shop_price from goods where shop_price > 4000;
	
	
默认地，union 操作符选取不同的值。如果允许重复的值，请使用 union all。

union操作后的结果集仍然可以进行排序等操作

## 35 union例题

	#留言板问题
	select user_name, msg_content, msg_time from feedback
	union
	select user_name, msg as msg_content, time as msg_time from comment;
	

**要注意union自动去重，如果不需要去重应该使用union all**


## 36 函数
MySQL自带很多函数

- 数学函数
	- abs() 绝对值函数
	- bin() 二进制函数，类似的oct()、hex()函数
	- ceiling() 向上取整
	- exp() e的幂
	- floor（）向下取整
- 聚合函数
	- avg()
	- count()
	- min()
	- max()
	- sum()
	- group_concat()
- 字符串函数
	- ascii() 
	- bit_length()
	- concat(s1, s2)
	- find_in_set()
	- lcase(str)
- 时间函数
	- DATE_ADD(date, INTERVAL expr type)/ADDDATE(date, INTERVAL expr type)
	- DATE_SUB(date, INTERVAL expr type)/SUBDATE(date, INTERVAL expr type)
	...
	
**多查手册，了解其它函数的用法，利用函数可以提高工作效率**

## 37 时间日期函数和流程控制函数

### 时间函数

- now()函数，返回格式为datetime
- curdate()函数，返回格式为date


	create table jiaban(
	num int,
	dt date
	) engine myisam charset utf8；
	
- week() 计算日期的星期
-

附：如何破解数据库密码, **这个bug还没修复？**：

1.通过任务管理器或者服务管理关掉mysqld（msyql的守护进程）
2.通过命令行+特殊参数开启mysqld

	mysqld --skip -grant -tables
3.此时数据库进程已经开启，而且不需要权限检查
4.mysql -uroot 无密码登录服务器
5.修改权限表：
	
	- use msysql;
	- update user set Password=password('111111') where user = 'root';
	- flush priviledges;
	

### 流程控制函数

**CASE WHEN函数**

**CASE var
**WHEN expr
THEN expr
[WHEN expr
THEN expr
ELSE expr
]
**END**

	select sname, case gender when 1 then 'male' when 0 then 'female' else 'none' end as gender from test;
	
**三元判断函数**
if(expr1, expr2, expr3)
	
	select sname, if(gender=0, 'first', 'second') as priviledge from test;
	
**null判断控制**
ifnull（expr1, expr2）
if expr is null then execute expr2
	
	select ifnull(1/0, 'yes');
	
	
## 38 函数注意事项

- 函数的使用必然会降低查询语句的效率，在建表时就应该考虑合理的结构来降低函数使用
- 一些业务相关的数据处理尽量在服务层处理，不要依赖数据库函数
- 对某列使用函数，则查询时该列的索引就不会被使用

## 39 视图的概念

视图（view）
在查询中，查询结果集可以看成是一张表，而view就是一张虚拟的表，将查询结果集作为一张虚拟表存储起来，下次查询的时候可以从view中查询,view直接使用查询语句创建。

**注意，创建view并不会真正创建一张表，只是在表上建立一种映射关系，所以并不会提高查询效率！**

	#创建view
	create view  viewName as (select ...)
	#删除view
	drop view viewName
	
视图的用处：

- 简化查询，存储视图可以简化我们查询语句
- 更精细的权限控制，通过开放视图权限来控制对原表的访问
- 数据多分表时，查询更方便


视图，是表的一个投影，表的数据修改时，视图随之被改变；同样的，视图的修改也会被映射到表上，但是这种逆向操作是有限制的（不能执行CRUD，只有视图和表有一一对应的关系时才可以进行修改操作）。

## 40 视图和表的关系、视图algorithm的概念

视图和表的一一对应是指：

根据select从表中取出的 行，只能计算出视图中确定的一行，反之视图中任意抽取一行，能够反推出表中确定的一行

因而含有order by limit等子句的view，即便是和表的列有对应关系，也不能做修改，因为order by会有不确定性

algorithm = merge/temptable/undefined

- Merge: 当引用视图时，引用视图的语句与定义视图的语句合并
- Temptable：当引用视图时，根据视图创建语句简历临时表
- Undefined：未定义，系统自动选

		
		#建立view，并通过algorithm设置为temptable声明需建立临时表
		create algorithm = temptable view v3
		as 
		select goods_id, goods_name form goods order by cat_id desc;
	


## 41 GB2312与UTF8编码

不同编码格式导致的乱码问题是在开发过程中遇到的一个常见问题！

**所谓字符集，就是二进制到字符的映射关系。**最基本的字符集如ASCII码

GB2312是一个中文字符集，收录了6763和常用汉字，更完善的库是GBK！！！

unicode 编码用4字节编码

utf-8，通过标志位来标识一个字符的长度

## 42 MySQL字符集参数

MySQL在执行语句过程中如果客户端编码格式和MySQL服务器的编码格式不一致，就会有乱码的情况产生。

在客户端和服务器端之间有一个连接器，作为两者之间的桥梁。连接器回对客户端的编码进行转化（按照预设），然后连接器把请求转发给服务器

	#设置服务器的编码格式，这条语句会将character_set_client/character_set_connection/character_set_result三个值都设置为utf8
	set names utf8;
	
	#在服务器上设置处理客户端请求的编码格式
	set character_set_client = utf8;
	
	#设置连接器编码格式
	set character_set_connection = gbk;
	
	#设置服务端返回查询结果编码格式
	set character_set_result = gbk;
	

**造成乱码的原因除了编码方式一样外，各个字符集的编码数量不同，在转换时会丢失**

## 43 uft8的BOM问题

在UCS 编码中有一个叫做"ZERO WIDTH NO-BREAK SPACE"的字符，它的编码是FEFF。而FFFE在UCS中是不存在的字符，所以不应该出现在实际传输中。UCS规范建议我们在传输字节流前，先传输 字符"ZERO WIDTH NO-BREAK SPACE"。这样如果接收者收到FEFF，就表明这个字节流是Big-Endian的；如果收到FFFE，就表明这个字节流是Little- Endian的。因此字符"ZERO WIDTH NO-BREAK SPACE"又被称作BOM。
 
UTF-8不需要BOM来表明字节顺序，但可以用BOM来表明编码方式。字符"ZERO WIDTH NO-BREAK SPACE"的UTF-8编码是EF BB BF。所以如果接收者收到以EF BB BF开头的字节流，就知道这是UTF-8编码了。
 
UTF- 8编码的文件中，BOM占三个字节。如果用记事本把一个文本文件另存为UTF-8编码方式的话，用UE打开这个文件，切换到十六进制编辑状态就可以看到开 头的FFFE了。这是个标识UTF-8编码文件的好办法，软件通过BOM来识别这个文件是否是UTF-8编码，很多软件还要求读入的文件必须带BOM。可 是，还是有很多软件不能识别BOM。
 
在Firefox早期的版本里，扩展是不能有BOM的，不过Firefox 1.5以后的版本已经开始支持BOM了。现在又发现，PHP也不支持BOM。PHP在设计时就没有考虑BOM的问题，也就是说他不会忽略UTF-8编码的文件开头BOM的那三个字符。

**utf-8本来就不应该加bom，除了 让编辑器知道它是个utf-8之外什么用处都没有**。实际上编辑器完全有能力在不太多的几个编码格式之间根据特征来判断一个文件是什么编码，就算不能自动识 别，编辑器也应该有设置编码的地方。所以我觉得BOM对于utf-8来说是多余的东西。
 
** utf-16才需要加bom。因为它是按unicode顺序编码，在BMP范围内是二字节，需要识别是大或小字节序。 **

## 44 中文截取utf-8无乱码
utf-8的编码需要标志位来表明字符的长度
如下：

	0XXX XXXX 						1字节
	110X XXXX 10XX XXXX			    2字节
	1110 XXXX 10XX XXXX 10XX XXXX 	3字节
	1111 0XXX 10XX XXXX 10XX XXXX 10XX XXXX 4字节
	...
	

根据编码格式可以判断utf-8编码的每个字符的长度，以此截取

这部分内容是PHP的字符截取函数的问题，Java中使用Unicode编码，截取字符串时不存在这种问题



## 45  存储引擎与事务简单介绍

#### MySQL的引擎

不同的engine不会改变存储的信息，但是存储的方式会不一样！

**Mysiam/InnoDB**/BDB/Memory/Archive

Mysiam:

	写入速度快，表级锁，不支持事务安全
	ISAM是一个定义明确且历经时间考验的数据表格管理方法，它在设计之时就考虑到数据库被查询的次数要远大于更新的次数。因此，ISAM执行读取操作的速度很快，而且不占用大量的内存和存储资源。ISAM的两个主要不足之处在于，它不支持事务处理，也不能够容错：如果你的硬盘崩溃了，那么数据文件就无法恢复了。如果你正在把ISAM用在关键任务应用程序里，那就必须经常备份你所有的实时数据，通过其复制特性，MySQL能够支持这样的备份应用程序。 
	
	
InnoDB：

	写入速度相对慢，行级锁，支持事务安全
	提供了事务控制能力功能，它确保一组命令全部执行成功，或者当任何一个命令出现错误时所有命令的结果都被回退，可以想像在电子银行中事务控制能力是非常重要的。支持COMMIT、ROLLBACK和其他事务特性。最新版本的Mysql已经计划移除对BDB的支持，转而全力发展InnoDB。 

出于数据安全的考虑，一般在开发中使用InnoDB引擎，default也是InnoDB
	
	#实际开发中一般使用InnoDB
	create table t1 (
		name varchar(12),
		age int,
		addr varchar(50)
	) engine innoDB charset utf-8;
	
	#修改engine
	alter table t1 engine = myisam;
	
	
以下是一些细节和具体实现的差别：

- 1.InnoDB不支持FULLTEXT类型的索引。
- 2.InnoDB中不保存表的 具体行数，也就是说，执行select count(\*) from table时，InnoDB要扫描一遍整个表来计算有多少行，但是MyISAM只要简单的读出保存好的行数即可。注意的是，当count(*)语句包含 where条件时，两种表的操作是一样的。
- 3.对于AUTO_INCREMENT类型的字段，InnoDB中必须包含只有该字段的索引，但是在MyISAM表中，可以和其他字段一起建立联合索引。
- 4.DELETE FROM table时，InnoDB不会重新建立表，而是一行一行的删除。
- 5.LOAD TABLE FROM MASTER操作对InnoDB是不起作用的，解决方法是首先把InnoDB表改成MyISAM表，导入数据后再改成InnoDB表，但是对于使用的额外的InnoDB特性（例如外键）的表不适用。

另外，InnoDB表的行锁也不是绝对的，如果在执行一个SQL语句时MySQL不能确定要扫描的范围，InnoDB表同样会锁全表，例如update table set num=1 where name like “%a%”
	



### 事务
ACID

原子性（atomic）、一致性（consistent）、隔离性（isolation）、持久性（duration）

InnoDB引擎支持事务操作
innodb_flush_log_at_trx_commit选项决定什么时候吧事务保存到日志

	#开始一个事务
	start transaction

	#保存点
	save point pointName;
	
	#sql
	update table t1 set t1.a = 'test' where t1.id = 4;
	
	#提交事务
	commit
	
	#回滚事务
	rollback
	
	
	

	








	