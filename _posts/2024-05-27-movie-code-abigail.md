---
layout: post
title:  "Код в фильмах: Эбигейл (Abigail)"
date:   2024-05-27 00:00:00
categories: [movie, code]
---

![]({{site.url}}/images/abigail/poster.webp)

Эбигейл... Идея интересная, а реализация слабенькая. Но в самом начале есть небольшая сцена, где один из персонажей изо всех сил использует компьютер для взлома охранной системы дома.

Итак, что нужно сделать первым делом, когда собираешься проникнуть в охраняемый дом? Правильно, отключить камеры:

![]({{site.url}}/images/abigail/home-cameras.webp)

И неважно, что на фоне виден листинг с объектами "TrafficCamera":

```
...rafficCamera camera6 = new TrafficCamera("Intersection TF");
                   ...7 = new TrafficCamera("Intersection TG");
                                    ...mera("Intersection TH");
                                                   ...ion TI");
```

Сказали, что "Home cameras are down", значит down!

Но вообще, странно, что в двух окнах код написан по-разному. В левом окне используются точки с запятой в качестве терминаторов строк, а в правом нет. Да и в целом, фу так писать, индусятина какая-то:

```
camera_system.add_camera(cameraT09)
camera_system.add_camera(cameraT10)
camera_system.add_camera(cameraT11)
camera_system.add_camera(cameraT12)
camera_system.add_camera(cameraT13)
camera_system.add_camera(cameraT14)
camera_system.add_camera(cameraT15)
camera_system.add_camera(cameraT16)
```

Но закончили с камерами. Теперь нужно открыть ворота.

![]({{site.url}}/images/abigail/gate-access-denied.webp)

"`Fuck!`". Что-то не получилось. Минута плотного стучания по клавиатуре, и, вот:

![]({{site.url}}/images/abigail/gate-access-granted.webp)

Теперь открываем дверь в дом. Что интересно, код на экране не изменился, только его стало лучше видно:

![]({{site.url}}/images/abigail/gate-access-granted-02.webp)

Теперь на фоне уж точно `Python`. 

Справа код на `tkinter` для отображения графики под `Tk`. И это очень похоже на реализацию интерфейса программы для ввода `PIN`. Вероятно, так она выглядела изначально, но потом призвали художников, и они навели красоты в интерфейсе, а старый код оставили в качестве декорации:

```python
"...Access Denied.")
    else:
        messagebox.showerror("Access Denied", "Front gate PIN is incorrect. Access Denied.")

import subprocess

pin_process = subprocess.Popen(['python', 'pin_generator.py'], shell=True)

pin_process.wait()

print("PIN generation completed.")

window = tk.Tk()

window.title("Enter PIN")

label = tk.Label(window, text="Enter PIN:")
label.pack()

pin_entry = tk.Entry(window, show=".")
pin_entry.pack()

pin_entry.bind("<Return>
```

А в левой части экрана код процедуры, открывающей дверь. И тут даже предусмотрен мастер-пароль:

```python
def unlock_door(door_name, entered_pin)
    if door_name in door_pins:
        if entered_pin == door_pins[door_name] or entered_pin == master_override:
            print(f"Access granted to {door_name}.")
        else:
            print(f"Access denied to {door_name}. Invalid PIN.")
    else:
        print(f"{door_name} is not a valid door.")

unlock_door(door_name, entered_pin)

door_pins = {
    "FR-001": 1889,
    "BK-001": 4723,
    "MN-001": 5678,
    "MN-002": 9012,
    "MN-003": 3156,
    "MN-004": 8901,
    "MN-005": 5183,
    "MN-006": 9443,
    "MN-007": 0020,
```

Сомнительно хранить список дверей и паролей для них прямо в коде, но вообще, это вполне рабочий листинг, только в результате проверки тут ничего не происходит, а просто выводится строка на экран: подошёл `PIN` или нет. 

А ещё тут спряталась забавная ошибка: в `Pyton 3` этот код не запустится, а в `Python 2` число, начинающееся с двух нулей, будет интерпретировано в восьмеричном виде. То есть, к двери "MN-007" код "0020" не подойдёт. К ней подходит "0016".
