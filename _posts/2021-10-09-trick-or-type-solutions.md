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

