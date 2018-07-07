---
layout: post
title: linux mysql 安装
description: linux mysql 安装
category: share
---

yum install mysql
yum install mariadb-server mariadb
systemctl enable mariadb
systemctl restart mariadb

mkdir software
cd software/
wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm

rpm -ivh mysql-community-release-el7-5.noarch.rpm

yum install mysql-community-server
service mysql start
cp /usr/sbin/mysqld /etc/init.d/
