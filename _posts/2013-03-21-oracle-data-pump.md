---
layout: post
title: "oracle数据的导入导出"
tagline: "Oracle Data Pump"
description: "帮测试组导数据，顺便整理下Oracle导入导出的命令以备后用"
tags: [Data Pump]
---
{% include JB/setup %}

<br/>
帮测试组导数据，顺便整理下Oracle导入导出的命令以备后用。
###简单的导入和导出命令  

	imp user/pass@dbserverName fromuser=xxx touser=xxx file=D:\xxx.dmp  
	exp user/pass@dbserverName file=D:\xxx.dmp  

效率极其地下，以前使用9i，4GB的数据要导很长时间。

###数据泵 Data Dump
  *Oracle 10g*，包含10g 出现了数据泵的概念，导入导出变的神速。
  
  *完整的命令如下*
	expdp user/pass@ip:port/sid directory=directory name dumpfile=dmp file name  
	impdp user/pass@ip:port/sid directory=directory name dumpfile=dmp file name remap_schema=old username:target username remap_tablespace=old tablespace name:target tablespace name  
  
  *directory* ： 指定导出的路径
	
	create or replace director dump_dir as '/tmp';

可通过 `SELECT * FROM dba_directories;` 查询路径，使用expdp和impdp命令时，直接引用dump_dir即可，Oracle会自动将dmp文件转存到次目录。

  * remap_schema *  
   如果源用户和目标用户不一致的话需要指定次参数 如`rempa_schema=userA:userB`
  * remap_tablespace *
   看字面意思也不难理解，表示源库和目的库表空间不一致，可通过次参数限定。 如`remap_tablespace=tablespaceA:tablespaceB`
 
  OK, 试试吧，不是一般的快啊。  
  
  问题来了，低版本导入到高版本会报错，加上version参数指定目的地库的版本号即可。  
  千万记得，加入版本号是针对导出时使用`expdp version=10.2.0.1.0`，版本号是目的库的版本号哦  
