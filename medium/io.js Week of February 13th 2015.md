# Еженедельник io.js, 13 февраля

    локализация на 29 языках, релиз 1.2.0, и многое другое.

### Добавили поддержку io.js…
 * [Postmark](http://blog.postmarkapp.com/post/110829734198/its-official-were-getting-cozy-with-node-js)
 * [node-serialport](https://github.com/voodootikigod/node-serialport/issues/439)
 * [Microsoft Azure](http://azure.microsoft.com/en-us/documentation/articles/web-sites-nodejs-iojs/)

### Репозиторий io.js на GitHub набрал 10,000 звёзд

13 февраля, io.js добрался до отметки в 10,000 звёзд на GitHub. Мы не могли бы добиться этого без поддержки JS сообщества. Спасибо вам!

### Выпущен релиз io.js версии 1.2.0
* **stream**: [упрощенное создание.](https://github.com/iojs/readable-stream/issues/102)
* **dns**: [в lookup() добавился аргумент «all», который по умолчанию принимает значение «false», но если поставить «true», то lookup вернет массив всех  ip-адресов для указанного hostname.](https://github.com/iojs/io.js/pull/744)
* **assert**: убрали [сравнение свойства «prototype» в deepEqual()](https://github.com/iojs/io.js/pull/636) и добавили метод deepStrictEqual(), который [повторяет deepEqual()](https://github.com/iojs/io.js/pull/639), но делает строгое сравнение примитивов.
* **tracing**: [добавили поддержку LTTng (Linux Trace Toolkit Next Generation) при компиляции с опцией «--with-lttng». Точки трассировки повторяют доступные из DTrace и ETW.](https://github.com/iojs/io.js/pull/702)
* **docs**: Много изменений в документации, смотрите индивидуальные коммиты; [новая страница описывающая специфику ошибок ](https://iojs.org/api/errors.html) JavaScript, V8 и io.js.
* **npm** обновился до версии 2.5.1
* **libuv **обновился до версии 1.4.0, прочитать полный список изменений вы можете [тут](https://github.com/libuv/libuv/blob/v1.x/ChangeLog).

В команду io.js добавились: Aleksey Smolenchuk ([@lxe](https://github.com/lxe)) и Shigeki Ohtsu ([@shigeki](https://github.com/shigeki))

### Мы открыли двери мировому сообществу

Оригинальную статью вы можете посмотреть [здесь](https://medium.com/@mikeal/how-io-js-built-a-146-person-27-language-localization-effort-in-one-day-65e5b1c49a62).

* В команды различных языков добавились заинтересованные участники.

* Команды зарегистрировали аккаунты в твиттере и других социальных сетях.

* Каждая комманда придумала свой процесс работы и стала уже не просто переводчиками, а организаторами сообщества.

#### Статистика по локализации

* в первый день 146 участников выразили желание помочь в локализации (сейчас их число уже больше 160)

* 27 языковых групп было создано в первый день (сейчас их 29)

#### Ссылки на репозитории

* [iojs-bn](https://github.com/iojs/iojs-bn) бенгальский
* [iojs-cn](https://github.com/iojs/iojs-cn) китайский
* [iojs-cs](https://github.com/iojs/iojs-cs) чешский
* [iojs-da](https://github.com/iojs/iojs-da) датский
* [iojs-de](https://github.com/iojs/iojs-de) немецкий
* [iojs-el](https://github.com/iojs/iojs-el) греческий
* [iojs-es](https://github.com/iojs/iojs-es) испанский
* [iojs-fa](https://github.com/iojs/iojs-fa) персидский
* [iojs-fi](https://github.com/iojs/iojs-fi) финский
* [iojs-fr](https://github.com/iojs/iojs-fr) французский
* [iojs-he](https://github.com/iojs/iojs-he) иврит
* [iojs-hi](https://github.com/iojs/iojs-hi) хинди
* [iojs-hu](https://github.com/iojs/iojs-hu) венгерский
* [iojs-id](https://github.com/iojs/iojs-id) индонезийский
* [iojs-it](https://github.com/iojs/iojs-it) итальянский
* [iojs-ja](https://github.com/iojs/iojs-ja) японский
* [iojs-ka](https://github.com/iojs/iojs-ka) грузинский
* [iojs-kr](https://github.com/iojs/iojs-kr) корейский
* [iojs-mk](https://github.com/iojs/iojs-mk) македонский
* [iojs-nl](https://github.com/iojs/iojs-nl) нидерландский
* [iojs-no](https://github.com/iojs/iojs-no) норвежский
* [iojs-pl](https://github.com/iojs/iojs-pl) польский
* [iojs-pt](https://github.com/iojs/iojs-pt) португальский
* [iojs-ro](https://github.com/iojs/iojs-ro) румынский
* [iojs-ru](https://github.com/iojs/iojs-ru) русский
* [iojs-sv](https://github.com/iojs/iojs-sv) шведский
* [iojs-tr](https://github.com/iojs/iojs-tr) турецкий
* [iojs-tw](https://github.com/iojs/iojs-tw) тайваньский
* [iojs-uk](https://github.com/iojs/iojs-uk) украинский

### io.js и node.js

Оригинальную статью вы можете прочитать [тут](https://medium.com/@iojs/io-js-and-a-node-js-foundation-4e14699fb7be).

* Scott Hammond, глава компании Joyent, выразил желание вернуть io.js в node.js.

#### Всего за несколько месяцев проект io.js…

* Вырос до 23 активных участников основной команды

* Нескольких рабочих групп

* 29 команд по локализации,

* Привлек больше участников чем node.js за все время его существования

* При поддержке сообщества выпускает качественное ПО.

_Мы не можем жертвовать прогрессом, принципами и той моделью управления, которую мы внедрили._

#### Будущее

* Ведутся переговоры с node.js.

* Как только будет выбрана оптимальная модель управления, будет проведен опрос на GitHub о том стоит ли io.js присоединиться к node.js. Обсуждение и голосование по этому вопросу пройдет на публичном заседании технического комитета.

_Для сообщества ничего не меняется._

#### Что вы можете сделать сейчас

* Продолжить присылать pull-реквесты

* Добавиться к одной из 27 [команд локализации](https://github.com/iojs/website/issues/125)

* Помочь одной из рабочих групп ([streams](https://github.com/iojs/readable-stream), [website](https://github.com/iojs/website), [evangelism](https://github.com/iojs/website/labels/evangelism), [tracing](https://github.com/iojs/tracing-wg), [build](https://github.com/iojs/build), [roadmap](https://github.com/iojs/roadmap)) и

* Продолжить использовать io.js в ваших проектах.



Это перевод статьи «io.js Week of February 13th 2015», которую вы можете найти на [medium](https://medium.com/node-js-javascript/io-js-week-of-february-13th-2015-7846b94074a2).
