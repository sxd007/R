1. 检查版本，下载对应的mysql， 本次安装中linux版本为centOS 5.5, 32位， 下载的版本为MySQL-5.5.40-1.rhel5.i386.rpm-bundle.tar

2. 解压缩文件

> tar -xf  MySQL-5.5.40-1.rhel5.i386.rpm-bundle.tar   ## -x 表示解压缩，-f  表示使用档案名字

3. 安装mysql- server

> rpm -ivh MySQL-server-5.5.40-1.rhel5.i386.rpm

4. 安装mysql client

> rpm -ivh MySQL-client-5.5.40-1.rhel5.i386.rpm

5. 安装mysql-devl 等

>rpm -ivh MySQL-devel-5.5.40-1.rhel5.i386.rpm
>rpm -ivh MySQL-shared-5.5.40-1.rhel5.i386.rpm
>rpm -ivh MySQL-shared-compat-5.5.40-1.rhel5.i386.rpm
>rpm -ivh MySQL-embedded-5.5.40-1.rhel5.i386.rpm


