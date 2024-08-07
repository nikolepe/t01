# Отчет по установке Ubuntu 20.04 Server LTS на VirtualBox

## Part 1. Установка ОС
- Скачала образ Ubuntu 20.04 Server LTS с официального сайта.
- Создала новую виртуальную машину в VirtualBox без графического интерфейса.
- Установила Ubuntu 20.04 Server LTS на созданную виртуальную машину.
- После установки, выполнила команду `cat /etc/issue` для проверки версии Ubuntu.
![Вывод команды cat /etc/issue](screen/Part_1.png)

## Part 2. Создание пользователя
- Вызов команды для создания нового пользователя "new_user" в группе adm `sudo adduser --ingroup adm new_user`:
![Вызов команды для создания нового пользователя](screen/Part_2_1.png)

- Вывод команды `cat etc/passwd` для инфо о данных нового пользователя :
![Вызов команды для создания нового пользователя](screen/Part_2_2.png)

## Part 3. Настройка сети ОС
- задаю/меняю название машины на "user-1" `sudo hostnamectl set-hostname user-1`:

![Замена названия машины на user-1](screen/Part_3_1_new_host_name.png)

- установка временной зоны, соответствубщей текущему положению:
    * устанавливаю временную зону, соответствующую текущему местоположению с помощью команды `sudo timedatectl set-timezone Europe/Moscow`:
    ![Установка текущей временной зоны](screen/Part_3_2_set_current_timezone.png)
    * проверка `timedatectl status` :
    ![проверка установленной зоны](screen/Part_3_3_check_current_timezone.png)

- Для вывода названия сетевых интерфейсов использую `ip link show`
![Вывод названия сетевых интерфейсов](screen/Part_3_4_network_interfaces_output.png)


На представленном скрине видно, что в системе есть 2 типа интерфейса. Физический интерфейс enp0s3 — реальный сетевой интерфейс Ethernet для подключения к локальной сети или интернету. Виртуальный интерфейс lo - специяльный виртуальный интерфейс, который используется для обратной связи между процессами на одной машине. Он позволяет программам общаться друг с другом внутри одной системы, как если бы они были разделены сетью.

- Получение IP-адреса от DHCP сервера с помощью команды `ip addr show`:
![Вывод данных ip-адреса](screen/Part_3_5_ip_addres_output.png)

DHCP — это протокол, который позволяет автоматически назначать IP-адреса и другие сетевые параметры устройствам в сети. Это упрощает управление сетевыми настройками, поскольку администратору не нужно вручную настраивать каждый сетевой интерфейс. Вместо этого, устройства запрашивают конфигурацию у DHCP-сервера при подключении к сети.

- Определение и вывод внешнего IP-адреса шлюза и внутреннего IP-адреса шлюза
    * Внешний IP-адрес шлюза можно определить с помощью команды `curl ifconfig.me`:
    ![внешний(публичный) ip-адрес; адрес маршрутизатора, который присваивается интернет-провайдером для идентификации вашей сети в глобальной сети](screen/Part_3_6_external_ip_addres.png)
    * Внутренний IP-адрес шлюза можно получить с помощью команды `ip route | grep default`:
    ![адрес внутри локальной сети, недоступен из вне, используется для обмена данными внутри сети или для отправки данных за пределы этой сети – этот адрес служит точкой доступа (адресом по умолчанию) для всех устройств в сети, через которую они могут подключаться к другим сетям](screen/Part_3_7_internal_ip_addres.png)

- Настраиваю статические(заданные вручную) настройки ip, gw, dns:
    * открываю едактирования файл конфигурации сети Netplan с правами суперпользователя `sudo nano /etc/netplan/00-installer-config.yaml`:
    ![исходное содержание файла Netplan](screen/Part_3_8_netplan_current.png)
    * отключаю DHCP, задаю значения ip, gw, dns:
    ![изменение файла](screen/Part_3_9_netplan_new.png)

- Перезагружаю виртуальную машину, чтобы убедиться, что статичные сетевые настройки (ip, gw, dns) соответствуют заданным в предыдущем пункте:
    * Вывод данных ip-адреса `ip addr show` и gw `ip route`:
    ![Вывод данных ip-адреса (ip addr show) и gw (ip route)](screen/Part_3_10_set_static_ip_gw_dns.png)

- Пинг удаленных хостов `ping 1.1.1.1` и `ping ya.ru`
![ping 1.1.1.1 и ping ya.ru](screen/Part_3_11_ping.png)
    * "0% packet loss" означает, что пакеты данных, отправленные с помощью команды `ping`, были успешно доставлены до целевого сервера и обратно, что свидетельствует о хорошем качестве соединения.

## Part 4. Обновление ОС
- Обновляем системные пакеты до последней на момент выполнения задания версии `sudo apt-get upgrade`. Сообщение при попытке повторного обновления пакетов:
![Вывод сообщения при попытке повторного обновления пакетов](screen/Part_4_OS_upgrade.png)

## Part 5. Использование команды sudo
- Для того чтобы пользователь мог выполнять команды с привилегиями суперпользователя через sudo, его нужно добавить в группу sudo или в файл sudoers. Это можно сделать с помощью команды usermod `sudo usermod -aG sudo имя_пользователя` bи после этого проверяем, в какой группе группе состоит пользователь и что права ему были переданы `groups new_user`:
![Передача прав позьзования командой sudoновому пользователю и проверка](screen/part_5_sudo_to_nwe_user.png)
        * Истинное назначение команды sudo
Команда sudo (SuperUser DO) позволяет пользователям выполнять команды с привилегиями суперпользователя (root). Это обеспечивает дополнительный уровень безопасности, поскольку не требует постоянного входа в систему под учетной записью суперпользователя, а позволяет временно повышать привилегии для выполнения определенных задач, требующих таких прав.
- для изменения hostname ОС от имени пользователя, созданного в пункте 2[Part 2]:
    * Переходим в учетку new_user `su new_user` и меняем hostname на "newhostname" `sudo hostnamectl set-hostname newhostname`, проверяем название машины с помощью команды `hostname`:
    ![вывод измененного hostname пользователем new_user](screen/part_5_new_hostname_to_new_user.png)

## Part 6. Установка и настройка службы времени
- Для включения и запуска службы автоматической синхронизации времени systemd-timesyncd используется следующиая команда `sudo timedatectl set-ntp true`. Для проверки, что автоматическая синхронизация включена выводим команду `timedatectl show` и смотрим на NTPSynchronized=yes:
![вывод времени часового пояса, где находимся](screen/part_6_timezone.png)

## Part 7. Установка и использование текстовых редакторов

### Создание текстового файла, редактирование и выход с сохранением измений

#### VIM
- Редактор VIM был предустановлен в ОС
- Создаем файл test_VIM.txt `vim test_VIM.txt`
- Для внесения изменений нажимаем `i` (insert), по завершении ввода нажимаем `ESC`
- Для сохранения изменений и выхода нажимаем `:wq` (write & quit) и `ENTER`:
![Содержание файла test_VIM.txt перед закрытием](screen/part_7_test_vim.png)

![Вывод после выхода из редактора VIM](screen/part_7_test_vim_after_exit.png)

#### NANO
- Редактор NANO был предустановлен в ОС
- Создаем файл test_NANO.txt `nano test_NANO.txt`
- Вносим текст; для сохранения изменений нажимаем `Ctrl`+`O` и `Enter`; для выхода из редактора далее `Ctrl`+`X`
    ![Содержание файла test_NANO.txt перед закрытием](screen/part_7_test_nano.png)

#### JOE
- Для установки JOY необходимо обновить систему `sudo apt update`
- после этого устанавливаем редактор с помощью команды `sudo apt install joe`
- Создаем файл test_JOE.txt `joe test_JOE.txt`
- Вносим текст; для сохранения изменений нажимаем `Ctrl`+`K` и `Enter`; для выхода из редактора далее `X`
    ![Содержание файла test_JOE.txt перед закрытием](screen/part_7_test_joy.png)
    ![Вывод после выхода из редактора JOE](screen/part_7_test_joy_after_exit.png)

### Редактирование файла и выход без сохранения изменений
#### VIM
- Открываем файл test_VIM.txt `vim test_VIM.txt`
- Нажимаем `i` и заменяем ник на 21 School 21, далее `ESC` 
- Для выхода из редактора без сохранения набираем `:q!` и `Enter`

    ![Содержание файла test_VIM.txt после редактирования](screen/part_7_test_vim_after_editing.png)

#### NANO
- Открываем файл test_NANO.txt `nano test_NANO.txt`
- Заменяем текст; для выхода из редактора без сохранения изменений нажимаем `Ctrl`+`X` и на вопрос редактора `"Save modified buffer?"` вводим `N`
    ![Содержание файла test_NANO.txt после редактирования](screen/part_7_test_nano_after_editing.png)

#### JOE
- Открываем файл test_JOE.txt `joe test_JOE.txt`
- Заменяем текcт; для выхода без сохранения нажимаем `Ctrl` + `c` и на вопрос редактора `"Lose changes to this file?"` вводим `y`
    
    ![Содержание файла test_JOE.txt после редактирования](screen/part_7_test_joy_after_editing.png)

### Поиск по содержимому и замена
#### VIM
- Открываем файл test_VIM.txt `vim test_VIM.txt`
- Вносим изменения в файл (см. выше)
- Для поиска вводим `/<искомое значение>`, в данном случае `/21` и `ENTER`, и система сразу его подсвечивает. Дальнейшее переключение между результатами производится вводом n (следующее значение) и `Shift` + `n` (предыдущее значение). 
Рузультат поиска:

    ![Содержание файла test_VIM.txt в режиме поиска подстроки '21'](screen/part_7_test_vim_search.png)
- Для замены во всем файле подстроки `"21"` на подстроку `"555"` в командном режиме вводим: `:%s/21/555/g`( %s – выполнить замену во всем файле, g – замена всех совпадений)    
    * Команда для замены всех вхождений:
![Замена всех вхождений](screen/part_7_test_vim_replace_all.png)
    * Команда для замены каждого отдельного значения:
![замена по одному](screen/part_7_test_vim_replace_one.png)

#### NANO
- Открываем файл test_NANO.txt `nano test_NANO.txt`

- Для поиска используем команду `Ctrl` + `W`. Внизу экрана появится окно `Search:`, вводим подстроку, которую хотим найти, и `Enter`. Для поиска следующего вхождения данного выражения используется сочетание клавиш `Alt` + `W` (`Option` + `W` для mac). Для поиска предыдущего - `Alt` + `Q` (`Option` + `Q` для mac):
![вывод поиска](screen/part_7_test_nano_search.png)

-  Для поиска и замены: `Ctrl` + `\`. Внизу экрана появится окно `Search (to replace)`, вводим подстроку, которую хотим найти и `Enter`. Далее появляется окно `Replace with`, вводим подстроку для замены и `Enter`. Система подсвечивает вхождение и спрашивает `Replace this instatnce?`:
    + Y - yes
    + N - no (при выборе этого варианта переходит к следующему вхождению)
    + A - all (замена всех вхождений)
    + ^С - cancel
![вывод после ввода команды поиска ](screen/part_7_test_nano_search_and_replace.png)

#### JOE
- Открываем файл test_JOE.txt `joe test_JOE.txt`
- Заходим в режим поиска и замены: `Ctrl` + `K`, далее `F` (find). Внизу экрана появится окно `Find (^K H for help)`:
![Результат поиска в файле test_VIM.txt подстроки '21'](screen/part_7_test_joy_search.png)

- вводим подстроку, которую хотим найти и `Enter`. Появляются варианты: `(I)gnore (R)eplace (B)ackwards Bloc(K) (^K H for help):` 
![ввод строки на замену](screen/part_7_test_joy_line_answer.png)
- - Выбираем `R` (replace)
![Ответ на запрос системы, что делать в файле test_VIM.txt с подстрокой '21'](screen/part_7_test_joy_result.png)


## Part 8. Установка и базовая настройка сервиса **SSHD**
SSHd (Secure Shell Daemon) – это серверная часть протокола SSH (пакета OpenSSH), которая отвечает за прием и обработку входящих соединений по протоколу SSH. SSH используется для безопасного удаленного доступа к серверам и другим устройствам в сети, обеспечивая шифрование (устанавливает защищенное соединение между клиентом и сервером, шифруя все передаваемые данные) и аутентификацию данных (поддерживает разные методы аутентификации - пароли, ключи SSH, сертификаты и др.).

- установка службы SSH:
    * Обновляем список пакетов `sudo apt update`
    * Устанавливаем пакет OpenSSH Server `sudo apt install openssh-server`
    * Проверяем, что служба работает `sudo systemctl status ssh`:
    ![Вывод результата команды 'sudo systemctl status ssh'](screen/part_8_Open_SSH_installation_check.png)

- Добавляем автостарт службы при загрузке системы `sudo systemctl enable ssh`:
![Добавление автостарта службы SSHd при загрузке системы'](screen/part_8_enable_ssh.png)

- Перенастраиваем службу SSHd на порт 2022:
    * Открываем файл конфигурации SSHd `sudo nano /etc/ssh/sshd_config` 
    * Находим строку `#Port 22`, изменяем её на `Port 2022` (обязательно убираем знак `#`)
    ![Результат изменения файла конфигурации SSHd](screen/part_8_ssh_config_file_updated.png)
    * Перезапускаем службу SSHd командой `sudo systemctl restart sshd`

- Используем команду ps, чтобы показать наличие процесса sshd
    + Команда `ps` используется для отображения информации о запущенных процессах. С этой командой используются следующие ключи:
        * `a` – показать процессы всех пользователей, не только текущего
        * `u` – показать процессы с деталями, такими как пользователь, под которым запущен процесс, и используемая память
        * `x` – показать процессы, не привязанные к терминалу 
        * `aux`: эти ключи используются для вывода всех процессов, запущенных всеми пользователями, в пользовательском формате.
    
    + Вводим `ps aux | grep sshd`
    ![Вывод команды, подтверждающий наличие процесса SSHd](screen/part_8_ssh_process_exists.png)

- Перезагружаем систему `sudo reboot`
- После перезагрузки проверьте, что служба SSHd запущена и слушает порт 2022 `sudo netstat -tan | grep :2022`:

** Если команда netstat отсутствует, установите пакет net-tools `sudo apt-get install net-tools`
![Вывод команды 'sudo apt-get install net-tools'](screen/part_8_install_net-tools.png)
![Вывод команды `sudo netstat -tan | grep :2022`](screen/part_8_netstat-tan.png)

- Kоманда `netstat` с ключем `-tan` выводит список всех активных сетевых соединений на компьютере, а также прослушивающих портов и состояний соединений.
Этот ключ включает в себя три отдельных ключа:
  + `-t` – указывает `netstat` отобразить только TCP-соединения
  + `-a` – указывает `netstat` отобразить все соединения и прослушиваемые порты
  + `-n` – указывает `netstat` отобразить числовые значения портов и ip-адресов вместо их символьных имен

- Вывод команды покажет, что служба SSHd слушает на порту 2022. Значение 0.0.0.0:2022 означает, что SSHd принимает соединения на порту 2022 с любого IP-адреса.


## Part 9. Установка и использование утилит **top**, **htop**
Команды `top` и `htop` – инструменты мониторинга производительности – позволяют просматривать информацию о текущем использовании системных ресурсов и запущенных процессах.

###Установка и запуск утилит
Утилита top обычно предустановлена во многих дистрибутивах Linux. используя команду `sudo apt-get update` я обновила пакет:
![вывод команды  'sudo apt-get update'](screen/part_9_update_top.png)

### Вывод на экран команды `top`
![Вывод на экран команды top](screen/part_9_top_result.png)
    - uptime: 24 min (время, прошедшее с момента последней загрузки системы)
    - количество авторизованных пользователей: 1 user 
    - общая загрузка системы: 0.00, 0.00, 0.00 (load average – среднее количество процесоов, ожидающих доступа к ЦП в течение последних 1, 5 и 15 минут)
    - общее количество процессов: 95 (Tasks total)
    - загрузка cpu:
        + 0.0 us: 0.0% времени процессора занято пользовательскими процессами
        + 0.0 sy: 0.0% времени процессора занято системными процессами
        + 0.0 ni: 0.0% времени процессора занято процессами с приоритетом ниже нормального
        + 100.0 id: 100% времени процессора простаивает, т.е. не используется
        + 0.0 wa: 0.0% времени процессора ждет ввода-вывода.
        + 0.0 hi: 0.0% времени процессора ждет обработки аппаратных прерываний
        + 0.0 si: 0.0% времени процессора ждет обработки программных прерываний
        + 0.0 st: 0.0% времени процессора занято виртуализацией
    - загрузка памяти (MiB Mem): общий объем оперативной памяти составляет 1971.6 мегабайт, из которых:
        + 1400.2 мегабайт свободны
        + 1237.1 мегабайт используются
        + 595,0 мегабайт используются для буферизации и кэширования
     - pid процесса занимающего больше всего памяти: 658 
    (для сортировки процессов по использованию памяти воспользуемся командой `Shift` + `M`)
    ![Вывод на экран команды top, отсортированный по занимаемой памяти](screen/part_9_top_result_sorted_by_memory.png)
    - pid процесса, занимающего больше всего процессорного времени: 1439 
    (для сортировки процессов по занимаемому процессорному времени воспользуемся командой `Shift` + `P`)
    ![Вывод на экран команды top, отсортированный по процессорному времени](screen/part_9_top_result_sorted_by_CPU.png)
    - для выхода нажимаем `q`

### Вывод команды `htop`
 Для сортировки после ввода команды `htop` нажимаем `F6` и переключаемся по элементам для выбора, затем `ENTER`)
    * отсортированный по `PID`:
    ![Вывод команды htop, отсортированный по PID](screen/part_9_ptop_PID_sorted.png)

    * отсортированный по `PERCENT_CPU`:
![Вывод команды htop, отсортированный по PERCENT_CPU](screen/part_9_htop_percent_cpu.png)

    + отсортированный по `PERCENT_MEM`:
![Вывод команды htop, отсортированный по PERCENT_MEM](screen/part_9_htop_percent_mem.png)

    + отсортированный по `TIME`:
![Вывод команды htop, отсортированный по TIME](screen/part_9_htop_time_sorted.png)

    Для фильтра нажимаем `F4`, вводим `sshd` + `ENTER`
    + отфильтрованный для процесса `sshd`:
![Вывод команды htop, отфильтрованный для sshd](screen/part_9_htop_sshd%20filtrated.png)

    Чтобы вывести `htop` с процессом `syslog` нажимаем `F3` для отображения строки поиска и вводим `syslog` + `ENTER`, система найдет и выделит процесс (если такой процесс запущен в системе); `syslog` собирает сообщения от различных компонентов системы (ошибки, предупреждения, информационные сообщения и отладочную информацию) и записывает их в лог-файлы 
    + с процессом syslog:
![Вывод команды htop с процессом syslog](screen/part_9_htop_sys_log.png)

    Чтобы вывести `htop` с добавленными полями `hostname`, `clock` и `uptime` нажимаем `F2` для отображения меню конфигурации; переходим в раздел `Setup`, выбираем подраздел `Meters`, перемещаемся стрелками направо в `Available meters`, выбираем нужный, нажимаем `ENTER`, перемещаем стрелками влево или вправо, чтобы разместить в левой или правой колонке (на скриншоте они расположены в левой колонке), для удаления используем `Delete`, для выхода – `F10` 
    + с добавленным выводом hostname, clock и uptime:
![Вывод команды htop с добавленным выводом hostname, clock и uptime](screen/part_9_htop_added_hostname_clock_uptime.png)

# Part 10. Использование утилиты **fdisk**
Утилита `fdisk` позволяет создавать, изменять, удалять и просматривать разделы на дисках

- Запускаем команду `fdisk -l`
![Вывод команды fdisk -l](screen)
    + Название жесткого диска:  /dev/sda
    + Размер жесткого диска: 25 ГБ
    + Количество секторов: 52428800
    + swap-раздел (используется для хранения данных, которые не умещаются в оперативной памяти) файл подкачки /swapfile размером 2 ГБ в данный момент не используется (USED равно 0B):
    ![вывод команды 'swapon --show'для инфо о swap](screen/part_10_swap.png)



## Part 11. Использование утилиты **df**
Утилита `df` (disk free) используется для вывода инфо о доступном месте на файловых системах – размер файловой системы, использованное пространство и т.д.
- Вывод команды `df`
![ Вывод команды 'df'](screen/part_11_df_output.png)
    + Размер раздела: 11758760 килобайт
    + Размер занятого пространства: 5350872 килобайт
    + Размер свободного пространства: 5788780 килобайт
    + Использовано 49%
- Вывод команды `df -Th`
![Вывод команды df -Th](screen/part_11_df-th.png)
    + Размер раздела: 12 Гб
    + Размер занятого пространства: 5.2 Гб
    + Размер свободного пространства: 5.6 Гб
    + Использовано 49%

## Part 12. Использование утилиты **du**
Утилита `du` (disk usage) предназначена для оценки использования дискового пространства файлами и директориями
- Вывод команды `du`
    
    ![Вывод команды du](screen/part_12_du_output.png)

- Вывод размера папок /home, /var, /var/log (в байтах, в человекочитаемом виде):

Вводим команды для каждой директории
`du -hs --bytes /home`
`du -hs --bytes /var`
`du -hs --bytes /var/log`
    * -h выводит в человекочитаемом формате
    * -s выводит только суммарный размер каждой указанной директории
    * --bytes выводит размер в байтах
![Вывод размера папок /home, /var, /var/log (в байтах, в человекочитаемом виде)](screen/part_12_du_for_home_var_varlog.png)

- Вывод размера каждого вложенного элемента в /var/log: `du -h /var/log/*` (убираем `--bytes`, но оставляем человекочитаемый формат, в итоге размер указан в килобайтах):
![Вывод размера каждого вложенного элемента в /var/log](screen/part_12_varlog_data.png)

## Part 13. Установка и использование утилиты **ncdu**
`ncdu` – это интерактивная командная утилита для анализа использования дискового пространства (позволяет быстро и удобно исследовать структуру каталогов и файлов на диске, оценить размер каждого элемента и удалить ненужные файлы прямо из интерфейса)

- Для установки утилиты используем команду `sudo apt install ncdu`
    + Вывод утилиты `ncdu` для каталога `/home`:
    ![Вывод утилиты ncdu для каталога /home](screen/part_13_ncdu_home.png)
    + Вывод утилиты `ncdu` для каталога `/var`:
    ![Вывод утилиты ncdu для каталога /var](screen/part_13_ncdu_var.png)
    + Вывод утилиты `ncdu` для каталога `/var/log`:
    ![Вывод утилиты ncdu для каталога /var/log](screen/part_13_ncdu_varlog.png)
    + Так как в [Part 12](#part-12-использование-утилиты-du) по заданию мы вычисляли размеры каталогов в байтах, на скрине ниже представлены размеры в K(KiB) и M(MiB) для сравнения с данными выше
    ![Вывод утилиты du для каталогов /home /var /var/log для сравнения с результатами ncdu](screen/part_13_check_results.png)

## Part 14. Работа с системными журналами
- Для открытия файлов для просмотра используем команду `cat`
- при выводе команды `/var/log/auth.log` видно 
* время последней успешной авторизации '21:08:42'
* имя пользователя user broderci
* uid = 0 показывает, что пользователь имеет права суперпользователя и вход был осуществлен при помощи логина
![вывод файла auth.log с данными времени последней авторизации, логином и способом входа](screen/part_14_auth_log.png)

- Перезапускаем службу `sshd` `sudo systemctl restart ssh`

- Часть файла `var/log/syslog` с сообщениями о перезапуске службы SSHd:
![Часть вывода файла /var/log/syslog с сообщениями о перезапуске службы SSHd](screen/part_14_ssd_restart.png)

## Part 15. Использование планировщика заданий **CRON**
`CRON` - это системный планировщик задач, который позволяет выполнять заданные команды или скрипты автоматически в заданное время или с интервалом

- для создания нового файла расписания CRON, который будет запускать команду uptimeкаждые две минуты используем команду `crontab -e`. После этого в открывшемся редакторе пишем строку `*/2 * * * * uptime 
Эта строка означает, что команда uptime будет выполняться каждые 2 минуты в любое время и день недели
![содержание файла CRON](screen/part_15_cron_file.png)

- еще раз запускаем команду `var/log/syslog` для просмотра выволнения команды `uptime`:
![Часть вывода файла /var/log/syslog с сообщениями о выволнении команды `uptime](screen/part_15_uptime_log.png)

- для вывода списка текущих заданий для CRON `crontab -l`:
    ![Вывод списка текущих заданий для CRON](screen/part_15_cron_tasks.png)

- для удаления всех заданий из планировщика заданий `crontab -r` и вывода списка текущих задач :
![Вывод списка текущих заданий для CRON после удаления всех задач](screen/part_15_tasks_cleanup.png)