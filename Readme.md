# OTUS ДЗ Сбор и анализ логов (Centos 7)
-----------------------------------------------------------------------
### Домашнее задание

Настраиваем центральный сервер для сбора логов
Цель: В результате выполнения ДЗ студент настроит центральный сервер для сбора логов.
в вагранте поднимаем 2 машины web и log
на web поднимаем nginx
на log настраиваем центральный лог сервер на любой системе на выбор
- journald
- rsyslog
- elk
настраиваем аудит следящий за изменением конфигов нжинкса

все критичные логи с web должны собираться и локально и удаленно
все логи с nginx должны уходить на удаленный сервер (локально только критичные)
логи аудита должны также уходить на удаленную систему


* развернуть еще машину elk
и таким образом настроить 2 центральных лог системы elk И какую либо еще
в elk должны уходить только логи нжинкса
во вторую систему все остальное
Критерии оценки: 4 - если присылают только логи скриншоты без вагранта
5 - за полную настройку
6 - если выполнено задание со звездочкой 

#### Выполнение ДЗ

**1. На веб-сервере WEB настроено:**
- nginx отправляет логи ошибок и логи доступа на сервер хранения логов (log:192.168.10.23), логи ошибок также сохраняются локально в ```/var/log/nginx/error.log``` (настраивается в конфигурационном файле ```/etc/nginx/nginx.conf```):
```
access_log syslog:server=192.168.10.23:514,tag=nginx_access main;
error_log syslog:server=192.168.10.23:514,tag=nginx_error notice;
error_log /var/log/nginx/error.log crit;
```
- настроен rsyslog, пересылающий все события на сервер хранения логов (log:192.168.10.23)
- настроен аудит конфигурационного файла nginx.conf с помощью audit, данные отправляются на сервер хранения логов (log:192.168.10.23)


**2. На сервере хранения логов настроено:**
- настроен rsyslog для приема логов с веб-сервера и сохранения их в папку ```/mnt/logging/192.168.10.22```:
```
[vagrant@log ~]$ sudo ls -l /mnt/logging/192.168.10.22/
total 52
-rw-------. 1 root root   73 Apr 27 00:01 CROND.log
-rw-------. 1 root root  477 Apr 27 00:13 anacron.log
-rw-------. 1 root root   62 Apr 26 23:34 audisp-remote.log
-rw-------. 1 root root  186 Apr 26 23:39 nginx.log
-rw-------. 1 root root  119 Apr 26 23:40 nginx_access.log
-rw-------. 1 root root  133 Apr 26 23:47 ntpd.log
-rw-------. 1 root root  473 Apr 26 23:39 polkitd.log
-rw-------. 1 root root  434 Apr 27 00:13 run-parts(.log
-rw-------. 1 root root 2120 Apr 27 00:15 sshd.log
-rw-------. 1 root root 3545 Apr 27 00:05 sudo.log
-rw-------. 1 root root  543 Apr 27 00:15 systemd-logind.log
-rw-------. 1 root root 1437 Apr 27 00:15 systemd.log
-rw-------. 1 root root   67 Apr 26 23:38 yum.log
```

- настроен audit для приема логов с веб-сервера, просмотр принятых логов с веб-сервера с помощью команды ```ausearch -ts today -i | grep nginx```:
```
[root@log vagrant]# ausearch -ts today -i | grep nginx 
node=web type=PROCTITLE msg=audit(04/26/20 23:38:24.148:1707) : proctitle=nano /etc/nginx/nginx.conf 
node=web type=PATH msg=audit(04/26/20 23:38:24.148:1707) : item=1 name=/etc/nginx/nginx.conf inode=67149945 dev=08:01 mode=file,644 ouid=root ogid=root rdev=00:00 obj=unconfined_u:object_r:user_tmp_t:s0 objtype=NORMAL cap_fp=none cap_fi=none cap_fe=0 cap_fver=0 
node=web type=PATH msg=audit(04/26/20 23:38:24.148:1707) : item=0 name=/etc/nginx/ inode=100724682 dev=08:01 mode=dir,755 ouid=root ogid=root rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 objtype=PARENT cap_fp=none cap_fi=none cap_fe=0 cap_fver=0 
node=web type=PROCTITLE msg=audit(04/26/20 23:39:18.320:1708) : proctitle=nano /etc/nginx/nginx.conf 
node=web type=PATH msg=audit(04/26/20 23:39:18.320:1708) : item=1 name=/etc/nginx/nginx.conf inode=67149945 dev=08:01 mode=file,644 ouid=root ogid=root rdev=00:00 obj=unconfined_u:object_r:user_tmp_t:s0 objtype=NORMAL cap_fp=none cap_fi=none cap_fe=0 cap_fver=0 
node=web type=PATH msg=audit(04/26/20 23:39:18.320:1708) : item=0 name=/etc/nginx/ inode=100724682 dev=08:01 mode=dir,755 ouid=root ogid=root rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 objtype=PARENT cap_fp=none cap_fi=none cap_fe=0 cap_fver=0 
node=web type=USER_CMD msg=audit(04/26/20 23:39:29.920:1710) : pid=7240 uid=vagrant auid=vagrant ses=5 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='cwd=/home/vagrant cmd=nano /etc/nginx/nginx.conf terminal=pts/0 res=success' 
node=web type=PROCTITLE msg=audit(04/26/20 23:39:29.927:1713) : proctitle=nano /etc/nginx/nginx.conf 
node=web type=PATH msg=audit(04/26/20 23:39:29.927:1713) : item=1 name=/etc/nginx/nginx.conf inode=67149945 dev=08:01 mode=file,644 ouid=root ogid=root rdev=00:00 obj=unconfined_u:object_r:user_tmp_t:s0 objtype=NORMAL cap_fp=none cap_fi=none cap_fe=0 cap_fver=0 
node=web type=PATH msg=audit(04/26/20 23:39:29.927:1713) : item=0 name=/etc/nginx/ inode=100724682 dev=08:01 mode=dir,755 ouid=root ogid=root rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 objtype=PARENT cap_fp=none cap_fi=none cap_fe=0 cap_fver=0 
node=web type=PROCTITLE msg=audit(04/26/20 23:39:38.128:1714) : proctitle=nano /etc/nginx/nginx.conf 
node=web type=PATH msg=audit(04/26/20 23:39:38.128:1714) : item=1 name=/etc/nginx/nginx.conf inode=67149945 dev=08:01 mode=file,644 ouid=root ogid=root rdev=00:00 obj=unconfined_u:object_r:user_tmp_t:s0 objtype=NORMAL cap_fp=none cap_fi=none cap_fe=0 cap_fver=0 
node=web type=PATH msg=audit(04/26/20 23:39:38.128:1714) : item=0 name=/etc/nginx/ inode=100724682 dev=08:01 mode=dir,755 ouid=root ogid=root rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 objtype=PARENT cap_fp=none cap_fi=none cap_fe=0 cap_fver=0 
node=web type=USER_CMD msg=audit(04/26/20 23:39:57.795:1718) : pid=7243 uid=vagrant auid=vagrant ses=5 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='cwd=/home/vagrant cmd=systemctl restart nginx terminal=pts/0 res=success' 
node=web type=SERVICE_START msg=audit(04/26/20 23:39:57.983:1721) : pid=1 uid=root auid=unset ses=unset subj=system_u:system_r:init_t:s0 msg='unit=nginx comm=systemd exe=/usr/lib/systemd/systemd hostname=? addr=? terminal=? res=success' 
node=web type=SERVICE_STOP msg=audit(04/26/20 23:39:57.983:1722) : pid=1 uid=root auid=unset ses=unset subj=system_u:system_r:init_t:s0 msg='unit=nginx comm=systemd exe=/usr/lib/systemd/systemd hostname=? addr=? terminal=? res=success' 
node=web type=SERVICE_START msg=audit(04/26/20 23:39:58.050:1723) : pid=1 uid=root auid=unset ses=unset subj=system_u:system_r:init_t:s0 msg='unit=nginx comm=systemd exe=/usr/lib/systemd/systemd hostname=? addr=? terminal=? res=success' 
node=web type=USER_CMD msg=audit(04/26/20 23:40:01.162:1727) : pid=7265 uid=vagrant auid=vagrant ses=5 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='cwd=/home/vagrant cmd=nano /etc/nginx/nginx.conf terminal=pts/0 res=success' 
node=web type=PROCTITLE msg=audit(04/26/20 23:40:01.170:1730) : proctitle=nano /etc/nginx/nginx.conf 
node=web type=PATH msg=audit(04/26/20 23:40:01.170:1730) : item=1 name=/etc/nginx/nginx.conf inode=67149945 dev=08:01 mode=file,644 ouid=root ogid=root rdev=00:00 obj=unconfined_u:object_r:user_tmp_t:s0 objtype=NORMAL cap_fp=none cap_fi=none cap_fe=0 cap_fver=0 
node=web type=PATH msg=audit(04/26/20 23:40:01.170:1730) : item=0 name=/etc/nginx/ inode=100724682 dev=08:01 mode=dir,755 ouid=root ogid=root rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 objtype=PARENT cap_fp=none cap_fi=none cap_fe=0 cap_fver=0 
type=USER_CMD msg=audit(04/26/20 23:40:56.960:1630) : pid=6838 uid=vagrant auid=vagrant ses=6 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='cwd=/home/vagrant cmd=less /mnt/logging/192.168.10.22/nginx.log terminal=pts/0 res=success' 
type=USER_CMD msg=audit(04/26/20 23:41:04.738:1636) : pid=6844 uid=vagrant auid=vagrant ses=6 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='cwd=/home/vagrant cmd=less /mnt/logging/192.168.10.22/nginx_access.log terminal=pts/0 res=success' 
```

**3. На сервере отображения логов настроено:**
- rsyslog - собирает данные с сервера web nginx логи (192.168.10.22)
- filebeat - настроен только модуль nginx (```/etc/filebeat/modules.d/nginx.yml```). Данный компонент "родной" для ELK стэка. На этом конкретном примере filebeat парсит файлы статистики nginx, который получает и первично обрабатывает rsyslog.
- elasticsearch - NOSQL движок, который хранит в себе данных, получаемые из различных источников, в нашем случае данные приходят с filebeat
- kibana - графический интерфейс для отображения данных из elasticsearch


**Скриншоты:**
![image](https://raw.githubusercontent.com/staybox/master/otus_dz13/screenshots/elk-stats.png)

![image](https://raw.githubusercontent.com/staybox/master/otus_dz13/screenshots/log-elk.png)

#### Как запустить

```git clone git@github.com:staybox/otus_dz13.git && cd otus_dz13 && vagrant up```


**Итого: Задание выполнено**



