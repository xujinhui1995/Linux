### ORACLE 12c INSTALLION

1. Update the system to the letest versions.

		# yum update -y

2. Install all the required dependencied for the RDBMS, along with the zip and unzip packages.

		# yum install - binutils.x86_64 compat-libcap1.x86_64 gcc.x86_64 gcc-c++.x86_64 glibc.i686 glibc.x86_64 glibc-devel.i686 glibc-devel.x86_64 ksh compat-libstdc++-33 libaio.i686 libaio.x86_64 libaio-devel.i686 libgcc.x86_64 libstgcc++.i686 libstdc++.x86_64 libstdc++-devel.i686 libstdgcc++-devel.x86_64 libXi.i686 libXi.x86_64 libXtst.i686 libXtst.x86_64 make.x86_64 sysstat.x86_64 zip unzip

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
	

8. Usually, I'll skip this step.

	![](https://i.imgur.com/muBi0km.png)

9. Choose Create and configure a database.

	![](https://i.imgur.com/Ko44J9g.png)

10. Select Desktop class since we are setting up a minimal configuration and a starter database.

	![](https://i.imgur.com/pu0B9T6.png)

11. Select the following options for basic configuration.

	- Oracle base: /u01/app/oracle
	- Software location: /u01/app/oracle/product/12.2.0/dbname_1
	- Database file location: /u01
	- OSDBA group: dba
	- Global database name: your choice. I followed tecmint on the virture marchine
	- Take note of the password, as you will be using it when you first connect to the database.
	- Unckeck Create as Container database.

	![](https://i.imgur.com/piPIUY5.png)

12. Leave the default Inventory Diractory as /u01/app/oraventory

	![](https://i.imgur.com/IYQXd8l.png)

13. verify that the installation pre-checks are completed without errors.

	![](https://i.imgur.com/a8P8SVs.png)

	The solution of errors also see the TIPS.

14. wait until the Oracle 12c installation completed.

	![](https://i.imgur.com/hUy9fo3.png)

	It is possible that at some point during the installation you will be asked to run a couple of scripts to set up further permissions or correct issues. That is illustrated here:

	![](https://i.imgur.com/6QDPbWo.png)

    And here:
		
		# cd /u01/app/oraInventory
		# ./orainstRoot.sh
		# cd /u01/app/oracle/product/12.2.0/dbhome_1
		# ./root.sh

	![](https://i.imgur.com/0096sf1.png)

15. After that, you will need to return to the previous screen in the GUI session and client OK so that the installation can continue.

	When it is finished, you will be presented with the following message indicating the URL of the Oracle Enterprise Maneger:

		https://localhost:5500/em

16. To allow connections from outside the server, you will need to open the following ports:
	
		1521/TCP
		5500/TCP
		5520/TCP
		3938/TCP

	As following:
		
		# firewall-cmd --zone=public --add-port=1521/tcp --add-port=1521/tcp --add-port=5500/tcp --add-port=3938/tcp --permanent
		# firewall-cmd --reload

17. Next, login as oracle using the password that was chosen previously and add the following lines to .bash_profile.

		TMPDIR=$TMP; export TMPDIR
		ORACLE_BASE=/u01/app/oracle; export ORACLE_BASE
		ORACLE_HOME=$ORACLE_BASE/product/12.2.0dbhome_1; export _HOME
		ORACLE_SID=tecmint; export ORACLE_SID
		PATH=$ORACLE_HOME/bin:$PATH; export PATH
		LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib:/usr/lib64; export LD_LIBRARY_PATH
		CLASSPATH=$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib; export CLASSPATH

	Actually, after this change, we should also reload the .bash_profile file too.

		# source .bash_profile

18. Finally, replace locahost with 0.0.0.0 on.

		# vim $ORACLE_HOME/netweork/admin/listener.ora

	
19. The reason we reload the .bash_profile is the 18 step, need the new configuration.

20. And then login to the database using the system account and the password chosen in step 11 of the previous section.

		# sqlplus system@tecmint

	Optionally, let's create a table inside the tecmint database where we insert some sample records as follows.

		SQL> CREATE TABLE NamesTBL
		(id NUMBER GENERATED AS IDENTITY,
		name VARCHAR2(20));

	Please note that IDENTITY columns werefirst introduced in Oracle 12c.

		SQL> INSERT INTO NamesTBL (name) VALUES ('Gabriel');
		SQL> INSERT INTO NamesTBL (name) VALUES ('Admin');
		SQL> SELECT * FROM NamesTBL;

	

21. To enable the database service to start automatically on boot, add the following lines to /etc/systemd/system/oracle-rdbms.service file.

		# /etc/systemd/system/oracle-rdbms.service
		# Invoking Oracle scripts to start/shutdown Instance defined in /etc/oratab
		# and starts Listener
	
		[Unit]
		Description=Oracle Database(s) and Listener
		Requires=netweork.target

		[Service]
		Type=forking
		Restart=no
		ExecStart=/u01/app/oracleproduct/12.2.0/dbhome_1/bin/dbstart /u01/app/oracle/product/12.2.0/dbhome_1
		ExecStop=/u01/app/oracle/product/12.2.0/dbhome_1/bin/dbshut /u01/app/oracle/product/12.2.0/dbhome_1
		User=oracle
	
		[Install]
		WantedBy=multi-user.target

22.Finally, we need to indicate that the tecmint database should be brought up during boot in /etc/oratab(Y:Yes)

	