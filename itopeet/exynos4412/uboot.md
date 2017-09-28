### 把uboot、linux内核和根文件系统烧写到SD卡 
http://blog.chinaunix.net/uid-8048969-id-4366895.html
#### 一、 配置minicom 
```
#minicom -s 
+-----[configuration]-- ------+ 
| Filenames and paths      | 
| File transfer protocols  | 
| Serial port setup        | 
| Modem and dialing        | 
| Screen and keyboard      | 
| Save setup as dfl        | 
| Save setup as..          | 
| Exit                     | 
| Exit from Minicom        | 
+-------------------------------------+ 
```
选择Serial port setup 
设置参数如下： 
```
    +--------------------------------------------------------------------------------------+ 
    | A -    Serial Device      : /dev/ttyS0                                | 
    | B - Lockfile Location     : /var/lock                                | 
    | C -   Callin Program      :                                          | 
    | D -  Callout Program      :                                          | 
    | E -    Bps/Par/Bits       : 115200 8N1                                | 
    | F - Hardware Flow Control : No                                        | 
    | G - Software Flow Control : No                                        | 
    |                                                                      | 
    |    Change which setting?                                              | 
    +--------------------------------------------------------------------------------------+ 
```
选择Save setup as dfl设置保存为默认 
```
+-----[configuration]---------+                                     
| Filenames and paths      |                                     
| File transfer protocols  |                                     
| Serial port setup        |                                     
| Modem and dialing        |                                     
| Screen and keyboard      | 
| Save setup as dfl        | 
| Save setup as..          | 
| Exit                     | 
| Exit from Minicom        | 
+-------------------------------------+ 
```

#### 二、配置tftp 
```
#setup 选择系统服务，把tftp选项打* 
#vim /etc/xinetd.d/tftp 
disable = no 
#service xinetd restart 
```
把zImage拷贝到/tftpboot下


#### 三、给SD卡分区
- 1.把SD卡插入PC机
- 2.卸载SD卡
```
fdisk -l 查看SD卡对应的设备
umount /dev/sdb1 卸载掉SD卡的分区
```
- 3.删除SD卡原有分区并建立新的分区
fdisk /dev/sdb  给SD卡创建分区表
```
zhiyongli@LZY:~ fdisk /dev/sdb 
You will not be able to write the partition table. 


Command (m for help): m  查看帮助
Command action 
  a   toggle a bootable flag 
  b   edit bsd disklabel 
  c   toggle the dos compatibility flag 
    d   delete a partition 
    l   list known partition types 
    m   print this menu 
    n   add a new partition 
    o   create a new empty DOS partition table 
    p   print the partition table 
    q   quit without saving changes 
    s   create a new empty Sun disklabel 
    t   change a partition's system id 
    u   change display/entry units 
    v   verify the partition table 
    w   write table to disk and exit 
    x   extra functionality (experts only) 


Command (m for help): p  打印分区表
Disk /dev/sdb: 1917 MB, 1917845504 bytes 
2 heads, 1 sectors/track, 1872896 cylinders, total 3745792 sectors 
Units = sectors of 1 * 512 = 512 bytes 
Sector size (logical/physical): 512 bytes / 512 bytes 
I/O size (minimum/optimal): 512 bytes / 512 bytes 
Disk identifier: 0x00000000 


    Device Boot      Start         End      Blocks   Id  System 
/dev/sdb1             149     3745791     1872821+   6  FAT16 


Command (m for help): d 删除原有分区
Selected partition 1 


Command (m for help):p
Disk /dev/sdb: 1917 MB, 1917845504 bytes 
2 heads, 1 sectors/track, 1872896 cylinders, total 3745792 sectors 
Units = sectors of 1 * 512 = 512 bytes 
Sector size (logical/physical): 512 bytes / 512 bytes 
I/O size (minimum/optimal): 512 bytes / 512 bytes 
Disk identifier: 0x00000000 


    Device Boot      Start         End      Blocks   Id  System 


Command (m for help): n 新建分区

Command action 
    e   extended 
    p   primary partition (1-4) 
p 
Partition number (1-4, default 1): 1 
First sector (2048-3745791, default 2048): 
Using default value 2048 
Last sector, +sectors or +size{K,M,G} (2048-3745791, default 3745791): +1G         


Command (m for help): n 
Command action 
    e   extended 
    p   primary partition (1-4) 
p 
Partition number (1-4, default 2): 2 
First sector (2099200-3745791, default 2099200): 
Using default value 2099200 
Last sector, +sectors or +size{K,M,G} (2099200-3745791, default 3745791): 
Using default value 3745791 


Command (m for help): p 


Disk /dev/sdb: 1917 MB, 1917845504 bytes 
2 heads, 1 sectors/track, 1872896 cylinders, total 3745792 sectors 
Units = sectors of 1 * 512 = 512 bytes 
Sector size (logical/physical): 512 bytes / 512 bytes 
I/O size (minimum/optimal): 512 bytes / 512 bytes 
Disk identifier: 0x00000000 


   Device Boot      Start         End      Blocks   Id  System 
/dev/sdb1            2048     2099199     1048576   83  Linux 
/dev/sdb2         2099200     3745791      823296   83  Linux 


Command (m for help):w 写入分区表
The partition table has been altered! 
Command (m for help)：q 退出
```
4.挂载分区2并写入跟文件系统
```
root@LZY:~# mkfs.ext3 /dev/sdb2
root@LZY:~# mount /dev/sdb2 /mnt/
root@LZY:/home/zhiyongli/smdk6410_lzy/rootfs# tar -xvf rootfs.tar -C /mnt/
root@LZY:~# umont /dev/sdb2
```

#### 三、烧写sd卡u-boot 
主机： 
```
#./write_sd  /dev/sdb u-boot-movi.bin 
```

#### 四、配置主机IP和开发板IP 
开发板： 
```
[u-boot-sd]# set serverip 192.168.1.10
[u-boot-sd]# set ipaddr 192.168.1.20
[u-boot-sd]# save
```

#### 五、下载并烧写内核到SD卡
```
[u-boot-sd]# tftp 50000000 zImage
[u-boot-sd]# movi write kernel 50000000
```

#### 六、设置自动启动和挂载跟文件系统
```
[u-boot-sd]# set bootargs “noinitrd root=179:2 rw console=ttySAC0,115200”
[u-boot-sd]# set bootcmd “movi read kernel 50008000;bootm 50008000”
[u-boot-sd]# save
[u-boot-sd]# reset
```

#### 七、启动根文件后, 使用nfs 挂载主机上的工作目录
```
mount -t nfs -o intr,nolock 192.168.1.10:/nfsroot /mnt
```
