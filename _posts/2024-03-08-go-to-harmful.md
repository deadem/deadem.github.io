---
layout: post
title:  "О вреде go to"
date:   2024-03-08 00:00:00 -0400
categories: [code]
---

Случился у нас очередной разговор про оптимизации в коде программ и вспомнилось каноничное "Преждевременная оптимизация - корень всех зол". Только, как оказалось, находятся люди, которые по-разному понимают эту цитату и приводят аргументы о том, что, де, Кнут писал вообще "не о том". Начал было разбирать его публикацию, и, хотя он достаточно серьёзно погружает в контекст, захотелось начать ещё раньше, с заметки Дейкстры про оператор `go to`: "Go To Statement Considered Harmful".

Перескажу её тут, хотя изначально и не хотел, ибо заметка, казалось бы, маленькая и всё, вроде, должно быть понятно. Но она таким жутким языком написана, что сомневаюсь в том, что её вообще можно сходу осилить, а те варианты, что есть, тоже никуда не годятся. Как вам "Разнузданное применение оператора goto..."?

И пара фактов: 
- Дейкстра не называл свою заметку словами "о вреде". Изначально она называлась "A Case Against the Goto Statement". Провокационное называние придумал Никлаус Вирт, который работал редактором [`CACM`](https://cacm.acm.org/) в 1968. 
- Заметка вышла в разделе "Письма редактору" исключительно потому, что Вирт хотел ускорить процесс её публикации и не проводить рецензирование, как было положено для работ, публикуемых в журнале.

## 1968, Эдсгер Дейкстра, ["Go To Statement Considered Harmful"](https://dl.acm.org/doi/10.1145/362929.362947)

Много лет я наблюдаю, что чем лучше программист, тем реже он использует оператор `go to`. И наоборот. А недавно я понял, почему использование `go to` приводит к таким ужасным последствиям, и теперь уверен, что `go to` должен быть удалён из языков программирования высокого уровня. То есть, из буквально всех языков, кроме, разве что, машинного кода.

Во-первых, отмечу, что хотя деятельность программиста и заканчивается с написанием корректной программы, на самом деле это не является его целью. По-настоящему предмет его деятельности - это код, который решает поставленную задачу и удовлетворяет требования спецификаций. И этот код пишется машиной, которая только руководствуется теми инструкциями, которые написал для неё программист.

Во-вторых, наши интеллектуальные способности гораздо шире в части анализа статических взаимосвязей, нежели процессов, развивающихся во времени. Поэтому мы, как мудрые программисты, должны стремиться к тому, чтобы программа, выраженная в тексте, максимально точно соответствовала динамическому процессу её выполнения.

Рассмотрим, как можно описать ход выполнения программы. Если текст программы состоит только лишь из последовательности некоторых математических действий и мы хотим выполнять эти действия по одному, по шагам, у нас появляются некоторые точки в тексте программы, где предыдущая команда уже выполнилась, а следующая ещё не началась. Эти позиции, расположенные между командами в тексте программы, назовём "текстовыми индексами".

Если мы добавим в процесс условные команды (*если B истинно, тогда выполнить A*), условные с альтернативой (*если B истинно, тогда выполнить A1, иначе A2*), то ход процесса выполнения от этого не изменится, и точно так же может быть описан с помощью текстового индекса.

Но как только в языке появляются процедуры, одного текстового индекса нам уже недостаточно. Нам нужен индекс для обозначения места выполнения в тексте основной части программы, и ещё один индекс для такого же места внутри процедуры. Если из процедуры нужно вызвать процедуру, у нас уже появляется список текстовых индексов, количество которых соответствует глубине вызова процедур.

Теперь давайте рассмотрим операторы повторений (*пока истинно B, выполнять действие A*). С логической точки зрения эти конструкции излишни, так как мы можем выразить такие повторения через рекурсивный вызов процедур. Но, исходя из существующей практики, не будем их исключать: такие операции повторения могут быть удобно реализованы аппаратно на имеющемся оборудовании. С другой стороны, как только мы отказываемся от математической индукции, текстовых индексов уже становится недостаточно для описания динамического процесса. Однако, мы можем придумать "динамический индекс", который будет описывать порядковый номер текущего повторения. А так как повторяющиеся конструкции и процедуры могут быть вложенными, получается, что ход выполнения программы может быть выражен последовательностью текстовых и динамических индексов, вперемешку.

Самое главное тут в том, что программист не оперирует этими индексами напрямую. Они генерируются (либо статически, при компиляции программы, либо динамически, во время её выполнения) вне зависимости от его желаний. Эти индексы представляют собой независимые координаты для описания хода выполнения программы.

Зачем нам нужны эти "независимые координаты?" Дело в том, что мы можем интерпретировать значения переменных только во время работы программы, причём, соотносясь с ходом её выполнения. Если наша задача посчитать количество людей в изначально пустой комнате, мы можем это реализовать, увеличивая счётчик N на единицу каждый раз, когда увидим, что кто-то входит к комнату. Однако, между операциями, когда мы увидели, что кто-то входит, но ещё не успели увеличить счётчик, значение N будет на единицу меньше количества людей в комнате!

Повсеместное использование оператора `go to` приводит к тому, что становится чрезвычайно сложно подобрать набор координат, в которых было бы можно путём статического анализа описать ход выполнения программы. Обычно люди говорят, что это всё ерунда и нужно просто использовать правильно подобранные и выразительные переменные. Но об этом не может быть и речи, потому что значения этих переменных определятся только в ходе выполнения программы!

Хотя, однозначно описать ход выполнения программы с использованием оператора `go to` всё-таки можно. Например, можно использовать в качестве системы координат счётчик, подсчитывающий общее количество выполненных действий с момента начала программы. Только проблема в том, что такой параметр, хотя и уникален, абсолютно бесполезен. В такой системе координат становится чрезвычайно сложной задачей определить, например, все точки выполнения, где N равно количеству людей в комнате минус один.

Оператор `go to` в его нынешнем виде слишком уж примитивен: он провоцирует создание хаоса. Можно расценивать разобранные в заметке выражения, как средства для его ограничения. Я не утверждаю, что упомянутые выражения исчерпывающи, в том смысле, что они удовлетворят все потребности и в будущем, но любые дополнения (например, обработка прерывания) должны следовать тому же принципу: неподвластная программисту система координат должна описывать процесс исполнения полезным и управляемым способом.

В завершение хочу отметить тех людей, чьё мнение оказало на меня значительное влияние. Довольно очевидно, что я не остался невосприимчив к идеям Питера Лэндина и Кристофера Стрейчи, и я не сожалею об их влиянии на меня. Наконец, я хотел бы отметить (поскольку я помню это весьма отчетливо), как Хайнц Земанек на предварительной встрече по ALGOL в начале 1959 года в Копенгагене весьма ясно выразил свои сомнения относительно того, должен ли оператор `go to` обладать такими же синтаксическими правами, как и оператор присваивания. В некоторой степени я виню себя за то, что ещё тогда не сделал выводов из его замечания.

Замечание о нежелательности оператора `go to` далеко не ново. Я даже где-то читал прямую рекомендацию ограничить использование оператора `go to` случаями аварийного завершения, но не могу вспомнить где. Вероятно, в одной из работ Хоара. А в "`A contribution to the development of ALGOL`" [<sup>1</sup>](#link1) Вирт и Хоар вместе делают выводы той же неправленности, обосновывая необходимость добавления в ALGOL новой конструкции `case` (прообраз современных `switch/case`): "Как и условный оператор, он [case] отражает динамическую структуру программы более четко, чем операторы `go to` и `switch`, и исключает необходимость вводить в программу большое количество меток".

> Оператор `SWITCH` в ALGOL 60 работает в связке с `GO TO`:
> 
> ```algol
> SWITCH S := L1,L2,L3
> ...
> GO TO S[J]
> ```
> `S` используется как "переключатель" между метками `L1`, `L2`, и `L3`, а конструкция `GO TO S[J]`; использует значение переменной `J` для определения, к какой метке выполнить переход. Причём, точка перехода определяется динамически, в рантайме, а не статически, что позволяет указывать в `SWITCH` массивы меток и т.п.

В "`Turing machines and languages...`" [<sup>2</sup>](#link2) Джузеппе Якопини, похоже, получилось доказать логическую несостоятельность оператора `go to`. Однако, нужно быть осторожным с упражнениями по переводу произвольной блок-схемы в схему без переходов. Если подойти к задаче чисто механически, от результирующей блок-схемы не следует ожидать большей прозрачности, чем от исходной.

### Материалы:

#### 1. Wirth, Niklaus, and Hoare, C.A.R. A contribution to the development of ALGOL. Comm. ACM 9 (June 1966), 413–432. {#link1}
#### 2. Böhm, Corrado, and Jacopini, Guiseppe. Flow diagrams, Turing machines and languages with only two formation rules. Comm. ACM 9 (May 1966), 366–371. {#link2}
