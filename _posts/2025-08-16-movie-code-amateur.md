---
layout: post
title:  "Код в фильмах: Новичок (Amateur) 2025"
date:   2025-08-16 00:00:00
categories: [movie, code]
---

![]({{site.url}}/images/amateur-movie/cover.webp)

В этом фильме мало прямых заимствований. Весь код и его подобие, похоже, создавались специально для фильма.
Одни и те же фрагменты кода повторяются в разных сценах. Такие повторы буду пропускать.

#### 05:08 <small>*(здесь и далее - время от начала фильма)*</small>
В сцене разглядывания спутниковых снимков слева виден интерфес с именами программных файлов:

![]({{site.url}}/images/amateur-movie/004.webp)

```
Editor 01
- exe_VideoComm...
- FindOptionsWid...
- Contribution.md
- DDI_usr.txt
- Package.json

Editor 02
- DDI_Compiler.js
- .metntion.bot
- .travis.yml
- .yarnrc
- Licence Plate Te...
- Contribution.md
- DDI_usr.md
- video_fileSource...
- .gulpfile.js
- package.json
- contentmenu.ts
- FindOptionsWid...
- Contribution.md
- Developer.txt
- dddos.Tools.ys
```

Выглядит, как какой-то проект на Тайпскрипте. Но ничего конкретного тут нет, и, вероятно, допущены опечатки: должно быть `package.json`, а не `Package.json`, опечатка в слове `metntion` (`mention`?), и расширение файла, вероятно, должно быть не `.ys`, а скорее `.ts` (клавиша `t` находится рядом с `y`).

Вот это сокращение "`DDI`" в начале файлов дальше будет мелькать весь фильм. Какого-то внятного объяснения, что это может значить, нет. Инициалы технического консультанта?

#### 05:11

А тут вполне понятные команды установки связи. С логичным отображением процесса.

![]({{site.url}}/images/amateur-movie/008.webp)

```
# Initializing Secure Satellite Network Connection...

$ ./initiate_secure_link.sh --satellite SAT-47 --encryption AES-256

# Authentication Required
Enter Access Key: ************
Enter Biometric Verification: [Verified]

Access Granted
Secure Uplink Established
Connected to CIA-SAT-47 at Lat: 34.0535° N, Long: -118.2449° W

Initializing Encrypted Communication Channels...
Encryption Protocol: AES-256-GCM
Session ID: 8FA79C90-20BE-4A38-AF18-349C12A6F4A7
Handshake Successful

# Loading Satellite Control Interface...
```

#### 05:16

Фрагмент кода, вычисляющий орбитальный период спутника Земли, в секундах, по формулам Ньютона и Кеплера. Уж не знаю, зачем такое может понадобиться рядовому работнику, который смотрит на спутниковые картинки, но код рабочий.
![]({{site.url}}/images/amateur-movie/018.webp)

```
import numpy as np
def orbital_period(a):
    G = 6.67430e-11
    M = 5.972e24
    mu = G * M

    T = 2 * np.pi * np.sqrt(a**3 / mu)
    return T
```

И сразу же пример вызова этого кода. Видимо, тестировали корректность его работы:
```
a = 7000000
period = orbital_period(a)
```

#### 06:27

По сюжету Чарльз пытается открыть зашифрованный файл, который ну никак не хочет открываться. Но картинка выглядит так, как будто открыли самый обычный вордовый ".doc" в текстовом редакторе, типа блокнота.

![]({{site.url}}/images/amateur-movie/046.webp)

#### 07:37

Только что у нас были зашифрованные документы, а теперь мы почему-то пытаемся расшифровать `outright-libdes-dev-1_1-s_rh.bin`. Интересно, откуда он взялся? В изначальном листинге файлов такого нет.

![]({{site.url}}/images/amateur-movie/052.webp)

#### 08:56

Но, видимо, мощности ноутбука не хватает, и Чарльз идёт в бункер к мощному компьютеру.

Запускает его и появляется код. Странный:

![]({{site.url}}/images/amateur-movie/058.webp)

Тут намешан синтаксис нескольких языков программирования: `Java` (*`System.out.println`*), `JavaScript` (*`document.write`*), макрос или прагма (*`#execute`*), Windows-путь? (*`E:admin 851212118/bin.files`*), строчка Linux-шелла? (*`{ (VAULT_user_851212518@DDI_terminal):~$`*)

Никакого смысла в этом фрагменте нет. Просто наброс команд.

#### 09:25

И вот пароль подобран, файлы расшифрованы, и в мелькающих фрагментах мешанины из разных языков добавляются фрагменты на `C`:

![]({{site.url}}/images/amateur-movie/077.webp)

В них содержатся намёки на софт для камер наблюдения (`CONFIG_CCTV`), фрагменты из кода ядра Линукса, из подсистемы работы с таймерами для чипов [`picoXcell`](https://en.wikipedia.org/wiki/PicoChip), предназначенных для использования в небольших и компактных устройствах (*`dw_apb_timer_of.c`*)

Этот фрагмент далее будет мелькать в разных сценах и окошках.

### 19:44

Чарльз не доверяет никому и начинает самостоятельное расследование. Опять какая-то мешанина из `Java` с непонятно чем:

![]({{site.url}}/images/amateur-movie/095.webp)

#### 20:49

А тут пошёл креатив. Взяли фрагмент кода из упомянутого выше ядра Линукса `dw_apb_timer_of.c` и перемешали строки с `JavaScript`-примером из [совета на `StackOverflow`](https://stackoverflow.com/a/47593316)

![]({{site.url}}/images/amateur-movie/102.webp)

```
    goto try_clock_freq;
switch (num_called) {
for (let i = 0, k; i < str.length; i++) {
    h1 = h2 ^ Math.imul(h1 ^ k, 597399067);
    function cyrb128(str) {

for (let i = 0, k; i < str.length; i++) {
    pr_debug("%s: found clockevent timer\n", __func__);
```

#### 21:27

Когда Чарльз готовит для демонстрации начальству 3D-модель уличной сцены, кто-то (он сам?) пишет код на [Three.js](https://threejs.org/) - `JavaScript`-библиотеке для создания 3D-графики в браузере.

Он что, в браузере делает презентацию?

![]({{site.url}}/images/amateur-movie/109.webp)

```
if (j > 0) geometry.faces.push(new THREE.Face3(0, j, j + 1));

const material = new THREE.MeshBasicMaterial({ color: 0xff0000 });
const mesh = new THREE.Mesh(geometry, material);
scene.add(mesh);

{
    import_3D MODELER
    media_source == collate_media_sources
```

И тут не обошлось тут без включения абсолютно чужеродного фрагмента про `import_3D_MODELER`.

Вот такое он в браузере намоделировал, получается:

![]({{site.url}}/images/amateur-movie/111.webp)
Неплохо!

#### 26:34

Опять мелькают два якобы редактора кода, но на этот раз на экране код на `C` под `Windows`, который является фрагментом [драйвера, который скрывает рабочие процессы](https://github.com/dhruval/HideProc/blob/82a06de192c92b9e6c8415cde3bca72cb0906f70/main.c#L98) в `Windows`. В основном этот трюк используется вирусами и прочими малварями. Но и без творческого исправления тут не обошлось: в коде системные префиксы заменены на те самые "`DDI`".

И оно вполне соответствует сюжету: в этой сцене Чарльз готовит разные отвлекающие штуки для слежения за коллегами и маскировки.

![]({{site.url}}/images/amateur-movie/118.webp)


#### 27:16

Фрагмент редактирования файла `DDI_buildfile.py`. Тут небольшой скрипт, которые использует библиотеку `OpenCV` для распознавания объектов на видео. Видимо, проиллюстрирован процесс отладки программы, которая будет следить за коллегами.

![]({{site.url}}/images/amateur-movie/121.webp)

```py
cap = cv2.VideoCapture('object_tracking_video.mp4')

tracker = cv2.TrackerKCF_create()

success, frame = cap.read()
bbox = cv2.selectROI(frame, False)
tracker.init(frame, bbox)

while cap.isOpened():
    ret, frame = cap.read()
```

#### 1:27:08

Чарльзу с подругой приходится убегать от преследователей, а на фоне запущен процесс удаления улик. И тут много чего удалятельного мелькает на экране. Каждый кадр удаляется что-то новое.

![]({{site.url}}/images/amateur-movie/149.webp)

Повторения приводить не буду, перечислю только уникальные строчки:

- Удалим записи из логов, но почему-то только устаревшие. Свежие оставим
```sql
DELETE FROM system_logs WHERE timestamp < 2023-02-12;
```

- Удаляем всю директорию логов
```sh
rm -rf /var/logs/*
```

- И безопасно удалим какой-то один очень-очень важный файлик
```sh
shred -u /path/to/sensitive_file
```

- Удаляем записи из логов ошибок, но только записи с низким приоритетом. Всё важное оставляем
```sql
DELETE FROM error_logs WHERE severity = 'low';
```

- И грохнем таблицу с записями об активных сессиях пользователей
```sql
TRUNCATE TABLE user_sessions;
```

- И что-то удаляем кодом (на `Java`? `JavaScript`?)
```js
db.systemData.remove({ ... });
```

- И удаляем кеш скачанных `deb`-пакетов
```sh
sudo apt-get clean
```

- И удаляем файлики логов через поиск
```sh
find /path/to/directory -type f -name "*.log" -exec rm -f {}
```

- И удаляем таблицу со старыми бэкапами
```sql
DROP TABLE IF EXISTS old)_backup_data;
```

- А! Так это PostgreSQL тут у вас
```sql
VACUUM FULL;
```

- Затираем записи в `syslog`
```sh
echo "" > /var/log/syslog
```

- А теперь удалим из базы временные файлы, причём, только обработанные
```sql
DELETE FROM temp_files WHERE status = 'processed';
```

- И очередь отправки писем почистим. Удалим из очереди те, что уже были отправлены. Неотправленные оставим
```sql
DELETE FROM email_queue WHERE sent = true;
```

- И, традиционно, микс из разных языков, `SQL`+`JavaScript`
```sql
DELETE FROM db.temporary.remove({});
```

- И `SQL`+`shell`
```sql
LOG ROTATE --delete-system-files
```

- Удалим информацию о временных файлах
```sql
TRUNCATE TABLE temporary_files;
```

- И сами временные файлы
```sh
rm -rf /var/tmp/*
```

Очень похоже на то, что авторы загуглили как удалить информацию и нашли кучу примеров на разных языках об освобождении места на диске и в БД, и всё подряд добавили в фильм.

Почистили, смотрю, в основном только кеши, временные файлы и всё несерьёзное. Всю важную информацию при этом оставили нетронутой.

#### 1:30:49

И, напоследок, сцена прошивки джейлбрейкнутого телефона, который за минуту до этого был куплен в уличной лавочке.

![]({{site.url}}/images/amateur-movie/150.webp)
