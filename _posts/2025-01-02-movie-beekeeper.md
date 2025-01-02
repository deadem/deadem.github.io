---
layout: post
title:  "Код в фильмах: Пчеловод (The Beekeeper)"
date:   2025-01-02 00:00:00
categories: [movie, code]
---

![]({{site.url}}/images/the-beekeeper/the-beekeeper.webp)

Кто бы мог подумать, что персонаж Стейтема в "Пчеловоде" вполне себе технически продвинутый!? Примерно на 55 минуте фильма есть короткий эпизод, где Пасечник проникает в убежище ещё одного "пчеловода", с целью вооружиться и раздобыть ценную информацию.

Эпизод со взломом проходной, но для отслеживания телефонной линии он подключает мобильный к одному из стоящих на столе компьютеров, входит в систему и запускает трассировку сигнала.

![]({{site.url}}/images/the-beekeeper/tracing-screen.webp)

На мобильном телефоне в это время бегут снизу вверх любопытные буковки:

![]({{site.url}}/images/the-beekeeper/mobile-hack.webp)


Чуть крупнее:

![]({{site.url}}/images/the-beekeeper/mobile-hack-crop-1.webp)

Это [исходный код `Roslyn`](https://github.com/dotnet/roslyn/blob/version-3.2.0/src/Compilers/CSharp/CSharpAnalyzerDriver/CSharpDeclarationComputer.cs#L50) - компилятора языка программирования `C#`:

```c#
        private static void ComputeDeclarations(
            SemanticModel model,
            SyntaxNode node,
            Func<SyntaxNode, int?, bool> shouldSkip,
            bool getSymbol,
            ArrayBuilder<DeclarationInfo> builder,
            int? levelsToCompute,
            CancellationToken cancellationToken)
        {
            cancellationToken.ThrowIfCancellationRequested();

            if (shouldSkip(node, levelsToCompute))
            {
                return;
            }

            var newLevel = DecrementLevel(levelsToCompute);
```

Только после `return` в фильме добавили строку `[################################]`. Интересно, зачем?..

Среди кода `Roslyn` мелькает строка `Linux` с приглашением `bash`, упоминание некоей `consol-Pro` (?) и вполне правдоподобные логи обработки данных: чтение `tty01` и запись в сокет:

![]({{site.url}}/images/the-beekeeper/mobile-hack-crop-2.webp)

И что-то странное на [`jQuery`](https://jquery.com/):

![]({{site.url}}/images/the-beekeeper/mobile-hack-crop-3.webp)

```js
(..\temp.fil).raxoStart(function() {
  $("body").addClass("LOGIN") });
$(..\tempfil).raxoStop(function() {
  $("body").removeClass("PASSWORD") });
```

```
Logging in  [################################]
Password verified - login complete
```

И снова `C#`. На этот раз [фрагмент CSharpAnalyzerDriver](https://github.com/dotnet/roslyn/blob/version-3.2.0/src/Compilers/CSharp/CSharpAnalyzerDriver/CSharpDeclarationComputer.cs#L13), только `Microsoft` в строке `Microsoft.CodeAnalysis.CSharp` заменили на `Idylhands` (`ldyl`? `Idyth`?). Интересно, что это? Инициалы какого-нибудь Дилана из съёмочной команды?

![]({{site.url}}/images/the-beekeeper/mobile-hack-crop-4.webp)

Все эти фрагменты кода зациклены и несколько раз прокручиваются на телефоне во время взлома.
