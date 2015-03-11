# Zlib

    Стабильность: 3 - Стабилен

Вы можете получить доступ к этому модулю следующим образом:

    var zlib = require('zlib');

Данный модуль предоставляет доступ к таким классам операций как
Gzip/Gunzip, Deflate/Inflate и DeflateRaw/InflateRaw. Каждый класс
принимает одинаковый набор параметров и является читаемым/записываемым
Потоком (Stream).

## Примеры

Сжатие и распаковка некоторого файла может быть выполнена
путем соединения потоков fs.ReadStream, zlib и fs.WriteStream
при помощи операции pipe.

    var gzip = zlib.createGzip();
    var fs = require('fs');
    var inp = fs.createReadStream('input.txt');
    var out = fs.createWriteStream('input.txt.gz');

    inp.pipe(gzip).pipe(out);

Сжатие и распаковка данных может быть выполнена
в один шаг при помощи удобных методов.

    var input = '.................................';
    zlib.deflate(input, function(err, buffer) {
      if (!err) {
        console.log(buffer.toString('base64'));
      }
    });

    var buffer = new Buffer('eJzT0yMAAGTvBe8=', 'base64');
    zlib.unzip(buffer, function(err, buffer) {
      if (!err) {
        console.log(buffer.toString());
      }
    });

Для использования данного модуля в HTTP-клиенте или HTTP-сервере используйте заголовок
[accept-encoding](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.3)
при формировании запроса, и заголовок
[content-encoding](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.11)
при формировании ответа.

**Примечание: данные примеры предельно упрощены для понимания основ**.
Zlib кодирование может быть дорогой операцией и полученные результаты
должны быть закешированы.
Смотрите [Настройка использования памяти](#zlib_memory_usage_tuning) для
получение детальной информации о компромиссах скорости/памяти/компрессии
при использовании zlib.

    // пример формирования запроса на стороне клиента
    var zlib = require('zlib');
    var http = require('http');
    var fs = require('fs');
    var request = http.get({ host: 'izs.me',
                             path: '/',
                             port: 80,
                             headers: { 'accept-encoding': 'gzip,deflate' } });
    request.on('response', function(response) {
      var output = fs.createWriteStream('izs.me_index.html');

      switch (response.headers['content-encoding']) {
        // or, just use zlib.createUnzip() to handle both cases
        case 'gzip':
          response.pipe(zlib.createGunzip()).pipe(output);
          break;
        case 'deflate':
          response.pipe(zlib.createInflate()).pipe(output);
          break;
        default:
          response.pipe(output);
          break;
      }
    });

    // пример формирования сервером ответа на запрос от клиента
    // Сжатие каждого запроса может быть очень ресурсоёмко.
    // Более эффективным решением будет хранить однажды сжатые
    // данные в кеше.
    var zlib = require('zlib');
    var http = require('http');
    var fs = require('fs');
    http.createServer(function(request, response) {
      var raw = fs.createReadStream('index.html');
      var acceptEncoding = request.headers['accept-encoding'];
      if (!acceptEncoding) {
        acceptEncoding = '';
      }

      // Примечание: Данный код не является совместимым
      // парсером заголовка accept-encoding
      // Смотрите http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.3
      if (acceptEncoding.match(/\bdeflate\b/)) {
        response.writeHead(200, { 'content-encoding': 'deflate' });
        raw.pipe(zlib.createDeflate()).pipe(response);
      } else if (acceptEncoding.match(/\bgzip\b/)) {
        response.writeHead(200, { 'content-encoding': 'gzip' });
        raw.pipe(zlib.createGzip()).pipe(response);
      } else {
        response.writeHead(200, {});
        raw.pipe(response);
      }
    }).listen(1337);

## zlib.createGzip([параметры])

Возвращает новый объект [Gzip](#zlib_class_zlib_gzip)
с параметрами [параметры](#zlib_options).

## zlib.createGunzip([параметры])

Возвращает новый объект [Gunzip](#zlib_class_zlib_gunzip)
с параметрами [параметры](#zlib_options).

## zlib.createDeflate([параметры])

Возвращает новый объект [Deflate](#zlib_class_zlib_deflate)
с параметрами [параметры](#zlib_options).

## zlib.createInflate([параметры])

Возвращает новый объект [Inflate](#zlib_class_zlib_inflate)
с параметрами [параметры](#zlib_options).

## zlib.createDeflateRaw([параметры])

Возвращает новый объект [DeflateRaw](#zlib_class_zlib_deflateraw)
с параметрами [параметры](#zlib_options).

## zlib.createInflateRaw([параметры])

Возвращает новый объект [InflateRaw](#zlib_class_zlib_inflateraw)
с параметрами [параметры](#zlib_options).

## zlib.createUnzip([параметры])

Возвращает новый объект [Unzip](#zlib_class_zlib_unzip)
с параметрами [параметры](#zlib_options).


## Класс: zlib.Zlib

Не экспортируется модулем 'zlib'. Данный класс описан здесь, потому что
является базовым классом для классов сжатия и распаковки.

### zlib.flush([тип], callback)

`тип` по умолчанию равен `zlib.Z_FULL_FLUSH`.

Сброс ожидаемых данных. Не стоит без необходимости вызывать данную функцию,
преждевременные сбросы негативно влияют на эффективность алгоритма компрессии.

### zlib.params(уровень, стратегия, callback)

Динамическое обновление уровня и стратегии сжатия.
Применимо только для алгоритма deflate.

### zlib.reset()

Сбросить настройки сжатия/распаковки в значения по умолчанию.
Применимо только для алгоритмов inflate и deflate.

## Класс: zlib.Gzip

Сжатие данных используя алгоритм gzip.

## Класс: zlib.Gunzip

Распаковка потока данных, упакованных при помощи алгоритма gzip.

## Класс: zlib.Deflate

Сжатие данных используя алгоритм deflate.

## Класс: zlib.Inflate

Распаковка потока данных, сжатых при помощи алгоритма deflate.

## Класс: zlib.DeflateRaw

Сжатие данных используя алгоритм deflate без добавления заголовка zlib.

## Класс: zlib.InflateRaw

Распаковка потока данных, сжатых при помощи алгоритма deflate
без добавления заголовка zlib

## Класс: zlib.Unzip

Распаковка потока данных, сжатых при помощи алгоритмов gzip или deflate.
Алгоритм распаковки выбирается на основе заголовка.

## Удобные методы

<!--type=misc-->

Все ниже указанные методы принимают на вход строку или буфер в качестве
первого аргумента, опционально в качестве второго аргумента принимаются
параметры для zlib классов и функцию обратного вызова, которая будет вызвана
как `callback(error, result)`.

Каждый метод имеет синхронный вариант реализации. Принимаемые параметры
такие же как и у асинхронного варианта, но без функции обратного вызова.

## zlib.deflate(buf[, options], callback)
## zlib.deflateSync(buf[, options])

Сжатие строки или буфера используя алгоритм deflate.

## zlib.deflateRaw(buf[, options], callback)
## zlib.deflateRawSync(buf[, options])

Сжатие строки или буфера используя алгоритм deflate
без добавления заголовка.

## zlib.gzip(buf[, options], callback)
## zlib.gzipSync(buf[, options])

Сжатие строки или буфера используя алгоритм gzip.

## zlib.gunzip(buf[, options], callback)
## zlib.gunzipSync(buf[, options])

Распаковка буфера с данными, сжатыми
при помощи алгоритма gzip.

## zlib.inflate(buf[, options], callback)
## zlib.inflateSync(buf[, options])

Распаковка буфера с данными, сжатыми
при помощи алгоритма deflate.

## zlib.inflateRaw(buf[, options], callback)
## zlib.inflateRawSync(buf[, options])

Распаковка буфера с данными, сжатыми
при помощи алгоритма deflate без добавления заголовка.

## zlib.unzip(buf[, options], callback)
## zlib.unzipSync(buf[, options])

Распаковка буфера с данными, сжатыми
при помощи алгоритма zip.

## Параметры

<!--type=misc-->

Каждый класс принимает параметры в качестве объекта.
Все параметры являются не обязательными.

Обратите внимание что некоторые параметры имеют смысл только
для сжатия и будут проигнорированы при распаковке.

* flush (по умолчанию: `zlib.Z_NO_FLUSH`)
* chunkSize (по умолчанию: 16*1024)
* windowBits
* level (только для компрессии)
* memLevel (только для компрессии) 
* strategy (только для компрессии)
* dictionary (только для алгоритма deflate/inflate, по умолчанию словарь пустой)

Для получения более подробной информации касательно 'deflateInit2' и 'inflateInit2'
смотри <http://zlib.net/manual.html#Advanced>.

## Настройка использования памяти

<!--type=misc-->

Из `zlib/zconf.h`, с изменениями для использования в io.js's:

Количество памяти используемое для операции deflate (в байтах):

    (1 << (windowBits+2)) +  (1 << (memLevel+9))

это 128K для windowBits=15  +  128K для memLevel = 8
(значения по умолчанию) плюс несколько килобайт для мелких объектов.

Например если Вы хотите уменьшить количество используемой по умолчанию
памяти с 256K до 128K, то установите опции следующим образом:

    { windowBits: 14, memLevel: 7 }

Конечно же данные действия в общем случае ухудшат компрессию
(бесплатных завтраков не бывает).

Количество памяти используемое для операции inflate (в байтах):

    1 << windowBits

это 32K для windowBits=15 (значение по умолчанию) плюс несколько
килобайт для мелких объектов.

Это в дополнение к одному внутреннему выходному буферу, размер которого
равен `chunkSize` (значение по умолчанию 16K).

Скорость zlib сжатия в большей степени зависит от уровня сжатия
(параметр `level`). Высокий уровень сжатия дает на выходе лучшее
сжатие данных, но требует больше времени на завершение, низкий уровень
дает на выходе не такие хорошие результаты в сжатии данных, но гораздо быстрее.

В общем случае, большее количество используемой памяти дает возможность zlib
обрабатывать больше данных за одну операцию записи, что ведет к тому что
io.js будет меньшее количество раз вызывать zlib. И это является ещё одним
фактором, который влияет на скорость.

## Константы

<!--type=misc-->

Все константы определённые в файле zlib.h также определены в модуле.

В ходе обычных операции Вам не нужно когда либо выставлять какую либо из них.
Все они описаны здесь, так что их присутствие не является сюрпризом. Данная секция
взята напрямую из [документации к zlib](http://zlib.net/manual.html#Constants).
Для более подробной информации смотри <http://zlib.net/manual.html#Constants>.

Возможные значения параметра flush.

* `zlib.Z_NO_FLUSH`
* `zlib.Z_PARTIAL_FLUSH`
* `zlib.Z_SYNC_FLUSH`
* `zlib.Z_FULL_FLUSH`
* `zlib.Z_FINISH`
* `zlib.Z_BLOCK`
* `zlib.Z_TREES`

Возвращаемые значения для функции сжатия и распаковки.
Отрицательные значение трактуются как ошибка, положительные
трактуются как специальные, но не ошибочные события.

* `zlib.Z_OK`
* `zlib.Z_STREAM_END`
* `zlib.Z_NEED_DICT`
* `zlib.Z_ERRNO`
* `zlib.Z_STREAM_ERROR`
* `zlib.Z_DATA_ERROR`
* `zlib.Z_MEM_ERROR`
* `zlib.Z_BUF_ERROR`
* `zlib.Z_VERSION_ERROR`

Уровень компрессии.

* `zlib.Z_NO_COMPRESSION`
* `zlib.Z_BEST_SPEED`
* `zlib.Z_BEST_COMPRESSION`
* `zlib.Z_DEFAULT_COMPRESSION`

Стратегия компрессии.

* `zlib.Z_FILTERED`
* `zlib.Z_HUFFMAN_ONLY`
* `zlib.Z_RLE`
* `zlib.Z_FIXED`
* `zlib.Z_DEFAULT_STRATEGY`

Возможные значения поля data_type.

* `zlib.Z_BINARY`
* `zlib.Z_TEXT`
* `zlib.Z_ASCII`
* `zlib.Z_UNKNOWN`

Метод сжатия для алгоритма deflate (только один поддерживается в данной версии).

* `zlib.Z_DEFLATED`

Для инициализации zalloc, zfree, opaque.

* `zlib.Z_NULL`
