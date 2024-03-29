---
layout: post
title:  "Код в фильмах: Неуязвимый (Invincible)"
date:   2023-06-07 00:00:00 -0400
categories: [movie, code]
---

![](/images/invincible.webp)

Второй эпизод первого сезона, примерно середина фильма. Сесил Стедман, глава Агентства Глобальной Защиты и тайный куратор всех действующих супергероев, объясняет Нолану и Марку боевую задачу: остановить космического негодяя, который направляется к Земле со стороны Марса.

![](/images/invincible-code-01.webp)

На экран с космическим кораблём зачем-то добавлен кусок кода. При этом, особого смысла в нём нет и в сюжет он никак не вписан: просто какой-то кусок кода. Беглое гугление показывает, что это кусок из старого ядра Линукса, файл `groups.c`: код для поддержки групп в правах доступа.

![](/images/invincible-code-02.webp)

Но есть нюанс: код немного отличается от [того, что находится в репозитории](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/kernel/groups.c?id=954f74bf45268bcee0af21b6393c9c8acca7e075):

```c
struct group_info init_groups = { .usage = ATOMIC_INIT(2) };

struct group_info *groups_alloc(int gidsetsize){
    struct group_info *group_info;
    int nblocks;
```

Для описания структур в Линуксе приняты [правила форматирования кода, основанные на K&R](https://en.wikipedia.org/wiki/Indentation_style#Variant:_Linux_kernel), и они никак не позволяют написать такую строку: `struct group_info *groups_alloc(int gidsetsize){`

Если в поиск вбить уже "неправильный" вариант, то концы приводят к сайту [hackertyper.com](https://hackertyper.com/) (и его аналог: [hackertyper.net](https://hackertyper.net/)) - почувствуй себя киношным хакером. 

Настоятельно рекомендую зайти на какой-нибудь из них и попробовать понажимать кнопки на клавиатуре: будет довольно быстро печататься "хакерский код" (на самом деле - этот самый кусочек из Линукса), а если несколько раз нажать на CAPS LOCK, то ещё и загорится надпись:

![](/images/invincible-code-03.webp)

Аутентичненько! Настолько, что в интернете люди начали [использовать этот фрагмент кода для троллинга](https://www.google.com/search?q=%22make+sure+we+always+allocate%22+ATOMIC_INIT). Эдакий хакерский рикролл.

Интересно, в результате это стёб Сесила над Ноланом и Марком, или авторов сериала над зрителями?
