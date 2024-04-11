方式一：cmd命令方式方式

```shell
sudo cp -n /lib/systemd/system/docker.service /etc/systemd/system/docker.service
sudo sed -i "s|ExecStart=/usr/bin/docker daemon|ExecStart=/usr/bin/docker daemon --registry-mirror=你的加速地址|g" /etc/systemd/system/docker.service
sudo sed -i "s|ExecStart=/usr/bin/dockerd|ExecStart=/usr/bin/dockerd --registry-mirror=<你的加速地址>|g" /etc/systemd/system/docker.service
sudo systemctl daemon-reload
sudo service docker restart
```
方式二：修改配置

```shell
sudo cp -n /lib/systemd/system/docker.service /etc/systemd/system/docker.service
vim /etc/systemd/system/docker.service
```
修改：

```vim
ExecStart=/usr/bin/dockerd --registry-mirror=你的加速地址
```
重启生效
```shell
 systemctl daemon-reload
service docker restart
```
