Ubuntu下安装R很方便，可以在软件管理中心搜索r-base直接安装，也可以将CRAN的相关目录添加到源，然后通过apt-get安装
：
>sudo apt-get install r-base

不过如果想清楚地知道R安装过程中的细节并控制相关的设置，可以采用手工编译.tar.gz的方式安装。

首先需要到CRAN上下载R的源码包，我使用的是最新的版本R-3.1.1.tar.gz。将其拷入某个目录，并解压缩：

>tar -zvxf R-3.1.1.tar.gz
　　
然后进入目录R-2.12.2，运行./configure检查安装的依赖环境并配置安装文件：

>./configure --prefix=/home/alpha/R --enable-R-shlib
　　
注意prefix参数可以设置R将要安装的路径，enable-R-shlib可以保证lib目录下的动态库能够共享，这个选项一定不要忘记添加，否则以后安装某些包的时候会出现Error in dyn.load的错误。
　　
系统提示无法安装，出现依赖错误，这时可以通过以下命令解决路径依赖问题

>apt-get -f install 

继续安装依赖包：

> sudo apt-get install g++

> sudo apt-get install build-essential

> sudo apt-get install gfortran

如果出现错误：configure: error: –with-readline=yes (default) and headers/libs are not available，需要安装libreadline6-dev：

> sudo apt-get install libreadline6-dev

如果出现错误：configure: error: –with-x=yes (default) and X11 headers/libs are not available，需要安装libxt-dev：

> sudo apt-get install libxt-dev

依赖环境装好以后，重新配置

>./configure --prefix=/home/alpha/R --enable-R-shlib


R is now configured for i686-pc-linux-gnu

  Source directory:          .
  Installation directory:    /home/me/R

  C compiler:                gcc -std=gnu99  -g -O2
  Fortran 77 compiler:       gfortran  -g -O2

  C++ compiler:              g++  -g -O2
  C++ 11 compiler:           g++  -std=c++11 -g -O2
  Fortran 90/95 compiler:    gfortran -g -O2
  Obj-C compiler:	      

  Interfaces supported:      X11
  External libraries:        readline
  Additional capabilities:   NLS
  Options enabled:           shared R library, shared BLAS, R profiling

  Recommended packages:      yes

configure: WARNING: you cannot build info or HTML versions of the R manuals
configure: WARNING: you cannot build PDF versions of the R manuals
configure: WARNING: you cannot build PDF versions of vignettes and help pages

显示配置成功！此时进行编译就能成功：

> make

> make install

安装结束后需要手动设置环境变量，可以打开.bashrc文件，添加R_HOME和R_LIBS变量，并修改PATH，这样R就完全安装好了。

1 添加PATH，"..."省略了安装文件路径
"# Source global definitions
if [ -f /etc/bashrc ]; then
. /etc/bashrc
fi
export PATH=/home/.../R/bin:$PATH"
2 添加R_HOME和R_LIB路径
“# R_HOME and R_LIB
PATH=${R_HOME}/bin:$PATH
export R_HOME=/home/.../R/bin/R
export R_LIBS=/home/.../R/lib64/R/library
export LD_LIBRARY_PATH=${R_HOME}/lib:${LD_LIBRARY_PATH}”



　　
