# Домашнее задание №3

## 1. Написать скрипт, который будет спрашивать имя пользователя и на основе ввода показывать нужную строчку из `/etc/passwd`.
Так же скрипт выводит построчно:
- шелл пользователя
- домашнюю директорию пользователя
- список групп, в которых он состоит
  
После этого скрипт должен спросить, что следует поменять – uid, домашнюю директорию или группу

Если uid, то сначала проверить, доступен ли такой uid, если нет – то один раз предложить ввести заново.

Если домашнюю директорию, то спросить, на какую директорию следует сменить, а также следует ли перемещать домашнюю директорию.

Если группу – то следует спросить, меняем ли мы основную группу или дополнительную.

После чего следует вывести на экран итоговую команду.

```
#!/bin/bash

# Запрос имени пользователя
read -p "Введите имя пользователя: " username

# Получаем строку из /etc/passwd для указанного пользователя
user_info=$(grep "^$username:" /etc/passwd)

if [ -z "$user_info" ]; then
    echo "Пользователь $username не найден."
    exit 1
fi

# Разбираем строку /etc/passwd
IFS=":" read -r user_name password uid gid full_name home_dir shell <<< "$user_info"

# Выводим информацию о пользователе
echo "Информация о пользователе $username:"
echo "Шелл: $shell"
echo "Домашняя директория: $home_dir"
echo "Список групп: $(id -Gn $username)"

# Запрос на изменение данных
read -p "Что хотите изменить? (uid, home_dir, group): " change_option

case "$change_option" in
  "uid")
    # Проверка, доступен ли новый uid
    while true; do
      read -p "Введите новый UID: " new_uid
      if ! getent passwd "$new_uid" > /dev/null; then
        echo "UID $new_uid доступен."
        break
      else
	echo "UID $new_uid уже занят, попробуйте снова."
      fi
    done
    # Вывод итоговой команды
    echo "Команда для изменения UID: sudo usermod -u $new_uid $username"
    ;;

  "home_dir")
    read -p "Введите новую домашнюю директорию: " new_home
    read -p "Перемещать ли содержимое старой домашней директории в новую? (y/n): " move_dir

    # Вывод итоговой команды
    if [ "$move_dir" == "y" ]; then
      echo "Команда для изменения домашней директории и перемещения данных: sudo usermod -d $new_home -m $username"
    else
      echo "Команда для изменения только домашней директории: sudo usermod -d $new_home $username"
    fi
    ;;

  "group")
    read -p "Меняем ли основную группу? (y/n): " change_primary_group
    if [ "$change_primary_group" == "y" ]; then
      read -p "Введите новую основную группу: " new_group
      # Вывод итоговой команды
      echo "Команда для изменения основной группы: sudo usermod -g $new_group $username"
    else
      read -p "Введите дополнительную группу для добавления: " new_group
      # Вывод итоговой команды
      echo "Команда для добавления пользователя в новую группу: sudo usermod -aG $new_group $username"
    fi
    ;;

  *)
    echo "Некорректный выбор. Выберите uid, домашнюю директорию или группу."
    ;;
esac

```

## 2. Написать скрипт, который будет спрашивать название и создавать файл, затем спрашивать права для этого файла и задавать их.

```
#!/bin/bash

# Функция для вывода справки по правам доступа
show_permissions_help() {
    echo -e "\nДоступные варианты прав и их описание:"
    echo -e "------------------------------------------"
    echo -e "Числовой формат (например, 755):"
    echo -e "Режим чтения (r), записи (w), выполнения (x) для пользователя, группы и других."
    echo -e "Каждое число указывает права для пользователя, группы и других соответственно."
    echo -e "   Число 7 = rwx (чтение, запись, выполнение)"
    echo -e "   Число 6 = rw- (чтение, запись)"
    echo -e "   Число 5 = r-x (чтение, выполнение)"
    echo -e "   Число 4 = r-- (только чтение)"
    echo -e "   Число 3 = wx- (запись, выполнение)"
    echo -e "   Число 2 = w-- (только запись)"
    echo -e "   Число 1 = x-- (только выполнение)"
    echo -e "   Число 0 = --- (нет прав)"
    echo -e ""
    echo -e "Пример прав в числовом формате (например, 755):"
    echo -e "  7 = rwx (пользователь)"
    echo -e "  5 = r-x (группа)"
    echo -e "  5 = r-x (другие)"
    echo -e ""
    echo -e "Символьный формат (например, rwxr-xr-x):"
    echo -e "  Символы r (чтение), w (запись) и x (выполнение) указываются для владельца, группы и остальных пользователей."
    echo -e "  Пример: rwxr-xr-x"
    echo -e "    rwx - права владельца (чтение, запись, выполнение)"
    echo -e "    r-x - права группы (чтение, выполнение)"
    echo -e "    r-x - права других пользователей (чтение, выполнение)"
    echo -e ""
}

# Запрос имени файла
read -p "Введите имя файла для создания: " filename

# Создаем пустой файл
touch "$filename"

# Проверяем, создан ли файл
if [ ! -e "$filename" ]; then
    echo "Не удалось создать файл."
    exit 1
fi

echo "Файл '$filename' успешно создан."

# Вывод справки с правами
show_permissions_help

# Запрос прав доступа
while true; do
    echo -e "\nВведите права доступа для файла в числовом формате (например, 755) или символьном (например, rwxr-xr-x):"
    read -p "Права доступа: " permissions

    # Проверка, является ли ввод числовым
    if [[ "$permissions" =~ ^[0-7]{3}$ ]]; then
        # Применяем числовые права
        chmod "$permissions" "$filename"
        echo "Права доступа для файла '$filename' установлены как $permissions."
        break
    elif [[ "$permissions" =~ ^[rwx-]{9}$ ]]; then
        # Применяем символьные права
        chmod "$permissions" "$filename"
        echo "Права доступа для файла '$filename' установлены как $permissions."
        break
    else
        echo "Ошибка: введены некорректные права. Пожалуйста, введите права в правильном формате."
    fi
done

# Выводим информацию о файле и его правах
echo -e "\nТекущие права доступа к файлу '$filename':"
ls -l "$filename" | awk '{print $1}'
```

## 3. Написать скрипт, который должен принять на вход путь до блочного устройства, проверить примонтировано оно или нет.
Если примонтировано - завершиться с `exitCode 90`

Если не примонтировано, то при помощи программы mktemp создать директорию, примонтировать в неё

```
#!/bin/bash

# Проверка, был ли передан путь к блочному устройству
if [ -z "$1" ]; then
    echo "Ошибка: не указан путь к блочному устройству."
    exit 1
fi

device=$1

# Проверка, примонтировано ли устройство
mount_point=$(mount | grep "$device" | awk '{print $3}')

if [ -n "$mount_point" ]; then
    # Устройство уже примонтировано
    echo "Устройство $device уже примонтировано в $mount_point."
    exit 90
else
    # Устройство не примонтировано, создаем временную директорию и монтируем
    temp_dir=$(mktemp -d)
    
    if [ $? -ne 0 ]; then
        echo "Ошибка при создании временной директории."
        exit 1
    fi

    echo "Создана временная директория $temp_dir. Монтируем устройство $device."

    # Примонтируем устройство в временную директорию
    mount "$device" "$temp_dir"
    
    if [ $? -eq 0 ]; then
        echo "Устройство $device успешно примонтировано в $temp_dir."
    else
        echo "Ошибка при монтировании устройства $device."
        exit 1
    fi
fi
```


## 4. Создайте файл со списком пользователей. С помощью `for` выведите на экран содержимое файла с нумерацией строк

Скрипт
```
#!/bin/bash

# Проверяем, существует ли файл users_list.txt
if [ ! -f "users_list.txt" ]; then
    echo "Файл users_list.txt не найден!"
    exit 1
fi

# Используем цикл for для вывода содержимого с нумерацией
i=1
for line in $(cat users_list.txt); do
    echo "$i: $line"
    ((i++))
done
```

Задание
```
[root@slonit hw_3]# cat /etc/passwd > users_list.txt
[root@slonit hw_3]# nano users_list.sh
[root@slonit hw_3]# chmod +x users_list.sh
[root@slonit hw_3]# ./users_list.sh
1: root:x:0:0:root:/root:/bin/bash
2: bin:x:1:1:bin:/bin:/sbin/nologin
3: daemon:x:2:2:daemon:/sbin:/sbin/nologin
4: adm:x:3:4:adm:/var/adm:/sbin/nologin
5: lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
6: sync:x:5:0:sync:/sbin:/bin/sync
7: shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
8: halt:x:7:0:halt:/sbin:/sbin/halt
9: mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
10: operator:x:11:0:operator:/root:/sbin/nologin
11: games:x:12:100:games:/usr/games:/sbin/nologin
12: ftp:x:14:50:FTP
13: User:/var/ftp:/sbin/nologin
14: nobody:x:65534:65534:Kernel
15: Overflow
16: User:/:/sbin/nologin
17: dbus:x:81:81:System
18: message
19: bus:/:/sbin/nologin
20: systemd-coredump:x:999:997:systemd
21: Core
22: Dumper:/:/sbin/nologin
23: systemd-resolve:x:193:193:systemd
24: Resolver:/:/sbin/nologin
25: tss:x:59:59:Account
26: used
27: for
28: TPM
29: access:/dev/null:/sbin/nologin
30: polkitd:x:998:996:User
31: for
32: polkitd:/:/sbin/nologin
33: clevis:x:997:993:Clevis
34: Decryption
35: Framework
36: unprivileged
37: user:/var/cache/clevis:/sbin/nologin
38: unbound:x:996:992:Unbound
39: DNS
40: resolver:/etc/unbound:/sbin/nologin
41: libstoragemgmt:x:995:991:daemon
42: account
43: for
44: libstoragemgmt:/var/run/lsm:/sbin/nologin
45: setroubleshoot:x:994:990::/var/lib/setroubleshoot:/sbin/nologin
46: cockpit-ws:x:993:989:User
47: for
48: cockpit
49: web
50: service:/nonexisting:/sbin/nologin
51: cockpit-wsinstance:x:992:988:User
52: for
53: cockpit-ws
54: instances:/nonexisting:/sbin/nologin
55: sssd:x:991:987:User
56: for
57: sssd:/:/sbin/nologin
58: chrony:x:990:986::/var/lib/chrony:/sbin/nologin
59: sshd:x:74:74:Privilege-separated
60: SSH:/var/empty/sshd:/sbin/nologin
61: tcpdump:x:72:72::/:/sbin/nologin
```


## 5. При помощи `HEREDOC` "сгенерировать" баш-скрипт, который на вход принимает три аргумента:
- вывод `usage` ( как пользоваться скриптом )
- количество генерируемых файлов
- маску имени генерируемых файлов
  
и реализует такую штуку:

Проверяет существование файла с именем скрипта и расширением `lock`
- если файл существует, вывести содержимое файла и завершить работу с кодом `64`
- если файла не существует, то записать в него pid скрипта
  
Перейти в домашнюю директорию пользователя root

Создать именованный пайп

Создать файлы различного размера ( от 10 КБ до 800 КБ ) по маске имени, количество взять из аргумента "количество генерируемых файлов"

Перейти в директорию `/tmp`

Заархивировать созданные файлы при помощи созданного именованного пайпа

Вывести список файлов ( без директорий ) одновременно и на экран, и в файл

Вывести на экран время всей работы скрипта в формате `unixtime`

```
#!/bin/bash

# Генерация скрипта при помощи HEREDOC
cat <<'EOF' > generated_script.sh
#!/bin/bash

# Проверяем, если переданы все необходимые аргументы
if [ "$#" -ne 3 ]; then
    echo "Usage: $0 <usage> <number_of_files> <file_name_pattern>"
    exit 1
fi

# Аргументы
usage="$1"
number_of_files="$2"
file_name_pattern="$3"

# Проверяем, что количество файлов - это положительное целое число
if ! [[ "$number_of_files" =~ ^[0-9]+$ ]] || [ "$number_of_files" -le 0 ]; then
    echo "Error: number_of_files must be a positive integer."
    exit 1
fi

# Выводим usage
echo "$usage"

# Проверка существования lock файла
script_name=$(basename "$0")
lock_file="/tmp/${script_name}.lock"
if [ -f "$lock_file" ]; then
    # Если файл существует, выводим содержимое и выходим с кодом 64
    echo "Lock file exists. Content:"
    cat "$lock_file"
    exit 64
else
    # Если файла нет, записываем pid в lock файл
    echo $$ > "$lock_file"
fi

# Переходим в домашнюю директорию пользователя root
home_dir="/root"
if [ "$EUID" -ne 0 ]; then
    echo "Warning: Not running as root. Switching to current user's home directory."
    home_dir="$HOME"
fi

cd "$home_dir" || { echo "Failed to change directory to $home_dir"; rm "$lock_file"; exit 1; }

# Создаем уникальное имя для пайпа с использованием PID
pipe_name="/tmp/mypipe_$$"  # $$ — это текущий PID

# Проверка и удаление существующего пайпа с уникальным именем
if [[ -p "$pipe_name" ]]; then
    echo "Named pipe $pipe_name already exists, removing it."
    rm "$pipe_name" || { echo "Failed to remove existing named pipe"; rm "$lock_file"; exit 1; }
fi

# Создаем именованный пайп
if [[ ! -p "$pipe_name" ]]; then
    mkfifo "$pipe_name" || { echo "Failed to create named pipe"; rm "$lock_file"; exit 1; }
    echo "Named pipe $pipe_name created."
fi

# Создаем файлы случайного размера от 10KB до 800KB
for ((i = 1; i <= number_of_files; i++)); do
    file_name=$(printf "$file_name_pattern" "$i")
    echo "Creating file: $home_dir/$file_name"
    size=$(( (RANDOM % 791) + 10 )) # Генерация случайного размера от 10 до 800 КБ
    dd if=/dev/urandom of="$file_name" bs=1K count=$size status=none
done

# Переходим в директорию /tmp
cd /tmp || { echo "Failed to change directory to /tmp"; rm "$lock_file"; exit 1; }

# Заархивируем созданные файлы через пайп
(
    cd "$home_dir"
    tar -cvf "$pipe_name" $(for ((i = 1; i <= number_of_files; i++)); do printf "$file_name_pattern " "$i"; done) &
) || { echo "Failed to archive files"; rm "$lock_file"; exit 1; }
tar_pid=$!

# Чтение из пайпа в файл
cat "$pipe_name" > archive.tar
wait $tar_pid

# Выводим список файлов на экран и в файл
ls -p | grep -v / > file_list.txt
cat file_list.txt

# Выводим время работы скрипта в формате unixtime
echo "Unix time: $(date +%s)"

# Удаляем lock файл после завершения работы
if [ -f "$lock_file" ]; then
    rm "$lock_file"
fi

# Удаляем пайп
if [[ -p "$pipe_name" ]]; then
    rm "$pipe_name"
fi

EOF

# Делаем сгенерированный скрипт исполняемым
chmod +x generated_script.sh

echo "Скрипт generated_script.sh успешно сгенерирован!"
```



## 6. Написать скрипт, который проверяет размер примонтированных разделов дисков
Аргументы скрипта:
- вывод помощи
- процент свободного пространства
Если свободного места осталось менее процента из аргумента, отправить сообщение в телеграм
1. Поместить вызов скрипта каждые 8 минут в крон
2. * поместить задание в крон не используя утилиту crontab с параметром `-e`
3. **Заменить вызов cron на `systemd-timers`

Скрипт
```
#!/bin/bash

# Токен и чат ID для Telegram
TOKEN="1436752844:AAHpTWM_yGsUmHFf3zA5JzlX47ectU6qU10"
CHAT_ID="701350489"

# Вывод помощи
usage() {
    echo "Usage: $0 <threshold_percentage>"
    echo "Threshold percentage is the minimum percentage of free space before warning."
    exit 1
}

# Проверка аргумента
if [ -z "$1" ]; then
    usage
fi

# Порог свободного пространства
THRESHOLD=$1

# Функция отправки сообщения в Telegram
send_telegram_message() {
  local message="$1"
  curl -s -X POST "https://api.telegram.org/bot$TOKEN/sendMessage" \
       -d chat_id="$CHAT_ID" \
       -d text="$message"
}

# Функция, которая проверяет свободное пространство на всех примонтированных дисках
check_disk_space() {
  df -h | awk -v threshold="$THRESHOLD" '
    $5 ~ /%/ {  # Проверяем, что процент использованного места в формате процентов
      used_percentage = substr($5, 1, length($5)-1)  # Извлекаем число из строки, например, 12 из 12%
      free_percentage = 100 - used_percentage  # Вычисляем процент свободного места
      if (free_percentage < threshold) {  # Если свободного места меньше порога
        print "Warning: " $1 " mounted at " $6 " has less than " threshold "% free space!" 
        print "Current free space: " $4 " (" used_percentage "% used)"
      }
    }
  '
}

# Проверяем диски и отправляем сообщение, если нужно
warning_message=$(check_disk_space)
if [ ! -z "$warning_message" ]; then
  send_telegram_message "$warning_message"
fi
```

крон
`sudo nano /etc/cron.d/check_disk_space.cron`
```
*/8 * * * * root /root/hw_3/check_disks.sh 95
```

Сервис
`sudo nano /etc/systemd/system/check-disk-space.service`
```
[Unit]
Description=Check disk space and send Telegram notification

[Service]
ExecStart=/root/hw_3/check_disks.sh 95
Type=oneshot
```

Таймер
`sudo nano /etc/systemd/system/check-disk-space.timer`
```
[Unit]
Description=Run disk space check every 8 minutes

[Timer]
OnBootSec=5min
OnUnitActiveSec=8min

[Install]
WantedBy=timers.target
```

Запуск
```
sudo systemctl daemon-reload
sudo systemctl enable check-disk-space.timer
sudo systemctl start check-disk-space.timer
```

Проверка таймера
```
sudo systemctl status check-disk-space.timer
```

## 7. Написать скрипт, который принимает на вход в виде аргумента IP-адрес
Скрипт должен проверить доступность адреса посредством ping три раза и записать в лог один раз за все три проверки:
- доступность хоста
- трассировку до хоста
- с какого роутера получен маршрут

```
#!/bin/bash

# Проверка, что IP передан в аргументе
if [ -z "$1" ]; then
    echo "Использование: $0 <IP-адрес>"
    exit 1
fi

IP=$1
LOG_FILE="host_check.log"

# Проверка доступности хоста с помощью ping (3 пакета)
ping -c 3 $IP > /dev/null 2>&1

if [ $? -eq 0 ]; then
    echo "$(date) - Хост $IP доступен." >> $LOG_FILE
else
    echo "$(date) - Хост $IP недоступен." >> $LOG_FILE
    exit 1
fi

# Выполнение трассировки маршрута
TRACE_RESULT=$(traceroute $IP 2>&1)

echo "$(date) - Результат трассировки до хоста $IP:" >> $LOG_FILE
echo "$TRACE_RESULT" >> $LOG_FILE

# Определение первого роутера из трассировки
FIRST_ROUTER=$(echo "$TRACE_RESULT" | grep -m 1 " *1" | awk '{print $2}')

if [ -n "$FIRST_ROUTER" ]; then
    echo "$(date) - Маршрут проходит через роутер с IP: $FIRST_ROUTER" >> $LOG_FILE
else
    echo "$(date) - Не удалось определить первый роутер в трассировке." >> $LOG_FILE
fi
```


## 8. Собрать `nginx` из исходников с модулем `https://github.com/ritchie-wang/nginx-upstream-dynamic-servers`
Убедиться в доступности `nginx` (страничку отдает или нет)
Положить `nginx` в созданный namespace ядра
Убедиться в доступности `nginx` (страничку отдает или нет)
Написать конфигурацию `reverse-proxy` для сайта `https://mokm51.ru/`



## 9. Написать скрипт, который принимает на вход два аргумента: `<path>` `<time>`
Скрипт должен вывести список файлов из `<path>` старше `<time>`
 может быть задан в любом формате, который понимает утилита date



## 10. Напишите скрипт, который использует один текстовый файл как источник данных
Файл имеет формат:
userName creationDate deletionDate homeDir
userName записывается в виде одного слова
creationDate записывается в формате unix timestamp
deletionDate записывается в формате unix timestamp, если пользователь не удалён, то используется символ тире
homeDir - путь к домашней директории пользователя
Напишите функции просмотра информации о пользователе(1), создания пользователя(2), удаления пользователя (3). Используйте case для аргументов скрипта:
Аргументы скрипта:
- s имяПользователя
- c имяПользователя
- d имяПользователя
- h путьКДомашнейДиректории
- a
s - выводит информацию о пользователе
c - добавляет пользователя с указанной в аргументе h домашней директории в файл и текущей датой
d - изменяет deletionDate с тире на текущую дату
a - выводит список всех пользователей в формате
Номер строки: имяПользователя датаСоздания датаУдаления путьКДомашнейДиректории



## 11. Написать скрипт, на вход которого подается либо PID, либо имя программы.
Скрипт постоянно проверяет что программа находится в памяти
Каждая проверка записывается в лог в формате:
день-месяц-год час-минута-секунда mySvcChecker: service <имя_сервиса> [isUP|isDown]
[isUP|isDown] - состояние сервиса



## 12. На вход скрипта подаются два аргумента
Первый аргумент - входной файл, второй - выходной файл
Если оба файла существуют, то скрипт завершает работу
Если входной файл не существует, то скрипт завершает работу
Скрипт должен вывести в выходной файл содержимое входного файла так, что бы каждая четная строка входного файла поменялась местами с предыдущей нечётной
пример входного файла:
1
2
3
4
5
6
7
8
пример выходного файла:
2
1
4
3
6
5
8
7
Порядок не важен - с начала файла или с конца, главное что бы строки поменялись местами.



## 13. При помощи утилиты nc отправить-принять с одного на другой сервер любое сообщение по TCP и по UDP
При помощи tcpdump посмотреть как устанавливается подключение, сообщение пересылается
Поменять mac-address, параллельно посмотреть tcpdump-ом что происходит



## 14. Переводим SELinux в Permissive режим
Пишем скрипт, который слушает 60 порт tcp ( неважно чем - хоть nc )
При подключении клиента к 60 порту ( telnet / nc ), скрипт выдает приветствие.
Если скрипт на 60 порту получил сообщение от клиента:
getDate
, то должен выдать текущую дату в формате ГОД-МЕСЯЦ-ДЕНЬ ЧАС-МИНУТА-СЕКУНДА
getEpoch
, то должен выдать время в unixtime
getInetStats
, то отдаем статистику всех сетевых карт
getInetStats <имяКарты>
, то отдаем статистику указанной сетевой карты
bye
, то завершить сессию
В ходе работы должен вестись лог в формате:
ГОД-МЕСЯЦ-ДЕНЬ ЧАС-МИНУТА-СЕКУНДА ПИД локальныйИП удаленныйИП командаОтКлиента результатВыполнения



## 15. Берём ранее сконфигурированный nginx
Включаем SELinux в режим Enforcing
Создаем директорию
/pubHtml
В конфиге web-сервера сменить root-директорию с дефолтной на /pubHtml
Кладём любой файл в /pubHtml
В качестве примера можно использовать файл index.html с содержимым
<html><body><h2>test file
При помощи утилит audit2allow и setroubleshoot разрешаем работу httpd-демона с /pubHtml
При обращении к виртуальной машине по IP-адресу на 80/tcp-порт получаем свой файл из директории /pubHtml от web-сервера
Добавляем в конфигурацию веб-сервера порт 4589
При обращении к виртуальной машине по IP-адресу на 4589/tcp-порт получаем свой файл из директории /pubHtml от web-сервера
Разрешаем работу скрипта при помощи утилит audit2allow и setroubleshoot



## 16. firewalld
Нарисовать правило сервиса с указанным портом (см. ниже) для публичной зоны в виде файла, который необходимо положить в то место, из которого firewalld читает правила.
На машине 1 запустить nc на любом порту в режиме прослушивания. На этой машине должно быть правило из пункта выше
На машине 2 запустить nc с подключением к машине 1
Изменить правило сервиса так, что бы подключение с машины 2 к машине 1 отбрасывалось
iptables
Написать правила для доступа к порту 22/tcp только для выделенного списка адресов, всем остальным адресам доступ запрещен
