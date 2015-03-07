# Еженедельник io.js, 6 марта 2015

Buffer.indexOf(), Tessel 2 и многое другое.

## Релиз io.js 1.5.0

В пятницу 6-го марта [@rvagg](https://github.com/rvagg) выпустил io.js [**v1.5.0**](https://iojs.org/dist/latest/).  Полный список изменений можно найти [на GitHub-е](https://github.com/iojs/io.js/blob/v1.x/CHANGELOG.md).

### Основные изменения

* **buffer**: Новый метод `Buffer#indexOf()`, подобный [`Array#indexOf()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/indexOf). Может принимать в качестве аргумента строку, Buffer или число. Строки интерпретируются как UTF8. (Trevor Norris) [#561](https://github.com/iojs/io.js/pull/561)
* **fs**: Объект параметров `options` в методах модуля `'fs'` больше не выполняет проверку `hasOwnProperty()`, позволяя объектам параметров иметь свойства в прототипах, которые будут применяться. (Jonathan Ong) [#635](https://github.com/iojs/io.js/pull/635)
* **tls**: Компания PayPal сообщила о вероятной утечке памяти в TLS. Некоторые недавние изменения в  **stream_wrap** оказались виновными в этом. Первоночальная правка расположена в [#1078](https://github.com/iojs/io.js/pull/1078), вы можете следить за прогрессом устранения утечки в [#1075](https://github.com/iojs/io.js/issues/1075) (Fedor Indutny).
* **npm**: Обновление npm до 2.7.0. Смотрите [npm CHANGELOG.md](https://github.com/npm/npm/blob/master/CHANGELOG.md#v270-2015-02-26), чтобы получить более подробную информацию, включая почему это минорное увеличение версии, хотя могло бы быть мажорным.
* **TC**: Colin Ihrig (@cjihrig) отказался от участия в Техническом Комитете из-за желания больше писать код и меньше участвовать в собраниях.

### Известные проблемы

* Возможна утечка памяти в TLS, более подробную информацию можно найти в [#1075](https://github.com/iojs/io.js/issues/1075).
* Некоторые тесты в ОС Windows всё ещё завершаются неудачно. Исправление связанных с ними ошибок является для нас приоритетной задачей. Смотрите [#1005](https://github.com/iojs/io.js/issues/1005).
* Суррогатная пара в REPL может привести к зависанию терминала [#690](https://github.com/iojs/io.js/issues/690)
* Невозможно собрать io.js как статическую библиотеку [#686](https://github.com/iojs/io.js/issues/686)
* `process.send()` не синхронный, как предполагает документация, откат введен в 1.0.2, смотретие [#760](https://github.com/iojs/io.js/issues/760) и правку в [#774](https://github.com/iojs/io.js/issues/774)

## Новости сообщества

* Вы можете спать спокойно, зная что io.js и последний node.js [**не подвержены**](https://strongloop.com/strongblog/are-node-and-io-js-affected-by-the-freak-attack-openssl-vulnerability/) [FREAK Аттаке](https://freakattack.com/).  Вы ведь запускаете io.js или последнюю версию node.js, не так ли?
* Walmart теперь спонсируют сборочную машину для системы io.js Jenkins CI.  Команда @iojs/build работает над созданием бинарников io.js для SunOS (подобно тем, которые вы можете получить на nodejs.org).  Необходимо внести правку в V8 ([iojs/io.js#1079](https://github.com/iojs/io.js/pull/1079)) для продолжения работы.
* Мы также хотели бы поблагодорить следующие компании за их вклад в технологии/поддержку/проектирование/оборудование для сборок io.js:
  * **Digital Ocean** (Linux)
  * **Rackspace** (Windows)
  * **Voxer** (OS X и FreeBSD)
  * **NodeSource** (ARMv6 & ARMv7)
  * **Linaro** (ARMv8)
  * **Walmart** (SmartOS / Solaris)
* Сообщество io.js усердно работает над интернационализацией всего своего контента. Более 20 языков опубликовано на [iojs.org](http://iojs.org) и международных сайтах сообщества. В дополнение к этому, в футер веб-сайта были добавлены ссылки на переводы ([iojs/website#258](https://github.com/iojs/website/pull/258)) для более удобного доступа. Ваш язык отсутствует?  [Помогите нам добавить его!](https://github.com/iojs/website/blob/master/TRANSLATION.md)
* Говоря о переводах, в [презентацию плана развития io.js](http://roadmap.iojs.org/) были добавлены ссылки на переводы на другие языки.
* Кажется, **PayPal** проводит эксперимент, сравнивая работу [Kappa](https://www.npmjs.com/package/kappa) на io.js, node.js 0.12, node.js v0.10.  Команда PayPal обнаружила вероятную утечку памяти в TLS. Первоначальная правка расположена в [#1078](https://github.com/iojs/io.js/pull/1078), а прогресс работы в отношении устранения утечки в [#1075](https://github.com/iojs/io.js/issues/1075)
* [**NodeSource**](http://nodesource.com) теперь поддерживает io.js. Пакет [Linux binary](https://nodesource.com/blog/nodejs-v012-iojs-and-the-nodesource-linux-repositories) как для Ubuntu/Debian, так и для дистрибутива RHEL/Fedora.
* io.js [Docker build](https://registry.hub.docker.com/u/library/iojs/) - одна из тринадцати новых [официальных Docker репозиториев](http://blog.docker.com/2015/03/thirteen-new-official-repositories-added-in-january-and-february/), добавленных в январе и феврале.
* Люди, интересующиеся NodeBots и IoT, должны быть счастливы услышать, что [**Tessel2**](http://blog.technical.io/post/112787427217/tessel-2-new-hardware-for-the-tessel-ecosystem) теперь запускает [io.js нативно](http://blog.technical.io/post/112888410737/moving-faster-with-io-js).
* [**@maxbeatty**](https://twitter.com/maxbeatty) работает над новой версией бэкэнда [jsperf.com](http://jsperf.com/), запускаемой на io.js и полностью [с открытым исходным кодом](https://github.com/jsperf/jsperf.com).  Желающие помочь, добро пожаловать!
* [@eranhammer](https://twitter.com/eranhammer) написал пост под названием [The Node Version Dilemma](http://hueniverse.com/2015/03/02/the-node-version-dilemma/), который обсуждает различные версии node.js / io.js и предлагает когда и какие из них можно использовать.

## Добавили поддержку io.js

* **[scrypt](https://npmjs.com/scrypt)** теперь поддерживает io.js. Узнайте больше из [GitHub issue](https://github.com/barrysteyn/node-scrypt/issues/39)
* **[proxyquire](https://github.com/thlorenz/proxyquire)** v1.3.2 опубликована с поддержкой iojs.