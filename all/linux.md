### 系统启动过程

#### 运行级别

- 0：系统停机状态，级别默认不能设置为0，否则无法正常启动
- 1：单用户工作状态，root权限，用于系统维护，禁止远程登录
- 2：多用户状态（没有NFS）
- 3：完全的多用户状态（有NFS），登录后进入命令行模式
- 4：系统预留，未使用
- 5：X11控制台，登陆后进入图形GUI模式
- 6：系统正常关闭并重启，默认不能设置6，否则无法正常启动

### 系统目录结构

#### 目录

- /bin：存放常用命令
- /boot：存放linux的核心文件，包括一些连接文件和镜像文件
- /dev：存放linux的外部设备，设备访问方式和文件访问是相同的
- /etc：存放系统管理所需要的配置文件和子目录
- /home：用户主目录，一般以用户账号命名
- /lib：存放系统基本的动态连接共享库，类似于windows的DDL文件，几乎所有的应用程序都需要用到这些共享库
- /lost+found：一般是空的。当系统非法关机时会存放一部分文件
- /media：linux系统会识别一些设备，例如U盘、光驱等，linux会把识别的设备挂载到该目录下
- /mnt：为了让用户挂载别的文件系统，可以将光驱挂载在/mnt上，便可以看到光驱的内容
- /opt：用于存放主机额外安全软件所需的目录
- /proc：process的缩写，/proc是一个伪文件系统，存储的时当前内核运行态的一些文件，该目录是一个虚拟的目录，可以通过访问这个目录来获取系统信息
- /root：超级权限这的目录
- /sbin：super bin，存放系统管原使用的系统管理程序
- /selinux：是一个安全机制，类似windows的防火墙
- /srv：存放一些服务启动之后需要提取的数据
- /usr：unix shared resources，用户共享资源
- /usr/bin：系统用户使用的应用程序
- /var：变量的缩写，系统将经常修改的文件防止该目录下，比如日志文件‘
- /tmp：存放临时文件
- /run：临时文件系统，存储系统启动依赖的信息。当系统重启时，该目录下文件将被清除

### 文件基本属性

![](../image/file-llls22.jpg)

**file type**

- d：目录
- -：文件
- l：链接文档
- b：可供存储的随机设备
- c：串行端口设备，例如键盘、鼠标

**chgrp**

更改文件数组，chgrp [-R] 属组名 文件名

-R 递归更改文件属组

**chown**

change owner：修改用户与组

**chmod**

change mode：修改用户权限

r4 w2 x1

u-用户，g-组，o-其他，a-所有

```
chmod 744 test.txt
chmod u=rwx,g=r,o=r test.txt
chmod u+x test.txt
# 减少user的x权限
chmod u-x test.txt
```

**硬链接**

磁盘中每个文件都会分配一个编号，称为索引节点

相当于一个文件对应多个名字，任意一个文件名都代表该文件。一个文件当硬链接个数为零时则文件失效，被清理

**软链接**

也称为符号链接，其内部保存目标文件地址，当目标地址失效后，软链接失效

```
[root@VM-0-11-centos self]# touch f1
# 创建硬链接
[root@VM-0-11-centos self]# ln f1 f2
# 创建软链接
[root@VM-0-11-centos self]# ln -s f1 f3
[root@VM-0-11-centos self]# ll
总用量 8
-rw-r--r-- 2 root root   0 6月   4 20:31 f1
-rw-r--r-- 2 root root   0 6月   4 20:31 f2
lrwxrwxrwx 1 root root   2 6月   4 20:31 f3 -> f1
[root@VM-0-11-centos self]# echo "hello" >> f1
[root@VM-0-11-centos self]# cat f1
hello
[root@VM-0-11-centos self]# cat f2
hello
[root@VM-0-11-centos self]# cat f3
hello
[root@VM-0-11-centos self]# rm -rf f1
[root@VM-0-11-centos self]# cat f2
hello
[root@VM-0-11-centos self]# cat f3
cat: f3: 没有那个文件或目录
```

### 文件组

```
＃ cat /etc/passwd

root:x:0:0:Superuser:/:
daemon:x:1:1:System daemons:/etc:
bin:x:2:2:Owner of system commands:/bin:
sys:x:3:3:Owner of system files:/usr/sys:
adm:x:4:4:System accounting:/usr/adm:
uucp:x:5:5:UUCP administrator:/usr/lib/uucp:
auth:x:7:21:Authentication administrator:/tcb/files/auth:
cron:x:9:16:Cron daemon:/usr/spool/cron:
listen:x:37:4:Network daemon:/usr/net/nls:
lp:x:71:18:Printer administrator:/usr/spool/lp:
ysc:x:1000:1000::/home/ysc:/bin/bash
```

```
用户名:口令:用户标识号:组标识号:注释性描述:主目录:登录Shell
口令：密文，用x代替
用户标识号：0表示root超级账户，1-99系统预留，普通用户从100开始
```

### 命令

**mkdir**

mkdir [-mp] 目录名

- -m：配置目录权限。mkdir -m 711 test
- -p：若父级目录不存在直接创建

**cp**

- -d：若来源为连接档的属性（link file），则拷贝链接档属性而非文件
- -r：递归持续复制
- -p：连通文件属性一起复制，而非使用默认属性
- -i：互动模式，询问下一步

**cat**

从文件第一行显示。tac与cat相反，从文件最后一行开始展示

- -b：列出行号，进对非空白行展示。其中-n列出行号，包括空白行

**more**

一页一页翻动

- 空白键：下一页
- enter键：下一行
- b：上一页
- :f :显示当前文档名和当前行数
- /关键字：向下搜索关键字，n下一个，N上一个。?关键字，向上查找









