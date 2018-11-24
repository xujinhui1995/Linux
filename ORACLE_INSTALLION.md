### ORACLE 12c INSTALLION

1. Update the system to the letest versions.

		# yum update -y

2. Install all the required dependencied for the RDBMS, along with the zip and unzip packages.

		# yum install - binutils.x86_64 compat-libcap1.x86_64 gcc.x86_64 gcc-c++.x86_64 glibc.i686 glibc.x86_64 glibc-devel.i686 glibc-devel.x86_64 ksh compat-libstdc++-33 libaio.i686 libaio.x86_64 libaio-devel.i686 libgcc.x86_64 libstgcc++.i686 libstc++.x86_64 libstdc++-devel.i686 libstgcc++-devel.x86_64 libXi.i686 libXi.x86_64 libXtst.i686 libXtst.x86_64 make.x86_64 sysstat.x86_64 zip unzip

3. Create the user account and groups for oracle

		# groupadd oinstall
	 	# groupadd dba
	    # useradd -g oinstall -G dba oracle

	set a password for the newly created oracle account

		# passwd oracle

4. Add the following kernel parameters to /etc/sysctl.conf file

		fs.aio-max-nr = 1048576 # 文件系统最大一部IO
		fs.file-max = 6815744 # 表示进程可能同时打开的最大句柄数
		kernel.shmall = 2097152 #可以使用的共享内存的总量
		kernel.shmmax = 8329226240 #最大共享内存段大小
		kernel.shmmni = 4096 #整个系统共享内存段的最大数目
		kernel.sem = 250 32000 100 128 #每个信号对象集的最大信号对象数；系统范围内最大信号对象数；每个信号对象支持的最大操作数；系统范围内最大信号对象集数
		net.ipv4.ip_local_port_range = 9000 65500 #UDP和TCP连接中本地端口（不包括连接的远端）的取值范围
		net.core.rmem_default = 262144 #内核套接字接收缓存区的默认值
		net.core.rmem_max = 4194304 #内核套接字接收缓存区的最大值
		net.core.wmem_default = 262144 #内核套接字发送缓存区的默认值
		net.core.wmem_max = 1048586 #内核套接字发送缓存区的最大值

	apply them
		
		# sysctl -p 
		# sysctl -a

5. Set the limits for oracle in /etc/security/limits.conf file

		oracle soft nproc 2047
		oracle hard nproc 16384
		oracle soft nofile 1024
		oracle hard nofile 65536

6. Create a directory named /stage and extract the zipped installation file

	Before that you need to get the zip file
		
		# wget http://download.oracle.com/otn/linux/oracle12c/122010/linuxx64_12201_database.zip

		# unzip linuxx64_12201_database.zip -d /stage/

	Before proceeding, create other directories that will be used during the actual installtion, and assign the necessary permissions.

		# mkdir /u01
		# mkdir /u02
		# chown -R oracle:oinstall /u01
		# chown -R oracle:oinstall /u02
		# chmod -R 775 /u01
		# chmod -R 775 /u02
		# chmod g+s /u01
		# chmod g+s /u02

7. Open a GUI session in the RHEL/CentOS 7 server and launch the installation script

		# su oracle
		# /stage/database/runInstaller

    and follow the steps presented by the installer.
	
	在使用SVN远程连接的情况下，会提示`display must be configured to display at least 256 colors`，这是因为默认VNC连接只能为root用户，而Oracle安装时用的时oracle用户，因此需要解除这个限制
		
		# su
		# export DISPLAY=localhost:1
		# host +

	这里看到的说法就是，本来是只有root用户能够通过VNC服务远程进行桌面连接，其他的用户没有此权限，所以在root的用户的基础上，将root作为显示服务器，`host +`则是将其他用户作为客户端允许其他用户通过root用户进行连接。

	这个只在虚拟机里可以成功，没有在服务器实验

	开始安装之后的配置界面，出现了几个问题
	
		swap 空间不足
		maximum size of stack limits

	Limit Minmum Stack size 问题：使用其fix无反应，但目前看来没有影响。
	


