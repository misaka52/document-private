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

### 磁盘管理

**df**

检查文件系统的磁盘空间占用情况

- -a：列举所有文件系统，包括系统特有的/proc等问文件系统
- -h：以易读的方式展示，KB，MB，GB等
- -H：以1M=1000K代替1M=1024K
- -T：显示文件系统类型
- -i：不用硬盘容量，使用inode数量来显示

```sh
[root@VM-0-11-centos ~]# df -h
文件系统        容量  已用  可用 已用% 挂载点
devtmpfs        909M     0  909M    0% /dev
tmpfs           919M   24K  919M    1% /dev/shm
tmpfs           919M  548K  919M    1% /run
tmpfs           919M     0  919M    0% /sys/fs/cgroup
/dev/vda1        50G  2.8G   45G    6% /
tmpfs           184M     0  184M    0% /run/user/0
```

**du**

查看文件或目录的磁盘空间使用情况

- -a：列出所有文件，包括隐藏文件
- -s：列出当前目录占用内存总量，包括子目录
- -S：列出当前目录占用内存总量（不包括子目录），子目录单独展示统计

```sh
[root@VM-0-11-centos /]# du -hS /home
4.0K    /home/self/test
8.0K    /home/self
4.0K    /home
```

**fdisk**

磁盘分区表操作工具

**fsck**

检查和维护不一样的文件系统，若系统掉电或磁盘发生问题，可通过fsck对文件系统进行检查

**mount**

磁盘的挂载与卸除。umount 磁盘卸载

### VI/VIM

#### 命令模式

- i a o：进入到输入模式，以输入字符
- r：进入到取代模式
- x删除当前光标字符
- ：切换到底线命令模式
- HOME/END：跳转至行首或行尾
- Page Up/Page Down：翻页
- e：光标向后移动一个单词
- b：光标向前移动一个单词
- ctrl+f：向下移动一页。ctrl+d，移动半页
- ctrl+b：向上移动一页。ctrl+u，移动半页
- 30+下箭头：向下移动30行。30+enter：向下移动30行
- dd：删除游标对应的一行。ndd删除对应的多行
- yy：复制游标所在的一行。nyy复制多行
- u：复用上一个动作，类似ctrl+z
- ctrl+r：重做上一动作，与u相反

#### 输入模式

在命名模式下输入i进入输入模式

- i：进入输入模式
- ESC：退出输入模式，切换到命令模式
- /word：在光标下搜索关键字。n搜索下一个关键字，N搜索上一个关键字
- ?word：向上搜索
- :%s/${origin}/${replacement}/g：将所有的匹配到的origin关键字替换成对应关键字

> 特殊字符需转移，添加\。如 * . /

#### 底线命名模式

- q：退出程序。增加！标识强制退出，当修改文件后需添加
- w：保存程序

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
- :f ：显示当前文档名和当前行数
- /关键字：向下搜索关键字，n下一个，N上一个。?关键字，向上查找

## shell

### 变量

- 变量和等号之间不能有空格
- 变量命名只能用英文字母、数字、下划线，不能以数组开头

**只读变量**

readonly variable_name

```
p="abc"
readonly p
p='def'
#运行结果
/bin/sh: NAME: This variable is read only.
```

**删除变量**

unset variable_name

```sh
p="abc"
readonly p
echo $p
# 运行结果：没有输出
```

**字符串**

- 单引号：无转义功能，不能表示变量，变量只能拼接
- 双引号：有转义功能，可以表示变量，转义字符

```sh
your_name="runoob"
# 使用双引号拼接
greeting="hello, "$your_name" !"
greeting_1="hello, ${your_name} !"
echo $greeting  $greeting_1
# 使用单引号拼接
greeting_2='hello, '$your_name' !'
greeting_3='hello, ${your_name} !'
echo $greeting_2  $greeting_3
# 输出结果
hello, runoob ! hello, runoob !
hello, runoob ! hello, ${your_name} !
```

- 获取字符串长度：${#string}
- 提取子字符串：${string:a:b}，[a,b] 下标从0开始

```sh
string="123456"
echo ${string:1:2}
# 输出：23
```

echo `expr index "$str1" ab`  查询目标字符在变量中出现的个数。例中str1中字符a和b出现的次数

### 数组

**定义数组**

用空格或换行分隔，shell仅支持一维数组

```
数组名=(值1 值2 值3...)
```

**读取数组**

${数组名[n]} 读取数组指定下标元素

```sh
array=(1 '2a' 3)
# 获取第一个元素
echo ${array[0]}
# 获取所有元素
echo ${array[@]}
# 获取数组长度
echo ${#array[*]}
echo ${#array[@]}
# 获取第一个元素长度
echo ${#array[0]}
```

### 注释

单行注释：#

多行注释：EOF可以替换为其他符号，比如! '

```sh
:<<EOF
注释内容...
注释内容...
EOF
```

### 参数传递

- $@: 传递到脚本中的参数个数
- $*: 以一个单字符显示所有向脚本传递的参数。以"$1 $2 ... $n"展示
- $@:  和$*类似，会单独显示"。以"$1" "$2" ... "$n"展示
- $$: 显示当前进程id
- $!: 后台运行的最后一个进程id
- $?: 显示最后命名的退出状态，0表示没有错误
- $-: 显示Shell当前使用的选项，与set命令功能相同

### 运算符

**算数运算符**

```
a=10
b=20
val=`expr $a + $b`
echo "a + b:$val"
echo "a - b:`expr $a - $b`"
echo "a * b:`expr $a \* $b`"
echo "a / b:`expr $a / $b`"
```

**布尔运算符**

- !: 非运算符
- -o: 或运算符
- -a: 与运算符

优先级：! > -a > -o

```sh
if [ $a -gt 5 -a $b -gt 5 ]
then
        echo "$a > 5 and $b > 5"
else
        echo "other"
fi

```

**逻辑运算符**

- &&: 逻辑与
- ||: 逻辑或

```sh
if [[ $a -ge 10 && $b -ge 10 ]]
then 
        echo "$a>=10 && $b>=10"
else
        echo "other"
fi
```

**字符串运算符**

- =: 判断两个字符串是否相等
- -z: 判断字符串长度是否为0
- -n: 判断字符串长度是否不为0
- $: 判断字符串是否为空。if [ $a ]

### echo

- -e: 开启转移，\n展示为换行

```sh
# -e：开启转移。\c 不换行
echo -e "OK!\c"
echo `date`
```

### printf

```sh
printf "%-10s %-8s %-4s\n" 姓名 性别 体重kg  
```

### test

```sh
num1=100
num2=100
if test $[num1] -eq $[num2]
then
    echo '两个数相等！'
else
    echo '两个数不相等！'
fi

result=$[num1+num2]
```

### 流程控制

**if**

```sh
if condition
then
	command1
	command2
	...
elif condition2
then
	command1
	command2
	...
else
	command1
	command2
	...
fi
```

**for**

```sh
for var in item1 item2 ... itemN
do
	command1
	command2
	...
done
```

```
# 无限循环
for (( ; ; ))
```

**while**

```sh
while condition
do
    command
done
# condition为true表示无限循环 
```

**util**

> 类似do...while，先执行循环体，再判断while条件

```
until condition
do
    command
done
```

**case...esac**

```sh
case 值 in
模式1)
    command1
    command2
    ...
    commandN
    ;;
模式2）
    command1
    command2
    ...
    commandN
    ;;
esac
```

```sh
case $1 in 
1) echo "select 1"
;;
2) echo "selecct 2"
;;
*) echo "select other"
;;
esac
```

**break**：跳出循环

**continue**： 调至下一循环

### 函数

- 返回结果只能是数字
- 所有函数必须在使用前定义，解释器顺序执行

**参数**

- $#: 传递到脚本的参数个数
- $*: 传递到脚本的参数

```sh
getGrade() {
        s=$1
        res=-1
        if [ $s -lt 60 ] 
        then
                res=3
        elif [ $s -lt 80 ]
        then
                res=2
        elif [ $s -le 100 ]
        then
                res=1
        fi
        return $res     
}

getGrade 59 
echo $?
getGrade 79
echo $?
getGrade 99
echo $?
```

### 输入/输出重定向

- command > file: 将输出重定向到file中
- command >> file: 将输出以追加的形式添加到file中

标准输出文件

- 标准输入文件(stdin)：stdin的文件描述符为0
- 标准输出文件(stdout): stdout的文件描述符为1
- 标准错误文件(stderr): stderr的文件描述符为2

### 文件包含

通过. filename.sh 引入其他脚本

```sh
url="http://www.baidu.com"
```

```sh
. ./test1.sh

# 或者使用以下包含文件代码
# source ./test1.sh

echo "$url"
```

## 基本指令

### grep

- -a 获取 --test：不忽略二进制数据
- -A <显示行数>：显示匹配行后几行数据。-B：前几行。-C：前后几行
- -i：忽略字符大小写
- -n: 显示匹配行的行号
- -w: 按照单词维度完全匹配
- -x: 按照行维度完全匹配
- -v: 排查匹配行，显示其他行

### curl

http请求

curl -H [header,如"Content-Type: application/json"] -X [GET,POST] -d [参数，常用于post方法参数] [url，建议加引号，不加的话&符号后面的参数直接忽略]

### nohup(no hang up)

表示以不挂起运行的方式运行程序，即后台运行，当前session关闭也不会关闭程序

> 仅使用命令行末尾追加&的形式，session一关闭程序就停止

- /dev/null 空设备文件，黑洞文件
- < 将文件内容作为前面命令的参数
- 2>&1 将标准错误重定向到标准输出中
- & 放在命令末尾，表示后台执行，若当前session关闭则退出

### 查找文件

#### find

```sh
# 查找当前目录下.sh的文件
find . -name "*.sh"
```

#### locate

根据文件名查询文件。从数据库/var/lib/mlocate/mlocate.db 中查询，数据库每天更新一次，将历史文件保存到数据库中

```sh
locate .sh
```

#### which

在@PATH和@MANPATH环境变量下指定的路径查找文件

#### whereas

在系统默认目录下（一般指root权限安装的文件）查找二进制文件

- 使用-B指定搜索位置
- 使用-f表示列出文件的信息
- 使用-s表示只搜索路径

### awk

字段截取，按照空格或tab分割

```sh
# 截取第一个参数
awk '{print $1}'
# 按照逗号分割
awk -F , '{print $1}'
# 首先按照空格分割，再按照逗号分割
awk -F '[ ,]' '{print $1}'
# 设置变量 awk -v 变量名
awk -va '{print $1+a}'
# 过滤第一个参数大于的0的数，若不满足条件则不输出
echo "1 2 3" |awk '$1>0 {print $1}'
# 输出大于指定长度的行
awk 'length>2' t.txt
# 脚本
awk 'BEGIN {
  PI = 3.14159265
  x = -10
  y = 10
  result = atan2 (y,x) * 180 / PI;

  printf "The arc tangent for (x=%f, y=%f) is %f degrees\n", x, y, result
}'
```

### sort

排序

### uniq

对数据去重

> 一般先sort再用uniq，直接用uniq不知道为啥不能真正的去重！！！

- -c：去重后统计数量
- -d：仅显示重复行

### wc

- wc -l 统计总行数

