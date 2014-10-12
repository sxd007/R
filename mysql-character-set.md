查看数据库编码：
show create database db_name;

查看表编码：
show create table tbl_name;

查看字段编码：
show full columns from tbl_name;
show full fields from tbl_name;

MySql 端配置
1. 修改my.ini文件
[mysql]
default-character-set=utf8    
说明：修改链接字符集和校对规则，它会同时设置character_set_client, character_set_connection, character_set_results

也可以修改
[mysqld]
default-character-set=utf8
说明： 这里修改的是服务器的字符集和校对规则。

查看当前服务器的字符集和校对规则：
mysql> show variables like 'character_set_server';   
mysql> show variables like 'collation_server';

2. 修改数据库和表的字符集和校队规则。
例如：
-- Create Database.
drop database if exists HRDB;
create database HRDB DEFAULT CHARACTER SET utf8;  # CHARSET=utf8
use HRDB;

-- 角色表
create table HR_ROLE ( 
ID bigint not null auto_increment,
NAME varchar(20) not null unique,
primary key (ID)
) ENGINE=INNODB DEFAULT CHARACTER SET utf8;   # CHARSET=utf8

查看当前数据库的字符集和校对规则：
mysql> show variables like 'character_set_database';   
mysql> show variables like 'collation_database';

查看表的字符集和校对规则：
mysql> show create table HR_ROLE \G;


MySQL字符集终极解决方案
开源数据库MySQL从来都是中小企业构建web应用的首选，特别是和PHP配合简直就是一对黄金搭档，深受web开发人员的喜爱。但自从4.1以来MySQL加入了多字符集的支持，很多MySQL使用者发现中文居然不能使用了，显示变成了一堆乱码！以致于很多人还在使用3.24.58的老版本，最近上MySQL网站，发现居然不提供3.24版本的下载了，MySQL已经彻底放弃3.24版本了。好在我还留有一份windows版的copy，就当作纪念吧。
怎么会产生乱码现象的，怎么解决？只要翻下网上的解决方案，马上就可以得出答案：“在获得连接之后执行一句set names 'gb2312'”，但这样做的原因是什么呢？总结一下我的经验。

MySQL处理连接时，外部连接发送过来的SQL请求会根据以下顺序进行转换：
character_set_client           //客户连接所采用的字符集
|
character_set_connection  //MySQL连接字符集
|
character_set_database    //数据库所采用的字符集（表，列）
|
character_set_results        //客户机显示所采用的字符集


一. 产生乱码的根本原因在于：
1.客户机没有正确地设置client字符集，导致原先的SQL语句被转换成connection所指字符集，而这种转换，是会丢失信息的，如果client是utf8格式，那么如果转换成gb2312格式，这其中必定会丢失信息，反之则不会丢失。一定要保证connection的字符集大于client字符集才能保证转换不丢失信息。
2. 数据库字体没有设置正确，如果数据库字体设置不正确，那么connection字符集转换成database字符集照样丢失编码，原因跟上面一样。

二.为什么set names 'gb2312'就可以了呢
set names 'gb2312'相当于这三条语句:
set character_set_client = gb2312;
set character_set_connection = gb2312;
set character_set_results = gb2312;
这样做的话，上述产生乱码的原因1就不存在了，因为编码格式都统一了，但是这样做并不是万金油。原因有：
1.你的client不一定是用gb2312编码发送SQL的，如果编码不是gb2312那么转换成gb2312就会产生问题。
2.你的数据库中的表不一定是gb2312格式，如果不是gb2312格式而是其他的比如说latin1，那么在存储字符集的时候就会产生信息丢失。

综上，终极解决方案如下:
1.首先要明确你的客户端时候何种编码格式，这是最重要的（IE6一般用utf8，命令行一般是gbk，一般程序是gb2312)
2.确保你的数据库使用utf8格式，很简单，所有编码通吃。
3.一定要保证connection字符集大于等于client字符集，不然就会信息丢失，比如： latin1 < gb2312 < gbk < utf8，若设置set character_set_client = gb2312，那么至少connection的字符集要大于等于gb2312，否则就会丢失信息
4.以上三步做正确的话，那么所有中文都被正确地转换成utf8格式存储进了数据库，为了适应不同的浏览器，不同的客户端，你可以修改character_set_results来以不同的编码显示中文字体，由于utf8是大方向，因此web应用是我还是倾向于使用utf8格式显示中文的。


以上就是我的心得了。附上连接源码，现行设置，程序中就可以不考虑字符集问题了
include "conf/system.php";

class Connection {
private $conn;

function __construct() {
global $mysql_ipaddr, $mysql_port, $mysql_db, $mysql_user, $mysql_pass;

try {
$this->conn = new PDO("mysql:host=$mysql_ipaddr;port=$mysql_port;dbname=$mysql_db", $mysql_user, $mysql_pass);
} catch (PDOException $e) {
print "MySQL服务器连接失败: " . $e->getMessage() . "<br>";
die();
}
}

public function getConnection() {
if ($this->conn != null) {
$this->conn->query("set character_set_client = gb2312");    //客户端使用gb2312格式
$this->conn->query("set character_set_connection = utf8"); //连接字符集使用utf8格式
$this->conn->query("set character_set_results = utf8");       //显示字符集使用utf8格式
return $this->conn;
}
}

public function closeConnection() {
if ($this->conn != null) {
$this->conn = null;
}
}
}
 
Q: 在写一个查询条件时的问题：如我想写一个字段中包含“李”字的所有记录 $str="李";
select * from table where field like '%$str%' ;
显示的记录中除了包含”李”字的记录，还有不包含“李”字的记录。为什么？
A: 在MySQL中，进行中文排序和查找的时候，对汉字的排序和查找结果是错误的。这种情况在MySQL的很多版本中都存在。如果这个问题不解决，那么MySQL将无法实际处理中文。

出现这个问题的原因是：MySQL在查询字符串时是大小写不敏感的，在编绎MySQL时一般以ISO-8859字符集作为默认的字符集，因此在比较过程中中文编码字符大小写转换造成了这种现象。

现在mysql上遇到一个问题,我们的字符集是gb2312.在中文模糊查找时,会有不相关的结果集.
从问题的根本原因分析，还有下面的问题。例： 
汉字“不”的第1、2字节ascii值分别为：178与187 
汉字“安”的第1、2字节ascii值分别为：176与178 
汉字“花”的第1、2字节ascii值分别为：187与168 
聪明的人已经看出来了：在字符串“安花”中模糊查找字符“不”字时，mysql系统也会认为两者匹配!
出现这个问题的原因是：MySQL在查询字符串时是大小写不敏感的，在编绎MySQL时一般以ISO-8859字符集作为默认的字符集，因此在比较过程中中文编码字符大小写转换造成了这种现象。

方法一:
解决方法是对于包含中文的字段加上"binary"属性，使之作为二进制比较，例如将"name char(10)"改成"name char(10)binary"。 

方法二:
如果你使用源码编译MySQL，可以编译MySQL时使用--with--charset=gbk 参数，这样MySQL就会直接支持中文查找和排序了。

方法三:
可以使用 Mysql 的 locate 函数来判断。以上述问题为例,使用方法为:
SELECT * FROM table WHERE locate(field,'李') > 0;
本站使用的就是这种方法，感觉还不错。:P

方法四:
把您的Select语句改成这样,SELECT * FROM TABLE WHERE FIELDS LIKE BINARY '%FIND%'即可!
升级的根本，如果想使用“正确”的字符集，还是先用mysqldump导出成文件，然后导入。
 
--------------------------------------------------------------------------------------------------------------------------------------

MySQL 字符集查询

1） status
[html] view plaincopyprint?
mysql> status;  
--------------  
mysql  Ver 14.14 Distrib 5.1.54, for debian-linux-gnu (x86_64) using readline 6.2  
  
Connection id:      74267  
Current database:     
Current user:       root@localhost  
SSL:            Not in use  
Current pager:      stdout  
Using outfile:      ''  
Using delimiter:    ;  
Server version:     5.5.16-log Source distribution  
Protocol version:   10  
Connection:     Localhost via UNIX socket  
Server characterset:    latin1  
Db     characterset:    latin1  
Client characterset:    latin1  
Conn.  characterset:    latin1  
UNIX socket:        /var/run/mysqld/mysqld.sock  
Uptime:         128 days 13 hours 4 min 59 sec  
  
Threads: 1  Questions: 356155  Slow queries: 2  Opens: 3975  Flush tables: 1  Open tables: 256  Queries per second avg: 0.032  
--------------  

2）show variables like 'collation_%';
[sql] view plaincopyprint?
mysql> show variables like 'collation_%';  
+----------------------+-------------------+  
| Variable_name        | Value             |  
+----------------------+-------------------+  
| collation_connection | utf8_general_ci   |  
| collation_database   | latin1_swedish_ci |  
| collation_server     | latin1_swedish_ci |  
+----------------------+-------------------+  

3）show variables like 'character_%'; 
[sql] view plaincopyprint?
mysql> show variables like 'character_%';  
+--------------------------+----------------------------+  
| Variable_name            | Value                      |  
+--------------------------+----------------------------+  
| character_set_client     | utf8                       |  
| character_set_connection | utf8                       |  
| character_set_database   | latin1                     |  
| character_set_filesystem | binary                     |  
| character_set_results    | utf8                       |  
| character_set_server     | latin1                     |  
| character_set_system     | utf8                       |  
| character_sets_dir       | /usr/share/mysql/charsets/ |  
+--------------------------+----------------------------+  

4） show create table table_name；
[sql] view plaincopyprint?
mysql> show create table t1;  
+-------+------------------------------------  
| Table | Create Table                         
+-------+------------------------------------  
| t1    | CREATE TABLE `t1` (  
  `id` int(11) NOT NULL,  
  `c1` varchar(30) DEFAULT NULL,  
  PRIMARY KEY (`id`)      
) ENGINE=InnoDB DEFAULT CHARSET=gbk |  
+-------+------------------------------------  
1 row in set (0.00 sec)   
                          
mysql> show full columns from t1;  
+-------+-------------+----------------+------+-----+-  
| Field | Type        | Collation      | Null | Key |   
+-------+-------------+----------------+------+-----+-  
| id    | int(11)     | NULL           | NO   | PRI |   
| c1    | varchar(30) | gbk_chinese_ci | YES  |     |   
+-------+-------------+----------------+------+-----+-  

5） show full fields from table_name;
[sql] view plaincopyprint?
mysql> show full fields from user_info;  
+------------+-------------+-------------------+------+-----+---------+-------+---------------------------------+---------+  
| Field      | Type        | Collation         | Null | Key | Default | Extra | Privileges                      | Comment |  
+------------+-------------+-------------------+------+-----+---------+-------+---------------------------------+---------+  
| uid        | bigint(18)  | NULL              | NO   |     | NULL    |       | select,insert,update,references |         |  
| mac_id     | char(17)    | latin1_swedish_ci | NO   |     | NULL    |       | select,insert,update,references |         |  
| name       | varchar(50) | latin1_swedish_ci | YES  |     | NULL    |       | select,insert,update,references |         |  
| nickname   | varchar(50) | latin1_swedish_ci | YES  |     | NULL    |       | select,insert,update,references |         |  
| gender     | tinyint(1)  | NULL              | YES  |     | 0       |       | select,insert,update,references |         |  
| age        | varchar(7)  | latin1_swedish_ci | YES  |     | NULL    |       | select,insert,update,references |         |  
+------------+-------------+-------------------+------+-----+---------+-------+---------------------------------+---------+  

6）查看mysql支持的字符集： show charset;  或 show char set;
[sql] view plaincopyprint?
mysql> show charset;  
+----------+-----------------------------+---------------------+--------+  
| Charset  | Description                 | Default collation   | Maxlen |  
+----------+-----------------------------+---------------------+--------+  
| big5     | Big5 Traditional Chinese    | big5_chinese_ci     |      2 |  
| dec8     | DEC West European           | dec8_swedish_ci     |      1 |  
| cp850    | DOS West European           | cp850_general_ci    |      1 |  
| hp8      | HP West European            | hp8_english_ci      |      1 |  
| koi8r    | KOI8-R Relcom Russian       | koi8r_general_ci    |      1 |  
| latin1   | cp1252 West European        | latin1_swedish_ci   |      1 |  
| latin2   | ISO 8859-2 Central European | latin2_general_ci   |      1 |  
| swe7     | 7bit Swedish                | swe7_swedish_ci     |      1 |  
| ascii    | US ASCII                    | ascii_general_ci    |      1 |  
| ujis     | EUC-JP Japanese             | ujis_japanese_ci    |      3 |  
| sjis     | Shift-JIS Japanese          | sjis_japanese_ci    |      2 |  
| hebrew   | ISO 8859-8 Hebrew           | hebrew_general_ci   |      1 |  
| tis620   | TIS620 Thai                 | tis620_thai_ci      |      1 |  
| euckr    | EUC-KR Korean               | euckr_korean_ci     |      2 |  
| koi8u    | KOI8-U Ukrainian            | koi8u_general_ci    |      1 |  
| gb2312   | GB2312 Simplified Chinese   | gb2312_chinese_ci   |      2 |  
| greek    | ISO 8859-7 Greek            | greek_general_ci    |      1 |  
| cp1250   | Windows Central European    | cp1250_general_ci   |      1 |  
| gbk      | GBK Simplified Chinese      | gbk_chinese_ci      |      2 |  
| latin5   | ISO 8859-9 Turkish          | latin5_turkish_ci   |      1 |  
| armscii8 | ARMSCII-8 Armenian          | armscii8_general_ci |      1 |  
| utf8     | UTF-8 Unicode               | utf8_general_ci     |      3 |  
| ucs2     | UCS-2 Unicode               | ucs2_general_ci     |      2 |  
| cp866    | DOS Russian                 | cp866_general_ci    |      1 |  
| keybcs2  | DOS Kamenicky Czech-Slovak  | keybcs2_general_ci  |      1 |  
| macce    | Mac Central European        | macce_general_ci    |      1 |  
| macroman | Mac West European           | macroman_general_ci |      1 |  
| cp852    | DOS Central European        | cp852_general_ci    |      1 |  
| latin7   | ISO 8859-13 Baltic          | latin7_general_ci   |      1 |  
| utf8mb4  | UTF-8 Unicode               | utf8mb4_general_ci  |      4 |  
| cp1251   | Windows Cyrillic            | cp1251_general_ci   |      1 |  
| utf16    | UTF-16 Unicode              | utf16_general_ci    |      4 |  
| cp1256   | Windows Arabic              | cp1256_general_ci   |      1 |  
| cp1257   | Windows Baltic              | cp1257_general_ci   |      1 |  
| utf32    | UTF-32 Unicode              | utf32_general_ci    |      4 |  
| binary   | Binary pseudo charset       | binary              |      1 |  
| geostd8  | GEOSTD8 Georgian            | geostd8_general_ci  |      1 |  
| cp932    | SJIS for Windows Japanese   | cp932_japanese_ci   |      2 |  
| eucjpms  | UJIS for Windows Japanese   | eucjpms_japanese_ci |      3 |  
+----------+-----------------------------+---------------------+--------+  


MySQL 字符集修改

MySQL中默认字符集的设置有四级：服务器级，数据库级，表级 ，字段级。注意前三种均为默认设置，并不代表你的字段最终会使用这个字符集设置。
MySQL中关于连接环境的字符集设置有 Client端，connection，results 通过这些参数，MySQL就知道你的客户端工具用的是什么字符集，结果集应该是什么字符集。这样MySQL就会做必要的翻译，一旦这些参数有误，自然会导致字符串在转输过程中的转换错误。基本上99%的乱码由些造成。

0） 查看默认数据库集： status
[sql] view plaincopyprint?
mysql> status;  
--------------  
mysql  Ver 14.14 Distrib 5.5.31, for debian-linux-gnu (x86_64) using readline 6.2  
  
Connection id:      41  
Current database:   tvbss_01  
Current user:       root@localhost  
SSL:            Not in use  
Current pager:      stdout  
Using outfile:      ''  
Using delimiter:    ;  
Server version:     5.5.31-0ubuntu0.12.04.1 (Ubuntu)  
Protocol version:   10  
Connection:     Localhost via UNIX socket  
Server characterset:    latin1  
Db     characterset:    latin1  
Client characterset:    utf8  
Conn.  characterset:    utf8  
UNIX socket:        /var/run/mysqld/mysqld.sock  
Uptime:         7 min 30 sec  
  
Threads: 1  Questions: 131  Slow queries: 0  Opens: 239  Flush tables: 1  Open tables: 58  Queries per second avg: 0.291  
--------------  
说明： 通过 sudo apt-get install mysql-server 安装的mysql，默认client和conn为utf8编码，server和db为latin1编码，修改client和conn编码请继续下看。

修改客户端，服务器级，数据库级方法如下：
（1） 使用超级用户root权限，打开 /etc/mysql/my.cnf 
            root@ubuntu:/# vi /etc/mysql/my.cnf        

（2） 修改客户端级，在 [client] 下添加一行：default-character-set=utf8
[html] view plaincopyprint?
[client]  
default-character-set=utf8  
port        = 3306  
socket      = /var/run/mysqld/mysqld.sock  
如果想修改client和conn为latin1，只需把utf8改为latin1，更多编码格式请见下面：show charset;

（3） 修改服务器级，在 [mysqld] 添加两行： character-set-server=utf8    和   collation-server=utf8_general_ci
[html] view plaincopyprint?
[mysqld]  
character-set-server=utf8  
collation-server=utf8_general_ci  
#  
# * Basic Settings  
#  
user        = mysql  
pid-file    = /var/run/mysqld/mysqld.pid  
socket      = /var/run/mysqld/mysqld.sock  

status 查询结果发现： Server 和 Db  变成了 utf8
[sql] view plaincopyprint?
mysql> status;  
--------------  
mysql  Ver 14.14 Distrib 5.5.31, for debian-linux-gnu (x86_64) using readline 6.2  
  
Connection id:      42  
Current database:     
Current user:       root@localhost  
SSL:            Not in use  
Current pager:      stdout  
Using outfile:      ''  
Using delimiter:    ;  
Server version:     5.5.31-0ubuntu0.12.04.1 (Ubuntu)  
Protocol version:   10  
Connection:     Localhost via UNIX socket  
Server characterset:    utf8  
Db     characterset:    utf8  
Client characterset:    utf8  
Conn.  characterset:    utf8  
UNIX socket:        /var/run/mysqld/mysqld.sock  
Uptime:         19 sec  
  
Threads: 1  Questions: 130  Slow queries: 0  Opens: 239  Flush tables: 1  Open tables: 58  Queries per second avg: 6.842  
--------------  

collation 和 character 查询结果发现： collation_server 和 character_set_server 也都变成了 utf8 
[sql] view plaincopyprint?
mysql> show variables like 'character_%';  
+--------------------------+----------------------------+  
| Variable_name            | Value                      |  
+--------------------------+----------------------------+  
| character_set_client     | utf8                       |  
| character_set_connection | utf8                       |  
| character_set_database   | utf8                       |  
| character_set_filesystem | binary                     |  
| character_set_results    | utf8                       |  
| character_set_server     | utf8                       |  
| character_set_system     | utf8                       |  
| character_sets_dir       | /usr/share/mysql/charsets/ |  
+--------------------------+----------------------------+  
  
mysql> show variables like 'collation_%';  
+----------------------+-----------------+  
| Variable_name        | Value           |  
+----------------------+-----------------+  
| collation_connection | utf8_general_ci |  
| collation_database   | utf8_general_ci |  
| collation_server     | utf8_general_ci |  
+----------------------+-----------------+  

（4）修改数据库字符集的两种方法
           a）修改db.opt文件： vi /var/lib/mysql/your_dbname/db.opt           # your_dbname是自己数据库的名称
[sql] view plaincopyprint?
default-character-set=latin1  
default-collation=latin1_swedish_ci  
  
修改为：  
  
default-character-set=utf8  
default-collation=utf8_general_ci  
                   修改后发现：Db     characterset 变为了 utf8
[sql] view plaincopyprint?
mysql> status;  
--------------  
mysql  Ver 14.14 Distrib 5.5.31, for debian-linux-gnu (x86_64) using readline 6.2  
  
Connection id:      42  
Current database:   tvbss_01  
Current user:       root@localhost  
SSL:            Not in use  
Current pager:      stdout  
Using outfile:      ''  
Using delimiter:    ;  
Server version:     5.5.31-0ubuntu0.12.04.1 (Ubuntu)  
Protocol version:   10  
Connection:     Localhost via UNIX socket  
Server characterset:    latin1  
Db     characterset:    utf8  
Client characterset:    utf8  
Conn.  characterset:    utf8  
UNIX socket:        /var/run/mysqld/mysqld.sock  
Uptime:         1 min 22 sec  
  
Threads: 1  Questions: 142  Slow queries: 0  Opens: 239  Flush tables: 1  Open tables: 58  Queries per second avg: 1.731  
--------------  

           b）命令行修改：  mysql> use your_dbname;    mysql> alter database your_dbname character set utf8;     结果同上。且此时命令行也修改了 /var/lib/mysql/your_dbname/db.opt 文件的编码为utf8（同方法a）


MySQL 表、字段的字符集修改
1） 修改表的字符集： ALTER TABLE tbl_name CONVERT TO CHARACTER SET character_name [COLLATE utf8_general_ci]
修改表字符集示例： 数据库表 tbl_name 从latin1 转为 utf8 
[sql] view plaincopyprint?
mysql> show create table db_name.tbl_name;  
+------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+  
| user_info | CREATE TABLE `user_info` (  
  `uid` bigint(18) NOT NULL,  
  `name` varchar(50) DEFAULT NULL,  
  `nickname` varchar(50) DEFAULT NULL,  
  `gender` tinyint(1) DEFAULT '0',  
  `age` varchar(7) DEFAULT NULL  
) ENGINE=InnoDB DEFAULT CHARSET=latin1 |  
+-----------  
  
mysql> alter table table db_name.tbl_name convert to character set utf8 collate utf8_general_ci;  
  
mysql> show create table db_name.tbl_name;  
+------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+  
| user_info | CREATE TABLE `user_info` (  
  `uid` bigint(18) NOT NULL,  
  `name` varchar(50) DEFAULT NULL,  
  `nickname` varchar(50) DEFAULT NULL,  
  `gender` tinyint(1) DEFAULT '0',  
  `age` varchar(7) DEFAULT NULL  
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |  

2）修改表的字段字符集： ALTER TABLE tbl_name CHANGE column_name column_name CHARACTER SET character_name [COLLATE utf8_general_ci...];
修改表的字段字符集示例：   字段 name 从 utf8 转为 latin1
[sql] view plaincopyprint?
mysql> show full fields from db_name.tbl_name;  
+------------+-------------+-----------------+------+-----+---------+-------+---------------------------------+---------+  
| Field      | Type        | Collation       | Null | Key | Default | Extra | Privileges                      | Comment |  
+------------+-------------+-----------------+------+-----+---------+-------+---------------------------------+---------+  
| uid        | bigint(18)  | NULL            | NO   |     | NULL    |       | select,insert,update,references |         |  
| name       | varchar(50) | utf8_general_ci | YES  |     | NULL    |       | select,insert,update,references |         |  
| nickname   | varchar(50) | utf8_general_ci | YES  |     | NULL    |       | select,insert,update,references |         |  
| gender     | tinyint(1)  | NULL            | YES  |     | 0       |       | select,insert,update,references |         |  
| age        | varchar(7)  | utf8_general_ci | YES  |     | NULL    |       | select,insert,update,references |         |  
+------------+-------------+-----------------+------+-----+---------+-------+---------------------------------+---------+  
  
mysql> alter table db_name.tbl_name change name name varchar(50) character set latin1 collate latin1_swedish_ci;  
  
mysql> show full fields from db_name.tbl_name;  
+------------+-------------+-------------------+------+-----+---------+-------+---------------------------------+---------+  
| Field      | Type        | Collation         | Null | Key | Default | Extra | Privileges                      | Comment |  
+------------+-------------+-------------------+------+-----+---------+-------+---------------------------------+---------+  
| uid        | bigint(18)  | NULL              | NO   |     | NULL    |       | select,insert,update,references |         |  
| name       | varchar(50) | latin1_swedish_ci | YES  |     | NULL    |       | select,insert,update,references |         |  
| nickname   | varchar(50) | utf8_general_ci   | YES  |     | NULL    |       | select,insert,update,references |         |  
| gender     | tinyint(1)  | NULL              | YES  |     | 0       |       | select,insert,update,references |         |  
| age        | varchar(7)  | utf8_general_ci   | YES  |     | NULL    |       | select,insert,update,references |         |  
+------------+-------------+-------------------+------+-----+---------+-------+---------------------------------+---------+  



MySQL 连接数
1） 查看连接数
show variables like "max_connections";
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_connections | 500   |
+-----------------+-------+

2） 修改连接数（命令）
set global max_connections = 200;
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_connections | 200   |
+-----------------+-------+
不用重启就生效

3） 修改连接数（配置文件）
sudo vi /etc/mysql/my.cnf
1）去掉注释，修改为： max_connections = 200
2） 重启MySQL生效
