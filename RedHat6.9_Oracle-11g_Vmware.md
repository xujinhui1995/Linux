## Oracle RAC 虚拟机安装 ##

### 一、安装环境 ###

- RAC节点操作系统： Redhat 6.9 x86_64
- Cluster software: Oracle Grid Infrastructure 11g r2(11.2.0.4)
- Oracle Dtabase software: Oracle 11g r2 (11.2.0.4)
- 共享存储: ASM

### 二、网络规划 ###

<table>
    <tr>
        <td><b>节点名称</b></td>
        <td><b>Public IP</b></td>
        <td><b>Private IP</b></td>
        <td><b>Virtual IP</b></td>
        <td><b>SCAN 名称</b></td>
        <td><b>SCAN IP</b></td>
    </tr>
    <tr>
        <td>rac1</td>
        <td>192.168.75.50</td>
        <td>192.168.0.2</td>
        <td>192.168.75.51</td>        
        <td rowspan="2">rac-scan</td>
        <td rowspan="2">192.168.75.52</td>
    </tr>
    <tr>
        <td>rac2</td>
        <td>192.168.75.53</td>
        <td>192.168.0.3</td>
        <td>192.168.75.54</td>        
    </tr>
</table>

说明：公有IP（公网）一般用于管理员，用来确保可以操作到正确的机器，可以理解为真实IP；专用IP（私网）用于心跳同步，这个用于用户层面，可以直接忽略，简单李节，这个IP用来保证两台服务器同步数据；虚拟IP用于客户端应用，以支持失效转意，通俗说就是一台挂了，另一台自动接管，客户端没有任何感觉；在11gR2中，SCAN IP是作为一个新增IP出现的，源由的CRS中的VIP仍然存在，SCAN主要是简化客户端连接。

### Oracle软件组 ###

<table>
    <tr>
        <td><b>软件组件</b></td>
        <td><b>用户</b></td>
        <td><b>辅助组</b></td>
        <td><b>用户主目录</b></td>
        <td><b>ORACLE_BASE</b></td>
        <td><b>ORACLE_HOME</b></td>
    </tr>
    <tr>
        <td>Grid Infrastructure</td>
        <td>grid</td>
        <td>asmadmin</br>asmdba</br>asmoper</br>dba</td>
        <td>/home/grid</td>        
        <td>/u01/app/grid/crs</td>
        <td>/u01/app/11.2.0/grid</td>
    </tr>
    <tr>
        <td>Oracle RAC</td>
        <td>oracle</td>
        <td>dba</br>dbaoper</br>asmdba</td>
        <td>/home/oracle</td>
        <td>/u01/app/oracle</td>
        <td>1/u01/app/oracle/11.2.0/db_1</td>       
    </tr>
</table>

### RAC节点 ###

<table>
    <tr>
        <td><b>节点名称</b></td>
        <td><b>实例名称</b></td>
        <td><b>数据库名称</b></td>
        <td><b>内存</b></td>
        <td><b>操作系统</b></td>
    </tr>
    <tr>
        <td>rac1</td>
        <td>rac1</td>
        <td rowspan="2">rac</td>
        <td>2G</td>        
        <td>Redhat 6.9 x86_64</td>
    </tr>
    <tr>
        <td>rac2</td>
        <td>rac2</td>
        <td>2G</td>
        <td>Redhat 6.9 x86_64</td>     
    </tr>
</table>

### 存储组件 ###

<table>
    <tr>
        <td><b>存储组件</b></td>
        <td><b>文件系统</b></td>
        <td><b>卷大小</b></td>
        <td><b>ASM卷组名</b></td>
        <td><b>ASM冗余</b></td>
        <td><b>磁盘名</b></td>
    </tr>
    <tr>
        <td>OCR、VOTING Disk</td>
        <td rowspan="3">ASM</td>
        <td>5G</td>        
        <td>CRS</td>
        <td>External</td>
        <td>OCR_VOTE</td>
    </tr>
    <tr>
        <td>数据库</td>
        <td>20G</td>        
        <td>DATA</td>
        <td>External</td>
        <td>ASMDATA</td>    
    </tr>
    <tr>
        <td>快速恢复区</td>
        <td>5G</td>        
        <td>FRA</td>
        <td>External</td>
        <td>BACKUP</td>    
    </tr>
</table>

### 删除自动生成的虚拟网卡 ###

执行命令删除虚拟网卡

	virsh list
	virsh net-destroy default
	virsh net-undefine default
	service libvirtd restart

### 网络验证 ###

注意关闭防火墙

- 物理机ping两台虚拟机rac1、rac2的public IP
- rac1节点ping rac2节点的Public IP和private IP
- rac2节点ping rac1节点的public IP和private IP

### 添加共享存储 ###

物理中添加共享磁盘（-a指定磁盘类型，-t 2表示直接划分一个顶分配空间的文件）

	vmware-vdiskmanager.exe -c -s 5G -a lsilogic -t 2 "F:\RAC\shared"\asm1.vmdk
	vmware-vdiskmanager.exe -c -s 5G -a lsilogic -t 2 "F:\RAC\shared"\asm2.vmdk
	vmware-vdiskmanager.exe -c -s 20G -a lsilogic -t 2 "D:\RAC\shared"\asm3.vmdk

关闭节点，编辑vmx文件，如rac1.vmx

添加如下内容

	#shared disks configure
	disk.EnableUUID = "TRUE"
	disk.locking = "FALSE"
	diskLib.dataCacheMaxSize = "0"
	diskLib.dataCacheMaxReadAheadSize = "0"
	diskLib.dataCacheMinReadAheadSize = "0"
	diskLib.maxUnsyncedWrites = "0"
	
	scsi1.present = "TRUE"
	scsi1.virtualDev = "lsilogic"
	scsil.sharedBus = "VIRTUAL"
	
	scsi1:0.present = "TRUE"
	scsi1:0.mode = "independent-persistent"
	scsi1:0.fileName = "F:\RAC\shared\asm1.vmdk"
	scsi1:0.deviceType = "disk"
	scsi1:0.redo = ""
	
	scsi1:1.present = "TRUE"
	scsi1:1.mode = "independent-persistent"
	scsi1:1.fileName = "F:\RAC\shared\asm2.vmdk"
	scsi1:1.deviceType = "disk"
	scsi1:1.redo = ""
	
	scsi1:2.present = "TRUE"
	scsi1:2.mode = "independent-persistent"
	scsi1:2.fileName = "D:\RAC\shared\asm3.vmdk" 
	scsi1:2.deviceType = "disk"
	scsi1:2.redo = ""

注意：
	
	1、此处添加了3块共享盘，因此添加3段内容scsi1:0, scsi1:2,scsi1:3, 即增加几块硬盘则增加几段；
	2、scsi1:*.fileName=后面的内容要与物理机创建的磁盘位置对应
	3、重启虚拟机软件后验证加载是否成功（设置内查看硬盘）

### 实现共享存储 ###

1、 划分共享磁盘（单节点执行即可）

`fdisk -l`查看硬盘

将3块磁盘分为主分区

2、 配置ASM磁盘

`rpm -qa | grep udev`检查是否安装udev

执行命令获取scsi id信息

	scsi_id -g -u -d /dev/sdb


配置udev配置文件

	vim /etc/udev/rules.d/99-x-asmdisk.rules

----
	
	KERNEL=="sdb1", BUS=="scsi", PROGRAM="scsi_id -g -u -d /dev/$parent", RESULT=="36000c29701fe06170d4bebc3309da371", NAME="asmdiskOCR", OWNER:="grid", GROUP:="dba", MODE="0660"
	KERNEL=="sdc1", BUS=="scsi", PROGRAM="scsi_id -g -u -d /dev/$parent", RESULT=="36000c296b6029fcfd67886f45b8d686f", NAME="asmdiskDATA", OWNER:="grid", GROUP:="dba", MODE="0660"
	KERNEL=="sdd1", BUS=="scsi", PROGRAM="scsi_id -g -u -d /dev/$parent", RESULT=="36000c296d760fed43e99f8843447f507", NAME="asmdiskFRA", OWNER:="grid", GROUP:="dba", MODE="0660"

----

	start_udev

----

若该命令提示成功，实际未生效，可尝试重启节点

	[root@rac1 rules.d]# ls -al /dev/asmdisk*
	brw-rw----. 1 grid dba 8, 33 Oct 12 03:27 /dev/asmdiskDATA
	brw-rw----. 1 grid dba 8, 49 Oct 12 03:27 /dev/asmdiskFRA
	brw-rw----. 1 grid dba 8, 17 Oct 12 03:27 /dev/asmdiskOCR

### 配置Linux系统

1、 用户组及用户设置

创建Oracle软件组

	groupadd -g 601 oinstall
	groupadd -g 602 dba
	groupadd -g 603 oper
    groupadd -g 604 asmadmin
    groupadd -g 605 asmdba
    groupadd -g 606 asmoper

创建grid、oracle用户

	useradd -u 601 -g oinstall -G asmadmin,asmdba,asmoper grid
    useradd -u 602 -g oinstall -G dba,oper,asmdba oracle

为oracle及grid用户设置密码

	passwd grid
	passwd oracle

2、配置hosts文件

添加
	
	#public:
	192.168.75.50 rac1
	192.168.75.53 rac2
	
	#vip:
	192.168.75.51 rac1-vip
	192.168.75.54 rac2-vip
	
	#pvip:
	192.168.0.2 rac1-pvip
	192.168.0.3 rac2-pvip
	
	#scan:
	192.168.75.52 rac-scan

3、修改Linux内核参数

	vim /etc/sysctl.conf

添加

	fs.aio-max-nr = 1048576
	fs.file-max = 6815744
	kernel.shmmni = 4096
	kernel.sem = 250 32000 100 128
	net.ipv4.ip_local_port_range = 9000 65500
	net.core.rmem_default = 262144
	net.core.rmem_max = 4194304
	net.core.wmem_default = 262144
	net.core.wmem_max = 1048576

执行`sysctl -p`生效

4、设置grid、oracle用户shell limits

	vim /etc/security/limits.conf

添加

	grid soft nproc 2047
	grid hard nproc 16384
	grid soft nofile 1024
	grid hard nofile 65536
	grid soft stack 10240
	grid hard stack 32768
	
	oracle soft nproc 2047
	oracle hard nproc 16384
	oracle soft nofile 1024
	oracle hard nofile 65536
	oracle soft stack 10240
	oracle hard stack 32768

------

	vim /etc/pam.d/login

添加

	session required pam_limits.so

5、创建Oracle Inventory Directory

	mkdir -p /u01/app/oraInventory
    chown -R grid.oinstall /u01/app/oraInventory
    chmod -R 775 /u01/app/oraInventory

6、创建Oracle Grid Infrastructure Home目录

	mkdir -p /u01/app/grid/crs
    mkdir -p /u01/app/grid/11.2.0
    mkdir -p grid.oinstall /u01/app/grid
    chmod -R 775 /u01/app/grid

7、创建Oracle RDBMS Home目录

	mkdir -p /u01/app/oracle
    chown -R oracle.oinstall /u01/app/oracle
    chmod -R 775 /u01/app/oracle
    mkdir -p /u01/app/oracle/product/11.2.0/db_1
    chown -R oracle.oinstall /u01/app/oracle/product/11.2.0/db_1
    chmod -R 775 /u01/app/oracle/product/11.2.0/db_1

8、安装依赖包（64&32bit）

	binutils
	compat-libstdc++-33
	elfutils-libelf
	elfutils-libelf-devel
	gcc
	gcc-c++
	glibc
	glibc-common
	glibc-devel
	glibc-headers
	ksh
	libaio
	libaio-devel
	libgcc
	libstdc++
	libstdc++-devel
	make
	numactl-devel
	sysstat
	unixODBC
	unixODBC-devel

9、修改环境变量

	su - grid
	vim .bash_profile

----

	export ORACLE_HOSTNAME=rac1(rac2)
	export ORACLE_UNQNAME=rac
	export ORACLE_BASE=/u01/app/grid/crs
	export ORACLE_HOME=/u01/app/grid/11.2.0
	export ORACLE_SID=+ASM1(+ASM2)
	export ORACLE_TERM=xterm
	export PATH=$ORACLE_HOME/bin:/usr/sbin:$PATH
	export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib
	export CLASSPATH=$ORACLE_HOME/JRE:$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib
	export TMP=/tmp
	export TMPDIR=$TMP

-----

	su - oracle
	vim .bash_profile

------

	export ORACLE_HOSTNAME=rac1(rac2)
	export ORACLE_UNQNAME=rac
	export ORACLE_BASE=/u01/app/oracle
	export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/db_1
	export ORACLE_SID=rac1(rac2)
	export ORACLE_TERM=xterm
	export PATH=$ORACLE_HOME/bin:/usr/sbin:$PATH
	export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib
	export CLASSPATH=$ORACLE_HOME/JRE:$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib
	export TMP=/tmp
	export TMPDIR=$TMP

### 关闭防火墙及SELINUX

	setenforce 0 --临时关闭
	getenforce --查看selinux状态
	vim /etc/selinux/config --永久关闭

	sevice iptables stop
	chkconfig iptables off

### 设置grid、oracle用户免密登录

	su - grid(oracle)
    rm -rf ~/.ssh
    mkdir ~/.ssh
    chmod 700 ~/.ssh
    ssh-keygen -t rsa
    ssh-keygen -t dsa
    cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
    cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
    scp ~/.ssh/authorized_keys rac2(rac1):~/.ssh/authorized_keys

检测连通性

	ssh rac1 date
    ssh rac2 date
    ssh rac2-pvip date
    ssh rac1-pvip date

### 时钟同步 ###

1、NTP同步

	vim /etc/ntp.conf #rac1
	Server 127.127.1.0
	Fudge 127.127.1.0 startum 11
	Broadcastdelay 0.008

	vim /etc/ntp.conf #rac2
	Server 192.168.0.2 prefer
	Driftfile /var/lib/drift
	Broadcastdelay 0.008

	vim /etc/sysconfig/ntpd
	SYNC_HWCLOCK=yes
	OPTIONS="-x -u ntp:ntp -p /var/run/ntpd.pid"
	
	/etc/init.d/ntpd restart  #启动服务
    chkconfig ntpd on         #开机自启
    netstat -an | grep 123    #查看相应UDP端口
    ntpstat                   #查看NTP服务状态

2、使用oracle集群软件ctss服务同步

11G R2默认有自己的时间同步机之，没有NTP也是可以的。有NTP的话，ctss运行观察模式，使用集群时间同步服务再集群中提供同步服务，需要卸载网络时间协议（NTP）及其配置

	servie ntpd stop
	chkconfig ntp off
	mv /etc/ntp.conf /etc/ntp.conf.original #ctss启用依据：/etc/ntp.conf
	rm /var/run/ntpd.pid

### 安装前准备 ###

rac1上传database及grid

rac2上传grid即可

安装系统程序包cvuqdisk，如果没有安装cvuqdisk，集群验证实用程序就无法发现共享磁盘（/dev/asm*），而且安装后期会收到"Package cvuqdisk not installed"的错误。

root用户下安装：

	export CVUQDISK_GRP=ointall #未执行未影响
	cd /setup/grid/rpm
	rpm -ivh cvuqdisk-1.0.9-1.rpm

环境检查

	su - grid
	cd /setup/grid
	./runcluvfy.sh stage-pre crsinst -n rac1,rac2 -fixup -verbose > 1.log

可根据日志进行对应的修复

对于网络问题、时间同步、DNS问题可忽略

如果还是有报错（未遇到），可执行如下命令后再次检查

	/tmp/CVU_11.2.0.4.0_grid/runfixup.sh
	rm -rf /tmp/bootstrap
	./runcluvfy.sh stage-pre crsinst -n rac1,rac2 -fixup -verbose

	#注：
	#两条CVU检查命令的结果均为passed才能继续安装
	#要保证4个IP（Public、Private）可以互相ping通

### 安装Oracle Grid Infrastructure ###

1、安装流程（单节点执行）


图形界面VNC下root用户执行`xhost +`，给图形化安装赋予权限。

	su - grid
	cd /setup/grid
	./runInstaller

- Installtion Type：选择 Advance Installation（方便参数设置）
- Grid Plug and Play：Cluster Name可以随意命名；SCAN Name需要与/etc/hosts中配置一致；端口默认1521；不配置GNS，取消Configure GNS选择
- Cluster Note Information：添加节点2信息
- Create ASM Disk Group：Disk Group Name修改为CRS；Redundancy　选择External；Change Discovery Path修改路径为/dev/asm*；磁盘选择OCR对应磁盘
- Failure Isolation：不使用PIMI，选择Do not use Intelligent Platform Management Interface(PIMI)
- Prerequisite Checks：如果两个节点执行`ll /dev/asm*`内容一致，则Device Checks for ASM警告可以忽略；如果使用Linux NTP服务，则出现Network Time Protocol警告（使用CTSS则不会报警）；对于DNS、网络等的警告，也可忽略
- 脚本执行，分节点依次执行
- INS-20802是监听错误，原因在于/etc/hosts文件中配置了SCAN的地址，如果可以PING通SCAN-IP，则可忽略此错误

2、安装后检查

	#检查CRS状态
	[root@rac1 ~]$ su - grid
	[grid@rac1 ~]$ crsctl check crs 
	CRS-4638: Oracle High Availability Services is online
	CRS-4537: Cluster Ready Services is online
	CRS-4529: Cluster Synchronization Services is online
	CRS-4533: Event Manager is online

	#检查Cluster资源
	[grid@rac1 ~]$ crs_stat -t
	Name           Type           Target    State     Host
	------------------------------------------------------
	ora.CRS.dg     ora....up.type ONLINE    ONLINE    rac1
	ora.DATA.dg    ora....up.type ONLINE    ONLINE    rac1
	ora.FRA.dg     ora....up.type ONLINE    ONLINE    rac1
	ora....ER.lsnr ora....er.type ONLINE    ONLINE    rac1
	ora....N1.lsnr ora....er.type ONLINE    ONLINE    rac2
	ora.asm        ora.asm.type   ONLINE    ONLINE    rac1
	ora.cvu        ora.cvu.type   ONLINE    ONLINE    rac1
	ora.gsd        ora.gsd.type   OFFLINE   OFFLINE
	ora....network ora....rk.type ONLINE    ONLINE    rac1
	ora.oc4j       ora.oc4j.type  ONLINE    ONLINE    rac1
	ora.ons        ora.ons.type   ONLINE    ONLINE    rac1
	ora.rac.db     ora....se.type ONLINE    ONLINE    rac1
	ora....SM1.asm application    ONLINE    ONLINE    rac1
	ora....C1.lsnr application    ONLINE    ONLINE    rac1
	ora.rac1.gsd   application    OFFLINE   OFFLINE
	ora.rac1.ons   application    ONLINE    ONLINE    rac1
	ora.rac1.vip   ora....t1.type ONLINE    ONLINE    rac1
	ora....SM2.asm application    ONLINE    ONLINE    rac2
	ora....C2.lsnr application    ONLINE    ONLINE    rac2
	ora.rac2.gsd   application    OFFLINE   OFFLINE
	ora.rac2.ons   application    ONLINE    ONLINE    rac2
	ora.rac2.vip   ora....t1.type ONLINE    ONLINE    rac2
	ora....ry.acfs ora....fs.type ONLINE    ONLINE    rac1
	ora.scan1.vip  ora....ip.type ONLINE    ONLINE    rac2

	#检查CRS节点信息
	[grid@rac1 ~]$ olsnodes -n
	rac1    1
	rac2    2

	#检查两节点Oracle TNS监听进程
	[grid@rac2 ~]$  ps -ef | grep lsnr | grep -v grep | grep -v ocfs | awk '{print$9}'
	LISTENER_SCAN1
	LISTENER

	#确认针对Oracle Cluster文件的Oracle ASM功能
	[grid@rac1 ~]$ srvctl status asm -a
	ASM is running on rac2,rac1
	ASM is enabled.
	
3、配置ASM磁盘组

	su - grid
	asmca

分别添加数据DATA及闪回FRA盘

### 安装Oracle Database ###

1、安装流程（单节点）
	
	图形界面VNC下root用户执行`xhost +`，给图形化安装赋予权限。

	su - oracle
	cd /setup/database
	./runInstaller
	
- Installation Option：选择仅安装数据库软件（Install database software only）
- Grid Installation Option：选择Oracle Real Application database installation安装RAC；select ALL选择全部节点并测试SSH连通性
- Prerequisite Checks：安装预检出现SCAN警告，原因为SCAN-IP最多可以配置3个地址并对应域名，此处未使用DNS Name且仅配置了一个IP，因此报错，但不会影响系统使用，可以忽略

2、安装数据库

	#检查监听程序是否存在
	[oracle@rac1 ~]$ lsnrctl status

	LSNRCTL for Linux: Version 11.2.0.4.0 - Production on 12-OCT-2020 18:10:40
	
	Copyright (c) 1991, 2013, Oracle.  All rights reserved.
	
	Connecting to (ADDRESS=(PROTOCOL=tcp)(HOST=)(PORT=1521))
	STATUS of the LISTENER
	------------------------
	Alias                     LISTENER
	Version                   TNSLSNR for Linux: Version 11.2.0.4.0 - Production
	Start Date                12-OCT-2020 11:48:53
	Uptime                    0 days 6 hr. 21 min. 46 sec
	Trace Level               off
	Security                  ON: Local OS Authentication
	SNMP                      OFF
	Listener Log File         /u01/app/oracle/diag/tnslsnr/rac1/listener/alert/log.xml
	Listening Endpoints Summary...
	  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=rac1)(PORT=1521)))
	Services Summary...
	Service "+ASM" has 1 instance(s).
	  Instance "+ASM1", status READY, has 1 handler(s) for this service...
	Service "rac" has 2 instance(s).
	  Instance "rac1", status READY, has 2 handler(s) for this service...
	  Instance "rac2", status READY, has 1 handler(s) for this service...
	Service "racXDB" has 2 instance(s).
	  Instance "rac1", status READY, has 1 handler(s) for this service...
	  Instance "rac2", status READY, has 1 handler(s) for this service...
	The command completed successfully

执行`dbca`创建数据库

- Database Identification：选择Admin-Managed，填入GLobal Database Name（.bash_profile中配置的ORACLE_UNQNAME），并选择所有节点
- Database File Location：选择Use Oracle-Managed Files，Database Area选择+DATA目录
- Recovery Configuration：选择闪回区ASM磁盘组，按需启用规模（默认不开启），勾选Sample Schemes
- Initialization Parameters：Character Sets中字符集修改为ZHS16CBK - CBK 16-bit Simplified Chinese

### Oracle RAC基础维护 ###

Oracle Clusterware的命令可以分为以下4类：

- 节点层：osnodes
- 网络层：oifcfg
- 集群层：crsctl、ocrcheck、ocrdump、ocrconfig
- 应用层：srvctl、onsctl、crs_stat

注意：CRS维护需要使用grid用户（root用户需要在grid ORACLE_HOME/bin目录下执行，建议在grid用户下维护）

1、节点层

	[grid@rac1 ~]$ olsnodes -n -i -s
	rac1    1       rac1-vip        Active
	rac2    2       rac2-vip        Active

2、网络层

	#列出CRS网卡
	[grid@rac1 ~]$ oifcfg iflist
	eth0  192.168.0.0
	eth0  169.254.0.0
	eth1  192.168.75.0

	#获取CRS网卡信息
	[grid@rac1 ~]$ oifcfg getif
	*  192.168.0.0  global  cluster_interconnect
	*  192.168.75.0  global  public

3、集群层

	#检查CRS状态
	[grid@rac1 ~]$ crsctl check crs
	CRS-4638: Oracle High Availability Services is online
	CRS-4537: Cluster Ready Services is online
	CRS-4529: Cluster Synchronization Services is online
	CRS-4533: Event Manager is online

	#检查CRS单个服务
	[grid@rac1 ~]$ crsctl check cssd
	CRS-272: This command remains for backward compatibility only
	Cluster Synchronization Services is online
	[grid@rac1 ~]$ crsctl check crsd
	CRS-272: This command remains for backward compatibility only
	Cluster Ready Services is online
	[grid@rac1 ~]$ crsctl check evmd
	CRS-272: This command remains for backward compatibility only
	Event Manager is online
	
	#查看votedisk磁盘位置
	[grid@rac1 ~]$ crsctl query css votedisk
	##  STATE    File Universal Id                File Name Disk group
	--  -----    -----------------                --------- ---------
	 1. ONLINE   8b27bd0c62a14f71bf70f457ca75c65d (/dev/asmdiskOCR) [CRS]
	Located 1 voting disk(s).

维护votedisk

以图新方式安装Clusterware的过程中，在配置votedisk时，如果选择External Redundancy策略。则只能填写一个votedisk。但是即使使用External Redundancy作为冗余策略，也可以添加多个votedisk，只是必须通过crsctl命令来添加，添加多个votedisk后，这些votedisk互为镜像，可以防止votedisk的单点故障。

需要注意的是，votedisk使用的是以中”多数可用算法“，如果有多个votedisk，则必须一半以上的votedisk同时使用，Cluterware才能正常使用，比如配置了4个votedisk，坏一个votedisk，集群可以正常工作，如果坏了2个，则不能满足半数以上，集群会立即宕掉，所有节点立即重启，所以如果添加votedisk，尽量不要添加一个，而应该添加2个，这点和OCR不一样，OCR只需配置一个。

添加和删除votedisk的操作比较危险，必须停止数据库，停止ASM，停止CRS Stack后操作，并且操作时必须使用-force参数。

以下操作均应当使用root用户执行

	1、查看当前配置
	./crsctl query css votedisk
	2、停止所有节点的CRS
	./crsctl stop crs
	3、添加votedisk
	./crsctl add css votedisk /dev/raw/raw1 -force
	注意：即使在CRS关闭后，也必须通过-force参数来添加和删除votedisk，并且-force参数只有在CRS关闭的场合下使用才安全，否则会报：Cluster is not a ready state for online disk addition.
	4、确认添加后的情况
	./crsctl query css votedisk
	5、启动CRS	
	./crsctl start crs

-----
	
	查看OCR状态
	[grid@rac1 ~]$ ocrcheck
	Status of Oracle Cluster Registry is as follows :
	         Version                  :          3
	         Total space (kbytes)     :     262120
	         Used space (kbytes)      :       3096
	         Available space (kbytes) :     259024
	         ID                       : 1630552974
	         Device/File Name         :       +CRS
	                                    Device/File integrity check succeeded
	
	                                    Device/File not configured
	
	                                    Device/File not configured
	
	                                    Device/File not configured
	
	                                    Device/File not configured
	
	         Cluster registry integrity check succeeded
	
	         Logical corruption check bypassed due to non-privileged user

维护OCR

- 查看OCR的自动备份：`ocrconfig-showbackup`
- 使用导出、导入进行备份、回复模拟案例（root用户）

Oracle推荐在对集群做调整时，比如增加、删除节点之前，应该对OCR做一个备份，可以使用export备份到指定文件，如果做了replace或者restore等操作，Oracle建议使用cluvfy comp ocr -n all命令来做一次全面的检查。该命令在clusterware的安装软件里。

	1）首先关闭所有节点的CRS
	./crsctl stop crs
	2)用root用户导出OCR内容
	./ocrconfig -export /u01/orc.exp
	3)重启CRS
	./crsctl start crs
	4)检查OCR状态
	./crsctl check crs
	5)破坏OCR内容
	dd if=/dev/zero of=/dev/raw/raw1 bs=1024 count=102400
	6)检查OCR一致性
	./ocrcheck
	7)使用clufy工具检查一致性
	./runcluvfy.sh comp ocr -n all
	8)使用import回复OCR内容
	./ocrconfig -import /u01/ocr.exp
	9)再次检查OCR
	./ocrcheck
	10)使用cluvfy工具检查
	./runcluvfy.sh comp  ocr -n all

- 移动OCR文件位置模拟案例（root用户）

将OCR从/dev/raw/raw1移动到/dev/raw/raw3上

	1)查看是否有OCR备份
	./ocrconfig -showbackup
	如果没有备份，可以立即执行一次导出作为备份：
	./ocrconfig -export /u01/ocrbackup -s online
	2)查看当前OCR配置
	./ocrcheck
	Status of Oracle Cluster Registry is as follows :
         Version                  :          3
         Total space (kbytes)     :     262120
         Used space (kbytes)      :       3096
         Available space (kbytes) :     259024
         ID                       : 1630552974
         Device/File Name         : /dev/raw/raw1
                                    Device/File integrity check succeeded

                                    Device/File not configured

                                    Device/File not configured

                                    Device/File not configured

                                    Device/File not configured

         Cluster registry integrity check succeeded
	输出显示当前只有一个Primary OCR，在/dev/raw/raw1，没有Mirror OCR。因为现在只有一个OCR文件，所以不能直接改变这个OCR的位置，必须现添加镜像后再修改，否则会报：Failed to initialize ocrconfig.
	3)添加一个Mirror OCR
	./ocrconfig -replace ocrmirror /dev/raw/raw4
	4)确认添加成功
	./ocrcheck
	5)改变primary OCR位置
	./ocrconfig -replace ocr /dev/raw/raw3
	确认修改成功
	./ocrcheck
	6)使用ocrconfig命令修改后，所有RAC节点上的/etc/oracle/ocr.loc文件内容也会自动同步了，如果没有自动同步，可以手动改成以下内容
	more /etc/oracle/ocr.loc
	ocrconfig_Ioc=/dev/raw/raw1
	Ocrmirrorconfig_Ioc=/dev/raw/raw3
	local_only=FALSE

4、应用层

	#查看OCR状态
	crs_stat -t -v
	#查看ora.scan1.vip状态
	crs_stat ora.scan1.vip
	#查看每隔资源的权限定理，权限定义格式和Linux一样	
	crs_stat -ls 

- onsctl命令

onsctl命令用于管理配置ONS（Oracle Notification Service），ONS时Oracle Clusterware实现FAN Event Push模型的基础。

在传统模型中，客户端需要定期检查服务器来判断服务器状态，本质上是一个pull模型，Oracle 10g引入了一个全新的push机之-FAN（Fast Application Notification），当服务器发生某些事件时，服务器会主动的通知客户端这种变化，这样客户端就能尽早得知服务端的变化，而引入这种机制就是依赖ONS实现，在使用onsctl命令之前，需要先配置ONS服务。

	#查看ONS服务状态
	onsctl ping
	#启动ONS
	onsctl debug

- srvctl命令

srvctl可以操作下面的集中资源：Database、Instance、ASM、Service、Listener和Node Application，其中Node Application又包括GSD、ONS、VIP。

	#查看数据库配置
	#显示数据库名
	srvctl config database
	#显示指定rac数据库的详细信息
	srvctl config database -d rac

	#查看各节点资源信息
	srvctl config nodeapps
	srvctl config nodeapps -n rac1 -a -g
	
	#查看监听信息
	srvctl config listener
	srvctl config listener -n rac1
	
	#配置数据库是否启动
	srvctl enable | disable database -d rac
	srvctl enable | disable database -d rac -i rac1

	#启动数据库
	srvctl start database -d rac
	
	#启动实例
	srvctl start instance -d raw

	参考：http://www.cnblogs.com/rootq/archive/2012/11/14/2769421.html

### CRS名词解释 ###

- CRS一些服务作用

---	
	cvu：负责oracle健康检查的进程
	ons：负责不同的节点间的通信，集群同步
	gsd：负责服务资源的分配，只用于9i RAC，但为了向后兼容才保留的，不影响性能。建议不作处理。
	oc4j：是用于DBWLM（Database Workload Management）的一个资源，WLM在11.2.0.2才可用。
	scfs（ASM Cluster File System）：基于ASM的集群文件系统，11.2后增加功能。支持更广泛的存储文件类型。ora.acfs表示该服务的支持与否。

-Oracle Cluster Registry（OCR）
	
OCR负责维护整个集群的配置信息，包括RAC以及Clusterware资源，为了解决集群的“健忘”问题，整个集群会有一份配置OCR，最多两份OCR，一个primary OCR和一个mirror OCR互为镜像，以防OCR的单点故障。

- votedisk

Voting Disk里面记录着节点成员的信息，必须存放在共享存储上，Voting Disk 主要为了再出现脑裂时，决定哪个Partion获得控制权，其他的Partion必须从集群中剔除。4个Votedisk，坏一个Votedisk，集群可以正常工作，如果坏了2个，则不能满足半数以上，集群会立即宕掉，所有节点立即重启。

- Admin Managed 和 Policy Managed

**Policy-Managed方式介绍**

基于策略的管理方式，是以服务器池（Server Pools）为基础的，简单地说，就是先定义一些服务器池，池中包含一定量的服务器，然后再定义一些策略，根据这些策略Oracle会自动决定让多少数据库实例运行在池中的机器上，数据库实例名后缀，数据库实例个数，所运行的主机，这些都是通过策略决定的，而不是数据库管理员事先定好的。

何种环境适合使用这种新的方式进行管理？当管理大量的服务器集群，并且再这些集群中运行着多种不通重要程度，不同策略的RAC数据库时，为了简化管理，建议使用Policy-Managed方式，实际上Oracle也建议只有在超过3台服务器的时候才使用Policy—Managed来管理整个数据库集群，想象一下使用Policy-Managed方式可以达到的效果：如果我们有10台服务器组成，根据不同的应用的重要性定义服务器池的关键程度，然后在其中某些机器意外停机的情况下，仍然可以自动地保持足够多的机器给重要地系统提供数据库服务，而将不关键地系统数据库服务器个数降低到最低限度。


策略管理：DBA指定数据库资源运行在哪个服务器池（排除generic or free）。Oracle Clusterware负责将数据库资源放在一台服务器。

**Admin-Managed方式介绍**

实际上上面的表述已经明确说明了，Policy-Managed和Admin-Managed方式的差别。让我们再回顾一下，在以往我们创建一个RAC数据库大概的方法是怎样的方法，我们在dbca的界面中会选择要将数据库实例运行在整个集群中的几台机器上，或者2台或者3台，甚至是更多，但是只要在安装的时候选定几台机器，那么以后如果不做增减节点的操作，就始终会在这几台机器上运行。而且，通常会根据主机名称的排序自动将每台主机上的数据库实例依次命名为dbname1到dbnameN。这些在管理员安装完毕后，都不会再自动变化，这就是Admin-Managed方式。

管理员管理：DBA指定数据库资源运行的所有服务器，并且按需手动放置资源。这是之前版本Oracle数据库使用的管理策略。

参考：http://czmmiao.iteye.com/blog/2128771

**Grid Name Service(GNS)**

RAC中SCAN的配置方式有三种方式：

- /etc/hosts
- DNS
- GNS

参考：

	http://blog.csdn.net/tianlesoftware/article/details/42970129
	http://download.csdn.net/detail/refengjun/4488454

**Intelligent Management Platform Interface (IPMI)**

智能平台管理接口，IPMI是一种规范的标准，其中最重要的物理部件就是BMC（Baseboard Management Controller），它是以中嵌入式管理微控制器，相当于整个平台管理的“大脑”。通过它IPMI可以监视各个传感器的数据并记录各种事件的日志。在Grid Infrastructure安装期间配置，但一般情况下不配置此选项。

### 安装问题 ###

- 安装界面乱码

默认系统字符集与安装软件字符集不同
	
解决：`export LANG=en_us`

- SQL内中文字符乱码

系统环境变量NLS_LANG与ORACLE SERVER不同

解决：`export NLS_LANG=export NLS_LANG=AMERICAN_AMERICA.ZHS16GBK`

- xhost + 报错

解决：
	
	export DISPLAY=:0
	xhost +
	su - grid
	export DISPLAY=0:0

- 网卡名称与MAC地址不能对应

解决：	
	
	1、vim /etc/udev/rules.d/70-persisent-net.rules
	2、vim /etc/sysconfig/network-scripts/ifcfg-eth0|1

	将1中的eth0、eth1名称及MAC地址修改为2中对应内容，重启网络即可

- udev配置RAC ASM的几种方式

--------

	方式1：60-raw.rules    #裸设备
	vim /etc/udev/rules.d/60-raw.rules
	ACTION="add", KERNEL=="sdb", RUN+="/bin/raw /dev/raw/raw1%N"
	ACTION="add", KERNEL=="raw1", OWNER="grid", GROUP="asmadmin", MODE="660"
	ACTION="add", KERNEL=="sdc", RUN+="/bin/raw /dev/raw/raw2%N"
	ACTION="add", KERNEL=="raw2", OWNER="grid", GROUP="asmadmin", MODE="660"
	ls -l /dev/raw/raw*
	brw-rw---- 1 grid asmadmin 8,96 Jun 29 21:56 /dev/raw1
	brw-rw---- 1 grid asmadmin 8.64 Jun 29 21:56 /dev/raw2

	方式2：99-oracle-asmdevices    #ASM
	vim  /etc/udev/rules.d/99-oracle-asmdevices.rules
	KERNEL=="sdb", NAME="asmdiskb", OWNER="grid", GROUP="asmadmin", MODE="0660"
	KERNEL=="sdc", NAME="asmdiskc", OWNER="grid", GROUP="asmadmin", MODE="0660"

	udevadm control --reload-rules
	start_udev
	Starting udev:[OK]
	ll /dev/asm*
	brw-rw---- 1 grid asmadmin 8,16 Dec 16 15:52 /dev/asmdiskb
	brw-rw---- 1 grid asmadmin 8,32 Dec 16 15:52 /dev/asmdiskc


	

root.sh

	Installing Trace File Analyzer
	Failed to create keys in the OLR, rc = 127, Message:
	  /u01/app/grid/11.2.0/bin/clscfg.bin: error while loading shared libraries: libcap.so.1: cannot open shared object file: No such file or directory


	cd /lib64
	ln -s libcap.so.2.16 libcap.so.1