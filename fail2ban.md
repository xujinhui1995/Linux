## Fail2Ban
	
When I bought my first cloud server from tecent, I found that there always some people try to brute force my password, although I'm aware it is almost impossible to accomplish that.

But I still want to stop that thing happened again, so I searched on the Internet, and I found a open source software fail2ban.

Although I'm not very clear about the configeration, but I stopped that things heppened again, like thousands of attemptions in a few hours.

1. The installtion is quite simple
	
		[root]# yum -y install epel-release
		[root]# yum -y install fail2ban

2. The struction of fail2ban
	
	There are 13 files under the main foloder including the file I created.
	
		
