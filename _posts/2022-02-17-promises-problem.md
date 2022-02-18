---
layout: post
title:  "JavaScript-промисы"
date:   2022-02-17 00:00:00 -0400
categories: [js, code]
---

Хотел рассказать коллегам про промисы, и наткнулся на годную заметку 2015 года: ["We have a problem with promises"](https://pouchdb.com/2015/05/18/we-have-a-problem-with-promises.html).

Так она понравилась объяснением концепций и тонких моментов, что ниже будет почти что её перевод.

## У нас проблемы с промисами

Нет, не с самими промисами. Промисы прекрасны. Проблема в том, что многие программисты используют промисы, не понимая до конца, как они работают.

## Проверьте себя:

В чем отличие этих четырёх фрагментов кода? Функции `doSomething()` и `doSomethingElse()` возвращают промисы, которые долго и асинхронно что-то делают.

```js
doSomething().then(function () {
  return doSomethingElse();
});
```
```js
doSomething().then(function () {
  doSomethingElse();
});
```
```js
doSomething().then(doSomethingElse());
```
```js
doSomething().then(doSomethingElse);
```

Если вы знаете ответ, поздравляю, дальше можете не читать. Ничего нового не будет.

А для тех, кто остался, начнём с самого начала:

## Зачем нужны промисы

В разной литературе их описывают, как "избавление от ада с колбеками" (`callback hell`), как способ избавиться от огромных пирамид кода, постоянно вылезающих за правый край экрана, типа таких:

```js
fs.readdir(source, function (err, files) {
  if (err) {
    console.log('Error finding files: ' + err)
  } else {
    files.forEach(function (filename, fileIndex) {
      console.log(filename)
      gm(source + filename).size(function (err, values) {
        if (err) {
          console.log('Error identifying file size: ' + err)
        } else {
          console.log(filename + ' : ' + values)
          aspect = (values.width / values.height)
          widths.forEach(function (width, widthIndex) {
            height = Math.round(width / aspect)
            console.log('resizing ' + filename + 'to ' + height + 'x' + height)
            this.resize(width, height).write(dest + 'w' + width + '_' + filename, function(err) {
              if (err) console.log('Error writing file: ' + err)
            })
          }.bind(this))
        }
      })
    })
  }
})
```

Они, конечно, решают эти проблемы, но их назначение гораздо шире, чем просто поддержание аккуратной идентации кода. Настоящая проблема с колбеками в том, что мы теряем контроль над потоком исполнения программы: теряем возможность вызвать `return` или `throw`. Вместо этого всё приобретает побочные эффекты: одна функция может вызвать другую в любой, даже неожиданный момент.

А, кроме того, колбеки лишают нас ещё и стека. Мы больше не можем вернуться на уровень выше, чтобы как-то изменить порядок работы. Это можно сравнить с ездой на автомобиле с неработающей педалью тормоза: никогда не знаешь, насколько она важна, пока не потребуется ею воспользоваться. 

Главное назначение промисов - вернуть нам потерянные средства языка: `return`, `throw` и стек. Однако, взамен придётся научиться их использовать правильно, чтобы получить все эти возможности.

## Типичные ошибки

Некоторые определяют промисы как *такую штуку, которая представляет собой значение, полученное асинхронно*.

Не думаю, что такое определение может быть полезным. Как по мне, промисы - это про структуру кода и поток выполнения программы. Поэтому лучше рассмотрим типичные ошибки и как их исправить.

### Типичная ошибка №1: Пирамида кода

Посмотрим, как люди используют наш проект, PouchDB, API которого построено на промисах. Наиболее частая ошибка такая:

```js
remotedb.allDocs({
  include_docs: true,
  attachments: true
}).then(function (result) {
  var docs = result.rows;
  docs.forEach(function(element) {
    localdb.put(element.doc).then(function(response) {
      alert("Pulled doc with id " + element.doc._id + " and added to local db.");
    }).catch(function (err) {
      if (err.name == 'conflict') {
        localdb.get(element.doc._id).then(function (resp) {
          localdb.remove(resp._id, resp._rev).then(function (resp) {
// et cetera...
```

Тут код работы с промисами написан так, как будто это просто колбеки. И да, это можно сравнить с полировкой ногтей шлифовальной машиной: результат-то будет, но...

И если вы думаете, что так ошибаются только новички, то удивитесь: код выше взят из официального блога разработчиков BlackBerry. От старых привычек сложно избавляться.

Лучше было написать так:

```js
remotedb.allDocs(...).then(function (resultOfAllDocs) {
  return localdb.put(...);
}).then(function (resultOfPut) {
  return localdb.get(...);
}).then(function (resultOfGet) {
  return localdb.put(...);
}).catch(function (err) {
  console.log(err);
});
```

Это называется композицией. И это одна из суперспособностей промисов. Каждая функция вызовется только когда предыдущий промис будет разрешён и передаст свой результат дальше. Позже рассмотрим в подробностях.

### Типичная ошибка №2: Как использовать forEach() с промисами?

Это то место, где у большинства людей начинает ломаться понимание промисов. Разработчики легко и привычно пишут `forEach()` или `for/while`, но не знают, что делать, когда в циклах появляются промисы. В итоге получается что-то типа этого:

```js
// I want to remove() all docs
db.allDocs({include_docs: true}).then(function (result) {
  result.rows.forEach(function (row) {
    db.remove(row.doc);  
  });
}).then(function () {
  // I naively believe all docs have been removed() now!
});
```

Какая тут проблема? А дело в том, что первая функция ничего не возвращает, и вторая, следовательно, не ждёт, что будут вызваны `db.remove()` и начнёт выполняться она в любой момент после того, как будут запущены все `db.remove()`. И совершенно не обязательно, что к этому времени успеет удалиться хотя бы один документ.

Это особенно гадкий баг, потому, что вы можете и не заметить, что что-то пошло не так, думая, что это PouchDB так быстро работает, что почти мгновенно удаляет документы. Ошибка вылезет только в состоянии гонок или в некоторых браузерах, что добавит сложностей отладке.

Чтобы правильно написать тут цикл, нужен `Promise.all()`:

```js
db.allDocs({include_docs: true}).then(function (result) {
  return Promise.all(result.rows.map(function (row) {
    return db.remove(row.doc);
  }));
}).then(function (arrayOfResults) {
  // All docs have really been removed() now!
});
```

Что изменилось? `Promise.all()` получает на вход массив промисов и разрешится только тогда, когда все переданные промисы будут разрешены. То есть, можно быть уверенным, что вторая функция выполнится только после того, как будут отресолвлены все вызовы `db.remove()`. 

Кроме того, `Promise.all()` передаёт следующей функции массив результатов, что может быть крайне удобным, если вы хотите передать дальше несколько объектов, полученных через `get()` из PouchDB. А если хотя бы один из промисов в `Promise.all()` будет отвергнут, будет отвергнута и вся пачка промисов, что может быть ещё более полезным.

### Типичная ошибка №3: Забыли добавить .catch()

Это ещё одна постоянно встречающаяся ошибка. Будучи в святой уверенности, что их промисы накогда не завершатся с ошибкой, многие забывают добавить `.catch()` в свой код. К несчастью, это может привести к тому, что сообщение об ошибке будет потеряно, и вы ни увидите никаких следов, даже сообщения в консоли. Это может стать настоящей болью при отладке.

Хотя современные браузеры и не "проглотят" неперехваченный промис, но возникнет ошибка уровня браузера и работа всего скрипта будет прервана с выдачей ошибки в консоли о необработанной ошибке в промисе.

Чтобы избежать такого, я всегда добавляю в цепочку промисов подобный код: 

```js
somePromise().then(function () {
  return anotherPromise();
}).then(function () {
  return yetAnotherPromise();
}).catch(console.log.bind(console)); // <-- this is badass
```

Даже если ошибка не может произойти никогда, всё равно желательно добавлять `.catch()`. Это сделает жизнь проще, если когда-нибудь вы ошибётесь.

### Типичная ошибка №4: Использование "deferred"

Я настолько часто сталкиваюсь с этой ошибкой, что не хочу про неё тут писать, чтобы не множить примеры неправильного использования.

У промисов довольно долгая и непростая история. У JavaScript-сообщества ушло много времени, чтобы сделать их правильно. На заре появления промисов в `jQuery` и в `Angular` использовался паттерн "deferred", взмен которого впоследствии появились ES6-промисы, позаимствованные из "хороших" библиотек: `Q`, `When`, `RSVP`, `Bluebird`, `Lie`, и др.

Поэтому, если вы пишете это слово в своём коде (больше я его не повторю), вы что-то делаете не так. Сейчас покажу, как его избегать.

Начнём с того, что многие библиотеки дают возможность "импортировать" реализации промисов откуда-то ещё. Например, модуль `$q` из Ангуляра позволяет обернуть сторонние реализации промисов в `$q.when()`. Поэтому пользователи Ангуляра могут использовать эту фичу так:

```js
$q.when(db.put(doc)).then(/* ... */); // <-- this is all the code you need
```
Другой стратегией будет использование шаблона "Открытый конструктор", который удобен для оборачивания асинхронных API, построенных не на промисах. Например, API, построенные на колбеках, типа `fs.readFile()` в `Node.js`:

```js
new Promise(function (resolve, reject) {
  fs.readFile('myfile.txt', function (err, file) {
    if (err) {
      return reject(err);
    }
    resolve(file);
  });
}).then(/* ... */)
```

Всё! Мы избавились от гадкого "def...".

Если хотите узнать подробнее, почему это анти-паттерн, загуглите "Bluebird wiki promise anti-patterns"

### Типичная ошибка №5: Использование побочных эффектов вместо возврата значений

Что не так с этим кодом?

```js
somePromise().then(function () {
  someOtherPromise();
}).then(function () {
  // Gee, I hope someOtherPromise() has resolved!
  // Spoiler alert: it hasn't.
});
```

Вот сейчас хороший момент, чтобы рассказать всё, что вам нужно знать о промисах. 

Серьёзно. Всего одна концепция, и как только вы её поймёте, сразу же будете избавлены от всех тех ошибок, про которые мы только что говорили. Готовы?

Как я рассказывал выше, магия промисов возвращает наши любимые return и throw. Но как это выглядит на практике?

Каждый промис предоставляет вам метод `then()` (или `catch()`, что является просто сахаром над `then(null, ...)`). 

Вот мы внутри `then`-функции:

```js
somePromise().then(function () {
  // I'm inside a then() function!
});
```

Что мы можем тут сделать? Три вещи:

* вернуть другой промис
* вернуть синхронное значение (или `undefined`)
* выбросить синхронное исключение

Вот и всё. Если вы в этом разобрались - вы поняли промисы. Давайте рассмотрим каждый пункт подробнее.

#### Вернуть другой промис

Этот пример чаще всего встречается в литературе о промисах под названием "композиция промисов":

```js
getUserByName('nolan').then(function (user) {
  return getUserAccountById(user.id);
}).then(function (userAccount) {
  // I got a user account!
});
```

Обратите внимание, что тут я возвращаю второй промис. Этот `return` критически важен. Если я не сделаю `return`, то метод `getUserAccountById()` фактически будет побочным эффектом, и следующая функция получит на вход `undefined` вместо `userAccount`.

#### Вернуть синхронное значение (или `undefined`)

Возврат `undefined` - это скорее всего ошибка, но вообще возврат синхронного значения - это отличная фича, чтобы обернуть синхронный код промисами. Представьте, что у нас есть кеш пользователей. Вот как мы можем написать:

```js
getUserByName('nolan').then(function (user) {
  if (inMemoryCache[user.id]) {
    return inMemoryCache[user.id];    // returning a synchronous value!
  }
  return getUserAccountById(user.id); // returning a promise!
}).then(function (userAccount) {
  // I got a user account!
});
```

Разве не здорово? Вторая функция понятия не имеет, каким образом был получен `userAccount`, синхронно или асинхронно. А первая функция может по своему выбору вернуть как синхронное, так и асинхронное значение.

К сожалению, если забыть написать `return` в функции на JavaScript, это равнозначно тому, что мы намеренно вернули `undefined`, что означает, что мы можем нечаянно привнесты побочные эффекты, если забудем написать `return`.

По этой причине я выработал у себя привычку всегда завершать функцию внутри `then()` с помощью `return` или `throw`. И вам советую.

#### Выбросить синхронное исключение

Когда дело касается `throw`, промисы становятся ещё более мощными. Предположим, что мы хотим сгенерировать ошибку в случае, если пользователь не авторизован. Это просто:

```js
getUserByName('nolan').then(function (user) {
  if (user.isLoggedOut()) {
    throw new Error('user logged out!'); // throwing a synchronous error!
  }
  if (inMemoryCache[user.id]) {
    return inMemoryCache[user.id];       // returning a synchronous value!
  }
  return getUserAccountById(user.id);    // returning a promise!
}).then(function (userAccount) {
  // I got a user account!
}).catch(function (err) {
  // Boo, I got an error!
});
```

Функция `catch()` получит синхронное сообщение об ошибке, если пользователь не авторизован. А также мы синхронно получим ошибку, если какой-либо из промисов будет отвергнут. И снова, обратите внимание: функция не заботится о том, каким образом возникло значение, которое она получает, синхронно или асинхронно.

Особенно это удобно в процессе разработки. Например, внутри функции `then()` мы делаем `JSON.parse`. Он может бросить синхронное исключение, и, если бы код был на колбеках, то оно просто потерялось бы. А на промисах обязательно вызовется обработчик в `catch()`-функции.

## Продвинутые ошибки

Теперь, когда вы знаете главное, что делает промисы простыми, поговорим о граничных случаях. Потому, что всегда есть граничные случаи.

Эти ошибки названы "продвинутыми", потому, что я встречал их только у тех, кто уже хорошо разбирается в промисах. Но нам придётся о них поговорить, чтобы разобраться в примере, с которого начинается эта заметка.

### Продвинутая ошибка №1: Незнание про Promise.resolve()

Как я показывал выше, промисы очень удобны для оборачивания синхронного кода в асинхронный. Но если вы поймали себя на том, что напечатали такое:

```js
new Promise(function (resolve, reject) {
  resolve(someSynchronousValue);
}).then(/* ... */);
```

Знайте, что вы можете выразиться более кратко, используя `Promise.resolve()`:

```js
Promise.resolve(someSynchronousValue).then(/* ... */);
```

Также это полезно для перехвата синхронных ошибок. Настолько полезно, что я завёл себе привычку начинать все свои функции, возвращающие промисы, подобным образом:

```js
function somePromiseAPI() {
  return Promise.resolve().then(function () {
    doSomethingThatMayThrow();
    return 'foo';
  }).then(/* ... */);
}
```

Запомните: любой код, который может синхронно бросить исключение - это кандидат на сложную, трудноотлавливаемую ошибку. Но если у вас всё завёрнуто в `Promise.resolve()`, вы всегда можете быть уверенными, что обработаете его позже в `catch()`.

Аналогично, есть `Promise.reject()`, который вы можете использовать, чтобы вернуть промис, который был отвергнут:

```js
Promise.reject(new Error('some awful error'));
```

### Продвинутая ошибка №2: then(resolveHandler).catch(rejectHandler) - это не то же самое, что then(resolveHandler, rejectHandler)

Раньше я говорил, что `catch()` - это просто синтаксический сахар. Так что это два выражения эквивалентны:

```js
somePromise().catch(function (err) {
  // handle error
});

somePromise().then(null, function (err) {
  // handle error
});
```

Однако, это не значит, что эквивалентны эти два:

```js
somePromise().then(function () {
  return someOtherPromise();
}).catch(function (err) {
  // handle error
});

somePromise().then(function () {
  return someOtherPromise();
}, function (err) {
  // handle error
});
```
Если вам интересно, почему они не эквивелентны, подумайте, что произойдёт, если первая функция бросит исключение:

```js
somePromise().then(function () {
  throw new Error('oh noes');
}).catch(function (err) {
  // I caught your error! :)
});

somePromise().then(function () {
  throw new Error('oh noes');
}, function (err) {
  // I didn't catch your error! :(
});
```

Как видите, если использовать обработчики вида `then(resolveHandler, rejectHandler)`, то в `rejectHandler` вы не поймаете ошибку, если она возникла в `resolveHandler`.

По этой причине я никогда не использую второй аргумент у `then()` и всегда предпочитаю `catch()`. Исключения бывают только когда я пишу Мока-тесты, где мне нужно проверить, что выбрасывается ошибка:

```js
it('should throw an error', function () {
  return doSomethingThatThrows().then(function () {
    throw new Error('I expected an error!');
  }, function (err) {
    should.exist(err);
  });
});
```

Из-за того, что `catch` - это сахар для `then`, нужно помнить, что цепочку промисов можно продолжить после `catch`. И можно использовать это для восстановления при ошибках:

```js
somePromise().then(function () {
  throw new Error('uh! oh!');
}).catch(function (err) {
  return 'recovered';
}).then(function (message) {
  // Got an "recovered" message
});
```

#### finally

Кроме `catch()` есть метод `finally()`, который удобно использовать, когда нужно сделать одинаковую работу, например, спрятать лоадер, как в случае успешного разрешения запроса на сервер, так и в случае ошибки.


```js
somePromise().then(/* ... */
).catch(/* ... */
).finally(() => {
  // Do some cleanup
});
```

Работает он похоже на `then(finallyHandler, finallyHandler)` но всё же с некоторыми отличиями:

* нет никакого способа отличить внутри `finally`, был ли он вызван на разрешённом или отвергнутом промисе: в него не передаётся никаких аргументов
* если `finally()` не бросает исключений, то его результат работы никак не влияет на цепочку обработки после него: он "пропускает" результат или ошибку к следующим обработчикам `then/catch`
* если `finally()` бросит исключение, то именно оно будет передано в обработчики `catch()`, находящиеся после него

### Продвинутая ошибка №3: Промисы и фабрики промисов

Представим, что вам нужно выполнить несколько промисов, один за другим, последовательно. Поэтому, вам нужно что-то типа `Promise.all()`, но не запускающее промисы одновременно.

Наивно можно написать так, передав массив промисов:

```js
function executeSequentially(promises) {
  var result = Promise.resolve();
  promises.forEach(function (promise) {
    result = result.then(promise);
  });
  return result;
}
```

К сожалению, оно всё равно будет работать параллельно. Проблема в том, что промисы, передаваемые на вход `executeSequentially()` тут же запускаются, так как промис начинает работу в момент своего создания.

По этой причине нам на вход нужен не массив промисов, а массив фабрик промисов:

```js
function executeSequentially(promiseFactories) {
  var result = Promise.resolve();
  promiseFactories.forEach(function (promiseFactory) {
    result = result.then(promiseFactory);
  });
  return result;
}
```

Я знаю, что вы подумали: "Что это ещё за программирование на Java, какие-такие фабрики?" Но фабрика промисов - это просто. Это всего лишь функция, которая вернёт промис:

```js
function myPromiseFactory() {
  return somethingThatCreatesAPromise();
}
```

Почему это заработает? А потому, что фабрика не даёт промису создаться раньше, чем он нам понадобится. Он работает аналогично функции `then()`. На самом деле, это она и есть.

Если посмотреть на `executeSequentially()` и представить, что `myPromiseFactory` будет подставлена внутрь `result.then(...)`, на вас должно снизойти озарение.

### Продвинутая ошибка №4: А что, если нужно вернуть результат двух промисов?

Бывает, что результат одного промиса зависит от другого, и при этом нужен результат обоих промисов. Например:

```js
getUserByName('nolan').then(function (user) {
  return getUserAccountById(user.id);
}).then(function (userAccount) {
  // dangit, I need the "user" object too!
});
```

В попытке быть хорошим программистом и не выстраивать пирамиды из промисов вы можете решить хранить полученное значение в объекте, уровнем повыше:

```js
var user;
getUserByName('nolan').then(function (result) {
  user = result;
  return getUserAccountById(user.id);
}).then(function (userAccount) {
  // okay, I have both the "user" and the "userAccount"
});
```

Так заработает, но мне кажется, что оно выглядит коряво. Вместо этого можно вернуть из первой функции кортеж:

```js
getUserByName('nolan').then(function (user) {
  return [ user, getUserAccountById(user.id) ];
}).then(function ([ user, userAccount ]) {
  // okay, I have both the "user" and the "userAccount"
});
```

Или отбросить предубеждения и построить-таки пирамиду:

```js
getUserByName('nolan').then(function (user) {
  return getUserAccountById(user.id).then(function (userAccount) {
    // okay, I have both the "user" and the "userAccount"
  });
});
```

...временно. Чтобы починить идентацию, вы можете вынести код в отдельную функцию:

```js
function onGetUserAndUserAccount(user, userAccount) {
  return doSomething(user, userAccount);
}

function onGetUser(user) {
  return getUserAccountById(user.id).then(function (userAccount) {
    return onGetUserAndUserAccount(user, userAccount);
  });
}

getUserByName('nolan')
  .then(onGetUser)
  .then(function () {
  // at this point, doSomething() is done, and we are back to indentation 0
});
```

По мере того, как ваш код будет усложняться, вам потребуется всё больше и больше кода выносить в отдельные функции. Что приведёт к аккуратному виду, типа такого:

```js
putYourRightFootIn()
  .then(putYourRightFootOut)
  .then(putYourRightFootIn)  
  .then(shakeItAllAbout);
```

Вот для этого и нужны промисы.

### Продвинутая ошибка №5: Проваливаемся сквозь промисы

А вот и ошибка, на которую я намекал в задачке о промисах в самом начале. Это очень экзотичная ситуация, которая может никогда не встретиться в живом коде, но она и меня озадачила.

Как вы думаете, что напечатает этот код?

```js
Promise.resolve('foo').then(Promise.resolve('bar')).then(function (result) {
  console.log(result);
});
```

Если думаете, что "bar", то вы ошиблись. Он печатает "foo".

Так происходит потому, что если в `then()` передаётся не функция, а что-то другое, то `then()` просто выбрасывается из цепочки, и приводит это к тому, что дальше "проливается" предыдущий результат. Можете проверить: 

```js
Promise.resolve('foo').then(null).then(function (result) {
  console.log(result);
});
```

Сколько бы `then(null)` вы не добавили, оно всегда напечатает "foo".


Что возвращает нас к предыдущему пункту про промисы и фабрики. Передать промис в функцию `then()` возможно, но навряд ли это то, что вы хотели. `then()` предназначен, чтобы принимать на вход функцию, поэтому, скорее всего, вы хотели написать так:

```js
Promise.resolve('foo').then(function () {
  return Promise.resolve('bar');
}).then(function (result) {
  console.log(result);
});
```

Будет напечатан "bar", как мы и хотели.

И ещё раз напомню: в `then()` всегда нужно передавать функцию!

## Решения:

Теперь, когда мы знаем всё (ну, почти) о промисах, мы можем решить задачу, с которой начиналась эта заметка.

Ответ к каждому пункту будет в графическом формате, для лучшего представления, что произошло:

### Задача 1
```js
doSomething().then(function () {
  return doSomethingElse();
}).then(finalHandler);
```
Ответ:

```
doSomething
|-----------------|
                  doSomethingElse(undefined)
                  |------------------|
                                     finalHandler(resultOfDoSomethingElse)
                                     |------------------|
```

### Задача 2

```js
doSomething().then(function () {
  doSomethingElse();
}).then(finalHandler);
```
Ответ:

```
doSomething
|-----------------|
                  doSomethingElse(undefined)
                  |------------------|
                  finalHandler(undefined)
                  |------------------|
```

### Задача 3

```js
doSomething().then(doSomethingElse())
  .then(finalHandler);
```
Ответ:

```
doSomething
|-----------------|
doSomethingElse(undefined)
|---------------------------------|
                  finalHandler(resultOfDoSomething)
                  |------------------|
```

### Задача 4

```js
doSomething().then(doSomethingElse)
  .then(finalHandler);
```
Ответ:

```
doSomething
|-----------------|
                  doSomethingElse(resultOfDoSomething)
                  |------------------|
                                     finalHandler(resultOfDoSomethingElse)
                                     |------------------|
```

Если что-то в этих ответах непонятно, перечитайте пост, и, напомню, что функции `doSomething()` и `doSomethingElse()` возвращают промисы, которые долго и асинхронно что-то делают: запрашивают данные по сети или из БД, `setTimeout`, или что-то другое, за пределами текущего цикла событий.

[Ссылка на JSBin с живой демонстрацией](https://jsbin.com/tuqukakawo/1/edit?js,console,output)

## Резюме

Промисы мощные. Если вы всё ещё используете колбеки, настоятельно рекомендую перейти на промисы: код уменьшится, станет более понятным и элегантным.

### async/await

В ES7 появился новый синтаксис для работы с промисами: `async/await`, который превращает псевдосинхронный код на промисах с методом `catch()`, который ведёт себя, почти как ключевое слово `catch`, но не совсем, в настоящий, синхронно выглядящий код, с привычными `try/catch/return`.

Прелесть `async/await` ещё и в том, что если вы ошибётесь, то скорее всего получите ошибку линтера/транспайлера вместо ошибок в рантайме.

Если у вас есть возможность использовать `async/await` - используйте их.
