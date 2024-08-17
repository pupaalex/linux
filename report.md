## Part 1. Установка ОС

Устанавливаем **Ubuntu 20.04 Server LTS** без графического интерфейса на виртуальную машину - VirtualBox

Чтобы узнать версию операционной системы выполним команду \
`cat /etc/issue`:

  ![os_version](image/task1.png)


## Part 2. Создание пользователя

- Cоздадим пользователя testuser \
`sudo adduser testuser`:

  ![add_user](image/task2.1.png)

- Добавим в adm группу пользователя testuser \
sudo usermod -aG adm testuser

  ![show_user](image/task2.2.png)

- Покажем пользователя testuser быть в выводе команды \
`cat /etc/passwd`

  ![show_user](image/task2.3.png)

## Part 3. Настройка сети ОС

### Изменение имени хоста

sudo hostnamectl set-hostname user-1

### Установка временной зоны

#### Найти временную зону
timedatectl list-timezones

#### Установить временную зону
sudo timedatectl set-timezone 'Europe/Moscow'

### Просмотр сетевых интерфейсов

ip link show

Расшифровка имени сетевого интерфейса enp0s3:

en — Ethernet
p0 — указывает на номер шины PCI, на которой находится сетевой интерфейс. В данном случае, 0 может означать первую шину или первый интерфейс на шине.
s3 — слот на шине, в который установлен сетевой интерфейс. Здесь 3 указывает на третий слот.

### Loopback интерфейс

Loopback интерфейс lo используется для внутренних коммуникаций на хосте. Это позволяет системе отправлять данные самой себе, что используется для тестирования и разработки. Обращения к IP-адресу 127.0.0.1 направляются на этот интерфейс.

### Получение IP адреса от DHCP

sudo dhclient -v <имя_интерфейса> в моём случае enp0s3

### Расшифровка DHCP

DHCP - Dynamic Host Configuration Protocol, протокол динамической конфигурации узла.

Если установить и выполнить утилиту ipconfig, то можно получить более подробную информацию в отношении сетевых интерфейсов и посмотреть ip адрес назначенный от DHCP
enp0s3 – имя сетевого интерфейса
192.168.0.240 - IP адрес, предоставленный DHCP
RX packets - принято в МБит
TX packets - передано в МБит и пр.

### Вывод IP-адреса шлюза

ip route show

![IP-адрес](image/task3.3.png)
### Получение внешнего IP-адреса

curl ifconfig.me

### Задание статических настроек сети

#### Открыть конфигурационный файл сети
sudo nano /etc/netplan/00-installer-config.yaml

введем:
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: no
      addresses: [192.168.1.10/24]
      gateway4: 192.168.1.1
      nameservers:
        addresses: [1.1.1.1, 8.8.8.8]

#### Применить настройки
sudo netplan apply

![настройки](image/task3.sudonetplan.png)

#### Перезагрузка 
sudo reboot

#### Проверим настройки

ip addr show enp0s3
ip route show
systemd-resolve --status

![addr](image/task3.addrshow.png)

![route](image/task3.ipshowroot.png)

##### Таким образом для выполнения наших задач использовались команды для изменения настроек имени машины, временной зоны, просмотра и настройки сетевых интерфейсов (ip, gw, dns), а также команды для перезагрузки системы.

### Пропингуем хосты 
ping -c 4 1.1.1.1
ping -c 4 ya.ru

  ![ping](image/task3.ping.png)

## Part 4. Обновление ОС

- Для обновления системных пакетов до последней версии выполним команду:

sudo apt update && sudo apt upgrade -y

apt update получает список пакетов, apt upgrade обновляет. Ключ y используется для автоматического ответа "да" на все запросы подтверждения

- Повторно вводим команду обновления, убеждаемся, что теперь пакеты для обновления отсутствуют 0 upgraded:

  ![update](image/task4.1.png)

## Part 5. Использование команды sudo

- Чтобы разрешить пользователю testuser выполнять команды с привилегиями суперпользователя через sudo, добавим его в группу sudo с помощью следующей команды:

sudo usermod -aG sudo testuser

- Объяснение назначения команды sudo:

Команда sudo (от англ. superuser do) позволяет обычным пользователям выполнять команды с привилегиями суперпользователя, как если бы они были пользователем root. Это обеспечивает управление правами пользователей и их действиями на системном уровне, позволяя выдавать ограниченные или полные административные привилегии для выполнения определенных команд. Это повышает безопасность, так как нет необходимости работать с привилегиями root постоянно, и улучшает отслеживание административных действий через журналы sudo.

- Переключимся на пользователя testuser в терминале, используйте команду su (switch user):

su - testuser

Эта команда переключит на пользователя testuser, запросив его пароль. Использование дефиса - после su обеспечивает запуск среды пользователя testuser, включая загрузку его профиля и рабочего каталога.

- Изменим имя хоста операционной системы из под пользователя testuser с sudo:

sudo hostnamectl set-hostname hostname2

Покажем, что имя хоста операционной системы изменилось

  ![update](image/task5.2.png)

## Part 6. Установка и настройка службы времени

- Вывод времени часового пояса, в котором мы находимся командой:

timedatectl

В выводе видим текущее время, временную зону, синхронизацию времени

  ![time_zone](image/task6.1.png)

- Проверка, что синхронизация времени с NTP включена

timedatectl show

Видим, строку NTPSynchronized=yes, указывающую на то, что синхронизация времени с NTP серверами активна:

  ![time_2](image/test6.2.png)

- Если б служба была не включена, то следовало б выполнить:

sudo apt install systemd-timesyncd #установка
sudo systemctl enable systemd-timesyncd #включение
sudo systemctl start systemd-timesyncd #старт
systemctl status systemd-timesyncd #проверка статуса

а также отредактировать конфигурационный файл /etc/systemd/timesyncd.conf, чтобы добавить свои NTP сервера, если не хочется использовать сервера по умолчанию.

## Part 7. Установка и использование текстовых редакторов

- Используя каждый из трех выбранных редакторов, создаем файл test_X.txt, где X - название редактора, в котором создан файл. Напишем в нём свой никнейм, закроем файл с сохранением изменений.

VIM 

  vim test_vim.txt

Скрин Vima

  ![VIM_username](image/task7.1.png)

Выход: ESC -> :wq

NANO

  nano test_nano.txt

Скрин NANO

  ![nano_username](image/task7.2.png)

Выход: Ctrl+X -> Y -> Enter

MCEDIT

sudo apt install mc
mcedit text_mcedit.txt

Скрин MCEDIT

  ![mc_username](image/task7.3.png)


Выход: F10 -> "Yes" - Enter

- Используя каждый из трех выбранных редакторов, откроем файл на редактирование, отредактируем файл, заменив никнейм на строку «21 School 21», закроем файл без сохранения изменений.

VIM 

  ![VIM_username](image/task7.4.png)

Выход без сохранения: ESC -> :q!

NANO

  ![nano_username](image/task7.5.png)

Выход без сохранения: Ctrl+X -> N 

MCEDIT

  ![mc_username](image/task7.6.png)

Выход: F10 -> "No" - Enter

- Используя каждый из трех выбранных редакторов, отредактируем файл ещё раз (по аналогии с предыдущим пунктом), а затем осуществим поиск по содержимому файла (слово) и замены слова на любое другое.

VIM. Для поиска использую /:

![vim_search](image/task7.7.png)

VIM. Для замены использую /%s/School/Age/g: (s - замена для всего файла, g - все вхождения)

![vim_rep](image/task7.8.png)

NANO. Для поиска использую Ctrl+W, после нажатия enter курсор переместится в начало слова

![nano_search](image/task7.10.png)

NANO. Для замены использую Ctrl+W -> Ctrl+R, водим слово для замены, затем на какое слово меняем: 

![nano_rep](image/task7.9.png)

затем после подтверждения Y и нажатия enter слово заменится

MCEDIT. Для поиска нажимаем F7

![mcedit_search](image/task7.11.png)

после нажатия enter слово будет выделено

MCEDIT. Для поиска и замены нажимаем F4

![mcedit_rep](image/task7.12.png)

после нажатия enter, будет предложено заменить первое вхождение или все вхождения, или отменить замену

## Part 8. Установка и базовая настройка сервиса SSHD

##### Устанавливаем пакет openssh-server, который содержит службу SSHd:

sudo apt install openssh-server 

##### Добавим автостарт службы при загрузке системы:

sudo systemctl enable ssh

(systemctl утилита для управления состоянием системы и служб. enable ssh включает автоматический запуск службы SSH при старте системы.)

##### Перенастроем службы SSHd на порт 2022:

Откроем файл конфигурации SSHd в редакторе:

sudo nano /etc/ssh/sshd_config
Найдем строку Port 22 и изменим её на Port 2022, удалим символ # (раскомментим) и заменим 22 на 2022.
Сохраним файл ``Ctrl + O` и закроем редактор Ctrl + X.

Применим изменения, перезапустив службу:

sudo systemctl restart sshd


##### Покажем наличие процесса sshd с помощью ps:

ps -C sshd

ps отображает текущие активные процессы, ключ -C sshd фильтрует процессы по имени команды, в этом случае по sshd.

![sshd](image/task8.1.png)
##### Перезагрузим систему:

sudo reboot

Повторно кратко опишем, что сделано:
- Установил службу SSHd.
- Включил автостарт службы при загрузке системы.
- Изменил порт службы SSHd на 2022.
- Проверил наличие процесса sshd с использованием команды ps.
- Перезагрузил систему.

##### Проверка порта 2022

Проверка порта с помощью netstat:

netstat -tan

![netstat](image/task8.2.png)

-t показывает TCP соединения.
-a отображает все соединения и прослушиваемые порты.
-n выводит адреса в числовом формате, без преобразования в имена.

Значение столбцов в выводе netstat:

- Proto: Протокол (н-р TCP).  
- Recv-Q и Send-Q: Количество байтов в очереди на приём и отправку.  
- Local Address: Локальный адрес и порт.  
- Foreign Address: Адрес и порт удалённого соединения.  
- State: Состояние соединения (например, LISTEN).  
- Значение 0.0.0.0: указывает, что служба доступна на всех IP-адресах машины.

## Part 9. Установка и запуск утилит top и htop

##### Установка утилит:

sudo apt update
sudo apt install top htop

Обычно утилита top предустановлена в большинстве дистрибутивов Linux, но htop нужно устанавливать отдельно.

##### Запуск top:

top

##### Затем найдем в выводе команды следующую информацию:

uptime: 00:42:18 - Время работы системы отображается в верхнем углу
1 user - Количество авторизованных пользователей - Количество сеансов пользователя указано в строке с "user(s)".
0.03, 0.03, 0.00 - Общую загрузку системы: видно в строках, начинающихся с "load average".
133 - Общее количество процессов: указано в строке, начинающейся с "Tasks".

- Загрузка cpu.  Процентное использование CPU указано в строке, начинающейся с "%Cpu(s)":
1. 0.0 us - действия в пользовательском пространстве в %  
2. 0.1 sy - затраченного на действия в пространстве ядра в %  
3. 0.0 ni - затраченного на процессы с низким приоритетом в %  
4. 99.9 id - процент (%) затраченного на простаивание - какое количество времени процессор делает ничего
5. 0.0 wa - процент (%) затраченного на ожидание дисковых операций
6. 0.0 hi - процент (%) процессорного времени, затраченного на обработку аппаратных прерываний
7. 0.0 si - процент (%) процессорного времени, затраченного на обработку прерываний ПО
8. 0.0 st - время в вынужденном ожидании виртуального CPU, пока гипервизор обслуживает другой процессор

- Загрузка памяти: Общее использование памяти отображается в строках, начинающихся с "MiB Mem":
1. 3916.0 total - всего памяти
2. 3214.1 free - доступно незамедлительно
3. 181.5 used - используется в данный момент
4. 520.4 buff/caсhe - сумма буферов и кэша

- Загрузка памяти: MiB Swap относится к использованию пространства подкачки (swap space) в системе. Здесь указывается:
1. 0.0 total: Общий размер выделенного пространства подкачки.
2. 0.0 used: Количество пространства подкачки, которое в данный момент используется.
3. 0.0 free: Количество свободного пространства в области подкачки.
4. 3579.4 avail Mem: Оценка того, сколько памяти доступно для запуска новых приложений, не вытесняя страниц в подкачку.
5. Пространство подкачки — это область на жёстком диске, которую операционная система использует для выделения памяти, когда физическая оперативная память (RAM) полностью используется. Когда системе не хватает оперативной памяти для выполнения задач, она может переместить некоторые данные из RAM в область подкачки, чтобы освободить память. Эти данные измеряются в мебибайтах (MiB), где 1 MiB равен 2^20 (1,048,576) байтам.

- Чтобы отсортировать top по памяти нажмем shift+m. pid процесса занимающего больше всего памяти - 632 (RES)
- Чтобы отсортировать top по CPU нажмем shift+p. pid процесса занимающего больше всего памяти - 3 (CPU)

##### Запустим утилиту htop. 

В отчёт вставим скрины с выводом команды htop отсортированному (используя F6):

1. по PID

![htop_pid](image/task10.1.png)

2. по PERCENT_CPU

![htop_cpu](image/task10.2.png)

3. по PERCENT_MEM

![htop_mem](image/task10.3.png)

4. по TIME

![htop_time](image/task10.4.png)

5. отфильтрованному для процесса sshd (нажатие F4)

![htop_sshd](image/task10.5.png)

6. с процессом syslog, найденным, используя поиск (нажатие F3)

![htop_syslog](image/task10.6.png)

7. с добавленным выводом hostname, clock и uptime (нажатие setup F2)

![htop_setup](image/task10.7.png)

## Part 10. Использование утилиты fdisk

![fdisk](image/task11.1.png)

- Название жесткого диска: отсутсвует
- Размер жесткого диска: 15G
- Количество секторов: 31457280
- Размер swap: swap не подключен

## Part 12. Использование утилиты du

Запустим команду sudo du 
![part12_1](image/task12.1.png)

- Выведи размер папок /home, /var, /var/log (в байтах)
c помощью команды 
1) sudo du -hs /home 
2) sudo du -hs /var 
3) sudo du -hs /var/log 
![part12_2](image/task12.2.png)
![part12_3](image/task12.3.png)
![part12_4](image/task12.4.png)

- Выведи размер всего содержимого в /var/log (не общее, а каждого вложенного элемента, используя*) 
c помощью команды 
sudo du -h /var/log/*
![part12_5](image/task12.5.png)

## Part 13. Установка и использование утилиты ncdu

установим ncdu с помощью команды sudo apt install ncdu
![part13_1](image/task13.1.png)

Выведи размер папок /home, /var, /var/log при помощи команды:
1) ncdu /home
2) ncdu /var
3) ncdu /var/log

/home:
![part13_2](image/task13.2.png)

/var:
![part13_4](image/task13.3.png)

/var/log:
![part13_3](image/task13.4.png)

## Part 14. Работа с системными журналами

- Открой для просмотра:

1. /var/log/dmesg - фиксируются ошибки работы драйверов и оборудования
![dmesg](image/task14.1.png)

2. /var/log/syslog
![syslog](image/task14.2.png)

3. /var/log/auth.log - содержат информацию об авторизации пользователей, то есть попытки не/успешных входов в систему и методов аутентификации.
![auth](image/task14.3.png)

- Напиши в отчёте время последней успешной авторизации, имя пользователя и метод входа в систему: информация об удачной попытки входа отсутствует. 
![part14_1](image/task14.1.png)

- Перезапусти службу SSHd.
- Вставь в отчёт скрин с сообщением о рестарте службы (искать в логах).
![restart](image/task14.4.png)

## Part 15. Использование планировщика заданий CRON

- Используя планировщик заданий, запусти команду uptime через каждые 2 минуты
с помощью команды crontab -e

![crontab](image/task15.1.png)

- Найди в системных журналах строчки (минимум две в заданном временном диапазоне) о выполнении.

![part15_2](image/task15.3.png)

- Выведи на экран список текущих заданий для CRON
c помощью команды crontab -l

![crontab](image/task15.2.png)

- Удали все задания из планировщика заданий
c помощью команды crontab -r

![part15_4](image/task15.4.png)