# MySQL 必知必会

## 第3章 使用MySQL

```mysql
# 启动 mysql服务（archlinux）
sudo systemctl start mysql.service 
# 连接数据库（没设密码
mysql -u yourname -p 
```



```mysql
#选择数据库
USE DatabaseName;

#展示数据库
SHOW DATABASES;

#了解数据库内数据表
SHOW TABLES;

# 描述数据表
SHOW COLUMNS FROM TableName;
DESCRIBE TableName ;

# 了解其他SHOW
HELP SHOW;
```

## 第4章 检索数据

```mysql
# 检索单个列
SELECT 列名 FROM 表名;

# 检索多个列
SELECT 列名1,列名2,列名3 FROM 表名;

# 检索所有列
SELECT * FROM 表名;

# 检索不同的行 DISTINCT （去除重复行）
SELECT DISTINCT 列名 FROM 表名;

# 限制结果,第一行为0,行数不够返回最多的
SELECT 列名 FROM 表名 LIMIT 行数;
SELECT 列名 FROM 表名 LIMIT 开始行,行数;
SELECT 列名 FROM 表名 LIMIT 开始行 OFFSET 行数;

# 使用完全限定的表名
SELECT 表名.列名 FROM 表名;
SELECT 表名.列名 FROM 数据库名.表名;
```

## 第5章 排序检索数据

```mysql
# 排序数据
SELECT 列名 FROM 表名 ORDER BY 排序依据列;

# 按多个列排序
SELECT 列名1,列名2 FROM 表名 ORDER BY 依据列1,依据列2;

# 指定排序方向 DESC只应用到直接位于前面的列名
# 默认A和a排序被认为相同，如果改变就需要专门设置
SELECT 列名1,列名2 FROM 表名 ORDER BY 依据列 DESC;
SELECT 列名1,列名2 FROM 表名 ORDER BY 依据列1 DESC,依据列2;
SELECT 列名1,列名2 FROM 表名 ORDER BY 依据列1 DESC,依据列2 DESC;

# ORDER BY 和 LIMIT 结合 求最值
SELECT 列名1,列名2 FROM 表名 ORDER BY 依据列1 LIMIT 行数;
```

## 第6章 过滤数据

```mysql
# 使用 WHERE 子句 如果和ORDER BY 一起用，要让ORDER BY 在 WHERE 之后 
# 操作符 =,<>,!=,<,<=,>,>=,BETWEEN
# 如果值是字符串需要用单引号来限定
SELECT 列名1,列名2,列名3 FROM 表名 WHERE 列名 条件（操作符+值）;

# 检索单个值
SELECT 列名1,列名2,列名3 FROM 表名 WHERE 列名 = 值;
SELECT 列名1,列名2,列名3 FROM 表名 WHERE 列名 < 值;

# 不匹配检查
SELECT 列名1,列名2,列名3 FROM 表名 WHERE 列名 <> 值;

#范围值检查
SELECT 列名1,列名2,列名3 FROM 表名 WHERE 列名 BETWEEN 值A AND 值B;

#空值NULL检查
#匹配过滤和不匹配过滤不会返回具有NULL值的行
SELECT 列名1,列名2,列名3 FROM 表名 WHERE 列名 IS NULL;
```

## 第7章 数据过滤

```mysql
# 组合 WHERE 子句
# AND 操作符
SELECT 列名1,列名2,列名3 FROM 表名 WHERE 列名1 条件1 AND 列名2 条件2;

# OR 操作符
# AND 优先级比 OR 高，可以使用圆括号
SELECT 列名1,列名2,列名3 FROM 表名 WHERE 列名1 条件1 OR 列名2 条件2;
SELECT 列名1,列名2,列名3 FROM 表名 WHERE (列名1 条件1 OR 列名2 条件2) AND 列名3 条件3;

# IN 操作符 指定条件范围
# IN 和 OR 功能类似，但是 IN 更清晰，更快，可以包含其他 SELECT 语句支持动态 WHERE
SELECT 列名1,列名2,列名3 FROM 表名 WHERE 列名1 IN（值1,值2);

# NOT 操作符 否定之后所跟的任何条件
# MYSQL 支持 NOT 对 IN、BETWEEN、EXISTS 子句取反
SELECT 列名1,列名2,列名3 FROM 表名 WHERE 列名1 NOT IN（值1,值2);
```

## 第8章 用通配符进行过滤

```mysql
# LIKE 搜索模式
# 可以通过配置区分大小写

# %表示任何字符出现任意次数
# 注意尾空格
SELECT 列名1,列名2,列名3 FROM 表名 WHERE 列名1 LIKE '%STATISTICS';
# 以s开头e结尾
SELECT 列名1,列名2,列名3 FROM 表名 WHERE 列名1 LIKE 's%e';

# 下划线表示匹配单个字符;
SELECT 列名1,列名2,列名3 FROM 表名 WHERE 列名1 LIKE '_ ton anvil';
```

## 第9章 用正则表达式进行搜索

```mysql
# 基本字符匹配
SELECT 列名1,列名2,列名3 FROM 表名 WHERE 列名1 REGEXP 'STATISTICS';

#点表示匹配仍以一个字符
#REGEXP 在列值内进行匹配，LIKE 匹配整个列，如果 REGEXP 要匹配整个列则用^ 和$
SELECT 列名1,列名2,列名3 FROM 表名 WHERE 列名1 REGEXP '.0';

# 进行 OR 匹配
SELECT 列名1,列名2,列名3 FROM 表名 WHERE 列名1 REGEXP 'STATISTICS|POOL';
SELECT 列名1,列名2,列名3 FROM 表名 WHERE 列名1 REGEXP '1000|2000|3000';

# 匹配几个字符之一 
# 匹配默认不区分大小写 Ton和ton效果相同
SELECT 列名1,列名2,列名3 FROM 表名 WHERE 列名1 REGEXP '[123] Ton';
# 区分 表示 1 或 2 或 3 Ton
SELECT 列名1,列名2,列名3 FROM 表名 WHERE 列名1 REGEXP '1|2|3 Ton';

# 匹配范围
SELECT 列名1,列名2,列名3 FROM 表名 WHERE 列名1 REGEXP '[1-9]';
SELECT 列名1,列名2,列名3 FROM 表名 WHERE 列名1 REGEXP '[a-z]';

# 匹配特殊字符
SELECT 列名1,列名2,列名3 FROM 表名 WHERE 列名1 REGEXP '\\.';
SELECT 列名1,列名2,列名3 FROM 表名 WHERE 列名1 REGEXP '\\t';

# 匹配多个实例 匹配连续4个数字
# * 0个或多个匹配，+ 1个或多个匹配，？ 0个或1个匹配
#{n}指定数目匹配，{n,}不少于指定数目匹配，{n,m}匹配数目的范围
SELECT 列名1,列名2,列名3 FROM 表名 WHERE 列名1 REGEXP '[[:digit:]]'{4};

# 定位符
# ^文本开始，$文本结尾，[[:<:]]此的开始，[[:>:]]词的结尾
# 找出以一个数（包括小数点开始的数）开始的所有产品
SELECT 列名1,列名2,列名3 FROM 表名 WHERE 列名1 REGEXP '^[0-9\\.]';
```

![image-20220329145216989](https://s2.loli.net/2022/03/29/1UGLq4yufsOkKFQ.png)

## 第10章 创建计算字段

```mysql
# 拼接 MySQL使用Concat()函数，其他的多数使用+或||
SELECT Concat(列名1,'(',列名2,')') FROM 表名;

# RTrim 去除右侧所有空格
SELECT Concat(RTrim(列名1),'(',列名2,')') FROM 表名;

# AS 使用别名（导出列）
SELECT Concat(RTrim(列名1),'(',列名2,')') AS 别名 FROM 表名;

# 执行算术计算 +-*/
SELECT prod_id,quantity,item_price,
	   quantity*item_price AS expanded_price
FROM orderitems
WHERE order_num = 20005;

# 测试计算 省略FROM
SELECT 3*2;
SELECT Trim(' abc ');
```

## 第11章 使用数据处理函数

```mysql
# 函数没有SQL可移植性强

# 文本处理函数 以Soundex()为例
SELECT cust_name,cust_contact
FROM customers
WHERE Soundex(cust_contact) = Soundex('Y Lie');

# 日期和时间处理函数
# 日期必须为yyyy-mm-dd，且使用四位数年份
SELECT cust_id,order_num
FROM orders
WHERE order_date = '2005-09-01';
# 如果要的仅仅是日期那就使用Date()
SELECT cust_id,order_num
FROM orders
WHERE Date(order_date) BETWEEN '2005-09-01' AND '2005-09-30';
```

![image-20220330094406232](https://s2.loli.net/2022/03/30/igMbx9jrU15J3mW.png)

![image-20220330094430762](https://s2.loli.net/2022/03/30/xIJqWDic8mFkl6f.png)

![image-20220330094446131](https://s2.loli.net/2022/03/30/UQSTLyjs2VvKrqB.png)

## 第12章 汇总数据

![image-20220330094523877](../../../../../.config/Typora/typora-user-images/image-20220330094523877.png)

```mysql
# AVG()
# 每个AVG()函数只用于单个列，多个列必须用多个AVG(),AVG()会自动忽略NULL
SELECT AVG(prod_price) AS avg_price FROM products;

# COUNT()
# COUNT(*) 表示对表中行计数，不管是不是NULL
SELECT COUNT(*) AS num_cust FROM custmoers;
# COUNT(column)对特定列具有值的行计数，忽略NULL
SELECT COUNT(cust_email) AS num_cust FROM custmoers;

# MAX()
# MAX() 要求指定列名,忽略 NULL
SELECT MAX(prod_price) AS max_price FROM products;

# MIN()
SELECT MIN(prod_price) AS max_price FROM products;

# SUM()
# 忽略 NULL
SELECT SUM(quantity) AS items_ordered
FROM orderitems
WHERE order_num =20005;

SELECT SUM(quantity*item_price) AS total price
FROM orderitems
WHERE order_num =20005;

# 聚集不同的值，上面几个聚集函数都可以进行以下使用
# 对所有行执行计算指定ALL参数或不给参数
# 值包含不同的值，指定DISTINCT参数
# 如果指定列名，DISTINCT 只能用于 COUNT(),不能用于COUNT(*),因此不允许COUNT（DISTINCT)
# DISTINCT 必须使用列名，不能用于计算或表达式
SELECT AVG(DISTINCT pro_price) AS avg_price
FROM products
WHERE vend_id = 1003;

# 组合聚集函数
SELECT COUNT(*) AS num_items,
		MIN(prod_price) AS price_min,
		MAX(prod_price) AS price_max,
		AVG(prod_price) AS price_avg
FROM products;
```

## 第13章 分组数据

* GROUP BY 子句可以包含任意数目的列
* 如果在GROUP BY子句中嵌套了分组，数据将在最后规定的分组上进行汇总
* GROUP BY子句中列出的每个列都必须是检索列或有效的表达式（但不能是聚集函数）。如果在SELECT中使用表达式，则必须在GROUP BY子句中指定相同的表达式
* 不能使用别名
* 除聚集计算语句外，SELECT语句中的每个列都必须在GROUP BY子句中给出
* 如果分组列中具有NULL值，则NULL将作为一个分组返回。如果列
	中有多行NULL值，它们将分为一组
* GROUP BY子句必须出现在WHERE子句之后，ORDER BY子句之前

```mysql
# 创建分组 GROUP BY
SELECT ven_id,COUNT(*) AS nums_prods
FROM products
GROUP BY ven_id;

# 使用 WITH ROLLUP关键字可以得到每个分组以及每个分组总级别的值
SELECT ven_id,COUNT(*) AS nums_prods
FROM products
GROUP BY ven_id WITH ROLLUP;

# 过滤分组
# WHERE 过的指定的行，而不是分组，WHERE没有分组概念。在数据分组前进行过滤
# 使用 HAVING 子句来过滤分组，HAVING 支持所有 WHERE 所有操作符，在分组后进行过滤
SELECT cust_id,COUNT(*) AS orders
FROM orders
GROUP BY cust_id
HAVING COUNT(*) >= 2


# 分组和排序
# 一般在使用GROUP BY子句时，也应该给出ORDER BY,不要仅依赖GROUP BY 排序数据
SELECT order_num,SUM(quantity*item_price) AS ordertota
FROM orderitems
GROUP BY order_num
HAVING SUM(quantity*item_price) >= 50;

```

![image-20220330103716852](https://s2.loli.net/2022/03/30/DSCz3ol5nbgrtLJ.png)



## 第14章 使用子查询

MySQL4.1 开始支持子查询

```mysql
# 利用子查询进行过滤
# 在WHERE 子句使用子查询应该保证 SELECT语句具有与 WHERE子句中相同数目的列
SELECT cust_id
FROM orders
WHERE order_num IN(SELECT order_num
				   FROM orderitems
				   WHERE prod_id = 'TNT2');
				   
# 作为计算字段使用子查询
# 相关子查询涉及外部查询的子查询，列名可能有多异性，就必须 表名.列名完全限定
SELECT cust_name,
	   cust_state,
	   (SELECT COUNT(*) 
        FROM orders
       	WHERE orders.cust_id = customers.cust_id) AS orders
FROM customers
ORDER BY cust_name;
```



## 第15章 联结表

数据存储在多个表中，如何使用单条 SELECT 语句检索出数据

```mysql
# 创建联结 
# FROM 指定要联结的表 然后 WHERE进行过滤即可（如果不用WHERE得到的是笛卡尔积）
SELECT vend_name,prod_name,prod_price
FROM vendors,products
WHERE vendors.vend_id = products.vend_id #WHERE 等值联结
ORDER BY vend_name,prod_name;

# 内部联结（等值联结）可以使用INNER JOINT 和 ON来达到相同效果 （推荐）
SELECT vend_name,prod_name,prod_price
FROM vendors INNER JOIN products
ON vendors.vend_id = products.vend_id;

# 联结多个表 
# 联结表可能很耗费资源，因此不要联结不必要的表
SELECT prod_name,vend_name,prod_name,prod_price,quantity
FROM orderitems,products,vendors
WHERE vendors.vend_id = products.vend_id
  AND orderitems.prod_id = products.prod_id
  AND order_num = 20005;
```



## 第16章 创建高级联结

```mysql
# 使用表别名
# 为了缩短SQL语句，允许在单条SELECT语句中多次使用相同的表
SELECT cust_name, cust_contact
FROM customers AS c, orders AS o, orderitems AS oi
WHERE c.cust_id = o.cust_id
  AND oi.order_num = o.order_num
  AND prod_id = 'TNT2';
  
  
# 自联结
# 自联结作为外部语句来替代从相同表中检索数据时使用的子查询语句，更快一些
SELECT p1.prod_id, p1.prod_name
FROM products AS p1, products AS p2
WHERE p1.vend_id = p2.vend_id
  AND p2.prod_id = 'DTNTR';
  
  
# 自然联结
# 自然联结排除多次出现，使每个列只返回一次，只能选择那些唯一的列
# 实际上我们的内部联结基本都是自然联结，很少用到不是自然联结的内部联结
# 通过对表使用通配符（SELECT *),对其他表的列使用明确的自己来完成
SELECT c.*, o.order_num, o.order_date.
	   oi.prod_id, oi.quantity, OI.item_price
FROM customers AS c, order AS o, orderitems AS oi
WHERE c.cust_id = o.cust_id
  AND oi.order_num = o.order_num
  AND prod_id = 'FB';
  

# 外部联结
# 将一个表的行与另一个表中的行相关联。有时候需要包含没有关联行的那些行
SELECT customers.cust_id, orders.order_num
FROM customers LEFT OUTER JOIN orders
	ON customers.cust_id = orders.cust_id;
	
SELECT customers.cust_id, orders.order_num
FROM customers RIGHT OUTER JOIN orders
	ON customers.cust_id = orders.cust_id;
	
# 使用带聚集函数的联结
SELECT customers.cust_name,
	   customers.cust_id,
	   COUNT(orders.order_num) AS num_ord
FROM customers INNER JOIN orders
	ON customers.cust_id = orders.cust_id
GROUP BY customers.cust_id;
	
```

## 第17章 组合查询

* 单个查询中从不同的表返回类i是结构的数据
* 对单个表执行多个查询，按单个查询返回数据

```mysql
# 创建组合查询 UNION
SELECT vend_id,prod_id,prod_price
FROM products
WHERE prod_price <= 5
UNION
SELECT vend_id,prod_id,prod_price
FROM products
WHERE vend_id IN(1001,1002)
ORDER BY vend_id,prod_price;
```

UNION 规则

* UNION 必须由两条或两条以上的 SELECT语句组成，语句之间关键词用 UNION 分隔
* UNION 中的每个查询必须包含相同的列、表达式或聚集函数（次序不用相同）
* 列数据类型必须兼容
* UNION默认取消重复的行，UNION ALL可以变成包含重复的行
* 对组合查询结果排序，ORDER BY 必须在最后一个SELECT语句之后，不过是对整个结果排序

## 第18章 全文本搜索

并非所有引擎都支持全文搜索`MyISAM`支持，`InnoDB`不支持

使用搜索机制的几个重要限制

* 性能
* 明确控制
* 智能化的结果

```mysql
# 创建表时启用全文本搜索支持
CREATE TABLE productnotes
(
	note_id		int			NOT NULL AUTO_INCREMENT,
    prod_id		char(10)	NOT NULL,
    note_date	datatime	NOT NULL,
    note_text 	text 		NULL,
    PRIMARY(note_id),
    FULLTEXT(note_text)	#不要在导入数据时使用FULLTEXT，太费时间了
) ENGINE = MyISAM;


# 进行全文搜索
# Match指定被搜索列，Against指定要是有的搜索表达式
# 传递给Match的值必须与FULLTEXT定义中的相同，如果指定多个列，必须按照顺序列出他们
# 全文搜索默认不区分大小写，默认按文本匹配的的良好程度排序
# 如果指定多个搜索项，则包含多数匹配词的行比较少的行优先级高
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('rabbit');


# 使用查询扩展 能找出可能相关的结果，即使它们并不精确包含所查找的词
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('anvils' WITH QUERY EXPANSION);

# 布尔文本搜索
# 即使没有定义 FULLTEXT索引也可以使用它，但很缓慢
# 在布尔方式中，不安等级值降序排序返回行
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('rabbit' IN BOOLEAN MODE);
```



## 第19章 插入数据

```mysql
# 插入完整行
# INSERT 一般不会有输出
INSERT INTO customers(cust_name,
			cust_adress,
        	cust_state,
       	 	cust_zip,
            cust_country,
            cust_contact,
            cust_email)
VALUES('Pep E.LaPew',
      '100 Main Street',
      'Los Angeles',
      'CA',
      '90046',
      'USA',
      NULL,
      NULL);
      
  
# 插入多个行 重复上面的例子即可
# 如果每条 INSERT 语句的列名相同，可以进行如下合并
INSERT INTO customers(cust_name,
			cust_adress,
        	cust_state,
       	 	cust_zip,
            cust_country,
            cust_contact,
            cust_email)
VALUES('Pep E.LaPew',
      '100 Main Street',
      'Los Angeles',
      'CA',
      '90046',
      'USA',
      NULL,
      NULL),
      ('M.Martian',
       '42 Galaxy Way',
       'New York',
       'NY',
       '11213',
       'USA');
       
# 插入检索出的数据
INSERT INTO customers(cust_id,
                      cust_contact,
                      cust_email,
                      cust_name,
                      cust_address,
                      cust_city,
                      cust_state,
                      cust_zip,
                      cust_country)
       SELECT cust_id,
              cust_contact,
              cust_email,
              cust_name,
              cust_address,
              cust_city,
              cust_state,
              cust_zip,
              cust_country
       FROM custnew;
```

## 第20章 更新和删除数据

```mysql
# UPDATE SET
UPDATE customers
SET cust_email = 'elmer@fudd.com',
	cust_name = 'The Fudd'
WHERE cust_id = 10005;

# UPDATE SET NULL 去除列的值
UPDATE customers
SET cust_email = NULL
WHERE cust_id = 10005;

# 删除数据
# DELETE 删除整个行的内容，但不会删除表本身
# 为了删除指定列，请使用UPDATE
#从表中删除所有行可以用 TRUNCATE TABLE
DELETE FROM customers
WHERE cust_id = 10005;
```

更新和删除的指导原则

* 除非确实打算更新和删除每一行，否则绝对不要使用不带WHERE 子句的UPDATE或DELETE语句。 
* 保证每个表都有主键（如果忘记这个内容，请参阅第15章），尽可能 像WHERE子句那样使用它（可以指定各主键、多个值或值的范围）。 
* 在对UPDATE或DELETE语句使用WHERE子句前，应该先用SELECT进 行测试，保证它过滤的是正确的记录，以防编写的WHERE子句不 正确。 
*  使用强制实施引用完整性的数据库（关于这个内容，请参阅第15 章），这样MySQL将不允许删除具有与其他表相关联的数据的行。

## 第21章 创建和操作表

```mysql
# 表创建基础
CREATE TABLE customers
(
    cust_id			int			NOT NULL AUTO_INCREMENT,
    cust_name		char(50)	NOT NULL,
    cust_address	char(50)	NULL,
    cust_city		char(50)	NULL,
    cust_state		char(5)		NULL,
    cust_zip		char(10)	NULL,
    cust_country	char(50)	NULL,
    cust_contact	char(50)	NULL,
    cust_email		char(255)	NULL,
    PRIMARY KEY		(cust_id)
) ENGINE = InnoDB;

# 更新表
ALTER TABLE vendors
ADD vend_phone CHAR(20)

ALTER TABLE vendors
DROP vend_phone 

# 删除表 没有确认也没法撤销
DROP TABLE customer2;

# 重命名表
RENAME TABLE backup_vendors TO vendors,
			 backup_customers TO customers;
```

## 第22章 使用视图

把查询包装成一个虚拟表

为什么使用视图

* 重用 SQL
* 简化 SQL操作
* 使用表的组成部分而不是整个表
* 保护数据
* 更改数据格式和表示

大量使用视图可能造成性能下降

视图的规则和限制

```

```

