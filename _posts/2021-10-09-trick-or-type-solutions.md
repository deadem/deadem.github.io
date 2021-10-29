---
layout: post
title:  "Type | Treat 2021 - решения"
date:   2021-10-09 01:00:00 -0400
categories: [code, typescript]
---

![](/images/type-or-treat-2021.png)

Тут напишу свои решения задачек из челленджа `Type | Treat 2021`, там запрета на публикацию нет, а даже совсем наоборот, приветствуется.

[Ссылка на плейбук с задачками](https://www.typescriptlang.org/play?#gist/927ccc66ae3022dc64c4f650109b937a-1)

[Лента с анонсами и обсуждениями на dev.io](https://dev.to/typescript)

### Day 1 Challenges
#### Beginner/Learner Challenge

Не знал, что для того, чтобы тип списка (массива) стал туплом, нужно массив сделать readonly. Для этого нужно добавить `as const` описанию массива. Очень полезная фича. А дальше тривиально:
 
 ```typescript
const playlist = [
    "The Legend of Sleepy Hollow by The Monotones.mp3",
    "(It's a) Monster's Holiday by Buck Owens.mp3",
    "Bo Meets the Monster by Bo Diddley.mp3",
    "Purple People Eater Meets the Witch Doctor by The Big Bopper.mp3",
    "Screamin Ball (at Dracula Hall) by The DuPonts.mp3",
    "Batman, Wolfman, Frankenstein, or Dracula by The Diamonds.mp3",
    "Frankenstein Twist by The Crystals.mp3",
    "'Thriller' by Michael Jackson.mp3"
] as const;
...
function playSong(song: typeof playlist[number]) {
    api.play(song)
}
 ```

#### Intermediate/Advanced Challenge

Это, наоборот, было легко: постоянно приходится пользоваться `ReturnType`. Жаль, конечно, что дженерики в `Typescript` они не про вывод типа, как шаблоны в `C++`, а просто про ограничения. Не так красиво, как могло бы быть, но всё же не теряем типы:

```typescript
const createNewMask = (costume: ReturnType<typeof findOldCostume>) => {
...
const createBodyCamera = (costume: ReturnType<typeof createNewMask>) => {
```

### Day 2 Challenges
#### Beginner/Learner Challenge

Быстрое решение: добавить гард для типа, на который намекают в задании:

```typescript
type RipePumpkin = { color: string; soundWhenHit: "echo-y" }
...
function isRipe(pumpkin: Pumpkin): pumpkin is RipePumpkin {
```

Если по-нормальному, то нужно добавить типы и остальным тыковкам, но, глядя на цвета, совершенно непонятно, конечный ли это набор цветов или может быть другим, поэтому придётся типизировать и `pumpkins`:

```typescript
const pumpkins: Pumpkin[] = [
// ...
];
...
type UnderripePumpkin = { color: string; soundWhenHit: "dull thud" }
type RipePumpkin = { color: string; soundWhenHit: "echo-y" }
type OverripePumpkin = { color: string; soundWhenHit: "squishy" }
...
function isRipe(pumpkin: Pumpkin): pumpkin is RipePumpkin {
```

...и тут ещё есть ряд решений, которые зависят от того, конечен ли набор цветов и может ли спелая тыква быть зелёной. (Мой опыт говорит, что может). Но нет, не хочу все варианты решения писать для всех неоговоренных случаев.

#### Intermediate/Advanced Challenge

Просто написать класс с методами. Оооокей.

```typescript
class PunchMixer {
  private _punch: Punch = {
    flavour: '',
    ingredients: []
  };

  private isPunch(value: Punch | Punch['ingredients'][number]): value is Punch {
    return typeof value == 'object' && typeof (value as Punch)?.flavour == 'string';
  }

  get punch(): Punch {
    return this._punch;
  }
  set punch(value: Punch | string | Punch['ingredients'][number]) {
    if (this.isPunch(value)) {
      this._punch = value;
      return;
    }
    this._punch.ingredients.push(value);
  }
}
```

Теперь раскомментируем часть 2 внизу...

И сразу оно ломается. Во-первых, хочет дженерик. Ннну, добавим. Только что это за тип - непонятно. Думал, что это заместо `Punch`, но как-то странно тогда первая часть получается. И логики между первым пуншем и типами не нашёл. Прогнул было, чтобы и оно и так и так работало. Такой ужас получился, что жуть. Открыл ответ. Лол! Вообще, оказывается, выдумывать ничего не надо. И где написано glass.size и cup.color - это не значит, что ждут, что я верну значение. А просто типы должны совпасть. Нууу, так легко:

```typescript
class PunchMixer<T> {
// ...
  public vend(): T {
    return {} as T;
  }
}
```

Буду иметь в виду, на будущее, что работать оно не должно, а мы только типы проверяем.

### Day 3 Challenges
#### Beginner/Learner Challenge

Тут понадобились типы на основе литералов. Это я помню - совсем недавно в тайпскрипт их добавили:

```typescript
type Length = `${number}${'in' | 'cm'}` | '0'
```

А теперь "злая" часть простого задания. Говорят, что это `secret *experts* section`. Ну, поглядим...
Жаль, рекурсивных типов не завезли, а то можно было бы адекватный парсинг написать. 

Ну, чтобы тесты пройти, можно так, хотя тут есть "неправильные" кейсы, которые пройдут. Но по итогам прошлого задания не буду заморачиваться тут.

```typescript
type Digit = '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9'
type ValidNumber = Digit | `${Digit}${number}`
type Length = `${ValidNumber}${'' | ' '}${'in' | 'cm'}` | '0'
```
Посмотрел ответ: пффф! Мой вариант и есть тот самый "Very cool solution".

#### Intermediate/Advanced Challenge

И это тоже было просто - новая фича, совсем недавно про неё в анонсах читал:

```typescript
function makeTitle<T extends string>(str: T) {
    return "<spooky>" + str.toUpperCase() + "</spooky>" as `<spooky>${Uppercase<T>}</spooky>`
}
```

А следующую без подсказки не догадался как раскручивать. Прикольно, что литералы могут и обратно в типы разбираться:

```typescript
function setupFooter<T1 extends string, T2 extends string, T3 extends string>(str: `${T1},${T2},${T3}`) {
    return { 
        name: str.split(",")[0] as T1,
        date: str.split(",")[1] as T2,
        address: str.split(",")[2] as T3
    }
}
```

### Day 4 Challenges
#### Beginner/Learner Challenge

Первая часть уже была в предыдущих днях. Странно, что повторяется. Но ок.

```typescript
function getBowl<T>(items: T) {
    return { items }
}
```

А тут есть продолжение...

Ну, несильно сложнее:

```typescript
function fillBowl<T extends string>(candy: T) {
    return { candy }
}
```
