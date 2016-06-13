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
	
	
				
		
		
	