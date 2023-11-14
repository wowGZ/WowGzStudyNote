# ORA-00257: 归档程序错误。只有在解析完成后才以 AS SYSDBA

最近客户项目现场的产品启动不起来了，看了启动日志之后发现出现如下问题：

> **ORA-00257: 归档程序错误。只有在解析完成后才以 AS SYSDBA** 

启动日志中大量出现上述报错日志，最终服务启动失败。

首先 sqlplus / as sysdba
执行命令

```sql
select * from v$flash_recovery_area_usage
```

发现：ARCHIVED LOG这一项的空间占用率已接近100%
执行命令

```sql
show parameter db_recover
```

发现之前设置的空间大小为41820M
如果磁盘内存充足使用如下命令调整空间上限：

```sql
alter system set db_recovery_file_dest_size=50G scope=both;
```

此处我将空间上限调整为50G。

### 如果磁盘内存空间不足？

**一、查看 日志提示一个REDO日志不能归档。**

```sh
[oracle@oel-01 ~]$ tail alert_bys001.log
ORA-19502: write error on file "", block number  (block size=)
ORA-00312: online log 3 thread 1: '/u01/app/oracle/oradata/bys001/redo03.log'
Sun Jul 21 18:01:18 2013
ARCH: Archival stopped, error occurred. Will continue retrying
ORACLE Instance bys001 - Archival Error
ORA-16014: log 3 sequence# 219 not archived, no available destinations
ORA-00312: online log 3 thread 1: '/u01/app/oracle/oradata/bys001/redo03.log'
Errors in file /u01/app/oracle/diag/rdbms/bys001/bys001/trace/bys001_arc1_6050.trc:
ORA-16014: log 3 sequence# 219 not archived, no available destinations
ORA-00312: online log 3 thread 1: '/u01/app/oracle/oradata/bys001/redo03.log'
```

**二、查看硬盘使用情况，发现ORACLE_HOME 即归档文件所在目录使用已经100%**

```sh
[oracle@oel-01 ~]$ df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/sda2              19G   18G  3.8M 100% /
/dev/sda1              99M   21M   74M  22% /boot
tmpfs                 3.0G  529M  2.5G  18% /dev/shm
/dev/sda5             4.6G  2.6G  1.9G  58% /backup
```

**三、这里我使用RMAN来删除归档日志**

```sh
[oracle@oel-01 ~]$ rman target /
RMAN> crosscheck archivelog all;
RMAN> delete expired archivelog all;
```

**==注意要用RMAN删除，遇到过通过手动从OS上rm时ORALCE不能及时检测到空闲空间的情况==**

删除今天之前的归档日志

```sh
RMAN> delete archivelog until time 'sysdate-1' ;
'sysdate-1‘代表一天前
也可以用
RMAN> delete  archivelog  all;  删除所有归档
```

**四、查看磁盘空间，已经释放出来了一部分。**

```sh
[oracle@oel-01 ~]$ df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/sda2              19G   18G  588M  97% /
/dev/sda1              99M   21M   74M  22% /boot
tmpfs                 3.0G  529M  2.5G  18% /dev/shm
/dev/sda5             4.6G  1.8G  2.6G  42% /backu
```

**五、此时使用DBA用户登陆SQLPLUS依然出错，使用SYSDBA用户登陆，切换当前日志文件。**

```sh
[oracle@oel-01 ~]$ sqlplus bys/bys
SQL*Plus: Release 11.2.0.1.0 Production on Sun Jul 21 18:03:48 2013
Copyright (c) 1982, 2009, Oracle.  All rights reserved.
ERROR:
ORA-00257: archiver error. Connect internal only, until freed
```

使用SYSDBA登陆
SYS@ bys001>alter system switch logfile; ----可能会需要较长时间。

```sh
System altered.
SYS@ bys001>select group#,status,archived from v$log;
    GROUP# STATUS           ARC
---------- ---------------- ---
         1 CURRENT          NO
         2 INACTIVE         YES
         3 ACTIVE           NO
```

日志可能如下：

```sh
[oracle@bys001 ~]$ cat alert_bys1.log
Sat Oct 05 12:44:41 2013
Suspending MMON action 'metrics monitoring' for 82800 seconds
Sat Oct 05 12:45:10 2013
Archiver process freed from errors. No longer stopped
Sat Oct 05 12:45:14 2013
Archived Log entry 106 added for thread 1 sequence 111 ID 0xebe3b9d9 dest 1:
krse_arc_driver_core: Successful archiving of previously failed ORL
Sat Oct 05 12:45:14 2013
Thread 1 advanced to log sequence 114 (LGWR switch)
  Current log# 3 seq# 114 mem# 0: /u01/oradata/bys1/redo03.log
Sat Oct 05 12:45:14 2013
AUD: Audit Commit Delay exceeded, written a copy to OS Audit Trail
Sat Oct 05 12:45:21 2013
Archived Log entry 107 added for thread 1 sequence 113 ID 0xebe3b9d9 dest 1:
Archived Log entry 108 added for thread 1 sequence 112 ID 0xebe3b9d9 dest 1:
```

**六、现在数据库恢复正常，可以登陆并操作**

```sh
SYS@ bys001>conn bys/bys
Connected.
BYS@ bys001>exit
[oracle@oel-01 ~]$ sqlplus bys/bys
BYS@ bys001>truncate table test1;
Table truncated.
最后要对归档日志进行备份和删除。
使用备份归档脚本如下：
[oracle@oel-01 ~]$ cat archback.sh
#!/bin/sh
#su - oracle
source /home/oracle/.bash_profile
##########
/u01/app/oracle/product/11.2.0/dbhome_1/bin/rman   log /home/oracle/rman-arch`date +%Y%m%d-%H%M`.log <<EOF
connect target /;
run{
backup archivelog all delete input
format '/backup/archlog/arch_%d_%T_%s';
}
exit
```

参考文档：https://blog.csdn.net/weixin_45843683/article/details/114253607
