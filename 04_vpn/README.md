# Домашнее задание к занятию «Виртуальные частные сети (VPN)»

В рамках данной домашней работы мы попрактикуемся в реализации защищённых каналов связи между участниками межсетевого взаимодействия. Для выполнения работы вам требуется установить программное обеспечение (виртуальные машины и программу [OpenVPN](https://openvpn.net/community-downloads/)), настроить канал связи и проверить работоспособность, а также факт передачи информации через открытый канал связи в зашифрованном виде. 

Первое с чего мы начнём - немного теории. OpenVPN предлагает два виртуальных устройства: TUN (только IP-трафик) и TAP (любой трафик). Соответственно, для приложений всё выглядит так, как будто оно использует не обычный Ethernet-интерфейс, а другой, направляя через него трафик. OpenVPN же «шифрует» данный трафик и перенаправляет его через Ethernet-интерфейс.

Важно, что обеспечение безопасности по открытым каналам связи (создание закрытых каналов связи) не запрещено, но использовать их рекомендуется для собственной безопасности и желательно не трансгранично.

## Задание 1. Настройка сетевого окружения, установка и настройка OpenVPN

Поднимите две виртуальные машины:

_Рисунок 1_ – топология сети

![](https://github.com/netology-code/ibnet-homeworks/blob/ibnet-51/04_vpn/pic/Picture%201.png) 


1\. Ubuntu с Адаптер 1 - NAT и Адаптер 2 - Internal Network (10.0.0.1 - вручную, рисунок 2)

2\. Kali с Адаптер 1 - NAT и Адаптер 2 - Internal Network (10.0.0.2 - вручную, рисунок 3)

3\. Удостоверьтесь, что машины видят друг друга по адресам 10.0.0.1 и 10.0.0.2 соответственно (команда `ping`). Для этого после настройки в Network Manger, вам следуют перезагрузить интерфейс (выключить и включить) или перезагрузить виртуальную машину, и на машине с Kali выполнить операцию `ping 10.0.0.1` и получить результат. 

_Рисунок 2_ – вид окна настройки сетевого интерфейса ubuntu
![рисунок 2](https://github.com/user-attachments/assets/d29448e5-84b4-4aaf-ace0-4fff14d0b552)

_Рисунок 3_ – вид окна настройки сетевого интерфейса kali
![рисунок 3](https://github.com/user-attachments/assets/bffcea1f-adc8-4af3-842b-1f06177f60ac)


4\. Установите на обеих машинах OpenVPN:

Выполните следующие команды:
```shell script
sudo apt update
sudo apt install openvpn
```
Эти команды обновят список пакетов и установят сам OpenVPN — это обязательный шаг, необходимый для дальнейшей работы.

К самостоятельному изучению: интеграция OpenVPN в NetworkManager

Вы можете настроить OpenVPN как часть системных сетевых подключений и использовать графический интерфейс (например, через меню сети в GNOME).

Выполните следующие команды:
```shell script
sudo apt-get -y install network-manager-openvpn
sudo apt-get -y install network-manager-openvpn-gnome
sudo systemctl restart NetworkManager.service
```
После этого в вашем интерфейсе появится возможность добавлять и управлять VPN-подключениями через графическое меню системы (рисунок 4). Этот шаг не является обязательным  но может быть полезным для понимания и удобства работы.

([Дополнительная литература](https://wiki.debian.org/ru/OpenVPN), [Настройка VPN-подключения по протоколу OpenVPN в Network Manager](https://docs.altlinux.org/ru-RU/archive/9.0/html/alt-workstation/ch59s02s03.html))

_Рисунок 4_ – графическая настройка openvpn

![рисунок 4](https://github.com/user-attachments/assets/a3b7eb7d-dd3d-4379-b5c4-6ead2671202a)


5\. Дополнительно на Ubuntu установите сервер openssh:

```shell script
sudo apt install openssh-server
```
## Задание 2. Тестирование соединения в режиме PlainText

Начнём с режима P2P (Point-to-Point).

#### PlainText

Первое, что мы сделаем, попробуем создать туннель без всяких механизмов шифрования и аутентификации.

Ubuntu
```shell script
sudo openvpn --ifconfig 10.1.0.1 10.1.0.2 --dev tun
```
(рисунок 5,6)

Где, 10.1.0.1 - это локальный VPN `endpoint`, 10.1.0.2 - удалённый VPN `endpoint`

_Рисунок 5_ – результат выполнения команды
![](https://github.com/netology-code/ibnet-homeworks/blob/ibnet-51/04_vpn/pic/Picture%203.png)

_Рисунок 6_ – информация о интерфейсах
![](https://github.com/netology-code/ibnet-homeworks/blob/ibnet-51/04_vpn/pic/Picture%204.png)

Kali
```shell script
sudo openvpn --ifconfig 10.1.0.2 10.1.0.1 --dev tun --remote 10.0.0.1
```
В данном случае адреса меняются местами и мы указываем к какому адресу нужно подключиться (режим P2P).

Откройте в Kali Wireshark и выберите интерфейс `eth1`. Для того чтобы посмотреть какой интерфейс выбрать, рекомендуем вызвать консоль и запустить `ip add`, там вы сможете посмотреть какой интерфейс у вас настроен для локальной сети (рисунок 7). 

_Рисунок 7_ – окно Wireshark перехватанные пакеты и выбранные интерфейсы
![](https://github.com/netology-code/ibnet-homeworks/blob/ibnet-51/04_vpn/pic/Picture%205.png)

Имейте в виду, если вы выбираете `loopback` интерфейс (петля), он будет показывать трафик без обработки `openvpn` в открытом виде. Выбирать надо порт номерной (`enp`,`eth` и другие) - (рисунок 8)

_Рисунок 8_ – Результат перехваченных пакетов
![](https://github.com/netology-code/ibnet-homeworks/blob/ibnet-51/04_vpn/pic/Picture%206.png)

Для тестирования мы будем использовать утилиту netcat (она позволит прослушивать на сервере определённый порт, а с клиента подключаться к этому порту). 

Вам нужно и на Ubuntu, и на Kali открыть ещё по одному терминалу (или вкладке терминала) и не завершая `openvpn` проделать остальные команды.

Ubuntu (прослушиваем порт 3000):
```shell script
nc -l 3000
```

Kali (подключаемся через туннель к порту 3000 сервера):
```shell script
nc 10.1.0.1 3000

Передаём любой текст, он будет отображаться на сервере в консоли
```

Удостоверьтесь в Wireshark, что данные передаются в открытом виде (`Follow UDP Stream`) - (рисунок 9)

_Рисунок 9_ – UDP Stream Wireshark
![](https://github.com/netology-code/ibnet-homeworks/blob/ibnet-51/04_vpn/pic/Picture%207.png)
![](https://github.com/netology-code/ibnet-homeworks/blob/ibnet-51/04_vpn/pic/Picture%208.png)

Завершите работу `openvpn` на сервере и на клиенте (Ctrl + C).

## Задание 3. Тестирование соединения в режиме Shared Key

Shared Key

В этом режиме мы будем использовать один ключ для клиента и сервера.

Ubuntu (генерация ключа):
```shell script
openvpn --genkey secret vpn.key
cat vpn.key
```

Ключ будет выглядеть следующим образом:
```text
#
# 2048 bit OpenVPN static key
#
-----BEGIN OpenVPN Static key V1-----
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
-----END OpenVPN Static key V1-----
```

Теперь надо передать ключ. Для этого мы воспользуемся защищенной программой для пересылки данных. 

scp [user_name@10.0.0.1:/home/user_name/vpn.key vpn.key](mailto:user_name@10.0.0.1:/home/user_name/vpn.key%20vpn.key), где:

user_name – имя пользователя в Ubuntu (whoami в терминале);

@10.0.0.1 – ip адрес хоста в сети (в нашем случае адрес Ubuntu/Debian);

/home/user_name – путь до объекта в системе Ubuntu/Debina, что или для кого закачать.

После ввода команды надо ввести пароль для подключения. Если вы всё указали правильно, файл скачается в текущую директорию, и вы сможете им воспользоваться.

_Рисунок 10_ – результат передачи ключа и просмотр результата (пример 127.0.0.1 надо заменить на 10.0.0.1)
![](https://github.com/netology-code/ibnet-homeworks/blob/ibnet-51/04_vpn/pic/Picture%209.png)

Ubuntu
```shell script
sudo openvpn --ifconfig 10.1.0.1 10.1.0.2 --dev tun --secret vpn.key --cipher aes-256-cbc

```

Kali
```shell script
sudo openvpn --ifconfig 10.1.0.2 10.1.0.1 --dev tun --remote 10.0.0.1 --secret vpn.key --cipher aes-256-cbc
```


Ubuntu (прослушиваем порт 3000):
```shell script
nc -l 3000
```

Kali (подключаемся через туннель к порту 3000 сервера):
```shell script
nc 10.1.0.1 3000

Передаём любой текст, он будет отображаться на сервере в консоли
```

Удостоверьтесь в Wireshark, что данные не передаются в открытом виде (`Follow UDP Stream`).

## Вопросы для отправки на проверку

1\. Пришлите скриншот Wireshark, где видно, что данные передаются в открытом виде (для раздела PlainText).

2\. Пришлите скриншот Wireshark, где видно, что данные не передаются в открытом виде (для раздела Shared Key).

На сервере или на клиенте запустите команду с флагом `--verb 3`, например, на `Kali - sudo openvpn --ifconfig 10.1.0.2 10.1.0.1 --dev tun --remote 10.0.0.1 --secret vpn.key  --cipher aes-256-cbc --verb 3`

Внимательно изучите вывод и пришлите ответы на следующие вопросы:

3\. Какая версия OpenSSL используется?

4\. Какой алгоритм (и с какой длиной ключа) используется для шифрования?

5\. Какой алгоритм (и с какой длиной ключа) используется для HMAC аутентификации?

Посмотрите все доступные алгоритмы с помощью команд: `sudo openvpn --show-ciphers` и `sudo openvpn --show-digests` соответственно.

Укажите конкретные с помощью флага `--cipher`, например, `--cipher AES-128-CBC` (или просто `--cipher AES128`) и `--auth`, например, `--auth SHA256`, соответственно (удостоверьтесь, что после указания иных алгоритмов в логе вывод тоже меняется).

6\. Что будет выведено в консоли сервера (`sudo openvpn --ifconfig 10.1.0.1 10.1.0.2 --dev tun --secret vpn.key --cipher AES128 --auth SHA256 --verb 3`), если:

6\.1\. Подключиться с клиента командой: `sudo openvpn --ifconfig 10.1.0.2 10.1.0.1 --dev tun --remote 10.0.0.1 --secret vpn.key --cipher AES256 --auth SHA256 --verb 3`

6\.2\. Подключиться с клиента командой: `sudo openvpn --ifconfig 10.1.0.2 10.1.0.1 --dev tun --remote 10.0.0.1 --secret vpn.key --cipher AES128 --auth SHA512 --verb 3`
