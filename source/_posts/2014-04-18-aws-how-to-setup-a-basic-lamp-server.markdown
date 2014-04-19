---
layout: post
title: "AWS: How to setup a basic LAMP server on an EC2 micro instance"
date: 2014-04-18 21:05:00 -0300
comments: true
categories: 
---
## Objective
These are the Yum packages I usually install on an EC2 instance with CentOS. I have done this so many times that I thought that I could save others time by publishing them.

This list may clearly vary according to particular needs so feel free to include or rule out any package.

## Main Packages

Apache, PHP, MySQL, FTP and APC cache

## Procedure

From the AWS EC2 instance dashboard, go to Security Profile and open the following ports: 20, 21, 443, 25, 5000-6000, 23 and 80.

Install PHP

    sudo yum install php php-mcrypt php-mbstring php-gd php-xml php-mysql php-soap php-devel php-pear

Edit /etc/php.ini and adjust the memory limit and your timezone

    memory_limit = 512M
    date.timezone = 'America/New_York'

Install Apache

    sudo yum install httpd httpd-devel 
    sudo service httpd start
    sudo chkconfig httpd on

Configure your virtual access
    
    sudo vim /etc/httpd/conf.d/<your virtual server.conf>

    <VirtualHost *:80>
        ServerName domain.com
        ServerAlias www.domain.com
        DocumentRoot /var/www/html/domain
        ErrorLog /var/log/httpd/domain_error.log
        TransferLog /var/log/httpd/domain_access.log
        LogLevel warn
        DirectoryIndex index.html index.php
    
        <Directory /var/www/html/domain>
            Options FollowSymLinks
            AllowOverride All
        </Directory>
    </VirtualHost>

*Important:* DNS settings are beyond the scope of this post

Install APC

    sudo yum install gcc make pcre-devel 
    pecl install apc

In order to enable the APC extension create 

    /etc/php.d/apc.ini

with the following content

    extension=apc.so

*Notice:* You will then need to edit the apc.ini file and add all the necessary APC configuration lines according to the configuration you need.

Set the server time according to your location

    sudo mv /etc/localtime /etc/localtime.bak
    sudo ln -s /usr/share/zoneinfo/America/New_York /etc/localtime

Make sure to run the services you need (*i.e. httpd*) and set them to be loaded at startup.

Install MySQL

    sudo yum install mysql-server mysql
    sudo service mysqld start

Set MySQL root user and password and run suggested commands to secure your MySQL server.

    /usr/bin/mysqladmin -u root password ‘new-password’
    /usr/bin/mysqladmin -u root -h ip-10-204-63-86 password ‘new-password’
    /usr/bin/mysql_secure_installation
    sudo chkconfig mysqld on

*Important:* If you are running a Micro instance please make sure to create a swap area and follow [these steps](http://www.prowebdev.us/2012/05/amazon-ec2-linux-micro-swap-space.html) to prevent MySQL server from crashing.    

Install FTP

    sudo yum install vsftpd

Install the DB4 package

    sudo yum install db4-utils

Create the list of virtual users

    /etc/vsftpd/virtual-users.txt 

and add your users like this

    username
    password
    username
    password

Create the DB4 database like this

    db_load -T -t hash -f virtual-users.txt /etc/vsftpd/virtual-users.db

Create the PAM file

    sudo vim /etc/pam.d/vsftpd-virtual

and add the following content

    auth required pam_userdb.so db=/etc/vsftpd/virtual-users
    account required pam_userdb.so db=/etc/vsftpd/virtual-users

Make sure the reference name (virtual-users) matches the name you chose for your virtual users file.

Create your configuration file 

    sudo vim /etc/vsftpd/vsftpd.conf

This is a configuration file as an example

    anonymous_enable=NO
    local_enable=YES
    write_enable=YES
    local_umask=022
    force_dot_files=YES
    listen_port=21
    listen=YES
    background=YES
    dirmessage_enable=YES
    xferlog_enable=YES
    connect_from_port_20=YES
    xferlog_std_format=YES
    guest_enable=YES
    guest_username=apache
    local_root=/var/www
    pam_service_name=vsftpd-virtual
    guest_enable=YES
    userlist_enable=YES
    tcp_wrappers=YES
    pasv_enable=YES
    pasv_max_port=6000
    pasv_min_port=5000
    pasv_address=54.201.210.12
    virtual_use_local_privs=YES
    hide_ids=YES
    user_config_dir=/etc/vsftpd/vsftpd_user_conf

*pasv_address* must be your instance public IP
*pam_service_name* corresponds to the PAM file we have previously created
*local_root* is the directory you will access when connecting via FTP
*user_config_dir* is an optional directory with files named after virtual users, storing custom paths and/or settings.
*pasv_max_port* and *pasv_min_port* are arbitrary ports that must be open otherwise there will be errors when transferring files.

Make sure the apache user has read and write permissions on the web root directory.

    chown apache:apache /var/www/html

Restart the service

    sudo service vsftpd restart
    sudo chkconfig vsftpd on

Important: This is a plain FTP server. If you prefer to access your files using FTP clients over SSH, please make sure to give your *ec2-user* access to your web folder.

    sudo usermod -a -G apache ec2-user






















