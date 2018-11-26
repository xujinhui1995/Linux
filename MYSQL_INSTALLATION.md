### MYSQL INSTALLATION

#### Step 1: Adding the MySQL Yum Repository

1. We will use official MySQL Yum Repository, which will provides RPM packages for installing the letest version of MySQL server, client, MySQL Utilities, MySQL Workbench, Connector/ODBC, and Connector/Python for the RHEL/CENTOS7/6/ and Fedora 28-26.

**Important**: These instructions only works on fresh installation of MySQL on the server, if there is already a MySQL installed using a third-party-distribution RPM package, then I recommand you to upgrade or replace the installed MySQL package using the MySQL Yum Repository.

Before Upgrading or Replacing old MySQL package, don't dorget to take all important databases backup and configuration files.

2. Now download and add the following MySQL Yum repository to your respective Linux distribution system's repository list to install the letest version of MySQL.

		# wget https:repo.musql.com/mysql180-community-release-e17-1.noarch.rpm

3. After downloading the package for your Linux platform, now install the downloaded package with the following command.

		# yum localinstall mysql180-community-release-e17-1.noarch.rpm

4. verify MySQL Yum repository has been added.

		# yum repolist enable | grep "mysql.*-community.*"

	![](https://i.imgur.com/6ZrlJla.png)

#### Step 2: Install

1. Install MySQL

		# yum install mysql-community-server

	The above command will install the packages for MySQL: mysql-community-server, mysql-community-client, mysql-community-common, mysql-community-libs.

#### Step 3: Installing MySQL Release Series

1. You can also install different MySQL version using different sub-repositories of MySQL Community Server. The sub-repository for the recent MySQL series (currently MySQL 8.0) is activated by default, and the sub-repositories for all other versions (for example, the MySQL 5.x series) are deactivated by default.

	To install specific version from specific sub-repository, you can use --enable or --disable options using yum-config-manager or dnf config-manager as shown:

		# yum-config-manager --disable mysql57-community
		# yum-config-manager --enable mysql56-community

#### Step 4: Starting the MySQL Server

1. start the server

		# systemctl start mysqld

2. verify the status

		# systemctl status mysqld

3. verify the version

		# mysql --version

#### Step 5: Securing the MySQL Installtion

1. After installation, we find that we don't set the password and never do the configuration.

	The command `mysql_secure_installation` can help us to do that, this command  perform inportant settings like setting the root password, removing anonymous users, removing root login, and so on.

    **Note**: MySQL version 8.0 or higher generates a temporary random password in `/var/log/mysqld.log` after installation.

	This can help us to login mysql and change the password.

	Use below command to see the password before running mysql secure command

		# grep 'temporary password' /var/log/mysqld.log

	Then we get the random default password, we can run 'mysql_secure_installation` command.

		# mysql_secure_installation

	In this step, we can input 'y' all the way.

	
		[root@VM_0_12_centos ~]# mysql_secure_installation
		
		Securing the MySQL server deployment.
		
		Enter password for user root:
		
		The existing password for the user account root has expired. Please set a new password.
		
		New password:
		
		Re-enter new password:
		The 'validate_password' component is installed on the server.
		The subsequent steps will run with the existing configuration
		of the component.
		Using existing password for root.
		
		Estimated strength of the password: 100
		Change the password for root ? ((Press y|Y for Yes, any other key for No) : y
		
		New password:
		
		Re-enter new password:
		
		Estimated strength of the password: 100
		Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y
		By default, a MySQL installation has an anonymous user,
		allowing anyone to log into MySQL without having to have
		a user account created for them. This is intended only for
		testing, and to make the installation go a bit smoother.
		You should remove them before moving into a production
		environment.
		
		Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
		Success.
		
		
		Normally, root should only be allowed to connect from
		'localhost'. This ensures that someone cannot guess at
		the root password from the network.
		
		Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y
		Success.
		
		By default, MySQL comes with a database named 'test' that
		anyone can access. This is also intended only for testing,
		and should be removed before moving into a production
		environment.
		
		
		Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
		 - Dropping test database...
		Success.
		
		 - Removing privileges on test database...
		Success.
		
		Reloading the privilege tables will ensure that all changes
		made so far will take effect immediately.
		
		Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
		Success.

		All done!

#### Step 6: Connecting to MySQL Server

	# mysql -u root -p

#### Step 7: Update

	# yum update mysql-server
