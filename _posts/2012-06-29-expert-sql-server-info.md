---
layout: post
title: 查看并导出SQL Server数据表中字段的注释信息
category: notes
---

以下是**SQL Server 2000**的：

<pre>SELECT sysobjects.name AS 表名, syscolumns.name AS 列名,
systypes.name AS 数据类型, syscolumns.length AS 数据长度, CONVERT(char,
sysproperties.[value]) AS 注释
FROM sysproperties RIGHT OUTER JOIN
sysobjects INNER JOIN
syscolumns ON sysobjects.id = syscolumns.id INNER JOIN
systypes ON syscolumns.xtype = systypes.xtype ON
sysproperties.id = syscolumns.id AND
sysproperties.smallid = syscolumns.colid
WHERE (sysobjects.xtype = 'u' OR
sysobjects.xtype = 'v') AND (systypes.name &lt;&gt; 'sysname')
--and CONVERT(char,sysproperties.[value]) &lt;&gt; 'null' --导出注释不为'null'的记录
--AND (sysobjects.name = 'Xms_Message' ) --逐个关联表名，可以用or连接条件
ORDER BY 表名</pre>

以下是**SQL Server 2005**的：

<pre>/*
*1.获取所有数据库名:
*Select Name FROM Master..SysDatabases ORDER BY Name
*2.获取所有表名:
*Select Name FROM DatabaseName..SysObjects Where XType='U' ORDER BY Name
*XType='U':表示所有用户表;
*XType='S':表示所有系统表;
*3.获取所有字段名:
*Select Name FROM SysColumns Where id=Object_Id('TableName')
*/

SELECT
表名 = CASE WHEN A.COLORDER=1 THEN D.NAME ELSE ' ' END,
表说明 = CASE WHEN A.COLORDER=1 THEN ISNULL(F.VALUE,' ') ELSE ' ' END,
字段序号 = A.COLORDER,
字段名 = A.NAME,
标识 = CASE WHEN COLUMNPROPERTY( A.ID,A.NAME,'ISIDENTITY ')=1 THEN '√'ELSE ' ' END,
主键 = CASE WHEN EXISTS(Select 1 FROM SYSOBJECTS Where XTYPE='PK ' AND PARENT_OBJ=A.ID AND NAME IN (
SELECT NAME FROM SYSINDEXES Where INDID IN(
SELECT INDID FROM SYSINDEXKEYS Where ID = A.ID AND COLID=A.COLID))) THEN '√' ELSE ' ' END,
类型 = B.NAME,
字段长度 = A.LENGTH,
精度 = COLUMNPROPERTY(A.ID,A.NAME,'PRECISION '),
小数位数 = ISNULL(COLUMNPROPERTY(A.ID,A.NAME,'SCALE '),0),
允许空 = CASE WHEN A.ISNULLABLE=1 THEN '√'ELSE ' ' END,
缺省值 = ISNULL(E.TEXT,' '),
字段说明 = ISNULL(G.[VALUE],' ')
FROM
SYSCOLUMNS A
LEFT JOIN SYSTYPES B
ON A.XUSERTYPE=B.XUSERTYPE
INNER JOIN SYSOBJECTS D
ON A.ID=D.ID AND D.XTYPE='U ' AND D.NAME&lt;&gt;'DTPROPERTIES '
LEFT JOIN SYSCOMMENTS E
ON A.CDEFAULT=E.ID
LEFT JOIN sys.extended_properties G
ON A.ID=G.major_id AND A.COLID=G.minor_id
LEFT JOIN sys.extended_properties F
ON D.ID=F.major_id AND F.minor_id=0
</pre>
