---
layout: post
title:  "Код в фильмах: Код на миллиард долларов (The Billion Dollar Code)"
date:   2021-12-13 00:00:00 -0400
categories: [movie, code]
---

Смотрел мини-сериал "Код на миллиард долларов". В нескольких сценах мелькнул экран с кодом, а любопытство заставило повнимательнее его рассмотреть.

Всего в фильме код на экране возникает трижды: один раз в первом эпизоде, и два раза в четвёртом, причём последний раз - это дубль кода из первого эпизода.

![](/images/the-billoin-dollar-code-s1e1-00-27-55_.png)

Эпизод 1, 00:27:55

1993 год, Берлин. Сцена где Юрию Мюллеру, автору алгоритма ТерраВижена объясняется задача - создать рендерилку карт в реальном времени. И рассказывается, насколько это всё сложно.

На самом деле на экране:

[Кусок небольшого кейлоггера](https://github.com/sweetsoftware/FTPKeyLogger/blob/master/keylogger.cpp#L358). Код написан на C++ под Windows с использованием [SFML](https://www.sfml-dev.org/). Проект создан в 2014 году.

```cpp
void KeyLogger::log(int vkCode) {

    std::map<int, std::string>::iterator found = specialKeys.find(vkCode);

    if (found != specialKeys.end()) {
        log(found->second);
    }

    else {
        // formatting scan code and shift state for the VkKeyScan function
        // http://msdn.microsoft.com/en-us/library/windows/desktop/ms646329%28v=vs.85%29.aspx
        int keyCode = 0;
        if (isLShiftDown() || isRShiftDown()) {
            keyCode += 1;
        }
        if (isLCtrlDown() || isRCtrlDown()) {
```

этот же код мелькает на фоне после финальных титров в эпизоде 4, 01:12:21:
![](/images/the-billoin-dollar-code-s1e4-01-12-21_.png)

Со вторым фрагментом повеселее. В фильме идёт речь о структуре данных "Дерево квадрантов", "Quad-tree". И в сцене подготовки к суду в 2014 году (4 эпизод 00:08:11) доктор Каллахан в бункере Гугла изучает исходный код Google Earth, с тем, чтобы сравнить его с кодом ТерраВижена.

![](/images/the-billoin-dollar-code-s1e4-00-08-11_.png)

Читает, делает записи в блокноте... А на экране и правда фрагмент кода Quad-tree.
```c#
    return bestIntersection;
}

/// <summary>
/// Return the point index or -1 if not a vertex of the polygon
/// </summary>
/// <param name="polygon"></param>
/// <param name="position"></param>
/// <returns></returns>
public static int FindPoint(this Polygon polygon, IntPoint position, KdTree<long, int> pointKDTree = null)
{
    if (pointKDTree != null)
    {
        foreach (var index in pointKDTree.GetNearestNeighbours(new long[] { position.X, position.Y }, 1))
```

Вот только он из [библиотеки для работы с 3D-принтерами](https://github.com/MatterHackers/MatterSlice/blob/dc55feeb1e043b0772aa2c21b4d6947e1cccc14f/QuadTree/PolygonExtensions.cs#L340) от фирмы MatterHackers, а не для работы с картографическим софтом. Проект создан в 2014 году и написан на C#.
