# KVM虚拟机扩容
## 扩容流程
通过virsh list --all可以看到新clone的虚拟机名称  
![在这里插入图片描述](https://img-blog.csdnimg.cn/89a850945ea045cf9cab678965219584.png)

查看磁盘所在位置  
virsh domblklist 虚拟机名称  
![在这里插入图片描述](https://img-blog.csdnimg.cn/6d76c97e35044ca2b64443f00435cebf.png)

查看虚拟机磁盘文件的大小  
qemu-img info 上述hda  
![在这里插入图片描述](https://img-blog.csdnimg.cn/10e2604e674b419d9052facf2b674ed6.png)

扩容之前先关闭虚拟机（后面所有的命令都在虚拟机关闭的状态下运行），关闭 kvm 虚拟机准备克隆    
这边如果不关闭 kvm 虚拟机则直接克隆会报错，例如：ERROR Domain with devices to clone must be paused or shutoff.  
virsh destroy 虚拟机名称  
如果不放心的话，可以先用如下命令clone一个虚拟机：   
virt-clone -o 需要克隆的虚拟机名称 -n 新的虚拟机名称 -f 新虚拟机磁盘空间的位置及名称   
virt-clone -o lion -n mouse -f /home/hxb/image/mouse.disk  
[克隆虚拟机流程](https://blog.csdn.net/wanglei_storage/article/details/51106096)
```
Options（一些基本的选项）：
--version：查看版本
-h，--help：查看帮助信息
--connect=URI：连接到虚拟机管理程序 libvirt 的URI
General Option（一般选项）：
-o ORIGINAL_GUEST, --original=ORIGINAL_GUEST：原来的虚拟机名称
-n NEW_NAME, --name=NEW_NAME：新的虚拟机名称
--auto-clone：从原来的虚拟机配置自动生成克隆名称和存储路径。
-u NEW_UUID, --uuid=NEW_UUID：克隆虚拟机的新的UUID，默认值是一个随机生成的UUID
Storage Configuration（存储配置）：
-f NEW_DISKFILE, --file=NEW_DISKFILE：指定新的虚拟机磁盘文件
--force-copy=TARGET：强制复制设备
--nonsparse：不使用稀疏文件复制磁盘映像

Networking Configuration:（网络配置）
-m NEW_MAC, --mac=NEW_MAC：设置一个新的mac地址，默认是一个随机的mac
```

实例：  
-f后面的参数可以直接写为test.qcow2即在当前目录下新建一个。完成后通过virsh list --all可以看到新clone的虚拟机。  

移动磁盘位置  
mv /var/lib/libvirt/images/ubuntu20.04.qcow2 /hdd/libvirt/images/  
给磁盘扩容  
qemu-img resize /hdd/libvirt/images/ubuntu20.04.qcow2 +75G  
qemu-img info /hdd/libvirt/images/ubuntu20.04.qcow2  

修改虚拟机ubuntu20.04的磁盘位置  

2.增加cpu和内存：  
这些修改需要在该虚拟的配置文件中进行。  
①配置文件目录：在目录/etc/libvirt/qemu下有一个虚拟机配置的xml文件，名称为：虚拟机名.xml。  
可以直接编辑配置文件，也可以通过命令virsh edit 虚拟机名称来编辑：  
在文件中有以下三行，修改前两个可以增加内存大小，修改第三个可以增加cpu。  
```
  <memory unit='KiB'>1048576</memory>
  <currentMemory unit='KiB'>1048576</currentMemory>
  <vcpu placement='static'>1</vcpu>
```
完成之后保存即可。  
②启动虚拟机需要从配置文件启动，命令如下：  
virsh create /etc/libvirt/qemu/配置文件名称  
3.最后检查配置是否生效：  
①查看虚拟机配置信息：  
virsh dominfo 虚拟名称  

## LVM方式扩容


另一篇文章  
1、查找虚拟机磁盘目录  
要添加新磁盘，首先得找到原有磁盘的目录$ virsh list --all    ##查看虚拟机列表  
$ virsh shutdown Name    ##关闭名称为Name的虚拟机  
$ virsh edit Name    ##Name为虚拟机名称  
找到source file行，后面的就是磁盘的存放目录  


克隆  
virsh list --all  
virsh shutdown generic-2-clone79  
 <source file='/home/vm/f24-clone-clone-clone.qcow2'/>  
virt-clone -o  kvm_client00 -n kvm_client01 -f /home/vm/f24-clone-from79.qcow2  
扩容  
扩容之前查看qemu-img info   
qemu-img resize  /home/vm/f24-clone-clone-clone.qcow2 +100G  
qemu-img info  /home/vm/f24-clone-clone-clone.qcow2  


