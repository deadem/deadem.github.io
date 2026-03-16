---
layout: post
title:  "Код в фильмах: Кремниевая долина, сезон 2 (Silicon Valley S02)"
date:   2026-03-16 00:00:00
categories: [movie, code]
---

![]({{site.url}}/images/silicon-valley/cover.webp)

Продолжаю разбирать сезонами.

[Сезон 1]({{site.url}}{% link _posts/2025-05-14-silicon-valley-s01.md %})

### Сезон 2, Серия 1

В этом сезоне мелькает довольно много кода, но весь он показан вдалеке и размыто, поэтому мало что удалось прочесть.

![]({{site.url}}/images/silicon-valley/s02/001.webp)

Числа на футболке Эрлиха - это слово `Bitcoin`

### Серия 3

![]({{site.url}}/images/silicon-valley/s02/002.webp)

Тут аж целых три фрагмента с кодом.

![]({{site.url}}/images/silicon-valley/s02/003.webp)

Слева вверху цитата из статьи ["`Multipeer Connectivity API`"](https://web.archive.org/web/20150502050438/http://nshipster.com/multipeer-connectivity/): фреймворк от Apple, который позволяет устройствам `iOS`, `iPadOS`, `macOS` и др. обнаруживать друг друга и обмениваться данными напрямую, без необходимости подключения к интернету или общей сети. Статья позже была дополнена примерами на `Swift`, но на момент выхода сериала - код на `Objective-C`.

Фрагмент кода:

```objc
#pragma mark - MCNearbyServiceAdvertiserDelegate

- (void)advertiser:(MCNearbyServiceAdvertiser *)advertiser
didReceiveInvitationFromPeer:(MCPeerID *)peerID
       withContext:(NSData *)context
 invitationHandler:(void(^)(BOOL accept, MCSession *session))invitationHandler
{
    if ([self.mutableBlockedPeers containsObject:peerID]) {
        invitationHandler(NO, nil);
        return;
    }

    [[UIActionSheet actionSheetWithTitle:[NSString stringWithFormat:NSLocalizedString(@"Received Invitation from %@", @"Received Invitation from {Peer}"), peerID.displayName]
                       cancelButtonTitle:NSLocalizedString(@"Reject", nil)
```

![]({{site.url}}/images/silicon-valley/s02/004.webp)

Слева внизу фрагмент из [документации на `Gradle`](https://github.com/gradle/gradle/blob/99d83f56d6db965e06bfabaaeb11023193b38065/platforms/documentation/docs/src/snippets/toolingApi/customModel/groovy/tooling/src/main/java/org/gradle/sample/Main.java) - система управления зависимостями и автоматизация сборки, создание всяких задач и скриптов.

Main.java
```java
package org.gradle.sample;

import org.gradle.tooling.*;
import org.gradle.sample.plugin.CustomModel;

import java.io.File;
import java.lang.Object;
import java.lang.String;
import java.lang.System;

public class Main {
    public static void main(String[] args) {
        // Configure the connector and create the connection
        GradleConnector connector = GradleConnector.newConnector();

        if (args.length > 0) {
            connector.useInstallation(new File(args[0]));
            if (args.length > 1) {
                connector.useGradleUserHomeDir(new File(args[1]));
            }
        }

        connector.forProjectDirectory(new File("../sample-build"));

        ProjectConnection connection = connector.connect();
        try {
            // Load the custom model for the project
            CustomModel model = connection.getModel(CustomModel.class);
            System.out.println("Project: " + model.getName());
            System.out.println("Tasks: ");
            for (String task : model.getTasks()) {
                System.out.println("   " + task);
            }
        } finally {
            // Clean up
            connection.close();
        }
    }
}
```
![]({{site.url}}/images/silicon-valley/s02/005.webp)

Справа вверху файл `R.java` - компиляция ресурсов приложения для `Android`. По скомпилированному файлу, к сожалению, не понять, какой интерфейс у приложения, для которого сгенерирован этот файл, и чем это приложение вообще должно заниматься.

```java
        public static final int activity_horizontal_margin=0x7f040000;
        public static final int activity_vertical_margin=0x7f040001;
    }
    public static final class drawable {
        public static final int ic_launcher=0x7f020000;
```

![]({{site.url}}/images/silicon-valley/s02/006.webp)

Что на экране опять не разобрать - слишком мыльно. C? Swift? Но в руках - [книга "`Arduino Projects Book`"](https://archive.org/details/arduino_projects_book/mode/2up). Для самого первого знакомства с Ардуино. Как собирать схемы на макетной плате, как писать к ним простенькие программы. Крупный кегль и много картинок.


### Серия 8

![]({{site.url}}/images/silicon-valley/s02/007.webp)

На экране файл с названием `Buffer.java`. Название проекта: `p2pradio`. Назначение кода - потоковый приём пакетов данных (кадры видео? аудио?) с некоторой заданной частотой.

```java
public synchronized Packet getFuturePacket(long streamPacketSeqNr, long maxTimeToWait)
{
    try
    {
        return get(streamPacketSeqNrToBufferSeqNr(streamPacketSeqNr));
    }
    catch (P__rException ex)
    {
        long waitUntil = System.currentTimeMillis() + maxTimeToWait;
        long time;

        while (waitUntil > (time = System.currentTimeMillis()))
        {
            try
            {
                wait(waitUntil - time);
            }
            catch (InterruptedException e)
            {
                Logger.severe("Buffer", "INTERNAL_ERROR", e);
            }

            long seqNr = 0;
            boolean packetAvailable = true;

            try
            {
                seqNr = streamPacketSeqNrToBufferSeqNr(streamPacketSeqNr);
```
![]({{site.url}}/images/silicon-valley/s02/008.webp)

Идёт массовая заливка видео:

```
pid 3  [rsync_user]  OK  UPLOAD : Client - 10.10.158.199  /mount/gluster/originals/yiffing/clip21598.mov
pid 3  [rsync_user]  OK  UPLOAD : Client - 10.10.158.199  /mount/gluster/originals/yiffing/clip21599.mov
pid 3  [rsync_user]  OK  UPLOAD : Client - 10.10.158.199  /mount/gluster/originals/yiffing/clip21983.mov
pid 3  [rsync_user]  OK  UPLOAD : Client - 10.10.158.199  /mount/gluster/originals/yiffing/clip21985.mov
pid 3  [rsync_user]  OK  UPLOAD : Client - 10.10.158.199  /mount/gluster/originals/yiffing/clip25198.mov
pid 3  [rsync_user]  OK  UPLOAD : Client - 10.10.158.199  /mount/gluster/originals/yiffing/clip25248.mov
pid 3  [rsync_user]  OK  UPLOAD : Client - 10.10.158.199  /mount/gluster/originals/yiffing/clip2525148.mov
pid 3  [rsync_user]  OK  UPLOAD : Client - 10.10.158.199  /mount/gluster/originals/yiffing/clip2526148.mov
```

Жаргонное "yiffing" - это секс с участием фурри - [антропоморфных животных](https://ru.wikipedia.org/wiki/%D0%A4%D1%83%D1%80%D1%80%D0%B8)

(в этой сцене проигрывается видео с логом загрузки задом наперёд, иллюстрируя стирание файлов)

### Серия 10

![]({{site.url}}/images/silicon-valley/s02/009.webp)

Собирают презентацию, используя кадры упавшего и покалеченного работника?

На экране разметка на [SMIL](https://en.wikipedia.org/wiki/Synchronized_Multimedia_Integration_Language) - язык разметки для создания мультимедийных презентаций. Брат `html`-разметки.

<hr>

[Сезон 1]({{site.url}}{% link _posts/2025-05-14-silicon-valley-s01.md %})
[Сезон 2]({{site.url}}{% link _posts/2026-03-16-silicon-valley-s02.md %})
