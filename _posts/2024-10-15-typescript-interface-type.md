---
layout: post
title:  "Typescript: types vs interfaces"
date:   2024-10-15 0:00:00
categories: [code, ts]
---

Сколько читал разных книжек и заметок - везде только общие слова и вкусовщина на тему разницы между `type` и `interface` в `Typescript`. В лучшем случае скажут, что `type` более гибкий, а `interface` - он для лучшей читаемости, и классы лучше наследовать от него. Что в `type` есть `union`, а в интерфейсах нет, и что дженерики в общем виде не выразить через `interface`...

Расскажу-ка о нетривиальных штуках, про которые почему-то не пишут:

#### Интерфейсы можно расширять. `type` так не умеет, про это написано везде. Но как быть, если надо расширить интерфейс в другом файле? Как это выразить через `import`/`export`?

```ts
// a.ts
export interface Person {
  name: string;
}

// b.ts
import { Person } from './a'; // ERROR: Import declaration conflicts with local declaration of 'Person'
export interface Person {
  age: number;
}
```

Чтобы это работало, нужно в файле `b.ts` изменять не сам интерфейс, а модуль:

```ts
// b.ts
export { Person } from './a';

declare module './a' {
  interface Person {
    age: number;
  }
}

// c.ts
import { Person } from './b'; // или from './a' - типы объединены в рамках всего проекта
const person: Person = { name: 'John', age: 30 };
```

Тут из `b.ts` можно даже не экспортировать `Person` - интерфейсы уже "слиты" вместе.

В `b.ts` просто нужен импорт из `a.ts` хоть в каком-то виде. Сгодится и просто `import "./a"`, но я делаю через экспорт, для явной иллюстрации намерений использования модифицированного интерфейса.

Это и минус интерфейсов - после такого расширения разделить интерфейс обратно и откатиться к оригинальному из `a.ts` в рамках проекта станет невозможно.

#### Но это и плюс: интерфейсы, в отличие от `type`, кешируются

...что даёт буст к производительности транслятора, поэтому [рекомендуется использовать интерфейсы там, где нужны пересечения типов](https://github.com/microsoft/TypeScript/wiki/Performance#preferring-interfaces-over-intersections)

#### Ещё одно различие: как `interface` и `type` работают с индексами в типах:

В интерфейсах индексы могут быть только или общими, или конкретными. Попытка описать через интерфейс перечисляемый тип - ошибка:

```ts
interface Options {
  [K in 'title' | 'text']: string; // Error
  // A computed property name in an interface must refer to an expression
  // whose type is a literal type or a 'unique symbol' type
}

type Options = {
  [K in 'title' | 'text']: string; // OK
}
```

Как следствие, через интерфейсы нельзя, например, выразить тип, где числовым ключам соотствует число, а строковым - строка:

```ts
interface A {
  [key: string]: string;
  [key: number]: number; // Error
  // 'number' index type 'number' is not assignable to 'string' index type 'string'.
}
```

А через `type` - можно:
```ts
type B = {
  [key in string | number]: key extends string ? string : number;
};
```
