---
layout: post
title:  "Охранный код для контроля порядка включения модулей"
date:   2019-08-18 11:00:00 -0400
categories: code cpp
---

Задача - есть ряд модулей, некоторые низкоуровневые, некоторые высокоуровневые, и хорошо бы контролировать порядок их включения, так, чтобы из системного модуля не зацепить модуль с бизнес-логикой.

Есть три модуля: модуль 1, модуль 2, модуль 3, которые расположены в порядке возрастания своего уровня. Модуль 3 может использовать код модулей 1 и 2, а модулю 2 разрешается пользоваться только модулем 1, но не модулем 3. Можно создать заголовочные файлы в этих модулях, примерно такого вида:

`module1.h`
~~~~~
#define LEVEL 1
#include "header.h"
... code
~~~~~
{: .language-cpp}

`module2.h`
~~~~~
#define LEVEL 2
#include "header.h"
... code
~~~~~
{: .language-cpp}

`module3.h`
~~~~~
#define LEVEL 3
#include "header.h"
... code
~~~~~
{: .language-cpp}

И всё было бы тривиально, если бы не ленивое разрешение макросов в C++. Если написать так, то оно запомнит не значение макроса LEVEL, а ссылку на него, и развернётся в итоге в код, который ничего не проверит.
~~~~~
#ifndef PREV_LEVEL
#define PREV_LEVEL 0
#endif

static_assert(PREV_LEVEL >= LEVEL, "Invalid include order");
#undef PREV_LEVEL
#define PREV_LEVEL LEVEL
~~~~~
{: .language-cpp}

Можно эту проблему решить с помощью буст-препроцессора:

`header.h`
~~~~~
#include <boost/preprocessor/slot/slot.hpp>
#define PREV_LEVEL BOOST_PP_SLOT(1)

#define BOOST_PP_VALUE LEVEL
#include BOOST_PP_ASSIGN_SLOT(1)

static_assert(PREV_LEVEL >= LEVEL, "Invalid include order");
~~~~~
{: .language-cpp}

И если в модуле 1 включить заголовок из модуля 2, то будет ошибка компиляции. Оно работает, но использовать его не хочется, ибо буст препроцессор - это какой-то оверкилл для такой задачи.

Как можно обойтись без буста? Первая идея, взять enable_if:
~~~~~
#ifndef GUARD_CHECK_CORE
#define GUARD_CHECK_CORE
namespace Guard
{
  template <int T>
  constexpr std::enable_if_t<0 <= T, bool> level_check()
  {
    return true;
  }
}
#endif

namespace Guard
{
  static_assert(level_check<LEVEL>(), "Invalid include order");

  template <int T>
  constexpr std::enable_if_t<T<LEVEL, bool> level_check()
  {
    return false;
  }
}
~~~~~
{: .language-cpp}

Этот пример работает не совсем правильно, но в принципе, идея рабочая. Хотя, нагружать сборщик лишними вычислениями в процессе компиляции, это, наверное, даже хуже буст препроцессора. 

И думается, что такие подходы вообще не совсем корректны - если мы в модуле 3 сначала подключим модуль 2, а потом модуль 1, то всё будет в порядке, а если в модуле 3 сначала подключим 1, а потом 2, то будет ошибка. Как-то это странно, обязывать выстраивать включения в файле, сообразно внешней иерархии модулей.

Можно ограничиться только значеним текущего модуля и впоследствии просто убеждаться, что мы не зацепили ничего из иерархии выше. А раз так, то достаточно сохранить только первое значение уровня, а затем его со всеми сравнивать. И в этом может помочь `pop_macro`/`push_macro` - расширение от Майрософта, которое поддерживают как минимум компиляторы "большой тройки".

~~~~~
#ifndef ROOT_VALUE
#pragma push_macro("LEVEL")
#define ROOT_VALUE
#endif

static_assert(LEVEL <=
#pragma pop_macro("LEVEL")
#pragma push_macro("LEVEL")
    LEVEL, "Invalid include order");
~~~~~
{: .language-cpp}

Код, конечно, непортируем на абстрактный компилятор, но мне и не надо, достаточно того, что clang, gcc и msvc это умеют.
