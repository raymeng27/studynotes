# Linux

## 一、了解Linux

### 1.linux的内核版本

Linux的内核版本编号类似如下：

```bash
3.10.0-123.e17.x86-64
#主板本.次版本.发布版本.修改版本
```

根据版本的发展历程，内核版本的定义有点不大相同。在2.6.X版本之前，托瓦兹将内核版本分为两类：

* 主、次版本为奇数：开发中版本。
* 主、次版本为偶数：稳定版本。

不过，这种奇、偶的编号格式在3.0版本之后就不再使用了。从3.0开始，内核主要依据主线版本来开发，开发完毕后会往下一个主线版本进行。而旧的版本在新的主线版本出现之后，会有两种机制来处理。一种机制为结束开发（End of Live，EOL），亦即为该程序代码已经结束，不会有继续维护的状态。另外一种机制为保持该版本的持续维护，亦即为长期维护版本（Longterm）。可以使用“uname -r”来查看内核版本，如下：

```shell
#uname -r
3.10.0-514.26.2.el7.x86_64
```

### 2.各硬件设备在Linux中的文件名

|        设备         |                    设备在Linux中的文件名                     |
| :-----------------: | :----------------------------------------------------------: |
| SATA、USB磁盘驱动器 |                         /dev/sd[a-p]                         |
|         U盘         |                         /dev/sd[a-p]                         |
|     Virtio接口      |                   /dev/vd[a-p]用于虚拟机内                   |
|     软盘驱动器      |                         /dev/fd[0-7]                         |
|       打印机        |   /dev/lp[0-2]（25针打印机）<br />/dev/usb/lp[0-15]USB接口   |
|        鼠标         |                /dev/input/mouse[0-15]（通用）                |
|   CD-ROM、DVD-ROM   | /dev/scd[0-1]（通用）<br />/dev/sr[0-1]（CentOS常见）<br />/dev/cdrom（当前CD-ROM） |
|       磁带机        |     /dev/ht0（IDE接口）<br />/dev/st0（SATA /SCSI接口）      |
|    IDE磁盘驱动器    |                         /dev/hd[a-d]                         |

## 二.Linux文件权限

### 1.用户与用户组

Linux文件的拥有者有三种概念：用户，用户组，其他人。

在Linux系统中，默认的情况下，所有的系统上的账号与一般用户的相关信息，都记录在/etc/passwd文件内。置于个人的密码则是记录在/etc/shadow文件内。Linux所有的组中都记录在/etc/group中。

### 2.Linux文件权限概念

在命令行输入ls -al得到以下界面：

```bash
[root@meng ~]# ls -al
total 52
dr-xr-x---.  5 root root 4096 May  4 13:51 .
dr-xr-xr-x. 18 root root 4096 Apr 30 11:26 ..
-rw-------   1 root root  272 May  4 13:40 .bash_history
-rw-r--r--.  1 root root   18 Dec 29  2013 .bash_logout
-rw-r--r--.  1 root root  176 Dec 29  2013 .bash_profile
-rw-r--r--.  1 root root  176 Dec 29  2013 .bashrc
drwx------   3 root root 4096 Aug 17  2017 .cache
-rw-r--r--.  1 root root  100 Dec 29  2013 .cshrc
drwxr-xr-x   2 root root 4096 Aug 17  2017 .pip
-rw-r--r--   1 root root   64 Aug 17  2017 .pydistutils.cfg
-rw-r--r--   1 root root   16 May  4 14:04 show.txt
drwx------   2 root root 4096 Apr 30 11:26 .ssh
-rw-r--r--.  1 root root  129 Dec 29  2013 .tcshrc
-rw-------   1 root root 3056 May  4 13:50 .viminfo
```

可以将每个文件的信息分为七个部分，如下：

-rw-r--r--   1 root root   16 May  4 14:04 show.txt

* ①-rw-r--r--：
  * 第一个字符代表这个文件的类型与权限：[d]则是目录，[-]则是普通文件，[l]表示链接文件，[b]表示为设备文件里面的可供存储的周边设备，[c]表示为设备文件里面的串行端口设备。
  * 接下来的字符中，以三个为一组，第一组为文件拥有者可具备的权限，第二组为加入此用户组账号的权限，第三组为其它人对本文件的权限。
* 1：表示有多少文件名链接到此节点，每个文件都会将它的权限与属性记录到文件系统的inode中，不过，我们使用的目录树却是使用文件名来记录，因此每个文件就会链接到一个inode，这个属性记录的就是有多少不同的文件名链接到相同的一个inode号码。
* root：第一个root表示这个文件的拥有者。
* root：第二个root表示整文件的所属用户组。
* 16：表示这个文件的容量大小，默认单位为Bytes。
* May 4 14:04：表示这个文件的创建日期或者是最近修改日期。
* show.txt：此文件的文件名。

root不受系统权限的限制，所有无论文件权限是什么，默认root都可以读写。

常见的几个修改文件属性与权限的命令：chgrp，修改文件所属用户组；chown，修改文件拥有者；chmod，修改文件的权限。

* chgrp：修改文件的所属用户

  命令：chgrp [OPTION]... GROUP FILE...

  例：

  ```bash
  [root@meng ~]# chgrp meng show.txt 
  ```

* chown：修改文件拥有者

  命令：chown [OPTION]... [OWNER]\[:[GROUP]] FILE...

  例：

  ```bash
  [root@meng ~]# chown -R meng:root show.txt 
  #-R：表示递归
  ```

* chmod：修改文件权限

  命令：chmod [OPTION]... MODE[,MODE]... FILE...

  Each MODE is of the form '[ugoa]*([-+=]([rwxXst]*|[ugo]))+|\[-+=][0-7]+'.

  文件权限修改的命令是chmod，但是，权限的设置方法有两种，分别可以使用数字或是符号来进行权限的修改。

  * 数字类型修改文件权限，Linux文件的基本权限有9个，分别是拥有者，所属组，其他人，三种身份各有自己的读、写、执行权限，这九个权限三个三个一组，可以用数来来代表各个权限。r：4，w：2，x：1。每种身份各自的三个权限数字累加，例如当权限为rwxrwx---，则该文件的权限数字就是770。因此，数字修改文件权限的方式可以为：

    ```bash
    [root@meng ~]# chmod 777 show.txt
    ```

  * 符号修改文件权限，u，g，o代表用户，所属组，其他人，a代表所有人，他们的读写权限r,w,x可以用+，-，=来设置，如下：

    ```shell
    [root@meng ~]# chmod u+x,g+r,o+x show.txt 
    ```

    或者

    ```shell
    [root@meng ~]# chmod u=rwx,g=rx,o=rx show.txt 
    ```

## 三.Linux文件与目录管理

### 1.目录的相关操作：

```javascript
.	代表当前目录
..	代表上一层目录
-	代表前一个工作目录
~	代表目前使用者身份所在的家目录
```

常见的处理目录的命令：cd，切换目录；pwd，显示当前目录；mkdir，建立一个新目录；rmdir，删除一个空目录。

* cd（change directory）切换目录

  命令：cd [-L|[-P [-e]]] [dir]

  如：cd /ect

* pwd（print working directory）显示目前所在目录的命令

  命令：pwd [-LP]

  如：pwd 

* mkdir（make directory）创建目录

  命令：mkdir [OPTION]... DIRECTORY...

  Create the DIRECTORY(ies), if they do not already exist。-P选项可以连同没有存在的父母录一起创建。如下：

  ```shell
  #mkdir /temp/meng
  ```

* rmdir（remove dorectory）删除空的目录

  命令：rmdir [OPTION]... DIRECTORY...

  Remove the DIRECTORY(ies), if they are empty。-P选项可以连同没有存在的父母录一起创建。如下：

  ```shell
  #rmdir -p /temp/meng
  ```

### 2.文件与目录

* 文件查看：ls

  命令：ls [OPTION]... [FILE]...

* 文件复制：cp

  命令：cp [OPTION]... SOURCE... DIRECTORY

* 删除：rm

  命令：rm [OPTION]... FILE...

  -r , -R：递归删除

* 文件/目录移动：mv

  命令：mv [OPTION]... SOURCE... DIRECTORY

### 3.文件内容查看

* 直接查看内容：cat

  命令：cat [OPTION]... [FILE]...

  如：

  ```shell
  #cat show.txt
  ```

* 反向查看：tac

  命令：tac [OPTION]... [FILE]...

* 可翻页查看：more（一页一页翻动）

  命令：more [options] file...

  在more这个程序的运行过程中，有几个按键可以使用：

  * 空格键：向下翻一页
  * Enter：向下翻一行
  * /字符串：在这个显示的内容中，向下查找字符串这个关键词。
  * :f：立刻显示出文件名和目前显示的行数。
  * q：退出
  * b或者[ctrl]-b：往回翻一页，这个操作只对文件有用。

* less（一页一页翻动）
  命令：less[options] file...

  less的用法比more更加有弹性，可以使用的按键：

  * 空格键：向下翻一页
  * [pagedown]：向下翻一页
  * [pageup]：向上翻一页
  * /字符串：向下查找字符串
  * ?字符串：向上查找字符串
  * n：重复前一个查找
  * N：反向的查找
  * g：前进到第一行
  * G：前进到最后一行
  * q：退出

### 4.命令与文件查找

#### 1.which，命令查找

which命令是根据$PATH这个环境变量所规范的路径，去查找执行文件的文件名，所以，是找出执行文件而已，且which后面接的是完整文件名。若加上-a，则列出所有可以找到的同名执行文件。

命令：which [options] [--] COMMAND [...]

#### 2.文件查找

文件查找的命令主要有：find、whereis、locate

* whereis

  whereis速度会比find快的多，whereis主要针对/bin/sbin下面的执行文件，以及/usr/share/man下面的man page文件，跟几个比较特定的目录来处理而已，所以速度当然快。

  命令：whereis [options] file

* locate

  locate的使用更简单，直接在后面输入文件的部分名称后，就能够得到结果。locate有使用的限制，locate来寻找数据特别快，这是因为locate寻找的数据时由已建立的数据库/var/lib/mlocate里面的数据所查找的，所以不用直接再读取硬盘当中的数据，所以特别的快。locate使用有限制是因为它是经由数据库来查找的，而数据库的建立默认是在每天执行一次，所以当新建起来的文件，却还在数据库更新之前查找该文件，locate遍查不到，手动更新数据库直接输入updatedb命令就可以。

* find

  find是很强大的命令，但是所用时间很多，因为find是直接查找硬盘。

### 3.文件压缩

在Linux的环境中，压缩文件的拓展名大多是：\*.tar，\*.tar.gz，\*.gz......Linux支持的压缩命令非常多，且不同的命令所用的压缩技术并不相同，当然彼此之间可能就无法互通压缩/解压缩文件。Linux常见的压缩文件拓展名如下：

| 文件拓展名 |                  软件                  |
| :--------: | :------------------------------------: |
|    *.Z     |         compress程序压缩的文件         |
|   *.zip    |           zip程序压缩的文件            |
|    *.gz    |           gzip程序压缩的文件           |
|   *.bz2    |          bzip2程序压缩的文件           |
|    *.xz    |            xz程序压缩的文件            |
|   *.tar    |    tar程序打包的文件，并没有压缩过     |
|  *.tar.gz  | tar程序打包的文件，并且经过gzip的压缩  |
| *.tar.bz2  | tar程序打包的文件，并且经过bzip2的压缩 |
|  *.tar.xz  |  tar程序打包的文件，并且经过xz的压缩   |

#### 1.gzip

gzip可以说是应用最广的压缩命令了，目前gzip可以解开compress、zip和gzip等软件所压缩的文件，gzip所建立的压缩文件为*.gz。

命令：gzip [OPTION]... [FILE]...

当使用gzip进行压缩时，在默认的状态下原本的文件会被压缩成.gz后缀的文件，源文件就不再存在了。gzip这个压缩命令主要想要用来替换compress。

#### 2.bzip2

若是gzip是为了替换comperss并提供更好的压缩比而成立的，那么bzip2则是为了替换gzip并提供更佳的压缩比而来。bzip2选项跟gzip一模一样，只是拓展名由.gz变成.bz2而已。

#### 3.xz

xz是比bzip2还要高压缩比的软件，用法和gzip/bzip2一样。

#### 4.打包命令tar

Linux大多压缩命令仅能针对单一文件来进行压缩，虽然gzip，bzip2，xz也能够针对目录进行压缩，不过，这两个命令对目录的压缩指的是将目录内的所有文件分别进行压缩的操作。tar可以将多个目录或文件打包成一个大文件，同时还可以通过gzip，bzip2，xz的支持，将该文件同时进行压缩。

命令：tar [OPTION...] [FILE]...

tar的选项和参数非常多，简单介绍如下几个：

* -c：建立打包文件，压缩的时候使用。
* -t：查看打包文件的内容包含哪些文件名。
* -x：解包或解压缩的功能，可以搭配-C在特定目录解压。
* -z：通过gzip的支持进行压缩/解压缩，此时文件名最好为*.tar.gz。
* -j：通过bzip2的支持进行压缩/解压缩，此时文件名最好为*.tar.bz2。
* -J：通过xz的支持进行压缩/解压缩，此时的文件名最好为*.tar.xz。
* -v：在压缩/解压缩的过程中，将正在处理的文件名显示出来。
* -f filename：-f后面要立刻要被处理的文件名。
* -C 目录：这个选项要在解压缩，将文件解压到指定的目录。