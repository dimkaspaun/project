### Vagrant private_network autoconfig

Вагрант при выключении хоста и потом включении делает автоконфиг сетевых устройств eth1 (private_network)

В vagranatfil:

```
config.vm.define boxname do |box|
  
 
    box.vm.network "private_network", ip: boxconfig[:ip_addr], auto_config: false
```
добавлен auto_config: false. 

**При разворачивании стенда с нуля нужно закомментировать этот параметр, иначе для интерфейса eth1 не создадутся файлы конфигурации**

### LVM

lvdisplay
vgdisplay

pvs - 

lvs

dd if=/dev/zero of=/mnt/share/data/test.log bs=1M count=800 status=progress


parted -s /dev/sdb mklabel gpt
parted /dev/sdb mkpart primary ext4 0% 100%
mkfs.ext4 /dev/sdb1 
mount -t etx4 /dev/sdb1 /var/lib/mysql

### MariaDB

mysql -u meno -p -e "CREATE DATABASE wordpress"; 

mysql -u meno -p -e "show databases"; 

CREATE DATABASE www;

mysql_secure_installation


CREATE USER 'meno'@'%' IDENTIFIED BY 'Pa$$w0rd';
GRANT ALL PRIVILEGES ON *.* TO 'meno'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;



SELINUX

MySQL (MariaDB) из командной строки подключается к серверу, но Wordpress (скрипты PHP) нет.

Причина в политиках безопасности SELinux.

По умолчанию политика httpd_can_network_connect_db отключена (веб-сервер не может связаться с удаленной БД.)
```
getsebool -a | grep httpd_can_network_connect_db 

setsebool -P httpd_can_network_connect_db 1 
```


### iSCSI

https://www.server-world.info/en/note?os=CentOS_7&p=iscsi&f=4

https://linux-admins.ru/article.php?id_article=66&article_title=Установка%20и%20настройка%20Iscsi%20на%20CentOS7


If SELinux is enabled, change SELinux Context.
```
chcon -R -t tgtd_var_lib_t /var/lib/iscsi_disks
semanage fcontext -a -t tgtd_var_lib_t /var/lib/iscsi_disks
```


Show LUN in server
```
tgtadm --mode target --op show 
```

client
```
iscsiadm -m node
```

### Nginx

SELinux

```
setsebool -P httpd_can_network_connect_db 1 

chcon -R -t httpd_sys_content_t /usr/share/nginx/html/wordpress
```


### Zabbix 

To solve the problem zabbix server is not running you have to :

First - Check that all of the database parameters in zabbix.conf.php ( /etc/zabbix/web/zabbix.conf.php) and zabbix_server.conf ( /etc/zabbix/zabbix_server.conf) to be the same. Including:
• DBHost
• DBName
• DBUser
• DBPassword

Second- Change SElinux parameters:
```
#setsebool -P httpd_can_network_connect on
#setsebool -P httpd_can_connect_zabbix 1
#setsebool -P zabbix_can_network 1
```

Zabbix: cannot start preprocessing service: Cannot bind socket to “/var/run/zabbix/zabbix_server_preprocessing.sock”: [98] Address already in use.
Zabbix error:

 10272:20190212:003104.073 cannot start preprocessing service: Cannot bind socket to "/var/run/zabbix/zabbix_server_preprocessing.sock": [98] Address already in use.
 10239:20190212:003104.078 One child process died (PID:10272,exitcode/signal:1). Exiting …

related to:
```
# cat /var/log/audit/audit.log|grep -Ei denied|tail
type=AVC msg=audit(1549949530.062:12551): avc:  denied  { unlink } for  pid=10521 comm="zabbix_server" name="zabbix_server_preprocessing.sock" dev="tmpfs" ino=3998803 scontext=system_u:system_r:zabbix_t:s0 tcontext=system_u:object_r:zabbix_var_run_t:s0 tclass=sock_file

is solved by:

# grep AVC /var/log/audit/audit.log | audit2allow -M systemd-allow && semodule -i systemd-allow.pp

```

dump mysql database 

```
mysqldump -uzabbix -pzabbix zabbix | gzip > ~vagrant/zabbix.sql.gz
```



### MySQL Replica

https://dev.mysql.com/doc/refman/8.0/en/replication-solutions-switch.html


pass = Otus#Linux2021


```
mysql -uroot -pOtus#Linux2021

mysql> SHOW MASTER STATUS\G
mysql> SHOW SLAVE STATUS\G
```

Check replication:
```
select id,post_name,post_title from wp_posts;
```

### Slave to Master
```
> mysql
STOP SLAVE;
RESET MASTER;
CHANGE MASTER TO MASTER_HOST='192.168.10.35';
```

Change in wordpress config file (wp-config.php) DB Server = 192.168.10.35

### Master to Slave
```
>mysql
RESET MASTER;
change master to master_host='192.168.10.35', master_port=3306, master_user='repl', master_password='Otus#Linux2021', master_auto_position=2;
START SLAVE;
```
cd repl_mysql
ansible-playbook master-slave.yml