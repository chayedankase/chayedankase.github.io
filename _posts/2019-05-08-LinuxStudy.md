---
layout: post
title: linux学习
categories: [linux]
description: linux入门
keywords: linux
---



# Linux学习

## 目录

1. / : 根目录，一个系统只有一个跟目录
2. root : 管理员目录，只有root用户可以进入
3. bin : 命令目录，linux中的常用命令都放在这个目录中
4. home : 家目录，每个用户都在家目录下有一个属于自己的文件夹，一般该文件夹是以用户名来命名的
5. boot : linux系统启动的核心文件，包括一些连接文件及镜像文件。
6. proc : 虚拟目录（没理解），系统内存的映射，访问这个目录可以获取系统信息
7. srv : 服务目录，存放服务启动的时候需要提取的数据
8. sys : 系统目录，2.6以后存放的文件系统
9. tmp : 临时文件目录
10. dev : 设备管理器目录，里面可以看到cpu等
11. media :  媒体映射目录，自动识别设备，如U盘等
12. mnt : 系统提供该目录是为了让用户临时挂载别的文件系统，可以将外部存储挂载在/mnt
13. opt： 存放软件客户端的目录
14. usr/local : 软件安装的目录
15. var : 这个目录一般存放不断扩充的文件，习惯将经常修改的文件存放到这里，比如日志
16. selinux : linux的安全文件夹
17. sbin :  超级命令文件夹

## vim 基本使用

>  使用 vim 文件名 命令  会使用vim编辑器打开文件，通过vim 可以对文件进行操作
>
>  打开文件后  按  a /  i   可以进入编辑模式，即可对文件进行修改操作。 按ESC退出编辑模式
>
>  按u可以将刚才的修改退回。在普通模式下 输入：进入指令模式  在指令模式中 set nu 可以添加行号
>
>  G 到文章尾  ，gg到文章头
>
>  10  shift+g  光标跳转至10行
>
>  /内容 查找内容       n下一个匹配的     N上一个匹配的。
>
>  u 撤销上一步的更改
>
>  yy 复制行    5yy 复制5行
>
>  p粘贴操作
>
>  ：wq 写退出
>
>  :q 退出不保存
>
>  ：q!
>
>  ![微信图片_20190703152443]({{site.url}}\imgs\linux\微信图片_20190703152443.png)

## 常用指令

### 关机、重启、用户登录切换

1. 关机

   > shutdown -h now  立即关机
   >
   > shutdown -h 1    一分钟后关机
   >
   > shutdown -r now 立即重启
   >
   > halt  直接关机命令

2. 重启

   > reboot  重启命令

3. 数据同步

   > sync  将内存中的数据写入磁盘，防止数据丢失。一般在重启或者关机前都应该执行一下。

4. 用户登录、切换、退出

   > su - 用户名 命令可以进行用户切换。 高权限切换到低权限时不用输入密码，低权限切换到高权限时需要输入密码。
   >
   > exit 命令可以退回切换前的用户
   >
   > logout 命令可以退出登录

### 用户管理

1. 用户添加

   > useradd 用户名 创建一个用户，默认分组为用户名分组，并在/home/下创建一个同用户名目录作为家目录。
   >
   > useradd -d 目录 用户名 创建一个用户按照指定的目录设置家目录。

2. 指定或修改密码

   > passwd 用户名 给用户指定或者修改密码

3. 删除用户

   > userdel 用户名 删除用户，但不删除这个用户的家目录
   >
   > userdel -r 用户名 删除用户，并删除用户的家目录
   >
   > 一般在删除用户时，并不会删除用户的家目录。

4. 查询用户信息

   > id 用户名 可以查询到用户的id,组id,组名

5. 用户组添加

   > groupadd 用户组名 此命令将会添加一个用户组

6. 用户组删除

   > groupdel 用户组名 此命令将会删除一个用户

7. 修改用户组

   > usermod -g 用户组名 用户名 此命令利用用户修改命令进行用户组修改。
   >
   > usermod -d 路径 用户名 修改用户家目录	
## 文件

### /etc/passwd

> 用户的配置文件，记录用户的各种信息
>
> 每行的含义 用户名：口令（隐藏）：用户id ：组id：注释性描述：家目录：shell

![1557455441970](imgs\linux\1557455441970.png)

### /etc/group

> 用户组配置文件，记录用户组的信息
>
> 每行含义  组名：口令：组标识号：组内的用户
>
> 组内用户为隐藏显示

![1557455874350](imgs\linux\1557455874350.png)

###/etc/shadow

> 口令文件 

### /etc/profile

> 环境变量文件

## 运行级别

> 运行级别说明
>
> 0：关机    这个级别的系统会进行关机操作。
>
> 1：单用户    这个级别只有一个用户既root用户。可用于root密码找回。
>
> 2：多用户无网路  可以多个用户登录，但无网络
>
> 3：多用户有网络  多个用户登录，有网络。我们常用的级别
>
> 4：此级别暂未启用   留空的级别
>
> 5：图形界面    开启图形界面的级别
>
> 6：系统重启	开机后无限重启
>
> 常用的运行级别是 3、5，要修改默认的运行级别可以修改配置文件
>
> /etc/inittab  的  id:5:initdefault:   这行的数字
>
> 切换运行级别命令  init 数字

## 帮助命令

### man

> man [命令或配置文件] 能够获取帮助信息

### help

> help 命令   获取shell内置命令的帮助信息


## 文件目录

### pwd

> pwd 显示当前目录的绝对路径

### ls

> ls 【选项】 【目录或文件】
>
> 选项 -a 显示当前目录下所有的文件和目录，包括隐藏的
>
> 选项 -l 以列表的方式显示信息

### cd

> cd 目录跳转，最常用的一个命令

### mkdir

> mkdir 【选项】要创建的目录     作用是创建目录
>
> 选项 -p 创建多级目录

### rmdir

> rmdir  删除一个空的目录，如果目录不为空则无法删除
>
> 删除不为空的目录需要使用 rm -rf  [目录名称] 指令

### touch

> touch 文件名    在当前目录下创建一个文件

### cp 重要指令

> cp 拷贝文件到指定目录
>
> cp [选项]  【需要拷贝的文件或目录】   【拷贝到的地址】
>
> 选项 -r 可以递归复制整个文件夹
>
>  cp -r  test/  zwj/
>
> \cp 强制覆盖，不提示询问是否覆盖

### rm

> rm 删除文件或目录指令
>
> rm [选项] 【文件或目录名称】
>
> 选项 -r 递归删除
>
> ​		-f  强制删除不提示

### mv

> mv 移动文件或重命名
>
> mv   aaa.txt    bbb.txt    将aaa.txt 重命名为 bbb.txt
>
> mv  aaa.txt    /home/kase/     将aaa.txt 移动至 /home/kase 目录下

### cat

> cat 查看文件内容指令，是以只读的方式打开文件。
>
> cat [选项] 文件名
>
> 选项 -n  附带行号显示文件内容
>
> 操作技巧，cat会将文件中所有内容显示，如果文件内容过多，则对用户不够友好，通常通过 管道符加分页操作进行整理
>
> cat -n 文件名 | more        这个指令显示文件中的内容，附带行号，并进行分页。

   ### more

> more 是基于vi编辑器的文本过滤器，它以全屏幕分页的方式对文本内容进行显示。
>
> ![1344575623132](imgs\linux\微信截图_20190704111853.png)

   ### less

>  less指令也是分页查看文件，但是比more指令更强大，支持各种终端显示，它在显示内容时并不是一次加载全部内容再显示，而是显示到那加载到哪，对于显示大型文件具有较高的效率。
>
> ![1562210654595](imgs\linux\1562210654595.png)

### > 和 >> 指令

> \>指令输出重定向，将输出的内容写入到文件中，会覆盖文件原有内容。
>
> \>>指令追加输出重定向，写入到文件中时不覆盖原有内容，而是在后面追加。
>
> 使用方式： 
>
> ​		ls -l > aa.txt   将当前目录中的文件列表写入到aa.txt文件中，如果文件不存在，则创建。覆盖文件内容。

    ### echo

> echo输出内容到控制台。
>
> echo [选项] 【输出内容】
>
> echo $PATH  输出环境变量到控制台

### head

> head 用于显示文本内容开头部分内容，默认情况下head指令只显示一个文件的前10行内容，
>
> head -n 5  文件名      显示文件内容前5行。

### tail

> tail 用于显示文本末位内容，默认显示一个文件的最后10行内容
>
> tail -n  5  文件名   显示文件最后5行内容
>
> tail -100f   文件名    显示文件最后100行内容，并实时追踪文件的更新内容，长用于服务器日志查看。

### ln

> ln 软连接，也叫符号连接，类似于windows里的快捷方式，主要存放了链接其他文件的路径
>
> ln  -s  [原文件或目录]   [软连接名]    给原文件创建一个软连接

### find

> find查找指令 将从指定目录向下递归的遍历其各个子目录，将满足条件的文件或者目录显示在终端。
>
> find   [搜索范围]    【选项】
>
> 选项    -name   按照指定的文件名查找文件。
>
> ​			-user    查找属于指定用户名的所有文件。
>
> ​			-size    查找符合指定大小的文件。
>
> find   /home    -name   hello.txt   查找home目录下文件名称为hello.txt的文件。（名称可用*匹配）
>
> find /opt   -user  kase   查找opt目录下属于用户kase的文件。
>
> find /local   -size +20M   查找local目录下大于20M的文件。   大于+   等于无符号   小于  -。

###locate

> locate指令可以快速定位文件路径，但是前提需要先创建locate数据库。locate指令无需遍历整个文件系统，查询速度较快。为了保证准确度，管理员应该定期更新locate时刻。
>
> 第一次运行locate指令前，需要创建locate数据库，通过指令  updatedb。
>
> locate  hello.txt   查找hello.txt的路径。

### grep

> grep过滤查找指令，一般是通过管道符“|”对其他指令的结果进行过滤操作。
>
> grep [选项]   查找内容    原文件
>
> 选项  -n  显示匹配行及行号
>
> ​			-i  忽略字母大小写
>
> cat hello.txt | grep yes   查看hello.txt文件，在结果中查找内容yes

## 压缩和解压缩

### gzip/gunzip

> gzip  文件  将文件压缩为 *.zip   压缩的原文件将消失
>
> gunzip 文件  将文件进行解压操作

### zip/unzip

> zip 【选项】 压缩后的文件名（XXX.zip）     要压缩的文件
>
> 选项： -r 递归压缩，及压缩目录
>
> zip  -r    mm.zip   hello.txt   将hello.txt文件夹压缩为mm.zip
>
> unzip [选项]   要解压的文件名
>
> 选项： -d   指定解压后文件存放的目录
>
> unzip -d  /home/kase   mm.zip  将mm.zip解压到/home/kase 目录

### tar

> tar指令是打包指令，最后的打包文件是*.tar.gz的文件
>
> tar [选项] XXXX.tar.gz   打包内容         将指定的打包内容打包为 XXXX.tar.gz
>
> | 选项 |        功能        |
> | :--: | :----------------: |
> |  -c  | 产生.tar的打包文件 |
> |  -v  |    显示详细信息    |
> |  -f  | 指定压缩后的文件名 |
> |  -z  | 打包的同时进行压缩 |
> |  -x  |   解包.tar的文件   |
>
> tar -zcvf  a.tar.gz    hello.txt  Hello.txt    将hello.txt和Hello.txt文件打包为 a.tar.gz
>
> tar -zcvf  myhome.tar.gz  /home/   将home文件夹打包压缩为 myhome.tar.gz
>
> tar -zxvf a.tar.gz  将a.tar.gz解压到当前目录
>
> tar -zxvf  a.tar.gz  -C  /home/    将a.tar.gz 解压到home目录下

## 文件用户组管理

### 查看文件所有者 ls -ahl

> ls  -ahl    查看当前目录下的文件所有者 
>
> ![1562319503109](imgs\linux\1562319503109.png)

### 修改文件所有者 chown  用户名   文件名

> chown   用户名    文件名
>
> 将文件hello.java的所有者kase 改为 zwj
>
> 指令  chown zwj hello.java
>
> ![1562319933480](imgs\linux\1562319933480.png)

### 创建组 groupadd 组名

> groupadd 组名
>
> 创建一个组 hero，创建一个用户superman,将创建的用户放入hero组内
>
> ![1562549170452](imgs\linux\1562549170452.png)
>
> ![1562549200919](imgs\linux\1562549200919.png)

### 修改文件所在组 chgrp 组名  文件名

> chgrp 组名  文件名
>
> 将Hello.java文件的组修改为hero1。  chgrp hero1 Hello.java
>
> ![1562549957351](imgs\linux\1562549957351.png)

## 文件的权限

> 当我们使用ls -ahl 命令查看文件时，最前面的标识代表了文件的操作权限。
>
> ![1562553015589](imgs\linux\1562553015589.png)

### 权限说明

> -rw-rw-r-- 1 zwj  kase    0 Jul  3 16:21 hello.java
>
> 第一位表示文件的类型。
>
> - \- :表示普通文件
> - d:表示目录
> - l: 表示软连接
> - c:表示字符设备【鼠标、键盘等】
> - b： 块文件【硬盘】
>
> 第二到四位表示文件所有者对文件的权限（user），分别使用 - r w x来表示
>
> - -：表示无权限
> - r：表示读权限，当目标是文件时表示可读取，可查看。当目标是目录时表示可读取，可用ls查看目录内的内容
> - w：表示写权限，当目标是文件时表示可以对文件进行修改，但不代表可以删除该文件，删除一个文件的前提是对文件所在目录有w权限。当目标是目录是，表示可以在该目录内进行创建、删除、重命名。
> - x：表示可执行，对目录来说表示可以进入。
>
> 第五到七位表示文件所在组的用户对文件的权限。
>
> 第八到十位表示其他组的用户对文件的权限。

权限后的数字

> 权限后的数字，对文件来说表示硬链接的个数一般都是1，对目录来说表示目录内的子目录，子目录包含了隐藏的. 与.. 目录。

文件所有者

> -rw-rw-r-- 1 zwj  kase    0 Jul  3 16:21 hello.java 中zwj表示文件所有者

文件所在组

> -rw-rw-r-- 1 zwj  kase    0 Jul  3 16:21 hello.java  中kase表示文件所在组

文件大小

> -rw-rw-r-- 1 zwj  kase    0 Jul  3 16:21 hello.java 中0表示文件大小，目录默认为4096

最后修改时间

> 文件详细信息中的时间表示文件最后修改的时间

### 权限修改 chmod

> 通过chmod可以修改文件的权限
>
> 1. 第一种方式通过  +、-、=来进行权限修改
>
>    u:所有者 g:所有组   o：其他人   a: 所有人（u+g+o）
>
>    chmod  u = rwx,g=rx,o=x  文件目录名     将指定的文件权限修改为。。。
>
>    chmod o+w 文件目录名    给指定文件的其他人赋予写权限
>
>    chmod a-x  文件目录   给指定文件的所有人去除执行权限
>
>    ![1562558216015](imgs\linux\1562558216015.png)
>
> 2. 第二种方式，通过数字变更权限
>
>    规则： r = 4 ,w=2,x=1            rwx = 4+2+1 = 7   
>
>    chmod 751 文件目录名    表示给文件所有者设置rwx,给所在组设置rx,给其他组设置x

### 文件所有者修改 chown

> chown 用户   文件名  修改指定文件的所有者
>
> chown 用户：组    文件名      修改指定文件的所有者和所在组
>
> 选项 -R   如果文件名为目录，增加-R选项后则为递归改变该目录下所有子文件及子目录。
>
> chown -R  用户   /home/kase/    将/home/kase 目录下的所有文件所有者进行修改

### 文件所在组修改chgrp

> chgrp 组名  文件名   修改指定文件的所在组
>
> 选项-R  也代表递归修改。

## 定时任务调度

### 定时任务调度 crontab

> 定时任务调度可以让服务器自动在我们需要的时间点执行命令，比如定时备份数据库操作。
>
> crontab [选项]
>
> | 选项 | 说明                            |
> | ---- | ------------------------------- |
> | -e   | 编辑crontab定时任务             |
> | -l   | 查询crontab定时任务             |
> | -r   | 删除当前用户所有crontab定时任务 |
>
> 使用示例：每个一分钟将当前日历写入到 /tmp/mycal  文件中
>
> ​    步骤1  crontab -e   打开定时任务编辑
>
> ​	步骤2  */1  *  *  *  *  cal >> /tmp/mycal          编写语句，保存并退出
>
> 定时任务编写成功。此处* 表示对时间的指定。
>
> **时间占位符说明**	
>
> | 项目      | 含义                 | 范围                 |
> | --------- | -------------------- | -------------------- |
> | 第一个 *  | 一小时当中的第几分钟 | 0-59                 |
> | 第二个 *  | 一天当中的第几个小时 | 0-23                 |
> | 第三个 *  | 一个月中的第几天     | 1-31                 |
> | 第四个 *  | 一年中的第几个月     | 1-12                 |
> | 第五个  * | 一周当中的星期几     | 0-7（0,7都表示周日） |
>
> **特殊符号说明**
>
> | 特殊符号 | 说明                                                         |
> | -------- | ------------------------------------------------------------ |
> | *        | 代表任何时间，比如第一个*代表一小时中每分钟都执行一次。      |
> | ,        | 代表不连续的时间，比如 “0 8,12,16 * * *”表示每天的8点0分、12点0分、16点0分执行一次 |
> | -        | 代表连续的时间范围，比如“0 5 * * 1-6”表示每周的周1到周六的5点0分执行一次 |
> | */n      | 代表每隔多久执行一次，比如 “*/10 * * * *”表示每隔10分钟执行一次 |
>
> 案例：
>
> ​	45 22 * * *   每天22点45分执行
>
> ​	0   17 * *  1 每周一17点0分执行
>
> ​	0 5 1,15 * * 每月1号、15号的5点0分执行
>
> ​	40 4 * * 1-5 每周一到周五 4点40 执行
>
> ​	*/10 4 * * * 每天4点每个10分钟执行一次
>
> ​	0 0 1,15 * 1  每个月的1号、15号并且是周一的时候执行

## 磁盘分区

> 查看当前系统分区 lsblk -f
>
> 略

## 磁盘情况查询 

###  查询系统磁盘整体使用 df -lh

> df -l    查询磁盘使用情况
>
> ![1562747564352](imgs\linux\1562747564352.png)
>
> df -lh  有格式的查询磁盘使用情况
>
> ![1562747629079](imgs\linux\1562747629079.png)

### 查询指定目录磁盘占用 du -h	/目录

> du -h  /目录    查询指定目录的磁盘占用情况
>
> ![1562748424249](imgs\linux\1562748424249.png)
>
> 附带参数说明
>
> -s	指定目录占用大小汇总
>
> -h	带计量单位
>
> -a	含文件
>
> --max-depth=1 	子目录深度
>
> ![1562749979265](imgs\linux\1562749979265.png)

### 磁盘情况-实用指令

> 1.统计/home文件夹下文件的个数
>
> ![1562750744842](imgs\linux\1562750744842.png)
>
> 命令说明： “^-”表示过滤以-开头的数据，而-开头的数据是文件，此处都是文件夹，都以d开头。
>
> 
>
> 2.递归统计/home 文件夹下文件的个数（R表示递归）
>
> ![1562752557254](imgs\linux\1562752557254.png)

## 进程管理

### 显示系统进程 ps

> ps 【选项】 查看系统进程指令，一般常用ps aux
>
> ps -a 显示当前终端的所有进程信息。
>
> ps -u 以用户的格式显示进程信息
>
> ps -x 显示后台进程的参数
>
> ![](H:\笔记\imgs\linux\1562812934406.png)
>
> | USER   | PID    | %CPU    | %MEM     | VSZ          | RSS          | TTY        | STAT     | START    | TIME          | COMMEND            |
> | ------ | ------ | ------- | -------- | ------------ | ------------ | ---------- | -------- | -------- | ------------- | ------------------ |
> | 用户名 | 进程id | 占用CPU | 占用内存 | 虚拟内存使用 | 物理内存使用 | 使用的终端 | 进程状态 | 启动时间 | 占用CPU总时间 | 进程执行时的命令行 |
>
> 进程状态：S-睡眠，s-进程是会话的先导进程，N-进程拥有比普通优先级更低的优先级，R-正在运行，D-短期等待，Z-僵死进程，T-被跟踪或被停止等等
>
> ps -ef 查看进程的父进程，以全格式显示
>
> ![](H:\笔记\imgs\linux\1562813823115.png)
>
> -e 显示所有进程，-f全格式
>
> UID 用户名   PID 进程号   PPID 父进程号
>
> 父进程为0表示没有父进程

### 终止进程 kill

> kill 【选项】 进程号   通过进程号终止进程
>
> killall 进程名称     通过进程名称终止进程，支持通配符。
>
> 选项 -9   强制终止。

### 查看进程树 pstree

> pstree [选项]   更直观的查看进程信息
>
> -p 显示进程的PID
>
> -u 显示进程的所属用户

## 服务管理

> 服务service 本质就是进程，但是是运行在后台的，通常都会监听某个端口，等待其他程序的请求，比如mysql、sshd、防火墙等。

### 服务管理指令 service

> serivce  服务名 [start|stop|reload|status|restart]  
>
> 在CentOS7.0以后service指令被替换为 systemctl
>
> 案例
>
> 查看防火墙状态
>
> service iptables status
>
> ![](H:\笔记\imgs\linux\1562834480072.png)
>
> 关闭防火墙
>
> service iptables stop
>
> ![](imgs\linux\1562835703690.png)

### 查看服务名称

> 1.使用setup命令
>
> ![](imgs\linux\1562836037427.png)
>
> ![](H:\笔记\imgs\linux\1562836057990.png)
>
> 2.查看 /etc/init.d/ 目录内的服务文件
>
> ![](imgs\linux\1562836552948.png)
>
> 3.给init.d中增加新的服务，就可以对该服务设置自启动了，如将mysql添加到开机自启动
>
> 添加服务，拷贝服务脚本到init.d目录，并设置开机启动 
>
> [注意在 /usr/local/mysql 下执行]
>
> 复制启动脚本到init.d目录
>
> cp support-files/mysql.server /etc/init.d/mysql
>
> 设置脚本自启动
>
> chkconfig mysql on
>
> service mysql start --启动MySQL

### 设置服务自启动chkconfig

> chkconfig 可以给每个服务的各个运行级别设置自启动的关闭/开启
>
> chkconfig --list  查看服务的自启动情况
>
> ![1562901250135](imgs\linux\1562901250135.png)
>
> chkconfig 服务名  --list  查看指定服务的自启动情况
>
> chkconfig --level 5  服务名 on/off 	 给指定服务的运行级别5设置自启动的开关。

## 动态监控进程

> top 指令与ps指令相似，不同的是top指令会定时更新正在运行的进程显示
>
> top [选项]
>
> -d 自动更新的间隔秒数，默认3秒
>
> -i 是top不显示任何闲置或僵尸进程
>
> -p 通过指定进程id来监控指定的进程
>
> 执行top指令后，还可以输入交互指令来实时操作
>
> P 以CPU使用率来进行排序
>
> M 以内存使用率来进行排序
>
> N 以pid来进行排序
>
> q 退出top

## 系统网络查看 netstat

> netstat [选项]
>
> netstat -anp   最常用的指令
>
> -an  按照一定顺序排列输出
>
> -p  显示那个进程在调用
>
> ![1562903326183](imgs\linux\1562903326183.png)

## RPM包管理

> 一种用于互联网下载包的打包及安装工具，它包含在某些Linux分发版中，它生成具有RPM扩展名的文件。

### 简单查询指令rpm -qa

> rpm -qa 查询所有已经安装的rpm列表
>
> ![1562916669826](imgs\linux\1562916669826.png)
>
> rpm -qa|grep 软件名     配合管道符查询是否安装指定的软件
>
> ![1562916814329](imgs\linux\1562916814329.png)
>
> rpm -q 软件包名    查询是否安装指定的软件
>
> ![1562916919564](imgs\linux\1562916919564.png)
>
> rpm -qi  软件名   查询安装的指定软件信息
>
> ![1562917314390](imgs\linux\1562917314390.png)
>

## YUM

> yum是一个shell前端软件包管理器，基于RPM包管理，能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包。使用yum的前提是可以联网

### yum基本指令

> yum list |grep XXX    查询服务器是否有需要安装的软件
>
> yum install XXX 下载安装
>
> yum list installed //列出所有已安装的软件包 
>
> yum remove xxx 移除安装的软件

## shell脚本

###脚本入门

> shell脚本都是以  #!/bin/bash 来开头的，如下是一个执行后在控制台输入hello的脚本
>
> ![1563263208436](imgs\linux\1563263208436.png)
>
> 脚本写好后，记得要把脚本文件权限改为可执行文件
>
> ![1563263269017](imgs\linux\1563263269017.png)

### shell变量介绍

> shell中的变量分为 系统变量和用户自定义变量
>
> 系统变量：\$HOME   、 \$PWD、\$USER等
>
> 查看所有的系统变量   set