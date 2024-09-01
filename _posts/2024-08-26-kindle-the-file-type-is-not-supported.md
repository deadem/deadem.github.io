---
layout: post
title:  "Amazon Kindle: The file type is not supported"
date:   2024-08-26 0:00:00
categories: [bugs, kindle]
---

`Kindle` перестал поддерживать формат `mobi`, стал требовать `epub`. Ну, чтож, пошёл, скачал новую версию официальной утилиты `Send To Kindle`, пытаюсь по привычке правой кнопкой отправить `epub`-книжку и получаю ошибку: "`The file type is not supported`".

Загуглил: ошибка глобальная, воспроизводится не только у меня. Через драг-н-дроп всё работает, а если отправлять правой кнопкой - ошибка. И решения нет, хотя [репорт об этом висит с мая 2023](https://www.amazonforum.com/s/question/0D56Q0000Bd2GzISQU/epub-error-this-file-type-is-not-supported).

Попробую разобраться. Обратил внимание, что одновременно с окном сообщения об ошибке открывается консоль, в которой выводится:

```
Amazon SendToKindle Handler Program; argc=2
STK executable path: D:\Program Files (x86)\Amazon\SendToKindle\SendToKindle.exe
Skipping: D:\Downloads\50-ts.epub
```

Ага, они специальной программой слушают вызов контекстного меню и вызывают отправлялку. А в свойствах этого процесса значится `StkSendToHandler.exe`. Открыл его в текстовом редакторе:

![]({{site_url}}/images/kindle/file-type-is-not-supported.png)

Какой-то тут подозрительный список. Уж очень он похож на перечисление поддерживаемых форматов... Похоже, в основной программе `mobi` на `epub` поменяли, а тут забыли.

Открыл файл в `hex`-редакторе, заменил `mobi` на `epub`, и - отправка правой кнопкой через `Send to...` тоже заработала.

Жаль, проект закрытый, пул-реквест не отправить.
