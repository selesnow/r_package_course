--- 
title: "Курс 'Разработка пакетов на языке R'"
author: "Алексей Селезнёв"
date: "2023-08-06"
site: bookdown::bookdown_site
documentclass: book
bibliography: [book.bib, packages.bib]
# url: your book url like https://bookdown.org/yihui/bookdown
# cover-image: path to the social sharing image like images/cover.jpg
description: |
  Бесплатный видео курс по разработке собственных пакетов на языке R.
biblio-style: apalike
csl: chicago-fullnote-bibliography.csl
---

# Введение {-}

------

## О курсе {-}
<a href="https://selesnow.github.io"><img src="img/cover.png" align="right" alt="Cover image" class="cover" width="230" height="366" /></a>Вся мощь языка R заключается в его огромном сообществе, и написанных этим сообществом пакетах, значительно расширяющих базовые возможности языка. На момент написания этих строк только в основном репозитории CRAN опубликовано почти 20000 активных пакетов, на GitHub их ещё больше.

Разработка пакетов один из лучших способов повысить свои навыки написания кода на R, и углуюиться в его изучение. Данный курс поможет вам пошагово освоить процесс разработки собственных пакетов. 

В основу курса легла книга Хедли Викхема ["R Packages (2e)"](https://r-pkgs.org/), тем не менее данный курс не является полным повторением книги, некоторые главы мы рассматривать не будем, но и в ходе курса будут рассмотрены некоторые темы, которые Хедли не упоминал в своей книге.

## Для кого этот курс {-}
Данный курс я не могу рекомендовать новичкам. Заниматься разработкой пакетов лучше имея за плечами определённый опыт написания кода на R. Поэтому не стоит начинать изучения R с данного курса, ниже я дам небольшую подборку подготовительных курсов, изучив которые можно попробовать себя в разработке пакетов.

## По поводу поддержки обучающихся на данном курса {-}
**Важно!** Поддержки учащихся на этом курсе со стороны автора нет. Я не занимаюсь частными консультациями, тем более не консультирую студентов бесплатных курсов. Поэтому не имеет никакого смысла писать мне в личку или на почту просьбы помочь с прохождением этого, или любого другого моего бесплатного курса. Если вы столкнулись с трудностями при прохождении курса и вам нужна помощь, то все вопросы можно адресовать в следующие telegram чаты:

* [R (язык программирования)](https://t.me/rlang_ru)
* [Горячая линия R](https://t.me/hotlineR_EU)

Отдельного чата со студентами непосредственно этого курса не существует, но при желании вы самостоятельно можете его организовать, и я с радостью добавлю на него ссылку.

К тому же, если у вас есть вопросы по одной из лекций курса, вы можете задавать его под видео лекции на YouTube, это приветствуется, и на такие комментарии я с радостью отвечу.

Буду рад любой конструктивной критике, и предложениям по улучшению курса "разработка пакетов на языке R", направлять их можно мне на почту selesnow@gmail.com. Если вы хотите выразить благодарность мне за курс, то в конце раздела описано как это можно сделать.

## Об авторе {-}
Меня зовут Алексей Селезнёв, с 2008 года я являюсь практикующим аналитиком. На данный момент основной моей деятельностью является развитие отдела аналитики в агентстве интернет-маркетинга [Netpeak](https://https://netpeak.group/).
<a href="https://selesnow.github.io"><img src="img/author.png" width="200" height="200" align="left" alt="Алексей Селезнёв" hspace="20" vspace="7" /></a>

Мною были разработаны такие R пакеты как: `rgoogleads`, `rfacebookstat`, `timeperiodsR` и некоторые другие. На данный момент написанные мной пакеты только с CRAN были установленны более 150 000 раз.

Также я являюсь автором некоторых других курсов по R (ссылки на них приведу ниже), лектором академии [Web Promo Experts](https://webpromoexperts.net/) и соавтором курса ["Веб-аналитика Pro"](https://webpromoexperts.net/courses/analytics-pro/).

Веду свой авторский [Telegram](https://t.me/R4marketing) и [YouTube](https://www.youtube.com/R4marketing/?sub_confirmation=1) канал R4marketing. Буду рад видеть вас в рядах подписчиков.

Периодически публикую статью на различных интернет медиа, зачастую это [Хабр](https://habr.com/ru/users/selesnow/) и [Netpeak Journal](https://netpeak.net/ru/blog/user/publication/826/).

Неоднократно выступал на профильных конференциях по аналитике и интернет маркетингу, среди которых Матемаркетинг, GoAnalytics, Analyze, eCommerce, 8P и прочие.

## Другие курсы автора {-}
Как я уже писал выше, помимо курса "Разработка пакетов на языке R" у меня есть ряд других бесплатных курсов:

1. [Язык R для интернет маркетинга](https://r-for-marketing.netpeak.net/auth/sign/in), для начинающих, требуется бесплатная регистрация
2. [Язык R для пользователей Excel](https://selesnow.github.io/r4excel_users/), для начинающих
3. [Введение в dplyr 1.0.0](https://selesnow.github.io/dplyr_1_0_0_course), средней уровень сложности
4. [Циклы и функционалы в языке R](https://selesnow.github.io/iterations_in_r/), средней уровень сложности
5. [Разработка telegram ботов на языке R](https://selesnow.github.io/build_telegram_bot_using_r/), высокий уровень сложности

## Каналы автора {-}
Если вы интересуетесь языком R, применяете его в работе, или планируете изучать, то думаю вам будут интересны мои каналы, о которых я писал выше. Буду рад видеть вас среди подписчиков:

* [Telegram канал R4marketing](https://t.me/R4marketing)
* [Youtube канал R4marketing](https://www.youtube.com/R4marketing/?sub_confirmation=1)

## Программа курса {-}
В данный момент курс "разработка пакетов на языке R" назодится в активной стадии разработки, поэтому программа постоянно расширяется, следить за обновлениями курса можно на страницк [Новости курса]. Ниже представлена актуальная программа на текущий момент:

1. [Обзор рабочего процесса разработки пакета]

Дата обновления курса: 06.08.2023

## Благодарности автору {-}
Курс, и все сопутствующие материалы предоставляются бесплатно, но если у вас есть желание отблагодарить автора за этот видео курс вы можете перечислить любую произвольную сумму на [этой странице](https://secure.wayforpay.com/payment/r4excel_users).

Либо с помощью кнопки:
<center>
<script type="text/javascript" id="widget-wfp-script" src="https://secure.wayforpay.com/server/pay-widget.js?ref=button"></script> <script type="text/javascript">function runWfpWdgt(url){var wayforpay=new Wayforpay();wayforpay.invoice(url);}</script> <button type="button" onclick="runWfpWdgt('https://secure.wayforpay.com/button/b9c8a14345975');" style="display:inline-block!important;background:#2B3160 url('https://s3.eu-central-1.amazonaws.com/w4p-merch/button/bg2x2.png') no-repeat center right;background-size:cover;width: 256px!important;height:54px!important;border:none!important;border-radius:14px!important;padding:18px!important;box-shadow:3px 2px 8px rgba(71,66,66,0.22)!important;text-align:left!important;box-sizing:border-box!important;" onmouseover="this.style.opacity='0.8';" onmouseout="this.style.opacity='1';"><span style="font-family:Verdana,Arial,sans-serif!important;font-weight:bold!important;font-size:14px!important;color:#ffffff!important;line-height:18px!important;vertical-align:middle!important;">Оплатить</span></button>
</center>
