# io.js Week of March 13th
io.js 1.5.1 и события сообщества

## Релиз io.js 1.5.1

В понедельник, 9-го мая, [@rvagg](https://github.com/rvagg) выпустил io.js [v1.5.1](https://iojs.org/dist/v1.5.1/). Полный список изменений можно найти [на GitHub-е](https://github.com/iojs/io.js/blob/v1.x/CHANGELOG.md).

### Основные изменения

* **tls**: В данном релизе, благодря множеству коммитов была решена проблема с ранее объявленной утечкой памяти в TLS. Текущие тесты указывают, что _возможно_ всё ещё существуют какие-либо проблемы связанные с утечкой памяти. Следите за полным прогрессом в [#1075](https://github.com/iojs/io.js/issues/1075).
* **http**: Решена проблема, объявленная в [joyent/node#9348](https://github.com/joyent/node/issues/9348) и [npm/npm#7349](https://github.com/npm/npm/issues/7349). Данные, находящиеся в ожидании, не были полностью обработаны при событии `'error'`, приводя к ошибке в `socket.destroy()`. (Fedor Indutny) [#1103](https://github.com/iojs/io.js/pull/1103).

### Извествные проблемы

* Возможно, ещё остались проблемы с утечкой памяти в TLS, подробная информация в [#1075](https://github.com/iojs/io.js/issues/1075).
* Windows всё ещё сообщает об ошибках в прохождении нескольких незначительных тестов. Их решения продолжают оставаться для нас приоритетной задачей. Смотрите [#1005](https://github.com/iojs/io.js/issues/1005)
* Суррогатная пара в REPL может привести к зависанию терминала [#690](https://github.com/iojs/io.js/issues/690)
* Невозможно собрать io.js как статическую библиотеку [#686](https://github.com/iojs/io.js/issues/686)
* `process.send()` не синхронный, как предполагает документация, откат введен в 1.0.2, смотрите [#760](https://github.com/iojs/io.js/issues/760) и правку в [#774](https://github.com/iojs/io.js/issues/774)
* Вызов `dns.setServers()` в то время, когда DNS-запрос ещё продолжает выполняться,  может привести к аварийному выходу процесса [#894](https://github.com/iojs/io.js/issues/894)


## Новости сообщества

* [Tony Pujals] (https://twitter.com/subfuzion) показал презентацию плана развития io.js в [BayNode](http://www.meetup.com/BayNode/events/220246228/). Видео было опубликовано в [vimeo](https://vimeo.com/121707989) 9-го марта
* [Johan Bergström](https://github.com/jbergstroem) работает надо патчем для [V8](https://codereview.chromium.org/990063002) на стороне io.js, чтобы вернуть поддержку Solaris в последнюю версию
* В [84-м эпизоде NodeUp](http://nodeup.com/eightyfour) вышла серия, посвящённая обновлению io.js. Ведущими были [Mikeal Rogers](https://github.com/mikeal), [Trevor Norris](https://github.com/trevnorris) и [Bradley Meck](https://github.com/bmeck)
* [Mikeal Rogers](https://github.com/mikeal) дал интерьвью программе [Descriptive](http://descriptive.audio) в эпизоде под названием [У нас никогда не было так много активных контрибьютеров](http://descriptive.audio/episodes/12)
* [Mark Wolfe](https://twitter.com/wolfeidau) [рассказал об io.js](https://twitter.com/wolfeidau/status/575785856545378304) на конференции [@melbjs](https://twitter.com/melbjs), слайды опубликованы [тут](https://speakerdeck.com/wolfeidau/iojs-bringing-es6-to-the-node)
* [dockeri.co](http://dockeri.co/) теперь работает на io.js. Вы можете посмотреть объявление [тут](https://twitter.com/wjblankenship/status/575867637680369665)
* [Node.js Advisory Board](https://nodejs.org/about/advisory-board/) обсуждают [предложение о примерении io.js/Node.js](https://github.com/iojs/io.js/issues/978). Вы можете ознакомиться с ходом обсуждения [здесь](https://github.com/joyent/nodejs-advisory-board/blob/master/meetings/2015-03-09/minutes.md#nodejsiojs-reconciliation-bb)

## Ожидаемые события

* [NodeConf](http://nodeconf.com/) 8-го и 9-го июня в Окленде, Калифорния, билеты доступны в продаже. А также NodeConf Adventure с 11-го по 14-е июня в Walker Creek Ranch, Калифорния.
* [CascadiaJS](http://2015.cascadiajs.com/) с 8-го по 10-е июня в штате Вашингтон, билеты доступны в продаже.
* [NodeConf EU](http://nodeconf.eu/) tickets are on sale, September 6th - 9th at Waterford, Ireland
* [NodeConf EU](http://nodeconf.eu/) c 6-го по 9-е сентября в Уотерфордe, Иралнди, билеты доступны в продаже.