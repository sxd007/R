下载并解压缩相应的mysql文件包 deb文件.

下载相应的依赖包libaio1_0.3.110-1_i386.deb, 注意对应的版本
  网址 http://us.archive.ubuntu.com/ubuntu/pool/main/liba/libaio/
  
  安装mysql组件，注意先后顺序
  > sudo dpkg -i mysql-common_5.6.21-1ubuntu14.04_i386.deb
  > sudo dpkg -i mysql-community-server_5.6.21-1ubuntu14.04_i386.deb
  > sudo dpkg -i mysql-server_5.6.21-1ubuntu14.04_i386.deb
  
  此时会要求设置mysql-server的根密码, 同时mysql-server 启动
  
  > sudo dpkg -i mysql-community-client_5.6.21-1ubuntu14.04_i386.deb
  > sudo dpkg -i mysql-client_5.6.21-1ubuntu14.04_i386.deb
  > sudo dpkg -i libmysqlclient18_5.6.21-1ubuntu14.04_i386.deb

运行mysql
> mysql -uroot -p

查看数据库列表
> show databases;

+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.01 sec)

添加用户alpha
> GRANT ALL ON *.* TO alpha@'localhost' IDENTIFIED BY "PASSWORD";
> GRANT ALL ON *.* TO alpha@'127.0.0.1' IDENTIFIED BY "PASSWORD";
查看授权表
> SELECT Host, User from mysql.user;

+-----------+-------+
| Host      | User  |
+-----------+-------+
| 127.0.0.1 | alpha |
| 127.0.0.1 | root  |
| ::1       | root  |
| localhost | alpha |
| localhost | root  |
| ubuntu    | root  |
+-----------+-------+
6 rows in set (0.00 sec)

