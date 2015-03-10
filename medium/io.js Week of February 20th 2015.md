### Еженедельник io.js, 20 февраля 2015

Релиз 1.3.0, MongoDB, планы развития и многое другое.

### Релиз 1.3.0

Самые главные изменения:

  * **url**: url.resolve('/путь/к/файлу', '.') теперь возвращает путь с косой чертой в конце, url.resolve('/', '.') возвращает «/» [#278](https://github.com/iojs/io.js/pull/278) (Amir Saboury).
  * **tls**: Набор шифров по умолчанию, используемый tls и https был изменен и теперь обеспечивает совершенную прямую секретность (Perfect Forward Secrecy) для всех современных браузеров. Кроме этого, небезопасные RC4-шифры были исключены из набора. Если вы не можете обойтись без RC4, пожалуйста, указывайте свой собственный набор шифров. [#826](https://github.com/iojs/io.js/pull/826) (Roman Reiss)

### Важные события в сообществе

  * **Руководство Node** — [William Bert](https://twitter.com/williamjohnbert) создал <http://nodegovernance.io/> для оповещения Скотта Хэммонда, генерального директора Joyent, о желании сообщества сделать модель открытого управления io.js базой для технического комитета Node. Реакция сообщества была *фантастической*!
  * **Node.js и io.js, улучшают производительность** — Raygun.io недавно провел тесты производительности Node.js и io.js, и оба повышали производительность с каждым релизом! [Узнайте подробности в статье](https://raygun.io/blog/2015/02/node-js-performance-node-js-vs-io-js/).
  * **Основы LTTng** — [Основы LTTng](https://asciinema.org/a/16785) с io.js от пользователя jgalar на asciinema
  * **Путь развития io.js** — [презентация направления развития io.js](http://roadmap.iojs.org/).

### Добавили поддержку io.js

  * [TravisCI](https://travis-ci.org/) добавил io.js. Hiro Asari (あさり), [написал в Twitter](https://twitter.com/hiro_asari/status/566268486012633088) в прошлую пятницу, что около 10% Node проектов были запущены под io.js.
  * [@thlorenz](https://github.com/thlorenz) обновил [nad](https://github.com/thlorenz/nad) (Node Addon Developer). Теперь он [поддерживает io.js](https://twitter.com/thlorenz/status/566328088121081856).
  * В [Catberry.js](https://github.com/catberry/catberry) добавлена поддержка io.js.
  * Официальный модуль MongoDB для node.js поддерживает io.js с версии [2.0.16](https://github.com/mongodb/node-mongodb-native/blob/2.0/HISTORY.md) от 16 февраля.
  * У [The Native Web](http://www.thenativeweb.io/) появился [Docker контейнер для io.js](https://registry.hub.docker.com/u/thenativeweb/iojs/).
  * [DNSChain](https://github.com/okTurtles/dnschain) от [okTurtles](https://okturtles.com/) добавили поддержку io.js.
  * [TDPAHACLPlugin](https://github.com/neilstuartcraig/TDPAHACLPlugin) и [TDPAHAuthPlugin](https://github.com/neilstuartcraig/TDPAHAuthPlugin) для [actionHero](http://www.actionherojs.com/) теперь поддерживают io.js.
  * [node-sass](https://npmjs.org/package/node-sass) добавил поддержку io.js 1.2 в версии [2.0.1](https://github.com/sass/node-sass/issues/655).
  * [Total.js](https://www.totaljs.com/) добавил поддержку io.js в [1.7.1](https://github.com/totaljs/framework/releases/tag/v1.7.1).
  * [Clever Cloud](https://www.clever-cloud.com/) добавили [поддержку для io.js](https://www.clever-cloud.com/blog/features/2015/01/23/introducing-io.js/).

### Собрания Рабочих Групп io.js

  * Собрание io.js Tracing Working Group — 19 февраля 2015: [YouTube](https://www.youtube.com/watch?v=wvBVjg8jkv0) — [Minutes](https://docs.google.com/document/d/1_ApOMt03xHVkaGpTEPMDIrtkjXOzg3Hh4ZcyfhvMHx4/edit)
  * Собрание io.js Build Working Group — 19 февраля 2015: [YouTube](https://www.youtube.com/watch?v=OKQi3pTF7fs) — [SoundCloud](https://soundcloud.com/iojs/iojs-build-wg-meeting-2015-02-19) — [Minutes](https://docs.google.com/document/d/1vRhsYBs4Hw6vRu55h5eWTwDzS1NctxdTvMMEnCbDs14/edit)
  * Собрание io.js Technical Committee — 18 февраля 2015: [YouTube](https://www.youtube.com/watch?v=jeBPYLJ2_Yc) — [SoundCloud](https://soundcloud.com/iojs/iojs-tc-meeting-meeting-2015-02-18) — [Minutes](https://docs.google.com/document/d/1JnujRu6Rfnp6wvbvwCfxXnsjLySunQ_yah91pkvSFdQ/edit)
  * Собрание io.js Website Working Group — 16 февраля 2015: [YouTube](https://www.youtube.com/watch?v=UKDKhFV61ZA) — [SoundCloud](https://soundcloud.com/iojs/iojs-website-wg-meeting-2015-02-16) — [Minutes](https://docs.google.com/document/d/1R8JmOoyr64tt-QOj27bD19ZOWg63CujW7GeaAHIIkUs/edit)
