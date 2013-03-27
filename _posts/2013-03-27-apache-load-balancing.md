---
layout: post
title: "apache负载均衡"
tagline: "apche load balancing"
description: "公司的实施项目很多，11年整理的负载均衡配置文档，多次给大家邮件过去，还是有很多人要，这次整理出来贴到网上来。"
tags: [Data Pump]
---
{% include JB/setup %}

<br/>
Apache负载均衡配置说明。
##写在前面
  公司的实施项目很多，11年整理的负载均衡配置文档，多次给大家邮件过去，还是有很多人要，这次整理出来贴到网上来。  
##关键词
  主服务器：master，用来提供服务的后端应用服务器  
  热备份：Hot Standby 用来提供宕机时切换的备份服务器  
  Apache：HttpServer 服务器  
  Apache模块：apache服务器内部使用不同的模块扩展其功能  
##负载均衡原理
  
  1. 将客户端的请求分发给后端部署的各个真实的服务器，达到分流均衡的目的。
  2. 将请求全部分发给主服务器群，在主服务器群全部宕机后立即切换到备份服务器群。  
  
##负载均衡配置

###基本配置
  以httpd-2.2.21-win32-x86-no_ssl为例，可自行去apache.org下载  
  1. 安装Apache，在地址栏输入http://localhost回车后看到It’s Works表示安装成功。最近新版本apache的初始化界面很漂亮哦。
  
  2. 打开conf目录下httpd.conf文件，启用如下三个模块。

	LoadModule proxy_module modules/mod_proxy.so
	LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
	LoadModule proxy_http_module modules/mod_proxy_http.so  

  只要去掉前面的#即可。其中
  
	mod_proxy提供代理服务器功能；
	mod_proxy_balancer提供负载均衡功能；
	mod_proxy_http让代理服务器能支持HTTP协议。
	
  3. 在httpd.conf文件最后添加如下配置：
  
	#以下为负载均衡配置
	ProxyRequests Off  
	<Proxy balancer://x>  
		BalancerMember http://www.baidu.com
		BalancerMember http://www.163.com
	</Proxy>
	ProxyPass /x balancer://x
	
  balancer://x  其中x为配置的负载均衡的唯一名称，可在后面引用；
  BalancerMember  实际服务器的地址，可配置多个，每个都会被指定分发到。
  ProxyPass /x  代理名称，值当前访问路径为/x均指定到负载均衡去处理

  ###负载比例分配配置
  需要给BalancerMember 加上一个参数loadfactor ，取值为1~100，默认为1，值越大分配到的请求数越多。
  
	ProxyRequests Off  
	<Proxy balancer://x>  
		BalancerMember http://www.baidu.com loadfactor=7 
		BalancerMember http://www.163.com loadfactor=1
	</Proxy>
	ProxyPass /x balancer://x  
	
	表示两个服务器已7:1的比例处理请求。

  ###负载算法配置
  Apache负载均衡的算法有三种，系统默认采取安装请求次数均衡（1:1）。
  lbmethod=byrequests 按照请求次数均衡(默认)。 
  lbmethod=bytraffic 按照流量均衡。
  lbmethod=bybusyness 按照繁忙程度均衡(总是分配给活跃请求数最少的服务器)
  
  具体配置如下：
  
	ProxyRequests Off  
	<Proxy balancer://x>  
		BalancerMember http://www.baidu.com loadfactor=7 
		BalancerMember http://www.163.com loadfactor=1
		ProxySet lbmethod=bytraffic
	</Proxy>
	ProxyPass /x balancer://x
	
  ###热备份配置
  
  留出备份服务器以提高系统的整体可靠性，具体见热备份章节。
  
  ###热备份
  需要给BalancerMember 加上一个status属性取值为+H  
  具体配置如下：
  
	ProxyRequests Off  
	<Proxy balancer://x>  
		BalancerMember http://www.baidu.com loadfactor=7 
		BalancerMember http://www.163.com status=+H
		
		ProxySet lbmethod=bytraffic
	</Proxy>
	ProxyPass /x balancer://x

  ###配置多个负载均衡
  
  上面一段可配置多个
  
	#以下为负载均衡配置
	ProxyRequests Off  
	<Proxy balancer://x>  
		BalancerMember http://www.baidu.com
		BalancerMember http://www.163.com
	</Proxy>
	ProxyPass /x balancer://x
	<Proxy balancer://y>  
		BalancerMember http://www.google.com
		BalancerMember http://www.163.com
	</Proxy>
	ProxyPass /y balancer://y
	
	
  示例：
  
	访问http://localhost/x 会被均衡的转发到http://www.baidu.com,和http://www.163.com中。
	访问http://localhost/y会被均衡的转发到http://www.google.com,和http://www.163.com中。




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
