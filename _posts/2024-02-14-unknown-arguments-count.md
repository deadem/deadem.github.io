---
layout: post
title:  "C++, неизвестное число параметров в конструкторе"
date:   2024-02-14 00:00:00 -0400
categories: [cpp, code]
---

Понадобилось тут создавать объекты, вызывая конструкторы с неизвестным числом аргументов. И главная проблема: а сколько параметров передавать? И если для функций и методов можно взять указатель и разобрать шаблоном на кирпичики, с конструктором такое не прокатит:  получить указатель на конструктор нельзя.

```cpp
template <typename Return, typename Obj, typename... Args>
constexpr int number_of_arguments(Return (Obj::*fp)(Args...) const) {
  return sizeof...(Args);
}
```

Облегчает задачу, что сами типы аргументов конструктора известны. Они все должны выводиться из некоторого общего типа, условно, "`Any`". Тогда можно пробовать вызывать конструктор с N+1 аргументами, и считать количество, с которым объекту-таки удалось создаться.

Сначала получим число параметров, рекурсивно проверяя конструктор, с каждым шагом увеличивая число параметров на один.

```cpp
template <int count, typename Any, typename T, typename... Args>
constexpr auto arguments_count() -> std::enable_if_t<std::is_constructible<T, Args...>::value, int> {
    return count;
}

template <int count, typename Any, typename T, typename... Args>
constexpr auto arguments_count() -> std::enable_if_t<!std::is_constructible<T, Args...>::value, int> {
    static_assert(count < 20, "Max constructor arguments");
    return arguments_count<count + 1, Any, T, Args..., Any>();
}
```

С полученным числом аргументов уже можно вызывать конструктор. Но нужно как-то собрать список из `Any` длиной `arguments_count`. И если схлопнуть последовательность можно удобно через `index_sequence`, то чтобы расковырять её обратно без хелпера не обойтись. И трикшот с оператором запятая, чтобы вместо каждого числа подсунуть аргументом `Any`:

```cpp
template <typename T, typename Pack>
struct Constructor;

template <typename T, std::size_t... arguments>
struct Constructor<T, std::integer_sequence<std::size_t, arguments...>> {
    template <typename Any>
    static constexpr T construct(const Any &any) {
        return T(((void)arguments, any)...);
    }
};
```

И, собственно, вызов:

```cpp
template <typename T, typename Any>
constexpr T construct(const Any &any) {
  return Constructor<T, std::make_index_sequence<arguments_count<0, Any, T>()>>::construct(any);
}
```

Пример использования в реальной жизни. [godbolt](https://godbolt.org/#z:OYLghAFBqd5QCxAYwPYBMCmBRdBLAF1QCcAaPECAMzwBtMA7AQwFtMQByARg9KtQYEAysib0QXACx8BBAKoBnTAAUAHpwAMvAFYTStJg1DIApACYAQuYukl9ZATwDKjdAGFUtAK4sGe1wAyeAyYAHI%2BAEaYxBIapAAOqAqETgwe3r56icmOAkEh4SxRMVxxdpgOqUIETMQE6T5%2BXLaY9rkM1bUE%2BWGR0bG2NXUNmc0KQ93BvUX9pQCUtqhexMjsHOYAzMHI3lgA1CYbbgQAnvGYAPoExEyECofYJhoAgpvbu5gHR16OtIQnDyerxezDYCniTFWewQrXOxHuLxMAHYrC89ui9gRMCx4gYsV83MECHs0F5BKRMWdGKxPs8GCcKadzqDPgAVRlUlkAOh5e2exGA9w2jzRGLQDHGmFU8WIeyYP1QcoFPkYBAUF1Jgggcz2AFoHntxugQCBqRF6Bc8FQroc3EaTXh1eLxsQvJVzZhbey%2BQKFDyuQ8TQA3MReTAUokG5Go54YuN7YiYAjLBgkpaCQ4xuPIgAiQKBcaxOLxn1tRLTZIIHOZNL59Or1LYe29TMbmH9PsFgNF6OdWOlsvlRCVwBVgid6YI2r1BvtpuYHst1oItrAYDnjo1AhdbscHq9FP5gv9gZAIe84b2keFBxRBfj6PGTEcyAuTAUSjqEE1xNtezMcQHGYZgALJMKoaYStcu4kCOY5quYZhzJm94PomybEKmtSjmw45bpWto/kBFh7M0dYMs2h6%2Bv6h70g82ooT2t55oiTFFriz6lkcrYspRlI1k2yiQgA1t2sYYjuDh7B4UGug4JCMcC4nouxJYEjxtbenOyQAF6XAQHbYfBQoisphrQVJMmSUQxAHuZxogESmDANEFxKAAjmGDCrLa2l4HpVwUkZuFqiewpRneTGFtiHH4raGlNnSALhVFEk1C%2BkGSgOzaZRZU59uRQEAGyGCcOrRqhD4YuhKbNlAEBBqgeDoHMwWqgoQX0nM/rIRsWZVbmlWDX1%2BZIixSmqZx6mcppDa8UlYl9lKMo5X2cn5duxJJcVpXlZF4k1Zh0KwtECJmSaVl5QpRxaQQDksEwwmXMEWCqG5mCeYwPlHDCtBwgoJptXhP62oBSUUqy9G9Y8KXnSgm3rRAu2KYNrHPNZfK3v16LPBA5ZMBGgh7BEcwgHsQZI8RxN7VYY2VeWQYo3TI1oxjJEVUxFh40TTCk%2BTSM06jZkM0zeYs0pGNuFj9NE0GXw5qRosoy85ZYuMXDThzZmHama27rarzChAXDdYz4tC0CquYOMZia/tcY67l622lYRuSKbSto1b4wbHb2MJkmtV6w4tpuPRAAcHvm3TXtEw9wR%2B6N40cAstCcAArLwfgcFopCoJwYeWNYhpLCspZmBsPCkAQmgpwswkgEVGhcuHAFmAAnFIXDtxsAEaGY%2BicJIWe13nnC8ADcQ1znKekHAsAwIg8M4nQ0TkJQaAr/QMSCsw8QKAgqBVjQtBYvClARKPETBLUJycFX1/MMQJwAPIRNoFTT1Xm8hS/DC0HfGepAsARC8MANwYhaAA24LwLAD0jDiCAfgRMlQgzW1HlKCoPw1i5ycmnIBfwIg3Gfh4LAo9rh4BYPfWeVADCCgAGp4EwAAdxfsyahMhBAiDEOwKQnD5BKDUKPXQzQDBGBQNYaw%2Bg8ARABpABYqB4jtGgbqI0hwcymCLpYLgSI9QvzMOiXUAB1aIxBYK6nOOgQwL5dSmJIAoQxRjIF6hMaoa4TAXGWOsXgZAjjRDjF1GIPAwBUzGP8QQXUnkxAuPFGg%2BEqQYk2V1OgJJMo8AhmQCcFx6AvC4h8ZxdAuoiGGGQDCBxxicl5NEFiQp4p0AuKlO41R2I8AuNoagZ8tiom0BcbQVAwB8m0F1IolxDABC6iDHgOoXgxDJJsi4zwhTxgnHoLqcJCy4l9KYFgQpkzpnROMYmHJDArGCGSRUKBLjkghLWQIOJyQBBXOCQwCxZiWCoF4KgOJxBmroPkS0NoqQXAnJGE0UggQpiFGKFkJIKQBCgphTkVIPQoWzABZ/KoEwEVjFaBigQnQ6gor6CUQYXRsWksJZC4lEgFgKFLqsGlg8OCZ1INnXO%2BcOB7FUOHIquoiqSD2MAZAviIDQQYMJHUEBcCEFgpsE2vBp5aDmPXEA6czBci4Bsdu7dJBcHThsDYkhdVmCKky4epAqGSA2FyDYRUzDh0ruHdO4dJAaCRBocOrLR4coniAKetcFjzyXksAg8Qfjr2/KgLe0RQg0k4Ny3l/KSRiOAHsUVrpxVzF4JgfANlmp6H4Fw0Q4g%2BGFoESodQQCRGkGYTceI1DU4ZxHkAjlL8fhhuJKgKgXKeV8oFUKkVYqJVpo8NG2Ucqs3VwDQsGE2z%2BjajNbwKh6cipciKhsJEXAirhy7unduHcB5ss%2BePWwfqp0z2VaQBuWqNVInbu6p14cNBFTvdqplGxm3spPYquuTKzCfuPRwBV07SD3KBZIIAA%3D)

```cpp
struct A {
    A(int a, int b): v(a + b) {}
    int v;
};

struct B {
    int v = 1;
};

int main() {
  construct<A>(1);
  construct<B>(2);
}
```
