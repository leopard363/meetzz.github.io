---
layout:     post
title:      JAVA常用配置及代码
subtitle:   一些springboot中常用配置及代码片段
date:       2019-07-12
author:     LSC
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - mysql
    - sql函数
---

>SQL函数之查询表注释

#####  此函数主要用于，后台返回给前端实体时，通过此函数查出相关表中字段注释，用以告知前端每个字段的含义，减少沟通成本，所返回结果符合json字符串，前端可根据需求转为对象使用；

```mysql
CREATE DEFINER=`root`@`%` FUNCTION `select_column_info`( param VARCHAR ( 36 ) ) RETURNS varchar(1000) CHARSET utf8mb4
BEGIN
-- 定义变量
DECLARE name VARCHAR ( 512 );
DECLARE comment VARCHAR ( 512 );
DECLARE a INT ( 36 );
DECLARE b INT ( 36 );
DECLARE result VARCHAR ( 512 );

-- 创建游标
DECLARE cur_column CURSOR for
SELECT
	column_name,
  column_comment
FROM
	information_schema.COLUMNS 
WHERE
	table_name = param;
	
-- 	查询表列总数
SELECT
	count(*)
FROM
	information_schema.COLUMNS 
WHERE
	table_name = param into b;
	
set a = 0;
set result = "";

-- 打开游标
open cur_column;

-- 执行循环
myLoop: LOOP
IF a=b THEN
	LEAVE myLoop; 
END IF; 
set a = a+1;
	-- 判断是否结束循环
-- 取游标中的值
FETCH cur_column into name,comment;
-- 执行更新操作
IF a=1 THEN
	set result = CONCAT('"',name,'"',":",'"',comment,'"',",");
ELSEIF a=b THEN
	set result = CONCAT(result,'"',name,'"',":",'"',comment,'"');
ELSE
	set result = CONCAT(result,'"',name,'"',":",'"',comment,'"',",");
END IF;


END LOOP myLoop;
CLOSE cur_column;
RETURN CONCAT("{",result,"}");

END
```
