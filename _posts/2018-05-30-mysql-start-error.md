mysql the control process exited with error code
Could not open unix socket lock file

service mysqld stop

# 查看配置文件路径
[root@vm11 ~]# mysqld --verbose --help | grep -A 1 'Default options'
Default options are read from the following files in the given order:
/etc/my.cnf /etc/mysql/my.cnf /usr/etc/my.cnf ~/.my.cnf

或者：
[root@vm11 ~]# whereis my.ini
my: /etc/my.cnf

# 查看错误日志
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

tail -n 200 -f /var/log/mysqld.log

chown mysql:mysql mysql.sock
chown mysql:mysql mysql.sock.lock

