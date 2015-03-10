# Path

    Стабильность: 3 - Стабильный
    

Этот модуль содержит утилиты для обработки и преобразования путей файлов. Почти все эти методы выполняют только строковые преобразования. Корректность путей не проверяется через файловую систему.

Для доступа к модулю используйте `require('path')`. Доступны следующие методы:

## path.normalize(p)

Нормализует строковый путь, заботясь о фрагментах `'..'` и `'.'`.

При нахождении нескольких косых черт, заменяет их на одну; когда путь содержит последнюю косую черту, она сохраняется. В Windows используются обратные косые черты.

Пример:

    path.normalize('/foo/bar//baz/asdf/quux/..')
    // возвращает
    '/foo/bar/baz/asdf'
    

## path.join(\[path1\]\[, path2\][, ...])

Объединяет все аргументы и нормализует результирующий путь.

Аргументы должны быть в строковом формате. В версии 0.8, не строковые аргументы игнорировались. В версиях 0.10 и выше возникает исключение.

Пример:

    path.join('/foo', 'bar', 'baz/asdf', 'quux', '..')
    // возвращает
    '/foo/bar/baz/asdf'
    
    path.join('foo', {}, 'bar')
    // возникает исключение
    TypeError: Arguments to path.join must be strings
    

## path.resolve([from ...], to)

Преобразует `to` в абсолютный путь.

Если `to` не является абсолютным, аргументы `from` перебираются справа налево до тех пор, пока не будет найден абсолютный путь. Если после использования всех путей `from` абсолютный путь не найден, то используется текущий рабочий каталог. В результате путь нормализуется и последняя косая черта удаляется, если только не получится корневой каталог. Не строковые аргументы `from` игнорируются.

Метод может быть представлен как последовательность команд `cd` в командной строке (терминале).

    path.resolve('foo/bar', '/tmp/file/', '..', 'a/../subfile')
    

Похож на:

    cd foo/bar
    cd /tmp/file/
    cd ..
    cd a/../subfile
    pwd
    

Разница в том, что различные пути могут и не существовать, ими могут быть и файлы.

Примеры:

    path.resolve('/foo/bar', './baz')
    // возвращает
    '/foo/bar/baz'
    
    path.resolve('/foo/bar', '/tmp/file/')
    // возвращает
    '/tmp/file'
    
    path.resolve('wwwroot', 'static_files/png/', '../gif/image.gif')
    // если на данный момент путь /home/myself/iojs, то возвращает
    '/home/myself/iojs/wwwroot/static_files/gif/image.gif'
    

## path.isAbsolute(path)

Определяет, является ли `path` абсолютным путем. Абсолютный путь всегда будет определен для данного места, независимо от рабочего каталога.

POSIX примеры:

    path.isAbsolute('/foo/bar') // true
    path.isAbsolute('/baz/..')  // true
    path.isAbsolute('qux/')     // false
    path.isAbsolute('.')        // false
    

Windows примеры:

    path.isAbsolute('//server')  // true
    path.isAbsolute('C:/foo/..') // true
    path.isAbsolute('bar\\baz')   // false
    path.isAbsolute('.')         // false
    

## path.relative(from, to)

Определяет относительный путь от `from` до `to`.

Иногда мы имеем 2 абсолютных пути, и нам необходимо получить относительный путь от одного к другому. Данный способ является обратным преобразованием `path.resolve`, в котором мы видим:

    path.resolve(from, path.relative(from, to)) == path.resolve(to)
    

Примеры:

    path.relative('C:\\orandea\\test\\aaa', 'C:\\orandea\\impl\\bbb')
    // возвращает
    '..\\..\\impl\\bbb'
    
    path.relative('/data/orandea/test/aaa', '/data/orandea/impl/bbb')
    // возвращает
    '../../impl/bbb'
    

## path.dirname(p)

Возвращает название каталога пути. Аналогична Unix команде `dirname`.

Пример:

    path.dirname('/foo/bar/baz/asdf/quux')
    // возвращает
    '/foo/bar/baz/asdf'
    

## path.basename(p[, ext])

Возвращает последнюю часть пути. Подобно Unix команде `basename`.

Пример:

    path.basename('/foo/bar/baz/asdf/quux.html')
    // возвращает
    'quux.html'
    
    path.basename('/foo/bar/baz/asdf/quux.html', '.html')
    // возвращает
    'quux'
    

## path.extname(p)

Возвращает расширение пути, от последней '.' в конец строки последней части пути. Если нет символа '.' в последней части пути или первый символ является '.', то возвращается пустая строка. Примеры:

    path.extname('index.html')
    // возвращает
    '.html'
    
    path.extname('index.coffee.md')
    // возвращает
    '.md'
    
    path.extname('index.')
    // возвращает
    '.'
    
    path.extname('index')
    // возвращает
    ''
    

## path.sep

Платформенный разделитель файлов. `'\\'` или `'/'` .

Пример на *nix системах:

    'foo/bar/baz'.split(path.sep)
    // возвращает
    ['foo', 'bar', 'baz']
    

Пример на Windows:

    'foo\\bar\\baz'.split(path.sep)
    // возвращает
    ['foo', 'bar', 'baz']
    

## path.delimiter

Платформенный разделитель путей, `;` или `':'`.

Пример на *nix системах:

    console.log(process.env.PATH)
    // '/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin'
    
    process.env.PATH.split(path.delimiter)
    // возвращает
    ['/usr/bin', '/bin', '/usr/sbin', '/sbin', '/usr/local/bin']
    

Пример на Windows:

    console.log(process.env.PATH)
    // 'C:\Windows\system32;C:\Windows;C:\Program Files\iojs\'
    
    process.env.PATH.split(path.delimiter)
    // возвращает
    ['C:\Windows\system32', 'C:\Windows', 'C:\Program Files\iojs\']
    

## path.parse(pathString)

Возвращает объект из строки пути.

Пример на *nix системах:

    path.parse('/home/user/dir/file.txt')
    // возвращает
    {
        root : "/",
        dir : "/home/user/dir",
        base : "file.txt",
        ext : ".txt",
        name : "file"
    }
    

Пример на Windows:

    path.parse('C:\\path\\dir\\index.html')
    // возвращает
    {
        root : "C:\",
        dir : "C:\path\dir",
        base : "index.html",
        ext : ".html",
        name : "index"
    }
    

## path.format(pathObject)

Возвращает строку пути из объекта, в противоположность `path.parse` приведенному выше.

    path.format({
        root : "/",
        dir : "/home/user/dir",
        base : "file.txt",
        ext : ".txt",
        name : "file"
    })
    // возвращает
    '/home/user/dir/file.txt'
    

## path.posix

Предоставляет доступ к методам вышеупомянутого `path`, но всегда взаимодействуют в posix совместимым способом.

## path.win32

Предоставляет доступ к методам вышеупомянутого `path`, но всегда взаимодействуют в win32 совместимым способом.