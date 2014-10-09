## 安装RmySQL前的准备工作
* 检查是否安装了libz， 如果没有的话需要安装libz

> yum install zlib-devel libz.so.1 -y

## 在Linux安装RmySQL
### RMySQL介绍
* RMySQL一个R语言程序包，提供了访问MySQL数据库的R语言接口程序，RMySQL需求依赖于DBI项目。RMySQL不仅提供了基本的数据库访问，SQL查询，还封装了一些方法。比较读整表，分页，data.frame快速插入等等的功能。掌握好RMySQL，数据库编辑将得心应手！！

###  RMySQL在Linux下安装
* 检查Linux的系统环境
 
> uname -a

> cat/etc/issue

> locale

> R --version

> mysql --version

* 在R环境中安装Rmysql;

> install.packages('RMySQL')  #also installing the dependency ‘DBI’

如果出现配置错误，解决后重新执行安装

> R CMD INSTALL RMySQL_.tar.gz

> R CMD INSTALL --configure-args='--with-mysql-dir=/usr/lib/mysql' /path/RMySQL_0.9-3.tar.gz

安装成功

### 测试检验连接

* 在mysql中建库建表

> mysql -uroot -p

mysql> create database mysql;
mysql> grant all on rmysql.* to rmysql@'%' identified by 'rmysql';

