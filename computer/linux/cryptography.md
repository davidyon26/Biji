# 虚拟盘加密
本文介绍了如何创建一个虚拟加密盘（同时也应该适用于普通的真实硬盘），然后使用LVM工具管理这个加密盘
## 所需工具
本节列出了所需要使用的工具，这些工具在debian系统中都应该可以使用包管理工具从官方的标准软件库中获得，debian的包管理请参考[deiban包管理](./Debian包管理.md)
* cryptsetup
* update-initramfs （如果需要在boot过程中自动mount加密盘）
* dd
* pv
* mkfs.ext4
* blkid
## 步骤
### 1. 准备盘
这里的盘可以是普通文件，某个虚拟机的虚拟盘（如果Linux系统安装在虚拟机中），硬盘或者硬盘的某个分区。为了增强安全性，首先要使要将加密的盘充满随机数据，使得盘上的数据无章可循。否则攻击者可能通过某些模式，如全0判断数据为空等等。

创建一个1G的文件磁盘
```
$ dd if=/dev/urandom of=/path/myEncrptedDisk bs=1M count=1024
```
如果我们要将某个硬盘或者分区加密，使用下面的命令，其中`/dev/sdb`是要加密的硬盘；如果是分区，文件的名称应该是/dev/sdb1，就是后面应该有个数字之类的。可以使用`sudo fdisk -l`命令查看硬盘或者分区所对应的文件
```
$ dd if=/dev/urandom of=/dev/sdb bs=1M 
```
如果不太关心安全性，可以使用`if=/dev/zero`，这样速度会很快；相反如果你特别在意安全性，可以使用`if=/dev/random`，这样速度最慢。

另外使用pv命令，利用管道可以查看磁盘初始化的速度，如下：
```
$ dd if=/dev/urandom iflag=fullblock | pV | dd of=./myEncryptDisk bs=1M count=1024 iflag=fullblock
```
### 2. 创建LUKS格式的加密容器
接着我们需要使用`cryptsetup`命令在上面的初始化的盘中创建加密容器（container）。注意：Are you sure？回答的yes要大写YES。使用`man cryptsetup`命令查看该命令更多的选项。如果是硬盘或者分区，将`./myEncryptDisk`换成相应的设备文件，通常是`/dev/sdb`或者`/dev/sdb1`
```
$ sudo cryptsetup -y luksFormat --key-size=512 ./myEncryptDisk
[sudo] password for david: 

WARNING!
========
This will overwrite data on ./myEncryptDisk irrevocably.

Are you sure? (Type uppercase yes): YES
Enter passphrase: 
Verify passphrase: 
```
利用file命令`file myEncryptDisk`可以查看文件myEncryptDisk是加密文件。另外，一定要牢记密码，否则加密盘中的数据就无法恢复了。
```
$ file myEncryptDisk 
myEncryptDisk: LUKS encrypted file, ver 1 [aes, xts-plain64, sha1] UUID: 40ed48b6-d461-4d1f-a42b-c3fd621b1b3a
```
### 3. 创建加密盘的映射文件
加密盘创建以后，可以使用`cryptsetup`命令打开这个加密文件，并将其映射为一个dev文件
```
$ sudo cryptsetup luksOpen ./myEncryptDisk disk_crypt
```
可以查看在`/dev/mapper`下面增加了一个`disk_crypt`，可以使用`cryptsetup`命令查看加密盘的属性
```
$ ls -l /dev/mapper/disk_crypt 
lrwxrwxrwx 1 root root 7 May 23 21:05 /dev/mapper/disk_crypt -> ../dm-7
$ sudo cryptsetup -v status disk_crypt 
/dev/mapper/disk_crypt is active.
  type:    LUKS1
  cipher:  aes-xts-plain64
  keysize: 512 bits
  device:  /dev/loop0
  loop:    /home/david/Documents/myEncryptDisk
  offset:  4096 sectors
  size:    2093056 sectors
  mode:    read/write
Command successful.
```
### 4. 直接管理
到这个步骤，可以有两个选择，直接在这个加密盘上创建文件系统，然后`mount`后直接使用。另一种是在其基础上创建逻辑卷，使用`lvm`工具进行管理。如果想使用`lvm`进行管理可以跳过这一章，直接进入下一章。
#### 4.1 创建文件系统
在虚拟盘创建一个系统，我们选择ext4文件系统
```
$ sudo mkfs.ext4 /dev/mapper/disk_crypt 
mke2fs 1.42.12 (29-Aug-2014)
Creating filesystem with 261632 4k blocks and 65408 inodes
Filesystem UUID: 64b33007-9961-44c7-b66a-e8e9fb9caf23
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done
```
#### 4.2 加载文件系统
创建好文件系统后，就可以使用`mount`加载新建的文件系统，如下：
```
$ sudo mkdir /mnt/disk1
$ sudo mount /dev/mapper/disk_crypt /mnt/disk1
$ df -h
Filesystem                Size  Used Avail Use% Mounted on
/dev/dm-0                 9.1G  5.3G  3.4G  62% /
udev                       10M     0   10M   0% /dev
tmpfs                     605M  8.5M  596M   2% /run
tmpfs                     1.5G  216K  1.5G   1% /dev/shm
tmpfs                     5.0M  4.0K  5.0M   1% /run/lock
tmpfs                     1.5G     0  1.5G   0% /sys/fs/cgroup
/dev/mapper/vg_data-tmp   453M  2.4M  423M   1% /tmp
/dev/mapper/vg_data-var   2.3G  734M  1.4G  35% /var
/dev/mapper/vg_data-home   13G  4.5G  7.3G  39% /home
tmpfs                     303M  8.0K  303M   1% /run/user/118
tmpfs                     303M   24K  302M   1% /run/user/1000
/dev/mapper/disk_crypt    990M  1.3M  922M   1% /mnt/disk1
$ cd /mnt/disk1
$ ls
lost+found
```
现在你可以在这个目录下创建任何文件了
如果想卸载加密盘，简单，使用下面的命令
```
$ sudo umount /mnt/disk1
$ sudo cryptsetup luksClose disk_crypt 
```
至此已经完成所有的工作，如果希望在系统启动时自动加载加密盘，还需要参考下面的章节。
### 5. LVM管理加密盘
#### 5.1 创建物理卷
#### 5.2 加入LVM虚拟组
#### 5.3 扩充已有LVM逻辑卷
### 6. 启动时装载加密盘
如果希望启动的时候有系统自动加载加密盘，需要进行如下的工作。
1. 首先使用`blkid`命令获得加密盘的UUID，当然需要你已经使用`cryptsetup`建立了加密盘的`dev`的映射
>```
>$ sudo cryptsetup luksOpen ./myEncryptDisk disk_crypt
>Enter passphrase for ./myEncryptDisk: 
>$ sudo blkid /dev/mapper/disk_crypt
>/dev/mapper/disk_crypt: UUID="64b33007-9961-44c7-b66a-e8e9fb9caf23" TYPE="ext4"
> ```
2. 使用你喜好的文本编辑器增加如下的内容到`/etc/crypttab`文件，最方便使用`sudo vi /deb/crypttab`命令
>```
>disk_crypt	/home/john/myEncryptDisk	none 	luks
>```
>含义如下：
>
>| **target name** |    **source device**     | **key file** | **options** |
>|      :---:      |           :---:          |     :---:    |    :---:    |
>|disk_crypt       | /home/john/myEncryptDisk | none         | luks        |
>  
>请确保`none`是小写的，`None`虽然在debian下工作，但是在Ubuntu下会报`keyfile not found`
>如果虚拟盘不是文件而是硬盘或者分区，`source device`可以使用这个设备对应的UUID，UUID请参考步骤\1
3. 使用下面命令更新初始ramdisk
```
$ sudo update-initramfs -u -k all
```
然后可以使用如下命令确认是否成功
```
$ sudo cryptdisks_start disk_crypt
```
4. 最后一步我们需要编辑`/etc/fstab`文件，自动加载文件系统
```
# add the line below in /etc/fstab file
/dev/mapper/disk_crypt  /mnt/disk1  ext4  default 0   2
```
增加完成后，可以使用`sudo mount -a`命令检查是否设置成功

至此，如果一切顺利，你已经创建了一个加密盘，你可以将你认为的私密文件放到你的保险箱了，是不是很兴奋，恭喜:thumbsup:  :smile:！

## 参考文献
* [Linux LVM简明教程i] (https://linux.cn/article-3218-1.html)
* [How to Extend/Reduce LVM’s (Logical Volume Management) in Linux – Part II] (https://www.tecmint.com/extend-and-reduce-lvms-in-linux/)
