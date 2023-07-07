---
layout: post
title:  "ECMAScript Explicit Resource Management"
date:   2023-07-04 00:00:00 -0400
categories: [js, code]
---

В JS появится, наконец, [штуковина для управления временем жизни ресурсов](https://github.com/tc39/proposal-explicit-resource-management). Позволит делать что-то вроде деструкторов.

Какую проблему решаем?

```ts
export function doSomeWork() {
    const path = ".some_temp_file";
    const file = fs.openSync(path, "w+");

    try {
        // Работаем с только что созданным файлом
   }
    finally {
        // Не забыть в конце работы закрыть файл и удалить с диска
        fs.closeSync(file);
        fs.unlinkSync(path);
    }
}
```

Код корректный, правда очень уж многословный. Но теперь можно будет не писать `try/finally` и не следить руками за утечками ресурсов: добавили специальную конструкцию `using` и символ `Symbol.dispose`, и в всё итоге работает аналогично [using в C#](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/using-statement), [with в Python](https://docs.python.org/3/reference/compound_stmts.html#the-with-statement) и [try-with-resource в Java](https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html).

```ts
// Класс, который умеет следить за собственным временем жизни
class TempFile implements Disposable {
    path: string;
    handle: number;

    constructor(path: string) {
        this.path = path;
        this.handle = fs.openSync(path, "w+");
    }

    // всякие методы

    [Symbol.dispose]() {
        // Закрываем и удаляем файл
        fs.closeSync(this.handle);
        fs.unlinkSync(this.path);
    }
}
```

Новым типом можно пользоваться так:

```ts
export function doSomeWork() {
    using file = new TempFile(".some_temp_file");

    // use file...
```

`using` вызовет метод `[Symbol.dispose]` при выходе переменной "file" за область видимости. Под капотом там тот же самый `try/finally`, но который намного чище и проще читать.

Кроме того, добавили `DisposableStack`, чтобы можно было писать "деструкторы" прямо по месту, не придумывая дополнительных классов:

```ts
function doSomeWork() {
    const path = ".some_temp_file";
    const file = fs.openSync(path, "w+");

    // колбек вызовется при выходе переменной "cleanup" из области видимости
    using cleanup = new DisposableStack();
    cleanup.defer(() => {
        fs.closeSync(file);
        fs.unlinkSync(path);
    });

    // use file...
```

[Фича уже доступна в Typescript 5.2.](https://devblogs.microsoft.com/typescript/announcing-typescript-5-2-beta/)
