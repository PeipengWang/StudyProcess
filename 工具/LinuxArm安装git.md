ar -zxf git-<version>.tar.gz

cd git-<version>

make configure

/configure --prefix=/usr

make install

查看默认安装位置：whereis git

