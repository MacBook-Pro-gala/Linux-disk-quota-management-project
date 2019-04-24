# Linux磁盘配额管理项目
[TOC]
***
>**ssh tools**：Termius
>
>**Virtual Machine**：vmware 14
>**Markdown**：印象笔记 & Typora

### Linux Set up the network card when boot
1. Enter the terminal
```
cd /etc/sysconfig/network-scripts
```
2. Vi opens the corresponding network card configuration file. Centos defaults to ens33 and most other systems to enth0
```
vi ifcfg-ens33
```
3. Change ONBOOT=no     to     ONBOOT=yes  Wq save

4. Restart the system, you will find that the network card has been booted with boot

### Configure system disk 60GB+ storage disk 10GB
1. Perform ` ` ` fdisk -l ` ` ` view new add hard disk, you can see the SDB for the newly added the hard disk.

2. ```fdisk /dev/sdb```Format new hard disk

3. 输入m可以查看帮助

4. 输入n新建分区，输入p新建主分区，输入1（主分区号），分别磁盘分区的起始终止位置，这里采用默认，即分区为硬盘大小。

5. 输入w对分区进行保存

6. ```fdisk -l ```查看分区，有了sdb1 ;
7. 输入```mkfs.ext4```将分区格式化为ext4格式
8. 编辑/etc/fstab ，在最下面添加：
/dev/sdb1              /app                     ext4    defaults        0 0
使得磁盘开机挂载到/app目录

### 修改内核fstab，对根目录开启磁盘配额限制
```
vim /etc/fstab 
```
![image](1.jpg)

标出来的地方就是需要新增的地方，这个表示是对根目录进行磁盘配额限制，当然，也可以加在其他行，则是对其他的目录进行磁盘配额限制。

### 查看目录挂载位置
```
df -h
```
![image](2.jpg)

### 重新挂载根目录分区，内核重新读取/etc/fstab文件
```
mount -o remount /app
mount | grep quota
```
![image](3.jpg)

### 安装quota
```
yum install quota
```

### 通过quotacheck命令在根目录下生成quota配置文件
```
quotacheck -cugm /dev/sdb1
ll / | grep quota
```
![image](4.jpg)

### 启动磁盘配额
```
quotaon /dev/sdb1
```

### 在/app/data创建用户
```
mkdir thenewuser #记得更改文件权限
cp .bash_logout  .bash_profil  .bashrc /app/data/thenewuser
useradd -m -d /app/data/thenewuser thenewuser
```

### 对用户开启
```
edquota -u thenewuser
```

### 模拟大文件写入
```
dd if=/dev/zero of=/app/data/file1 bs=1M count=450  #未超上限，无报错
dd if=/dev/zero of=/app/data/file2 bs=1M count=450  #报错
```


### 查看磁盘使用情况
```
repquota /dev/sdb1
```
![image](5.jpg)

### 脚本正文
```

#!/bin/bash
#userlist=cat /etc/passwd | awk 'BEGIN {FS=":"}{print $1}'
TEXT1=`cat "/etc/passwd"`
USERNUMBER=`wc -l /etc/passwd|cut -d' ' -f1`
echo $USERNUMBER
USERALLLIST=`echo "$TEXT1" | awk 'BEGIN {FS=":"}{print $1}'`

echo $USERALLLIST
echo "******************************************************************************"
for((i=1;i<=$USERNUMBER;i++));do
    echo "这是第 $i 个用户";
    FORUSER=`cut -d: -f1 /etc/passwd | head -$i | tail -1`
    echo $FORUSER
    #CHECKFULLUSER=`repquota /dev/sdb1 | grep $FORUSER | tr -s ' ' |cut -d ' ' -f1`
    CHECKFULLUSED=`repquota /dev/sdb1 | grep $FORUSER | tr -s ' ' |cut -d ' ' -f3`
    CHECKFULLUSED=${CHECKFULLUSED:=0} #若 num 为空或未设置时，则 num 设为值 0
    #echo $CHECKFULLUSER
    echo $CHECKFULLUSED
    if [ $CHECKFULLUSED -eq 500000 ]
    then
       echo "over used"
       mv /app/data/$FORUSER/* /app/overuseddata  #移动所有滥用文件到overuseddata（不包括隐藏的配置文件）
       echo "文件已移动"
    else
       echo "used well"
    fi
done;
```