---
layout: custom_post
title: "Сравнение урлов"
date: 2018-07-05
author: Yuri Kalinin
category: Work
tags: testing daily
finished: true
---

# URL vs URI

Нашёл прикольную библитечку [galimatias](https://github.com/smola/galimatias)
В описании сказано, что ребята сделали это потому что стандартные джавовские 
`java.net.URL` и `java.net.URI` их не удовлетворили:
```text
galimatias started out of frustration with `java.net.URL` and `java.net.URI`
```

Вот [здесь](https://stackoverflow.com/a/3771123) описаны примерные проблемы:
1) Метод equals() в URL возвращает true если хосты резолвятся с одинаковым IP.
Виртуальные хосты резолвятся с одинаковым IP. То есть при подключении к интернету виртуальные хосты
бесмыссленно сравнивать. Вместо этого предлагает сравнивать через `java.net.URI`
2) В зависимости от реализации `java.net.URI` может по разному сравнивать урлы вида
```text
http://example.com
http://example.com/
```

Чтобы проверить у себя я беру `java.net.URI` и сравниваю 2 адреса. Получается так:
```text
java.lang.AssertionError: 
Expected: is <https://ya.ru/>
     but: was <https://ya.ru>
Expected :is <https://ya.ru/>
```
То есть моя реализация предполагает, что `https://ya.ru/` и `https://ya.ru` это  что-то разное.

# GALIMATIAS

Хорошо, что нам предлагает galimatias?

Подключаем зависимость:
```xml
<dependency>
  <groupId>io.mola.galimatias</groupId>
  <artifactId>galimatias</artifactId>
  <version>0.2.0</version>
</dependency>
```

Сравниваем:
```java
@Test
public void shouldBeEqual() {
    io.mola.galimatias.URL x = URL.parse("http://ya.ru");
    io.mola.galimatias.URL y = URL.parse("http://ya.ru/");
    assertThat(x, is(y));
}
```

Всё проходит! Адреса совпадают. Но здесь есть ещё одно «но».
Так как я привык, что работаю со строчками, меня немного смуило, что также эквивалентны и эти адреса:
```text
1) http://ya.ru:80
2) http://ya.ru/
```
Но тут всё также, как и раньше с простым "/". 
Смотрим подробнее в [rfc3986](http://www.ietf.org/rfc/rfc3986.txt) пункт `6.2.3` и видим, что на самом деле,
в случае с `http` совпадают адреса вида:
```text
http://example.com
http://example.com/
http://example.com:/
http://example.com:80/
```

Тут нужно быть осторожным. В тестировании я хочу проверить, что нахожусь на нужном мне
адресе без служебных элементов, путь даже если это и порт, потому что в продакшен версии сайта его не должно быть.
