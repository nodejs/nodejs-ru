# Stream

    Стабильность: 2 - Стабильный

Stream это абстрактный интерфейс реализованный разными объектами
в io.js. Например [запрос к HTTP серверу](http.html#http_http_incomingmessage) - это stream,
[stdout] - тоже stream. Streams могут быть Readable (читаемые), Writable (записываемые)
или одновременно (Duplex и Transform).
Все streams унаследованы от [EventEmitter][].

Вы можете подключить основные stream классы через `require('stream')`.
Этот модуль обеспечивает [Readable][], [Writable][],
[Duplex][] и [Transform][] streams классы.

Далее документ разделен на 3 части.

**Первая** объясняет API streams
для тех, кто будет просто использовать их в своей программе.
Если вы никогда не будете реализовывать Stream API,
вы можете остановиться здесь.

**Вторая часть** исчерпывающе объясняет элементы API Streams необходимые
для реализации ваших собственных классов Stream.

**Третья часть** даёт более углубленное объяснение того, как работают streams,
включая некоторые внутренние механизмы и функции,
которые, возможно, не должны быть модифицированы до того
как вы не поймете, как они работают.

## API для Пользователей Streams

<!--type=misc-->

Streams могуть быть либо [Readable][], либо [Writable][],
либо одновременно ([Duplex][],[Transform][]).

Все streams унаследованы от EventEmitters, и имеют свои
специальные методы и свойства зависящие от вида stream:
Readable, Writable, или Duplex.

Если stream одновременно Readable и Writable, значит он реализует все их методы и
события которые описаны ниже. API [Duplex][]
и [Transform][] streams также подробно описаны,
хотя их реализация может несколько отличаться.

Если вы желаете создать свои особенные
streaming интерфейсы, знать это не столь важно.
Более подробно читайте часть [API для реализации Stream][],
что расположена ниже.

Почти все io.js программы, так или иначе,
используют Streams в разных целях.
Вот к примеру простая io.js программа, использующая Streams:

```javascript
var http = require('http');

var server = http.createServer(function (req, res) {
  // req - это http.IncomingMessage, реализованный Readable Stream
  // res - это http.ServerResponse,  реализованный Writable Stream
  var body = '';
  // Мы будем получать данные как utf8 strings
  // Если не установить нужную кодировку, по умолчанию мы получим объекты класса Buffer
  req.setEncoding('utf8');

  // Readable streams будет посылать событие 'data'
  // при их получении. Просто добавляем к нему обработчик.
  req.on('data', function (chunk) {
    body += chunk;
  });

  // Событие 'end', говорит нам, что запись данных в body зверешена
  req.on('end', function () {
    try {
      var data = JSON.parse(body);
    } catch (er) {
      // Ой-ой, плохой Json!
      res.statusCode = 400;
      return res.end('error: ' + er.message);
    }

    // Напишем что-нибудь интересное в ответ пользователю
    res.write(typeof data);
    res.end();
  });
});

server.listen(1337);

// Далее,с помощью curl , отправляем запросы к нашей программе
// предварительно запустив её: $ node program.js
// Атрибут - d означает DATA. (примеч. переводчика.)

// $ curl localhost:1337 -d '{}'
// object
// $ curl localhost:1337 -d '"foo"'
// string
// $ curl localhost:1337 -d 'not json'
// error: Unexpected token o
```

### Class: stream.Readable

<!--type=class-->

Readable stream интерфейс это просто абстракция для вашего некоего
*источника* данных, которые вы хотите получить.
Другими словами - данные *приходят* из Readable stream.

Он не будет выдавать данные, до тех пор пока
вы не будете готовы получать их.

У Readable streams есть два режима:

* **flowing mode**
* **paused mode**

В режиме **flowing mode** - данные читаются непосредственно
(и быстро -насколько это возможно) из специальной underlying системы
(приватного метода,именованного с "_" -прим. переводчика).

В режиме **paused mode** - данные, по умолчанию - в виде кусков (chunk),
должны быть получены явным вызовом `stream.read()`.

**Будьте внимательны**: если у события 'data' не был установлен
обработчик, а у вызова [`pipe()`][] не указан получатель
и stream находится в режиме **flowing mode** - данные будут утеряны.


В **flowing mode** можно переключиться с помощью:

* Добавлением к событию '[`data`][]' обработчика данных.
* Вызовом метода [`resume()`][] для явного открытия потока данных.
* Вызовом метода [`pipe()`][] с указанием получателя [Writable][],
для отправки ему данных.

Переключиться обратно, в **paused mode**, можно путём:

* Если получатель не был указан (в методе .pipe) - вызовом [`pause()`][]
* Если получатель(-ли) был указан - удалением обработчика события '[`data`][]'
  и удалением всех pipe получателей через [`unpipe()`][] метод.

**Имейте также ввиду**: в целях обратной совместимости, удаление
всех `'data'` событий **не** будет автоматически приостанавливать
(pause) stream. Также, если были указаны piped-получатели,
и затем вызван `pause()`, stream не будет гарантировать
что поток выдачи данных будет приостановленен, в момент,
когда все piped-получатели получат данные и запросят их остатки.

Вот примеры Readable streams:

* [http responses, on the client](http.html#http_http_incomingmessage)
* [http requests, on the server](http.html#http_http_incomingmessage)
* [fs read streams](fs.html#fs_class_fs_readstream)
* [zlib streams][]
* [crypto streams][]
* [tcp sockets][]
* [child process stdout and stderr][]
* [process.stdin][]

#### Событие: 'readable'

Когда кусок (chunk) данных может быть прочитан stream,
он будет посылать `'readable'` событие.

В некоторых случаях, прослушка `'readable'` события,
будет вызывать считывание данных во внутренний
*буффер* из  underlying системы, если это не сделано ранее.

```javascript
var readable = getReadableStreamSomehow();
readable.on('readable', function() {
  // Данные для чтения будут доступны здесь
});
```
В момент освобождения внутреннего буфера (данные *вытекут* - drained),
и остальная часть данных будет доступна,
обработчик этого `readable` события будет снова вызван.

#### Событие: 'data'

* `chunk` {Buffer | String} Кусок данных.

Установка обработчика события `data` для stream, который
не был явно приостановлен, будет переключать его в flowing mode,
и данные, как только станут доступны, будут переданы получателю.

Это лучший способ получить данные, как только они станут доступны.

```javascript
var readable = getReadableStreamSomehow();
readable.on('data', function(chunk) {
  console.log('получено %d байтов данных', chunk.length);
});
```

#### Событие: 'end'

Обработчик этого события вызывается, когда
данные больше не могут быть считаны.

Имейте ввиду, что это событие, не будет послано
до тех пор, пока данные не будут полностью получены
получателем, что, в свою очередь, возможно либо в flowing mode (см.выше),
либо многократным (по мере поступления данных) вызовом метода `read()` (об этом ниже).

```javascript
var readable = getReadableStreamSomehow();
readable.on('data', function(chunk) {
  console.log('получено %d байтов данных', chunk.length);
});
readable.on('end', function() {
  console.log('здесь больше не будет данных.');
});
```

#### Событие: 'close'

Посылается в момент закрытия внутреннего underlying ресурса
(например в момент возвращения дескриптора файла). Не все
streams будут посылать это событие.

#### Событие: 'error'

* {Error Object}

Посылается при получение данных в процессе которого произошла ошибка.

#### readable.read([size])

* `size` {Number}  Опциональный аргумент, для указания количества данных для считывания.
* Return {String | Buffer | null}

Метод `read()` возвращает данные из внутреннего буффера.
Если данные не доступны - null.

Здесь `size` указывает методу количество возвращаемых байтов.

Если `size` недоступен - возвращает null.

Если `size` не указан, метод вернет все данные из буффера.

Этот метод должен вызываться только в `paused mode`.
В `flowing mode` этот метод вызывается автоматически,
вплоть до опустошения внутреннего буффера.

```javascript
var readable = getReadableStreamSomehow();
readable.on('readable', function() {
  var chunk;
  while (null !== (chunk = readable.read())) {
    console.log('получено %d байтов данных', chunk.length);
  }
});
```

При каждом вызове метода, stream посылает событие [`'data'`][].

#### readable.setEncoding(encoding)

* `encoding` {String} Кодировка данных.
* Return: `this`

Вызов этой функции заставит stream выдавать данные в определенной
кодировке.

Например: `readable.setEncoding('utf8')` - будет выдавать
данные в виде UTF-8 strings.

Или:`readable.setEncoding('hex')` - данные будут представлены в hexadecimal
string формате.

Этот метод правильно обрабатывает multi-byte символы,
что бы предотвратить порчу данных, возникающую при
извлечение данных напрямую из внутреннего буффера
и вызове на них `buf.toString(encoding)`.
Если вы хотите читать данные как strings, всегда используйте этот метод.

```javascript
var readable = getReadableStreamSomehow();
readable.setEncoding('utf8');
readable.on('data', function(chunk) {
  assert.equal(typeof chunk, 'string');
  console.log('получено %d символов из строки данных', chunk.length);
});
```

#### readable.resume()

* Return: `this`

Метод позволяет readable stream продолжить посылку события `data`.

Метод переключает stream в flowing mode.
Если вы не хотите получать данные из stream, но вы
вы хотите полность окончить приём и послать событие `end`,
вы можете вызвать [`readable.resume()`][] для продолжения
выдачи данных.

```javascript
var readable = getReadableStreamSomehow();
readable.resume();
readable.on('end', function(chunk) {
  console.log('выдача данных завершена, но ничего не прочитано');
});
```

#### readable.pause()

* Return: `this`

Метод приостанавливает выдачу данных и посылку
`data` события, переключая stream из `flowing mode` в `pause mode`.
Оставшиеся доступные данные остаются во внутреннем буфере.

```javascript
var readable = getReadableStreamSomehow();
readable.on('data', function(chunk) {
  console.log('получено %d байт данных', chunk.length);
  readable.pause();
  console.log('остальные данные, на одну секунду, будут недоступны');
  setTimeout(function() {
    console.log('теперь поток данных открыт');
    readable.resume();
  }, 1000);
});
```

#### readable.isPaused()

* Return: `Boolean`

Метод сообщит приостановлен ли `readable` stream явно
клиентским кодом (с использованием `readable.pause()`
без `readable.resume()`).

```javascript
var readable = new stream.Readable

readable.isPaused() // === false
readable.pause()
readable.isPaused() // === true
readable.resume()
readable.isPaused() // === false
```

#### readable.pipe(destination[, options])

* `destination` {[Writable][] Stream} Получатель данных.
* `options` {Object} Опции передачи данных (piping).
  * `end` {Boolean} Послать событие 'end' получателю, когда считывание данных будет окончено. По умолчанию = `true`

Метод вытягивает данные из readable stream и
записывает их в получателя, автоматически управляя потоком
передачи, дабы получатель не был перегружен быстрым
readable stream.


```javascript
var readable = getReadableStreamSomehow();
var writable = fs.createWriteStream('file.txt');
// Все данные из readable пойдут в 'file.txt'
readable.pipe(writable);
```
Доступен безопасный *слив* данных в множество получателей.

Функция просто вернет получателя, что позволяет
создавать цепочки вызовов:

```javascript
var r = fs.createReadStream('file.txt');
var z = zlib.createGzip();
var w = fs.createWriteStream('file.txt.gz');
r.pipe(z).pipe(w);
```
(Это доступно благодаря комбинированным
streams - transform и duplex, о них ниже
- примеч. переводчика).

Например, эмулируем Unix `cat` комманду:

```javascript
process.stdin.pipe(process.stdout);
```

По умолчанию [`end()`][] вызывается на получателе, когда
stream-источник посылает `end` событие, так что
`получатель` становится больше не writable. Передача `{ end: false }`
как `параметр` в этот метод, сохраняет stream-получателя открытым (writable).

Это позволяет *попрощаться* с получателем (writable stream)
в нужный момент:

```javascript
reader.pipe(writer, { end: false });
reader.on('end', function() {
  writer.end('Goodbye\n');
});
```

Имейте ввиду, `process.stderr` и `process.stdout`
не будут закрыты, до тех пор, пока процесс существует,
независимо от указанных параметров.

#### readable.unpipe([destination])

* `destination` {[Writable][] Stream} Для указания получателя, опционально.

Метод удаляет хуки установленные для предыдущенго вызова `pipe()`.

Если получатель не указан, будуте удалены все остальные,
ранее указанные.

Если получатель указан, но передача (*слив* ) данных в процессе
- метод ничего не сделает.

```javascript
var readable = getReadableStreamSomehow();
var writable = fs.createWriteStream('file.txt');
// Все данные из readable источника переходят в 'file.txt',
// но только в первую секунду
readable.pipe(writable);
setTimeout(function() {
  console.log('остановить запись file.txt');
  readable.unpipe(writable);
  console.log('закроем file stream в ручную');
  writable.end();
}, 1000);
```

#### readable.unshift(chunk)

* `chunk` {Buffer | String} Кусок данных для извлечения из очереди чтения и сдвига его назад.

Это полезно в определенных случаях, когда stream используется
парсером, который нуждался временно *не использовать*
данные из очередни, которые он (парсер) получал, так, что бы
stream мог передать их другой партией.

Если вы очень часто вызываете этот метод,
рассмотрите реализацию [Transform][] stream.
(см. ниже по статье).

```javascript
// Вытащим header отделенный \n\n
// используя unshift() если получим их слишком много
// Вызовем callback с (error, header, stream)
var StringDecoder = require('string_decoder').StringDecoder;
function parseHeader(stream, callback) {
  stream.on('error', callback);
  stream.on('readable', onReadable);
  var decoder = new StringDecoder('utf8');
  var header = '';
  function onReadable() {
    var chunk;
    while (null !== (chunk = stream.read())) {
      var str = decoder.write(chunk);
      if (str.match(/\n\n/)) {
        // Нашли границу header
        var split = str.split(/\n\n/);
        header += split.shift();
        var remaining = split.join('\n\n');
        var buf = new Buffer(remaining, 'utf8');
        if (buf.length)
          stream.unshift(buf);
        stream.removeListener('error', callback);
        stream.removeListener('readable', onReadable);
        // Теперь тело сообщения может быть прочитано из stream
        callback(null, header, stream);
      } else {
        // Все ещё читаем header.
        header += str;
      }
    }
  }
}
```

#### readable.wrap(stream)

* `stream` {Stream} Старый вид readable stream.

До версии  v0.10, API streams не были полноценно реализованы,
(подробно, о обратной "Совместимости"  см. ниже) поэтому,
если вы использовали старую библиотеку io.js, которая
посылает `'data'` события, и имеет рекомендальный метод [`pause()`][], тогда
вы можете "обернуть" методом `wrap()` для создания [Readable][] stream, которые использует
старые stream с их источниками.

Вы будете редко вызывать эту функцию, т.к. она удобна только
в случае взаимодействия со старыми io.js библиотеками и программами.

Например:

```javascript
var OldReader = require('./old-api-module.js').OldReader;
var oreader = new OldReader;
var Readable = require('stream').Readable;
var myReader = new Readable().wrap(oreader);

myReader.on('readable', function() {
  myReader.read(); // etc.
});
```


### Class: stream.Writable

<!--type=class-->

Writable stream интерфейс это просто абстракция для вашего для
вашего *получателя*, который *принимать* данные.

Примеры writable streams включают:

* [http requests, on the client](http.html#http_class_http_clientrequest)
* [http responses, on the server](http.html#http_class_http_serverresponse)
* [fs write streams](fs.html#fs_class_fs_writestream)
* [zlib streams][]
* [crypto streams][]
* [tcp sockets][]
* [child process stdin](child_process.html#child_process_child_stdin)
* [process.stdout][], [process.stderr][]

#### writable.write(chunk[, encoding][, callback])

* `chunk` {String | Buffer} Данные для записи.
* `encoding` {String} Кодировка, если `chunk` это String.
* `callback` {Function} Callback для вызова в момент сброса кусков (chunks) данных.
* Returns: {Boolean} True - если данные полностью обработаны.

Метод записывает некоторые данные в underlying систему,
и вызывает, один раз, указанный callback, когда
данные будут полностью обработаны.

Возвращаемое значение предупреждает, что вы должны продолжить
запись прямо сейчас. Если данные были буфферизированны, то
метод вернет `false`. В остальных случаях - `true`.

Это событие строго рекомендательное. Вы возможно продолжите
запись данных, даже если будет возвращено `false`, хотя
это будет излишне, несмотря на то, что данные также будут отправлены в буффер,
лучше выждите событие `drain` и продолжайте запись.

#### Событие: 'drain'

Если вызов [`writable.write(chunk)`][] возвращает false, тогда событие `drain`
будет сообщать когда удобно начать новую запись данных в stream.

```javascript
// Запись данных 1MM раз в указанный writable stream.
// Будьте внимательны к обратной реакции.
function writeOneMillionTimes(writer, data, encoding, callback) {
  var i = 1000000;
  write();
  function write() {
    var ok = true;
    do {
      i -= 1;
      if (i === 0) {
        // Последний момент!
        writer.write(data, encoding, callback);
      } else {
        // смотрим, должны ли мы выждать или продолжить
        // не передаем callback, т.к. мы еще не закончили.
        ok = writer.write(data, encoding);
      }
    } while (i > 0 && ok);
    if (i > 0) {
      // остновимся раньше!
      // и запишем еще немного данных
      writer.once('drain', write);
    }
  }
}
```

#### writable.cork()

Заставляет буфферизировать все данные при записи в stream.

Данные в буфере могут быть *"удалены"*
по вызову `.uncork()` или `.end()`.

#### writable.uncork()

*"Удаляет"* данные из буффера, с момента последнего вызова`.cork()`.

#### writable.setDefaultEncoding(encoding)

* `encoding` {String} Новая кодировка по умолчанию.

Устанавливает кодировку по умолчанию.

#### writable.end([chunk][, encoding][, callback])

* `chunk` {String | Buffer} Опцианально - данные для записи.
* `encoding` {String} Кодировка, если `chunk` будет в виде string.
* `callback` {Function} Опциально - Вызывается по окончании работы stream.

Вызывайте этот этот метод, по окончании передачи данных в этот
stream. Если callback был указан, он станет обработчиом события `finish`.

Вызов [`write()`][]  после вызова метода [`end()`][] вызовет ошибку.

```javascript
// напишем 'hello, ' а затем, в конце, 'world!'
var file = fs.createWriteStream('example.txt');
file.write('hello, ');
file.end('world!');
// Запись данных больше не доступна!
```

#### Событие: 'finish'

Когда вызван [`end()`][], и данные были отосланы в underlying систему,
stream посылает это событие.

```javascript
var writer = getWritableStreamSomehow();
for (var i = 0; i < 100; i ++) {
  writer.write('hello, #' + i + '!\n');
}
writer.end('this is the end\n');
writer.on('finish', function() {
  console.error('Запись данных окончена.');
});
```

#### Событие: 'pipe'

* `src` {[Readable][] Stream} Stream-источник, которые передает данные в текущий stream.

Это событие посылается каждый раз, когда вызывается метод `pipe()`
у Readable stream.

```javascript
var writer = getWritableStreamSomehow();
var reader = getReadableStreamSomehow();
writer.on('pipe', function(src) {
  console.error('что-то передаём во writer');
  assert.equal(src, reader);
});
reader.pipe(writer);
```

#### Событие: 'unpipe'

* `src` {[Readable][] Stream} Stream-источник, что [unpiped][] (отменил передачу данных) в текущий stream.

Это событие посылается каждый раз, когда метод [`unpipe()`][] будет вызван
у источника (реализующего Readable методы),
удаляющего текущий stream из набора получателей.

```javascript
var writer = getWritableStreamSomehow();
var reader = getReadableStreamSomehow();
writer.on('unpipe', function(src) {
  console.error('почему-то передача данных в writer остановлена');
  assert.equal(src, reader);
});
reader.pipe(writer);
reader.unpipe(writer);
```

#### Событие: 'error'

* {Error object}

Событие посылается, если во время записи или передачи данных
произошла ошибка.

### Class: stream.Duplex

Duplex streams это streams реализующий [Readable][] и
[Writable][] интерфесы. Смотрите выше, для использования.

Примеры Duplex streams:

* [tcp sockets][]
* [zlib streams][]
* [crypto streams][]


### Class: stream.Transform

Transform streams это [Duplex][] streams где выходящие данных,
предварительно, перед передачей, были обработаны особым образом.
Этот тип stream реализует [Readable][] и [Writable][] интерфесы.
Смотрите выше, для использования.

Примеры Transform streams:

* [zlib streams][]
* [crypto streams][]


## API для реализации Stream

<!--type=misc-->

Стандартный порядок реализации ваших
собственных классов stream обычно таков:

1. Расширьте подходящий класс stream в вашем классе.
   (Обязательно используйте [`util.inherits`][])
2. Вызовите конструктор наследуемого класса в вашем
   конструкторе.
3. Реализуйте один или несколько специальных методв, что описаны ниже.

Класс для расширения и методы необходимые для этого,
зависят от вида класса, которые вы хотите реализовать:

<table>
  <thead>
    <tr>
      <th>
        <p>Предмет использования</p>
        <p>(с точки зрения пользователя вашим классом).</p>
      </th>
      <th>
        <p>Класс, от которого нужно наследовать интерфейс.</p>
      </th>
      <th>
        <p>Интерфейс для реализации.</p>
      </th>
    </tr>
  </thead>
  <tr>
    <td>
      <p>Только чтение данных.</p>
    </td>
    <td>
      <p>[Readable](#stream_class_stream_readable_1)</p>
    </td>
    <td>
      <p><code>[_read][]</code></p>
    </td>
  </tr>
  <tr>
    <td>
      <p>Только запись данных.</p>
    </td>
    <td>
      <p>[Writable](#stream_class_stream_writable_1)</p>
    </td>
    <td>
      <p><code>[_write][]</code>, <code>_writev</code></p>
    </td>
  </tr>
  <tr>
    <td>
      <p>Запись и чтение.</p>
    </td>
    <td>
      <p>[Duplex](#stream_class_stream_duplex_1)</p>
    </td>
    <td>
      <p><code>[_read][]</code>, <code>[_write][]</code>, <code>_writev</code></p>
    </td>
  </tr>
  <tr>
    <td>
      <p>Оперирование над данным и запись (получение) их результатов.</p>
    </td>
    <td>
      <p>[Transform](#stream_class_stream_transform_1)</p>
    </td>
    <td>
      <p><code>_transform</code>, <code>_flush</code></p>
    </td>
  </tr>
</table>

В вашей программе реализации, очень важно, что бы
методы описанные в [API для Пользователей Streams][] (выше)
не были никогда вызваны. В противном случае, это потенциально может привести
к неприятным эффектам в программе, использующей ваш класс.

### Class: stream.Readable

<!--type=class-->

`stream.Readable` это абстрактный интерфейс, разработанный для расширения
с помощью underlying реализации [`_read(size)`][] метода.

Как пользоваться streams - смотрите выше - [API для Пользователей Streams][].
Здесь же, следует объяснение особенностей реализации собственныъ Readable streams
в ваших программах.

#### Пример: A Counting Stream

<!--type=example-->

Это базовый пример Readable stream.
Он посылает цифры от 1 до 1 000 000
и затем завершается работу.

```javascript
var Readable = require('stream').Readable;
var util = require('util');
// Наследуем методы Readable stream
util.inherits(Counter, Readable);

function Counter(opt) {
  // Наследуем свойства Readable stream
  Readable.call(this, opt);
  this._max = 1000000;
  this._index = 1;
}

// Реализуем специальный underlying метод
Counter.prototype._read = function() {
  var i = this._index++;
  if (i > this._max)
   // Вызов специального метода
   // по окончании потока данных
   // подробно о нём - ниже
    this.push(null);
  else {
    var str = '' + i;
    var buf = new Buffer(str, 'ascii');
    this.push(buf);
  }
};
```

#### Пример: SimpleProtocol v1 (Sub-optimal)

Этот stream очень похож на `parseHeader` функцию (что выше), но
реализован как кастомный stream. Примите также во внимание,
что наш случай не конвертирует приходящие данные в string.

Реализовать наш пример, лучше как [Transform][] stream.
Смотрите ниже, более лучшую реализацию.

```javascript
// Парсер для простого протокола данных.
// "Header" это JSON объект перед 2-мя \n символами
/  далее - тело сообщения.
//
// Внимание!: как было сказано ранее,
// эта реализация менее оптимальна, чем в виде Transform stream!
// Другой пример, находится в секции Transform stream.

var Readable = require('stream').Readable;
var util = require('util');

util.inherits(SimpleProtocol, Readable);

function SimpleProtocol(source, options) {
  if (!(this instanceof SimpleProtocol))
    return new SimpleProtocol(source, options);

  Readable.call(this, options);
  this._inBody = false;
  this._sawFirstCr = false;

  // source здесь readable stream, как socket или file
  this._source = source;

  var self = this;
  source.on('end', function() {
    self.push(null);
  });

  // дадим небольшой пинок, каждый раз, когда source станет readable
  // read(0) просто получит данные нулевой длины
  source.on('readable', function() {
    self.read(0);
  });

  this._rawHeader = [];
  this.header = null;
}

SimpleProtocol.prototype._read = function(n) {
  if (!this._inBody) {
    var chunk = this._source.read();

    // если у источника нет данных - у нас тоже
    if (chunk === null)
      return this.push('');

    // проверим кусок (chunk) данных на наличие \n\n
    var split = -1;
    for (var i = 0; i < chunk.length; i++) {
      if (chunk[i] === 10) { // '\n'
        if (this._sawFirstCr) {
          split = i;
          break;
        } else {
          this._sawFirstCr = true;
        }
      } else {
        this._sawFirstCr = false;
      }
    }

    if (split === -1) {
      // всё еще ждем для \n\n
      // спрячем chunk и попробуем снова
      //
      this._rawHeader.push(chunk);
      this.push('');
    } else {
      this._inBody = true;
      var h = chunk.slice(0, split);
      this._rawHeader.push(h);
      var header = Buffer.concat(this._rawHeader).toString();
      try {
        this.header = JSON.parse(header);
      } catch (er) {
        this.emit('error', new Error('invalid simple protocol data'));
        return;
      }
      // теперь, т.к. мы получили дополнительные данные, вытащим остатки
      // вернем их в очередь для чтения, что бы пользователь увидел их
      var b = chunk.slice(split);
      this.unshift(b);

      // и дадим знать, что закончили парсинг header.
      this.emit('header', this.header);
    }
  } else {
    // начиная отсюда, просто отдадим данныхе нашему пользователю
    // будьте осторожны с push(null), поскольку это будет означать EOF
    var chunk = this._source.read();
    if (chunk) this.push(chunk);
  }
};

// Использование:
// var parser = new SimpleProtocol(source);
// Теперь парсер, это readable stream, который будет посылать событие 'header'
// со спарсенными header-ами.
```


#### new stream.Readable([options])

* `options` {Object}
  * `highWaterMark` {Number}  Максиальное количества байтов
    для хранения во внутреннем буфере перед прекращением их чтения
    из underlying ресурса. Default=16kb, или 16 для  `objectMode` streams.
  * `encoding` {String} Если указано, буфер будет декодировать
    в strings используя эту кодировку. Default=null.

  * `objectMode` {Boolean} Будет ли этот stream вести себя
    как stream состоящий из объектов (как поток объектов - прим. переводчика).
    Означает, что stream.read(n) будет возвращать простое значение
    вместо буффера размером n. Default=false.

В классах, что расширяют Readable class,
всегда нужно вызывать его конструктор. Это необходимо
для провильной инициализации настроек буфферизации.

#### readable.\_read(size)

* `size` {Number} Количество байт для асинхронного считывания.

Внимание: **Реализуйте эту функцию, но никогда не вызывайте её на прямую.**

Эта функция предназначена для вызова внутренними Readable class методами,
её следует реализовать в вашем классе и никогда не вызывать её напрямую.

Любая реализации Readable stream должны обеспечивать `_read` для
получения данных из underlying ресурса.

Этот метод имеет нижнее подчеркивание (underscore) т.к. он
предназначен для внутренней системы и ожидается, что **вы
перегрузите** его (override) в своём классе, а пользователь
(вашим классом) никогда не будет вызывать его напрямую.

(Если не перегрузить - в момент инициализации stream
 stream пошлёт событие `error`, см. iojs/lib/_stream_readable.js
 - примеч. переводчика).

Когда данные доступны, положите их в очередь вызовом
`readable.push(chunk)`. Если `push` вернет false, вы должны
остановить чтение. Когда `_read` будет вызван снова,
вы должны стартовать передачу данных методом `push`.

Аргумент `size` здесь опцинален. Реализация, где
"read" - простой вызов, который возвращает данные,
может использовать `size`, что бы знать, как много отдавать данных.
Реализация, где это не важно, например в TCP или TLS,
возможно игнорировать этот аргумент и просто позволить
данным быть доступными в любой момент.
Также, нет необходимости, для примера, "ждать" до тех пор, пока
`size` байты будут доступны перед вызовом [`stream.push(chunk)`][].

#### readable.push(chunk[, encoding])

* `chunk` {Buffer | null | String} Chunk of data to push into the read queue
* `encoding` {String} Encoding of String chunks.  Must be a valid
  Buffer encoding, such as `'utf8'` or `'ascii'`
* return {Boolean} Whether or not more pushes should be performed

Note: **This function should be called by Readable implementors, NOT
by consumers of Readable streams.**

The `_read()` function will not be called again until at least one
`push(chunk)` call is made.

The `Readable` class works by putting data into a read queue to be
pulled out later by calling the `read()` method when the `'readable'`
event fires.

The `push()` method will explicitly insert some data into the read
queue.  If it is called with `null` then it will signal the end of the
data (EOF).

This API is designed to be as flexible as possible.  For example,
you may be wrapping a lower-level source which has some sort of
pause/resume mechanism, and a data callback.  In those cases, you
could wrap the low-level source object by doing something like this:

```javascript
// source is an object with readStop() and readStart() methods,
// and an `ondata` member that gets called when it has data, and
// an `onend` member that gets called when the data is over.

util.inherits(SourceWrapper, Readable);

function SourceWrapper(options) {
  Readable.call(this, options);

  this._source = getLowlevelSourceObject();
  var self = this;

  // Every time there's data, we push it into the internal buffer.
  this._source.ondata = function(chunk) {
    // if push() returns false, then we need to stop reading from source
    if (!self.push(chunk))
      self._source.readStop();
  };

  // When the source ends, we push the EOF-signaling `null` chunk
  this._source.onend = function() {
    self.push(null);
  };
}

// _read will be called when the stream wants to pull more data in
// the advisory size argument is ignored in this case.
SourceWrapper.prototype._read = function(size) {
  this._source.readStart();
};
```


### Class: stream.Writable

<!--type=class-->

`stream.Writable` is an abstract class designed to be extended with an
underlying implementation of the [`_write(chunk, encoding, callback)`][] method.

Please see above under [API для Пользователей Streams][] for how to consume
writable streams in your programs.  What follows is an explanation of
how to implement Writable streams in your programs.

#### new stream.Writable([options])

* `options` {Object}
  * `highWaterMark` {Number} Buffer level when [`write()`][] starts
    returning false. Default=16kb, or 16 for `objectMode` streams
  * `decodeStrings` {Boolean} Whether or not to decode strings into
    Buffers before passing them to [`_write()`][].  Default=true
  * `objectMode` {Boolean} Whether or not the `write(anyObj)` is
    a valid operation. If set you can write arbitrary data instead
    of only `Buffer` / `String` data.  Default=false

In classes that extend the Writable class, make sure to call the
constructor so that the buffering settings can be properly
initialized.

#### writable.\_write(chunk, encoding, callback)

* `chunk` {Buffer | String} The chunk to be written. Will **always**
  be a buffer unless the `decodeStrings` option was set to `false`.
* `encoding` {String} If the chunk is a string, then this is the
  encoding type. If chunk is a buffer, then this is the special
  value - 'buffer', ignore it in this case.
* `callback` {Function} Call this function (optionally with an error
  argument) when you are done processing the supplied chunk.

All Writable stream implementations must provide a [`_write()`][]
method to send data to the underlying resource.

Note: **This function MUST NOT be called directly.**  It should be
implemented by child classes, and called by the internal Writable
class methods only.

Call the callback using the standard `callback(error)` pattern to
signal that the write completed successfully or with an error.

If the `decodeStrings` flag is set in the constructor options, then
`chunk` may be a string rather than a Buffer, and `encoding` will
indicate the sort of string that it is.  This is to support
implementations that have an optimized handling for certain string
data encodings.  If you do not explicitly set the `decodeStrings`
option to `false`, then you can safely ignore the `encoding` argument,
and assume that `chunk` will always be a Buffer.

This method is prefixed with an underscore because it is internal to
the class that defines it, and should not be called directly by user
programs.  However, you **are** expected to override this method in
your own extension classes.

### writable.\_writev(chunks, callback)

* `chunks` {Array} The chunks to be written.  Each chunk has following
  format: `{ chunk: ..., encoding: ... }`.
* `callback` {Function} Call this function (optionally with an error
  argument) when you are done processing the supplied chunks.

Note: **This function MUST NOT be called directly.**  It may be
implemented by child classes, and called by the internal Writable
class methods only.

This function is completely optional to implement. In most cases it is
unnecessary.  If implemented, it will be called with all the chunks
that are buffered in the write queue.


### Class: stream.Duplex

<!--type=class-->

A "duplex" stream is one that is both Readable and Writable, such as a
TCP socket connection.

Note that `stream.Duplex` is an abstract class designed to be extended
with an underlying implementation of the `_read(size)` and
[`_write(chunk, encoding, callback)`][] methods as you would with a
Readable or Writable stream class.

Since JavaScript doesn't have multiple prototypal inheritance, this
class prototypally inherits from Readable, and then parasitically from
Writable.  It is thus up to the user to implement both the lowlevel
`_read(n)` method as well as the lowlevel
[`_write(chunk, encoding, callback)`][] method on extension duplex classes.

#### new stream.Duplex(options)

* `options` {Object} Passed to both Writable and Readable
  constructors. Also has the following fields:
  * `allowHalfOpen` {Boolean} Default=true.  If set to `false`, then
    the stream will automatically end the readable side when the
    writable side ends and vice versa.
  * `readableObjectMode` {Boolean} Default=false. Sets `objectMode`
    for readable side of the stream. Has no effect if `objectMode`
    is `true`.
  * `writableObjectMode` {Boolean} Default=false. Sets `objectMode`
    for writable side of the stream. Has no effect if `objectMode`
    is `true`.

In classes that extend the Duplex class, make sure to call the
constructor so that the buffering settings can be properly
initialized.


### Class: stream.Transform

A "transform" stream is a duplex stream where the output is causally
connected in some way to the input, such as a [zlib][] stream or a
[crypto][] stream.

There is no requirement that the output be the same size as the input,
the same number of chunks, or arrive at the same time.  For example, a
Hash stream will only ever have a single chunk of output which is
provided when the input is ended.  A zlib stream will produce output
that is either much smaller or much larger than its input.

Rather than implement the [`_read()`][] and [`_write()`][] methods, Transform
classes must implement the `_transform()` method, and may optionally
also implement the `_flush()` method.  (See below.)

#### new stream.Transform([options])

* `options` {Object} Passed to both Writable and Readable
  constructors.

In classes that extend the Transform class, make sure to call the
constructor so that the buffering settings can be properly
initialized.

#### transform.\_transform(chunk, encoding, callback)

* `chunk` {Buffer | String} The chunk to be transformed. Will **always**
  be a buffer unless the `decodeStrings` option was set to `false`.
* `encoding` {String} If the chunk is a string, then this is the
  encoding type. If chunk is a buffer, then this is the special
  value - 'buffer', ignore it in this case.
* `callback` {Function} Call this function (optionally with an error
  argument and data) when you are done processing the supplied chunk.

Note: **This function MUST NOT be called directly.**  It should be
implemented by child classes, and called by the internal Transform
class methods only.

All Transform stream implementations must provide a `_transform`
method to accept input and produce output.

`_transform` should do whatever has to be done in this specific
Transform class, to handle the bytes being written, and pass them off
to the readable portion of the interface.  Do asynchronous I/O,
process things, and so on.

Call `transform.push(outputChunk)` 0 or more times to generate output
from this input chunk, depending on how much data you want to output
as a result of this chunk.

Call the callback function only when the current chunk is completely
consumed.  Note that there may or may not be output as a result of any
particular input chunk. If you supply output as the second argument to the
callback, it will be passed to push method, in other words the following are
equivalent:

```javascript
transform.prototype._transform = function (data, encoding, callback) {
  this.push(data);
  callback();
}

transform.prototype._transform = function (data, encoding, callback) {
  callback(null, data);
}
```

This method is prefixed with an underscore because it is internal to
the class that defines it, and should not be called directly by user
programs.  However, you **are** expected to override this method in
your own extension classes.

#### transform.\_flush(callback)

* `callback` {Function} Call this function (optionally with an error
  argument) when you are done flushing any remaining data.

Note: **This function MUST NOT be called directly.**  It MAY be implemented
by child classes, and if so, will be called by the internal Transform
class methods only.

In some cases, your transform operation may need to emit a bit more
data at the end of the stream.  For example, a `Zlib` compression
stream will store up some internal state so that it can optimally
compress the output.  At the end, however, it needs to do the best it
can with what is left, so that the data will be complete.

In those cases, you can implement a `_flush` method, which will be
called at the very end, after all the written data is consumed, but
before emitting `end` to signal the end of the readable side.  Just
like with `_transform`, call `transform.push(chunk)` zero or more
times, as appropriate, and call `callback` when the flush operation is
complete.

This method is prefixed with an underscore because it is internal to
the class that defines it, and should not be called directly by user
programs.  However, you **are** expected to override this method in
your own extension classes.

#### Events: 'finish' and 'end'

The [`finish`][] and [`end`][] events are from the parent Writable
and Readable classes respectively. The `finish` event is fired after
`.end()` is called and all chunks have been processed by `_transform`,
`end` is fired after all data has been output which is after the callback
in `_flush` has been called.

#### Example: `SimpleProtocol` parser v2

The example above of a simple protocol parser can be implemented
simply by using the higher level [Transform][] stream class, similar to
the `parseHeader` and `SimpleProtocol v1` examples above.

In this example, rather than providing the input as an argument, it
would be piped into the parser, which is a more idiomatic io.js stream
approach.

```javascript
var util = require('util');
var Transform = require('stream').Transform;
util.inherits(SimpleProtocol, Transform);

function SimpleProtocol(options) {
  if (!(this instanceof SimpleProtocol))
    return new SimpleProtocol(options);

  Transform.call(this, options);
  this._inBody = false;
  this._sawFirstCr = false;
  this._rawHeader = [];
  this.header = null;
}

SimpleProtocol.prototype._transform = function(chunk, encoding, done) {
  if (!this._inBody) {
    // check if the chunk has a \n\n
    var split = -1;
    for (var i = 0; i < chunk.length; i++) {
      if (chunk[i] === 10) { // '\n'
        if (this._sawFirstCr) {
          split = i;
          break;
        } else {
          this._sawFirstCr = true;
        }
      } else {
        this._sawFirstCr = false;
      }
    }

    if (split === -1) {
      // still waiting for the \n\n
      // stash the chunk, and try again.
      this._rawHeader.push(chunk);
    } else {
      this._inBody = true;
      var h = chunk.slice(0, split);
      this._rawHeader.push(h);
      var header = Buffer.concat(this._rawHeader).toString();
      try {
        this.header = JSON.parse(header);
      } catch (er) {
        this.emit('error', new Error('invalid simple protocol data'));
        return;
      }
      // and let them know that we are done parsing the header.
      this.emit('header', this.header);

      // now, because we got some extra data, emit this first.
      this.push(chunk.slice(split));
    }
  } else {
    // from there on, just provide the data to our consumer as-is.
    this.push(chunk);
  }
  done();
};

// Usage:
// var parser = new SimpleProtocol();
// source.pipe(parser)
// Now parser is a readable stream that will emit 'header'
// with the parsed header data.
```


### Class: stream.PassThrough

This is a trivial implementation of a [Transform][] stream that simply
passes the input bytes across to the output.  Its purpose is mainly
for examples and testing, but there are occasionally use cases where
it can come in handy as a building block for novel sorts of streams.


## Simplified Constructor API 

<!--type=misc-->

In simple cases there is now the added benefit of being able to construct a stream without inheritance.

This can be done by passing the appropriate methods as constructor options:

Examples:

### Readable
```javascript
var readable = new stream.Readable({
  read: function(n) {
    // sets this._read under the hood
  }
});
```

### Writable
```javascript
var writable = new stream.Writable({
  write: function(chunk, encoding, next) {
    // sets this._write under the hood
  }
});

// or

var writable = new stream.Writable({
  writev: function(chunks, next) {
    // sets this._writev under the hood
  }
});
```

### Duplex
```javascript
var duplex = new stream.Duplex({
  read: function(n) {
    // sets this._read under the hood
  },
  write: function(chunk, encoding, next) {
    // sets this._write under the hood
  }
});

// or

var duplex = new stream.Duplex({
  read: function(n) {
    // sets this._read under the hood
  },
  writev: function(chunks, next) {
    // sets this._writev under the hood
  }
});
```

### Transform
```javascript
var transform = new stream.Transform({
  transform: function(chunk, encoding, next) {
    // sets this._transform under the hood
  },
  flush: function(done) {
    // sets this._flush under the hood
  }
});
```

## Streams: Under the Hood

<!--type=misc-->

### Buffering

<!--type=misc-->

Both Writable and Readable streams will buffer data on an internal
object called `_writableState.buffer` or `_readableState.buffer`,
respectively.

The amount of data that will potentially be buffered depends on the
`highWaterMark` option which is passed into the constructor.

Buffering in Readable streams happens when the implementation calls
[`stream.push(chunk)`][].  If the consumer of the Stream does not call
`stream.read()`, then the data will sit in the internal queue until it
is consumed.

Buffering in Writable streams happens when the user calls
[`stream.write(chunk)`][] repeatedly, even when `write()` returns `false`.

The purpose of streams, especially with the `pipe()` method, is to
limit the buffering of data to acceptable levels, so that sources and
destinations of varying speed will not overwhelm the available memory.

### `stream.read(0)`

There are some cases where you want to trigger a refresh of the
underlying readable stream mechanisms, without actually consuming any
data.  In that case, you can call `stream.read(0)`, which will always
return null.

If the internal read buffer is below the `highWaterMark`, and the
stream is not currently reading, then calling `read(0)` will trigger
a low-level `_read` call.

There is almost never a need to do this.  However, you will see some
cases in io.js's internals where this is done, particularly in the
Readable stream class internals.

### `stream.push('')`

Pushing a zero-byte string or Buffer (when not in [Object mode][]) has an
interesting side effect.  Because it *is* a call to
[`stream.push()`][], it will end the `reading` process.  However, it
does *not* add any data to the readable buffer, so there's nothing for
a user to consume.

Very rarely, there are cases where you have no data to provide now,
but the consumer of your stream (or, perhaps, another bit of your own
code) will know when to check again, by calling `stream.read(0)`.  In
those cases, you *may* call `stream.push('')`.

So far, the only use case for this functionality is in the
[tls.CryptoStream][] class, which is deprecated in io.js v1.0.  If you
find that you have to use `stream.push('')`, please consider another
approach, because it almost certainly indicates that something is
horribly wrong.

### Compatibility with Older Node.js Versions

<!--type=misc-->

In versions of Node.js prior to v0.10, the Readable stream interface was
simpler, but also less powerful and less useful.

* Rather than waiting for you to call the `read()` method, `'data'`
  events would start emitting immediately.  If you needed to do some
  I/O to decide how to handle data, then you had to store the chunks
  in some kind of buffer so that they would not be lost.
* The [`pause()`][] method was advisory, rather than guaranteed.  This
  meant that you still had to be prepared to receive `'data'` events
  even when the stream was in a paused state.

In io.js v1.0 and Node.js v0.10, the Readable class described below was added.
For backwards compatibility with older Node.js programs, Readable streams
switch into "flowing mode" when a `'data'` event handler is added, or
when the [`resume()`][] method is called.  The effect is that, even if
you are not using the new `read()` method and `'readable'` event, you
no longer have to worry about losing `'data'` chunks.

Most programs will continue to function normally.  However, this
introduces an edge case in the following conditions:

* No [`'data'` event][] handler is added.
* The [`resume()`][] method is never called.
* The stream is not piped to any writable destination.

For example, consider the following code:

```javascript
// WARNING!  BROKEN!
net.createServer(function(socket) {

  // we add an 'end' method, but never consume the data
  socket.on('end', function() {
    // It will never get here.
    socket.end('I got your message (but didnt read it)\n');
  });

}).listen(1337);
```

In versions of Node.js prior to v0.10, the incoming message data would be
simply discarded.  However, in io.js v1.0 and Node.js v0.10 and beyond,
the socket will remain paused forever.

The workaround in this situation is to call the `resume()` method to
start the flow of data:

```javascript
// Workaround
net.createServer(function(socket) {

  socket.on('end', function() {
    socket.end('I got your message (but didnt read it)\n');
  });

  // start the flow of data, discarding it.
  socket.resume();

}).listen(1337);
```

In addition to new Readable streams switching into flowing mode,
pre-v0.10 style streams can be wrapped in a Readable class using the
`wrap()` method.


### Object Mode

<!--type=misc-->

Normally, Streams operate on Strings and Buffers exclusively.

Streams that are in **object mode** can emit generic JavaScript values
other than Buffers and Strings.

A Readable stream in object mode will always return a single item from
a call to `stream.read(size)`, regardless of what the size argument
is.

A Writable stream in object mode will always ignore the `encoding`
argument to `stream.write(data, encoding)`.

The special value `null` still retains its special value for object
mode streams.  That is, for object mode readable streams, `null` as a
return value from `stream.read()` indicates that there is no more
data, and [`stream.push(null)`][] will signal the end of stream data
(`EOF`).

No streams in io.js core are object mode streams.  This pattern is only
used by userland streaming libraries.

You should set `objectMode` in your stream child class constructor on
the options object.  Setting `objectMode` mid-stream is not safe.

For Duplex streams `objectMode` can be set exclusively for readable or
writable side with `readableObjectMode` and `writableObjectMode`
respectively. These options can be used to implement parsers and
serializers with Transform streams.

```javascript
var util = require('util');
var StringDecoder = require('string_decoder').StringDecoder;
var Transform = require('stream').Transform;
util.inherits(JSONParseStream, Transform);

// Gets \n-delimited JSON string data, and emits the parsed objects
function JSONParseStream() {
  if (!(this instanceof JSONParseStream))
    return new JSONParseStream();

  Transform.call(this, { readableObjectMode : true });

  this._buffer = '';
  this._decoder = new StringDecoder('utf8');
}

JSONParseStream.prototype._transform = function(chunk, encoding, cb) {
  this._buffer += this._decoder.write(chunk);
  // split on newlines
  var lines = this._buffer.split(/\r?\n/);
  // keep the last partial line buffered
  this._buffer = lines.pop();
  for (var l = 0; l < lines.length; l++) {
    var line = lines[l];
    try {
      var obj = JSON.parse(line);
    } catch (er) {
      this.emit('error', er);
      return;
    }
    // push the parsed object out to the readable consumer
    this.push(obj);
  }
  cb();
};

JSONParseStream.prototype._flush = function(cb) {
  // Just handle any leftover
  var rem = this._buffer.trim();
  if (rem) {
    try {
      var obj = JSON.parse(rem);
    } catch (er) {
      this.emit('error', er);
      return;
    }
    // push the parsed object out to the readable consumer
    this.push(obj);
  }
  cb();
};
```


[EventEmitter]: events.html#events_class_events_eventemitter
[Object mode]: #stream_object_mode
[`stream.push(chunk)`]: #stream_readable_push_chunk_encoding
[`stream.push(null)`]: #stream_readable_push_chunk_encoding
[`stream.push()`]: #stream_readable_push_chunk_encoding
[`unpipe()`]: #stream_readable_unpipe_destination
[unpiped]: #stream_readable_unpipe_destination
[tcp sockets]: net.html#net_class_net_socket
[zlib streams]: zlib.html
[zlib]: zlib.html
[crypto streams]: crypto.html
[crypto]: crypto.html
[tls.CryptoStream]: tls.html#tls_class_cryptostream
[process.stdin]: process.html#process_process_stdin
[stdout]: process.html#process_process_stdout
[process.stdout]: process.html#process_process_stdout
[process.stderr]: process.html#process_process_stderr
[child process stdout and stderr]: child_process.html#child_process_child_stdout
[API для Пользователей Streams]: #stream_api_for_stream_consumers
[API для реализации Stream]: #stream_api_for_stream_implementors
[Readable]: #stream_class_stream_readable
[Writable]: #stream_class_stream_writable
[Duplex]: #stream_class_stream_duplex
[Transform]: #stream_class_stream_transform
[`end`]: #stream_event_end
[`finish`]: #stream_event_finish
[`_read(size)`]: #stream_readable_read_size_1
[`_read()`]: #stream_readable_read_size_1
[_read]: #stream_readable_read_size_1
[`writable.write(chunk)`]: #stream_writable_write_chunk_encoding_callback
[`write(chunk, encoding, callback)`]: #stream_writable_write_chunk_encoding_callback
[`write()`]: #stream_writable_write_chunk_encoding_callback
[`stream.write(chunk)`]: #stream_writable_write_chunk_encoding_callback
[`_write(chunk, encoding, callback)`]: #stream_writable_write_chunk_encoding_callback_1
[`_write()`]: #stream_writable_write_chunk_encoding_callback_1
[_write]: #stream_writable_write_chunk_encoding_callback_1
[`util.inherits`]: util.html#util_util_inherits_constructor_superconstructor
[`end()`]: #stream_writable_end_chunk_encoding_callback
[`'data'` event]: #stream_event_data
[`resume()`]: #stream_readable_resume
[`readable.resume()`]: #stream_readable_resume
[`pause()`]: #stream_readable_pause
[`unpipe()`]: #stream_readable_unpipe_destination
[`pipe()`]: #stream_readable_pipe_destination_options
