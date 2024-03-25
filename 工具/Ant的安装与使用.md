# Ant的安装使用
首先需要从官网下载apache ant，地址为： http://ant.apache.org/bindownload.cgi
安装Ant
vim /etc/profile
export ANT_HOME=/home/polo/apache-ant-1.9.15/
export PATH=$ANT_HOME/bin:$PATH
让系统配置生效
source /etc/profile
验证 ant
ant -version

