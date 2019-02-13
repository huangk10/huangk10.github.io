---
layout:     post
title:      crontab定时任务(转载)
subtitle:   
date:       2019-02-13
author:     huangk
header-img: 
catalog: true
tags:
    - linux
---



### crontab定时任务

#### 命令格式

```
crontab [-u user] file crontab [-u user] [-e|-l|-r]
```

#### 命令参数

+ -u user：用于设定某个用户的crontab服务；
+ file：file是命令文件的名字,表示将file做为crontab的任务列表文件并载入crontab。如果在命令行中没有指定这个文件，crontab命令将接受标准输入（键盘）上键入的命令，并将它们载入crontab。
+ -e：编辑某个用户的crontab文件内容。如果不指定用户，则表示编辑当前用户的crontab文件。
+ -l：显示某个用户的crontab文件内容，如果不指定用户，则表示显示当前用户的crontab文件内容。
+ -r：从/var/spool/cron目录中删除某个用户的crontab文件，如果不指定用户，则默认删除当前用户的crontab文件。
+ -i：在删除用户的crontab文件时给确认提示。

#### crontab的文件格式

```
分 时 日 月 星期 要运行的命令
```

+ 第1列分钟0~59
+ 第2列小时0~23（0表示子夜）
+ 第3列日1~31
+ 第4列月1~12
+ 第5列星期0~7（0和7表示星期天）
+ 第6列要运行的命令

```
# (put your own initials here)echo the date to the console every
# 15minutes between 6pm and 6am
# 系统将每隔15分钟向控制台输出一次当前时间
0,15,30,45 18-06 * * * /bin/echo 'date' > /dev/console
```

#### 实例

##### 每1分钟执行一次

```
* * * * * myCommand
```

##### 每小时的第3和第15分钟执行

```
3,15 * * * * myCommand
```

##### 在上午8点到11点的第3和第15分钟执行

```
3,15 8-11 * * * myCommand
```

##### 每隔两天的上午8点到11点的第3和第15分钟执行

```
3,15 8-11 */2  *  * myCommand
```

##### 每周一上午8点到11点的第3和第15分钟执行

```
3,15 8-11 * * 1 myCommand
```

##### 每晚的21:30重启smb

```
30 21 * * * /etc/init.d/smb restart
```

##### 每月1、10、22日的4 : 45重启smb

```
45 4 1,10,22 * * /etc/init.d/smb restart
```

##### 每周六、周日的1 : 10重启smb

```
10 1 * * 6,0 /etc/init.d/smb restart
```

##### 每天18 : 00至23 : 00之间每隔30分钟重启smb

```
0,30 18-23 * * * /etc/init.d/smb restart
```

##### 每星期六的晚上11 : 00 pm重启smb

```
0 23 * * 6 /etc/init.d/smb restart
```

##### 每一小时重启smb

```
* */1 * * * /etc/init.d/smb restart
```

##### 晚上11点到早上7点之间，每隔一小时重启smb

```
0 23-7 * * * /etc/init.d/smb restart
```

##### 实际项目

```
20 4 * * * /bin/sh /home/ETL_V2/etl/data_monitor/daily_check/bash_run_check
```

###### bash_run_check

```
root/anaconda3/bin/python3 /home/ETL/etl/data_monitor/daily_check/daily_check_lost/update.py -e `date -d last-day +"%Y%m%d"` -s `date -d last-day +"%Y%m%d"`
```

