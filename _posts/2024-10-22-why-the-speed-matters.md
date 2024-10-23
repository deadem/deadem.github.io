---
layout: post
title:  "Зачем увеличивать скорость работы сайта?"
date:   2024-10-21 0:00:00
categories: [code, web]
---

В продолжение [предыдущей заметки]({{site.url}}{% link _posts/2024-10-21-site-speed.md %}).

А нафига вообще заморачиваться скоростью сайта? Есть ли с этого какой-то осязаемый практический и финансовый выхлоп? Не какие-то абстрактные "так будет лучше", а конкретные кейсы?

Интернет завален выдуманными примерами и мифами, которые сеошники цитируют друг у друга. Пришлось покопать глубже.

- Пользователь зашедший на сайт, может покинуть его из-за медленной работы и вернуться на страницы результатов поисковых систем, чтобы найти другие сайты с более быстрой загрузкой контента. Медленная загрузка вызывает у пользователей разочарование, заставляя их покидать веб-страницы, даже если контент отвечает их информационным потребностям. ([Bartuskova, A.; Krejcar, O.; Sabbah, T.; Selamat, A. Website speed testing analysis using speedtesting model. J. Teknol. 2016](https://journals.utm.my/index.php/jurnalteknologi/article/view/10028))

- Увеличение времени загрузки страниц с одной до трех секунд увеличивает количество отказов на 32%. Увеличение с 1 до 5 - на 90%, с 1 до 6 - на 106%, с 1 до 10 - на 123%. (Google/SOASTA Research, 2017. Оригинал уже недоступен, поэтому ссылка на цитату: [Find Out How You Stack Up to New Industry Benchmarks for Mobile Page Speed](https://www.thinkwithgoogle.com/marketing-strategies/app-and-mobile/mobile-page-speed-new-industry-benchmarks/))

- Если мобильный сайт загружается больше 3 секунд, пользователь откажется его использовать и уйдёт с вероятностью 53%. (Google Data, Aggregated, anonymized Google Analytics data from a sample of mWeb sites opted into sharing benchmark data, n=3.7K, Global, March 2016)

- Когда люди испытывают связанные с неким брендом негативные впечатления на мобильных устройствах, вероятность того, что они совершат покупку у этого бренда в будущем снижается более чем на 60%. (Google/Purchased, “How Brand Experiences Inspire Consumer Action,” U.S. smartphone owners 18+=2,010, brand experiences=17,726, April 2017)

- Сайты компаний `BBC` теряли 10% пользователей за каждую дополнительную секунду загрузки контента. ([Clark, M. How the BBC Builds Websites That Scale](https://www.creativebloq.com/features/how-the-bbc-builds-websites-that-scale))

- Увеличение времени загрузки на 10% увеличивает на 1,7 процентных пункта количество отказов от использования сайта ([Speed Matters: What to Prioritize in Optimization for Faster Websites](https://www.mdpi.com/2813-2203/1/2/12))

- Исследование, проведённое компанией `COOK`, развивающейся в пищевой промышленности, показало, что снижение времени загрузки сайта на 850 миллисекунд привело к увеличению коэффициента конверсии на 7%, количества просмотренных страниц за сессию на 10%, а также к снижению показателя отказов на 7%. ([NCC Group. COOK Case Study](https://www.nccgroup.trust/globalassets/resources/uk/case-studies/web-performance/cook-case-study.pdf))

- Оптимизация платного поискового трафика: `Santander` увеличил скорость работы лендингов на 23%, что увеличило коэффициент достижения целей на 11%, снизило стоимость клика на 22%, а стоимость конверсии снизилась на 15%. ([Think with Google. How Santander and iProspect Gained More Conversions by Optimising Their Mobile UX and Paid Media spend.](https://www.thinkwithgoogle.com/intl/en-emea/marketing-strategies/app-and-mobile/how-santander-and-iprospect-gained-more-conversions-optimising-their-mobile-ux-and-paid-media-spend/))

- В `Mobify`, занимающейся созданием интернет-магазинов, каждые 100 миллисекунд уменьшения времени загрузки главной страницы и страницы оформления заказа увеличивали конверсию на 1,11% - 1,53%. Это приносило доход от 380 000 до 530 000 долларов США в год. ([Yordanov, E. How Web Performance Affects Business Results (22 Case Studies)](https://nitropack.io/blog/post/web-performance-matters-case-studies))

- Улучшение времени загрузки сайта `AutoAnything` привело к увеличению онлайн-продаж на 12-13%. ([Enright, A. Web Accelerator Revs up Conversions, Cart Size and Sales For AutoAnything.com](https://www.digitalcommerce360.com/2010/08/19/web-accelerator-revs-conversion-and-sales-autoanything/))

- Компания `Furniture Village` обнаружила, что после уменьшения времени загрузки сайта на 20% коэффициент конверсии на мобильных устройствах увеличился на 10%, процент отказов снизился на 9%, а доходы от продаж выросли на 12%. ([Think with Google. Furniture Village and Greenlight Slash Page Load Times, Boosting the User Experience.](https://www.thinkwithgoogle.com/intl/en-gb/marketing-strategies/app-and-mobile/furniture-village-and-greenlight-slash-page-load-times-boosting-user-experience/))

- Уменьшение времени ответа сервера магазина сумок "Пан Чемодан" с 5 до 2 секунд привело к увеличению переходов на страницу заказа на 20%. ([«Пан Чемодан»: как мы снизили нагрузку на сервер в 2 раза](https://intensa.ru/blog/pan-chemodan-kak-my-snizili-nagruzku-na-server-v-2-raza/))

- Уменьшение скорости загрузки всех страниц сайта на 0,1 секунду приводит к повышению конверсии на 8% для ретейла и на 10% для туристических сайтов. При этом посетители задерживаются на сайте до 10% дольше. ([Milliseconds make Millions. Deloitte Digital. 2020](https://www.deloitte.com/content/dam/Deloitte/ie/Documents/Consulting/Milliseconds_Make_Millions_report.pdf))

### Широко распространённая ошибочная и просто ложная информация

- Замедление страниц Amazon.com на 1 секунду может привести к снижению продаж в 1,6 миллиарда ежегодно. (А может и не привести, да). Часто приводится в формулировке совершенного вида, как доказанный факт. ([Компиляции и пересказы инфографики 2006 года](https://www.fastcompany.com/1825005/how-one-second-could-cost-amazon-16-billion-sales))

- Некорретная трактовка результатов исследования `Google/SOASTA Research, 2017`. Вместо того, чтобы читать "изменение времени загрузки с 1 до 3 секунд приводит к росту отказов на 32%", любят написать без "изменения", и получается "время загрузки страницы в 1 - 3 секунды приводит к росту отказов на 32%", что, конечно, не так.
