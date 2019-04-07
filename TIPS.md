1. putty 3 mins 客户端不发送数据自动断开问题：
	
	1.在登陆时点击connection，将Seconds between keepalives 数值置为60，表示60s发送一次空包维持连接。

2. 安装GNOME桌面
	
		# yum groupinstall "GNOME Desktop" "Graphical Administration Tools" 

	进入桌面
	
		# startx

	此时输入进入桌面命令后会提示缺少文件，需要重启后，进入配置。

	在远程登录时，首先重启，重启之后SSH连接，安装VNC-SERVER，配置用户，开启相应端口，连接后自动出现桌面。vncservermore不是开机自启的。重启后需要使用命令`vncserver`启动服务，否则会提示：拒绝连接。

	设置开机启动

		# ln -sf /lib/systemd/system/runlevel5.target /etc/systemd/system/default.target

3. VNC连接需要手动开启端口

		firewall-cmd --zone=public --add-port=5901/tcp --permanent #开启端口
		firewall-cmd --reload #重新载入
		firewall-cmd --zone=public --query-port=5901 #查看
		firewall-cmd --zone=public --remove-port=5901/tcp --permanent #关闭端口


4. 为普通用户增加sudo权限

		# vim /etc/sudoers

	add `username ALL=(ALL)ALL` after `root ALL=(ALL)ALL`

		root ALL=(ALL)ALL
		username ALL=(ALL)ALL
		# :wq!

5. VNC连接

	启用窗口
	
		vncserver :1

	关闭窗口

		vncserver -kill :1

	TLS加密

		服务端：
		vncserver -SecurityTypes=VenCrypt,TLSVnc

		客户端：
		vncviewer -SecurityTypes=VeNCrypt,TLSVnc 192.168.130.12:1

6. swap空间不足最容易解决
	
	`free`命令查看swap空间大小
	
		# free

	通过dd命令创建一个临时的swap file，大小为1Gb
	
		# dd if=/dev/zore of=/home/oracle/swap.file bs=1024k count=1024

	通过mkswap命令格式化上步中的临时交换文件

		# mkswawp /home/oracle/swap.file

	通过swapon命令使swap文件生效

		# swapon /home/oracle/swap.file

	如果要永久有效，需要将新的swap

7. 修改`.bash_profile`后重新载入
	
		source .bash_profile

8. 时区与时间
	
	查看时间
		
		# date

	同步时间

		# ntpdate

	选择时区
	
		# tzselect

	直接更换时区

		# cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

9. 宿主机与虚拟机不通
    
	    因为宿主机存在防火墙，所以虚拟机ping不通宿主机很正常
		虚拟机的基本网络信息配置完成后，ping不通的一个可能的原因就是宿主机和虚拟机不在同一个网段，所以需要更改宿主机虚拟网卡网络适配器选项，使得主宿网络处于同一网段。

10. zsh 安装
       
        # yum -y install zsh
        # sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"