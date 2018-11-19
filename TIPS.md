putty 3 mins 客户端不发送数据自动断开问题：
	
	1.在登陆时点击connection，将Seconds between keepalives 数值置为60，表示60s发送一次空包维持连接。

安装GNOME桌面
	
	# yum groupinstall "GNOME Desktop" "Graphical Administration Tools" 
进入桌面
	
	# startx

设置开机启动

	# ln -sf /lib/systemd/system/runlevel5.target /etc/systemd/system/default.target