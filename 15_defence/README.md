# Домашнее задание к занятию «Suricata»

В качестве результата пришлите ответы на вопросы в личном кабинете студента на сайте [netology.ru](https://netology.ru).

## Suricata

### Вводная

Мы уже посмотрели, как настраивать Suricata на лекции. Сейчас стоит задача выяснить, как она по умолчанию реагирует на попытки DoS'а (например, с помощью `ab`).

### Задача

1. Настройте suricata согласно материалам из лекций
1. Отключите fail2ban, если он у вас был включен
1. Уберите из nginx rate limit
1. Проведите с машины с Kali с помощью ab в трёх вариантах (последовательно):
    1. 10_000 запросов по 100 одновременно
    1. 10_000 запросов по 1000 одновременно
    1. 10_000 запросов по 10 одновременно

В качестве ответа пришлите описание реакции Suricata на подобное воздействие (п.4) в ЛК студента Нетологии.

<details>
<summary>Описание выполнения</summary>

У вас должно быть две виртуальные машины:
* Ubuntu с nginx (10.0.0.1)
* Kali (10.0.0.2)

У вас уже должен быть настроенный nginx, если вдруг так случилось, что его нет, то вернуться [к соответствующему уроку](../10_internet).

#### Ubuntu

1\. Удостоверяетесь, что сервис nginx запущен, порт 80 открыт.

2\. Если вы не устанавливали nginx, то:
```shell script
sudo apt update
sudo apt install nginx
```

3\. Редактируете конфигурацию `sudo mcedit /etc/nginx/sites-enabled/default`, чтобы она выглядела следующим образом:

![](pic/nolimit.png)

4\. Сохраняете изменения

5\. Проверяете корректность конфигурации `sudo nginx -t`

6\. Применяете конфигурацию `sudo nginx -s reload`

7\. Отключаете fail2ban с помощью команды `sudo systemctl stop fail2ban` (чтобы отключить полностью и он не запускался после перезагрузки машины, нужно дополнительно ввести команду `sudo systemctl disable fail2ban`)

8\. Устанавливаете suricata с помощью следующих команд:

```shell script
sudo apt install software-properties-common
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt update
sudo apt install suricata
sudo suricata-update
```

Примечание*: suricata-update скачивает набор правил от [ET Labs](https://rules.emergingthreats.net).

9\. Проверяете, что сервис запустился после установки

```shell script
sudo systemctl status suricata
```

10\. Меняете значение `EXTERNAL_NET` в `/etc/suricata/suricata.yaml` на `"any"`:

![](pic/suricata.png)

11\. Перезапускаете suricata:

```shell script
sudo systemctl restart suricata
```

12\. Смотрим доступные сетевые интерфейсы:

```shell script
ip addr show
```

Выбираем тот, на который у вас настроен ip 10.0.0.1 (в примере `enp0s8`):

![](pic/ipaddr.png)

13\. Запускаем suricata на выбранном интерфейсе:

```shell script
sudo suricata -c /etc/suricata/suricata.yaml -i enp0s8
```

14\. В другой консоли запускаете просмотр лога:

```shell script
sudo tail -f /var/log/suricata/fast.log
```

Полностью посмотреть лог вы можете с помощью уже знакомых вам инструментов, например, `sudo mcedit /var/log/suricata/fast.log` (хотя более правильным будет просмотр с помощью специальных pager'ов `sudo less /var/log/suricata/fast.log`, выход из которых осуществляется с помощью клавиши `q`).

#### Kali

Для генерации запросов мы будем использовать утилиту [Apache Benchmarks](https://httpd.apache.org/docs/2.4/programs/ab.html).

```shell script
ab --help
```

Если у вас выводится сообщение о том, что программа не найдена (`bash: ab command not found`), то установите её:
```shell script
sudo apt update
sudo apt install ab
```

Далее делаем 10_000 запросов по 100 одновременно:
```shell script
ab -n 10000 -c 100 http://netology.local
``` 

Далее делаем 10_000 запросов по 1000 одновременно:
```shell script
ab -n 10000 -c 1000 http://netology.local
``` 

Далее делаем 10_000 запросов по 10 одновременно:
```shell script
ab -n 10000 -c 10 http://netology.local
``` 

Опишите поведение suricata (сообщения, появляющиеся в log-файле) после:
1. 10_000 запросов по 100
1. 10_000 запросов по 1000
1. 10_000 запросов по 10

Ответ пришлите в свободной форме в ЛК студента Нетологии.

</details>

## Suricata Rules*

**Важно**: это бонусная задача, выполнять её не обязательно (её (не)выполнение не влияет на зачёт по ДЗ).

Если вы ставили nginx по умолчанию, так, как описано в ДЗ, то в логах suricata обнаружите запись:
```text
ET INFO Unconfigured nginx Access
```

Правила, которые мы использовали в предыдущем ДЗ, располагаются в файле `/var/lib/suricata/rules/suricata.rules`.

Найдите среди правил правило, которое отвечает за эту строку (`ET INFO Unconfigured nginx Access`) и кратко опишите:
1. На основании чего происходит срабатывание данного правила
1. Ваше предположение о том, на что оно направлено

Ответ пришлите в свободной форме в ЛК студента Нетологии.
