---
layout: post
title:  "Завершающая точка в имени домена"
date:   2022-11-29 00:00:00 -0400
categories: [code]
---

Шикарная заметка от [Julia Evans](https://jvns.ca/blog/2022/09/12/why-do-domain-names-end-with-a-dot-/), которой не могу не поделиться.

# Почему доменные имена иногда заканчиваются на точку?

Привет! Когда я писала заметку в журнал о том, как работает DNS, кто-то спросил меня: "А зачем иногда ставят точку в конце доменного имени?" В самом деле, если получить IP-адрес через dig для example.com результат будет таким:

```s
$ dig example.com
example.com.		5678	IN	A	93.184.216.34
```

dig добавил точку в конец example.com, и теперь это "example.com."! Но зачем?

А некоторые DNS-утилиты требуют указания точки в конце домена. Если попытаться передать "example.com" в miekg/dns будет ошибка:

```s
// trying to send this message will return an error
m := new(dns.Msg)
m.SetQuestion("example.com", dns.TypeA)
```

Я думала, что знаю ответ на этот вопрос, и что точка в конце означает, что имя домена указано полностью. И это так: полное имя домена действительно завершается точкой.

Но это не объясняет в чём польза или важность точек в конце адреса.

## В DNS-запросах домены не заканчиваются точкой

Когда-то я думала (неправильно), что точки в конце нужны потому, что это вот такой формат работы DNS, и запросы-ответы к нему должны иметь точки в конце адреса. Поэтому мы их и добавляем, чтобы корректно работать с DNS.

Но на самом деле, как оказалось, в запросах-ответах от DNS нет точек. Вообще нет. Ни одной.

Вместо точек домены кодируются сериями пар длина-строка. Например, домен "example.com" кодируется в 13 байт: `7example3com0`

Ни одной точки. Доменное имя (типа "example.com") преобразуется вот в такой формат запросов для работы с DNS.

Теперь поговорим о файлах зон, которые используются для формирования ответов в DNS.

## Завершающая точка в файлах доменных зон

Один из способов редактирования DNS-записей, это создать файл зоны ("zone file") и настроить DNS-сервер (типа `nsd` или `bind`) на обслуживание записей, указанных в этом файле.

Представим вот такой файл зоны для example.com:

```s
orange  300   IN    A     1.2.3.4
fruit   300   IN    CNAME orange
grape   3000  IN    CNAME example.com.
```

В этом файле зоны ко всему, что не заканчивается на точку, будет добавлен суффикс ".example.com". Из "orange", например, получится "orange.example.com". DNS-сервер знает, что это файл зоны для домена "example.com", так что он автоматически добавляет этот путь в конец адреса, не завершающегося точкой.

Думаю, что идея тут была в том, чтобы уменьшить объём набираемого текста. Представьте, что в этом файле зоны все адреса указаны полностью:

```s
orange.example.com.  300   IN    A     1.2.3.4
fruit.example.com.   300   IN    CNAME orange.example.com.
grape.example.com.   3000  IN    CNAME example.com.
```

Многабукоф.

## Для работы с DNS не нужны файлы зон

Несмотря на то, что формат файла зон закреплён в DNS RFC (1035), вам он может никогда не понадобиться для настройки DNS. Например, сервис `AWS Route 53` не использует файлы зон для хранения DNS-записей. Вы редактируете их через веб-интерфейс и я уверена, что эти данные хранятся в базе данных, а в не в куче текстовых файлов.

Но, конечно, `Route 53` (как и многие другие DNS-утилиты) поддерживает импорт и экспорт файлов зон, потому, что это понятный способ для миграции DNS-записей от одного провайдера к другому.

## Завершающая точка в dig

Рассмотрим такой вывод dig:

```s
$ dig example.com
; <<>> DiG 9.18.1-1ubuntu1.1-Ubuntu <<>> +all example.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 10712
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;example.com.			IN	A

;; ANSWER SECTION:
example.com.		81239	IN	A	93.184.216.34
```

Налицо странная вещь: практически каждая строка начинается с ";;". Что это? А вот что: точка с запятой - это символ начала комментария в файлах зон!

Я думаю, что такой странный вывод у dig нужен для того, чтобы его весь можно смело копировать, вставлять в файл зоны и всё бы работало без каких-либо изменений.

Это объясняет и завершающую точку в конце "example.com." - файл зоны требует завершающей точки в конце домена, чтобы интерпретировать его как абсолютный, а не относительный адрес. Так что и dig делает то же самое.

А ещё мне бы очень хотелось, чтобы в dig был флаг "+human", который бы печатал всю эту информацию в более человекопригодном виде, но я слишком ленива чтобы законтрибутить такой код (а ещё я плохой программист на C), так что я просто побухчу в бложике. :)

## Завершающая точка в curl

Посмотрим на ещё один пример, где возникает завершающая точка: curl!

Один из компьютеров у меня дома называется "grapefruit" и на нём запущен вебсервер. Вот что происходит, когда я запускаю "curl grapefruit":

```s
$ curl grapefruit
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">

<html>
<head>
```

Работает! Круто! Но что, если добавить точку в конец? Работать перестаёт:

```s
$ curl grapefruit.
curl: (6) Could not resolve host: grapefruit.
```

Почему? Чтобы разобраться, нужно рассказать про домены поиска.

## Встречаем! Домены поиска

Когда я запускаю "curl grapefruit", как оно транслируется в DNS-запрос? Думаете, что мой компьютер отправляет запрос за реквизитами домена `grapefruit`, да? Но нет.

Запустим tcpdump, чтобы посмотреть, что именно происходит, и увидим вот это:

```s
$ sudo tcpdump -i any port 53
[...] A? grapefruit.lan. (32)
```

Отправляется запрос к `grapefruit.lan`. Но как?

Вот что произошло: чтобы найти адрес домена для "grapefruit" curl вызвал функцию `getaddrinfo`. `getaddrinfo` заглянул в файл `/etc/resolv.conf`. А в `/etc/resolv.conf` прописаны такие строки:

```s
nameserver 127.0.0.53
search lan
```

В конфигурации указано `search lan`, вот `getaddrinfo` и дописал `lan` в конец адреса, так что адрес превратился в `grapefruit.lan`.

## Когда используются домены поиска

Налицо непонятное. Когда мы ищем домен, иногда что-то дополнительное (типа `lan`) добавляется в конец адреса. Но когда?

* Если имя домена заканчивается на точку (типа `curl grapefruit.`), то поиск не используется.
* Если внутри имени домена есть точка (типа `example.com`), домены поиска тоже по умолчанию не используется, но это можно настроить: см. параметр `ndots`.

Теперь понятно, почему отличается результат `curl grapefruit.` и `curl grapefruit` - потому, что первый пошёл к домену `grapefruit.`, а второй нашёл `grapefruit.lan.`

## Но как МОЙ компьютер узнаёт, что нужно использовать поисковый домен?

Когда компьютер устанавливает соединение с маршрутизатором, он получает имя для доменного поиска через `DHCP`, тем же способом как назначается IP-адрес.

## Так зачем же люди пишут точку в конце доменного имени?

Теперь мы знаем про файлы зон и поисковые домены, и это те причины, по которым, я думаю, люди пишут точки в конце доменного имени.

Возможны две ситуации, когда имена доменов изменяются и что-то добавляется в конец:

 - в файле зоны, например, для `example.com`, `grapefruit` преобразуется в `grapefruit.example.com`
 - в локальной сети (когда компьютер сконфигурирован использовать домен поиска), `grapefruit` преобразуется в `grapefruit.lan`

Из-за того, что введённое имя домена в некоторых случаях может быть преобразовано во что-то ещё, люди дописывают точку, чтобы сказать: "ЭТО ИМЯ ДОМЕНА, НИЧЕГО НЕ НАДО ДОБАВЛЯТЬ В КОНЕЦ, ЭТО ДОМЕННОЕ ИМЯ ПОЛНОСТЬЮ". Потому, что иначе могут возникнуть разночтения.

Говоря техническим языком, фраза "ЭТО ДОМЕННОЕ ИМЯ ПОЛНОСТЬЮ" обозначается "fully qualified domain name" (полностью определённое имя домена) или "FQDN". Таким образом `google.com.` - это полное имя домена, в то время как `google.com` - нет.

Я редко использую файлы зон и домены поиска, и иногда приходится вспоминать это вот всё, но, честно говоря, чаще хочется просто закричать: "Конечно, я имела в виду "google.com", а не "google.com.something.else"! Зачем мне хотеть что-то другое? Это же тупо!"

Но людям, которые сталкиваются с файлами зон и доменами поиска (например в `Kubernetes`), эта точка в конце важна, чтобы на 100% быть уверенными, что ничего дополнительного не будет добавлено.

В 2009 году такая пропущенная точка в конце доменной зоны ".se" [привела к отключению Швеции от интернета](https://web.archive.org/web/20091017025605/http://www.theinquirer.net/inquirer/news/1558515/dns-screwup-knocks-sweden-net)

## Когда же писать точки в конце?

**Да**: при конфигурировании DNS.

Нет ничего плохого в том, чтобы использовать полные доменные имена в конфигурации DNS. Это не всегда требуется и иногда всё работает и без точки, но я никогда не встречала ни одной программы для DNS, которая бы не понимала полное имя домена.

А некоторый софт для DNS требует точек и отказывается работать в противном случае, дописывая имя домена в конец. Мне не нравится дизайн этого решения, но это не такая уж и проблема, просто добавить точку в конце.

**Нет**: в браузерах.

Удивительно, но некоторые сайты не будут работать, если вы укажете полное имя домена в адресной строке браузера. Если вбить в браузер `https://twitter.com.`, получим ошибку 404.

Думаю, что это из-за заголовка HTTP-заголовка `Host`, устанавливаемого в `Host: twitter.com.`, в то время как обслуживающий сервер настроен отвечать только на `Host: twitter.com`.

## Думаю, что относительные доменные имена раньше использовались гораздо шире

Напоследок: я думаю, что "относительные" доменные имена (типа `grapefruit`, который я использую для домашнего доступа к `grapefruit.lan`) использовались намного более широко, так как DNS разрабатывался в контексте университетов с большими внутренними подсетями.

В сегодняшнем же интернете более привычны "абсолютные" доменные имена (типа `example.com`).
