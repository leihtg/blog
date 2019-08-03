---
layout: post
title:  "安装MySQL"
date:   2018-11-08 15:30:21 +0800
categories: MySQL
tags:  mysql
author: leihtg
---

* content
{:toc}

## 在ubuntu中安装mysql

ubuntu上安装mysql非常简单只需要几条命令就可以完成。

> 1. sudo apt-get install mysql-server
> 2. sudo apt-get install mysql-client
> 3. sudo apt-get install libmysqlclient-dev

第三个是本地开发时动态链接库，可以不安装



### 设置MySQL允许远程访问

1. 注释 bind-address = 127.0.0.1
    > sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf

        将bind-address = 127.0.0.1注释掉（即在行首加#），如下：
        
        代码如下:
        
        # Instead of skip-networking the default is now to listen only on
        # localhost which is more compatible and is not less secure.
        # bind-address          = 127.0.0.1
    
    
2. 删除匿名用户登录
    
    > mysql -uroot -p
    
    > use mysql;
    
    > delete from user where user='';
    
3. 增加允许远程访问的用户或者允许现有用户的远程访问。

    接着上面，删除匿名用户后，给root授予在任意主机（%）访问任意数据库的所有权限。SQL语句如下：
    代码如下:
    
    mysql> grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option;
    
    如果需要指定访问主机，可以把%替换为主机的IP或者主机名。另外，这种方法会在数据库mysql的表user中，增加一条记录。如果不想增加记录，只是想把某个已存在的用户（例如root）修改成允许远程主机访问，则可以使用如下SQL来完成：
    代码如下:
    
    update user set host='%' where user='root' and host='localhost';
   
        
4. 重启数据库
   
   > sudo service mysql restart
   