# Домашнее задание №2

## 1. Отразить 7 последних строк из: `wtmp` `last`
```sh
[root@slonit ~]# last -n 7
root     pts/0        77.91.70.162     Fri Nov 22 23:20   still logged in
root     pts/0        77.91.70.162     Fri Nov 15 13:43 - 16:38  (02:55)
reboot   system boot  5.4.17-2136.310. Fri Nov 15 13:39   still running
root     pts/0        77.91.70.162     Fri Nov 15 13:10 - 13:39  (00:28)
reboot   system boot  5.4.17-2136.310. Fri Nov 15 13:10 - 13:39  (00:28)
root     pts/0        77.91.70.162     Fri Nov 15 13:03 - 13:07  (00:04)
reboot   system boot  5.4.17-2136.310. Fri Nov 15 13:02 - 13:07  (00:04)
```

```sh
[root@slonit ~]# last -f /var/log/wtmp -n 7
root     pts/0        77.91.70.162     Fri Nov 22 23:20   still logged in
root     pts/0        77.91.70.162     Fri Nov 15 13:43 - 16:38  (02:55)
reboot   system boot  5.4.17-2136.310. Fri Nov 15 13:39   still running
root     pts/0        77.91.70.162     Fri Nov 15 13:10 - 13:39  (00:28)
reboot   system boot  5.4.17-2136.310. Fri Nov 15 13:10 - 13:39  (00:28)
root     pts/0        77.91.70.162     Fri Nov 15 13:03 - 13:07  (00:04)
reboot   system boot  5.4.17-2136.310. Fri Nov 15 13:02 - 13:07  (00:04)
```


## 2. При помощи `logger` записать в общий журнал послание добра и мира посланникам из Альфа-Центавра с facility `kernel`.

```sh
[root@slonit ~]# logger -p kern.info "Добро и мир посланникам из Альфа-Центавра"
[root@slonit ~]# journalctl | grep "Добро и мир посланникам из Альфа-Центавра"
Nov 22 23:28:23 slonit.local root[104898]: Добро и мир посланникам из Альфа-Центавра
Nov 22 23:31:25 slonit.local root[104926]: Добро и мир посланникам из Альфа-Центавра
```

## 3. Посмотреть последние 25 строчек лога сервиса `systemd-logind`.

```sh
[root@slonit ~]# journalctl -u systemd-logind -n 25
-- Logs begin at Fri 2024-11-15 13:39:28 UTC, end at Fri 2024-11-22 23:32:26 UTC. --
Nov 15 13:39:33 slonit.local systemd[1]: Starting Login Service...
Nov 15 13:39:33 slonit.local systemd-logind[624]: New seat seat0.
Nov 15 13:39:33 slonit.local systemd-logind[624]: Watching system buttons on /dev/input/event0 (Power But>
Nov 15 13:39:33 slonit.local systemd-logind[624]: Watching system buttons on /dev/input/event1 (AT Transl>
Nov 15 13:39:33 slonit.local systemd[1]: Started Login Service.
Nov 15 13:43:06 slonit.local systemd-logind[624]: New session 1 of user root.
Nov 15 16:38:38 slonit.local systemd-logind[624]: Session 1 logged out. Waiting for processes to exit.
Nov 15 16:38:38 slonit.local systemd-logind[624]: Removed session 1.
Nov 22 23:20:41 slonit.local systemd-logind[624]: New session 3 of user root.
lines 1-10/10 (END)
```

## 4. Создать конфигурацию `syslog` для отправки сообщений уровня `info` в файл `/var/log/my.log`.

```sh
[root@slonit ~]# sudo nano /etc/rsyslog.d/mylog.conf
[root@slonit ~]# sudo systemctl restart rsyslog
[root@slonit ~]# cat /etc/rsyslog.d/mylog.conf
*.info /var/log/my.log
[root@slonit ~]# logger -p user.info "Тестовое сообщение для проверки файла /var/log/my.log"
[root@slonit ~]# tail -n 10 /var/log/my.log
Nov 22 23:36:28 slonit systemd[1]: Starting System Logging Service...
Nov 22 23:36:28 slonit rsyslogd[105013]: [origin software="rsyslogd" swVersion="8.2102.0-7.el8_6.1" x-pid="105013" x-info="https://www.rsyslog.com"] start
Nov 22 23:36:28 slonit systemd[1]: Started System Logging Service.
Nov 22 23:36:28 slonit sudo[105008]: pam_unix(sudo:session): session closed for user root
Nov 22 23:36:28 slonit rsyslogd[105013]: imjournal: journal files changed, reloading...  [v8.2102.0-7.el8_6.1 try https://www.rsyslog.com/e/0 ]
Nov 22 23:36:42 slonit sshd[105019]: Connection closed by authenticating user root 185.242.233.6 port 56658 [preauth]
Nov 22 23:37:08 slonit sshd[105022]: Invalid user  from 35.216.253.131 port 32774
Nov 22 23:37:18 slonit sshd[105022]: Connection closed by invalid user  35.216.253.131 port 32774 [preauth]
Nov 22 23:37:19 slonit sshd[105025]: Connection closed by authenticating user root 185.242.233.6 port 38488 [preauth]
Nov 22 23:37:28 slonit root[105029]: Тестовое сообщение для проверки файла /var/log/my.log
```

## 4. Все события `ssh` записывать параллельно в отдельный файл, производить ротацию каждые сутки или по размеру (не более 1 мегабайта), всего 10 файлов в ротации.

```sh
[root@slonit ~]# sudo nano /etc/rsyslog.d/ssh.conf
[root@slonit ~]# sudo systemctl restart rsyslog
[root@slonit ~]# sudo nano /etc/logrotate.d/ssh
[root@slonit ~]# cat /etc/rsyslog.d/ssh.conf
if $programname == 'sshd' then /var/log/ssh.log
& stop
[root@slonit ~]# cat /etc/logrotate.d/ssh
/var/log/ssh.log {
    daily
    rotate 10
    size 1M
    compress
    missingok
    notifempty
    create 0640 root adm
    postrotate
        systemctl restart rsyslog > /dev/null
    endscript
}
[root@slonit ~]# sudo logrotate -f /etc/logrotate.d/ssh
[root@slonit ~]# ls -lh /var/log/ssh.log*
-rw-r-----. 1 root adm     0 Nov 22 23:48 /var/log/ssh.log
-rw-------. 1 root root 1.2K Nov 22 23:48 /var/log/ssh.log.1.gz
```
## 5. Вывести сообщения с последнего запуска системы. Вывести сообщения не старше одного часа.

```sh
[root@slonit ~]# journalctl -b

[root@slonit ~]# journalctl --since "1 hour ago"
```
