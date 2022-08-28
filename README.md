# How to install Oracle DB 21 Express Edition (XE), Oracle Applications Express, Oracle Rest Data Services (ORDS) on Centos8Stream
## and secure everything with a reverse proxy using Nginx / LetsEncrypt automatic HTTPS
### and not die trying. 

The following instructions are quite specific for the versions indicated below
- Oracle DB XE 21c 1.0.1
- Oracle APEX version 22.1
- Oracle REST Data Services (ORDS) 22.2.1

These are the current versions as of 26 Aug 2022. They might, or might not be still available at the time you read this. Although I will try to update them regularly, this is a small project I'm doing besides my regular, paying, daytime job so I don't promise I will always have the time to keep it updated with the latest versions. 

I'm using a 6€ a month CX21 Hetzner Cloud server. They are affordable, reliable and if you want to help me pay for my cloud costs, here is a referal link to subscribe: https://hetzner.cloud/?ref=uuMclGJHJMf4 

## Preliminaries:

1. Get a server or a cloud instance with 4 GB RAM, 30 (better 40) GB of SSD and at least a public IPv4 address. IPv6 optional but recommended. Operating system should be CentOS 8 Stream. 
2. Have a A/AAAA (or both) DNS record pointing at that server's IP(s).
3. Update your OS with ```dnf update```
4. Install some prerequisites with ```dnf install -y nano git java-17-openjdk yum-utils epel-release firewalld sudo unzip```
5. After adding the epel-release (CentOS Contrib repo) you need to install some more tools: ```dnf install -y nginx certbot python3-certbot-nginx```
6. Download the Oracle Database Preinstall package: ```wget https://yum.oracle.com/repo/OracleLinux/OL8/appstream/x86_64/getPackage/oracle-database-preinstall-21c-1.0-1.el8.x86_64.rpm```

## Install the preinstall package:

The Preinstall .RPM will do the following things for you: 

1. Create the "oracle" user and its home directory and "orainst" group
2. Install pre-requisites for Oracle DB XE
3. Configure some aspects of your system. 

You can install it by running the following command in the directory where you downloaded the package: ```dnf localinstall -y oracle-database-preinstall-21c-1.0-1.el8.x86_64.rpm```. Some dependencies will also be installed. 

Done! 

## Install and configure Oracle DB XE 21c 1.0.1

I also recommend to install Oracle DB from .rpm packages. It's easier and faster than other methods. 

1. Installation: 

    a. Download the installation package: ```wget https://download.oracle.com/otn-pub/otn_software/db-express/oracle-database-xe-21c-1.0-1.ol8.x86_64.rpm```

    b. Install it with ```dnf localinstall -y oracle-database-xe-21c-1.0-1.ol8.x86_64.rpm```. This process will take some time, on my machine it takes around 10 minutes. It will also install some dependencies. 

2. Configuration and database setup: 

    a. Run ```/etc/init.d/oracle-xe-21c configure```. This command fails the first time I run it, at least on Centos 8 Stream. If that happens to you, just re-run it, it should work fine the second time. 

    b. Enter the password of your choice when for the administrative users when prompted to do so by the script.   

    c. After the process finishes, set Oracle DB XE and TNSListener to start automatically on system reboot: ```chkconfig oracle-xe-21c on```.  

    d. Change to the Oracle user: ```sudo su - oracle```. 

    e. Add the following lines to the end of that users' .bash_profile after the line ```# User specific environment and startup programs```. 

    ```
    export ORACLE_SID=XE 
    export ORAENV_ASK=NO 
    . /opt/oracle/product/21c/dbhomeXE/bin/oraenv 
    ```

    f. Exit the oracle user. The next time you use the oracle user, the proper environment will be loaded



> **If, for whatever reason, anything goes wrong with your database, one of the installation steps of APEX or ORDS fails and you would like to start over, you can run ```/etc/init.d/oracle-xe-21c delete``` to completely delete your database instance and start over by re-running ```/etc/init.d/oracle-xe-21c configure```.**

## Install and configure APEX
> **Please conduct the next steps as the ```oracle``` user!**. 

1. Download the Oracle APEX .zip packege: ``` wget https://download.oracle.com/otn_software/apex/apex_22.1.zip```
2. Unzip the downloaded package: ```unzip apex_22.1.zip```
3. Change into the unzipped folder: ```cd apex```
4. Start **```sqlplus```** and run the following commands as follows, when prompted for a password, specify the one given during Oracle DB installation:
```
[oracle@oracle ~]$ sqlplus /nolog

SQL*Plus: Release 21.0.0.0.0 - Production on Sun Aug 28 11:09:58 2022
Version 21.3.0.0.0

Copyright (c) 1982, 2021, Oracle.  All rights reserved.

SQL> connect sys as sysdba
Enter password: 
Connected.
SQL> alter session set container=xepdb1;

Session altered.

SQL> @apexins.sql SYSAUX SYSAUX TEMP /i/
```
This process will take quite a while. Be patient, with the limits of Oracle XE and the suggested machine size, it can take around 8-12 minutes. 

5. In the same **```sqlplus```** session, run the following command: 
```
@apxchpwd.sql
```

Please follow the prompts given. These credentials provided will be the main administrator for your APEX instance. 

6. Run the following commands in the same **```sqlplus```** session: 
````
ALTER USER APEX_PUBLIC_USER ACCOUNT UNLOCK




# --- Work In Progress - To be continued ---