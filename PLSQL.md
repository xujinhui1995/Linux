### THE INSTALLATION AND CONFIGURATION OF PLSQL

1. packages needed

	- instantclient-sqlplus-windows.x64-12.2.0.1.0.zip
	- instantclient-basic-windows.x64-12.2.0.1.0.zip

	We decompress the "basic" file to such as D:\; Then decompress the "sqlplus" file to the basic directory.

2. After that, we need make a new directory in sqlplus directory, named network, touch a new file named tnsnames.ora, to config the listener.

		D:\PLSQL Developer 12\instantclient-12.2.0.1.0>mkdir network
		D:\PLSQL Developer 12\instantclient-12.2.0.1.0>cd network
		D:\PLSQL Developer 12\instantclient-12.2.0.1.0>type nul>tnsnames.ora

3. configuration of listener file

		OName=
		    (DESCRIPTION =
		        (ADDRESS_LIST =
		            (ADDRESS = (PROTOCOL = TCP)(HOST =192.168.130.12)(PORT =1521))
		        )
		        (CONNECT_DATA =
		            (SERVICE_NAME =tecmint)
		        )
		    )

	Althought the name of connection can be any string, but we'd better use a better name, for we'll log in ORACLE by this name.

4. Log in

	Double click plsqldev.exe to start, and it seemed to create a .lnk file linked to the plsqldev.exe automatically.

	![](https://i.imgur.com/EeHEFuF.png)

	login as DBA
		
		username:system
		database:OName
		Connect as Normal
	
	login as SYSDBA
		
		username:sys
		database:OName
		connect as SYSDBA

	the password is the possword you setted when you install.

	If you choose cancel, you will enter without log in.
		