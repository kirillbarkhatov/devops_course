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


```sh

```
