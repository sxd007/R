6. 安装完成以后的配置

6.1 如果你使用RPM分发版在Linux上安装MySQL，服务器RPM运行mysql_install_db ##目的是初始化授权表 （首次安装必须使用）。

> mysql_install_db

6.2 首次启动mysql server

> mysqld_safe --user=mysql

6.3 创建用户，设置密码

> mysql -u root #登录mysql， 以下为mysql环境

如果你不知道是哪个主机名，在SET PASSWORD之前执行下面的语句：

> mysql> SELECT Host, User FROM mysql.user;

> GRANT ALL ON *.* TO alpha@'127.0.0.1' IDENTIFIED BY "password";
> GRANT ALL ON *.* TO alpha@'localhost' IDENTIFIED BY "beforward";

用服务器主机名替换第二个SET PASSWORD语句中的host_name。这是指定的user表中的root non-localhost记录的Host列名。

> SET PASSWORD FOR ''@'localhost' = PASSWORD('newpwd');
> set password for root@'localhost' = password('rootxxx'); #举例，注意'的使用

> SET PASSWORD FOR ''@'host_name' = PASSWORD('newpwd');

查找在User列有root和在Host列没有localhost的记录。然后在第二个SET PASSWORD语句中使用该Host值。

为匿名账户指定密码的另一种方法是使用UPDATE直接修改用户表。用root连接服务器，运行UPDATE语句为相应user表记录的Password列指定一个值。在Windows和Unix中的过程是相同的。下面
的UPDATE语句同时为两个匿名账户指定密码：

> UPDATE mysql.user SET Password = PASSWORD('newpwd') WHERE User = '';
> mysql> FLUSH PRIVILEGES; #在user表中直接使用UPDATE更新密码后，必须让服务器用FLUSH PRIVILEGES重新读授权表。否则，重新启动服务器前，不会使用更改。

如果你宁愿删除匿名账户，操作方法是：
> DELETE FROM mysql.user WHERE User = '';
> FLUSH PRIVILEGES;
