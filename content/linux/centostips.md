---
title: "CentosTips"
date: 2017-07-29 18:36
---
[TOC]

# 配置 sudo 用户.
首先使用 `useradd username` 添加新用户，然后设置用户密码 `passwd usernmae`.
然后 使用 `visudo` 自动打开 `/etc/sudoers` 配置文件，在文件加入sudo权限，配置如下：
```
tianqi05    ALL=(ALL)    ALL
‘不要密码
tianqi05    ALL=(ALL)    NOPASSWD:ALL
```
最后切换用户
sudo -iu tianqi05

# rpm删除安装
```
rpm -qal | grep 
sudo rpm -e xx.rpm
sudo rpm -ivh XX.rpm
```

系统中的awk命令到底是执行哪个可以执行文件呢？
```
$ readlink /usr/bin/awk  
/etc/alternatives/awk  ----> 其实这个还是一个符号连接  
$ readlink /etc/alternatives/awk  
/usr/bin/gawk  ----> 这个才是真正的可执行文件  
-f 选项：
-f 选项可以递归跟随给出文件名的所有符号链接以标准化，除最后一个外所有组件必须存在。
简单地说，就是一直跟随符号链接，直到直到非符号链接的文件位置，限制是最后必须存在一个非符号链接的文件。
$ readlink -f /usr/bin/awk  
/usr/bin/gawk  
```

# 修改密码出错
```bash
passwd: Authentication token manipulation error
passwd: password unchanged
```
`mount -o remount,rw /`

# screen 启动一个可执行文件
```bash
function check_service {
    local URL=$1
    local TIMEOUT=$2
    #echo "Check service $URL"
    if [ -z $TIMEOUT ]; then
        TIMEOUT=300
    fi
    if [[ $1 == https* ]]; then
        OPT='-k'
    else
        OPT=''
    fi
    if ! timeout $TIMEOUT sh -c "while ! curl $OPT -s $1 >/dev/null; do sleep 1; done"; then
        #echo "Cannot reach!"
        return 1
    else
        #echo "Success!"
        return 0
    fi
}

function quick_check_service {
    if check_service $1 2; then
        return 0
    else
        return 1
    fi
}

function screen_it {
    NL=`echo -ne '\015'`
    SESSION=$(screen -ls | awk '/[0-9].'$1'/ { print $1 }')
    if [ ! -n "$SESSION" ]; then
        screen -d -m -S $1 sh -c 'script /dev/null'
        sleep  1.5
    fi
    screen -S $1 -p 0 -X stuff "$2$NL"
}
function get_screen_pid {
    screen -ls | awk '/[0-9].'$1'/ { print $1 }' | awk -F '.' '{print $1}'
}

function start_service () {
    local URL=$1
    local SERVICE=$2
    local CMD=$3
    local NO_CHECK=$4

    if [[ $NO_CHECK = "NO_CHECK" ]]; then
	screen_it $SERVICE "$CMD"
	echo "$SERVICE started successfully"
    return
    fi

    if ! quick_check_service $URL; then
        screen_it $SERVICE "$CMD"
        sleep 1
        local MAX_TRIES=300
        local TRIES=0
        while (( $TRIES < $MAX_TRIES ))
        do
            local PID=$(pgrep -P $(pgrep -P $(get_screen_pid $SERVICE)))
            if [[ -z "$PID" ]]; then
                echo "Failed to start $SERVICE"
                exit 1
            fi

            TRIES=$(( TRIES+1 ))
            if check_service $URL 1; then
                echo "$SERVICE started successfully"
                TRIES=$(( MAX_TRIES + 1))
            else
                sleep 1
            fi
        done
        if [ $TRIES -eq $MAX_TRIES ]; then
            echo "Failed to start $SERVICE"
            exit 1
        fi
    else
        echo "$SERVICE is running"
    fi
}
```


# shell 各种判断
Shell判断
##按照文件类型进行判断
-b 判断文件是否存在，并且是否为快设备文件（是块设备文件为真）
-c 判断文件是否存在，并且是否为字符设备文件（是字符设备文件为真）
-d 判断文件是否存在，并且是否为目录文件（是目录为真）
-e 判断文件是否存在，存在为真
-f 判断文件是否存在，并且是否为普通文件（存在为真）
-L 判断文件是否存在，并且是否为符号链接文件（是符号链接文件为真）
-p 判断文件是否存在，并且是否为管道文件（是管道文件为真）
-s 判断文件是否存在，并且是否为空（非空为真）
-S 判断文件是否存在，并且是否为套接字文件（是套接字文件为真）

##按照文件权限进行判断
-r 判断文档是否有读权限
-w 判断是否有写权限
-x 判断是否可执行

##两个文件之间的比较
文件1 -nt 文件2 判断文件1的修改时间是否比文件2的新（如果新为真）
文件1 -ot 文件2 判断文件1的修改时间是否比文件2的旧（如果旧为真）
文件1 -ef 文件2 判断文件1是否和文件2的inode号一致，可以理解为两个文件是否为同一个文件，这个判断是判断硬链接的最好方法

##两个整数之间的比较
-eq 判断两个数值是否相等
-ne 判断两个数值是否不相等
-gt 判断是否大于
-lt 判断是否小于
-ge 判断是否大于等于
-le 判断是否小于等于

##字符串之间的判断
-z 判断字符串是否为空
-n 判断字符串是否为非空
字符串1 == 2 判断字符串1是否和字符串2相等 
字符串1 != 2 判断字符串1是否和字符串2不相等

##多重条件判断
判断1 -a 判断2 逻辑与，判断1和判断2都成立，最终结果为真
判断1 -o 判断2 逻辑或，判断1和判断2有一个成立，结果为真
！判断 逻辑非  使原始的判断式取反

## 判断条件
 1. Linux的shell中的测试命令，用于测试某种条件或某几种条件是否真实存在
 2. 测试条件为真，返回一个0值；为假，返回一个非0整数值

测试命令有两种方式，一种test expression（表达式）；另一种命令格式[ expression ]

其中”[“是启动测试命令，”]”要与之配对，而且”[“和”]”前后的空格必不可少

此方式常作为流程控制语句的判断条件

### 2.1 字符串判断 
  
str1 = str2　　　　　　当两个串有相同内容、长度时为真 
str1 != str2　　　　　  当串str1和str2不等时为真 
-n str1　　　　　　　 当串的长度大于0时为真(串非空) 
-z str1　　　　　　　 当串的长度为0时为真(空串)

这个地方有必要举个小例子，我们编程的时候经常做一些使用喜欢使用空格表示空

但shell中空格会被判断成一个字符,比如：

[  -n  ” ” ]  这个值echo $?会返回0，说明字符串不为空。
[  -z  ”  ” ] 这个值echo $?会返回非空，说明里边不是空。
  
### 2.2 数字的判断 
  
int1 -eq int2　　　　两数相等为真 
int1 -ne int2　　　　两数不等为真 
int1 -gt int2　　　　int1大于int2为真 
int1 -ge int2　　　   int1大于等于int2为真 
int1 -lt int2　　　　 int1小于int2为真 
int1 -le int2　　　    int1小于等于int2为真 
  
### 2.3 文件的判断 
  
-e file                                                    若文件存在，则为真 
-d file                                                    若文件存在且是一个目录，则为真
-b file                                                    若文件存在且是一个块特殊文件，则为真
-c file            若文件存在且是一个字符特殊文件，则为真
-f file            若文件存在且是一个规则文件，则为真
-g file            若文件存在且设置了SGID位的值，则为真
-h file            若文件存在且为一个符合链接，则为真
-k file            若文件存在且设置了“sticky”位的值
-p file            若文件存在且为一已命名管道，则为真
-r file            若文件存在且可读，则为真
-s file            若文件存在且其大小大于零，则为真
-u file            若文件存在且设置了SUID位，则为真
-w file            若文件存在且可写，则为真
-x file            若文件存在且可执行，则为真
-o file            若文件存在且被有效用户ID所拥有，则为真 
  
### 2.4 逻辑判断 
  
!expr                                                      若expr为假则复合表达式为真，expr可以是任何有效的测试表达式
expr1 -a expr2         若expr1和expr2都为真则整式为真
expr1 -o expr2         若expr1和expr2有一个为真则整式为真  
  
补充： 系统变量 
  
$n                 该变量与脚本被激活时所带的参数相对应

                  .n是正整数，与参数位置相对应($1,$2…) 
$?                                                                    前一个命令执行后的退出状态
$#                 提供脚本的参数号
$*                 所有这些参数都被双引号引住。若一个脚本接收两个参数，$*等于$1$2 
$0                 正在被执行命令的名字。对于shell脚本而言，这是被激活命令的路径
$@                 所有这些参数都分别被双引号引住。若一个脚本接收到两个参数，$@等价于$1$2
$$                 当前shell的进程号。对于shell脚本，这是其正在执行时的进程ID
$!                 前一个后台命令的进程号

### 退出状态
1. Linux系统，每当命令执行完成后，系统返回一个退出状态。若退出状态值为0，表示命令运行成功；反之若退出状态值不为0，则表示命令运行失败。最后一次执行命令的退出状态值被保存在内置变量”$?”中。

2. exit命令格式：exit status（status在0～255之间），返回该状态值时伴随脚本的退出，参数被保存在shell变量$?中。

