---
layout: post
title:  "TypeScript: Все перестановки в union"
date:   2024-05-27 17:00:00
categories: [typescript, code]
---

Понадобилось по работе собрать из `union` список туплов со всеми возможными перестановками. То есть, чтобы из `"a" | "b"` получалось `["a"] | ["b"] | ["a", "b"] | ["b", "a"]`. Написал в лоб решение на 15 строк и подумал, что это как-то дофига. Начал ковырять плейграунд и пытаться соптимизировать.

[Дооптимизировал до одной строки](https://www.typescriptlang.org/play/?#code/PTAEiQQRBEEVhBUfhBqAEQalBCIIBhBC8IIPhBBMIILhBAOEFF1AFcA7ASwHsKAoAFwE8AHAU1AAV2AnAWzKMAho1oUAzgB4AKgBpQAVVABeUDIB8q9aHYAPRuwoATCUtAB+UAG0dAXVAAfG+oUA6DzwFDR46QFE9AGMAGzJjdilFBU0tBwAuUAp2ADc+AG56ehBQQCIQFFRAaRAmNk5DCUYABm0vQRExOmlktN4NTJzQToA9CyyWDlByxgBGGr4630apAHJhabbssE7QHr7SwfYKgCYx73q-GbmnUGmAI3n2pe7ekoGhgGZdiYbJQ+njs-fnaaCLxeWVjd+mVNowACxPHwvaSzL4nc4fX4fYx-DrXLIAb1AQUajFAemGiSGOzU1lhCk+dnSoAAvvQsTjJHi9FsiaCSTZPhS5lTafTsbj8fc2dttGTzry6QzBXowSLGByyTzqXTMQKmfjCRsKo9SbDJfzGRVNfLdZyJSrDTKtQ8xT9pgbpRqCaa7XMKRa+U7jS7tYwzUrphTfo71T6baCA1yTsqvWHmRGdXbzsGHZbvQnXXrfty03GjZm-VGc-C81L4yai26g6XU6GC5XbXr3ScS5T0xXfU3zTXya2y1bnYn-cne23Y+WG13I3bx73PZPrVnOW2UzGB-9AHgggAkQQiARhAiPF-gABRgSAC0+g4QUYl94vBovEH4eXgY9vYnJ7Pl7019vfAfJ8M0bGdmzHcCDRyU8LyvdgbzvQDn0Lbs33XSCwGgn8-wQx8kJApNSXQ0BMNg+CANw4DpwIntc1zA0gA), и полчаса сам вкуривал, как это работает:

```ts
// Все перестановки в union
type Permutations<T, U = T> = T extends U ? [ T ] | [ T, ...Permutations<Exclude<U, T>> ] : never;

type Test = Permutations<'a' | 'b'>; // ["a"] | ["b"] | ["a", "b"] | ["b", "a"]
```

Всё время я про эту фичу забываю. Запишу-ка тут, чтобы потом легче вспоминалось.

Магия происходит из-за того, что вот такая конструкция `T extends U ? [ T ] : never`, раскладывается на юнион из кортежей. То есть, в качестве `T`, в результате тернарного `extends`, будет по очереди подставлено каждое значение из пересечения `T` и `U`:

```ts
type Test<T, U = T> = T extends U ? [ T ] : never;
type Test2 = Test<'a' | 'b'>; // ["a"] | ["b"]
```

Ну, а дальше уже понятно.
