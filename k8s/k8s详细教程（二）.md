关闭selinux

临时   setenforce  0

永久 sed -i '/senforcing/disabled/' /etc/selinux/config