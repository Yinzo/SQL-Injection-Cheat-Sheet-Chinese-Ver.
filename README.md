# SQL-Injection-Cheat-Sheet-Chinese-Ver.-
Translate SQL Injection Cheat Sheet(http://ferruh.mavituna.com/sql-injection-cheatsheet-oku/) into Chinese.

Source:	http://ferruh.mavituna.com/sql-injection-cheatsheet-oku/

Translated by:	Yinzo

Date(Y/M/D):	2015/08/09

## Progress:

Date | Progress
--- | ---
08/09 | 30%
08/10 | 70%
08/11 | 99%

**(Finish) Date: 	2015/08/11**

---

# HEADER & CATALOGUE


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
    1. 盲注
        1. 关于盲注
        1. 实战中的盲注实例
    1. 延时盲注
        1. `WAITFOR DELAY [time]`(S)
        1. 实例
        1. `BENCHMARK()`(M)
        1. 实例
        1. `pg_sleep(seconds)`(P)
    1. 掩盖痕迹
        1. `-sp_password log bypass`(S)
    1. 注入测试
    1. 一些其他的MySQL笔记
        1. MySQL中好用的函数
    1. SQL注入的高级使用
        1. 强制SQL Server来得到NTLM哈希
        1. Bulk insert UNC共享文件 (S) 
1. 待办事项 / 联系方式 / 帮助
