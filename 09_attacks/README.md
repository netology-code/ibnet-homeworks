# Домашнее задание к занятию «Атаки»

В качестве результата пришлите ответы на вопросы в личном кабинете студента на сайте [netology.ru](https://netology.ru).

## Metasploitable

Скачайте и установите на виртуальную машину Metasploitable: https://sourceforge.net/projects/metasploitable/

Это типовая ОС для экспериментов в области информационно безопасности, с которой следует начать при анализе уязвимостей.

Просканируйте эту VM, используя nmap.

Попробуйте найти уязвимости, которым подвержена данная виртуальная машина.

Сами уязвимости можно поискать на сайте https://www.exploit-db.com/.

Для этого нужно в поиске ввести название сетевой службы, обнаруженной на атакуемой машине, и выбрать подходящие по версии уязвимости.

Ответьте на следующие вопросы:

1. Какие сетевые службы в ней разрешены?
1. Какие уязвимости были вами обнаружены (список со ссылками - достаточно 3х уязвимостей)

Пришлите ответы на вопросы в ЛК студента.

## SYN, FIN, Xmas, UDP

Проведите сканирование Metasploitable в режимах SYN, FIN, Xmas, UDP, запишите сеансы сканирования в Wireshark.

Ответьте в свободной форме на следующие вопросы:
1. Чем отличаются эти режимы сканирования с точки зрения сетевого трафика?
1. Как отвечает сервер?

Пришлите ответы на вопросы в ЛК студента.