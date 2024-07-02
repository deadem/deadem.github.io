---
layout: post
title:  "Числа и регулярные выражения"
date:   2024-07-02 0:00:00
categories: [code, regex]
---

Всю жизнь был уверен, что `\d` матчит строго символы в диапазоне от `0` до `9`. Но, сюрприз! Подкралась подстава от разработчиков юникода: в языках программирования, нативно оперирующих юникодом, числами считаются все, относящиеся к категории "`Nd`" (числа): [https://www.fileformat.info/info/unicode/category/Nd/list.htm](https://web.archive.org/web/20231214134825/https://www.fileformat.info/info/unicode/category/Nd/list.htm)

Примеры:
- [Python](https://regex101.com/r/RMsRTe/1) 
- [.NET](https://regex101.com/r/oDJr3V/1)
- [Rust](https://regex101.com/r/KoxL0k/1)

Вот жеж!