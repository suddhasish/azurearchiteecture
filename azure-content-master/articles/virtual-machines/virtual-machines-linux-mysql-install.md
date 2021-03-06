<properties
	pageTitle="Set up MySQL on a Linux VM | Microsoft Azure "
	description="Learn how to install the MySQL stack on a Linux virtual machine (Ubuntu or RedHat family OS) in Azure"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="SuperScottz"
	manager="timlt"
	editor=""
	tags="azure-resource-manager,azure-service-management"/>

<tags
	ms.service="virtual-machines-linux"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-linux"
	ms.devlang="na"
	ms.topic="article"
	ms.date="02/01/2016"
	ms.author="mingzhan"/>


#How to install MySQL on Azure


In this article, you will learn how to install and configure MySQL on an Azure virtual machine running Linux.

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-both-include.md)]


##Install MySQL on your virtual machine

> [AZURE.NOTE] You must already have a Microsoft Azure virtual machine running Linux in order to complete this tutorial. Please see the
[Azure Linux VM tutorial](virtual-machines-linux-quick-create-cli.md) to create and set up a Linux VM with `mysqlnode` as the VM name and `azureuser` as user before proceeding.

In this case, use 3306 port as the MySQL port.  

Connect to the Linux VM you created via putty. If this is the first time you use Azure Linux VM, see how to use putty connect to a Linux VM [here](virtual-machines-linux-ssh-from-linux.md).

We will use repository package to install MySQL5.6 as an example in this article. Actually, MySQL5.6 has more improvement in performance than MySQL5.5.  More information [here](http://www.mysqlperformanceblog.com/2013/02/18/is-mysql-5-6-slower-than-mysql-5-5/).


###How to install MySQL5.6 on Ubuntu
We will use Linux VM with Ubuntu from Azure here.

- Step 1: Install MySQL Server 5.6
    Switch to `root` user:

            #[azureuser@mysqlnode:~]sudo su -

    Install mysql-server 5.6:

            #[root@mysqlnode ~]# apt-get update
            #[root@mysqlnode ~]# apt-get -y install mysql-server-5.6

    During installation, you will see a dialog window poping up to ask you to set MySQL root password below, and you need set the password here.

    ![image](./media/virtual-machines-linux-mysql-install/virtual-machines-linux-install-mysql-p1.png)


    Input the password again to confirm.

    ![image](./media/virtual-machines-linux-mysql-install/virtual-machines-linux-install-mysql-p2.png)

- Step 2: Login MySQL Server

    When MySQL server installation finished, MySQL service will be started automatically. You can login MySQL Server with `root` user.
    Use the below command to login and input password.

             #[root@mysqlnode ~]# mysql -uroot -p

- Step 3: Manage the running MySQL service

    (a) Get status of MySQL service

             #[root@mysqlnode ~]# service mysql status

    (b) Start MySQL Service

             #[root@mysqlnode ~]# service mysql start

    (c) Stop MySQL service

             #[root@mysqlnode ~]# service mysql stop

    (d) Restart the MySQL service

             #[root@mysqlnode ~]# service mysql restart


###How to install MySQL on Red Hat OS family like CentOS, Oracle Linux
We will use Linux VM with CentOS or Oracle Linux here.

- Step 1: Add the MySQL Yum repository
    Switch to `root` user:

            #[azureuser@mysqlnode:~]sudo su -

    Download and install the MySQL release package:

            #[root@mysqlnode ~]# wget http://repo.mysql.com/mysql-community-release-el6-5.noarch.rpm
            #[root@mysqlnode ~]# yum localinstall -y mysql-community-release-el6-5.noarch.rpm

- Step 2: Edit below file to enable the MySQL repository for downloading the MySQL5.6 package.

            #[root@mysqlnode ~]# vim /etc/yum.repos.d/mysql-community.repo

    Update each value of this file to below:

        \# *Enable to use MySQL 5.6*

        [mysql56-community]
        name=MySQL 5.6 Community Server

        baseurl=http://repo.mysql.com/yum/mysql-5.6-community/el/6/$basearch/

        enabled=1

        gpgcheck=1

        gpgkey=file:/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

- Step 3: Install MySQL from MySQL repository
    Install MySQL:

           #[root@mysqlnode ~]#yum install mysql-community-server

    MySQL RPM package and all related packages will be installed.

- Step 4: Manage the running MySQL service

    (a) Check the service status of the MySQL server:

           #[root@mysqlnode ~]#service mysqld status

    (b) Check whether the default port of  MySQL server is running:

           #[root@mysqlnode ~]#netstat  ???tunlp|grep 3306


    (c) Start the MySQL server:

           #[root@mysqlnode ~]#service mysqld start

    (d) Stop the MySQL server:

           #[root@mysqlnode ~]#service mysqld stop

    (e) Set MySQL to start when the system boot-up:

           #[root@mysqlnode ~]#chkconfig mysqld on


###How to install MySQL on SUSE Linux
We will use Linux VM with OpenSUSE here.

- Step 1: Download and install MySQL Server

    Switch to `root` user through below command:  

           #sudo su -

    Download and install MySQL package:

           #[root@mysqlnode ~]# zypper update

           #[root@mysqlnode ~]# zypper install mysql-server mysql-devel mysql

- Step 2: Manage the running MySQL service

    (a) Check the status of the MySQL server:

           #[root@mysqlnode ~]# rcmysql status

    (b) Check whether the default port of the MySQL server:

           #[root@mysqlnode ~]# netstat  ???tunlp|grep 3306


    (c) Start the MySQL server:

           #[root@mysqlnode ~]# rcmysql start

    (d) Stop the MySQL server:

           #[root@mysqlnode ~]# rcmysql stop

    (e) Set MySQL to start when the system boot-up:

           #[root@mysqlnode ~]# insserv mysql

###Next Step
Find more usage and information regarding MySQL [Here](https://www.mysql.com/).
