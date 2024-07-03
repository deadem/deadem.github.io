---
layout: post
title:  "JavaScript: Promise.withResolvers"
date:   2024-07-03 0:00:00
categories: [code, js]
---

В начале 2024 в js завезли удобный хелпер: при работе с промисами, которые нужно контролировать снаружи, теперь можно не городить внешние переменные и перекладывать в них колбеки. Теперь есть `Promise.withResolvers`:

```js
const { promise, resolve, reject } = Promise.withResolvers();
```

Создаёт промис и возвращает от него методы для `resolve` и `reject`. Так что теперь не нужно выворачивать логику.

Было:

```js
let outerResolve;
let outerReject;

const promise = new Promise((resolve, reject) => {
  outerResolve = resolve;
  outerReject = reject;
});

promise
    .then((resolvedValue) => { console.log(resolvedValue); })
    .catch((rejectedValue) => { console.error(rejectedValue); });

...
outerResolve("Resolved!");
```

А теперь можно написать чище, без "перекладывательной" логики:

```js
const { promise, resolve, reject } = Promise.withResolvers();

promise
    .then((resolvedValue) => { console.log(resolvedValue); })
    .catch((rejectedValue) => { console.error(rejectedValue); });

...
outerResolve("Resolved!");
```

И полифилится эта штука элементарно. Надо брать!
