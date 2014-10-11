在R 中安装 RMySQL之后会提示缺少libz 库

> install.packages('RMySQL')

在ubuntu软件源里zlib和zlib-devel叫做zlib1g zlib1g.dev

>sudo apt-get install zlib1g

>sudo apt-get install zlib1g

发现不能安装，只能进到 http://packages.ubuntu.com/trusty/zlib1g； 以及 http://packages.ubuntu.com/trusty/zlib1g-dev； 

安装后，重新加载RMySQL

Configuration error:
  could not find the MySQL installation include and/or library
  directories.  Manually specify the location of the MySQL
  libraries and the header files and re-run R CMD INSTALL.

INSTRUCTIONS:

1. Define and export the 2 shell variables PKG_CPPFLAGS and
   PKG_LIBS to include the directory for header files (*.h)
   and libraries, for example (using Bourne shell syntax):

      export PKG_CPPFLAGS="-I<MySQL-include-dir>"
      export PKG_LIBS="-L<MySQL-lib-dir> -lmysqlclient"

   Re-run the R INSTALL command:

      R CMD INSTALL RMySQL_<version>.tar.gz

2. Alternatively, you may pass the configure arguments
      --with-mysql-dir=<base-dir> (distribution directory)
   or
      --with-mysql-inc=<base-inc> (where MySQL header files reside)
      --with-mysql-lib=<base-lib> (where MySQL libraries reside)
   in the call to R INSTALL --configure-args='...' 

   R CMD INSTALL --configure-args='--with-mysql-dir=DIR' RMySQL_<version>.tar.gz

ERROR: configuration failed for package ‘RMySQL’
* removing ‘/home/me/R/lib/R/library/RMySQL’

The downloaded source packages are in
	‘/tmp/Rtmp0ziREN/downloaded_packages’
Updating HTML index of packages in '.Library'
Making 'packages.html' ... done
Warning message:
In install.packages("RMySQL") :
  installation of package ‘RMySQL’ had non-zero exit status
