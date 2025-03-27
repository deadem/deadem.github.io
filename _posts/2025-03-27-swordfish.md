---
layout: post
title:  "Код в фильмах: Пароль «Рыба-меч» (Swordfish)"
date:   2025-03-27 00:00:00
categories: [movie, code]
---

![]({{site.url}}/images/swordfish/image_000.webp)

И снова про код. Сразу скажу, шифрования касаться не буду. Ибо герои фильма несут на эту тему какую-то лютую дичь, а на экране никакого шифрования/дешифрования не происходит. Но зато мелькают любопытные фрагменты кода.

Знаменитая эпичная сцена. "Взломать за 60 секунд". Получить доступ к какому-то-там ресурсу минобороны. На экране мелко: `DES 128 bit`. (Алгоритмы шифрования будут упоминаться в том или ином виде весь фильм).

![]({{site.url}}/images/swordfish/image_001.webp)

Приступаем... На экране какой-то компилятор (не просто же так надпись `Compiler` в левом верхнем углу?), и, якобы пишем код.

На экране код из книги `Cracking DES`, 1998. Фрагмент из комплекта программ по демонстрации уязвимости алгоритма шифрования `DES`. `Search Engine Controller Program`, написана Полом Кохером.

![]({{site.url}}/images/swordfish/image_002.webp)

Правда, код в фильма набран такой гарнитурой, что на первый взгляд кажется, что это какая-то нерабочая ерунда. Но нет. Вот оригинал:

![]({{site.url}}/images/swordfish/image_002-1.webp)

Продолжаем ультра-взлом за 60 секунд. Теперь на экране список самых частоупотребляемых логинов и паролей из руководства по хакингу 1993 года: "`THE NEOPHYTE'S GUIDE TO HACKING`".

![]({{site.url}}/images/swordfish/image_003.webp)

```
COMMON ACCOUNTS:
Username    Password
--------    --------
root
daemon
adm         admin, sysadm, sysadmin, operator, manager
uucp
bin
sys
123         lotus, lotus123
adduser
admin       adm,sysadm,sysadmin,operator,manager
```

Дальше на экране мелькают числа. К чему они относятся - непонятно. Возможно, стилизация, демонстрирующая процесс хеширования/шифрования пароля. Но, время вышло, а у героя всё получилось: `ACCESS GRANTED`. Ноутбук у него отнимают.

![]({{site.url}}/images/swordfish/image_005.webp)

Квалификация проверена, через какое-то время можно и выдать настоящий компьютер. Нечто крутое, семиэкранное, с декоративной подсветкой и всё такое. На котором нужно написать червя. Да не просто червя, а "многоголовую гидру".

![]({{site.url}}/images/swordfish/image_006.webp)

Благо, писать придётся не с нуля: есть тайная заначка, оставленная в КалТехе на забытом в подвале `PDP-10`, который до сих пор подключён к сети. Кстати, прикольные там у них браузеры:

![]({{site.url}}/images/swordfish/image_007.webp)

Однако, требуется логин. Но, это, понятное дело, не проблема. Процесса не покажут, но зато похвастаются логотипом ещё одного алгоритма шифрования. На этот раз асимметричного: 128-битный `RSA`.

![]({{site.url}}/images/swordfish/image_008.webp)

Начинается работа над червём. На экране - листинг перлового исходника. Фрагмент `cmount.pl` из испанского гайда по хакингу: `INTRODUCCION AL HACKING v2.0`, 1998.

Небольшой скрипт для получения списка доступных точек доступа в `NFS`-серверах. Типа, сканируем сеть, ищем открытые пути. На экран исходник влез почти целиком:

![]({{site.url}}/images/swordfish/image_009.webp)

```pl
# Assign null list to @URLs which will be added to later.
my(@result) = ();
my(@domains) = ();
my($program) = "showmount -e ";

# Pull off filename from commandline. If it isn't defined, then assign default.
my($DomainFilename) = shift;
$DomainFilename = "domains" if !defined($DomainFilename);

# Do checking on input.
die("mountDomains: $DomainFilename is a directory.\n") if (-d $DomainFilename);

# Open $DomainFilename.
open(DOMAINFILE, $DomainFilename) or
  die("mountDomains: Cannot open $DomainFilename for input.\n");

while (<DOMAINFILE>) {
  chomp($_);
  print "Now checking: $_";

  # Note difference in program output capture from "geturl.pl".
  open (EXECFILE, "$program $_ |");
  @execResult = <EXECFILE>;
  next if (!defined($execResult[0]));
  if ($execResult[0] =~ /^Export/) {
    print " - Export list saved.";
    open (OUTFILE, ">$_.export");
    foreach (@execResult) {
        print OUTFILE;
    }
    close (OUTFILE);
  }
  close(EXECFILE);
  print "\n";
}

# We are done. Close all files and end the program.
close (DOMAINFILE);
0;
```

А следом за ним экран "Connection routing". Герою обещали, что "можно одновременно подключиться к семи разным сетям". Вот. На выбор. Это перепечатка таблицы региональных X.25 подсетей из "`The Hackers' Handbook`", 1985, Хьюго Корнуолл, стр. 71. Точнее, первые её 18 строк.

![]({{site.url}}/images/swordfish/image_010.webp)

А, это, видимо, рабочий интерфейс. С копирайтом, как полагается: `Worm generator tool v.1.2 (Jobson, S.)`.

Занятная опечатка в команде. Интересно, специально? `cat /ect/passwrd` (должно быть `cat /etc/passwd`)

Справа фрагмент кода на чём-то перлоподобном, которым разбираются строки: пропускаем `#` (видимо, комментарии), и бьём на фрагменты по точке или `~`. А судя по тому, что код с теми же переменными встречается и в куске листинга слева и перемешан с дампом `passwd`, этот экран, похоже, иллюстрирует работу сетевого сниффера: слушаем сетевую активность и следим за ней, вываливая на экраны пакеты в порядке захвата:

![]({{site.url}}/images/swordfish/image_012.webp)

Запускаем сборку:

![]({{site.url}}/images/swordfish/image_013.webp)

Ещё один фрагмент из уже упоминавшейся `INTRODUCCION AL HACKING v2.0`, 1998. Теперь это - листинг утилиты `getdomain.pl`: парсинг списка записей о доменах с `rs.internic.net`. Да, в те времена можно было скачать всю базу доменов одним архивом. (В наши дни доменами теперь занимается `ICANN`) Список записей затем скармливается сканеру `cmount.pl`. Помните скриншот первого листинга с сервера КалТеха? Вот ему.

![]({{site.url}}/images/swordfish/image_014.webp)

```perl
#!/usr/bin/perl
# GetDomain By Nfin8 / Invisible Evil
# Questions /msg i-e or /msg i^e
#
# Retrieve command line arguments.
my($inputfile, $domain) = @ARGV;
usage() if (!defined($inputfile) || !defined($domain));

# Open and preprocess the input file.
open(INFILE, "<$inputfile") or die("Cannot open file $inputfile for reading!\n");
my(@lines) = <INFILE>;

# Initialize main data structure.
my(%hash) = {};
my($key) = "";

foreach (@lines) {
$key = (split(/\ /))[0];
chop($key);
next if ((($key =~ tr/.//) < 1) ||
...
```

Ещё немного красочного сканирования:

![]({{site.url}}/images/swordfish/image_015.webp)

Ещё один листинг. И опять из книги "`Cracking DES`", теперь это исходник на `VHDL`, языке моделирования электронных схем, для реализации алгоритма в железе на `ASIC` и прочих `FPGA`.

![]({{site.url}}/images/swordfish/image_016.webp)

`addr_key.vhd`, за авторством Тома Ву. На листинге часть аппаратной системы для взлома алгоритма `DES`, на то время основной американской системы шифрования.
![]({{site.url}}/images/swordfish/image_017.webp)

Ещё чуть-чуть... Готово!

![]({{site.url}}/images/swordfish/image_018.webp)

Дальше запускают червя, он гоняет деньги по миру туда-сюда... В общем, все счастливы, а больше никакого кода не покажут.

![]({{site.url}}/images/swordfish/image_019.webp)

Конец.
