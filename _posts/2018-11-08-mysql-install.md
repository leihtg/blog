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


## 在windows中安装mysql

下载地址： https://dev.mysql.com/downloads/mysql/



1. 解压到指定文件夹
2. 配置my.ini(不配置的话会使用默认配置),在主目录打开my.ini，如果没有则创建，内容如下：
    ```ini
    [mysql]
    
    # 设置mysql客户端默认字符集
    
    default-character-set=UTF8MB4
    
    [mysqld]
    
    #设置3306端口
    
    port = 3306
    
    #binlog日志名称前缀
    log-bin=binlog
    
    #默认值未0，如果使用默认值则不能和从节点通信，这个值的区间是：1到(2^32)-1
    server-id=1
    
    # 设置mysql的安装目录
    
    basedir=D:\Program Files\mysql-8.0.22-winx64
    
    # 设置mysql数据库的数据的存放目录
    
    datadir=E:\mysql\data
    
    # 允许最大连接数
    
    max_connections=200
    
    # 服务端使用的字符集默认为8比特编码的latin1字符集
    
    character-set-server=UTF8MB4
    
    # 创建新表时将使用的默认存储引擎
    
    default-storage-engine=INNODB
    ```
3. 管理员权限进入cmd命令窗口，且cd到安装路径的bin目录下。
4. 执行命令
    ```bash
     # 安装 
     mysqld --install
     # 初始化
     mysqld --initialize
     # 运行 
     net start mysql
    ```
5. 修改密码 在data文件夹下找寻.err后缀的文件,密码在里面有
6. 成功登陆后,会提示修改密码
    ```bash 
    alter user user() identified by "123456";
    
    # 使可远程登录
    alter  user 'root'@'localhost' identified with mysql_native_password by '123456';
    
    alter  user 'root'@'localhost' identified by '123456' password expire never;
    ```


.启动mysql命令：

     net start mysql;

.停止mysql命令：
 
     net stop mysql;
     
如果中途修改数据目录，可能会无法重启
问题描述：

mysql初始化的时候找不到对应的数据库存储目录
```
2018-10-13T03:29:24.205939Z 1 [ERROR] [MY-011011] [Server] Failed to find valid data directory.
2018-10-13T03:29:24.207560Z 0 [ERROR] [MY-010020] [Server] Data Dictionary initialization failed.
```
执行命令     

    mysqld -remove MySQL

然后重新初始化mysql


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
    
4. 设置数据库表大小写不敏感
    
    SHOW VARIABLES LIKE "%case%";
    
    Variable_name           Value   
    ----------------------  --------
    lower_case_file_system  OFF     
    lower_case_table_names  0       
    
    在 mysqld.cnf 中加入一行
    > lower_case_table_names=1    
        
5. 重启数据库
   
   > sudo service mysql restart
   