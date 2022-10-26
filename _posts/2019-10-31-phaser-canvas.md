---
layout: post
title:  Попробовал порисовать в браузере через Phaser
date:   2019-10-31 11:00:00 -0400
categories: [code]
---

Увидел где-то рекомендацию использовать Phaser для работы с графикой в JS. Стало интересно попробовать.

Заточен он на двухмерную графику, причем, умеет автоматически определять поддержку WebGL и рендерить в него, а в случае, если его нет, откатываться на канвас.

Актуальная на сегодня версия 3, но на сайте документация ко второй версии, и описания толком никакого нет. Пришлось, в лучших традициях, копать исходники примеров, хорошо, что есть портированные версии, и смотреть, как это всё работает.

[В целом достаточно бодро получилось](https://github.com/deadem/stars/blob/master/index.js), можно попробовать использовать в реальном коде.

<div id="stars"></div>

<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/phaser/3.19.0/phaser.min.js"></script>
<script type="text/javascript">
let width = 800;
let height = 600;

let centerX = width / 2;
let centerY = height / 2;

let delta = 5000;
let deltaZ = 2000;

let config = {
    type: Phaser.AUTO,
    parent: 'stars',
    width,
    height,
    scene: {
        create: create,
        update: update,
    }
};

let game = new Phaser.Game(config);
let stars = [];

function create () {
    graphics = this.add.graphics();

    for (let i = 0; i < 2000; ++i) {
        stars.push({
            coords: newDot(),
            rect: this.add.rectangle(0, 0, 4, 4, 0xffffff)});
    }
}

function newDot() {
    return {
        x: -delta / 2 + Math.random() * delta,
        y: -delta / 2 + Math.random() * delta,
        z: -deltaZ
    };
}

function translate(coords) {
    let temp = height / coords.z;

    return {
        x: coords.x * temp,
        y: coords.y * temp
    };
}

function update () {
    stars.forEach(v => {
        let coords = translate(v.coords);
        let x = centerX + coords.x;
        let y = centerY + coords.y;
        v.coords.z += 10;

        if (v.coords.z >= 0 || x < 0 || x > width || y < 0 || y > height) {
            v.coords = newDot();
            v.rect.setAlpha(0);
            return;
        }

        v.rect.setPosition(x, y);
        v.rect.setAlpha(0.01 + (deltaZ - Math.abs(v.coords.z)) / deltaZ);
    });
}
</script>
