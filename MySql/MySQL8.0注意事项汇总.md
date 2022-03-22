# MySQL8.0 注意事项以及解决方案

## 1. MySQL8.0 修改大小写敏感配置

天坑MySQL8.0! 在安装后, 便无法通过修改配置文件,重启服务,或者执行sql来更改数据库配置, 要想配置的话, 必须在MySQL安装完成后, 进行修改配置文件, 否则需要**删除**`/var/lib/mysql`, **如果需要保留数据的话, 记得备份!!!!**

安装活已经删除`/var/lib/mysql`后, 可以对`/etc/my.conf`进行修改, 在`[mysqld]`下添加`lower_case_table_names = 1`, 随后执行`systemctl start mysqld`或者`service mysqld restart`来启动/重启MySQL.

> 别问我为什么这么sb, 我也不知道! 总之不是初始化状态的MySQL, 要是改了配置, 就会启动不起来!!!!

## 2. 项目从MySQL5.7切换到MySQL8.0, 项目SQL报错怎么办?

好家伙! 这个也是MySQL8.0的铁锅! MySQL8.0默认开启了很多强约束, 导致我们项目中的很多SQL语句都无法执行! 咔咔咔报错简直了!!!!

可以进入mysql, 执行`set @@global.sql_mode=''`和`set @@session.sql_mode=''`临时解决这些约束问题, 不过会在下一次重启的时候变回原来的样子.

我推荐直接修改`/etc/my.conf`, 在`[mysqld]`下添加`sql_mode=`, 然后重启MySQL即可~

## 3. 我的SQL文件跑一半中断了!

可能是SQL文件太大了, 修改`my.conf`, 在`[mysqld]`下添加`max_allowed_packet=900M`, 重启MySQL

## 4. 结语

大概我目前遇到的坑就这些, 如果大家还有其它的坑, 可以留言补充! 以后遇到我也会继续补充.