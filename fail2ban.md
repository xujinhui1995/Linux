## Fail2Ban
	
When I bought my first cloud server from tecent, I found that there always some people try to brute force my password, although I'm aware it is almost impossible to accomplish that.

But I still want to stop that thing happened again, so I searched on the Internet, and I found a open source software fail2ban.

Although I'm not very clear about the configeration, but I stopped that things heppened again, like thousands of attemptions in a few hours.

1. The installtion is quite simple
	
		[root]# yum -y install epel-release
		[root]# yum -y install fail2ban

2. The main configuration file types:
	1. fail2ban.conf: `Fail2Ban global configuration(such as logging)(全局配置文件)`
	2. filter.d/*.conf: `Filters specifying how to detect authentication failures(过滤器配置文件)`
	3. action.d/*.conf: `Actions defining the commands for banning and unbanning of IP adress(对于两类IP的处理配置文件)`
	4. jail.conf: `Jails defining combinations of Filters with Actions(过滤器与处理方法匹配配置文件)`

3. CONFIGURATION FILES FORMAT
	1. *.conf files should remain unchanged to ease upgrade. If needed, sustomizations should be provided in *.local files.(为了以后软件升级，对于配置的更改放在local文件中)
	2. 