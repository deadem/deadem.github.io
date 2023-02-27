---
layout: post
title:  "Type challenges"
date:   2023-02-26 00:00:00 -0400
categories: [typescript, code]
---

На Ютубе наткнулся на видосик, где человек разбирал какую-то задачку из челленджа. Заинтересовался, что это такое за челлендж, и сам того не заметил, как включился. Интересный оказался, но сложность местами не соответствует разделу, как мне показалось. В частности, некоторые Easy совсем не изи оказались. Это если их с максимальным уровнем сложности "Extreme" сравнивать.

[https://github.com/type-challenges/type-challenges](https://github.com/type-challenges/type-challenges)

Ниже решения первого раздела задач

## Warm Up

### [Hello World](https://github.com/type-challenges/type-challenges/blob/main/questions/00013-warm-hello-world/README.md)

```ts
type HelloWorld = string // expected to be a string
```

## Easy

### [Pick](https://github.com/type-challenges/type-challenges/blob/main/questions/00004-easy-pick/README.md)

```ts
type MyPick<T, K extends keyof T> = { [V in K]: T[V] };
```

### [Readonly](https://github.com/type-challenges/type-challenges/blob/main/questions/00007-easy-readonly/README.md)

```ts
type MyReadonly<T> = { readonly [K in keyof T]: T[K] }
```

### [Tuple to Object](https://github.com/type-challenges/type-challenges/blob/main/questions/00011-easy-tuple-to-object/README.md)

```ts
type TupleToObject<T extends readonly (string | number](]> = { [K in T[number]]: K }
```

### [First of Array](https://github.com/type-challenges/type-challenges/blob/main/questions/00014-easy-first/README.md)

```ts
type First<T extends unknown[]> = T extends [] ? never : T[0]
```

### [Length of Tuple](https://github.com/type-challenges/type-challenges/blob/main/questions/00018-easy-tuple-length/README.md)

На этой штуке завис... Вот что значит, не читать сообщения компилятора. Думал, что он ругается, что .length нет, но нет, он ругался на точку и просил ['length'].

```ts
type Length<T extends readonly unknown[]> = T['length']
```

### [Exclude](https://github.com/type-challenges/type-challenges/blob/main/questions/00043-easy-exclude/README.md)

Тут тоже залип. Напрочь забыл как раскручиваются юнионы типов.

```ts
type MyExclude<T, U> = T extends U ? never : T;
```

### [Awaited](https://github.com/type-challenges/type-challenges/blob/main/questions/00189-easy-awaited/README.md)

Тут было проще, но пришлось рекурсивно вложенные Promise раскрутить. И само решение уже не то, чтобы уж совсем "easy" получилось

```ts
type MyAwaited<T extends { then: (arg: any) => any }> = T['then'] extends (a: (value: infer R) => any) => any ? (R extends Promise<any> ? MyAwaited<R> : R) : never
```

### [If](https://github.com/type-challenges/type-challenges/blob/main/questions/00268-easy-if/README.md)

А вот это было реально простым.

```ts
type If<C extends boolean, T, F> = C extends true ? T : F
```

### [Concat](https://github.com/type-challenges/type-challenges/blob/main/questions/00533-easy-concat/README.md)

И это опять простое

```ts
type Concat<T extends any[], U extends any[]> = [...T, ...U]
```

### [Includes](https://github.com/type-challenges/type-challenges/blob/main/questions/00898-easy-includes/README.md)

Нифига себе изи! Справился только через рекурсивный разбор и строгое сравнение типов из-за совершенно нелогичных кейсов, вида `Includes<[1], 1 | 2>`, который должен отдавать false, хотя, казалось бы, что проверяем список `[1]` на соответствие `1` или `2`... Но нет, в тестах хотят именно строгий матч и чтобы тут был false.

И сравнение типов на равенство подсмотрел на SO: проверки на взаимный extends недостаточно из-за того что `true extends boolean` и `boolean extends true` - оба `true`, хотя казалось бы...

```ts
type Exact<A, B> = (<T>() => T extends A ? 1 : 0) extends (<T>() => T extends B ? 1 : 0) ? true : false
type Includes<T extends Array<any>, U> = T extends [infer V, ...infer Rest] ? Exact<V, U> extends true ? true : Includes<Rest, U> : false
```

### [Push](https://github.com/type-challenges/type-challenges/blob/main/questions/03057-easy-push/README.md)

Ну, тут тривиально:

```ts
type Push<T extends Array<unknown>, U> = [...T, U]
```

### [Unshift](https://github.com/type-challenges/type-challenges/blob/main/questions/03060-easy-unshift/README.md)

И это тоже, по сути, тот же Push, только отзеркаленный

```ts
type Unshift<T extends Array<unknown>, U> = [ U, ...T]
```

### [Parameters](https://github.com/type-challenges/type-challenges/blob/main/questions/03312-easy-parameters/README.md)

Написать собственный Parameters. Ну, такое я писал, пока его ещё не было в стандартной библиотеке. Но ни разу такое не изи, хотя и не сказать, чтобы очень сложно.

```ts
type MyParameters<T extends (...args: any[]) => any> = T extends (...args: infer R) => any ? R : never
```

## Extreme

### [Get Readonly Keys](https://github.com/type-challenges/type-challenges/blob/main/questions/00005-extreme-readonly-keys/README.md)

Проверки ради попробовал сразу extreme-уровень, первый же `Get Readonly Keys`. Оказалось, что пол дела уже сделано. Сравнение придумалось сразу: Exact у меня уже был (из Includes), а долго провозился со способом хранения найденных ключей.

```ts
type Exact<A, B> = (<T>() => T extends A ? 1 : 0) extends (<T>() => T extends B ? 1 : 0) ? true : false
type GetReadonlyKeys<T> = {
  [K in keyof T]-?: Exact<Pick<T, K>, Readonly<Pick<T, K>>> extends true ? K : never
}[keyof T];
```

По сравнению с Includes, радикального усложнения в Get Readonly Keys я не заметил. Получается, что или Easy не весь такой уж изи, по крайней мере, некоторые задачи из него. Или Extreme не очень-то и экстремальный. Я больше склоняюсь к первому варианту.
