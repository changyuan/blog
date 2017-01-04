---
title: shell 相关
date: 2016-11-09 16:23:13
updated: 2016-11-09 16:23:13
tags:
categories:
---
记录一些用shell编写的小脚本，等等。
<!-- more -->

### 用shell脚本，实现多台服务器代码同步更新的东东（rsync）

```bash
# 同步多台服务器文件代码
#!/bin/bash
if [ $1 == "f" ]
then
	rsync -av ./ --exclude-from='frontend_rsync_exclude.txt' changyuan@172.21.0.86:/home/changyuan/frontend/
elif [ $1 == "b" ]
then
	rsync -av ./ --exclude-from='backend_rsync_exclude.txt' changyuan@172.21.0.86:/home/changyuan/backend/
elif [ $1 == "a" ]
then
	rsync -av ./ --exclude-from='api_rsync_exclude.txt' changyuan@172.21.0.86:/home/changyuan/api/
else
	rsync -av ./ --exclude-from='frontend_rsync_exclude.txt' changyuan@172.21.0.86:/home/changyuan/frontend/
	rsync -av ./ --exclude-from='backend_rsync_exclude.txt' changyuan@172.21.0.86:/home/changyuan/backend/
	rsync -av ./ --exclude-from='api_rsync_exclude.txt' changyuan@172.21.0.86:/home/changyuan/api/
fi

# 其中.txt 文件是一些不需要的同步的目录文件
```

rsync 是一个远程数据同步工具，可通过LAN,WAN快速同步多台主机间的文件。Rsync使用所谓的“Rsync算法”来使本地和远程两个主机之间的文件达到同步，这个算法只传送两个文件的不同部分，而不是每次都整份传送，因此速度相当快。`rsync --help` 查看帮助，其中常使用的参数如下，有两种实现方式：
>- -a, --archive 归档模式，表示以递归方式传输文件，并保持所有文件属性，等于-rlptgoD
>- -v, --verbose 详细模式输出
>- -P 等同于 --partial
>- --progress 显示备份过程
>- -z, --compress 对备份的文件在传输时进行压缩处理
>- --exclude=PATTERN 指定排除不需要传输的文件模式
>- --include=PATTERN 指定不排除而需要传输的文件模式
>- --exclude-from=FILE 排除FILE中指定模式的文件，在一个文件中指定的目录或文件
>- --include-from=FILE 不排除FILE指定模式匹配的文件 必须包含的,在一个文件中指定的目录或文件
>- --delete 删除那些DST中SRC没有的文件
>- --delete-excluded 同样删除接收端那些被该选项指定排除的文件
>- --delete-after 传输结束以后再删除

#### SSH方式(:)
```bash
	service sshd start
	rsync -vzrtopg --progress -e ssh --delete work@172.16.78.192:/www/* /databack/experiment/rsync

	#如何通过rsync只复制目录结构，忽略掉文件呢?
	rsync -av --include '*/' --exclude '*' source-dir dest-dir
```

#### 后台服务方式(::)
##### 启动rsync服务
`vim /etc/xinetd.d/rsync` , 将其中的`disable=yes`改为`disable=no`,并重启`xinetd`服务
##### 创建配置文件
`vi /etc/rsyncd.conf`
```bash
	uid=root
	gid=root
	max connections=4
	log file=/var/log/rsyncd.log
	pid file=/var/run/rsyncd.pid
	lock file=/var/run/rsyncd.lock
	secrets file=/etc/rsyncd.passwd
	hosts deny=172.16.78.0/22

	[www]
	comment= backup web
	path=/www
	read only = no
	exclude=test
	auth users=work
```
##### 创建密码文件
```bash
echo "work:abc123" > /etc/rsyncd.passwd
chmod 600 /etc/rsyncd.passwd
```
##### 备份,恢复
```bash
	rsync -avz --progress --delete work@172.16.78.192::www /databack/experiment/rsync
	rsync -avz --progress /databack/experiment/rsync/ work@172.16.78.192::www
```

[更多关于rsync参考](https://rsync.samba.org/) , [官方给出的样例](https://rsync.samba.org/examples.html)


### crontab 相关参数

```bash
 .---------------- minute (0 - 59)
 |  .------------- hour (0 - 23)
 |  |  .---------- day of month (1 - 31)
 |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
 |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
 |  |  |  |  |
 *  *  *  *  * user-name command to be executed

eg：
0 3 * * * bash /home/demo.sh >> /home/sh.log  # 三点开始执行
8 12 * * 3 bash /data1/auto.sh  # 每周3的12点8分开始执行
0 21 * * 2 scp /data1/{kid_kid_ask_set,kid_period_ask_set} root@192.168.2.3:/data/ #每周二晚上21点执行
10 14 * * * bash /home/updataRedis.sh >>/home/log/log.txt 2>&1 # 14:10 开始
*/1 * * * * curl http://www.xxx.com/test.php  #每分钟开始执行

```
### 创建文件夹

```bash
#!/bin/bash 
cd /home/changyuan/test/
str=`date '+%Y%m%d%H%M'`

if [ -d $str ] 
then
	echo "exist floder"
else
	mkdir $str
fi
```
### shell 基础
```bash
touch online.sh

#!/bin/bash
echo "Hello World !"

#变成可执行文件
chmod +x ./online.sh

#或者直接
/bin/sh online.sh

#变量使用
for file in `ls /etc`

your_name="qinjx"
echo $your_name
echo ${your_name}

#只读变量
myUrl="http://www.w3cschool.cc"
readonly myUrl

#删除变量
unset variable_name



#字符串
echo "\"It is a test\"" # 转义
echo -e "OK! \n" # 转义换行
echo '$name'  # 单引号为输出
string="runoob is a great site"
echo ${#string} #提取子字符串长度
echo ${string:1:4} # 输出 unoo

echo $string $string1 #连接字符串

echo `expr index "$string" is` # 查找is位置expr


#数组
array_name=(value0 value1 value2 value3) #定义
array_name[n]=valuen

${array_name[1]} #取值
${array_name[@]} , ${array_name[*]}  #全部

${#array_name[0]} #单个长度
${#array_name[@]} 或者 ${#array_name[*]} #长度

# 参数传递
$0,$1,$2... # 分别为第一个，第二个参数等等
$#	 #为传递到脚本的参数个数
$* , $@   #所有向脚本传递的参数,只有放到双引号中，前者相当于一个参数"1 2 3"，后台还是多个(1,2,3)
$$  #脚本运行的当前进程ID号
$!  #后台运行的最后一个进程的ID号

# 运算符 放到 `` 符号中
val=`expr 2 + 2`

# 布尔运算符  ! 非 -o 或运算(or) -a 与运算(and)
# 逻辑运算符 && || 
# 字符串运算符
=:   #两个字符串是否相等
!=:  #两个字符串是否相等
-z:  #字符串长度是否为0，为0返回 true(zero) [ -z $a ]
-n:  #字符串长度是否为0，不为0返回 true(no zero) 
str: #[ $a ]

#文件测试运算符
-b,-c,-d,-f,-g,-k,-p,-u,-r,-w,-x,-s,-e
#其中-c：是字符设备文件，-d目录，-f文件，-r只读，-w可写，-x可执行，-s文件大小是否>0,不为空返回true，-e（exist）文件是否存在


# 屏幕读取变量
read name
echo "$name It is a test"

printf # 格式化输出

#test 检查某个条件是否成立
# 流程控制
if test $[num1] -eq $[num2]
then
    echo '两个数相等！'
else
    echo '两个数不相等！'
fi

if condition1
then
    command1
elif condition2 
then 
    command2
else
    commandN
fi


for var in item1 item2 ... itemN
do
    command1
    command2
    ...
done


#!/bin/sh
int=1
while(( $int<=5 ))
do
        echo $int
        let "int++"
done

until condition
do
    command
done

case var in
	1)
    command1
    ;;
	2）
    command2
    ;;
esac

break
continue

# 函数
[function] demoFun(){
    echo "demo"
    return ($#,$*); # 返回所有参数个数和全部参数
}

#Shell 输入/输出重定向

cat test.txt > test1.txt
cat test.txt >> test1.txt

#如果希望执行某个命令，但又不希望在屏幕上显示输出结果，那么可以将输出重定向到 /dev/null：
command > /dev/null #会起到"禁止输出"的效果

# 如果希望屏蔽 stdout 和 stderr,0 是标准输入（STDIN）,1 是标准输出（STDOUT）,2 是标准错误输出（STDERR）
command > /dev/null 2>&1  


# 文件包含
. ./test1.sh 或者
source ./test1.sh

```