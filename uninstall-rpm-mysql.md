*先备份mysql数据库

*检查安装的mysql 文件

> rpm -qa|grep MySQL

*执行卸载命令

> rpm -e mysql.xx.rpm #先client, before the server

*如有必要强制卸载

> rpm -e --nodeps mysqlxxxx.rpm

*完成后检查

> rpm -q MySQL   #or
> rpm -qa|grep MySQL
