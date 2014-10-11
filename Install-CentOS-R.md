##CentOS 编译安装R

在编译R之前，需要通过yum安装以下几个程序：

#yum install gcc-gfortran             #否则报”configure: error: No F77 compiler found”错误

#yum install gcc gcc-c++              #否则报”configure: error: C++ preprocessor “/lib/cpp” fails sanity check”错误

#yum install readline-devel           #否则报”–with-readline=yes (default) and headers/libs are not available”错误

#yum install libXt-devel              #否则报”configure: error: –with-x=yes (default) and X11 headers/libs are not available”错误

然后下载源代码,编译

下载源代码 /R-3.1.1.tar.gz
#tar -zxvf R-3.1.1.tar.gz
#cd R-2.13.1

#./configure --prefix=/home/alpha/R --enable

#make

#make install

即可完成编译安装
