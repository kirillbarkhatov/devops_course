# Домашнее задание №1

## 1. Узнать IP-адрес интерфейса, подключенного к сети Интернет
```sh
[root@slonit ~]# ip addr show | grep 'inet' | grep -v '127.0.0.1'
```
## 2. Создать именованный пайп ( named pipe ). Вывести в файл через созданный pipe вывод команды ss -plnt
```sh
[root@slonit ~]# mkfifo my_pipe
[root@slonit ~]# ss -plnt my_pipe &
[1] 66690
[root@slonit ~]# cat my_pipe > output.txt
```

## 3. При помощи именованного пайпа заархивировать всё, что в него отправляем.
Например, содержимое файла /var/log/messages
На выходе получить архив tar или любой другой
```sh
[root@slonit ~]# cat my_pipe | tar czf messages_arch.tar.gz -C /var/log messages &
[2] 66707
[root@slonit ~]# cat /var/log/messages > my_pipe
[2]+  Done                    cat my_pipe | tar czf messages_arch.tar.gz -C /var/log messages
```

## 4. Вывести дату в unixtime
На вход команды date через пайп подать свой формат выводимой даты
```sh
[root@slonit ~]# echo "15 Nov 2024 12:00:00" | date -d "$(cat)" +%s
1731672000
```

## 5. При помощи HEREDOC записать в файл многострочное сообщение
```sh
[root@slonit ~]# cat <<EOF > text.txt
> str1
> str2
> str3
> EOF
```


## Задание повышенного уровня сложности (его выполнять не обязательно, но если сделаете, будет вашим преимуществом)
Требуется переместить ядро или переименовать его. Перезагрузить систему.
Восстановить систему двумя способами.

#### Узнаем ядро
```sh
[root@slonit ~]# hostnamectl
   Static hostname: slonit.local
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 24d3a5858c6c4165a06981dc5c0e303c
           Boot ID: c38747f38f9948b19697cd070035da8d
    Virtualization: bhyve
  Operating System: Oracle Linux Server 8.6
       CPE OS Name: cpe:/o:oracle:linux:8:6:server
            Kernel: Linux 5.4.17-2136.310.7.1.el8uek.x86_64
      Architecture: x86-64
```
#### Делаем бекап, переименовываем ядро, перезагружаемся
```sh
[root@slonit ~]# ls /boot
System.map-4.18.0-372.26.1.0.1.el8_6.x86_64              initramfs-5.4.17-2136.310.7.1.el8uek.x86_64.img
System.map-5.4.17-2136.310.7.1.el8uek.x86_64             loader
config-4.18.0-372.26.1.0.1.el8_6.x86_64                  symvers-4.18.0-372.26.1.0.1.el8_6.x86_64.gz
config-5.4.17-2136.310.7.1.el8uek.x86_64                 symvers-5.4.17-2136.310.7.1.el8uek.x86_64.gz
efi                                                      vmlinuz-0-rescue-06088db1377a47b2bcd19b4355d5886f
grub2                                                    vmlinuz-4.18.0-372.26.1.0.1.el8_6.x86_64
initramfs-0-rescue-06088db1377a47b2bcd19b4355d5886f.img  vmlinuz-5.4.17-2136.310.7.1.el8uek.x86_64
initramfs-4.18.0-372.26.1.0.1.el8_6.x86_64.img
[root@slonit ~]# cp /boot/vmlinuz-5.4.17-2136.310.7.1.el8uek.x86_64 /boot/vmlinuz-5.4.17-2136.310.7.1.el8uek.x86_64.bak
[root@slonit ~]# mv /boot/vmlinuz-5.4.17-2136.310.7.1.el8uek.x86_64 /boot/vmlinuz-5.4.17-2136.310.7.1.el8uek.custom.x86_64
[root@slonit ~]# ls /boot
System.map-4.18.0-372.26.1.0.1.el8_6.x86_64              initramfs-5.4.17-2136.310.7.1.el8uek.x86_64.img
System.map-5.4.17-2136.310.7.1.el8uek.x86_64             loader
config-4.18.0-372.26.1.0.1.el8_6.x86_64                  symvers-4.18.0-372.26.1.0.1.el8_6.x86_64.gz
config-5.4.17-2136.310.7.1.el8uek.x86_64                 symvers-5.4.17-2136.310.7.1.el8uek.x86_64.gz
efi                                                      vmlinuz-0-rescue-06088db1377a47b2bcd19b4355d5886f
grub2                                                    vmlinuz-4.18.0-372.26.1.0.1.el8_6.x86_64
initramfs-0-rescue-06088db1377a47b2bcd19b4355d5886f.img  vmlinuz-5.4.17-2136.310.7.1.el8uek.custom.x86_64
initramfs-4.18.0-372.26.1.0.1.el8_6.x86_64.img           vmlinuz-5.4.17-2136.310.7.1.el8uek.x86_64.bak
[root@slonit ~]# reboot
```

#### Жмем `e`

<img width="1191" alt="Снимок экрана 2024-11-15 в 14 59 07" src="https://github.com/user-attachments/assets/9d347187-0045-4fab-89cd-c944815267ce">

#### Указываем явно на наше перемещенное/переименнованное ядро и жмем `ctrl-x`

<img width="864" alt="Снимок экрана 2024-11-15 в 15 22 10" src="https://github.com/user-attachments/assets/4d6be2f4-0cf6-4913-a7f0-0d33275166f5">

Либо копируем ядро из бекапа
```sh
[root@slonit ~]# cp /boot/vmlinuz-5.4.17-2136.310.7.1.el8uek.x86_64.bak /boot/vmlinuz-5.4.17-2136.310.7.1.el8uek.x86_64
```
Либо переименовываем обратно
```sh
[root@slonit ~]# mv /boot/vmlinuz-5.4.17-2136.310.7.1.el8uek.custom.x86_64 /boot/vmlinuz-5.4.17-2136.310.7.1.el8uek.x86_64
```
