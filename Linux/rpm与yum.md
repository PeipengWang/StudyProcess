@[toc]
# 软件管理--rpm
rpm 是一种包管理工具，用于管理RPM（Red Hat Package Manager）格式的软件包，通常在Red Hat、Fedora、CentOS和SUSE等Linux发行版中使用。以下是一些常见的rpm命令和示例用法：
## 安装软件包：
rpm -i package.rpm
## 升级软件包：
rpm -U package.rpm
## 卸载软件包：
rpm -e package-name
## 查询已安装的软件包信息：
rpm -q packagname
## 查询尚未安装但已下载的软件包信息：
rpm -qp package.rpm -i
## 列出已安装的软件包：
rpm -qa
与grep联合使用更好
rpm -qa |grep snmp
## 列出软件包的文件信息
rpm -ql pa ckage-name
## 验证软件包完整性
rpm -V package-name
## 导出软件包信息到文件
rpm -qi package-name > package-info.txt
## 查询软件包所属的文件
rpm -qf /path/to/file


# Yum 包管理工具
`yum` 是一种包管理工具，主要用于管理和安装 RPM（Red Hat Package Manager） 软件包，通常在 Red Hat、CentOS 和 Fedora 等 Linux 发行版中使用。以下是一些常见的 `yum` 命令和基本用法：
安装软件包：
sudo yum install package-name
sudo yum install httpd
升级软件包
sudo yum update package-name
卸载软件包
sudo yum remove package-name
搜索软件包
yum search keyword
列出已安装的软件包
yum list installed
列出可用的软件包
yum list available
列出软件包的文件信息
yum list files package-name
查看软件包信息
yum info package-name
清除 Yum 缓存（软件包数据）
sudo yum clean all
列出已安装软件包的更新
sudo yum check-update
## 启用或禁用软件仓库
启用软件仓库
sudo yum-config-manager --enable repository-name
禁用软件仓库
sudo yum-config-manager --disable repository-name
查看当前可用的软件仓库
yum repolist
