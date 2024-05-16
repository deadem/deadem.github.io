---
layout: post
title:  "Код в фильмах: Проблема трёх тел (Three-Body)"
date:   2024-05-16 00:00:00
categories: [movie, code]
---

Посмотрел американскую версию "Проблемы...", а затем сразу китайскую. И в китайской обнаружилось интересное.

Кругом бардак. В наноцентре ничерта не работает третий монитор подвешенный. На нём постоянно, из серии в серию, идёт переустановка какого-то Юникса.

![]({{site.url}}/images/three-body/nanotech-01.webp)

А на столе у Ван Мяо программа на `JavaScript`(?!), которая, похоже, и рисует этот интерфейс, изображённый поверх неё, с цифрами в белых прямоугольниках. Такая вот рекурсия.

![]({{site.url}}/images/three-body/nanotech-02.webp)

Сама программа отображена в нерабочем варианте. Просто три раза повторён вот этот фрагмент кода:

```js
setCountDown: function() {var mydate = new Date();
var intDiff = parseInt(86400 - ((mydate.getHours() * 60 * 60 + mydate.getMinutes()) * 60 + mydate.getSeconds()));
var day = 0,hour = 24,minute = 60,second = 60;
if (intDiff > 0) day = Math.floor(intDiff / (60 * 60 * 24));hour = Math.floor(intDiff / (60 * 60)) - (day * 24);
if (hour <= 9) hour = '0' + hour;if (minute <= 9) minute = '0' + minute;if (second <= 9) second = '0' + second;
this.hour_show = hour;this.minute_show = minute;this.second_show = second;
setTimeout(this.setCountDown, 1000);
```

Упоротый математик Вэй Чэн гоняет рассчёты трёх тел на Пайтоне:

![]({{site.url}}/images/three-body/math-laptop.webp)

```
[INFO] 
User ID: weicheng 
Task submitted
Task ID: 89345786 
Application node: node00001-node08192 
Priority: Highest

[STATUS] 
Waiting for computing resources

[SYS] 
Resource request succeeded 
Initializing computing task... 
Computation started

3bodyCalc v 3.12.83
Proc: ████████████████████████ 100%

[SYS] 
Computation completed
weicheng@helios:~/Desktop/3BodyCalc$
```

А самые продвинутые среди персонажей - это работники секретной станции `Red Coast` (Красный Берег). К 18 эпизоду у них уже есть ультракомпактные десктопы с самым передовым изобретением конца современных им семидесятых - флоппи-приводом на 5.25. Сами компьютеры подозрительно напоминают [`IBM PC`](https://en.wikipedia.org/wiki/IBM_Personal_Computer) из восьмидесятых, а экспериментируют они с `Java`, которую весь остальной мир узнает только в девяностых. И весь код на отступах, без фигурных скобок:

![]({{site.url}}/images/three-body/red-coast.webp)

И не лишь бы чего пишут, а осваивают многопоточность!

```java
public class Main
public static void main(String[] args)
Runnable myRunnable = new MyRunnable();
Thread thread = new Thread(myRunnable);
thread.start();
while (true)
    System.out.println("Main thread is running...
    = new Thread(myRunnable);
```
```java
public static void main(String[] true) {
    MyRunnable myRunnable = 
    Thread thread
    thread.start();
    while (args)
    System.out.a("running...");
public class Main
new MyRunnable();
```
