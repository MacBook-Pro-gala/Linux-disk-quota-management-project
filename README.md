# Linux磁盘配额管理项目
[TOC]
***
>**ssh工具**：Termius
>**虚拟机**：vmware 14
>**Markdown编辑器**：印象笔记 & Typora

### Linux设置网卡开机启动
1. 进入终端
```
cd /etc/sysconfig/network-scripts
```
2. vi打开对应网卡配置文件，centos默认网卡为ens33，其它系统大多为enth0
```
vi ifcfg-ens33
```
3. 将ONBOOT=no改为ONBOOT=yes
  wq保存

4. 重启系统，你将发现网卡已随开机启动

### 配置系统磁盘60GB+存储盘10GB
1. 执行``` fdisk -l ```查看新添加的硬盘，可以看到 sdb为新添加的硬盘。

2. ```fdisk /dev/sdb```对新加硬盘格式化

3. 输入m可以查看帮助

4. 输入n新建分区，输入p新建主分区，输入1（主分区号），分别磁盘分区的起始终止位置，这里采用默认，即分区为硬盘大小。

5. 输入w对分区进行保存

6. ```fdisk -l ```查看分区，有了sdb1 ;
7. 输入```mkfs.ext4```将分区格式化为ext4格式
8. 编辑/etc/fstab ，在最下面添加：
  /dev/sdb1              /app                     ext4    defaults        0 0
  使得磁盘开机挂载到/app目录



### 数学公式测试
$y=\sum_{b}^{a}x^2$_